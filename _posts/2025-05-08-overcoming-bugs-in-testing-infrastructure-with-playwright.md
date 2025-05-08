---
layout: post
title: "Overcoming Bugs in Testing Infrastructure with Playwright"
date: 2025-05-08 10:52:28 +0200
tags: testing
---

Sometimes, as a test engineer, you meet tasks where problems are around every corner.

In this post, I’d like to share a real-world example of how a seemingly simple and straightforward task turned into a battle with our testing infrastructure.

In my current project, we ran end-to-end (e2e) tests on a frontend application in isolation, and all backend requests were mocked [1]. At some point, we needed to strengthen our test coverage by running full-fledged e2e tests against a fully operational test environment, with actual requests to the backend. That sounded even simpler: just open the site via Playwright and test it [2].

But our application came with a few complications:

1. Two APIs — HTTP and WebSocket;
2. Authentication, followed by authorization via headers (cookies);
3. A third-party component for the map required WebGL support.

![Application](/assets/2025-05-08/01-application.png)

_Fig. 1. Application_

We had no special browser requirements, so we used the Chromium version (134.0.6998.35) bundled with Playwright (v1.51.1).

### First bug: extraHTTPHeaders does not work with WebSockets on Chromium

Adding [authentication](https://playwright.dev/docs/auth) to the tests was not particularly difficult. Before running the tests, we used the API to retrieve the authentication token and passed it as a Cookie header by [`extraHTTPHeaders`](https://playwright.dev/docs/api/class-testoptions#test-options-extra-http-headers):

```javascript
extraHTTPHeaders: {
  Cookie: `session=${authToken}`,
},
```

On the very first test run, things mostly worked. The data from HTTP API responses came through just fine, but data from the WebSockets was missing. WebSocket connection failed due to authentication:

![Console log error: WebSocket connection failed due to authentication](/assets/2025-05-08/02-wss-failed.png)

_Fig. 2. Console log error: WebSocket connection failed due to authentication_

Our first thought was to double-check the application code, but nothing there pointed to any issues with authentication headers. We turned to the Playwright issue tracker and, sure enough, #28948 [extraHTTPHeaders does not work with WebSockets on Chromium](https://github.com/microsoft/playwright/issues/28948).

We quickly found a workaround — switch to Firefox (as I mentioned earlier, we had no strict browser requirements). And voilà, with Firefox, the WebSocket connection started working, and all the expected data came through.

### Second bug: Firefox does not support WebGL in headless mode

But now, our map component has started crashing. Debugging this issue turned out to be trickier cause it only occurred in the CI environment.

![Map component failed to load (crashed)](/assets/2025-05-08/03-failed-to-load.png)

_Fig. 3. Map component failed to load (crashed)_

Like the first bug, we reviewed our application’s code and found nothing suspicious. So we turned to the bug trackers and, sure enough, #1375585 [Firefox does not support WebGL in headless mode](https://bugzilla.mozilla.org/show_bug.cgi?id=1375585). And since we were running our tests in headless mode on CI, this became a blocker for the map component ([Mapbox GL JS](https://docs.mapbox.com/mapbox-gl-js/guides/)), which works in browsers that support [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API).

The workaround in this case was to run the tests in Firefox with [`headless: false`](https://playwright.dev/docs/api/class-testoptions#test-options-headless) option, using the [`xvfb-run`](https://playwright.dev/docs/ci#running-headed) prefix, allowing us to simulate a display environment even when there is no physical monitor:

```
xvfb-run npm run playwright
```

In the end, instead of having a unified test infrastructure for running both mocked and full e2e tests in the same environment, we had to use different browsers and different launch commands. It is still great that we could find workarounds and implement automated testing.

Of course, features like WebSockets and WebGL are not the most common on typical websites, but when they are part of the stack, they can introduce surprising complexity to your testing setup.

References:

1. [API Contract Testing on Frontend with Playwright](https://adequatica.github.io/2023/12/25/api-contract-testing-on-frontend-with-playwright.html);
2. [Pros and Cons of the Ways of End-to-End Automated Testing in CI](https://adequatica.github.io/2023/12/04/pros-and-cons-of-the-ways-of-end-to-end-automated-testing-in-ci.html).

Copy @ [Medium](https://medium.com/@adequatica/overcoming-bugs-in-testing-infrastructure-with-playwright-5fbce14e11bc)
