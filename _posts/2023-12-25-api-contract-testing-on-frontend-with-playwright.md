---
layout: post
title: "API Contract Testing on Frontend with Playwright"
date: 2023-12-25 07:56:19 +0100
tags: api automation playwright testing
---

Sometimes, as a test engineer, business requirements for testing may be quite weird, and you have to adopt different types of testing in one suite.

![Midjourney prompt](/assets/2023-12-25/00-cover.jpg)

_Midjourney prompt: http api contact test frontend application, 2d abstract scheme, engineering drawing, kandinsky style, white background_

Contract testing is a type of software testing that focuses on verifying the interaction between separate components/services (more often, it is two microservices). When two microservices interact via the API, one service sends requests in a predefined format, and another responds in a predefined format. This format is called a «contract» — an agreement between services (and even dev teams) on how they commit to communicating with each other.

A «contract» can be an API specification, but more often, it is just a request and response bodies’ schemas as JSON files, which are shared between two services, and both of them test their APIs against these schemas — this approach is even distinguished into a separate testing method «[schema-based contract testing](https://pactflow.io/blog/contract-testing-using-json-schemas-and-open-api-part-1/)».

In the case of client-server architecture, the frontend can act as a consumer or provider for various APIs and vice versa:

![Frontend application as a consumer or provider for contract testing](/assets/2023-12-25/01-contract-provider-consumer.png)

_Fig. 1. Frontend application as a consumer or provider for contract testing_

In many articles (see links at the end of the article), contract testing is opposed to integration or end-to-end testing. But in this article, I want to show that **contract testing can be a part of end-to-end testing** — it can be just a tool for specific checks.

This occasion can happen in the case of **specific business requirements for frontend autotests**, for example, to check that your frontend makes specific requests to third-party APIs in a particular format. In other words, to ensure that UI sends correct data.

Moreover, these third-party APIs [may not allowed to be requested](https://adequatica.github.io/2022/12/22/layers-of-defense-against-data-modification.html) during the tests. That looks like an obvious necessity to «close» these third-party APIs by mocks, but what if you don’t have any complex mocking infrastructure on your test project (and/or don’t want to have)? It can be so if your [tests run against a live staging environment](https://adequatica.github.io/2023/12/04/pros-and-cons-of-the-ways-of-end-to-end-automated-testing-in-ci.html). In such cases, you can «close» requests to third-party APIs on a [network level by Playwright](https://playwright.dev/docs/network).

> It was not a problem to find a similar project on the Internet. There are a lot of DeFi startups who use open APIs for their infrastructure, but these APIs are mostly GraphQL and JSON-RPC — which adds a little bit of complexity to the example. The description of their differences from REST API is not the topic of this article.

At least I found [Sushi](https://www.sushi.com/swap) cryptocurrency swap page, whose frontend does just a few desired POST requests to third-party API (API’s URL is different from the current website):

![Website’s frontend does POST requests to third-party APIs](/assets/2023-12-25/02-sushi-swap.png)

_Fig. 2. [Website](https://www.sushi.com/swap)’s frontend does POST requests to third-party APIs_

The same case in a scheme representation looks like this:

![Website’s frontend does POST request to third-party API](/assets/2023-12-25/03-contract-third-party.png)

_Fig. 3. Website’s frontend does POST request to third-party API_

> Let me remind you that I focus on third-party APIs because checking an internal API is not the case for this article — you can check your internal APIs by your internal API tests and/or integrational ones.

In contract testing, it is implied that each component/service is isolated from each other. And here, you can easily isolate frontend from third-party API with Playwright’s network capabilities:

1. Roughly [abort request](https://playwright.dev/docs/network#abort-requests) — the request **will not be sent** to an external API;
2. Or [mock it](https://playwright.dev/docs/mock#mock-api-requests).

For the second case, you can modify the response with the abortion of the request if you use only [`fulfill()`](https://playwright.dev/docs/api/class-route#route-fulfill) class. But if you use `fulfill()` with [`fetch()`](https://playwright.dev/docs/api/class-route#route-fetch), the request will be sent to the external API. In both ways, **when you fulfill the response body with JSON — you make contract testing** (checking that the client is processing the fulfilled response) if this JSON scheme is the same as the one used for testing on the side of the external API.

For both cases, **you intercept the request by [`waitForRequest()`](https://playwright.dev/docs/api/class-page#page-wait-for-request) class for testing POST’s [request body](https://playwright.dev/docs/api/class-request#request-post-data) against your contract** (and, of course, for PUT or PATCH methods):

![HTTP request interception through Playwright](/assets/2023-12-25/04-http-request-interception-through-playwright.png)

_Fig. 4. HTTP request interception through Playwright_

If your request body is in JSON format (I think this will happen 90 percent of the time), you can instantly use [`postDataJSON()`](https://playwright.dev/docs/api/class-request#request-post-data-json) class for comparison JSON schemes by your favorite tool: [Ajv](https://ajv.js.org/json-schema.html), [Zod](https://zod.dev/), or use [`toEqual()`](https://playwright.dev/docs/api/class-genericassertions#generic-assertions-to-equal) assertion, if for some reason you decide to compare the two JSON objects head-on.

While you are checking only the request’s contract, you may not need the response and may simply abort it (attention, the right behavior depends on your application, and maybe you have to mock the response to prevent the application’s crash):

![How route.abort() works in Playwright Inspector](/assets/2023-12-25/05-sushi-swap-abort.png)

_Fig. 5. How `route.abort()` works in Playwright Inspector_

Here is the [code example](https://github.com/adequatica/ui-testing/blob/main/tests/sushi-swap-contract-testing.spec.ts) of such a test:

```javascript
import { expect, type Page, test } from '@playwright/test';
import { z } from 'zod';

// Contract
const schema = z.object({
  jsonrpc: z.string(),
  id: z.number(),
  method: z.string(),
  params: z.array(z.union([z.string(), z.boolean()])),
});

let page: Page;

test.beforeAll(async ({ browser }) => {
  const context = await browser.newContext();
  page = await context.newPage();

  await page.route(/.+lb\.drpc\.org\/ogrpc\?network=ethereum.+/, async (route) => {
      if (route.request().method() === 'POST') {
        await route.abort();

        return;
      }
    },
  );
});

test('Open Sushi Swap', async () => {
  // Waiting for a request should be before .goto() method,
  // because desired request can be done before the page is fully loaded.
  const requestPromise = page.waitForRequest(
    (request) =>
      request.url().includes('lb.drpc.org/ogrpc?network=ethereum') &&
      request.method() === 'POST',
  );

  await page.goto('/swap');

  const request = await requestPromise;
  await expect(
    () => schema.parse(request.postDataJSON()),
    'Should have a request by the contract',
  ).not.toThrowError();
});
```

Where,

- `const schema` is a scheme declaration in [Zod](https://zod.dev/)’s format;
- In `beforeAll` hook, all POST requests to URLs match https://lb.drpc.org/ogrpc?network=ethereum&dkey=Ak765fp4zUm6uVwKu4annC8M80dnCZkR7pAEsm6XXi_w are blocking;
- `const requestPromise` receives data from the first request matches https://lb.drpc.org/ogrpc?network=ethereum&dkey=Ak765fp4zUm6uVwKu4annC8M80dnCZkR7pAEsm6XXi_w;
- In `expect()` assertion, the reference `scheme` is parsed against the request’s data. The test passes if the parsing/validating process does not fail — [`toThrowError()`](https://jestjs.io/docs/expect#tothrowerror).

The test presented above may contain more steps and checks because the contract’s check may be just a part of the end-to-end suite.

Read more about contract testing:

- [What is contract testing and why should I try it](https://pactflow.io/blog/what-is-contract-testing/)?
- [Contract Testing Vs Integration Testing](https://pactflow.io/blog/contract-testing-vs-integration-testing/);
- [A Complete Guide to API Contract Testing](https://testsigma.com/blog/api-contract-testing/);
- [API contract testing: 4 things to validate to meet expectations](https://blog.postman.com/api-contract-testing-4-things-to-validate/);
- [Contract Testing: The Key to Unlocking E2E Testing Bottlenecks in CI/CD pipelines](https://www.youtube.com/watch?v=RSl_JcWKE3M).

---

Furthermore, as an idea, the same approach to mocking can be applied to all HTTP API requests on frontend:

![Mock APIs](/assets/2023-12-25/06-contract-mock-apis.png)

_Fig. 6. Mock APIs_

Read more about how [Playwright mocks API](https://playwright.dev/docs/mock).

This article was featured in:

- [Appium Interceptor, Automation Trends, Playwright and More](https://www.youtube.com/watch?v=GNxuqakgkgs) by Automation Testing with Joe Colantonio;
- [Contract Testing in Playwright using Zod](https://www.youtube.com/watch?v=jtg4By7I8XI) by Uncle Aaroh Testing.

Copy @ [Medium](https://adequatica.medium.com/api-contract-testing-on-frontend-with-playwright-4509b74b3008)
