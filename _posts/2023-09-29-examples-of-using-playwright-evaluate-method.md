---
layout: post
title: 'Examples of Using Playwright Evaluate Method'
date: 2023-09-29 07:32:44 +0100
tags: automation playwright
---

Sometimes, as a test engineer, you wish to execute unintended scripts in your autotests to set preconditions or check unusual functionality.

![Examples of Using Playwright Evaluate Method](/assets/2023-09-29/00-cover.jpg)

[Playwright page.evaluate() method](https://playwright.dev/docs/api/class-page#page-evaluate) allows to execute code within the context of a web page. If the function returns a Promise, the `evaluate()` method will wait for the Promise to resolve before returning. In other words, it allows to run custom JavaScript functions and manipulate the DOM of a web page — this implies the use of [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API) from a test’s code.

In a perfect world, you should not have to interfere with a testing page (this means, you should use only built-in methods of your testing framework), but in the real world, some web app functionality can be checked only by custom scripts:

- Interact with the clipboard;
- Run anything in the console;
- Interact with localStorage;
- Modify elements;
- Get web page data;
- Execute custom JavaScript functions.

---

## Interact with the clipboard

It is useful when you click on a [Copy] link or a button and need to check that some text was copied in the clipboard ([that is, read from the clipboard](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/readText)):

```JavaScript
test('Should copy text into the clipboard', async () => {
  const pageObjectModel = new PageObjectModel(page);
  await pageObjectModel.clickOnCopyButton();

  const clipboardText = await page.evaluate('navigator.clipboard.readText()');
  expect(clipboardText, 'Should have copied text in the clipboard').toBe('Foo bar');
});
```

Note, for this action, you need to update your Playwright config — [grant permission to the browser context to read from the clipboard](https://playwright.dev/docs/api/class-browsercontext#browser-context-grant-permissions): `use.permissions: ['clipboard-read'].`

Besides reading, you can also [write to the browser’s clipboard](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText):

```JavaScript
await page.evaluate('navigator.clipboard.writeText("Foo bar")');
```

Note, for this action, you need to grant permission too: `use.permissions: ['clipboard-write'].`

Read more:

- [Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API);
- [How do I access the browser clipboard with Playwright](https://playwrightsolutions.com/how-do-i-access-the-browser-clipboard-with-playwright/).

## Run anything in the console

From the examples above about the clipboard, you may notice that the syntax of the evaluate method is simple: `page.evaluate('foobar');`

So you can use it as an executor of anything in the **browser’s Console command line**, like:

```JavaScript
await page.evaluate('window.alert("Hello world!");');
```

## Interact with localStorage

It is useful when you need to set the state for a web application:

```JavaScript
await page.evaluate(() => {
  localStorage.setItem('keyName', 'keyValue');
});
```

Using the syntax of [Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage) interface of the [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API), you can get the value of certain localStorage items, remove items, or even delete all items out of the storage.

To get all localStorage of the origin:

```JavaScript
await page.evaluate('window.localStorage');
```

Read more:

- [Window: localStorage property](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage);
- [Setting state using cookies](https://www.checklyhq.com/learn/headless/managing-cookies/#localstorage-and-sessionstorage) — this article has an example of the practical application of `page.evaluate('window.localStorage')`.

## Modify elements

The most desired way of using `evaluate()` is for modifying web page elements. It may be to set some part of the page into the desired state by adding/changing/deleting elements’ attributes, or even delete the whole elements (if you need to remove pop-ups of iframes from the page).

- Updating element’s attribute by [setAttribute() method](https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute):

```JavaScript
await page.evaluate(() => {
  // Get element inside evaluate() mathod
  const selector = document.querySelector('input');

  // This IF is only aimed to overcome «is possibly 'null'» TypeScript error
  if (selector) {
    selector.setAttribute('placeholder', 'Foo bar');
  }
});
```

Or:

```JavaScript
await test.step('Updating element attribute', async () => {
  const selector = await page.locator('[class^=Input_]');
  await selector.evaluate(node => node.setAttribute('placeholder', 'Foo bar'));
```

- Removing element by [remove() method](https://developer.mozilla.org/en-US/docs/Web/API/Element/remove):

```JavaScript
await page.evaluate(() => {
  const selector = document.querySelector('.class_name');

  if (selector) {
    selector.remove();
  }
});
```

Or:

```JavaScript
await test.step('Remove element attribute', async () => {
  const selector = await page.locator('.class_name');
  await selector.evaluate(node => node.removeAttribute('readonly'));
```

Read more:

- [Document: querySelector() method](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector);
- [Element](https://developer.mozilla.org/en-US/docs/Web/API/Element) class;
- [Add and Remove an HTML Attribute using Playwright](https://testerops.com/add-and-remove-an-html-attribute-using-playwright/).

## Get web page data

From the paragraphs above, `window.*` and `document.*` are properties of the [Window](https://developer.mozilla.org/en-US/docs/Web/API/Window) and the [Document](https://developer.mozilla.org/en-US/docs/Web/API/Document) interfaces, respectively. In the same way, you can use its other properties, for example, to get various parameters of the testing page:

```JavaScript
const getPageParams = await page.evaluate(() => {
  return {
    devicePixelRatio: window.devicePixelRatio,
    width: document.documentElement.clientWidth,
    height: document.documentElement.clientHeight,
  };
});

console.log(getPageParams);
```

This data can be useful for debugging on CI agents:

```
{
  devicePixelRatio: 1,
  width: 1280,
  height: 720
}
```

## Execute custom JavaScript functions

Last but not least, you can use `evaluate()` method as an environment for running any scripts:

```JavaScript
const customFunction = await page.evaluate(() => {
  const today = new Date();
  const unixtime = today.valueOf();

  return Math.sqrt(unixtime);
});

console.log(customFunction); // prints 41179.69533641549
```

Read more:

- [Evaluate](https://playwright.dev/docs/api/class-page#page-evaluate);
- [Enhancing Web Testing with Playwright’s Evaluate Method](https://ceroshjacob.medium.com/enhancing-web-testing-with-playwrights-evaluate-method-73615d4ffc9e).

Copy @ [Medium](https://adequatica.medium.com/simple-examples-of-using-playwright-evaluate-method-9b00d01cadc1)
