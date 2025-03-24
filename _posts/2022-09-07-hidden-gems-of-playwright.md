---
layout: post
title: 'Hidden Gems of Playwright'
date: 2022-09-07 11:35:14 +0300
tags: playwright
---

The main tools for writing UI tests are: browser, IDE, and testing framework documentation.

![Hidden Gems of Playwright](/assets/2022-09-07/00-cover.jpg)

The Playwright’s documentation is rich and intricate at the same time. Sometimes, you can make a check or perform an action one way, and after a while, find that it could be done in another way — just because you look into the [API](https://playwright.dev/docs/api/class-playwright) section instead of [Docs](https://playwright.dev/docs/intro) or vice versa.

I picked up a few features that excited me when I discovered them:

- maxFailures
- Custom Expect Messages
- test.slow
- expect.soft
- test.step
- test.fixme
- last
- Explore Selectors in Playwright Inspector

All of them are «illustrated» in [this GitHub repository](https://github.com/adequatica/ui-testing).

## maxFailures

[Docs](https://playwright.dev/docs/api/class-testconfig#test-config-max-failures)

Let’s start from a config file: after reaching the set number of maximum failures, Playwright will stop testing and exit with an error.

The maxFailures parameter is the ultimate way to speed up a feedback loop of your CI/CD in case of a broken test environment. If too many tests fail simultaneously, it often means that something is wrong with the environment, not with test cases — it is preferable to stop the testing and investigate the reason for the mass failure, than to wait for the completion of all remaining tests.

```JavaScript
const config: PlaywrightTestConfig = {
  maxFailures: 10,
};
```

Or, if you pass the CI option (as an environment variable) to determine the run in a continuous integration server ([TeamCity](https://www.jetbrains.com/teamcity/), [Jenkins](https://www.jenkins.io/), [Travis CI](https://www.travis-ci.com/), [Drone](https://www.drone.io/), etc.):

```JavaScript
const config: PlaywrightTestConfig = {
  maxFailures: process.env.CI ? 10 : 0,
};
```

![Example of env.CI](/assets/2022-09-07/01-env-ci.png)

_Fig. 1. Example of env.CI = true in a build configuration in TeamCity_

## Custom Expect Messages

[Docs](https://playwright.dev/docs/test-assertions#custom-expect-message)

Any assertion (`expect()` function) can be endowed with its own error message. This allows to describe checks more precisely, which is a great asset for the test report.

```JavaScript
await test('Should have proper page title', async () => {
  await expect(page, 'Should have "Home" in title').toHaveTitle(/Home/);
});
```

![Example of custom expect error message in HTML report](/assets/2022-09-07/02-custom-expect-messages.png)

_Fig. 2. Example of custom expect error message in HTML report_

## test.slow

[Docs](https://playwright.dev/docs/api/class-test#test-slow-1)

Declaration of a «slow» test will triple the default timeout. If your `playwright.config.ts` file has `timeout: 10000`, then a specified test will be allowed to run 3 \* 10000 = 30000 ms. It is very handy when only one test in a group of tests has a long execution.

```JavaScript
test('Test name', async ({ page }) => {
  test.slow();
});
```

## expect.soft

[Docs](https://playwright.dev/docs/test-assertions#soft-assertions)

Playwright will terminate test execution in case of a failed assertion, but soft assertions prevent this behavior. Test with soft assertions will continue to run, but in the end it will be marked as failed. This may be useful if you have minor checks inside a big test.

```JavaScript
await test.step('Should have proper page title', async () => {
  await expect.soft(page, 'Should have "Home" in title').toHaveTitle(/Home/);
});
```

![Example of HTML report with failed soft assertion](/assets/2022-09-07/03-expect-soft.png)

_Fig. 3. Example of HTML report with failed soft assertion — all test steps are passed, but the entire test was marked as failed_

I personally am against having soft assertions in tests — tests must be unambiguous and obvious (soft assertions are hiding problems instead of highlighting them), but this feature could be counted as a nice gem.

## test.step

[Docs](https://playwright.dev/docs/api/class-test#test-step)

As an old user of testing frameworks like [mocha](https://mochajs.org/) and [jest](https://jestjs.io/), I am used to dividing a big test into a bunch of small tests and [logically grouping them](https://playwright.dev/docs/api/class-test#test-describe-1) with `describe()` method inside one test file:

```JavaScript
`test.describe('Home page toolbar', async () => {`
  test('Should have proper page title', async ( => {…});
  test('Should have toolbar', async () => {…});
  test('Should have proper toolbar title', async () => {…});
});
```

Playwright has an additional way of writing tests with `step()` method:

```JavaScript
test('Home page toolbar', async ({ page }) => {
  await test.step('Should have proper page title', async () => {…});
  await test.step('Should have toolbar', async () => {…});
  await test.step('Should have proper toolbar title', async () => {…});
});
```

Both approaches have pros and cons:

**Describes:**

- Each test in `describe()` is clearly represented in [reporters](https://playwright.dev/docs/test-reporters#built-in-reporters);
- Tests inside the `describe()` [can be run as sequentially, as parallel](https://playwright.dev/docs/api/class-test#test-describe-configure).

**Steps:**

- Steps are hidden in console reporters (list, line, and dot), but can be shown in the «Test Steps» section in [HTML reporter](https://playwright.dev/docs/test-reporters#html-reporter). For some engineers, this may be acceptable — no extra data in a report, more steps and actions could be included in one test, — but for others, this can be considered as less informative reports and non-[atomic](https://medium.com/swlh/creating-fast-reliable-focused-ui-automation-with-atomic-tests-582e4318c0bb) tests.
- [Recorded videos](https://playwright.dev/docs/test-configuration#record-video) and [traces](https://playwright.dev/docs/test-configuration#record-test-trace) in case of failure are recorded within the `test()` — which means you can playback a full test with all steps. **This is highly convenient for debugging.** This debugging method (through videos and traces) is almost impossible for atomic tests inside describes, because artifacts would be extremely short and will not contain data about previous test’s «steps».

![Example of HTML report of identical tests](/assets/2022-09-07/04-describe-steps.png)

_Fig. 4. Example of HTML report of identical tests: one with describe and child tests, another with child steps_

Furthermore, you can combine describes and steps inside a single test file to gather all the power of Playwright’s test runner.

Read more:

- [Keep your Playwright tests structured with steps](https://timdeschryver.dev/blog/keep-your-playwright-tests-structured-with-steps).

## test.fixme

[Docs](https://playwright.dev/docs/api/class-test#test-fixme-2)

Marking a test with the «fixme» method will skip it during testing. It is the easiest and fastest way to skip tests (or a group of tests if it stands under `describe()`).

```JavaScript
test('Should have proper page title', async () => {
  test.fixme();
  await expect(page, 'Should have "Home" in title').toHaveTitle(/Home/);
});
```

![Example of skipped test in a list reporter](/assets/2022-09-07/05-test-fixme.png)

_Fig. 5. Example of skipped test in a list reporter_

## last

[Docs](https://playwright.dev/docs/api/class-locator#locator-last)

Last but not least, an easy way to match the last selector from several. It is an explicit «shortcut» for the `.nth(-1)` method and, for example, useful for selecting recently appeared identical elements.

```JavaScript
const lastNotification = await page.locator('.notification').last();
```

Read more:

- [Playwright Select First or Last Element](https://www.programsbuzz.com/article/playwright-select-first-or-last-element).

---

## Explore Selectors in Playwright Inspector

[Docs](https://playwright.dev/docs/debug-selectors)

Sometimes, when you try to choose the right selector for a locator, it is not easy to do it the first time, especially if the page layout is complicated. For such cases, you can test your locators in the «Explore» input in [Playwright Inspector](https://playwright.dev/docs/debug#playwright-inspector).

![Example of exploring selector](/assets/2022-09-07/06-explore-inspector.png)

_Fig. 6. Example of exploring selector_

For thoughtful debugging, you can add [`await page.pause()`](https://playwright.dev/docs/debug#pagepause) on a desired line of the code — it will stop test execution and will not close the Inspector window after a timeout.

---

More gems in [part 2](https://adequatica.github.io/2024/05/23/hidden-gems-of-playwright-part-2.html).

Copy @ [Medium](https://adequatica.medium.com/hidden-gems-of-playwright-68fcf8896bcb)
