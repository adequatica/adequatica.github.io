---
layout: post
title: "Pros and Cons of the Ways of End-to-End Automated Testing in CI"
date: 2023-12-04 09:26:13 +0100
tags: automation testing
---

Over the last couple of years of setting up testing processes in different projects, the most difficult and debatable question that had to be solved was how to run autotests: inside the isolated container or against a staging environment?

Disclaimer:

- In this article, end-to-end automated tests mean UI autotests written on [Playwright](https://playwright.dev/), [Cypress](https://www.cypress.io/), or with a sort of Selenium usage;
- Functional testing can be called manual testing;
- «Mocks» is a collective name for all [test doubles](https://medium.com/pragmatists/test-doubles-fakes-mocks-and-stubs-1a7491dfa3da) (fakes, mocks, stubs, and proxies) that can replace third-party services.

### Here are two of the most common ways/cases of automation:

![Tests run against an isolated environment with mocked dependencies](/assets/2023-12-04/01-isolated.png)

_1). Tests run against an isolated environment with mocked dependencies_

![Tests run against a live staging environment](/assets/2023-12-04/02-staging.png)

_2). Tests run against a live staging environment_

The first diagram has fewer steps between making a pull request and merging it into the main branch, but to decide if this is the best option, it is worth looking at all aspects of each of the cases.

## 1. Run tests against an isolated environment with mocked dependencies

Suitable for projects:

- Applications that can safely run locally (`localhost:port`) — that makes themes suitable to run inside isolated containers inside CI [build agents](https://www.jetbrains.com/help/teamcity/build-agent.html)/[runners](https://docs.github.com/en/actions/using-github-hosted-runners).

Pros:

- **Stable tests.** More predictable/durable tests with no dependencies on the third-party services ⇒ less flaky. **You test only your software and not anything else.**
- Always run tests for all pull requests.
- Ideal for [continuous delivery](https://en.wikipedia.org/wiki/Continuous_delivery) flow.

Concerns:

- **Building the infrastructure of mocking.** Running a mocking server, managing mocks, and keeping API contacts up to date is not as easy as it seems.
- **Limited computer resources.** The speed of tests and the number of parallel workers are limited by build agents’ hardware resources.
- Test reports must be as informative as possible. Just a single stack trace is now enough to debug isolated tests. Traces, screenshots, and videos should be presented in the test report.
- Inconvenient reproduction. To reproduce a failed test, you have to build and run the application, which takes time and effort.

## 2. Run tests against a live staging environment

Suitable for projects:

- Complicated applications with multiple databases and the need to interact with multiple third-party services. **Sometimes, building and deploying a staging environment, which has permanent access to third-party services, is easier than mocking all dependencies.**

Concerns:

- Dependence on the reliability of third-party services ⇒ the test infrastructure of the whole company should be very mature.
- **Network problems.** Unfortunately, timeouts between requests between services will always be.
- Dependence on the release cycle of third-party services. Your tests _may_ face incompatible changes or other’s downtime due to updates.
- Dependence on constantly changing data ⇒ tests should have very flexible asserts.
- **Flaky tests.** All the concerns listed above lead to flaky tests ⇒ flakiness can be reduced by [writing well-designed tests](https://adequatica.medium.com/principles-of-writing-automated-tests-a2b72218264c), but it won’t get rid of it completely. This point is usually the most criticized against running tests on staging environments, but, unfortunately, flaky tests happen on isolated ones, too.
- Complicated CI settings. Because tests’ action/build can’t start without an environment, the CI pipeline had to have triggers and dependencies between systems, often not assumed to have a connection between each other._ In all projects with such kind of autotests I worked on, this step was the most painful._
- Long-lasting CI flow. The need to deploy the environment increases the time of the whole flow, and the whole flow itself has more points of failure.

Pros:

- **Real integration testing!** More confidence for QAs, that everything works together.
- **No need to build infrastructure for mocking.** You do not need to keep it up to date mocks and update API contracts.
- **Fast and straightforward reproducibility.** When autotest fails, it is extremely convenient to rerun it immediately — the already live environment allows to do that. Any member of the Dev team can reproduce/rerun failed tests without building and running the application locally.
- **Resources utilization.** After your DevOps team built staging environments for functional testing, those environments became used for autotesting as well.
- **Scalable autotests.** When your autotests run as a separate process/action — they can be easily scaled and/or parallelized, unlike isolated tests that stick with an application inside one agent/container.
- **Run tests in the cloud.** If your staging environment can be accessed from the outer internet, then you can use such cloud tools as [Sauce Labs](https://saucelabs.com/), [BrowserStack](https://www.browserstack.com/), and [Microsoft Playwright Testing](https://azure.microsoft.com/en-us/products/playwright-testing/).
- Third-party services may want to be used as a test instance by your staging environment’s tests ⇒ it increases the robustness of the whole infrastructure. _At one of the projects I worked on, the team from another service asked to run tests of my service to test theirs because our test environments were connected._

## 3. Mixed way

At least, you can run tests both ways: inside the isolated container AND against a staging environment (and after that, even in production).

In the end, why not? if you can build test infrastructure and invest in writing universal tests (or separate tests depending on the environments via [conditional skips](https://playwright.dev/docs/test-annotations#conditionally-skip-a-test)). If the architecture of the project allows, the whole frontend end-to-end autotests may work on [network mocks](https://playwright.dev/docs/network#mock-apis) and run everywhere.

---

In my own experience, there is no final answer as to which way is the best one. _One project with a single backend and database was perfectly tested isolated, another project with multiple backends, databases, and WebSocket APIs could be tested only through a staging environment._

Developers always prefer to run isolated tests, cause it is closer to their own development environment. However, QA teams prefer to run tests against staging environments because it is more convenient for them to do that — they test the same environments manually and can write tests for new functionality right away without creating mocks or local running of the application.

Anyway, it all depends on the project. If the architecture allows to painlessly run the app locally, then isolated tests are preferable. If it is overcomplicated, then running tests against staging environments is OK.

Copy @ [Medium](https://adequatica.medium.com/pros-and-cons-of-the-ways-of-end-to-end-automated-testing-in-ci-9bec51e231cb)
