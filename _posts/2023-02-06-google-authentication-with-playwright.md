---
layout: post
title: "Google Authentication with Playwright"
date: 2023-02-06 07:40:05 +0100
tags: authentication automation playwright
---

Sometimes, as a test engineer, you need to log in to Google (or other third-party authentication providers) to give access to your website for autotests.

In my recent project, I had to test a web app under an authenticated user, but the only authentication provider was Google. The problems with third-party authentication providers for autotests are:

- Complicated API;
- Auth providers have anti-robot detectors.

> UPDATE, February 2023: In recent changes to the Playwright documentation, the [recommended authentication concept](https://playwright.dev/docs/auth#core-concepts) has been changed, but the approach described in the article still works.

> UPDATE, April 2025: This approach **may be stale** cause used packages [playwright-extra](https://www.npmjs.com/package/playwright-extra) and [puppeteer-extra-plugin-stealth](https://www.npmjs.com/package/puppeteer-extra-plugin-stealth) maintenance were stopped in March 2023. Anyway, you can give it a try for your personal needs.

### Complicated API

It is always faster for autotests to authenticate through API than through UI, but it is not that simple with Google. Google requires you to have a project on [Google Cloud Console](https://console.cloud.google.com/getting-started) to get a Client ID and Client secret as credentials — [this is doable](https://developers.google.com/identity/protocols/oauth2), but if your web app works with Google Workspace (as mine does), this significantly complicates the problem.

Eventually, I decided it would be easier to authenticate by user’s login and password through Sign in form.

### Auth providers have anti-robot detectors

Firstly, Playwright, Puppeteer, or Selenium autotests can be detected by indirect signs, and your autotests may suddenly be asked to solve the captcha.

Secondly, your test user may be identified in robotness activity and may be banned.

I had to deal with both cases and worked out a couple of rules:

- **Authenticate as rarely as possible.** The less activity from your test user, the less [bot detection algorithms](https://www.sciencedirect.com/science/article/pii/S0950705121003373) will pay attention to it;
- **Try to imitate the behavior of an ordinary (human) user.** The most common practices for this are: add[ User Agent](https://playwright.dev/docs/emulation#user-agent) header as a real browser (pretend not to be a headless chrome) and add [timeouts](https://playwright.dev/docs/api/class-page#page-wait-for-timeout) for 0,3–1 sec between test commands — it is a [test automation anti-pattern](https://dev.to/checkly/avoiding-hard-waits-in-playwright-and-puppeteer-272), but not this time. Or you can use the «stealth» plugin for Puppeteer or Playwright to prevent robotness detection.

According to these rules I used [puppeteer-extra-plugin-stealth](https://github.com/berstend/puppeteer-extra/tree/master/packages/puppeteer-extra-plugin-stealth) for login on Sign in form and reused signed in state during a test run (it means that I authenticate only one time for all tests).

## Step-by-step authentication with Playwright

### Config Configuration

Authentication should execute before tests will start — it is regulated by [Playwright’s config](https://playwright.dev/docs/test-advanced#configuration-object). You should add `globalSetup` option to determine a file (in my case, it is `./lib/global-setup.ts`) with the [function which will be run once before all the tests](https://playwright.dev/docs/test-advanced#global-setup-and-teardown).

Global setup should contain not only authentication, but also saves a signed in state (current cookies and local storage snapshot). For reusing signed in state in your tests, you should add `storageState` option to determine a file (in my case, it is `./setup/storage-state.json`) where this state stores.

```JavaScript
import { PlaywrightTestConfig, devices } from '@playwright/test';

const config: PlaywrightTestConfig = {
  …
  globalSetup: process.env.SKIP_AUTH ? '' : './lib/global-setup',
  use: {
    …
    storageState: './setup/storage-state.json',
  },
  …
};

export default config;
```

I added the CLI option `SKIP_AUTH` to skip global setup with authentication to reduce the number of calls of Sing in form during debugging (it will work if you already have a file of a signed in state).

### Global Setup

Inside the global setup function, you should:

1. Initiate a new browser page via `playwright-extra` with `puppeteer-extra-plugin-stealth` ([playwright-extra](https://github.com/berstend/puppeteer-extra/tree/master/packages/playwright-extra) needs to extend normal Playwright with the «stealth» plugin);
2. Navigate to the login page on a tested site;
3. Open Google Sign in form;
4. Fill in the user’s credentials;
5. Open (wait for redirect) back a tested site;
6. Save signed in state by [storageState](https://playwright.dev/docs/api/class-browsercontext#browser-context-storage-state) method;
7. Close the browser.

During CI runs of this function I noticed that it was failing from time to time — I found out that Google can show different kinds of Sign in form. Because of this, I had to add `if` to switch between to different versions. Moreover, after each input, you need to click [Next]:

![Google Sign in form for email (login) and for password](/assets/2023-02-06/00-google-sign-in.png)

_Google Sign in form for email (login) and for password_

```JavaScript
// Credentials in the format: username@gmail.com
const userLogin = process.env.USER_LOGIN || '';
const userPass = process.env.USER_PASS || '';

// playwright-extra is a drop-in replacement for playwright,
// it augments the installed playwright with plugin functionality
import { chromium } from 'playwright-extra';
// Load the stealth plugin and use defaults (all tricks to hide playwright usage)
import stealth from 'puppeteer-extra-plugin-stealth';

import { baseURL } from '../playwright.config';

// Add the plugin to playwright
chromium.use(stealth);

async function globalSetup(): Promise<void> {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();

  // Open log in page on tested site
  await page.goto(`${baseURL}/user/login`);
  await page.getByText('Google').click();
  // Click redirects page to Google auth form,
  // parse https://accounts.google.com/ page
  const html = await page.locator('body').innerHTML();

  // Determine type of Google sign in form
  if (html.includes('aria-label="Google"')) {
    // Old Google sign in form
    await page.fill('#Email', userLogin);
    await page.locator('#next').click();
    await page.fill('#password', userPass);
    await page.locator('#submit').click();
  } else {
    // New Google sign in form
    await page.fill('input[type="email"]', userLogin);
    await page.locator('#identifierNext >> button').click();
    await page.fill('#password >> input[type="password"]', userPass);
    await page.locator('button >> nth=1').click();
  }

  // Wait for redirect back to tested site after authentication
  await page.waitForURL(baseURL);
  // Save signed in state
  await page.context().storageState({ path: './setup/storage-state.json' });

  await browser.close();
}

export default globalSetup;
```

Now, when you run your tests like this:

```
USER_LOGIN='{username@gmail.com}' USER_PASS='{password}' npm run test
```

you will notice a creation of `storage-state.json` file. And you do not need to care about the precondition step for authentication in your tests, because they will start already authenticated thanks to specified `storageState` in config.

### And one last thing

Do not forget to untrack your state file in Git for security reasons — add it to `.gitignore`!

---

See the [code on GitHub](https://github.com/adequatica/ui-testing-auth) of the implementation of the authentication process described above.

Further reading about alternative realizations:

- [Authentication](https://playwright.dev/docs/auth) via Playwright;
- [Fast and easy authentication with Playwright](https://timdeschryver.dev/blog/fast-and-easy-authentication-with-playwright);
- [Automatically sign in with Google using Puppeteer](https://marian-caikovski.medium.com/automatically-sign-in-with-google-using-puppeteer-cc2cc656da1c);
- [Google Authentication](https://docs.cypress.io/guides/end-to-end-testing/google-authentication) via Cypress;
- [Google Sign in with Cypress](https://filiphric.com/google-sign-in-with-cypress).

Copy @ [Medium](https://adequatica.medium.com/google-authentication-with-playwright-8233b207b71a)
