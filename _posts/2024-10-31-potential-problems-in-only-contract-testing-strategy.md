---
layout: post
title: "Potential Problems in Only Contract Testing Strategy"
date: 2024-10-31 07:37:17 +0100
tags: testing
---

As a test engineer, you must know about the weaknesses of your test strategy in advance.

If you test every system’s component completely isolated, **you still need integration tests and, even better — E2E tests.** But sometimes, «business» circumstances restrain conducting comprehensive testing.

To begin with, I would like to define some terms I use in this article when describing testing activities.

- A **unit test** is a type of software test that verifies the functionality of a specific, isolated piece of code (typically a single function or method) by checking its output against expected results. _Unit tests do not require building or compiling an application._

- An **integration test** is a type of software test that verifies how different components or modules of a system work together by checking that combined components produce the expected outcomes. _Integration tests, in most cases, require building or compiling an application or its parts._

- An **end-to-end (E2E) test** is a type of software test that evaluates the entire application flow by simulating real user scenarios. _End-to-end tests require building or compiling an application or production-like environment, including frontend, backend, and database._

However, it is not apparent with integration tests when the integration should be tested with third-party components or modules. In this case, the third-party systems or their APIs are usually mocked ⇒ then it turns out that integration tests verify only a single component (the one that is not mocked). It is OK because we check the _integration,_ and it does not matter that one of the components may be artificial if the interaction between components is standardized. Moreover, this type of testing can be called **contract testing**.

- Under **mocking**, I mean all kinds of «fake» data and overriding of any API requests and responses. Differences between different types of mocks (or [test doubles](https://en.wikipedia.org/wiki/Test_double)) are perfectly described in detail in chapters about testing in «[Software Engineering at Google](https://www.oreilly.com/library/view/software-engineering-at/9781492082781/)» book by Titus Winters, Tom Manshreck, and Hyrum Wright.

Then, the quality assurance engineer builds a [test strategy](https://en.wikipedia.org/wiki/Test_strategy) based on various factors, from the application’s architecture (whether it is a monolith, microservices, or some legacy) and technical stack to the organizational structure and software development process, and from the level and experience of the tester and developers to the financial capabilities of the company.

Therefore, the insufficiency of testing for some projects can be caused by different factors (in my experience, it is resource [people, budget, time] limitations). Thus, categorical statements from some «professionals» that «_you must test only this way_» may not be suitable for many real-life cases.

![Options for interaction between subsystems with different types of testing](/assets/2024-10-31/01-eraser-io.png)

_Fig. 1. Options for interaction between subsystems with different types of testing_

That is how it went for one of my latest projects.

- The main limitation was the lack of test environments ⇒ fully end-to-end testing was unavailable;
- The second limitation was CI: runners’ instances were limited in resources ⇒ test infrastructure should be as «light» as possible;
- The last limit was time ⇒ time allowed only the most «profitable» tests to be written.

Based on these, the base of automated testing of my application became unit tests by developers and contact tests by QAs. That was pretty «ambitious» because, without comprehensive integration and E2E tests, it is quite challenging to ship the application without bugs.

**If you know the weaknesses of your test strategy in advance, you can targetly improve your autotests** to achieve a high level of confidence in testing. Here, I have highlighted some challenges that appear on a testing project with contract tests only.

## True integration between components

Integration testing is called integration because it tests _integrations_. However, when all subsystems around the tested component are mocked with contracts, the component is tested in isolation.

«Isolation» has its advantages, primarily for development due to easier testing, like granular control over the system under test and independence from the unwanted behavior of third-party components.

That is why the main flaw in only contract tests is that they do not provide proper integration testing, which is worth considering when choosing a testing strategy. Of course, that does not mean you should avoid contract testing; it just means that integration checks must be executed somehow.

> _After one release, our API started responding with empty response bodies. We discovered this only on the frontend, but the problem was in the integration between the backend and the database. The proper integration or E2E test could prevent this failure._

## Contracts do not save from mismatches between APIs

Unfortunately, a contract (in the case of contract testing) does not guarantee that the API will comply with it. You must double-check the data in your code regardless of tests.

In general, it is not a flaw of the contract. Such a problem may occur and pass any type of test. Here, it is worth considering this in advance in the application’s code.

> _After one release, our API started responding with undocumented fields even despite the generated contract. Because of this, our frontend crashed. The proper integration or E2E test may prevent this failure, but double-checking the response types from the API inside the frontend code saved us from such possible problems in the future._

## Backward compatibility

It is always tempting to test only the latest version of your API, but if your application has third-party consumers, you need to support some old version of the API (if it is versioned). A contract may not prevent bugs of backward compatibility if developers support only a single contract’s version for testing. That is the question of enhanced test infrastructure.

> _After one release, a few outdated third-party systems encountered problems due to our new API. Problems were fixed on both sides: we supported old versions of the API, and some of them supported a new version of our API._

## Relevance of the contract or up-to-date mocks

Over time, especially during active development or when interacting with third-party systems, contracts and mocks may go stale. Tests _may_ keep passing, but it is already a matter of time before the bug will appear.

Actually, this is a common technical problem of a test infrastructure with mocks. Mocks can be updated/autogenerated automatically, or developers/QAs can have established procedures for updating them. After all, that is the subject of another topic.

---

Types of tests should not be opposed to each other; they should complement each other. E2E tests should be added to the project with contract tests. Conversely, a project with only E2E tests also needs unit and integration tests.

Read more on discussed topics:

- [Contract Testing on Frontend](https://adequatica.github.io/2023/12/25/api-contract-testing-on-frontend-with-playwright.html);
- [Contract Testing vs Integration Testing](https://pactflow.io/blog/contract-testing-vs-integration-testing/);
- [Unit testing vs Integration testing](https://birdeatsbug.com/blog/unit-testing-vs-integration-testing);
- [Test Doubles — Fakes, Mocks and Stubs](https://medium.com/pragmatists/test-doubles-fakes-mocks-and-stubs-1a7491dfa3da).

Copy @ [Medium](https://adequatica.medium.com/potential-problems-in-only-contract-testing-strategy-22ceb20c6806)
