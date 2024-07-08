---
layout: post
title: "API Testing with Vitest"
date: 2023-12-04 09:26:13 +0100
tags: api automation testing vitest
---

As a test engineer (actually, as any engineer and web developer), you should reconsider your stack from time to time. Using Jest with Got or Axios for API testing can sound outdated in 2K23 cause we have Vitest and Fetch API in Node.js.

![API Testing with Vitest](/assets/2023-12-05/00-cover-caledoscope.jpg)

Someone can say that «_testing API with JS is a bad idea_»; «_JS fits only for frontend…_» The argument in support of these theses is that APIs should be tested on «backend languages» (or on a language in which a serverside code is written, unless, of course, it is written on Node.js): Python, Java, Go, etc.

However, testing API is not the same as testing backend code. API is an Application Programming Interface — it is an interface to the backend — a collection of defined HTTP endpoints that are not tied up to programming language. **API testing is suitable for any convenient language.**

Therefore, testing API with JavaScript can not considered delirious. If a project already has end-to-end frontend autotests on JavaScript (most likely based on Puppeteer, Playwright, Cypress, or WebdriverIO), that JavaScript for API testing can be scored as a stack unification.

Ten years ago, JavaScript infrastructure could not provide enough tools for testing. Your selection was lacking: Mocha, Jasmine, Karma, Webdriver (it did not even have a test runner at that time and provided only Node.js API to Selenium), and probably something else that no one will remember now. But shortly, the explosive growth of web technologies spawned advanced testing tools, and it became possible to build a JavaScript testing architecture of any complexity from various open-source projects. You can even settle your API tests based on [UI testing frameworks like Cypress and Playwright](https://javascript.plainenglish.io/api-testing-comparison-cypress-vs-playwright-vs-jest-2ff1f80c5a7b)!

**Jest + favorite HTTP client** ([Axios](https://axios-http.com/), [Got](https://github.com/sindresorhus/got), [superagent](https://github.com/ladjs/superagent)/[supertest](https://github.com/ladjs/supertest), or[ node-fetch](https://github.com/node-fetch/node-fetch)) **was a popular bundle** in recent years. At the same time, a few significant changes have happened over the past year: [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) started working out of the box in Node.js [21](https://nodejs.org/en/blog/announcements/v21-release-announce) (previously, you had to use it under the experimental flag), and [Vitest just released version 1.0](https://github.com/vitest-dev/vitest/releases/tag/v1.0.0). It is time to shake up the habitual testing stack!

## Why should you choose or switch to Vitest?

- **Vitest is a modern testing framework.** It was created for a present-day approach to JavaScript and dev experience, like TypeScript and ESM support out of the box.
- **Vitest is fully compatible with the Jest syntax** — [the migration won’t ​​take much effort](https://vitest.dev/guide/migration.html).
- **Vitest provides almost zero configuration** as opposed to the painful Jest one. No need for directories, file types, and environment setups — most of the default options are exactly what you need.

Vitest’s minimum config can be really minimalistic:

```JavaScript
import {defineConfig} from 'vitest/config';

export default defineConfig({
  test: {},
});
```

- **Vitest has retries out of the box!**
- **Vitest has built-in [Chai](https://www.chaijs.com/) assertions** and advanced assertions for types (via [expect-type](https://github.com/mmkal/expect-type), more convenient for unit testing).
- **Vitest has skip conditions** ([describe](https://vitest.dev/api/#describe) and [test](https://vitest.dev/api/#test) functions have a lot more useful methods than other test-runners).
- **Vitest runs tests in parallel by default.** However, for API testing, it may be highly convenient to limit the number of simultaneous API requests on the test-file level and make tests (test files) run forcedly sequentially.

Multi-threading can be disabled:

1. By passing `--pool=forks` in the CLI (from version 1.0; in older versions, it was `--no-threads`);
2. AND [adding the option singleFork=true into config](https://vitest.dev/config/#pooloptions-forks-singlefork) — it can be considered as an analog of `--runInBand` [in Jest](https://jestjs.io/docs/cli#--runinband):

```JavaScript
export default defineConfig({
  test: {
    poolOptions: {
      forks: {
        singleFork: true,
      },
    },
  },
});
```

(In older versions (the last one was 0.34.6), it was [command --no-threads](https://vitest.dev/guide/features.html#threads), which could be considered as an analog of [`--runInBand` in Jest](https://jestjs.io/docs/cli#--runinband).)

> NOTE: you must use `--pool=forks` when using `fetch()` [due to Vite’s peculiarity](https://github.com/vitest-dev/vitest/issues/3077#issuecomment-1815767839).

- **Vitest is fast** (some [user’s benchmark](https://dev.to/mbarzeev/from-jest-to-vitest-migration-and-benchmark-23pl#benchmark-again-and-compare)). It is fast not only for unit testing but even for API testing, despite the speed of testing framework usually being leveled by HTTP requests’ timings.

I compared approximately identical[ API tests written on Jest](https://github.com/adequatica/api-testing) and [their replication on Vitest](https://github.com/adequatica/api-vitesting).

- Average Jest’s execution time (`--runInBand`): 5.4 sec
- Average Vitest’s execution time (`--pool=forks` & `singleFork=true`): 3.2 sec

Vitest is almost more than one and a half times faster!

- **Vitest has a thoughtful reporter.** For example, in the final report it will collapse successful tests up to the file level. Anyway, you do not need test steps if everything is OK.

![Vitest reporter after a successful run](/assets/2023-12-05/01-run-passed.png)

_Fig. 1. Vitest reporter after a successful run_

![Vitest shows how tests are running](/assets/2023-12-05/02-run-beforeall.png)

_Fig. 2. Vitest shows how tests are running (multi-threading execution — notice beforeAlls are running at the same time)_

![Vitest reporter of a single test file shows all tests](/assets/2023-12-05/03-run-epic.png)

_Fig. 3. Vitest reporter of a single test file shows all tests_

- Vitest even has a [UI interface](https://vitest.dev/guide/ui.html). It may be convenient for test engineers to check the regression reports in a fancy style, especially if there are a lot of tests on a project.

![Vitest UI Dashboard](/assets/2023-12-05/04-ui-dashboard.png)

_Fig. 4. Vitest UI Dashboard_

![Vitest UI Report](/assets/2023-12-05/05-ui-report.png)

_Fig. 5. Vitest UI Report_

![Vitest UI Module Graph tab](/assets/2023-12-05/06-ui-module-graph.png)

_Fig. 6. Vitest UI Module Graph tab allows visually discover the dependencies of the test_

Unfortunately, the UI interface works only with a default «always on watch mode» of Virest. You will not be able to use it with such kind of running: `vitest run --pool=forks`. However, it is possible to represent a report of a single test run as an HTML report with the same interface.

> NOTE: UI mode is optional (thank god); you will need to install it from a separate package (`@vitest/ui`).

- Vitest has had an [enormous growth of popularity in the JavaScript community](https://2022.stateofjs.com/en-US/libraries/testing/) in just two years. With [the recent release of the major version](https://github.com/vitest-dev/vitest/releases/tag/v1.0.0), its usage will only increase.

Read more:

- All [Vitest Features](https://vitest.dev/guide/features.html);
- [The Road to Vitest 1.0 \| ViteConf 2023](https://www.youtube.com/watch?v=Zq7EyP8VpYw) (talk);
- [Vitest: Blazing Fast Unit Test Framework](https://lo-victoria.com/vitest-blazing-fast-unit-test-framework);
- [Unit Testing with Vitest: A Powerful Jest Alternative](https://javascript.plainenglish.io/unit-testing-with-vitest-a-powerful-jest-alternative-3cf4b6ddf863).

## Why should you choose or switch to Node.js as an HTTP client?

- Why choose something else if you already have it? That is a rhetorical question. Removing a third-party library from dependencies of your project reduces the number of dependencies.
- External HTTP clients can provide some additional features, like retries, autotransforming of request and response data, advanced timings, and so on. In most of the cases of simple HTTP request testing, these features are excess.
- [Axios](https://github.com/axios/axios) is based on [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest), which is an outdated technology ([see the differences](https://stackoverflow.com/a/35552420)).
- [Got](https://github.com/sindresorhus/got) may force to [reconfigure your TypeScript project to work](https://github.com/sindresorhus/got/releases/tag/v13.0.0).
- [node-fetch](https://github.com/node-fetch/node-fetch) and [ky](https://github.com/sindresorhus/ky) are already based on Fetch API. Using [Node.js’s fetch()](https://nodejs.org/dist/latest-v21.x/docs/api/globals.html#fetch) makes autotests a little bit low-level and more robust because you have fewer intermediate links between requests and tests.

The only benefit of third-party clients is their syntax. For example, some of them allow to pass JSON as requests’ parameters, but `fetch()` requires to pass parameters in the [URL](https://developer.mozilla.org/en-US/docs/Web/API/Request/url).

Example of Got request:

```JavaScript
const urlQuery = {
  api_key: 'DEMO_KEY',
  feedtype: 'json',
  ver: '1.0',
};

<...>

response = await got.get('https://api.nasa.gov/insight_weather/, {
  searchParams: urlQuery,
});
```

Example of `fetch()` request — you have to use [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) to convert JSON into URL-like string:

```JavaScript
const urlQuery = {
  api_key: 'DEMO_KEY',
  feedtype: 'json',
  ver: '1.0',
};

<...>

const queryParams = new URLSearchParams(urlQuery).toString();
response = await fetch(`https://api.nasa.gov/insight_weather/?${queryParams}`);
```

Read more:

- [Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch);
- [Fetch API in Node.js](https://www.atatus.com/blog/fetch-api-in-nodejs/);
- [How to Make HTTP Requests in Node.js With Fetch API](https://oxylabs.io/blog/nodejs-fetch-api) (outdated).

## And finally, testing…

That is an easy part if you understand the principles:

- Vitest works as a test-runner and assertion library (exactly what it is intended to do);
- `fetch()` makes HTTP requests to pass response data to Vitest for checking.

**For the full code example, [see this GitHub repository](https://github.com/adequatica/api-vitesting)** (there are linters and additional infrastructure around tests that are not mentioned in the article).

> NOTE: The following code examples assume the use of TypeScript.

To set up the project, you only need to add the latest Vitest package and Node >=21 (that is important because in previous versions, `fetch()` does not work without the experimental flag) into `package.json`:

```JavaScript
"engines": {
  "node": ">=21"
},
"type": "module",
"dependencies": {
  "vitest": "^1.0.0"
}
```

Where:

- `"type": "module"` needs to be pinpointed using ESM [instead of CJS Node API](https://vitejs.dev/guide/troubleshooting.html#vite-cjs-node-api-deprecated).

To set up commands for running tests, you need to add scripts into `package.json`:

```
"scripts": {
  "test": "vitest run --pool=forks"
}
```

By default (`vitest` command), Vitest runs in the watch mode, waits for file changes, and reruns tests for each change. This mode is excess for API tests — a single run is enough ([`vitest run`](https://vitest.dev/guide/cli.html#vitest-run)).

Now you can run tests using the following command: `npm test`

To run a single test, just pass a part of a test-file name after the command: `npm test foobar` — will run `foobar.test.ts` test file.

To set up Vitest’s config, you need to create `vitest.config.ts` file with the following content:

```JavaScript
import {defineConfig} from 'vitest/config';

export default defineConfig({
  test: {},
});
```

Now you finish with infrastructure and can start writing tests. The best way is to store tests in the `/tests` directory with `.test.ts` filetype.

### Test Example

Let’s request a sample [NASA API](https://api.nasa.gov/) handler and execute some basic checks.

```JavaScript
import { beforeAll, describe, expect, expectTypeOf, test } from 'vitest';

const BEFORE_ALL_TIMEOUT = 30000; // 30 sec

describe('Request Earth Polychromatic Imaging Camera', () => {
  let response: Response;
  let body: Array<{ [key: string]: unknown }>;

  beforeAll(async () => {
    response = await fetch(
      'https://api.nasa.gov/EPIC/api/natural?api_key=DEMO_KEY',
    );
    body = await response.json();
  }, BEFORE_ALL_TIMEOUT);

  test('Should have response status 200', () => {
    expect(response.status).toBe(200);
  });

  test('Should have content-type', () => {
    expect(response.headers.get('Content-Type')).toBe('application/json');
  });

  test('Should have array in the body', () => {
    expectTypeOf(body).toBeArray();
  });

  test('The first item in array should contain EPIC in caption key', () => {
    expect(body[0].caption).to.have.string('EPIC');
  });
});
```

Where:

- Import contains all involved [Vitest’s functions](https://vitest.dev/api/);
- `BEFORE_ALL_TIMEOUT` constant will be used in [`beforeAll`](https://vitest.dev/api/#beforeall) function to extend the termination time of the request’s execution;
- `beforeAll` contains `fetch()` HTTP request. `beforeAll` [must not contain any assertions](https://adequatica.medium.com/principles-of-writing-automated-tests-a2b72218264c#6da4). Response data and response body are stored in variables for subsequent checks;
- «Should have response status 200» test checks response code;
- «Should have content-type» test checks on one of the headers;
- «Should have array in the body» checks the type of the body’s root;
- «The first item in array should contain EPIC in caption key» test checks an arbitrary key in the body with [Chai](https://www.chaijs.com/api/bdd/).

---

Read more about alternative ways of testing API with Node.js:

- [Testing API with TypeScript, Jest, and Got](https://github.com/adequatica/api-testing) (code example: Jest + Got);
- [Writing & organizing Node.js API Tests the right way](https://dev.to/larswaechter/writing-organizing-nodejs-api-tests-the-right-way-34lb) (Jest + SuperTest + Chai);
- [How To Test Your Rest API With Jest And SuperTest](https://faun.pub/how-to-test-your-rest-api-with-jest-and-supertest-i-196bc84e6c5f) (Jest + SuperTest);
- [Beyond API testing with Jest](https://circleci.com/blog/api-testing-with-jest/) (Jest + SuperTest);
- [How to Test an API in Node.js](https://www.educative.io/answers/how-to-test-an-api-in-nodejs) (Mocha + SuperTest);
- [How to Test APIs Like a Pro](https://www.testim.io/blog/supertest-how-to-test-apis-like-a-pro/) (SuperTest);
- [Test a Node RESTful API with Mocha and Chai](https://www.digitalocean.com/community/tutorials/test-a-node-restful-api-with-mocha-and-chai#conclusion) (Mocha + Chai).

In the list above, there are a few examples of using [SuperTest](https://github.com/ladjs/supertest) as an HTTP client, but I am very skeptical about it. Supertest provides a limited set of assertions stuck to HTTP requests ([superagent](https://github.com/ladjs/superagent) as HTTP library) with `describe`/`it` syntax from Mocha.

- Mocha + SuterTest is a redundant bundle because SuperTest is already based on Mocha;
- Jest + SuperTest is a redundant bundle, too. Why use Jest for running tests, while SuperTest can do it through Mocha? Why use SuperTest only for HTTP requests, when it can be taken only superagent?

Each tool should have a reasonable usage and not duplicate features of its neighbors by package.

Copy @ [Medium](https://adequatica.medium.com/api-testing-with-vitest-391697942527)
