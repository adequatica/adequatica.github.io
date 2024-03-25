---
layout: post
title: "Automated Accessibility Testing of Keyboard Navigation on Tab"
date: 2024-03-25 09:58:30 +0100
tags: accessibility testing
---

One of the first and easiest ways to start accessibility testing on the site is to navigate through the page using just a keyboard.

![DALL-E 3 prompt](/assets/2024-03-25/00-cover-dall-e-3.png)

_DALL-E 3 prompt: make minimalistic illustration of accessible website with keyboard navigation. Do not use any text_

## Introduction to Accessible Keyboard Navigation

[Testing your website’s keyboard navigation](https://medium.com/@adequatica/accessibility-manual-testing-85826e161071#d6b0) functionality will help guarantee accessibility for users who rely on keyboards. Usually, keyboard navigation is performed by pressing the [TAB] key, which moves focus between interactive elements, and pressing [ENTER] interacts with them.

Proper accessible keyboard navigation implementation benefits all users regardless of which disabilities (physical or technical) they may have.

A good keyboard navigation includes several aspects:

1. Focused elements should be highlighted;
2. Navigation order must be logical;
3. Extra elements should be skipped.

All of these aspects are regulated by different sections of Web Content Accessibility Guidelines (WCAG), the foremost of which is the [2.1.1 Keyboard](https://www.w3.org/TR/WCAG22/#keyboard).

### Focused elements should be highlighted

This means that interactive elements (buttons, links, inputs) should have a visible and obvious focus style. Unfortunately, many websites remove the focus style by the CSS rule `{outline: none;}` — that is very sad, and it is often done out of ignorance — [that’s not how it’s done](https://www.outlinenone.com/).

![Default focus outline highlighting](/assets/2024-03-25/01-focus.png)

_Fig. 1. Default focus outline highlighting_

Read more:

- [2.4.7 Focus Visible](https://www.w3.org/TR/WCAG22/#focus-visible) WCAG requirement;
- [Introduction to Focus](https://web.dev/articles/focus);
- [Never remove CSS outlines](https://www.a11yproject.com/posts/never-remove-css-outlines/).

### Navigation order must be logical

For keyboard users, the sequence in which interactive elements receive focus matters. It should follow a logical and intuitive order — **typically left to right, top to bottom.** Predictable navigation usually starts with the header, moves to the main navigation, any page content, and finally, the footer.

Focus should flow between elements as they are positioned on the page, not jump back and forth. The most common way of broken navigation is to have elements with `tabindex` attribute of 1 or greater because [`tabindex` should only be -1 or 0](https://www.csun.edu/universal-design-center/web-accessibility-criteria-tab-order).

Read more:

- [1.3.2 Meaningful Sequence](https://www.w3.org/TR/WCAG22/#meaningful-sequence) and [2.4.3 Focus Order](https://www.w3.org/TR/WCAG22/#focus-order) WCAG requirements;
- [Tab order](https://www.a11y-collective.com/glossary/tab-order/);
- [Creating a logical tab order through links, form controls, and objects](https://www.w3.org/TR/WCAG20-TECHS/H4.html).

### Extra elements should be skipped

Among expected interactive elements, extra/minor/unwanted and/or inaccessible widgets should be excluded from keyboard navigation. **Just for this case, an attribute `tabindex=-1` is required.**

Read more:

- [Use the tabindex attribute](https://www.a11yproject.com/posts/how-to-use-the-tabindex-attribute/).

The correct layout of the document structure with [landmarks](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/landmark_role) simplifies navigation for screen reader users by allowing them to navigate straight through page blocks, for example, [to skip repetitive navigation](https://www.a11yproject.com/posts/skip-nav-links/).

Read more:

- [Using ARIA landmarks to identify regions of a page](https://www.w3.org/WAI/WCAG22/Techniques/aria/ARIA11.html).

This point can also include avoiding [keyboard traps](https://www.a11y-collective.com/glossary/keyboard-trap/). This issue mainly occurs on [pop-ups and overlays](https://adequatica.medium.com/vectors-of-testing-pop-up-overlays-modal-windows-65c2ede32344#1fdc) when tabbing loops focus on one interactive component. The user should be able to leave a focused element.

Read more:

- [2.1.2 No Keyboard Trap](https://www.w3.org/TR/WCAG22/#no-keyboard-trap) WCAG requirements;
- [Keyboards Traps](https://www.csun.edu/universal-design-center/web-accessibility-criteria-keyboard-traps).

---

General reading on keyboard navigation testing:

- [Navigate using just your keyboard](https://www.a11yproject.com/posts/navigate-using-just-your-keyboard/);
- [Keyboard Accessibility](https://webaim.org/techniques/keyboard/);
- [Why Keyboard Usability Is More Important Than You Think](https://www.usertesting.com/blog/why-keyboard-usability-is-more-important-than-you-think);
- [One of my favourite accessibility testing tools: The Tab Key](https://www.matuzo.at/blog/testing-with-tab/).

## Introduction to Automation Testing on Accessible Keyboard Navigation

Unfortunately, there is no silver bullet for accessibility automation. The same applies to keyboard navigation. Each of its aspects needs its own separate approach:

- Focus style on active elements can be tested using screenshot testing techniques;
- [Lighthouse accessibility audit](https://developer.chrome.com/docs/lighthouse/accessibility/scoring) can expose[ the absence of landmark roles](https://dequeuniversity.com/rules/axe/4.8/aria-allowed-role) and[ heading semantics](https://dequeuniversity.com/rules/axe/4.8/heading-order);
- But even if you scan your site or its individual elements for WCAG compliance, **you will not automatically get the correct navigation order** because only you (as a user) know and understand the right order.

For the first time, tabbed navigation should be done only manually. Afterward, if everything is fine (or when all the bugs are fixed), this case can be automated.

Read more:

- Playwright: [Accessibility testing](https://playwright.dev/docs/accessibility-testing);
- [Automated accessibility testing](https://web.dev/learn/accessibility/test-automated);
- [Automated Accessibility Testing Is a Good Start — But You Need To Test Manually Too](https://dev.to/eevajonnapanula/automated-accessibility-testing-is-a-good-start-but-you-need-to-test-manually-too-13f2).

Next, I will focus only on [TAB] navigation.

## TAB Navigation Order Automation {#tab-navigation-order-automation}

_In my current project, we invested some development time in proper keyboard navigation, with a pretty successful result — it became possible for the user to access almost all functionality by tabbing. But after a few releases, we accidentally noticed that an unintended `<div>` receives focus. That was a sign that the Tab order must be automatically tested._

The test scenario for this case is pretty simple:

1. Open the page;
2. [Press](https://playwright.dev/docs/api/class-keyboard#keyboard-press) [TAB] key — check the focused element;
3. Press [TAB] key — check the focused element;
4. Etc.

For the implementation of this check, you need:

1. **Playwright** or any frontend test automation framework;
2. Playwright’s [`evaluate()`](https://playwright.dev/docs/api/class-page#page-evaluate) method [to invoke a custom function](https://adequatica.medium.com/simple-examples-of-using-playwright-evaluate-method-9b00d01cadc1#1d0e);
3. Document’s [activeElement property](https://developer.mozilla.org/en-US/docs/Web/API/Document/activeElement) to get the current element on the page that has focus.

To see how the activeElement property works, open DevTools Console and write the command: `document.activeElement` (read more about [detecting focused elements through the browser’s console](https://adequatica.medium.com/browser-console-for-functional-testing-650004cc4641#cc4e)).

For further examples, I randomly selected the [CERN](https://home.cern/) website. But I suddenly faced with a completely unoptimized Tab order — the user could not get on the main navigation list! Well, finding bugs is not the topic of this article. Therefore, I will have to limit the example to just a toolbar with a logo.

![Default focus outline highlighting](/assets/2024-03-25/02-document-active-element.gif)

_Fig. 2. Toolbar’s Tab order on CERN’s website_

The first [TAB] press on CERN’s website receives a special element, «Skip to main content». This is a11y hack — [an invisible link for skipping navigation](https://webaim.org/techniques/skipnav/).

The only problem with `evaluate()` function with `document.activeElement` command is that it returns a [Node](https://developer.mozilla.org/en-US/docs/Web/API/Node):

```javascript
console.log(await page.evaluate(() => document.activeElement));

ref: <Node>
```

Thus, we need to refer to the Node’s or [Element](https://developer.mozilla.org/en-US/docs/Web/API/Element)’s interface for getting data, like [innerHTML property](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) (it depends on what you decide to check for the focused elements).

Here is an example of a single step of the first [TAB] press:

```javascript
await test.step("Press TAB key", async () => {
  await page.keyboard.press("Tab");

  const focusedOn = await page.evaluate(() => {
    const selector = document.activeElement;
    return selector ? selector.innerHTML : null;
  });

  expect(focusedOn, "Should have correct active element").toBe(
    "\n  Skip to main content\n"
  );
});
```

All subsequent steps are the same as the first except for the expected value.

To avoid declarative enumeration of numerous repetitive steps, it would be better to wrap the test step in a loop through the array of the values of expected active elements.

```javascript
import { expect, test } from '@playwright/test';

const activeElements = [
  '\n	Skip to main content\n',
  '\n            CERN\n            <span>Accelerating science</span>\n        ',
  'Sign in',
  'Directory',
  '\n              <img src="/sites/default/files/logo/cern-logo.png" alt="home">\n            ',
] as const;

test('Keyboard Navigation', async ({ page }) => {
  await test.step('Open the page', async () => {
    await page.goto('/');
  });

  for (const element of activeElements) {
    await test.step('Press TAB key', async () => {
      await page.keyboard.press('Tab');

      const focusedOn = await page.evaluate(() => {
        const selector = document.activeElement;
        return selector ? selector.innerHTML : null;
      });
      expect(focusedOn, 'Should have correct active element').toBe(element);
    });
  }
});
```

See the sample code in the [GitHub repository](https://github.com/adequatica/ui-testing/blob/main/tests/keyboard-navigation-on-tab.spec.ts), where actions on the page are moved into the [page object model](https://playwright.dev/docs/pom).

Copy @ [Medium](https://adequatica.medium.com/automated-accessibility-testing-of-keyboard-navigation-on-tab-89d30087c111)
