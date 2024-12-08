---
layout: post
title: "One Technique for Fixing/Preventing Flaky Tests"
date: 2024-12-08 09:48:35 +0100
tags: testing
---

Often, as a test engineer, you have to manifest and fix flaky tests regardless of how great your test automation is.

According to the 2021 study «**[An Empirical Analysis of UI-based Flaky Tests](https://arxiv.org/abs/2103.02669)**» [1], «_increasing delays between actions_» and «_adding the `await` keyword where needed_» are among the popular solutions for fixing flakiness. But instead of these superficial ways, in some particular cases, **the fix can be done by waiting for some conditions of an element just related to the tested one.**

Let’s take a look at one example of a flaky case…

The autotest has only a few steps:

1. Click on [Open smth] button → popover and [Add geobar] should appear;
2. Click on [Add geobar] button;
3. Check for a new entity that appeared on the map.

```javascript
// Example code snippet on Playwright
await page.getByRole("button", { name: "Open smth" }).click();
await page.getByRole("button", { name: "Add geobar" }).click();
await expect(page.getByTestId("a-new-entity")).toBeVisible();
```

In the second step, the test occasionally fails because sometimes the autotest framework (Playwright) successfully clicks on [Add geobar]. Still, sometimes, the click misses or does not trigger any action.

The root problem of that flaky step is that when the user/autotest framework clicks on [Open smth] button, the popover with the [Add geobar] button **_opens with smooth animation_**. At the moment of the appearance of the [Add geobar] button, most of the test frameworks have already considered the [Add geobar] button to be [actionable](https://playwright.dev/docs/actionability) and have tried to click on it. Sometimes they succeed, and sometimes they do not.

![Fig. 1.](/assets/2024-12-08/01.jpg)

_Fig. 1. User clicks on [Open smth] button, and the popover with the [Add geobar] button opens with smooth animation. [Add geobar] is already actionable, but the click may fail._

![Fig. 2.](/assets/2024-12-08/02.jpg)

_Fig. 2. The popover with the [Add geobar] button continues to open with smooth animation. [Add geobar] is already actionable, but the click may fail._

![Fig. 3.](/assets/2024-12-08/03.jpg)

_Fig. 3. The popover with the [Add geobar] opened. [Add geobar] is actionable, and the click will be successful._

There are a few ways to fix this; to make the [Add geobar] button [deterministically](https://en.wikipedia.org/wiki/Deterministic_system#In_computer_science) clickable.

1. Add delay or unconditional timeout/sleep before a click, like [`waitForTimeout()`](https://playwright.dev/docs/api/class-page#page-wait-for-timeout);
2. Wait for a particular state of the clickable element before clicking on it (which may be difficult if the code does not explicitly reflect its state).

Adding delay does not fix the root cause of the flakiness; it just patches an autotest code and does not guarantee that the test will not fail in the future because of exceeding a set timeout/sleep. Unconditional timeouts are considered a bad practice even though they [help in some despairing cases](https://adequatica.github.io/2024/09/04/timeouts-against-flaky-tests-true-cases-with-playwright.html).

The strategy of waiting for an element’s state [is much better](https://adequatica.github.io/2022/09/20/principles-of-writing-automated-tests.html#4-no-unconditional-expectation). But what if the element is already actionable as soon it appears, as was mentioned above happened with [Add geobar]?

Some QA engineers concentrate too much only on the single element being tested. No matter how correct the ideas of independence and atomicity of checks are (by the way, they are definitely correct), the code of modern dynamic web applications is highly complicated, and its components are intertwined.

So, using the case example under consideration, let’s inspect other components related to the element under test. The [Add geobar] button depends on the popover, while the state of the popover’s content is definable!

Thus, here is a third way of fixing this case: **wait for a particular state of the related element before clicking on the desired one.**

![Fig. 4.](/assets/2024-12-08/04.jpg)

_Fig. 4. The popover state is defined as OK. Click on [Add geobar] button will be successful because it is already actionable, too._

In the rewritten form, the autotest will have the following steps:

1. Click on [Open smth] button → popover and [Add geobar] should appear;
2. Wait for the popover’s content to be rendered;
3. Click on [Add geobar] button;
4. Check for a new entity that appeared on the map.

```javascript
// Example code snippet on Playwright
await page.getByRole("button", { name: "Open smth" }).click();
await expect(page.getByText("Tell me, O muse")).toBeVisible();
await page.getByRole("button", { name: "Add geobar" }).click();
await expect(page.getByTestId("a-new-entity")).toBeVisible();
```

It is worth clarifying that this method requires the test engineer to have an excellent understanding of the application’s architecture under test. The right choice of related elements is crucial.

---

According to the study mentioned at the beginning of the article, this strategy of fixing flaky tests may be attributed to the «Refactor Logic Implementation» category when performing checks were changed or reordered. Also, the benefit of executing some code rather than simply using `sleep` is pinpointed in the 2014 study «**[An Empirical Analysis of Flaky Tests](https://mir.cs.illinois.edu/marinov/publications/LuoETAL14FlakyTestsAnalysis.pdf)**» [2].

References:

1. A. Romano, Z. Song, S. Grandhi, W. Yang, and W. Wang, “An Empirical Analysis of UI-based Flaky Tests,” in Proceedings of the 43rd International Conference on Software Engineering, 2021. https://doi.org/10.48550/arXiv.2103.02669
2. Q. Luo, F. Hariri, L. Eloussi, and D. Marinov, “An empirical analysis of flaky tests,” in Proceedings of the 22nd ACM SIGSOFT International Symposium on Foundations of Software Engineering, 2014. https://doi.org/10.1145/2635868.2635920

Copy @ [Medium](https://adequatica.medium.com/one-technique-for-fixing-preventing-flaky-tests-afbfd4f46639)
