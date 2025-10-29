---
layout: post
title: "Make Enriched E2E API Tests Reports"
date: 2025-10-29 10:00:31 +0100
tags: testing
---

Sometimes, as a test engineer, you need to have as much debug information as possible.

**First of all, let’s figure out what end-to-end (E2E) API tests are.** Unlike usual API integration tests, which involve one or a few API calls to test «an integration», E2E API tests involve a cascade of API calls that can be executed in a single test to check the entire system.

In other words, these are the kinds of tests that create and check the required scenario’s flows completely through the API. For example:

- Precondition: Authorize through the API.
- Step 1: Create the first POST request by `/the-first` endpoint, then check and save the request.
- Step 2: Create a second POST request with data from the first step by `/the-second` endpoint. Check the response and save some more data.
- Step 3: Create a third GET request using the `/the-third` endpoint. Check the response containing data from step 2 and verify that the system has reached the required state.
- Postcondition: Return the system to the initial state.

That is quite complicated, but unfortunately, E2E tests have this specificity: they test the entire system in its «assembled» state and in its working environment. At the same time, despite the fact that it is an API and not a UI, tests will be subject to false positives due to the imperfection of the system being tested, network lags, concurrency, and other topics specific to flaky tests. However, flaky tests are outside the scope of this article.

In my current project, I came across such tests. These are complex E2E API tests running against an unstable environment, resulting in a high flaky rate. Moreover, debugging failures were hellish due to unintelligible assertion messages, so test reports did not indicate what had happened. Thus, QA engineers needed to dig into the source code each time rather than immediately understanding the problem by glancing at the test report.

The first step on working with such a test suite is reducing the time spent debugging failed tests and **making the test report as the only comprehensive source of information about a failure.** Depending on the complexity of the system and test scenarios being tested, this task requires QA expertise for refactoring autotest’s code.

> The E2E API test report is important not so much for determining whether the test passed or failed, but rather for understanding exactly which step the test failed at and why. In E2E tests, all checks on all steps matter.

All subsequent sections of the article will relate to the readability of the test report.

1. Do not treat end-to-end tests as unit tests
2. Stick to BDD assertion style
3. Output expected and actual data
4. Try to avoid Boolean checks
5. Pay attention to errors in Golang
6. Log API requests
7. Log full API response
8. Output unique IDs as separate parameters
9. Add trace ID into requests
10. Link app’s logs
11. Log and index retries
12. Do not use assertions as breakpoints
13. Always show debug info

---

## Do not treat end-to-end tests as unit tests

As I mentioned earlier, one of the biggest problems with my test suite was the obscure assertion messages. The reason was that the previous developers had written the assertion messages as they were intended for simple unit tests.

For example: “has no error”, “expected an error”, “not expected output”, “wrong function calls”, “build setup failed”, “happened unexpected error”, “failed to send message”, “key is required”, “invalid happy path”, etc. — all these assertion messages do not give a clear understanding of what is going on.

Yes, the tests should be simple, but **the checks in E2E tests involving a complex sequence of actions are different from checks for a single function.** Even comparing two numbers should provide more detail in the case of an E2E test suite.

![Test report with obscure assert message; with clear assert message.](/assets/2025-10-29/01-obscure-clear-assert-message.png)

_Fig. 1. Test report with obscure assert message; with clear assert message._

Unit and integration tests are outside the scope of this article, but these articles are worth reading:

- [Testing in Go: Writing Practical Failure Messages](https://ieftimov.com/posts/testing-in-go-writing-practical-failure-messages/)
- Google Go Style Guide on [Error handling](https://google.github.io/styleguide/go/best-practices#error-handling) and [Adding information to errors](https://google.github.io/styleguide/go/best-practices#adding-information-to-errors) sections.

## Stick to BDD assertion style

BDD ([behavior-driven development](https://en.wikipedia.org/wiki/Behavior-driven_development)) assertion style is one of the best ways to write assertion messages for E2E tests. Test frameworks and assertion libraries, such as [Chai](https://www.chaijs.com/api/bdd/) for JavaScript or [Testify](https://github.com/stretchr/testify) for Go, provide a variety of ways to verify invariants that are suitable for BDD. **This assertion style assumes that there is some kind of _expectation_ that _should_ correspond to something.** Annotation of each assertion with a custom message in BDD style makes the test report more understandable.

```go
// This and subsequent code examples will relate to the Golang and Testify package
func TestSomething(t *testing.T) {
  assert.Equal(t, foo, bar, "foo and bar variables should be equal")
}
```

Perceiving a failure message as «variables should be equal» is cognitively simpler than perceiving it as «variables not equal». The latter statement raises the question: «Did it fail because of equality or inequality?» The BDD statement clearly indicates what was expected from the check.

![Test report with plain assert message; with BDD-style assert message.](/assets/2025-10-29/02-bdd-style.png)

_Fig. 2. Test report with plain assert message; with BDD-style assert message._

Using BDD assertion style does not force to follow other aspects of behavior-driven development.

Read further:

- [Streamline Your Assertion Testing](https://www.browserstack.com/guide/assertion-testing)
- [Fluent and BDD assertions](https://web.dev/learn/testing/assertions/tools#fluent_and_bdd_assertions)

## Output expected and actual data

This issue often depends on the test framework, but in any case output the expected value and actual result into the test report whenever possible. Even with a good error message, it is difficult to understand exactly what went wrong without this data. This is why **it is crucial for debugging.**

![Test report without tested data; with expected and actual data.](/assets/2025-10-29/03-expected-actual-data.png)

_Fig. 3. Test report without tested data; with expected and actual data._

## Try to avoid Boolean checks

With a Boolean check it is difficult to determine what went wrong in the test.

In most test frameworks, a failure occurs during such a check like [`assert.True(t, fooBar)`](https://pkg.go.dev/github.com/stretchr/testify/assert#True) will lead to an error message «**false is not true**». This is not very convenient for debugging due to quite an unmeaningful report.

If you perform Boolean checks, be sure to add a custom assertion message for informational purposes.

```go
func TestSomething(t *testing.T) {
  assert.True(t, fooBar, "fooBar element should be true after the event has occurred")
}
```

It is much better to change the check to a more explicit one. Of course, if business requirements allow it.

![Test report with Boolean check; with explicit check and clear assertion message.](/assets/2025-10-29/04-boolean-check.png)

_Fig. 4. Test report with Boolean check; with explicit check and clear assertion message._

## Pay attention to errors in Golang

If your tests are written in Go, then you probably have a lot of error checks with this code pattern:

```go
someObj, err := SomeFunction()
if err != nil {
  // handle error
}
```

If your E2E API tests are written in Go, then you probably handle errors using [`assert.NotNil(t, err)`](https://pkg.go.dev/github.com/stretchr/testify/assert#NotNil) or [`assert.Error(t, err)`](https://pkg.go.dev/github.com/stretchr/testify/assert#Error). Unfortunately, this is not very convenient for debugging because this type of check can result in a misleading error message, such as «**error has no error**».

If you perform checks on errors, be sure to add a custom assertion message with additional information about the called function.

```go
someObj, err := SomeFunction()
assert.Error(t, err, "SomeFunction should not return error")
```

![Test report with error check; with error check and explanatory assert message.](/assets/2025-10-29/05-error-golang.png)

_Fig. 5. Test report with error check; with error check and explanatory assert message._

Read further on handling errors in Go:

- [Error handling and Go](https://go.dev/blog/error-handling-and-go)
- [Errors are values](https://go.dev/blog/errors-are-values)
- [[ On | No ] syntactic support for error handling](https://go.dev/blog/error-syntax)
- [Comprehensive Guide to Testing in Go](https://blog.jetbrains.com/go/2022/11/22/comprehensive-guide-to-testing-in-go/)
- [Go’s Error Handling Is a Form of Storytelling](https://preslav.me/2023/04/14/golang-error-handling-is-a-form-of-storytelling/)

## Log API requests

End-to-end API tests consist of a sequence of HTTP (or another protocol) calls. Therefore, it is highly valuable to log and output these calls into the test report. This information is the most important part of the API test.

I recommend focusing on the bare minimum: URL and [cURL](https://curl.se/). **cURL will include all information about the request,** including query parameters, headers, and body. It can be copy-pasted, and adding [`-vv`](https://curl.se/docs/manpage.html#-v) inside cURL’s string will help a lot with manual debugging.

![Test report without logging; with logging API request.](/assets/2025-10-29/06-logging-request.png)

_Fig. 6. Test report without logging; with logging API request._

Unfortunately, API requests’ data is not standard information for test frameworks. It must be processed manually and added to the report. Additional coding is unavoidable here.

- Playwright has an [annotations](https://playwright.dev/docs/api/class-testinfo#test-info-annotations) method in `TestInfo` class;
- Allure Report, an advanced reporting tool, has everything for [improving readability of test reports](https://allurereport.org/docs/gettingstarted-readability/).

## Log full API response

The response is the second most important part of the API test, along with the request. If the status code can be output and verified inside its own assertion, the response body should be fully displayed in the report. In case of an error, **the body may contain useful information and stack traces.**

![Test report without logging; with logging API response.](/assets/2025-10-29/07-logging-response.png)

_Fig. 7. Test report without logging; with logging API response._

If the body is too large, it can be attached to the report as a file. Even better, attach the full response, including the status line, headers, and message body. Again, coding is unavoidable here.

- Playwright has an [attachments](https://playwright.dev/docs/api/class-testinfo#test-info-attachments) method in `TestInfo` class.

## Output unique IDs as separate parameters

Sometimes, rather than examining the request body, it is easier to display the main test parameters and unique IDs in a separate list of parameters within the report itself.

![Test report with test data IDs.](/assets/2025-10-29/08-test-data-parameters.png)

_Fig. 8. Test report with test data IDs._

Parameters that will remain constant throughout the whole test can be put to the very top of the report.

## Add trace ID into requests

In the application’s logging system, each API request from autotests should be distinguishable from those of real users. This distinction is useful for debugging and analytics.

The most common way to do this is to specify a **unique identifier using the optional HTTP request header, X-Request-ID.**

```
curl --request GET \
  --url 'https://test.example.net/api/endpoint' \
  --header 'X-Request-ID: e2e_api_test-202cb962-ac59-075b-964b-07152d234b70'
```

As mentioned above, the unique ID (trace ID) should be included in the test parameter annotations.

![Test report with request’s trace ID.](/assets/2025-10-29/09-trace-id.png)

_Fig. 9. Test report with request’s trace ID._

Read further:

- [X-Request-ID](https://http.dev/x-request-id)
- [Understanding Request ID: Why It’s Essential for Modern APIs](https://dev.to/kittipat1413/understanding-request-id-why-its-essential-for-modern-apis-1916)

## Link app’s logs

The next big deal for debugging is the ability to quickly access an application’s logging system, metrics, or dashboard. By correctly transmitting the time range of the request and its unique parameters, you can **immediately gain insight into the app’s state during the test request.**

![Test report with a link to the logs.](/assets/2025-10-29/10-link-logs.png)

_Fig. 10. Test report with a link to the logs._

## Log and index retries

It is useful to know about retries of individual requests, steps, or entire tests when reviewing a test report. **Seeing how many retries occurred is helpful as well.**

Some contemporary test frameworks contain advanced retry mechanisms. For example, Playwright allows:

- [Retry](https://playwright.dev/docs/test-retries#retries) tests in case of failure through config setting;
- Poll expectations until pass through the [`expect.poll`](https://playwright.dev/docs/test-assertions#expectpoll) method;
- Retry blocks of blocks of code until they are passing successfully through the [`expect.toPass`](https://playwright.dev/docs/test-assertions#expecttopass) method;
- [Rerun only failed tests](https://playwright.dev/docs/running-tests#run-last-failed-tests) from the last test run through `--last-failed` (kind of manual retry on the CLI level).

![Example of Playwright report with retries.](/assets/2025-10-29/11-playwright-retries.png)

_Fig. 11. Example of Playwright report with retries._

Since retries are mainly needed for UI and E2E tests, some test frameworks do not offer this option out of the box. Implementing this functionality requires good coding skills of test automation engineers.

![Test report without retries output; with a clear display of retries.](/assets/2025-10-29/12-retries.png)

_Fig. 12. Test report without retries output; with a clear display of retries._

## Do not use assertions as breakpoints

Please try to avoid using assertions as breakpoints within the logic of your tests. Assertions should verify invariants of your code rather than stopping (or continuing) the test execution.

Here is an example of this anti-pattern:

```go
if someСondition {
  assert.True(t, true, "this logical branch is fine")
} else {
  assert.True(t, false, "this logical branch is out of order")
}
```

There are two more anti-patters in the provided example:

- Test containing conditional logic (IF statements or loops);
- Test containing hardcoded assertions, like `assert.True(t, true)`

[Test smells](https://testsmells.org/) and test automation **anti-patterns make test reports poor and substandard.**

The fix of this issue depends on the context of each individual test.

Read further:

- [Conditional Assertions](https://test-smell-catalog.readthedocs.io/en/latest/Test%20semantic-logic/Other%20test%20logic%20related/Conditional%20Assertions.html), The Open Catalog of Test Smells
- [Conditional Verification Logic](https://test-smell-catalog.readthedocs.io/en/latest/Test%20semantic-logic/Other%20test%20logic%20related/Conditional%20Verification%20Logic.html), The Open Catalog of Test Smells
- [Test smells: cleaning up unit tests](https://qameta.io/blog/cleaning-up-unit-tests/) (these recommendations also apply to E2E tests)
- [Principles of Writing Automated Tests](https://adequatica.github.io/2022/09/20/principles-of-writing-automated-tests.html)

## Always show debug info

Debug information is important not only in case of a failure, but also in case of successful tests. Therefore, **the report should always contain debug information.** Such a report makes it possible to restore the test flow, manually reproduce all steps, use it as an example for documentation purposes, and obtain all test parameters and IDs for follow-up analysis.

![Test report for passed test with debug info.](/assets/2025-10-29/13-passed-test-debug-info.png)

_Fig. 13. Test report for passed test with debug info._

Do not worry about including «redundant or unnecessary» information in the output. **Redundancy in reports is OK for E2E tests.** You’ll thank yourself later when you start debugging your tests.

There is always room for improvement in test reports. Even the listed items only represent part of what is logged in real production test reports for complex E2E API tests.

Copy @ [Medium](https://adequatica.medium.com/make-enriched-e2e-api-tests-reports-c01d7dd79fe9)
