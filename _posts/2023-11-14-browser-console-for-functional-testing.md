---
layout: post
title: "Browser Console for Functional Testing"
date: 2023-11-14 06:56:50 +0100
tags: testing devtools
---

As a test engineer, you have to use DevTools for efficient testing of UI and frontend, and the browser’s Console is an essential part of it.

![Firefox Web Console](/assets/2023-11-14/00-cover.jpg)

Browser DevTools provides invaluable assistance for the functional testing of web services. At a minimum, opened Browser Console will display errors on a testing web page.

The topic of this article is the **Console’s command line** — a tool that allows to evaluate JavaScript inside a browser. This means it allows to run scripts, modify DOM, and control the browser. That is why some popular websites show warning messages in the Console for ordinary users.

![Stop](/assets/2023-11-14/01-stop.png)

_Fig. 1. Facebook.com Console Logs_

The capabilities of the Console’s command line have several use cases, which will be extremely useful for test engineers.

1. Run JavaScript in the Console;
2. Find elements, evaluate and validate selectors;
3. Modify selected elements;
4. Copy almost anything;
5. Parse a page;
6. Represent data in a table view;
7. Turn on design mode;
8. Take screenshots (Firefox only).

---

## 1. Run JavaScript in the Console

Yes, you can execute code on JavaScript inside the browser’s Console! The possibilities are limited, but they are enough for simple calculations, scripts, and testing of proof of concepts.

```
// Calculation in the Console
2 * 2

// JavaScript script for getting a random number
const getRandomInt = (max) => {
  return Math.floor(Math.random() * max);
};
console.log(getRandomInt(10));

// Get Unix timestamp
console.log(Math.floor(Date.now()));
```

![JavaScript in the Console](/assets/2023-11-14/02-javascript-in-the-console.png)

_Fig. 2. JavaScript in the Console_

This case includes the [handy command](https://developer.chrome.com/docs/devtools/console/utilities/#recent) `$_`. It returns the result of the last executed expression and can be used to engage a previous result in subsequent scripts.

```
2 * 2

// Return the result of the last expression
$_

// Calculate the square root of the last expression
Math.sqrt($_)
```

![$__](/assets/2023-11-14/03-underscore.png)

_Fig. 3._ $\_

Further reading:

- [Run JavaScript in the Console](https://developer.chrome.com/docs/devtools/console/javascript/) (Chrome);
- [Run JavaScript in the Console](https://learn.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/console/console-javascript) (Microsoft Edge);
- [Running JavaScript in the Browser Console](https://www.codecademy.com/article/running-javascript-in-the-browser-console).

## 2. Find elements, evaluate and validate selectors

Use [`document.querySelector()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) method to fetch a single element on the page (or the first one if several elements fit the selector). `$` is a shorter way of the same command.

```
document.querySelector('.notecard');
```

or

```
$('.notecard');
```

![document.querySelector()](/assets/2023-11-14/04-document-queryselector.png)

_Fig. 4. document.querySelector(). In this and and the following illustrations the browser’s Console is split-opened in the Inspector tab_

These methods are useful for finding elements on the page and for **validating your selectors that are going to be used in autotests.** Thus you can test [pattern-matching of CSS selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_selectors) for required elements.

```
$('[class*=banner]');
```

![document.querySelector() or $()_](/assets/2023-11-14/05-document-queryselector.png)

_Fig. 5. document.querySelector() or_ $()\_

Use [document.querySelectorAll()](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll) for fetching multiple elements on the page, or `$$` as a short alias.

```
document.querySelectorAll('[class^=code]');
```

or

```
$$('[class^=code]');
```

![document.querySelectorAll() or $$()_](/assets/2023-11-14/06-document-queryselectorall.png)

_Fig. 6. document.querySelectorAll() or \$$()_

For manual testing, the output of the `querySelectorsAll()` in the form of an array is of practical importance. Test engineer can instantly get the number of required elements by the length of the array.

The task of counting particular elements on the page can also be done in a more programming way:

`const numberOfSelectors = $$('[class^=code]').length;`

```
console.log(numberOfSelectors);
```

![$$().length_](/assets/2023-11-14/07-length.png)

_Fig. 7. \$$().length_

For fetching [XPath](https://developer.mozilla.org/en-US/docs/Web/XPath) selectors, use `$x`.

```
$x('.//h1');
```

![$x()_](/assets/2023-11-14/08-x.png)

_Fig. 8. $x()_

If nothing matches with your selector, you will get `null` on the empty array.

Further reading:

- [Locating DOM elements using selectors](https://developer.mozilla.org/en-US/docs/Web/API/Document_object_model/Locating_DOM_elements_using_selectors);
- [How to Locate Elements Using CSS Selectors in Selenium](https://www.lambdatest.com/learning-hub/css-selectors).

## 3. Modify selected elements

Console allows to store currently-inspected element and refer to it for DOM modifications. To do that:

1. Pic an element on the page;
2. Return to console;
3. Type the command: `$0`

Now, if you type `$0` again, you will get the element in the Console, and can modify the properties of the element ([node](https://developer.mozilla.org/en-US/docs/Web/API/Node) in the DOM terms) or get the [element](https://developer.mozilla.org/en-US/docs/Web/API/Element)’s properties and call [HTMLElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement) methods.

```
// Evaluate stored element
$0

// Replace text context of the node
$0.innerText="Ad hoc"

// Evaluate the click on the element
$0.click();
```

![$0_](/assets/2023-11-14/09-0.png)

_Fig. 9. $0_

Console allows to store up to five elements, and they will be available by `$0`, `$1`, `$2`, `$3`, and `$4.`

Further reading:

- [Console Utilities API reference](https://developer.chrome.com/docs/devtools/console/utilities/) (Chrome);
- [Web Console Helpers](https://firefox-source-docs.mozilla.org/devtools-user/web_console/helpers/index.html) (Firefox);
- [A few developer console tricks](https://gomakethings.com/a-few-developer-console-tricks/#trick-2-getting-the-currently-selected-item-in-the-elements-tab-as-a-variable).

## 4. Copy almost anything

After evaluation of the element (or an output of JS code), you can copy it directly from the Console.

```
copy($('[class*=beta]'));
```

Copied content:

```
<sup class="new beta">Beta</sup>
```

![copy()](/assets/2023-11-14/10-copy.png)

_Fig. 10. copy()_

**NOTE:** `copy()` does not copy recursive objects!

For example, if you get an array of elements by `querySelectorsAll()` and want to copy them through `copy()` — you will get nothing because of an array of elements.

```
// Evaluate all matching elements
$$('button.top-level-entry');

// Copy the result of the last expression
copy($_);
```

Copied content:

```
[
  {},
  {},
  {}
]
```

Unfortunately, **there is no way to fix it** [due to recursive objects](https://stackoverflow.com/a/25140576).

![copy() does not copy recursive objects](/assets/2023-11-14/11-cyclic-object-value.png)

_Fig. 11. copy() does not copy recursive objects_

But there is good news:

- You can easily copy all localStorage: `copy(localStorage);`
- And copy all cookies: `copy(document.cookie);`

Further reading:

- [How to Copy an Object or Array from the Console tab](https://bobbyhadz.com/blog/google-chrome-copy-object-or-array-from-console);
- [How to Copy a Big Object or Array From Console to Clipboard](https://dev.to/vtrpldn/how-to-copy-a-big-object-or-array-from-console-to-clipboard-3hi).

## 5. Parse a page

Combining all the previous knowledge, you can perform advanced actions, for example, parse links from a page.

```
// Evaluate all links with .footer-nav-item class
const footerLinks = $$('.footer-nav-item a');

// Get only href attribute from an each element
const footerLinksArr = footerLinks.map((item) => item.getAttribute('href'));

// Convert array of objects into a plane object
const footerLinksObj = Object.assign({}, footerLinksArr);

// Copy object
copy(footerLinksObj);
```

![Parse links from the page’s footer](/assets/2023-11-14/12-parse-a-page.png)

_Fig. 12. Parse links from the page’s footer_

Copied content:

```
{
  "0": "/en-US/about",
  "1": "/en-US/blog/",
  "2": "https://www.mozilla.org/en-US/careers/listings/?team=ProdOps",
  "3": "/en-US/advertising",
  "4": "https://support.mozilla.org/products/mdn-plus",
  "5": "/en-US/docs/MDN/Community/Issues",
  "6": "/en-US/community",
  "7": "https://discourse.mozilla.org/c/mdn/236",
  "8": "/discord",
  "9": "/en-US/docs/Web",
  "10": "/en-US/docs/Learn",
  "11": "/en-US/plus",
  "12": "https://hacks.mozilla.org/"
}
```

The beauty of this operation is that it was done just inside the browser without any additional tooling!

## 6. Represent data in a table view

Console has the [`table()` method](https://developer.mozilla.org/en-US/docs/Web/API/console/table) for the readability of arrays and plain objects. It accepts an array as parameters (or index-values of a plain object) and displays them in a table view.

```
console.table($_);
```

![console.table()](/assets/2023-11-14/13-console-table.png)

_Fig. 13. console.table()_

## 7. Turn on design mode

The cherry on top for manual testers is a chance to set ALL texts on the page to editable. That is done by changing [Document designMode property’s](https://developer.mozilla.org/en-US/docs/Web/API/Document/designMode) value.

```
document.designMode = "on"
```

![Design mode is on](/assets/2023-11-14/14-document-designmode-on.png)

_Fig. 14. Design mode is on ⇒ all document is editable_

Further reading about all Console’s features:

- [Console](https://developer.mozilla.org/en-US/docs/Web/API/console) (Web APIs);
- [A Guide to Console Commands](https://css-tricks.com/a-guide-to-console-commands/);
- [How to Use the Browser Developer Tools Console](https://blog.teamtreehouse.com/mastering-developer-tools-console).

## 8. Take screenshots (Firefox only)

Firefox has one special feature — taking screenshots by Console’s command:

- `:screenshot` — taking a screenshot of a currently visible viewport;
- `:screenshot --fullpage` — taking a screenshot of the full page.

![Screenshot in Firefox](/assets/2023-11-14/15-screenshot.png)

_Fig. 15. :screenshot in Firefox_

Further reading:

- [Taking screenshots with the web console](https://firefox-source-docs.mozilla.org/devtools-user/taking_screenshots/index.html#Taking_screenshots_with_the_web_console);
- [Essential Tool: Firefox’s screenshot Command](https://meyerweb.com/eric/thoughts/2015/10/22/firefoxs-screenshot-command/).

## 9. Detect which element has the focus

That hack is essential during [A11y testing](https://adequatica.github.io/2021/08/29/accessibility-manual-testing.html): when you navigate through the page by pressing the TAB key, you can lose a focus element (the outline property can be none, a focused element can be hidden, or elements’ order can be unpredictable).

`document.activeElement` — shows an element that currently has focus.

![document.activeElement in Firefox console](/assets/2023-11-14/16-activeelement-firefox.png)

_Fig. 16. document.activeElement in Firefox console_

Moreover, in **Chrome DevTools**, you can use [Live Expression](https://developer.chrome.com/docs/devtools/console/live-expressions#create) to watch your command’s values in real time:

1. Click on [Create live expression] (an eye-like button);
2. In the appeared «Expression» input, type `document.activeElement`
3. Now click on the page or tab around and see how the live expression’s output will update the currently focused element.

![document.activeElement in Chrome DevTools Live Expression](/assets/2023-11-14/17-activeelement-chrome-live-expression.png)

_Fig. 17. document.activeElement in Chrome DevTools Live Expression_

Further reading:

- [Document: activeElement property](https://developer.mozilla.org/en-US/docs/Web/API/Document/activeElement);
- [Track element focus](https://developer.chrome.com/docs/devtools/accessibility/focus);
- [Detect the element with focus at any time](https://devtoolstips.org/tips/en/track-focused-element/).

---

General documentation:

- [Console overview](https://developer.chrome.com/docs/devtools/console/) (Chrome);
- [Browser Console](https://firefox-source-docs.mozilla.org/devtools-user/browser_console/index.html) (Firefox);
- [The Console](https://developer.apple.com/library/archive/documentation/AppleApplications/Conceptual/Safari_Developer_Guide/Console/Console.html) (Safari).

Further tricks:

- [Console DevTools Tips](https://devtoolstips.org/tag/console/).

Copy @ [Medium](https://adequatica.medium.com/browser-console-for-functional-testing-650004cc4641)
