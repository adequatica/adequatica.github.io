---
layout: post
title: "Field Notes in API Testing, Part 1: Areas of Focus"
date: 2025-01-18 17:51:44 +0100
tags: testing
---

Being a test engineer (QA) for a decade, I’ve collected over a hundred notes related only to API testing.

![DALL-E 3 prompt](/assets/2025-01-18/00-cover-dall-e-1.jpg)

_DALL-E 3 prompt: draw a technical drawing of complicated motion transmission and force-by-gear wheels, 2.5D view of the gear train with spur gears showing tangent contact between their pitch circles_

I already shared [my field notes on general testing topics](https://adequatica.github.io/2022/09/26/field-notes-in-software-testing.html), but this time, I summarized my notes using AI (GPT-4o). I gave all my hundreds of notes on API testing to AI to process them (~ 91650 characters, ~ 19k tokens). Asking summarization prompts to the corpus of my notes, **I got a pretty decent list of aspects of API testing** after heavy manual editing and adding edge cases. The final result I decided to be divided into two parts: areas of focus and [practical approaches to API testing](https://adequatica.github.io/2025/01/18/field-notes-in-api-testing-part-2-approaches.html).

**Key Areas of Focus for API Testing:**

1. Functionality Testing
2. Performance Testing (may include several types of non-functional testing)
3. Compatibility Testing
4. Security Testing
5. Error Handling Testing
6. Usability Testing

_Disclaimer: Many sources of information have been lost, and I can not add proofs for some categorical or controversial cases. The provided information mainly relates to RESTful APIs over HTTP._

---

### Background

First of all, you cannot start API testing without a background in the concept of [REST](https://en.wikipedia.org/wiki/REST) (Representational State Transfer) API and a [basic understanding of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) (Hypertext Transfer Protocol), or rather, what parts the request and response consist of.

![An example of HTTP request from client to server](/assets/2025-01-18/01-request.png)

_Fig. 1. An example of HTTP request from client to server (may contain a body for some methods like POST, PUT, or PATCH)_

![An example of HTTP response from server to client](/assets/2025-01-18/02-response.png)

_Fig. 2. An example of HTTP response from server to client (body in optional)_

Secondly, you need to figure out confusing terminology. The terms _resource, endpoint,_ and _URI_ mean about the same thing. A _resource_ is an entity on the server whose _endpoint_ specifies the path to it by _URI_ ([Uniform Resource Identifier](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier), [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986)). While the term _handler_ implies a way to call a specific resource, so it is a _method + endpoint_ (e.g., `GET http://example.com/api/v1/foobar`).

A URL ([Uniform Resource Locator](https://en.wikipedia.org/wiki/URL), [RFC 1738](https://datatracker.ietf.org/doc/html/rfc1738)) is a subset of URI, but it differs from it in that it provides an access mechanism (scheme). For example, `example.com/api/v1/foobar` is a URI; however, `https://example.com/api/v1/foobar` is already a URL (and still a URI, [RFC 6570](https://www.rfc-editor.org/rfc/rfc6570)).

Protocol, domain, path, and query parameters are parts of the URL and, therefore, parts of the endpoint (and URI). Though API endpoints are often specified without a domain for the sake of simplicity, like `/api/v1/foobar?lang=en` — it calls relative URIs.

---

## Key Areas of Focus for API Testing

_Getting the API right is ¾ of how the product works._

### 1. Functionality Testing

Ensure the API behaves according to the defined specifications and performs the required operations correctly.

What it involves:

- Verifying that the API endpoints return the expected results;
- Validating the response for correct data and appropriate data formats;
- Ensuring that the API works according to HTTP specification and inherits best practices of API development;
- **End-to-end, exploratory, and contract testing are preferable methods for this scope of testing.**

All other types of testing that are not related to checking business logic can be classified as non-functional testing. In some companies, non-functional testing can be performed by dedicated specialists, not common test engineers.

### 2. Performance Testing (may include several types of non-functional testing)

Assess the speed and stability of the API under different load conditions.

What it involves:

- Measuring performance metrics (e.g., response time);
- Ensuring that performance does not degrade as usage increases;
- **Load testing** to evaluate API performance under no-load and peak traffic;
- **Stress testing** to determine the API’s limits;
- **Reliability testing** to ensure the API is stable over time.

### 3. Compatibility Testing

Ensure that the API works as intended when the application is deployed on different instance configurations or has different network configurations, which can affect the usage on other devices and third-party modules.

What it involves:

- Running the application on various environments;
- Validating API responses requested by different types of clients and tools;
- Testing backward compatibility, ensuring that clients using older versions still function as expected when the API is updated. — Keep in mind [Hyrum’s law](https://www.hyrumslaw.com).

Do not confuse compatibility API testing with cross-browsing testing — it is a common mistake of AI’s «recommendations» — it does not make sense cause REST API over HTTP works the same way and independently from the browser (unless the behavior of the API depends on the `User-Agent` header).

### 4. Security Testing

Ensure the API is secure, protects sensitive data (that it is transmitted securely over HTTPS), and can handle unauthorized access or potential vulnerabilities.

What it involves:

- Checking API URLs through HTTP and HTTPS protocols according to specifications;
- Testing authentication mechanisms (e.g., API keys, [OAuth Access Tokens](https://oauth.net/2/access-tokens/));
- Ensuring that the API performs authorization checks and denies access to resources based on user roles or permissions;
- [Testing for common vulnerabilities](https://adequatica.github.io/2019/07/28/use-postman-collection-runner-as-vulnerability-scanner.html) like cross-site scripting (XSS) in API requests. — It’s good to get acquainted with [OWASP Top 10 API Security Risks](https://owasp.org/API-Security/editions/2023/en/0x11-t10/);
- Ensuring no sensitive information is leaked in error messages in responses (see error handling testing).

### 5. Error Handling Testing

Ensure the API handles errors gracefully and provides helpful feedback.

What it involves:

- Testing error responses for cases of missing, invalid, or malformed requests;
- Trying _to break_ the application;
- Checking that status codes and error messages align with standards and documentation;
- **Ensuring that error responses do not contain implementation details** (an indication of programming language or framework version) **and stack traces for external users** (but internal users (developers) may need debug information in testing environments).

### 6. Usability Testing

Evaluate the ease of use, integration, and developer-friendliness of the API.

What it involves:

- Asking developers to add Swagger and follow [OpenAPI specification](https://www.openapis.org/what-is-openapi);
- Checking API documentation for clarity, completeness, comprehensibility, and unambiguity;
- Ensuring that developers (end users) can easily understand how to authenticate, make requests, handle responses, and process errors;
- Ensuring consistent and predictable API design with uniform naming conventions, clear parameter structures, and typed responses.

---

Read [the second part of **Field Notes in API Testing**](https://adequatica.github.io/2025/01/18/field-notes-in-api-testing-part-2-approaches.html), where practical approaches to API testing are covered, and [Field Notes in Software Testing](https://adequatica.github.io/2022/09/26/field-notes-in-software-testing.html) about general testing topics.

I deliberately skipped mentioning automation testing and testing tools because they are a whole different discussion area. Moreover, I already have articles on [Pragmatic Tools for Manual API Testing](https://adequatica.github.io/2022/06/05/pragmatic-tools-for-manual-api-testing.html) and [Browser DevTools as an Essential Tool for API Testing](https://adequatica.github.io/2022/06/01/browser-devtools-as-an-essential-tool-for-api-testing.html).

To keep abreast of current trends in API testing, check out the paper [Testing RESTful APIs: A Survey](https://dl.acm.org/doi/10.1145/3617175) (2023).

Copy @ [Medium](https://adequatica.medium.com/field-notes-in-api-testing-part-1-areas-of-focus-46b516ccacf4)
