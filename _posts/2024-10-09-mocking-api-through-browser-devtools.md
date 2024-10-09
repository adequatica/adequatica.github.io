---
layout: post
title: "Mocking API Through Browser DevTools"
date: 2024-10-09 04:11:47 +0200
tags: mocking devtools
---

For decades, to mock API responses for UI testing, QA engineers used special proxies and interceptors like [Charles](https://www.charlesproxy.com/), [Fiddler](https://www.telerik.com/fiddler), [Proxyman](https://proxyman.io/), [Requestly](https://requestly.com/), or [HTTP Toolkit](https://httptoolkit.com/); even [Postman has acquired similar functionality](https://learning.postman.com/docs/sending-requests/capturing-request-data/capture-with-proxy/).

Under «mocking» API responses, I figuratively mean all kinds of overriding of different parts of [HTTP requests and responses](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages). This is a very important and useful tool by itself, which helps to test, prototype, and develop the frontend.

Today, browsers have started to provide the ability to **override HTTP responses** out of the box, straight from DevTools, without any third-party extensions. This is very convenient, especially if you do not have the opportunity or desire to install and deal with a new tool for a one-time occasion.

Currently, only Chrome (and other Chromium-based browsers) and Safari have such a feature, but I hope Firefox will implement it someday, too. Chrome has started [to support overrides since the 117 version](https://developer.chrome.com/blog/new-in-chrome-117#local-overrides), which was released in September 2023 — more than a year ago, but I was surprised when a vast number of colleagues around did not know about this feature. And Safari (WebKit) has been supporting overriding since around 2020!

I will give an example of overriding HTTP response bodies in both cases: [Chrome DevTools](https://developer.chrome.com/docs/devtools) and [Safari Web Inspector](https://developer.apple.com/documentation/safari-developer-tools/web-inspector). I took the [OpenWeather website](https://openweathermap.org/) and its API as a sample because their client-server interaction is very clear.

---

## Chrome

Open DevTools and go to the Network tab. Choose a resource you want to override, right-click it, and select [Override content].

![Chrome 01. [Override content]](/assets/2024-10-09/chrome-01-override-content.png)

_Fig. Chrome 01. [Override content]_

Chrome will first ask you to select a directory to store override files.

![Chrome 02. Select folder](/assets/2024-10-09/chrome-02-select-folder.png)

_Fig. Chrome 02. Select folder_

After you grant access to it, Chrome will redirect you to the _Sources_ tab.

![Chrome 03. DevTools Sources tab](/assets/2024-10-09/chrome-03-sources.png)

_Fig. Chrome 03. DevTools Sources tab_

Here, you can edit the JSON body just in the DevTools!

After the page refresh, the response body of the resource will be overwritten by the new data.

![Chrome 04. Enable Local Overrides](/assets/2024-10-09/chrome-04-enable-local-overrides.png)

_Fig. Chrome 04. Enable Local Overrides_

In the screenshot above, you can see the implausible weather values that turned out to be overwritten by the new data. This data is saved in a file in the selected folder.

**Chrome DevTools documentation: [Override web content and HTTP response headers locally](https://developer.chrome.com/docs/devtools/overrides/)**.

---

## Safari

Open Web Inspector and go to the _Sources_ tab. Click [+] and select [Local Override…].

![Safari 01. [Local Override…]](/assets/2024-10-09/safari-01-local-override.png)

_Fig. Safari 01. [Local Override…]_

In the _Local Overrides_ section, you can fill the popover’s inputs with the corresponding values to override the required HTTP resource. In the Type field, choose Resonse > [File] to override the response’s body.

![Safari 02. Local Overrides](/assets/2024-10-09/safari-02-local-overrides-file.png)

_Fig. Safari 02. Local Overrides_

In my experience, filling all the fields correctly from scratch is quite tricky, and the difficulty varies from site to site. **The easiest way** to make an override rule for response is to find a required HTTP resource and choose [Create Response Local Override].

![Safari 03. [Create Response Local Override]](/assets/2024-10-09/safari-03-create-response-local-override.png)

_Fig. Safari 03. [Create Response Local Override]_

It creates a fully prefilled popover to override the response. In the last step, you need to select a file where you saved the new response body in advance.

![Safari 04. Response local override popover](/assets/2024-10-09/safari-04-response-popover.png)

_Fig. Safari 04. Response local override popover_

After the page refresh, the response body of the resource will be overwritten by the chosen file’s contents.

![Safari 05. This resource came from a local override](/assets/2024-10-09/safari-05-reveal-override.png)

_Fig. Safari 05. This resource came from a local override_

In the screenshot above, you can see the implausible weather values that turned out to be overwritten from the file.

**WebKit documentation: [Local Overrides](https://webkit.org/web-inspector/local-overrides/).**

---

Read further:

- [Take advantage of Local Overrides when developing with Safari](https://medium.com/@hbko/take-advantage-of-local-overrides-when-developing-with-safari-5c4da8577923);
- [Overriding HTTP Response Headers in Chrome Dev Tools](https://scotthelme.co.uk/overriding-http-response-headers-in-chrome-dev-tools/);
- [Replace HTTP Responses Through Charles](https://adequatica.github.io/2021/09/21/replace-http-responses-through-charles.html).

Copy @ [Medium](https://adequatica.medium.com/mocking-api-through-browser-devtools-f84ce77d01fd)
