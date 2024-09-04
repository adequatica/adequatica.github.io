---
layout: post
title: "Many Ways to Speed up Autotests"
date: 2022-10-19 05:23:18 +0100
tags: automation testing
---

As a test engineer who has to maintain a bunch of autotests, you will definitely face the task of speeding up the execution of your test suite.

![Many Ways to Speed up Autotests](/assets/2022-10-19/00-cover.jpg)

When your autotests grow in number, they are running slower and slower. Managers and developers do not usually like to wait long to find out the results of regression testing. To shorten the feedback loop, your test suite should run as fast as possible (within reason).

I started working on JavaScript testing automation back in those days when Cypress and Playwright did not exist, and Mocha and WebdriverIO did not support parallelisation yet — instead of them, there were [mocha-parallel-tests](https://github.com/mocha-parallel/mocha-parallel-tests) and [Hermione](https://github.com/gemini-testing/hermione) as forks of the above-mentioned frameworks, but with its own implementation of parallelism.

Based on the experience since that time, I have put together several ways in which the test automation team can achieve the speed:

1. Parallelisation;
2. Less retries;
3. No unconditional expectations;
4. Fail fast;
5. Cancel subsequent steps when the first one fails;
6. No knowingly failed tests;
7. Filter tests;
8. Bundle tests;
9. Keep Node and test framework up to date;
10. Less dependencies and installations;
11. Preconditions via API, not via UI;
12. Authorize from cache;
13. Hardcode precondition data.

---

The covered ways are mostly fit for [JavaScript/TypeScript test frameworks](https://adequatica.github.io/2022/09/23/selection-criteria-of-javascript-test-framework.html), but also can take into account another stack.

## 1. Parallelisation

**It is the most essential way.** All other ways are just neat tuning that might not dramatically speed up the execution time as parallelisation does.

If your tests are running simultaneously in parallel with each, **the execution time of the entire test suite will correspond to the slowest test.**

The ability to parallelize tests depends _mostly_ on a test runner:

- Some test runners still do not support parallelisation;
- Some mean by parallelism only tests inside test files — when all tests inside a file are run in parallel;
- Others can run in parallel test files — test files are run in parallel, but tests in a single file are run in order — the most convenient way for UI automation.

The number of parallel threads is usually configured by the number «workers» or «jobs». Each test is put into a queue and executed as workers become available.

When you run tests in parallel, you should keep in mind the performance of your local machine or cloud instance. If you have more than 1000 tests and you will run them all in parallel at the same time, you may have a performance issue — you should find a balance between execution time and number of workers.

Read further about parallelisation in popular frameworks:

- Cypress: [parallelization](https://docs.cypress.io/guides/guides/parallelization);
- Jest: [JavaScript Unit Testing Performance — Optimal Scheduling of a Test Run](https://jestjs.io/blog/2016/03/11/javascript-unit-testing-performance#optimal-scheduling-of-a-test-run);
- Mocha: [parallel tests](https://mochajs.org/#parallel-tests);
- Playwright: [parallelism and sharding](https://playwright.dev/docs/test-parallel);
- WedbriverIO: [maxInstances](https://webdriver.io/docs/options/#maxinstances).

Unfortunately, one does not simply run tests in parallel if they are not isolated from each other. **Parallelisation and test isolation go together**, because while one test goes, it should not affect the other.

Therefore, when you start test automation, you should implement isolation from the very beginning to prepare for the parallel launch of your tests in advance (but «how to write isolated tests» is not the topic of this article).

Further reading:

- [Speeding Up Your Tests: Parallel Testing with Short Tests](https://saucelabs.com/blog/speeding-up-your-tests-short-tests-in-parallel).

## 2. Less retries

Retries are good to avoid flakiness, but **retrying more than one time may not be a good idea.**

If the test really fails, then it will fail every next retry, and it will take time to run before repeated failing, and it will hold a worker for another test  —  it slows down an entire execution.

Retries mainly help with accidental network flaps or animation lags, but not with beliefs that the test will pass from the N-th try. If your tests require retries, that is a flag to rewrite them or investigate the problems in the test environment.

Read further about retries in popular frameworks:

- Cypress: [test retries](https://docs.cypress.io/guides/guides/test-retries);
- Mocha: [retry tests](https://mochajs.org/#retry-tests);
- Playwright: [test retry](https://playwright.dev/docs/test-retries);
- WedbriverIO: [retry flaky tests](https://webdriver.io/docs/retry/).

## 3. No unconditional expectations

Get rid of `pause` and `timeout` functions — they just slow down tests for a specified time.

Use conditional expectation (wait for something) instead, because **the condition often happens earlier than its timeout.**

Further reading:

- [Principles of Writing Automated Tests](https://adequatica.github.io/2022/09/20/principles-of-writing-automated-tests.html).

## 4. Fail fast

If you see multiple simultaneous failures in different tests during a test run, that is a sign of problems with the test environment, there is no reason to execute more tests.

Find a way to stop test execution when a certain threshold of failed tests is exceeded — **it will not speed up tests by themselves, but it will speed up a feedback loop (which is more significant).**

Further reading:

- [Hidden Gems of Playwright](https://adequatica.github.io/2022/09/07/hidden-gems-of-playwright.html#maxfailures).

## 5. Cancel subsequent steps when the first one fails

This way is similar to «4. Fail fast», but refers to an individual test (test file).

If you have a test with multiple steps, make a setting to fail all subsequent steps just after the first failed step. There is no point in waiting for all steps if one has already fallen — **fast failing will free up a worker for the next test and speed up a feedback loop.**

The opponents of this way argue that «you won’t know which steps have passed further», but really it does not matter: a step in your test failed ⇒ you will have to investigate the problem ⇒ you will have to rerun the whole test ⇒ in the end, you will check all steps in the test anyway.

Read further about retries in popular frameworks:

- Cypress: [Cancel test run when a test fails](https://docs.cypress.io/guides/dashboard/smart-orchestration#Cancel-test-run-when-a-test-fails);
- In Playwright, by default, a failed assertion will terminate test execution, but in opposite to this, you can use [soft assertions](https://playwright.dev/docs/test-assertions#soft-assertions) to go against the described way;
- WebdriverIO: [Stop testing after failure](https://webdriver.io/docs/organizingsuites/#stop-testing-after-failure).

## 6. No knowingly failed tests

Are you familiar with this situation, when after a test run with a few failed tests, developers or QA say: «It’s fine, these tests always fail»?

It should not be like this. If the test fails, it must be fixed or [skipped](https://playwright.dev/docs/test-annotations#skip-a-test). An «always failed» tests do nothing but take your time and attention. **Keeping your autotest suite «green» makes it reasonably fast.**

## 7. Filter tests

Most modern test runners are allowed to filter tests by filenames or test titles/descriptions:

- Cypress (support via [plugin](https://github.com/cypress-io/cypress-grep)): `--env grep=<grep>`
- [Jest](https://jestjs.io/docs/cli#running-from-the-command-line): `-t <filename>`
- [Mocha](https://mochajs.org/#-grep-regexp-g-regexp): `--grep <grep>`
- [Playwright](https://playwright.dev/docs/test-cli#reference): `--grep <grep>`
- [WedbriverIO](https://webdriver.io/docs/frameworks/#grep): `--grep <grep>`

Thanks to this you can run only a part of your test suite for the case of running tests for a certain environment or a certain feature.

As you can imagine, **fewer tests will run faster than all of them.** But of course, this item may not be applicable for regression testing, which requires the full set of tests to run.

## 8. Bundle tests

This way is best applicable for CI/CD and combines items «1. Parallelisation» and «7. Filter tests».

You can filter tests by a few disjoint criteria and run each filtered bundle simultaneously in a separate cloud instance of your CI/CD system.

![Bundle tests](/assets/2022-10-19/01-bundle-tests.png)

In turn, tests in each bundle can also run parallel, so **you can _theoretically_ increase the speed by N times depending on the number of test bundles/instances.** Theoretically — because test runs in a cloud can also be queried for an available instance or CI/CD agent and it is really hard to divide a test suite into equal bundles with the same execution time.

In addition, there are several nuances:

- This method does not make sense for a small number of tests — it is a desperate way for mature projects;
- You will have to create a system to join test reports from different bundles;
- You will have to pay for computational resources: additional cloud instances or CI/CD agents — not everyone can afford it.

## 9. Keep Node and test framework up to date

[Every release of Node.js](https://github.com/nodejs/release) has performance improvements ⇒ the code of your tests will execute **a little bit faster.**

Every release of a test framework contains improvements and bug fixes ⇒ the code of your tests will execute **a little bit more optimized.**

Keeping Node.js and test framework up to date has a pleasant side effect: when it comes time to update one major version of Node.js or test framework to another, you will not notice it much. For teams who have not updated their packages for years, moving from one version to another can be extremely painful.

## 10. Less dependencies and installations

This item is about the infrastructure of your tests.

Try to make your test with standard tools of your framework:

- If you need to make HTTP requests — do it by build-in library ([Cypress](https://docs.cypress.io/api/commands/request) and [Playwright](https://playwright.dev/docs/test-api-testing) have it);
- If you need visual/screenshot testing — do it by build-in solutions (Cypress with [plugins](https://docs.cypress.io/plugins/directory#Visual%20Testing), [Playwright](https://playwright.dev/docs/screenshots), and [WebdriverIO](https://webdriver.io/docs/wdio-image-comparison-service/) can do it);
- If you need mocks or stubs — do it by build-in tools ([Cypress](https://docs.cypress.io/guides/guides/stubs-spies-and-clocks), [Playwright](https://playwright.dev/docs/mock), and [WebdriverIO](https://webdriver.io/docs/mocksandspies/) can do it);
- Etc…

As fewer external tools you use, as less dependencies will be downloaded at the `npm install` step in CI/CD pipeline.

If you use docker, you can add all required browsers into your docker image and thus skip an installing step (`npx playwright install --with-deps chromium`) in CI/CD pipeline.

This way has many paths, and **it will not speed up tests by themselves, but it will speed up the preparation for test launch.**

## 11. Preconditions via API, not via UI

For browser/UI testing, most of all preconditions like authorization and generation of test data should be done through the API. Opening a browser is quite an «expensive» operation compared to HTTP requests. **As much you can do through API instead of UI, the faster your test suite will be.**

## 12. Authorize from cache

If your test requires authorization, it is not necessary to authorize each time in preconditions. During a test run, you can authorize only the first time, put authorization credentials (cookies or tokens) into a cache file, and take it from there by other tests.

**Getting data from the cache is faster than making an API request.**

![Authorize from cached credentials](/assets/2022-10-19/02-authorize-from-cached-credentials.png)

The Playwright has already provided a mechanism for this «caching» by [reusing a signed in state](https://playwright.dev/docs/auth#reuse-signed-in-state) through config.

This point can be applied to any precondition test data required in tests more than once.

## 13. Hardcode precondition data

**Attention, this is a controversial way!** It depends on your project and requires full awareness of what you are doing.

If you need to generate/calculate/extract IDs or URLs as a precondition for your tests, you can skip this step and just use [hardcoded](https://en.wikipedia.org/wiki/Hard_coding) values.

**You will save a certain amount of time for getting testing data**, but your tests become less credible and this data may stale over time.

I put the last point only because it works, but **it is not recommended.**

Copy @ [Medium](https://adequatica.medium.com/many-ways-to-speed-up-autotests-c79308db8664)
