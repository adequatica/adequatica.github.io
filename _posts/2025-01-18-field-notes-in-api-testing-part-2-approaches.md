---
layout: post
title: "Field Notes in API Testing, Part 2: Practical Approaches"
date: 2025-01-18 17:00:00 +0100
tags: testing
---

Being a test engineer (QA) for a decade, I’ve collected over a hundred notes related only to API testing.

![DALL-E 3 prompt](/assets/2025-01-18/00-cover-dall-e-2.jpg)

_DALL-E 3 prompt: provide application programming interface in the form of a technical drawing of complicated motion transmission and force-by-gear wheels, 2.5D view of the gear train with spur gears showing tangent contact between their pitch circles_

This is the second part of the summarization of my notes using AI (GPT-4o). I gave all my hundreds of notes on API testing to AI to process them. Asking summarization prompts to the corpus of my notes, **I got a pretty decent list of aspects of API testing** after strong manual compilation and editing. The final result I decided to be divided into two parts: [key areas of focus for API testing](https://adequatica.github.io/2025/01/18/field-notes-in-api-testing-part-1-areas-of-focus.html) and its practical approaches.

**Practical Approaches to API Testing:**

1. HTTP Methods
2. HTTP Status Codes
3. Authentication and Authorization
4. Headers
5. Path and Query Parameters
6. Versioning (a special case of path and query parameters)
7. Pagination (a special case of path and query parameters)
8. Input Payload Validation
9. Response Payload Validation
10. Error Handling

_Disclaimer: Many sources of information have been lost, and I can not add proofs for some categorical or controversial cases. The provided information mainly relates to RESTful APIs over HTTP._

---

## Practical Approaches to API Testing

During API testing, most of the described approaches are intertwined because you can not test HTTP codes and headers by ignoring authentication and body, and so on… everything is connected.

### 1. HTTP Methods

Ensure correct usage of HTTP methods for their intended purposes.

Dev best practices:

- Use methods appropriate to [HTTP specification](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) (GET for retrieval, POST for creation, DELETE for deletion, etc.);
- Keep GET and DELETE [idempotent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods#safe_idempotent_and_cacheable_request_methods);
- Avoid side effects in GET requests.

Testing methods:

- Send requests using each HTTP method and verify behavior.

Examples of test cases:

- Call the GET request several times in a row, and it must always return to the same state — **this is the idempotency confirmation test**;
- Send GET request with a body — API should ignore payload;
- Verify that POST request creates a resource (and responds with `200 OK` or `201 Created` сodes) — check it in the database or use a subsequent GET request;
- Evaluate the differences between POST, PUT, and PATCH methods if the API supports all of them;
- Request the same handler simultaneously — this check may reveal a race condition;
- Ensure that repeated DELETE requests on the same endpoint have no additional effects.

### 2. HTTP Status Codes

Verify that the API returns appropriate status codes for each operation.

Dev best practices:

- Use standardized codes according to [HTTP specification](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) (e.g., 200s for successful responses, 400s for client errors, 500s for server errors).

Testing methods:

- Validate response codes for positive and negative scenarios.

Examples of test cases:

- Ensure 200 codes come after successful operations and 300 codes come in case of redirects;
- Make sure that API does not have confusing differences between the status and the response (e.g., `200 OK` status code with a body like `{message: '500 Error'}` — this is an anti-pattern of API development);
- Check for `405 Method Not Allowed` when requesting an endpoint with an unsupporting method;
- Check for `400 Bad Request` when entering incorrect values (examples of such checks will be discussed in the «Error Handling» paragraph);
- Treat any 500 status code as a bug and potential vulnerability — **the backend should be able to handle errors, not crashes**.

### 3. Authentication and Authorization

Verify that the API endpoints use proper authentication and authorization mechanisms.

Know the difference:

- Authentication is the process of verifying the identity of a user/client/device/etc., but authorization determines their access rights and permissions.

Dev best practices:

- Use token-based authentication (e.g., [bearer tokens](https://swagger.io/docs/specification/v3_0/authentication/bearer-authentication/));
- Passwords/logins should not be shared in plain text;
- Require authentication by default. Only exceptional endpoints may work without authentication.

Testing methods:

- Send requests using specified authentication methods;
- Send requests using the credentials of users with different roles.

Examples of test cases:

- Test access with valid, invalid, and expired tokens;
- Check for `401 Unauthorized` and `403 Forbidden` status codes on restricted access;
- Compare to compliance of responses to requests with different credentials;
- Try to request endpoints without authentication methods.

### 4. Headers

Ensure correct usage of HTTP headers.

Dev best practices:

- Use [HTTP headers properly](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers);
- Include [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header to specify the format unambiguously.

Testing methods:

- Send requests with or without required headers and check the presence of required headers in responses;
- Read the documentation on support of «special» or custom headers. Is there any business logic based on [`Etag`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) of `If-*` (such as `If-Modified-Since`) headers?

Examples of test cases:

- Validate API’s behavior with missing or invalid headers;
- Check that the response does not contain unnecessary headers because they may cause unwanted client behavior (e.g., [`Host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Host) header makes sense only in the request, but if it appears in the response, your server is probably misconfigured);
- Check that the response does not contain duplicate headers because it increases the size of the overall response;
- Pay attention to header size — [there is a limit of 4–16K depending on the server](https://stackoverflow.com/questions/686217/maximum-on-http-header-values);
- Recheck [`Content-Security-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) header because it may accumulate outdated or duplicated data over time;
- Ensure [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header matches the payload format (e.g., if the backend successfully receives JSON but has `Content-Type: text/html` header — this can lead to XSS exploitation by passing the HTML tag inside JSON);
- Modify [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) headers (e.g., [`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin)) to test cross-origin requests — this is a part of security testing;
- Check API’s behavior dependence on [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) header — this is a part of compatibility testing;
- Check that API reacts correctly (compresses correctly) on [`Accept-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding) parameters;
- Send request with and without [`Connection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection): keep-alive header;
- Etc… If needed, it is possible to create test cases for any single header to test all the aspects of [content negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation).

Using headers for non-functional testing:

- [`Content-Length`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Length) can help collect metrics about the size of the transmitted information;
- [`Date`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Date) can be used as a reference point for calculating a response time.

### 5. Path and Query Parameters

Ensure that the API endpoints and their associated inputs operate as intended, adhere to best practices, and are robust against misuse or unexpected inputs. There is a lot to check out in this space: sorting, pagination, filtering, searching, etc.

Dev best practices:

- Endpoints should represent resources, not actions. Avoid verbs like `/getUsers`, prefer `/users`;
- Use plural forms for collections/datasets (e.g., `/users` instead of `/user`);
- Avoid filtering logic in the path; use query parameters for these (e.g., instead of `/users/page/1` do `/users?page=1`);
- Do not disclose API’s backend stack in endpoints like `/rest/node.js/foobar` — this is unacceptable;
- Ensure that paths and parameters are predictable, hierarchical, and consistent across all API handlers.

Testing methods:

- Send requests with modified path and query parameters;
- Boundary value analysis and equivalence class testing;
- Pairwise testing (a special case of [all-pairs testing](https://en.wikipedia.org/wiki/All-pairs_testing) to cover all query parameter combinations).

**Testing of paths and query parameters is directly related to the business logic.** When you change values of sorting, filtering, searching, and other query parameters, the response should be obtained in accordance with the business logic. It is up to the test engineers’ experience, purpose, and conditions of the conducted testing to decide whether to combine these checks or not.

Examples of test cases:

- Check endpoints naming consistency;
- Request resources with and without backslash at the end of the URL: `/foobar` and `/foobar/` — although the behavior depends on the backend configuration, it should be the same for all endpoints;
- Test each query parameter with more than one correct value (equivalence classes help there);
- Test combinations of query parameters (pairwise testing helps there with a set of test cases);
- Provide for each query parameter: the edge case of the value, invalid value, and invalid data type value;
- Use [randomization to generate values](https://adequatica.github.io/2020/11/22/randomization-testing-of-filter-handler-in-postman.html) for search and filter parameters;
- Ensure that rearranging query parameters does not affect the result — this is the test for [commutativity](https://en.wikipedia.org/wiki/Commutative_property);
- Fill query parameter values [with XSS vectors or SQL injections](https://adequatica.github.io/2019/07/28/use-postman-collection-runner-as-vulnerability-scanner.html) (e.g., `/foobar?items=1;drop table temp --`) — this is a part of security testing;
- Request resources with partly [URL encoding](https://en.wikipedia.org/wiki/Percent-encoding) endpoints (e.g., `/users?search=D%C5%BEejms%20D%C5%BEojs%26lang%3Den` which is the same as `/users?search=Džejms Džojs&lang=en`);
- Assemble the endpoint with the maximum possible number of parameters and its values. If the limit set on the server is exceeded, you will receive a response with `414 URI Too Long` status.

### 6. Versioning (a special case of path and query parameters)

Verify the API supports multiple versions without breaking backward compatibility.

Dev best practices:

- Specify the version number, even if there is no plan to change the API — everything can change quickly;
- Use semantic versioning (e.g., `v1.20.345`);
- Specify the version in the endpoint’s path (URI versioning), like `/api/v1.5/foobar` or in query parameters `/api/foobar?ver=1.5`, or in headers (Media Type versioning);
- Maintain older versions.

Despite all the existing best practices, developers often use only a major number for API versioning, like `v1` or `v2` (`/api/v1/foobar` or `/api/foobar?ver=1`). Actually, it is pretty convenient for clients to stick to one version and not keep updating versions with every minor bugfix or update (while strictly following [semantic versioning guidelines](https://semver.org) requires incrementing the version in case of any change).

Testing methods:

- Regression testing for old versions;
- Back-to-back testing — [comparing two versions of the same resource](https://adequatica.github.io/2020/09/23/a-brief-comparison-of-responses-in-postman.html).

Examples of test cases:

- Call endpoint without a version — some APIs may have a fallback to the default one;
- Verify that updates to `v2` do not break `v1` — this is a part of backward compatibility testing;
- Ensure that new fields in requests/responses are optional and ignored in older versions.

### 7. Pagination (a special case of path and query parameters)

Validate that the API correctly returns data in chunks according to the specified range.

Dev best practices:

- Include pagination for listable collections/datasets, even if results are small, to future-proof the API;
- It is more flexible (and convenient for clients) to provide pagination through `limit` and `offset` parameters (`?limit=10&offset=0`) than a fixed page (`?page=1`);
- Use the same pagination method across all endpoints;
- Constrain the maximum value for a `limit` (or a page size) to avoid overwhelming the system;
- Provide hypermedia controls (next and previous links) in the response body to allow clients to navigate pages easily:

```json
{
  "items": [...],
  "links": {
    "next": "/items?page=2",
    "previous": null
  }
}
```

Testing methods:

- Boundary value analysis and equivalence class testing.

Examples of test cases:

- Zero page (e.g., `?page=0`, `?limit=0&offset=0`);
- First page (e.g., `?page=1`, `?limit=1&offset=1`);
- Last page;
- Exceed the total number of pages (e.g., `?page=2147483647`) — **ensure an empty result is returned instead of an error**;
- The last item in response to `?limit=10&offset=1` and the first item in `?limit=10&offset=10` should be the same (in this case, getting a completely «new page» is obtained by `?limit=10&offset=11`);
- Try maximum limit and offset. Does API allow getting all items? — this can be a part of performance testing on how the backend handles processing a large amount of data;
- Call endpoints without pagination;
- Does API have default values for pagination parameters?

Be aware of exotic pagination methods, like through [`Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range) header.

### 8. Input Payload Validation

Ensure the API handles user inputs interpreted correctly and effectively. Input payload testing applies only to methods that produce changes and include a body in the request, like POST, PUT, or PATCH.

Dev best practices:

- Enforce type and format validation; for this aim, set constraints like minimum/maximum lengths for strings, number ranges, and allowed values for enumerations;
- Ignore unexpected fields;
- Sanitize inputs to prevent SQL injection and XSS;
- Return detailed error messages in case of [bad requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400) so that the user (another developer) can clearly understand the reason for his error.

Testing methods:

- Send requests with different request body values and verify the API’s behavior in the response;
- Boundary value analysis, equivalence class testing, and pairwise testing.

**Testing input payload is directly related to the business logic.** When you create (POST) and update (PUT or PATCH) something via transmission of the body, this should be reflected in the response or subsequent related requests. For this reason, **end-to-end testing is the right and comprehensive approach for that task.**

Examples of test cases:

- Send valid and invalid JSON, XML, or other appropriate data format;
- Ensure `Content-Type` header matches the payload format;
- Send unintended fields in JSON body;
- Send an empty body (by the way, it is a valid JSON);
- Send _enormous_ body payload (maximum payload size can be a non-functional requirement);
- Test boundary values for numeric fields.

### 9. Response Payload Validation

Validate the content of the response payload for correctness and consistency.

Again, **testing response payload is directly related to the business logic.** Response body highly depends on the received request payload, filtering and sorting parameters, and many other factors. **Contract testing can serve a good purpose in this area.**

Dev best practices:

- Use a standard response structure for all resources;
- Return only necessary data;
- Provide responses in JSON format;
- Provide only valid JSON response payload;
- Use standardized types for standard fields, like [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) for dates: `2025-01-14T15:16:20Z`;
- Provide additional metadata for responses, such as pagination details or a total number of items:

```json
"meta": {
  "page": 1,
  "page_size": 10,
  "total_pages": 5
}
```

- Return meaningful responses for empty datasets:

```json
{
  "data": [],
  "meta": {
    "total": 0
  }
}
```

- Provide hypermedia controls (next and previous links) in the response body to allow clients to navigate pages easily;
- Avoid exposing internal details in the response — do not return stack traces, implementation details, or internal identifiers (file path to the resources).

Testing methods:

- Send requests with different request body values and verify behavior according to business logic;
- Schema validation.

Examples of test cases:

- [Validate the received body for valid JSON](https://adequatica.github.io/2022/07/19/json-data-validation-levels.html) (by the way, an empty body is a valid JSON);
- Ensure `Content-Type` header matches the payload format;
- Check for required fields in the response;
- Validate data types and formats;
- Check for default key values;
- Check metadata fields (if they exist) for correctness;
- Test for missing, null, or unexpected fields;
- Pay attention to the consistency of answers. If one handler responds with 200 OK status with the body `"result": {}`, then the other one with the same method and the same status code should not have a body like `"result": { "data": {} }` (but of course, it depends on the context).

### 10. Error Handling

Validate that the API handles errors and returns meaningful error messages with corresponding HTTP codes. This is a rare place where you need to try _to break_ the application.

One of the purposes of API testing is to find all `500 Internal Server Error` errors — each of which is a bug and potential vulnerability (the end user sees only an error code, but under the hood, the backend may be totally crashed); **robust API should not crash.**

Dev best practices:

- Use standard and structured error format for all errors;
- Use descriptive error messages;
- Avoid revealing stack traces or internal implementation details.

Testing methods:

- Send any kind of invalid data: malformed endpoints, missing fields, and incorrect payloads;
- Boundary value analysis and equivalence class testing, but in this case, you need to go beyond boundary values. Here, you should know the maximum integer limits in different programming languages;
- Error guessing test-design technique.

Testing API for error handling should be done not separately but jointly with testing other API areas: check error status codes (№2), check errors for unauthorized uses (№3), check [bad requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400) in case malformed requests (№5 and №8), check error messages in response payload (№9), etc.

Examples of test cases:

- Pass invalid path and query parameter values in URL;
- Pass valid path but to absent/missing resource to get `404 Not Found` status;
- Pass invalid payload in the body;
- For parameters/keys with the number type, pass the value exceeding its maximum limit (e.g., it can be [9007199254740992](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER) for backends on Node.js);
- Ensure that [bad request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/400) errors have clear messages about the reasons for an error;
- Send _many_ requests in a row. If the limit set on the server is exceeded, you will receive a response with `429 Too Many Requests` status or worse, 503 or 504 error codes if the backend cannot handle the load — this is a part of performance testing;
- **Ensuring that error responses do not contain implementation details** (an indication of programming language or framework version) **and stack traces for external users.**

---

Read [the first part of **Field Notes in API Testing**](https://adequatica.github.io/2025/01/18/field-notes-in-api-testing-part-1-areas-of-focus.html), where key areas of focus for API testing are covered, and [Field Notes in Software Testing](https://adequatica.github.io/2022/09/26/field-notes-in-software-testing.html) about general testing topics.

I deliberately skipped mentioning automation testing and testing tools because they are a whole different discussion area. Moreover, I already have articles on [Pragmatic Tools for Manual API Testing](https://adequatica.github.io/2022/06/05/pragmatic-tools-for-manual-api-testing.html) and [Browser DevTools as an Essential Tool for API Testing](https://adequatica.github.io/2022/06/01/browser-devtools-as-an-essential-tool-for-api-testing.html).

Copy @ [Medium](https://adequatica.medium.com/field-notes-in-api-testing-part-2-approaches-42973b9911cb)
