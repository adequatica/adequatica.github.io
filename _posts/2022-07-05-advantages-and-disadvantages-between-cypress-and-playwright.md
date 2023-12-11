---
layout: post
title: "Advantages and Disadvantages between Cypress and Playwright"
date: 2022-07-05 04:53:20 +0400
tags: cypress playwright testing
---

As a test engineer, I have been using WebdriverIO with Selenium WebDriver for frontend automation for a few years.

![Cypress tree and a playwrighter](/assets/2022-07-05/00-cypress-tree-playwriter.jpg)

_Cypress tree and a playwrighter_

Recently I got an opportunity to move away from Selenium and therefore had to compare alternatives (inside my usual stack of TypeScript and Node.js):

- [Cypress](https://www.cypress.io/), as the most hyped automation tool;
- [Playwright](https://playwright.dev/), as a more high-level tool close to the browser.

[Puppeteer](https://pptr.dev/) did not hit my shortlist because it is only an API ([Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)) to control Chromium, and it needs an additional testing framework (Jest, Mocha, or something else). [WebdriverIO](https://webdriver.io/) can [also work by DevTools Protocol](https://webdriver.io/docs/automationProtocols#devtools-protocol), but I wanted to try something different.

Disclaimer: all listed pros and cons are subjective and based on my own experience.

---

## Cypress

### Advantages:

1. **Luxury dev experience!** Running and debugging tests [in the app](https://docs.cypress.io/guides/core-concepts/cypress-app) is extremely convenient.
2. **Brilliant documentation!** Cypress has the best documentation I have ever met. During the [installation process](https://docs.cypress.io/guides/getting-started/installing-cypress) and writing the first tests, I used only documentation and examples from it.
3. All in one instrument. Test runner, assertion library, HTTP request library, and many more are already built in Cypress. No need to choose different tools for building testing infrastructure. It also supports [plugins](https://docs.cypress.io/plugins/directory) for more functionality.
4. [Chai](https://www.chaijs.com/api/bdd/) as an assertion library (unlike Playwright with humble [Jest expects](https://jestjs.io/docs/expect)).
5. [Getting elements](https://docs.cypress.io/api/commands/get) and interactions with them have a built-in «wait until» for the actionable state of the element.
6. Easy access to network activity. No need for any external proxies [to stub network requests and responses](https://docs.cypress.io/api/commands/intercept) — it is also built in Cypress.
7. Ready-made support of [custom commands](https://docs.cypress.io/api/cypress-api/custom-commands) (and even types for your custom commands if you use TypeScript).
8. Auto-making [screenshots on failures](https://docs.cypress.io/guides/references/configuration#Screenshots).

### Curious features:

1. There is an [Electron](https://www.electronjs.org/) as the default «browser». Despite the fact that the Electron is based on Chromium, it is not the same environment as a pure browser. For example, my tested web app started behaving unstable and sending console errors, so I had to switch to Chromium — lucky that Cypress supports cross-browser testing in [Firefox and Chrome-like browsers](https://docs.cypress.io/guides/guides/launching-browsers).
2. It watches for [uncaught exceptions from your web app](https://docs.cypress.io/guides/references/error-messages#Uncaught-exceptions-from-your-application) during the test run. It could be useful if your project has zero tolerance for console errors.
3. Tests can work only in one tab. You can not test muli-tab scenarios in Cypress.
4. Cypress uses its own [environment variables](https://docs.cypress.io/guides/guides/environment-variables). If you are used to using Node.js’s `process.env` variables, you have to switch to Cypress’ ones and include each in the config file.

### Disadvantages:

1. Cypress includes tons of dependencies. Your node_modules directory will get heavier by several tens of pounds.
2. It is forbidden to use `console.log` for debugging. You have to use `cy.log`, which [accepts only a string](https://docs.cypress.io/api/commands/log).
3. All the tests are somehow merged during the test run. It means that you can not use variables with the same names in different test files.
4. Cypress did not start from the first time inside CI — I got [a weird error](https://on.cypress.io/not-installed-ci-error) in TeamCity.

---

## Playwright

### Advantages:

1. [Inspector](https://playwright.dev/docs/inspector) — a GUI tool for debugging tests. Not as good as Cypress has, but still worthwhile.
2. Fast work in headless mode (I did not make accurate performance measurements, but it just feels faster than Cypress).
3. [Auto-waiting](https://playwright.dev/docs/actionability) of elements before performing some actions.
4. [Visual regression testing](https://playwright.dev/docs/test-snapshots) is working out of the box (in Cypress, you have to use a plugin).
5. Detailed HTML report, which could be useful both at local run and in CI.
6. Auto-making [screenshots](https://playwright.dev/docs/test-configuration#automatic-screenshots), [videos](https://playwright.dev/docs/test-configuration#record-video), and traces on failures — all of them are great for debugging failed tests, especially in [Trace Viewer](https://playwright.dev/docs/trace-viewer).

### Curious or controversial features:

1. [Test generator](https://playwright.dev/docs/codegen) — a GUI tool for record tests. Of course, the generated results can not be used for real tests: chosen selectors are odd and need to be replaced, and expects should be added manually later. But it could be handy for creating rough drafts for simple tests.
2. There is no support for custom commands, as Cypress and WebdriverIO have. Playwright pushes you to use [Page Object Model](https://playwright.dev/docs/test-pom) instead of creating custom syntactic sugar around your testing infrastructure.
3. Most of [page](https://playwright.dev/docs/api/class-page)’s and [locator](https://playwright.dev/docs/locators)’s methods work only with the first element on the page. You have to seek out special methods that allow you to work with multiple elements. Playwright pushes you to choose extremely unambiguous selectors, while Cypress or WebdriverIO do not limit you this way.
4. ~~Playwright does not allow passing an arbitrary header to the test page~~ (sorry, I figured it out, [it is possible](https://playwright.dev/docs/api/class-page#page-set-extra-http-headers)). But anyway, I could not pass a single header `Cookies` with multiple cookies, I had to [set each cookie](https://playwright.dev/docs/api/class-browsercontext#browser-context-add-cookies) one by one. The same strange thing with localStorage: I could not pass only a single localStorage to [the new browser’s context](https://playwright.dev/docs/api/class-browser#browser-new-context), because setting a storage state implies the object of both mandatory cookies and localStorage (and not just one thing). On the one hand, this is strictly right, but on the other hand, it is an excessive restriction.

### Disadvantages:

1. Сonfusing documentation. Without a doubt a Playwright’s documentation is good, but looking for lore takes time, and examples are quite poor. After the [installation process](https://playwright.dev/docs/intro) I had to use Google and Stackoverflow to figure out some of the details.
2. Despite the fact that Playwright allows [API testing](https://playwright.dev/docs/test-api-testing), its [request method](https://playwright.dev/docs/api/class-apirequest) is not advanced enough. For example, it does not support disabling follow redirects ([but Cypress does](https://docs.cypress.io/api/commands/request)).
3. Screenshots, videos, and traces on failures are made inside one [test()](https://playwright.dev/docs/api/class-test) function. It means that if you have a few tests inside `test.describe()`, you will get videos and traces for each nested test, but not for a whole root test function ([steps](https://playwright.dev/docs/api/class-test#test-step) have the same behavior). It is a shame that so many artifacts can not be assembled into one piece.
4. TeamCity is not supported out of the box — you have to write your own reporter for CI builds.

---

Playwright is a more technical tool, which works closer to the browser and has precise commands. Cypress is a more flexible tool with an expectable API for testers, which allows to develop simple tests easy and fast enough.

Copy @ [Medium](https://adequatica.medium.com/advantages-and-disadvantages-between-cypress-and-playwright-6b29e3989108)
