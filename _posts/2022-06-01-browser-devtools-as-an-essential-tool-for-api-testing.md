---
layout: post
title: "Browser DevTools as an Essential Tool for API Testing"
date: 2022-06-01 04:29:29 +0400
tags: api devtools testing
---

The DevTools «Network» tab provides almost everything for API testing — from checking responses to making requests.

As a web application test engineer, you need to test the integration of backend (APIs) with frontend (UI in a browser). During this activity, the [DevTools](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_are_browser_developer_tools) «Network» tab should be opened as a «must-have» tool because of the features described below:

- Network Activity
- Request Details
- Advanced Filters
- Content Search
- Disable Cache
- Preserve Log
- Request Blocking
- Edit and Resend Requests (only in Firefox)
- Open in New Tab
- Copy as cURL

Of course, the list can go on and on according to the tasks…

All examples are based on the [OpenWether web page](https://openweathermap.org/city/616052); the revealed `appid` token in some screenshots does not make any security sense.

## Network Activity

[Network Monitor](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/index.html) (in Firefox) or [Network Log](https://developer.chrome.com/docs/devtools/network/#load) (in Chrome) on the «Network» tab provides a list of all requests occurring on the page:

![«Network» tab in Firefox](/assets/2022-06-01/01-firefox-network-monitor-tab.png)

_Fig. 1. «Network» tab in Firefox_

![«Network» tab in Chrome](/assets/2022-06-01/02-chrome-network-log-tab.png)

_Fig. 2. «Network» tab in Chrome_

Depending on the [client-server](https://en.wikipedia.org/wiki/Client%E2%80%93server_model) interaction of your web app, you may be interested in [XHR](https://en.wikipedia.org/wiki/XMLHttpRequest) requests — most REST API requests are done through it (of course, if your web app is not completely server-side rendered).

There are basic filters by type of request:

![Firefox Network monitor toolbar filters by type](/assets/2022-06-01/03-firefox-network-monitor-toolbar-filters.png)

_Fig. 3. Firefox Network monitor toolbar filters by type (XHR)_

Docs:

- [Network monitor toolbar](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/toolbar/index.html) (Firefox)
- [Filter requests by type](https://developer.chrome.com/docs/devtools/network/reference/#filter-by-type) (Chrome)

## Request Details

All resources from the network monitor/log can be examined for [all the elements of the request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview#http_messages) in the context of API testing: request/response headers and request/response body.

To my mind, Firefox represents details in a nicer format: with text highlighting, filters, raw data view (in Chrome, it calls «View source»), and references to [documentation](https://developer.mozilla.org/en-US/docs/Web) for each header:

![Firefox header details](/assets/2022-06-01/04-firefox-header-details.png)

_Fig. 4. Firefox header details_

![Chrome header details](/assets/2022-06-01/05-chrome-header-details.png)

_Fig. 5. Chrome header details_

The same for response’s body details — [Firefox’s built-in JSON viewer](https://firefox-source-docs.mozilla.org/devtools-user/json_viewer/index.html) highlights syntax and allows filtering. Chrome has only a basic parser on «Preview» tab:

![Firefox response details](/assets/2022-06-01/06-firefox-response-details.png)

_Fig. 6. Firefox response details_

![Chrome response details](/assets/2022-06-01/07-chrome-response-details.png)

_Fig. 7. Chrome response details_

Docs:

- [Network request details](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/request_details/index.html) (Firefox)
- [Inspect a resource’s details](https://developer.chrome.com/docs/devtools/network/#details) (Chrome)

## Advanced Filters

Contemporary web applications make tens of hundreds of requests per page, so it is very important to be able to filter out excess. There are a sufficient number of different parameters available for [filtering](https://developer.chrome.com/docs/devtools/network/#filterbox). For example:

- `status-code` — shows specific HTTP status code — `status-code:304`
- `method` — shows specific HTTP method — `method:options`
- `domain` — shows specific domain — `domain:googleapis.com`
- `-` — negates the filter’s query string — `-domain:openweathermap.org` (shows all resources **not** from specified domain);
- `larger-than` — shows resources that are larger than the specified size in bytes. You can use suffix `k` for kilobytes (1k = 1024 bytes) and `m` for megabytes — `larger-than:1m`
- `is:running` — shows incomplete requests. This filter is also good for showing [WebSocket](https://en.wikipedia.org/wiki/WebSocket) resources (if you forgot to set the filter by type in advance).

![Filter JSON data in Firefox](/assets/2022-06-01/08-firefox-filter-json-data.png)

_Fig. 8. Filter JSON data in Firefox — `mime-type:application/json`_

Docs:

- [Filtering by properties](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/request_list/#filtering-by-properties) (Firefox)
- [Filter requests by properties](https://developer.chrome.com/docs/devtools/network/reference/#filter-by-property) (Chrome)

## Content Search

Unlike the query filter through requests, «Search» runs a full-text search through headers and content:

![Search content in responses in Firefox](/assets/2022-06-01/09-firefox-search-content-in-responses.png)

_Fig. 9. Search content in responses in Firefox_

![Search content in headers in Chrome](/assets/2022-06-01/10-chrome-search-content-in-headers.png)

_Fig. 10. Search content in headers in Chrome_

Docs:

- [Search in requests](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/request_list/index.html#search-in-requests) (Firefox)
- [Search network headers and responses](https://developer.chrome.com/docs/devtools/network/#search) (Chrome)

## Disable Cache

Browsers like to cache responses to improve performance and efficient traffic utilization (they can be filtered by `is:cached`) for the next page load:

![Cached resources in Chrome](/assets/2022-06-01/11-chrome-cached-resources.png)

_Fig. 11. Cached resources in Chrome_

Use «Disable Cache» to prevent caching — all resources will be requested as at the first time.

![Disable Cache checkbox in Firefox](/assets/2022-06-01/12-firefox-disable-cache-checkbox.png)

_Fig. 12. Disable Cache checkbox in Firefox_

Docs:

- [Emulate a first-time visitor by disabling the browser cache](https://developer.chrome.com/docs/devtools/network/reference/#disable-cache) (Chrome)

## Preserve Log

If you have to refresh the page in order for testing or your test page makes redirects or auto-refreshes, the network activity is cleared each time the page reloads. Enable «Persist Logs» (in Firefox) or «Preserve Log» (in Chrome) to save all requests across page loads (they will be saved until you disable them).

![In Firefox this setting is hidden inside the gear icon](/assets/2022-06-01/13-firefox-gear-icon.png)

_Fig. 13. In Firefox this setting is hidden inside the gear icon_

![Preserve log checkbox in Chrome](/assets/2022-06-01/14-chrome-preserve-log-checkbox.png)

_Fig. 14. Preserve log checkbox in Chrome_

Docs:

- [Network monitor toolbar](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/toolbar/index.html) (Firefox)
- [Save requests across page loads](https://developer.chrome.com/docs/devtools/network/reference/#preserve-log) (Chrome)

## Request Blocking

Blocking requests is more demanded for frontend rather than backend testing, but at least this feature is quite useful — you do not need third-party proxies to do it.

![Blocking in Firefox](/assets/2022-06-01/15-firefox-blocking.png)

_Fig. 15. Blocking in Firefox_

Chrome allows you to block by URL and by domain. In the «Network request blocking» panel, you can edit blocking rules by patterns and wildcards (very convenient to block trackers):

![Network request blocking in Chrome](/assets/2022-06-01/16-chrome-network-request-blocking.png)

_Fig. 16. Network request blocking in Chrome_

All blocked requests will be highlighted and can be easily filtered in both DevTools (Firefox and Chrome).

Docs:

- [Blocking specific URLs](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/request_list/index.html#network-monitor-blocking-specific-urls) (Firefox)
- [Block requests](https://developer.chrome.com/docs/devtools/network/#block) (Chrome)

## Edit and Resend Requests (only in Firefox)

This magnificent feature is not sufficiently covered in the documentation, but it is incredibly useful for API testing or debugging. You can take any request (even POST and other methods that modify data), edit it, and resend it on behalf of the page!

![«New Request» form in Firefox](/assets/2022-06-01/17-firefox-new-request-form.png)

_Fig. 17. «New Request» form in Firefox_

Open request details on «Headers» tab → Resend → Edit and Resend → [Send] the «New Request» form:

![Edit and resend request in Firefox](/assets/2022-06-01/18-firefox-edit-and-resend-request.gif)

_Fig. 18. Edit and resend request in Firefox_

From the same context menu, you can also resend requests without modification (maybe if you want to test the handler for [idempotence](https://en.wikipedia.org/wiki/Idempotence)).

Docs:

- [Context menu](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/request_list/#context-menu) (Firefox)

## Open in New Tab

You can resend any request in a new browser’s tab for further inspection of the body (in case of API requests) or for downloading scripts, fonts, images, and other content:

![Open in a new tab in Chrome](/assets/2022-06-01/19-chrome-open-in-a-new-tab.png)

![Open in a new tab in Chrome](/assets/2022-06-01/20-chrome-open-in-a-new-tab.png)

_Fig. 19 & 20. Open in a new tab in Chrome_

Firefox’s way of showing API requests in a new tab again is nicer than Chrome’s — there are JSON viewer and info about headers:

![Edit and resend request in Firefox](/assets/2022-06-01/21-firefox-edit-and-resend-request.png)

![Edit and resend request in Firefox](/assets/2022-06-01/22-firefox-edit-and-resend-request.png)

![Edit and resend request in Firefox: Raw Data](/assets/2022-06-01/23-firefox-edit-and-resend-request-raw-data.png)

![Edit and resend request in Firefox](/assets/2022-06-01/24-firefox-edit-and-resend-request.png)

_Fig. 21–24. Edit and resend request in Firefox (with proofs)_

Take a closer look at the screenshots: thanks to developers for the [Copy] buttons — it saves time.

## Copy as cURL

For testing API through other tools, you can copy requests in [cURL](https://curl.se/) format exactly the same as a browser does:

![Copy as cURL in Firefox](/assets/2022-06-01/25-firefox-copy-as-curl.png)

_Fig. 25. Copy as cURL in Firefox_

The result of copying to the clipboard:

```
curl 'https://openweathermap.org/data/2.5/weather?id=2643743&appid=439d4b804bc8187953eb36d2a8c26a02' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:100.0) Gecko/20100101 Firefox/100.0' -H 'Accept: */*' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate, br' -H 'Referer: https://openweathermap.org/city/2643743' -H 'DNT: 1' -H 'Connection: keep-alive' -H 'Cookie: october_session=eyJpdiI6IlpYR1NmbEpJRUk2RDhHZkRsNVVZR2c9PSIsInZhbHVlIjoiRDBxbW9NdmdoSkdtV2FCVmwxUWZDY2hJSkpMcWIrN2xqMW14NXZmSXloTkwwVzdPMXE0UzdhRDg1VThFVWVpMFlna1FIcW9Tb1kzcnJUdHZJZlgxRjNCUHRCRXNDTHRRTjZCNzlRQUZqelVQV0gyOENkenRtVWpjc09ERW11SHEiLCJtYWMiOiI1NDYyM2RiMTM3MTZmNTgxMjJmY2QxMDRhM2Y1MjU2MTgwZWE3ZWIyOGQzMjg2MGE3ZjA3NGExMDMxNDg3OTRlIn0%3D; units=metric; stick-footer-panel=false; cityid=2643743' -H 'Sec-Fetch-Dest: empty' -H 'Sec-Fetch-Mode: cors' -H 'Sec-Fetch-Site: same-origin'
```

Besides cURL, Chrome has more options for copy:

![Copy as cURL in Chrome](/assets/2022-06-01/26-chrome-copy-as-curl.png)

_Fig. 26. Copy as cURL in Chrome_

Docs:

- [Copy as cURL](https://firefox-source-docs.mozilla.org/devtools-user/network_monitor/request_list/#request-list-copy-as-curl) (Firefox)
- [Copy one or more requests to the clipboard](https://developer.chrome.com/docs/devtools/network/reference/#copy) (Chrome)

I hope that the reader knows why he needs a cURL and how to use it:

![cURL request in Terminal](/assets/2022-06-01/27-curl-request-in-terminal.png)

_Fig. 27. cURL request in Terminal_

In the example above I added `-i --output -` to cURL request to show response headers and binary output.

Copy @ [Medium](https://adequatica.medium.com/browser-devtools-as-an-essential-tool-for-api-testing-c2ace3fc47f2)
