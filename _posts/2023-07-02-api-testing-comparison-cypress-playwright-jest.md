---
layout: post
title: "API Testing Comparison: Cypress vs. Playwright vs. Jest"
date: 2023-07-02 07:15:38 +0100
tags: api automation cypress playwright testing
---

The presence of HTTP request realizations makes frontend testing frameworks like Cypress and Playwright suitable tools for API testing as well.

I was very skeptical about testing API with all-in-one frontend testing tools like [Cypress](https://www.cypress.io/) and [Playwright](https://playwright.dev/), but I decided to give it a try. [Jest](https://jestjs.io/) — another popular frontend testing framework — also can be used for API testing, but it should be extended with third-party libraries like [Axios](https://axios-http.com/), [GOT](https://github.com/sindresorhus/got), [node-fetch](https://github.com/node-fetch/node-fetch), etc., to provide HTTP requests.

I have been using Jest + HTTP library in several of my projects, and it has proven to be a simple and reliable tool. It can serve as a starting point for comparison.

## Jest

- **Lightweight.** Easy to run locally and in CI/CD pipelines.
- **It is a tool for «constructor-based» test infrastructure** — you take Jest as a test runner and choose any suitable tool for HTTP requests. It is very convenient if your tests are located inside the tested application codebase and have to depend on the application’s stack: it is already using a specific HTTP library or/and the application already has unit tests on Jest.
- **Manual customization.** All «helpers» (getting testing environment settings, bindings for API calls, retries and timeouts, etc.) had to be written, cause nothing goes out of the box.
- **Default reporter** well suited for API tests output.

![Jest API tests reports](/assets/2023-07-02/01-jest-report.png)

![Jest API tests reports](/assets/2023-07-02/02-jest-report-fail.png)

_Fig. 1 & 2. Jest API tests reports: pass and fail_

Further reading:

- [Testing API with TypeScript, Jest, and Got](https://github.com/adequatica/api-testing) (GitHub repository);
- [Beyond API Testing with Jest](https://circleci.com/blog/api-testing-with-jest/);
- [Building a Scalable API Testing Framework with Jest And SuperTest](https://www.velotio.com/engineering-blog/scalable-api-testing-framework-with-jest-and-supertest).

## Cypress

- **Debugging through UI.** Cypress is still a frontend testing tool, so when you wright and debug tests you have to use its GUI for E2E testing.

But there is a very convenient feature in this: you can open DevTools Console in GUI and see the full response object of the request. With a special [plugin](https://github.com/filiphric/cypress-plugin-api), you can get the response in a Postman-like view in the main window. In Jest and Playwright, you will have to debug with `console.log()` for that.

![Cypress can check only the status, body, headers, and duration of the response](/assets/2023-07-02/03-cypress-console.png)

_Fig. 3. Take note of the object in the console: Cypress can check only the status, body, headers, and duration of the response_

You can even record videos of how your API tests run and take screenshots in case of failure, but for some skilled automation engineer it would be overhead.

![Screenshot of a failed API test](/assets/2023-07-02/04-cypress-screenshot.png)

_Fig. 4. Screenshot of a failed API test_

- **Ambiguous CLI reporter.** It looks beautiful, but contains a lot of redundant information. It has a gorgeous stack trace if the test fails at the moment of the request, but it does not help much if fails one of the many checks.

![Cypress API tests reports](/assets/2023-07-02/05-cypress-report.png)

![Cypress API tests reports](/assets/2023-07-02/06-cypress-report-fail.png)

![Cypress API tests reports](/assets/2023-07-02/07-cypress-report-fail.png)

_Fig. 5–7. Cypress API tests reports: pass and fail examples_

- **Complexity of Cypress test infrastructure.** Cypress wraps your tests into its own global variable, and it is daunting to add/use something different (like a third-party library for some precondition/postcondition reasons). Adding an additional package of Cypress into an existing application may also not be painless. For example, [it conflicts with Jest](https://github.com/cypress-io/cypress/issues/22059), Karma, and Jasmine and needs an individual `tsconfig` file. The need for additional binary installation may be problematic in CI/CD pipelines.
- **Nice argument** `failOnStatusCode` in the [request method](https://docs.cypress.io/api/commands/request).
- **Extremely slow** (see «Test execution time» chapter at the end of the article). It takes time to first start Cypress app (a few seconds, even the headless one in CLI) and only then do a test run.

Further reading:

- [API & Integration Tests](https://learn.cypress.io/advanced-cypress-concepts/integration-and-api-tests) (Advanced Cypress Testing Concepts);
- [A Step-By-Step Guide to Cypress API Testing](https://www.lambdatest.com/blog/cypress-api-testing/);
- [Cypress API Testing: A Comprehensive Guide](https://www.browserstack.com/guide/cypress-api-testing);
- [Cypress Best Practices for API Test Automation](https://qaautomationlabs.com/cypress-best-practices-for-api-test-automation/).

## Playwright

- **Lightweight.** API testing in Playwright does not require the installation of browsers, you can use it only as a runner (with a built-in HTTP library).
- **All Playwright features are applied to API tests!** [Retries](https://playwright.dev/docs/test-retries), [parallelism](https://playwright.dev/docs/test-parallel), [flexible timeouts](https://playwright.dev/docs/test-timeouts), [global setup](https://playwright.dev/docs/test-global-setup-teardown), precise [options](https://playwright.dev/docs/test-use-options) for each test, etc. — a magnificent set of features, especially if you are already used to spartan Jest test runner.
- [Setting the project through Playwright config](https://playwright.dev/docs/api-testing#configuration) allows making requests short and clear: base URL and extra HTTP headers will be hidden from requests in test files.
- **Implicit behavior:** Playwright automatically adds `storageState` into requests inside the [context](https://playwright.dev/docs/api/class-apirequest#api-request-new-context) — sometimes it can be useful, but other times it causes a headache.
- **Nice option** `failOnStatusCode` in each [request method](https://playwright.dev/docs/api/class-apirequestcontext#methods).
- **Embarrassing output on the default CLI reporter.** Anyway, the HTML report at the end of the run can give all the clues.

![Playwright API tests reports](/assets/2023-07-02/08-playwright-report.png)

![Playwright API tests reports](/assets/2023-07-02/09-playwright-report-fail.png)

![Playwright API tests reports](/assets/2023-07-02/10-playwright-report-html.png)

_Fig. 8–10. Playwright API tests reports: pass and fail_

Further reading:

- [API testing](https://playwright.dev/docs/api-testing) (Playwright Docs);
- [The Definitive Guide to API Test Automation With Playwright](https://playwrightsolutions.com/is-it-possible-to-do-api-testing-with-playwright-the-definitive/);
- [Playwright API Testing Tutorial: A Complete Guide with Examples](https://www.lambdatest.com/learning-hub/playwright-api-testing);
- [Using Playwright for API testing](https://reflect.run/articles/using-playwright-for-api-testing/).

## Test Execution Time

To compare frameworks by these criteria, [I implemented](https://github.com/adequatica/api-testing-comparison) a bunch of the same API tests for each of them (I understand that it may be too simple, but it gives a minimal representation of a tool):

- 4 GET requests of [OpenWeatherMap API](https://openweathermap.org/api);
- Each test has checks for the status code, one of the headers, and body values;
- I ran each test suite for each framework 10 times and calculated the average value of test suite execution.

Results of the test suite:

- Cypress:run command time: 12,96 sec
- Playwright command time: 3,21 sec
- Jest command time: 3,84 sec

Jest and Playwright have approximately the same speed (although Playwright is insignificantly faster), but **Cypress is 4 times slower!** Unfortunately, it is unacceptable for API testing, where tests’ execution time is crucial.

Further reading:

- [API Testing Comparison: Cypress vs. Playwright vs. Jest](https://github.com/adequatica/api-testing-comparison) (GitHub repository);
- [Cypress vs Selenium vs Playwright vs Puppeteer speed comparison](https://www.checklyhq.com/blog/cypress-vs-selenium-vs-playwright-vs-puppeteer-speed-comparison/);
- [Cypress vs Playwright: Let the Code Speak Recap](https://applitools.com/blog/cypress-vs-playwright/);
- [Playwright vs Cypress: A Comparison](https://www.browserstack.com/guide/playwright-vs-cypress).

---

As I mentioned earlier, I was very skeptical about Playwright and Cypress as API testing tools before a try, which is why I was very surprised by the convenience of both of them. If I need to choose a tool for testing API and UI for a new web project, I will definitely take Playwright.

Copy @ [Medium](https://javascript.plainenglish.io/api-testing-comparison-cypress-vs-playwright-vs-jest-2ff1f80c5a7b)
