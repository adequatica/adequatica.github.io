---
layout: post
title: 'Core Takeaway on Designing Automation Test Suites from «Software Engineering at Google»'
date: 2025-03-28 13:06:19 +0100
tags: testing
---

Nowadays, there are no decent books on software testing, but interesting thoughts on testing can be found in the relevant chapters of other programming books.

![Software Engineering at Google: Lessons Learned from Programming Over Time](/assets/2025-03-28/00-cover-software-engineering-at-google.jpg)

In the relatively fresh (2020) book, «**Software Engineering at Google**» [1], there are a few chapters dedicated to software testing. I do not intend to give a full summary, but I want to share one bold idea that may be applicable to practice in any project.

In all the projects I worked on, we categorized automated tests into unit tests, integration tests, and end-to-end tests. This categorization corresponds to the generally accepted practices and is expressed in the [test pyramid](https://martinfowler.com/articles/practical-test-pyramid.html).

However, in the very first chapter about testing (Chapter 11. [Testing Overview](https://abseil.io/resources/swe-book/html/ch11.html)), it is said that **Google categorizes tests into small, medium, and large when designing test suites.**

Interesting… The following chapters on testing are dedicated to showing a different perspective on automated tests on a massive scale.

First, what exactly does a «small (medium and large)» test mean? There is no exact criterion for it. Instead of strict classification, **each test case has two dimensions: size and scope.**

## Test size

> _“Size refers to the resources that are required to run a test case: things like memory, processes, and time.”_

It means that it does not determine the number of lines of code but by how the test runs and how many resources it consumes. At Google’s scale, where tens of thousands of tests are constantly running, speed and computing resources are crucial. That is why small, medium, and large may reflect the constraints of the testing infrastructure.

![Test sizes according to Google](/assets/2025-03-28/01-test-sizes.png)

_Fig. 1. Test sizes according to Google_

### Small tests

The primary constraint on small tests is that they must run in a single process.

> _“A test that runs on a single process and never makes blocking calls can effectively run as fast as the CPU can handle. It’s difficult to accidentally make such a test slow or nondeterministic. <…> The other important constraints on small tests are that they aren’t allowed to sleep, perform I/O operations, or make any other blocking calls.”_

Such small tests make the entire test suite run fast and reliable.

### Medium tests

> _“Medium tests can span multiple processes, use threads, and can make blocking calls, including network calls, to `localhost`. The only remaining restriction is that medium tests aren’t allowed to make network calls to any system other than `localhost`. In other words, the test must be contained within a single machine.”_

Permission to run multiple processes allows tests to interact with databases, locally build applications, and combine testing of web UI through [WebDriver](https://www.w3.org/TR/webdriver2/) and server code. This is a kind of extended integration testing or limited end-to-end testing.

Unfortunately, such medium tests are slower and maybe nondeterministic (flaky).

### Large tests

> _“Large tests remove the `localhost` restriction imposed on medium tests, allowing the test and the system being tested to span across multiple machines.”_

These are tests without restrictions for full-system checks, validating configurations, and testing legacy components, running only during the release process.

## Test scope

> _“Test scope refers to how much code is being validated by a given test.”_

This dimension is already more akin to the familiar one:

- **Narrow-scoped tests** — tests designed to validate the logic in a small, focused part of the codebase, like an individual class or method = _unit tests_;
- **Medium-scoped tests** — tests designed to verify interactions between a small number of components, for example, between a server and its database = _integration tests_;
- **Large-scoped tests** — tests designed to verify the interaction of several distinct parts of the system or emergent behaviors that are not expressed in a single class or method = _end-to-end tests or system tests_.

Unit tests are an excellent base for automation testing because they are fast and stable and dramatically narrow the scope, which makes failure diagnosis quick and easy. That is why developers try to keep the unit test ratio around 80%. The remaining 15% comes from testing integrations between two or more components, and 5% validates entire systems.

![Google’s version of the test pyramid](/assets/2025-03-28/02-test-pyramid.png)

_Fig. 2. Google’s version of the test pyramid [2]_

The following chapters delve deeply into the topics of writing unit tests (Chapter 12. [Unit Testing](https://abseil.io/resources/swe-book/html/ch12.html)), working with [test doubles](https://martinfowler.com/bliki/TestDouble.html) and mitigating flaky tests (Chapter 13. [Test Doubles](https://abseil.io/resources/swe-book/html/ch13.html)), and overcoming challenges with large tests (Chapter 14. [Larger Testing](https://abseil.io/resources/swe-book/html/ch14.html)).

---

How can this approach to automation test suite design be implemented, even if your test infrastructure is not even close to Google’s scale?

Quite straightforward: base on the resources used to run the tests. Consider any tests that have access beyond `localhost`, do not have mocks, or use third-party systems — for large tests. Any tests that can run locally without the need to build an application or set up a database — for small tests (most likely, these will be unit tests). All other tests (most likely, these will be some integration API tests) — for medium ones.

For example, if you have E2E browser tests that can test your application both on `localhost` with mocks and in a real environment, then the first suite can be considered as medium tests and the second as large ones. Run large tests less frequently to conserve resources, but run small tests more often.

By the way, the categorization of tests into small, medium, and large [was also mentioned in the book](https://testing.googleblog.com/2011/03/how-google-tests-software-part-five.html) «How Google Tests Software» [3], but without technical details as in «Software Engineering at Google».

Read further:

- [Test Sizes](https://testing.googleblog.com/2010/12/test-sizes.html) by Simon Stewart;
- [Small, Medium, Large](https://mike-bland.com/2011/11/01/small-medium-large.html) by Mike Bland.

References:

1. Titus Winters, Tom Manshreck, Hyrum Wright, “[Software Engineering at Google: Lessons Learned from Programming Over Time](https://www.oreilly.com/library/view/software-engineering-at/9781492082781/),” O’Reilly, 2020;
2. The test pyramid was introduced by Mike Cohn in “[Succeeding with Agile: Software Development Using Scrum](https://www.amazon.com/Succeeding-Agile-Software-Development-Using/dp/0321579364/),” Addison-Wesley Professional, 2009;
3. James A. Whittaker, Jason Arbon, Jeff Carollo, “[How Google Tests Software](https://www.amazon.com/Google-Tests-Software-James-Whittaker/dp/0321803027),” Addison-Wesley Professional, 2012.

Read further on Google’s testing practices:

- [Chapter 17 - Testing for Reliability](https://sre.google/sre-book/testing-reliability/), “Site Reliability Engineering: How Google Runs Production Systems,” O’Reilly, 2016.

Copy @ [Medium](https://adequatica.medium.com/core-takeaway-on-designing-automation-test-suites-from-software-engineering-at-google-58c4ade31446)
