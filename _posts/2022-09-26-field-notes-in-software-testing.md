---
layout: post
title: "Field Notes in Software Testing"
date: 2022-09-26 07:55:19 +0300
tags: testing
---

Being a test engineer (QA) for a decade, «[I’ve seen things you people wouldn’t believe](https://www.acmi.net.au/stories-and-ideas/blade-runner-ive-seen-things/).» I’ve tested enough web applications to accumulate experience that can be shared.

![Field Notes in Software Testing](/assets/2022-09-26/00-cover.jpg)

I collected tricky test cases, which somehow led to bugs and hacks to perform unusual checks. They are sorted by logical blocks in no particular order. Enjoy!

## General cases

- Divide by zero wherever it is possible;
- Plug in a real mouse to notebook — it can affect scrollbars, which may be hiding in the case of touchpad;
- Click on disabled buttons (controls) — sometimes they can just look like disabled but actually be clickable;
- Open links and click buttons by [double-click](https://en.wikipedia.org/wiki/Double-click);
- Repeat the action multiple times, like clicking on the same button 10 times in a row;
- Click on all arrows pointing downwards `∨` or upwards `∧` — something must open or close:
- Open pop-ups and dropdowns near the edge of the screen — they can be cut off by viewport;
- Open long dropdowns and suggests near the bottom of the screen — they can be cut off by the viewport if the developers have not provided a scroll for that element:

![The height of the suggest does not have a maximum value and therefore goes off the screen](/assets/2022-09-26/01-suggest.png)

_Fig. 1. The height of the suggest does not have a maximum value and therefore goes off the screen_

- If there is a countdown timer on the page, you must check what happens when it stops. For example, I saw `NaN` and the continuation of the countdown with negative values;
- Try a leap day ([February 29](https://en.wikipedia.org/wiki/February_29)) in a date input. Rather, a wrong leap day (29.02.2021) is more important. Almost every popular frontend and backend frameworks handle this case, but it is a constant trip for custom ones;
- Keep in mind that there are [magic numbers](<https://learn.microsoft.com/en-us/previous-versions/software-testing/ee621251(v=msdn.10)>) for every test case.

## Non-general cases

- Use the tested website after an idle (after a few hours or a day of inactivity) — it can affect sessions with expiring cookies or other timers;
- Flip the monitor in portrait mode and then open the tested website — it can affect layouts that are not designed for narrow and long resolutions. For example, in this case, the width of a 23-inch monitor will be only 1080 px;
- Check the print version of the tested website:

![Printing is an outdated case for most modern websites](/assets/2022-09-26/02-print.png)

_Fig. 2. Printing is an outdated case for most modern websites, but for documentation, long reads, maps, and tickets, it is still valid and valuable_

## On coordinates

- Double check [geographic coordinates](https://en.wikipedia.org/wiki/Geographic_coordinate_system): the values of latitude and longitude **should NOT be mixed up in places** — surprisingly, it is quite a common bug because some APIs operate lat/long, but others long/lat (and there is a special place in hell for developers using a single `ll` parameter for positioning due to non-obviousness);
- For all human-readable places and inputs, **it is highly recommended to put latitude coordinates first and longitude second** because it is a traditional [positioning](https://sailingissues.com/navcourse1.html) order in all maps;
- Latitude and longitude ranges are not the same (again, surprisingly, such bugs occur very often):
  - latitude range is from `-90` to `90`, and
  - longitude range is from `-180` to `180`.
- Latitude and longitude can be `0` — it’s a perfect edge case and [Null Island](https://en.wikipedia.org/wiki/Null_Island) on Earth;
- Watch out for coordinates in exponential format, like `4.4816236e+1, 2.0460467e+1` instead of regular numbers;
- Carry on the [coordinates precision](https://wiki.openstreetmap.org/wiki/Precision_of_coordinates). Unlimited precision may cause unnecessary calculations; therefore, slicing or rounding the coordinate value may have advantages. Eight decimals are already [nearly 1 mm in accuracy](https://gis.stackexchange.com/questions/8650/measuring-accuracy-of-latitude-and-longitude).

## On opening URLs

- Open URL with `/` and without `/` at the end ([https://medium.com/](https://medium.com/) or [https://medium.com](https://medium.com/)) — opening the page can depend on the app’s routing or server’s settings;
- Open URL with [encoded characters](https://www.url-encode-decode.com/) (`Dvo%C5%99%C3%A1k` instead of `Dvořák`);
- Keep in mind that in some cases, space can be transformed into `+`
- If there is pagination in the URL’s path, then open `0` or negative page (`-1`);
- Try to request an HTTPS page by HTTP — it can depend on the server’s redirect rules.

## On search inputs (applies to any input fields) {#on-search-inputs-applies-to-any-input-fields}

- Search for a [space](<https://en.wikipedia.org/wiki/Space_(punctuation)>);
- Search with a space before and after the request;
- Search for an empty request — it can affect inputs with blank field validation (read more about [client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation));
- Search for [special characters](https://wiki.mozilla.org/Help:Special_characters): `! * ' ( ) ; : @ & = + $ , / ? % # [ ] &lt; > .`
- Search for special characters through mnemonic [entities](https://www.w3.org/wiki/Common_HTML_entities_used_for_typography), for example: `&laquo; &raquo;` instead of `« »`
- Search for `:` — it can affect Python’s backends, especially if the search is performed on the [clickhouse](https://github.com/ClickHouse/ClickHouse) database;
- Search for cyrillic `ё` (or any non-latin letters or hieroglyphs). [There was a bug in iPhone](https://nymag.com/intelligencer/2018/02/apple-ios-11-telugu-bug-locks-users-out-of-messages.html) with the Telugu character జ్ఞా
- Search for text in different encodings;
- Search for `[object Object]`
- Search for `%`, by `%%`, and by `%` with combination of special characters;
- Search for XSS — actually, it is already a part of [security testing](https://systemweakness.com/a-brief-security-testing-for-manual-testers-f2f59d56fbb5);
- If there are a few inputs, then put XSS values in several inputs at once;
- If a test page has custom hotkeys, then use these keys when filling in the input;
- If a tested page has a video, then: [Play] it → [Pause] it → and try to search on a tested page through the browser’s search; do not forget to use [space] — it can affect paused video, because [space] is a common hotkey for pausing.
- Input a long unbroken text — it can cause page overflows and other artifacts (very long Wikipedia links usually break layout). For instance: _Eximperituserqethhzebibšiptugakkathšulweliarzaxułum_
- Try to find a limit on the number of characters in the search request — at least there must be some kind of restriction. Otherwise, the users can load our backend with monstrous requests;
- How is the filled field cleared?
- Remove multi-character emojis: paste [family emoji](https://emojipedia.org/family) in the input field and then hit the backspace — if the input field can handle multi-character emojis, you will delete family members one by one:

![Removing multi-character emoji](/assets/2022-09-26/03-emojis.gif)

_Fig. 3. Removing multi-character emoji_

## On usernames

- Register account with space before and/or after a username: `augustdvorak `
- Register account with lower-case and upper-case letters in a username: `AugustDvoraK`
- Register account with XSS in a username, something like: `&lt;img/src='x'onerror=alert(august)>`
- Register account with a username in angle brackets: `&lt;augustdvorak>` — it can affect both frontend and backend. If [escaping](https://en.wikipedia.org/wiki/Escape_character) works too strictly, the whole username can be erased;
- Register account with `.` or `-` in a username: `august.dvorak` or `august-dvorak` — in some cases `.` and `-` can behave as the same symbol.

## On mobile version of the websites

- Open mobile version of the site on 480x800 px resolution and/or on a 4-inch screen or less;
- After opening the page, the insertion caret (that blinking input cursor) should not stand in the input field (of course, if it is not provided by the documentation) — this causes the keyboard to open automatically and may irritate the users;
- Try to request a desktop version of the site in a mobile browser:

![Request Desktop Website in Safari](/assets/2022-09-26/04-request-desktop-website.png)

_Fig. 4. Request Desktop Website in Safari_

- If the layout of a tested website is based on [media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries) for responsive design, try to **slowly** resize the width.

## On crossbrowserness

In all projects, I adhered to the principle:

- For browsers over 5 % — the layout must match the design mockup;
- For browsers between 5–1 % — the layout may be broken, but the functionality should work;
- Do not support browsers less than 1 %;

Of course, there may be exceptions for each case. For example, in one project, we turned off an outdated functionality for 0,5 % of users, but it was more than 12 000 daily users!

Further reading about [cross browser testing](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing).

## On browser behavior {#on-browser-behavior}

- Back to the previous page by `[Back]` — it can affect [SPA websites](https://en.wikipedia.org/wiki/Single-page_application) and websites which use [History API](https://developer.mozilla.org/en-US/docs/Web/API/History/back);
- Switch to another tab by `[CMD]+[NUM_KEY]` or switch to another application by `[ALT]+[TAB]`;
- Search on a tested page through the browser’s search: `[CMD]+[F]` — it can affect pages with pop-ups, modal windows, dynamic loading scripts, and [infinite scroll](https://www.smashingmagazine.com/2013/05/infinite-scrolling-lets-get-to-the-bottom-of-this/);
- Navigate by keyboard `[TAB]` — actually, it is already a part of [accessibility testing](https://adequatica.github.io/2021/08/29/accessibility-manual-testing.html);
- [Hard refresh the browser](https://fabricdigital.co.nz/blog/how-to-hard-refresh-your-browser-and-clear-cache): `[CMD]+[SHIFT]+[R]` — the browser will clear the cache on refresh, but the URL’s path should remain unchanged;
- Check Safari’s Console in DevTools, because additional [CSP errors](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP/Errors) can occur in this browser;
- Make the browser window fullscreen — most likely, nothing will break, but who knows?

## On browser settings

- Try to use a tested site in a [private](https://support.mozilla.org/en-US/kb/private-browsing-use-firefox-without-history) (incognito) mode;
- Try to use a tested site with [blocked third-party cookies](https://support.mozilla.org/en-US/kb/third-party-cookies-firefox-tracking-protection);
- Try to use a tested site with [tracking protection](https://wiki.mozilla.org/Security/Tracking_protection):

![privacy.trackingprotection.enabled](/assets/2022-09-26/05-privacy-trackingprotection-enabled.png)

_Fig. 5. Set privacy.trackingprotection.enabled = true at about:config_

- Try to use a tested site with [HTTPS-only mode](https://support.mozilla.org/en-US/kb/https-only-prefs);
- Change (increase) default [font size and/or zoom on the page](https://support.mozilla.org/en-US/kb/font-size-and-zoom-increase-size-of-web-pages) — it can affect the layout of any kind, but only less than 4 % of the users have such a setting (this number is based on private statistics).

## On network settings

- Try to use a tested site with [network throttling](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/throttling/index.html) (DevTools setting);
- Try to use a tested site under VPN from another country;
- Try to use a tested site under VPN from another country with a different time zone.

## On arrow controls

If a web application supports control by arrow keys (like driving or [docking](https://iss-sim.spacex.com/) simulators in a browser), it is very easy to put the software keys into the sticking state. This common issue has a lot of cases of reproduction.

With the arrow keys ⬆️ ⬇️ ⬅️ ➡️ clamped down, try to do:

1. A lot of frequent/short clicks on the arrow keys;
2. (For Mac) Press `[CMD]` and release the pressed arrow keys;
3. (For Mac) Press `[FN]`;
4. (For Mac) Open Spotlight by the button on the [touch bar](https://support.apple.com/guide/mac-help/use-the-touch-bar-mchlbfd5b039/mac);
5. (For Mac) Move the cursor to the menu bar;
6. (For Mac/Linux) Press `[WIN]` key on the extended keyboard;
7. ([For Windows](https://en.wikipedia.org/wiki/Windows_key)) Press `[WIN]`;
8. Press `[CTRL]` or `[ALT]` or `[SHIFT]` or a combination of them;
9. Press `[ESC]`;
10. Switch to another program by `[CMD]+[TAB]` (for Mac) / `[ALT]+[TAB]` ([for Windows](https://en.wikipedia.org/wiki/Alt-Tab));
11. Switch to another program by mouse click;
12. Switch to another browser tab by mouse click;
13. Switch to another browser tab by `[CTRL]+[TAB]` or `[TAB]+[{NUM}]`;
14. Move the mouse away from the active browser’s tab/window;
15. Make right-click (Right Mouse Button) and left-click (Left Mouse Button);
16. Reconnect the network (the loss of internet connection).

## On odd utilities

- Run a «crawler» or link checker to find broken links on the pages of your website. There are plenty of apps for that, or you can program your own HTML parser.
- The best way to check access (connection) to the resource is by using [telnet](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/telnet) — any sysadmin will appreciate a stack trace from it rather than a report «the site does not work». Unfortunately, telnet does not run out of the box, so you have to install it (for the Mac, it can be done through [Homebrew](https://formulae.brew.sh/formula/telnet)). The command is like: `telnet {domain} {port}`

![Telnet](/assets/2022-09-26/06-telnet.png)

_Fig. 6. telnet developer.mozilla.org 80_

- You can find out your IP address through the terminal by curl request for [ifconfig.me](https://ifconfig.me/) or [ipinfo.io](https://ipinfo.io/): `curl ifconfug.me` or `curl ipinfo.io`
- If your test environment requires you to use third-party browser extensions to work with (for example, permanent headers modification as a workaround to bypass CORS), it is a potential problem to miss a bug.

## How to find out the user-agent through the browser console?

```
console.log(navigator.userAgent);
```

![navigator.userAgent](/assets/2022-09-26/07-navigator-useragent.png)

_Fig. 7. [User-agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent)_

## How to change user-agent without addons? {#how-to-change-user-agent-without-addons}

This trick [works only in Chrome browser](https://developer.chrome.com/docs/devtools/device-mode/override-user-agent/):

1. Open Command menu: `[CMD]+[Shift]+[P]`
2. Type «network conditions» and select **Show Network conditions**;
3. In the «User agent» section, disable the «Use browser default» checkbox;
4. Input a desired user-agent:

![User-agent client hints](/assets/2022-09-26/08-user-agent-client-hints.png)

_Fig. 8. [User-agent client hints](https://web.dev/user-agent-client-hints/)_

## How to add a cookie through the browser console?

```
document.cookie = "key=Value";
```

## How to add data to localStorage through the browser console?

```
localStorage.setItem("foobar_popup", "1");
```

Further reading about [interacting with localStorage from the Console](https://developer.chrome.com/docs/devtools/storage/localstorage/#console).

## How to prevent auto closing (hide on unhover) panels/pop-ups/menus?

It is always inconvenient to write autotests for panels/pop-ups/menus that show on hover and close on unhover — how to find out the classes of hidden elements?

1. Open [Debugger](https://developer.mozilla.org/en-US/docs/Tools/Debugger) in Firefox DevTools ([Source panel](https://developer.chrome.com/docs/devtools/javascript/sources/#debug) in Chrome DevTools) — there is a [Pause] button;
2. Open desired panel/pop-up/menu;
3. Hit the pause’s hotkey: `[F8]` — JS execution on a page is stopped, none of the elements will hide automatically, and you can inspect them with no rush.

![Paused at Execution](/assets/2022-09-26/09-paused-at-execution.gif)

_Fig. 9. Paused at Execution_

---

The article will probably be updated in case extraordinary test cases are discovered in the future.

Copy @ [Medium](https://adequatica.medium.com/field-notes-in-software-testing-5e395424124)
