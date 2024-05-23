---
layout: post
title: "Selection Criteria of JavaScript Test Framework"
date: 2022-09-23 05:46:25 +0300
tags: automation javascript testing
---

When a frontend team starts building a testing infrastructure, there is always a debate about the choice of testing tools.

![Selection Criteria of JavaScript Test Framework](/assets/2022-09-23/00-cover.png)

There are dozens of tools for testing web applications on **JavaScript, TypeScript, and Node.js**.

- [Ava](https://github.com/avajs/ava) — test runner;
- [CodeceptJS](https://codecept.io/) — end-to-end testing framework;
- [Cypress](https://www.cypress.io/) — end-to-end browser testing framework;
- [Hermione](https://github.com/gemini-testing/hermione) — test runner for browser testing based on Mocha and WebdriverIO;
- [Jasmine](https://jasmine.github.io/) — testing framework;
- [Jest](https://jestjs.io/) — testing framework;
- [Karma](https://karma-runner.github.io/) — test runner;
- [Mocha](https://mochajs.org/) — test runner;
- [Nightwatch](https://nightwatchjs.org/) — end-to-end testing framework;
- [Node Tap](https://node-tap.org/) — a Test-Anything-Protocol library (mostly for any kind of unit tests);
- [Playwright](https://playwright.dev/) — end-to-end testing framework;
- [Protractor](https://www.protractortest.org/) — deprecated end-to-end test framework for Angular applications;
- [Puppeteer](https://pptr.dev/) — headless Chrome Node.js API (actually, it is not a testing tool/framework);
- [SelenideJS](https://github.com/KnowledgeExpert/selenidejs) — browser testing framework;
- [TestCafe](https://testcafe.io/) — end-to-end browser testing framework;
- [Vitest](https://vitest.dev/) — Jest-compatible modern testing framework;
- [WebdriverIO](https://webdriver.io/) — end-to-end testing framework.

Some of these tools can run out-of-the-box and consist of a test runner, assertion library, and much more. Some are dying, though, and you need to choose wisely. Some can be used as building blocks, for instance, Mocha as a test runner + Puppeteer as an API for a browser + [Chai](https://www.chaijs.com/) as an assertion library + [Allure](https://github.com/allure-framework/allure-js) as a test reporter.

Anyway, there should be a list of criteria by which to make a choice in favor of a particular tool. The last time when I took part in choosing a test framework for a new project, we were guided by the following requirements:

1. Do you need unit or integration/end-to-end testing?
2. Browser support (including mobile browsers);
3. Documentation;
4. Community (articles, video tutorials, stackoverflow answers);
5. Support by developers (reactions on GitHub issues) and frequency of updates;
6. Parallelization (the power of test runner);
7. Retries (the power of test runner);
8. Skips and conditional skips (the power of test runner);
9. Assertions (the power of assertion library);
10. Reports (variety of console and HTML reports);
11. WebSocket support;
12. Selenium ([WebDriver](https://w3c.github.io/webdriver/)) support? [DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) support? Or both?
13. Working with tabs (for example, [Cypress does not support that](https://docs.cypress.io/guides/references/trade-offs#Multiple-tabs));
14. Mocks/stubs;
15. Proxy HTTP requests/responses;
16. Support HTTP API requests;
17. Setting cookies, localStorage, and user-agents;
18. Visual/screenshot testing;
19. Debug mode;
20. Headless and headed modes (for browsers);
21. Integration with IDE (for example, [Playwright can run test in VSCode](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright));
22. Run on localhost;
23. Run in CI/CD systems;
24. Run a single test;
25. Filter tests (ability to grep a bunch of tests);
26. Logging the test execution process;
27. Making screenshots and videos in case of failure (for example, [Playwright can even record traces](https://playwright.dev/docs/trace-viewer));
28. Distribution size (the number of dependencies in package.json);
29. Maintainability (and how quickly new team members will get used to the project);
30. Open Source and/or [license](https://opensource.org/licenses).

---

In recent times (according to my own observations), the short list of test frameworks for browser testing usually looks like this:

- Playwright;
- Cypress;
- WebdriverIO.

And for unit or API testing:

- Jest;
- Mocha;
- And an additional HTTP request library for Node.js, like [GOT](https://github.com/sindresorhus/got) or [Axios](https://axios-http.com/).

> UPDATE for 2023 year:
>
> - Vitest is a preferable option for unit testing;
> - [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) starts work out of the box from Node.js [21](https://nodejs.org/en/blog/announcements/v21-release-announce), so you may skip adding a library for HTTP requests ([see an example](https://adequatica.github.io/2023/12/04/api-testing-with-vitest.html)).

---

After all, criteria and tools listed above can be expanded depending on the context of your project.

Moreover, the stated criteria can be used for another (non JS) testing stack.

Copy @ [Medium](https://adequatica.medium.com/selection-criteria-of-test-framework-d091a18c3c4)
