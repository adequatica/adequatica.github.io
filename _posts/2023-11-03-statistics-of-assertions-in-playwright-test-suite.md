---
layout: post
title: "Statistics of Assertions in Playwright Test Suite"
date: 2023-11-03 08:22:59 +0300
tags: automation playwright statistics
---

When the number of tests (test files) for my current work project passed over a hundred, it became interesting to see what kind of assertions are the most frequently used.

# Introductory notes about the testing project:

My testing project is a modern client-server application. Backend (API) is tested separately. Frontend (on React) is full of business logic and has a multimodal interface:

![UI mockup](/assets/2023-11-03/01-mockup.png)

_UI mockup; [background image source](https://www.esa.int/ESA_Multimedia/Images/2006/12/Location_of_buried_basins_detected_by_MARSIS)_

Automated tests cover more than 80% of the functionality and are the main component of regression testing.

# Playwright test suite has:

- **103 test files;**
- **955 tests;**
- **601 [`expect`](https://playwright.dev/docs/api/class-playwrightassertions#playwright-assertions-expect-generic) methods** (in 97 test files);
- There are 50 negative checks among them (assertions has `.not` before matchers) ~ 8.5% of expects;
- No assertions in page object models, but there are **185 awaits for locators;**
- No assertions in `beforeAll` or `afterAll` hooks (see [№2 in Principles of Writing Automated Tests](https://medium.com/@adequatica/principles-of-writing-automated-tests-a2b72218264c#6da4));
- All assertions are executable (no if-statements in tests, see [№7 in Principles of Writing Automated Tests](https://medium.com/@adequatica/principles-of-writing-automated-tests-a2b72218264c#8fbe));
- No [soft assertions](https://playwright.dev/docs/test-assertions#soft-assertions);
- All tests run for 14 min in 4 threads in CI.

I need some clarification on it: I use [waitFor()](https://playwright.dev/docs/api/class-locator#locator-wait-for) method in page objects for the waiting locator’s specific state which _may considered as a kind of assertion_ because the test will fail if the locator does not satisfy the condition. That is why some tests (test files) can have none of [`expect` assertions](https://playwright.dev/docs/api/class-playwrightassertions) because they consist only of page objects with `waitFor()` like this:

Page object example:

```JavaScript
export class Dungeon {

  private gemstone: Locator;

  constructor(page: Page) {
    this.gemstone = page.locator('.mineral_crystal');
  }

  async waitForGem(visibleState: boolean): Promise<void> {
    await this.gemstone.first().waitFor(state: 'visible');
  }
}
```

Test example:

```JavaScript
test('Should have a gem', async () => {
  const dungeon = new Dungeon(page);
  await dungeon.waitForGem(true);
});
```

That is why I added `waitFor()` tn the top list of assertions along with [Genetic Assertions](https://playwright.dev/docs/api/class-genericassertions).

## Assertions top list:

1. `.toBe` — 328
2. `waitFor()` — 185
3. `.toStrictEqual` — 67
4. `.toContain` — 54
5. `.toEqual` — 39
6. `.toHaveURL` — 34
7. `.toHaveAttribute` — 22
8. `.toBeChecked` — 15
9. `.toHaveClass` — 14
10. `.toBeGreaterThanOrEqual` — 10
11. `.toBeNull` — 6
12. `.toMatch` — 4
13. `.toBeTruthy` — 4
14. `.toBeGreaterThan` — 2
15. `.toBeFalsy` — 2

![Assertions top list](/assets/2023-11-03/02-chart.png)

_Assertions top list: expects + waitFor()_

Results are very expectable:

- A little over **60% — are checking data for some value;**
- About a quarter — are waiting for something (locators/text);
- The remaining 10–15% — are URLs/titles matching, uncommon checks for CSS attributes, etc.

That is very similar to previous projects I have worked on.

Read more about Platwright assertions: [Assertions](https://playwright.dev/docs/test-assertions).

Copy @ [Medium](https://adequatica.medium.com/statistics-of-assertions-in-playwright-test-suite-9e464866982d)
