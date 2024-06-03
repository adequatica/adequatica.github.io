---
layout: post
title: "Hidden Gems of Playwright: Part 2"
date: 2024-05-23 05:33:13 +0100
tags: playwright
---

[In the previous article](https://adequatica.github.io/2022/09/07/hidden-gems-of-playwright.html), I highlighted some notable Playwright methods that made testing easier when I started using it as a default test automation framework in production.

![Hidden Gems of Playwright: Part 2](/assets/2024-05-23/00-cover-2.jpg)

Here I continued to pick up some interesting features of that tool:

- Rewrite global config for any test
  - …devices[]
- setOffline
- expect.toPass
- waitForSelector / waitFor
- CI reporter for GitHub Actions
- Boxed steps

Some of them are «illustrated» in [this GitHub repository](https://github.com/adequatica/ui-testing).

## Rewrite global config for any test

Docs: [Test use options](https://playwright.dev/docs/test-use-options), [TestOptions](https://playwright.dev/docs/api/class-testoptions/)

Playwright provides a [config file to manage all options](https://playwright.dev/docs/test-configuration) for running your tests. However, some tests may require a completely different setup for the base URL, browser settings, or a special user’s environment (viewport, geolocation, etc.).

To rewrite the global config, you can set the necessary parameters for a particular single test via the [`test.use()` method](https://playwright.dev/docs/api/class-test#test-use) at the top of a test:

```JavaScript
test.use({
  baseURL: 'http://localhost:3000',
  ...devices['Pixel 7'],
});

test('Home page on mobile', async ({ page }) => {
  await test.step('Open the page', async () => {
    await page.goto('/');
  });
});
```

### …devices[]

[Doc](https://playwright.dev/docs/emulation#devices)

Another advantage of Playwright’s configuration options is its device «emulation». Instead of setting a custom User Agent, the viewport size, and other settings for mobile browsers, you can directly specify the required device in the config (or rewrite the config through `test.use()` as shown above):

```JavaScript
use: {
  ...devices['iPhone 14'],
},
```

The whole list of devices can be found in [Plyawright’s repository](https://github.com/microsoft/playwright/blob/main/packages/playwright-core/src/server/deviceDescriptorsSource.json).

## setOffline

[Doc](https://playwright.dev/docs/api/class-browsercontext#browser-context-set-offline)

I used it in one test, especially to check the application’s behavior in case of a loss of connection. Offline mode turns on through [BrowserContext](https://playwright.dev/docs/api/class-browsercontext):

```JavaScript
test('Go offline', async ({ browser, page }) => {
  await test.step('Open the page', async () => {
    const context = await browser.newContext();
    page = await context.newPage();
    await page.goto('/');
    await context.setOffline(true);
  });
```

**Watch out, this is not fully offline mode.** Network activity will stop (as an emulation of a network being offline), but you will not be able to test features of your application that use [online/offline events](https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event) in the [`addEventListener()` method](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener):

```JavaScript
// If your application’s code has this:
window.addEventListener('offline', (event) => {});
// Then browserContext.setOffline(true) won’t work
```

## expect.toPass

[Doc](https://playwright.dev/docs/test-assertions#expecttopass)

That trickiest method allows «retry» the [assertion](https://playwright.dev/docs/test-assertions) inside `expect`:

```JavaScript
await expect(async () => {
  // Retry by intervals until the request is successful
  const response = await page.request.get('https://sso-motd-api.web.cern.ch/api/motd/');
  expect(response.status()).toBe(200);
}).toPass({
  // Probe, wait 1s, probe, wait 2s, probe, wait 10s, probe, wait 10s, probe
  intervals: [1000, 2000, 10000],
  // toPass timeout does not respect custom expect timeout
  timeout: 60000,
});
```

This is extremely useful for checking unreliable backend responses.

_There is also a similar, but not quite, [`expect.poll` method](https://playwright.dev/docs/test-assertions#expectpoll), which implements the idea of [HTTP polling](https://medium.com/cache-me-out/http-polling-and-long-polling-bd3f662a14f#0f5c) inside assertions._

## waitForSelector / waitFor

[Doc](https://playwright.dev/docs/api/class-elementhandle#element-handle-wait-for-selector)

This is another brilliant method suitable for checking selectors.

There are recommendations that **assertions should not be placed inside [page object models](https://playwright.dev/docs/pom)**, even despite the implementation example in Playwright itself.

![Please, do not do that inside pageObjects](/assets/2024-05-23/04-please-do-not-do-that.png)

_Fig. 1. Please, do not do that inside pageObjects_

Instead, you can wait for the required selector without explicit assert/expect:

```javascript
// Page’s toolbar object
export class Toolbar {
  private page: Page;
  private toggleLocator: Locator;

  constructor(page: Page) {
    this.page = page;
    this.toggleLocator = page.locator('[class*=toggle]');
  }

  async clickOnToggle(): Promise<void> {
    await this.toggleLocator.click();
    // Deprecated, use locator-based locator.waitFor() instead
    await this.page.waitForSelector('[data-testid="dropdown-menu"]');
  }
}
```

Unfortunately, **this method is deprecated,** and [`waitFor()`](https://playwright.dev/docs/api/class-locator#locator-wait-for) must be used. So the pageObject’s code above should be rewritten as follows:

```javascript
// Page’s toolbar object
export class CernToolbar {
  private page: Page;
  private toggleLocator: Locator;
  private dropdownMenu: Locator;

  constructor(page: Page) {
    this.page = page;
    this.toggleLocator = page.locator('[class*=toggle]');
    this.dropdownMenu = page.getByTestId('dropdown-menu');
  }

  async clickOnToggle(): Promise<void> {
    await this.toggleLocator.click();
    await this.dropdownMenu.waitFor({state: 'visible'});
  }
}
```

Read more:

- [Principles of Writing Automated Tests](https://adequatica.github.io/2022/09/20/principles-of-writing-automated-tests.html).

## CI reporter for GitHub Actions

[Doc](https://playwright.dev/docs/test-reporters#github-actions-annotations)

If you are using [GitHub Actions](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration) for your CI/CD, then `github` [reporter](https://playwright.dev/docs/test-reporters) is your «must have» config option:

```JavaScript
// 'github' for GitHub Actions CI, and 'list' when running locally
reporter: process.env.CI ? 'github' : 'list',
```

Documentation tells that this reporter has _annotations_ without describing what it is. These annotations look like very useful widgets inside PR’s diff in the place of a failed code line.

![github reporter annotations in PR](/assets/2024-05-23/01-reporter-github-actions-annotations.png)

_Fig. 2. github reporter annotations in PR_

`github` reporter’s report in a workflow’s job looks like a normal `list` report.

![github reporter in the job](/assets/2024-05-23/02-reporter-github-actions-jobs.png)

_Fig. 3. github reporter in the job_

## Boxed steps

When I first read the [release notes](https://playwright.dev/docs/release-notes#hide-implementation-details-box-test-steps) about a new option for the [`test.step()` method](https://playwright.dev/docs/api/class-test#test-step), I was confused and considered it useless. But later, I realized that it could be useful for steps with helpers.

If you develop test automation for fairly complex applications, sooner or later, you will have to add abstraction layers inside tests to perform repetitive and/or compound actions. These pieces of code are usually called helpers or utils and are imported into tests for execution as their steps.

Sometimes, developers can be frustrated by redundant test reports and want to see only the upper-level failures. The «box» step serves exactly this scenario — it hides error details of the test’s inner helper functions:

```JavaScript
// Helper
async function openHomePage(page: Page) {
  await page.goto('/');
  await expect(page, 'Should open / page').toHaveURL(/.*\//);
  await expect(page.getByRole('main')).toBeVisible();
}

test('Home page toolbar about overlay on mobile',
  async ({ page }) => {
    await test.step('Open the page', async () => openHomePage(page), {
      box: true,
    });
```

![HTML report: on the left — {box: true}, on the right is an ordinary test step](/assets/2024-05-23/03-test-step-box.png)

_Fig. 4. HTML report: on the left — {box: true}, on the right is an ordinary test step_

Anyway, that is quite a controversial feature, and **it depends a lot on the helper functions and the assertions inside them (your errors may look completely different than in the example above),** as well as the test requirements.

Read more:

- [Box Test Steps in Playwright](https://dev.to/playwright/box-test-steps-in-playwright-15d9).

---

Take a look at [part 1](https://adequatica.github.io/2022/09/07/hidden-gems-of-playwright.html).

Copy @ [Medium](https://adequatica.medium.com/hidden-gems-of-playwright-part-2-ca3e38a5954a)
