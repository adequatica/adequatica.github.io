---
layout: post
title: "Pragmatic Tools for Manual API Testing"
date: 2022-06-05 06:55:22 +0400
tags: api curl postman testing
---

As a web application test engineer, you need to test backend (APIs) as easily as frontend (UI in a browser).

Dozens of articles appear every year that tell about «the best tools for API testing», including [Swagger](https://swagger.io/) (which is a tool for documentation and should be implemented by developers), [REST Assured](https://rest-assured.io/) (which is a [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) for Java projects and requires programming skills), [JMeter](https://jmeter.apache.org/) (which is a tool for load testing) and other frameworks for automation testing — that is a total mess of approaches and technologies.

**I will focus only on tools for functional testing of HTTP/HTTPS REST APIs.**

There are several types of tools for testing API by manual testers — from the simplest to the text-based:

- Browsers
- GUI: Postman, Insomnia, Paw, Hoppscotch, TestMace
- CLI: сURL, HTTPie
- Visual Studio Code: Rest Client, Thunder Client

For all examples, I used the latest versions of the apps for macOS Big Sur and [OpenWeather API](https://openweathermap.org/api/one-call-3) handler:

```
https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&appid={api_key}
```

## Browsers

As strange as it may sound, a web browser is the very first tool to start API testing. You can only stop at this tool and [test almost everything from DevTools](https://medium.com/@adequatica/browser-devtools-as-an-essential-tool-for-api-testing-c2ace3fc47f2), but it is not too handy.

The browser is best suited for ad hoc checking GET requests. You just need to insert the [URI](https://stackoverflow.com/q/176264) in the address bar. This method is suitable for manual testing of parameters in the [query string](https://en.wikipedia.org/wiki/Query_string) or testing the correctness of the incoming response — in most cases, in [JSON](https://en.wikipedia.org/wiki/JSON) format.

[Firefox has a built-in JSON viewer](https://firefox-source-docs.mozilla.org/devtools-user/json_viewer/index.html) with syntax highlighting and a search filter:

![Firefox JSON viewer](/assets/2022-06-05/01-firefox-json-viewer.png)

_Fig. 1. Firefox JSON viewer_

«Raw Data» tab shows an unformatted body as it is, and «Pretty Print» option converts plain text into a readable form — very convenient for copying:

![Firefox «Raw Data» «Pretty Print» JSON viewer](/assets/2022-06-05/02-firefox-raw-data-pretty-print.png)

_Fig. 2. Firefox «Raw Data» «Pretty Print» JSON viewer_

Chromium-based browsers do not provide JSON viewer out of the box (despite that a primitive viewer in DevTools «Network» tab):

![Chrome](/assets/2022-06-05/03-chrome.png)

![Chrome preview](/assets/2022-06-05/04-chrome-preview.png)

_Fig. 3 & 4. Chrome [preview](https://developer.chrome.com/docs/devtools/network/reference/#preview)_

For a nice look at JSON output, you had to install extensions: [JSON Viewer](https://chrome.google.com/webstore/detail/json-viewer/gbmdgpbipfallnflgajpaliibnhdgobh) or [JSONVue](https://chrome.google.com/webstore/detail/jsonvue/chklaanhfefbnpoihckbnefhakgolnmc).

![Chrome with JSON Viewer extension](/assets/2022-06-05/05-chrome-json-viewer-extension.png)

_Fig. 5. Chrome with JSON Viewer extension_

If the API has authorization through cookies ([Cookie headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)), the browser will automatically path your cookie to a single GET request in the new tab. So, you can test API responses for unauthorized requests by using a Private (Incognito) window.

Even if browsers give enough room for API testing, the use of specialized tools (which will be discussed above) gives more flexibility to work with the API.

## GUI

### Postman

Today [Postman](https://www.postman.com/) is the most popular tool for API testing. Starting from a browser extension, it has become an extremely powerful desktop application for all platforms: Windows, Mac OS, and Linux. There is no reason to talk about its features here because all of them are repeatedly covered in many articles, [videos](https://www.youtube.com/postman), talks, and even [courses](https://medium.com/javarevisited/7-best-courses-to-learn-postman-tool-for-web-service-and-api-testing-f225c138fa5a).

Despite the ongoing promotion of its cloud platform, the app continues to provide for free its main and perfectly working function — issue HTTP requests to an HTTP API.

![Postman](/assets/2022-06-05/06-postman.png)

_Fig. 6. Postman_

### Insomnia

[Insomnia](https://insomnia.rest/) is a worthy alternative to Postman. It is [open source](https://github.com/Kong/insomnia), free, has almost the same set of features, more interface themes, and supports [plugins](https://insomnia.rest/plugins) that extend the standard functionality.

If you are an inexperienced user in API testing or you are [overwhelmed with Postman](https://www.aviskase.com/articles/2019/11/25/why-i-dont-use-postman/)’s features that you do not need — Insomnia is your choice. Advanced Postman’s users may face the absence of some of their usual things, but comparison is not a topic of this article.

![Insomnia](/assets/2022-06-05/07-insomnia.png)

_Fig. 7. Insomnia_

Further reading:

- [Using Insomnia for API exploration](https://www.aviskase.com/articles/2020/04/22/using-insomnia-for-api-exploration/)
- [API Testing with Insomnia](https://levelup.gitconnected.com/api-testing-with-insomnia-a090db3dbde6)
- [How to use Insomnia to Test API Endpoints](https://dev.to/kmcknight91/how-to-use-insomnia-to-test-api-endpoints-1lad)
- [Understanding Insomnia REST Client Made Easy 101](https://hevodata.com/learn/insomnia-rest-client/)
- [Postman vs. Insomnia: Comparing the API Testing Tools](https://itnext.io/postman-vs-insomnia-comparing-the-api-testing-tools-4f12099275c1)

### Paw

[Paw](https://paw.cloud/) is a powerful API client for Mac users. Since Paw is a native application, it is compact and has macOS graphical user interface. [It loads and parses JSON faster than Postman and Insomnia](https://rapidapi.com/blog/insomnia-vs-postman-vs-paw/#ux-and-performance), but it is gorgeously better exactly for 50$.

![Paw](/assets/2022-06-05/08-paw.png)

_Fig. 8. Paw_

Further reading:

- [Introducing Paw, an HTTP client and API tool for developers](https://setapp.com/news/paw-joins-setapp)
- [Paw Tutorial](https://pawtutorial.github.io/)
- [Insomnia vs. Postman vs. Paw: Comparing the Top API Clients](https://rapidapi.com/blog/insomnia-vs-postman-vs-paw/)

## Hoppscotch

[Hoppscotch](https://hoppscotch.io/) (previously known as [Postwoman](https://medium.com/@gopih/how-i-met-postwoman-now-hoppscotch-a29b80ab304a)) is a web online API request builder. It means that you do not need to install any desktop application — it is an app inside your browser and can be used on mobile devices. It is [open source](https://github.com/hoppscotch/hoppscotch), free, simple, and has a familiar Postman-like interface.

![Hoppscotch](/assets/2022-06-05/09-hoppscotch.png)

_Fig. 9. Hoppscotch_

Hoppscotch is a full-fledged web application that requests API directly via browser without any third-party layers. It means you can safely request your company’s internal API inside VPN.

![Hoppscotch does not issue unwanted requests](/assets/2022-06-05/10-hoppscotch.png)

_Fig. 10. Hoppscotch does not issue unwanted requests (proof from [Firefox DevTools](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/index.html))_

Further reading:

- [Api Testing with Hoppscotch](https://medium.com/@cyrilgeorge153/api-testing-with-hoppscotch-bc5b8c935fe8)
- [How to Test with Hoppscotch](https://hackernoon.com/relaible-and-secure-api-how-to-test-with-hoppscotch)
- [Learn API basics with Hoppscotch](https://aviyel.com/post/162/learn-api-basics-with-hoppscotch)

### TestMace

[TestMace](https://testmace.com/) is an IDE for API ([as developers call it](https://medium.com/@testmace.official/powerful-ide-to-work-with-api-b5753020ffe8)). It is yet another web app inside [Electron](https://www.electronjs.org/)’s wrapper as Postman and Insomnia. Unfortunately, I have not found any advantages over competitors (besides, the interface is quite buggy), and I included this tool in the list only because such a thing exists and is under development.

![TestMace](/assets/2022-06-05/11-testmace.png)

_Fig. 11. TestMace_

## CLI

### сURL

[сURL](https://curl.se/) is a command-line tool for transferring data through various network protocols, including HTTP — exactly what we need for REST API testing. By using cURL you can isolate your requests from the limitations and influences of the frontend environment and generate requests of any complexity that cannot be reproduced in any GUI client.

![cURL in Terminal](/assets/2022-06-05/12-curl.png)

_Fig. 12. cURL in Terminal_

[cURL has a fairly clear syntax for composing requests](https://curl.se/docs/manpage.html): `-X` for request method, `-H` for headers, and `-d` for body and other options.

My top cURL options:

- `-i` — show response headers in the output;
- `-k` — skip certificate verification;
- `-l` — follow redirects;
- `-m` — set timeout;
- `-v` — show the whole HTTP exchange (verbose mode, perfect for serious debugging).

The advantage of cURL is that you can get cURL’s request straight ahead from any browser by «[Copy as cURL](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/request_list/#request-list-copy-as-curl)» in a resource’s menu or by [code snippet in Postman](https://learning.postman.com/docs/sending-requests/generate-code-snippets/).

![«Copy as cURL» from Firefox DevTools](/assets/2022-06-05/13-copy-as-curl.png)

_Fig. 13. «Copy as cURL» from Firefox DevTools_

![cURL code snippet in Postman](/assets/2022-06-05/14-curl-postman.png)

_Fig. 14. cURL code snippet in Postman_

The next advantage of the console utility is that it could be combined with other Unix commands. In the example below, I used [time](<https://en.wikipedia.org/wiki/Time_(Unix)>) to determine the duration of the request and hid output by redirecting `stdout` to [/dev/null](https://en.wikipedia.org/wiki/Null_device):

![time curl -s](/assets/2022-06-05/15-time-curl-s.png)

_Fig. 15. time curl [-s](https://curl.se/docs/manpage.html#-s)_

The only difficulty with cURL is that it does not show JSON from the body in a pretty way. Therefore, [you have to use additional tools to parse the response](https://adequatica.medium.com/how-to-parse-json-after-curl-71e9413daa0c).

Further reading:

- [How to test a REST API from command line with curl](https://www.codepedia.org/ama/how-to-test-a-rest-api-from-command-line-with-curl/)
- [How to use curl to test a REST API](https://terminalcheatsheet.com/guides/curl-rest-api)
- [Testing Endpoints With Curl](https://www.smashingmagazine.com/2018/01/understanding-using-rest-api/#testing-endpoints-with-curl)

By the way, if your API responds with files or you have to test files downloading, then you probably should use [Wget](https://www.gnu.org/software/wget/) instead of cURL.

### HTTPie

[HTTPie](https://httpie.io/docs/cli) is a fancy command-line HTTP tool. It is promoted as an API testing tool, focused only on HTTP protocol, and has built-in JSON support. The request syntax is easy (but differs from cURL), and you do not need to have any special skills to start using it.

![Httpie in Terminal with verbose output](/assets/2022-06-05/16-httpie.png)

_Fig. 16. Httpie in Terminal with [verbose output](https://httpie.io/docs/cli/verbose-output)_

The developers are preparing a desktop [version of HTTPie](https://httpie.io/product), which is currently in private beta.

Further reading:

- [HTTPie: human-friendly CLI HTTP client for the API era](https://rustrepo.com/repo/httpie-httpie)
- [API Testing with HTTPie](https://levelup.gitconnected.com/api-testing-with-httpie-2d2d0dbe71ab)

## Visual Studio Code

### Rest Client

Some IDEs have integrated tools for debugging HTTP. For example, [Visual Studio Code](https://code.visualstudio.com/) has a [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) extension which allows sending HTTP requests and viewing the response directly from the editor. It supports [HTTP syntax](https://datatracker.ietf.org/doc/html/rfc7230) and cURL commands.

Once you have prepared the HTTP request, you should evoke the [command palette](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette) (`CMD/CTRL + Shift + P`) and choose «Rest Client: Send Request» — the response will open on the next tab:

![Rest Client extension](/assets/2022-06-05/17-rest-client-extension.png)

_Fig. 17. VS Code Rest Client extension_

### Thunder Client

If you do not want to type the requests, you can compose them directly from Visual Studio Code in a GUI way through [Thunder Client](https://www.thunderclient.com/). It is very handy to have a Postman-like interface and not to leave your code editor for making HTTP requests.

Thunder Client is a lightweight [extension](https://marketplace.visualstudio.com/items?itemName=rangav.vscode-thunder-client) with a limited set of functions, but all of them are exactly what you need for testing. It is also full of neat picky details like full screen mode, opening JSON response in a new tab, and even collections and environments as in a big app.

![Thunder Client extension](/assets/2022-06-05/18-thunder-client-extension.png)

_Fig. 18. VS Code Thunder Client extension_

A «Tests» tab is especially good — it consists of presets of the most useful checks:

![Thunder Client tests](/assets/2022-06-05/19-thunder-client-tests.png)

_Fig. 19. Thunder Client tests_

Further reading:

- [Thunder Client — lightweight alternative to Postman](https://rangav.medium.com/thunder-client-alternative-to-postman-68ee0c9486d6)
- [Thunder Client — An Alternative Way to Test Restful APIs](https://www.freecodecamp.org/news/thunder-client-for-vscode/)
- [How to use Thunder Client for API testing](https://www.katk.dev/thunder-client)?

---

I did not mention [Soap UI](https://www.soapui.org/) (despite the name it works with REST APIs), which is quite popular in the enterprise sector, but I did not have a chance to use it. Because of this, I can not talk about what I did not touch with my own hands.

Copy @ [Medium](https://adequatica.medium.com/pragmatic-tools-for-manual-api-testing-29f2a9da44a3)
