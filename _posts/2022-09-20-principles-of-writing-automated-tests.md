---
layout: post
title: "Principles of Writing Automated Tests"
date: 2022-09-20 17:29:25 +0300
tags: automation testing
---

While working on test automation in different projects, I’ve learned that there are not enough static analyzers and code formatters for writing good tests. The team had to have an agreement on how the tests should be written.

![Principles of Writing Automated Tests](/assets/2022-09-20/00-cover.jpg)

If any team member writes tests with his own ideal vision, it will be a mess. Some patterns applicable to unit tests on Java or Python do not fit integration tests on JavaScript and vice versa. Tests of a certain type should be consistent and should conform to acceptable rules for the project.

A presented compilation of principles is based on years of experience and has proven their effectiveness in real projects. They are mostly applicable for end-to-end tests (API and UI) on JavaScript/TypeScript and corresponding test frameworks: [mocha](https://mochajs.org/), [Jest](https://jestjs.io/), [WebdriverIO](https://webdriver.io/), [Playwright](https://playwright.dev/), and the like. Some principles may overlap or even conflict with each other, some may be controversial — common sense will judge it, at least it all depends on the context of the testing project.

1. No tests without assertions;
2. No assertions in before or after hooks;
3. No actions without expectations;
4. No unconditional expectation;
5. No commented test;
6. No hanging locators;
7. No IF statements inside tests;
8. No assertions inside page object models;
9. One expect for each test step;
10. Do not put await inside expect;
11. Do not reload the page, reopen it;
12. Do not check URLs through includes;
13. Avoid regexp in checks;
14. Wrap clicks and expectations into a promise;
15. Do not use global variables for page object methods;
16. Do not dispel checks over multiple test scenarios;
17. Do not mix different kinds of tests;
18. Test IDs must be unique;
19. Use linters and formatters from the testing (parent) project.

> _Disclaimer: some statements may seem controversial or not be provided with references, but they reflect only the author’s experience as a test automation engineer._

---

## 1. No tests without assertions

There should not be test steps without checks.

Do not make test steps like this:

```JavaScript
test("Should open menu", async () => {
  await page.locator('.button').click();
});
```

Each test step has to have an assertion:

```JavaScript
test("Should open menu", async () => {
  await page.locator('.button').click();
  const locator = await page.locator('.dropdown-menu');
  await expect(locator).toBeVisible();
});

```

## 2. No assertions in before or after hooks

`beforeAll, beforeEach, afterAll, afterEach` hooks should not have assertions. Preconditions/postconditions should contain only pure actions (for example, [authorization](https://playwright.dev/docs/test-auth#sign-in-with-beforeeach)). Checks should be done inside tests.

If you still need to check something in preconditions/postconditions, then use [try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch) and/or [throw](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) errors.

## 3. No actions without expectations

After all actions in tests, such as clicks, hovers, gotos, etc., there should be an assertion with an expectation to check that the action was definitely committed.

```JavaScript
test("Should open menu", async () => {
  await page.locator('.button').click();
  await page.locator('.dropdown-menu').waitFor({ state: 'visible' });
});
```

Or:

```JavaScript
test("Should open menu", async () => {
  await page.locator('.button').click();
  const locator = await page.locator('.dropdown-menu');
  await expect(locator).toBeVisible();
});
```

The second example is valid because [expect(locator).toBeVisible()](https://playwright.dev/docs/test-assertions#locator-assertions-to-be-visible) contains conditional expectation (see №4).

## 4. No unconditional expectation

Do not add [pauses](https://webdriver.io/docs/api/browser/pause/), `sleep`s, and [timeouts](https://playwright.dev/docs/api/class-page#page-wait-for-timeout) for N seconds between action and assertion to prevent flakiness — it only slows down the tests and does not make them reliable and efficient.

Instead of unconditional expectation:

```JavaScript
it('Should open menu', async () => {
  const button = await $('.button');
  await button.click();
  await browser.pause(3000);
  const menu = await $('.dropdown-menu');
  await menu.isDisplayedInViewport();
});
```

Use [wait for something](https://webdriver.io/docs/api/element/waitForExist) (for some element’s state):

```JavaScript
it('Should open menu', async () => {
  const button = await $('.button');
  await button.click();
  const menu = await $('.dropdown-menu');
  await menu.waitForExist({timeout: 3000});
  await menu.isDisplayedInViewport();
});
```

The second example works faster in case of passing the test and will fail for obvious and unambiguous reasons.

## 5. No commented test

If the test should be turned off, it should be skipped by the test framework feature ([skip](https://playwright.dev/docs/test-annotations#skip-a-test)), not by commented code.

Instead of:

```JavaScript
// test("Should have a menu", async () => {
//  const locator = await page.locator('.dropdown-menu');
//  await expect(locator).toBeVisible();
// });
```

Do:

```JavaScript
test.skip("Should have a menu", async () => {
  const locator = await page.locator('.dropdown-menu');
  await expect(locator).toBeVisible();
});
```

The number of skipped tests will be presented in the test report.

If the test is outdated and/or not needed, it should be deleted without regret.

## 6. No hanging locators

Tests should not contain lines of code with «meaningless» locators.

In this code snippet [page.locator()](https://playwright.dev/docs/api/class-page#page-locator) do nothing:

```JavaScript
test("Should do something", async () => {
  await page.locator('.button');
  …
```

The code in the tests has to do something: perform actions and/or assertions.

## 7. No IF statements inside tests

Avoid conditionals in tests — this will lead to missed errors. Performing different checks depending on the application’s behavior makes tests complicated and weak.

In the example below, there are checks of different things dependent on pop-up conditions:

```JavaScript
test("Should have a pop-up", async () => {
  const button = await $('.button');
  await button.click();

  const popUpisVisible = await $('.pop-up').isVisible();

  if (popUpisVisible) {
	const checkedLocator = await $('.one-locator').isChecked();
	await expect(checkedLocator).toBeTruthy();
  } else {
	const checkedAnother = await $('.another-locator').isChecked();
	await expect(checkedAnother).toBeTruthy();
  }
});
```

Don’t do that, because if such kind of test fails, you will have to investigate what conditional statement was triggered and, moreover, you will miss a bug if the functionality of not invoked check in the conditional statement failed.

If your application has A/B experiments and the interface can change randomly, then:

- Prepare the UI state of your application in precondition;
- White separate tests for each UI state.

This rule also applies to functions in page object models because they are part of the test infrastructure. You can use[ switch statements](https://en.wikipedia.org/wiki/Switch_statement) (as more deterministic and predictable) there instead of[ if-statements](https://en.wikipedia.org/wiki/Conditional_%28computer_programming%29), but don’t use any statements in the tests themselves at all.

## 8. No assertions inside page object models

A Page Object Model is a common design pattern in test automation that enhances test maintenance and reduces code duplication. A page object encapsulates common operations on the testing page and/or stores all web elements.

The logic is simple: you do interactions on the page through page objects and then do checks inside the test through `expect`.

Therefore, do not mess up interactions in page objects with checks ([assertions](https://playwright.dev/docs/test-assertions)) in the tests themselves. Even the [Playwright’s documentation](https://playwright.dev/docs/pom) is spoiled by this mixing. Please just do not use `exect` in page objects.

## 9. One expect for each test step

Test steps should be short, and each step should check only one thing.

Do not put more than one or two assertions inside one test step.

Do not try to do everything and/or check everything in a single step.

The more «atomic» test steps, the more intelligible test reports and test logs will be.

## 10. Do not put await inside expect

One operation inside another operation leads to a complication.

Instead of:

```JavaScript
test("Should have title on the button", async () => {
  expect(await page.locator('.button')).toHaveText(/Menu/);
});

// OR:
test("Should have title on the button", async () => {
  await expect(page.locator('.button')).toHaveText(/Menu/);
});

```

Do:

```JavaScript
test("Should have title on the button", async () => {
  const button = await page.locator('.button');
  await expect(button).toHaveText(/Menu/);
});
```

It is more verbose, but less chance to forget about await.

## 11. Do not reload the page, reopen it

Refreshing a page by a standard command ([page.reload()](https://playwright.dev/docs/api/class-page#page-reload) for Playwright or [browser.refresh()](https://webdriver.io/docs/api/webdriver/#refresh) for WebdriverIO) is not a good idea — it makes the test flaky.

Instead of:

```JavaScript
test("Should have something after reload", async () => {
  await page.reload();
  …
});
```

Get the current page URL and just open it:

```JavaScript
test("Should have something after reload", async () => {
  const uri = await page.url();
  await page.goto(uri);
  …
});
```

This makes tests robust.

This pattern also applies to [goBack()](https://playwright.dev/docs/api/class-page#page-go-back) and [goForward()](https://playwright.dev/docs/api/class-page#page-go-forward) methods, but unfortunately, does not fit for [SPA web applications](https://developer.mozilla.org/en-US/docs/Glossary/SPA) in which the state of the page can differ from the URL.

## 12. Do not check URLs through includes

Do not use [string.prototype.includes()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/includes) for string comparison in assertions, because includes() returns **true** or **false**. When your check fails, you will get a report that **false is not true** — and no more details.

Instead of:

```JavaScript
test("Should have corresponding URL", async () => {
  const uri = await page.url();
  await expect(uri.includes('example')).toBeTruthy();
});
```

Use the [appropriate method](https://playwright.dev/docs/test-assertions#page-assertions-to-have-url):

```JavaScript
test("Should have corresponding URL", async () => {
  const uri = await page.url();
  await expect(uri).toHaveURL(/example/);
});
```

Or [builtin assertions](https://jestjs.io/docs/expect#expectstringcontainingstring) in case of an unusual checks:

```JavaScript
test("Should have corresponding URL", async () => {
  const uri = await page.url();
  await expect(uri).toEqual(expect.stringContaining('example'));
});
```

This pattern applies for checking any strings and affects the readability and clarity of test reports.

## 13. Avoid regexp in checks

Checks with [regular expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions) make tests too sensitive and do not add much reliability to the tests, but make it difficult to analyze after failures.

There are two exceptions:

- regexp for checking URLs;
- regexp for date and time.

Both types of this kind of data are suitable for checking by regexp.

If your testing project includes IDs of a specific domain that can be attributed to some pattern, then testing them by regexp is also OK.

## 14. Wrap clicks and expectations into a promise

Instead of:

```JavaScript
await page.locator('.button').click();

const response = await page.waitForResponse('https://example.com/');

await expect(response.ok()).toBe(true);
```

Do:

```JavaScript
const [response] = await Promise.all([
  page.waitForResponse('https://example.com/'),
  page.locator('.button').click(),
]);

await expect(response.ok()).toBe(true);
```

[Promise.all prevents a race condition between clicking and waiting](https://playwright.dev/docs/api/class-page#page-wait-for-response) for something. The first example is likely to be extremely flaky.

## 15. Do not use global variables for page object methods

Isolate tests/steps from each other. Do not use global variables which are used and rewritten by multiple test steps in a single test suite.

Instead of:

```JavaScript
const myPageObject = new MyPageObject(page);

test('Should do something', async () => {
  await myPageObject.doSomething();
  …
});

test('Should have something', async () => {
  await myPageObject.haveSomething();
  …
});
```

Do:

```JavaScript
test('Should do something', async () => {
  const myPageObject = new MyPageObject(page);
  await myPageObject.doSomething();
  …
});

test('Should have something', async () => {
  const myPageObject = new MyPageObject(page);
  await myPageObject.haveSomething();
  …
});
```

If variables are not rewritten, it reduces the probability of rewriting them incorrectly or asynchronously — it increases the overall stability of the tests.

## 16. Do not dispel checks over multiple test scenarios

The same functionality should be checked the same way everywhere.

For example, instead of having `test-1.spec.ts` test where you check a banner through «expect A», and having `test-2.spec.ts` test where you check the same banner (but perhaps on the other page) through «expect B», — you should test that banner by both expects (A and B) in each of the tests (`test-1.spec.ts` and `test-2.spec.ts`).

Instead of:

```JavaScript
test-1.spec.ts

test.describe('Banner A', async () => {
  test('Banner should have title', async () => {
    …
  });
});

test-2.spec.ts

test.describe('Banner B', async () => {
  test('Banner should have image', async () => {
    …
  });
});
```

Do:

```JavaScript
test-1.spec.ts

// The same functionality is checked in the same way
test.describe('Banner A', async () => {
  test('Banner should have title', async () => {
    …
  });

  test('Banner should have image', async () => {
    …
  });
});

test-2.spec.ts

// The same functionality is checked in the same way
test.describe('Banner B', async () => {
  test('Banner should have title', async () => {
    …
  });

  test('Banner should have image', async () => {
    …
  });
});
```

## 17. Do not mix different kinds of tests

If you want to check API and UI for a single user action — do two tests: API test and UI test.

If you want to check UI functionality and check the layout by screenshot simultaneously — do two tests: UI test and screenshot test.

If you want to check end-to-end API scenarios and check JSON schemes simultaneously — do two integration API tests each of which makes certain checks.

## 18. Test IDs must be unique

If you use test identifications, like [data-testid attributes](https://playwright.dev/docs/locators#locate-by-test-id), these IDs must be unique on the page (and preferably for the whole site). It means that only one selector with a specific ID attribute can be on a testing page.

Instead of:

```Html
<div class="block_element__modificator" role="banner" data-testid="banner-element"></div>
…
<div class="block_element__modificator view-more" data-testid="banner-element"></div>
```

Do:

```Html
<div class="block_element__modificator" role="banner" data-testid="banner-element-one"></div>
…
<div class="block_element__modificator view-more" data-testid="banner-element-more"></div>
```

## 19. Use linters and formatters from the testing (parent) project

If a directory with tests is located inside a testing project, or tests are located in a separate repository, or if tests are written by dedicated autotest engineers or by developers, tests should inherit linter and formatting rules from the testing (parent) project.

Tests will be closer to the testing code (and autotest engineers will be closer to developers) if [ESLint](https://eslint.org/) and [Prettier](https://prettier.io/) rules are the same.

Useful remark:

- Tests often contain many JSON objects, therefore, it is very necessary to have permission to use [trailing commas](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Trailing_commas) — it simplifies diff and [code review](https://antimonit.github.io/2022/01/25/trailing_commas.html).

---

Note that the principles for your own project may be dramatically different.

Read more:

- Playwright [Best Practices](https://playwright.dev/docs/best-practices);
- Cypress [Best Practices](https://docs.cypress.io/guides/references/best-practices);
- WebdriverIO [Best Practices](https://webdriver.io/docs/bestpractices/);
- [UI Testing Best Practices](https://github.com/NoriSte/ui-testing-best-practices/).

Copy @ [Medium](https://adequatica.medium.com/principles-of-writing-automated-tests-a2b72218264c)
