---
layout: post
title: "Scan Postman Traffic Through Charles"
date: 2021-09-23 10:31:37 +0300
tags: charles postman testing
---

Sometimes, as a test engineer, you need to go deeper for checking API.

Once in my practice, I made requests from Postman, which responded in an unpredictable way. I suggested that Postman could somehow additionally change the request when sending it. To check this hypothesis, I needed to scan Postman traffic, and [Charles Proxy](https://www.charlesproxy.com/) was the right tool for this task.

Disclaimer:

- As an example, [COVID-19 Rich Data Services](https://documenter.getpostman.com/view/2220438/SzYevv9u) API was selected, because it is open and can be accessed without token;
- [Charles should be installed and configured](https://adequatica.medium.com/replace-http-responses-through-charles-f2954c372b40#c750) at least for working with web applications;
- I use Charles 4.5.6 on macOS Catalina 10.15.7 and Postman 8.12.1.

![Charles](/assets/2021-09-23/00-charles.png)

## Setup Charles for Postman

Fortunately, a [free version of Charles](https://www.charlesproxy.com/download/) is enough to complete our discovery.

1\. Go to _Proxy → Proxy Settings…_ and fill in the field _HTTP Proxy_ = 8888 and check the box _Support HTTP/2_ in the «Proxy Settings» window.

![Charles → Proxy → Proxy Settings](/assets/2021-09-23/01-charles-proxy-proxy-settings.png)

_Charles → Proxy → Proxy Settings_

2\. Then go to _Proxy → SSL Proxying Settings…_ and check the box _Enable SSL Proxying_ in the «SSL Proxying Settings» window.

![Charles → Proxy → SSL Proxying Settings](/assets/2021-09-23/02-charles-proxy-ssl-proxing-settings.png)

_Charles → Proxy → SSL Proxying Settings_

3\. Add a new endpoint in the «SSL Proxying Settings» — click [Add] under _Include_ table and fill the fields according to your API:

- Host = covid19.richdataservices.com
- Port = 443

![Charles → Proxy → SSL Proxying Settings → SSL Proxying → Edit location](/assets/2021-09-23/04-postman-settings-proxy-tab.png)

_Charles → Proxy → SSL Proxying Settings → SSL Proxying → Edit location_

## Setup Postman for Charles

1\. Go to SETTINGS → «Proxy» tab and check: _Add a custom proxy configuration, HTTP_ and _HTTPS_ boxes. Fill the field _Proxy Server_ = 127.0.0.1:8888 (port should be the same as in Charles).

![Postman → SETTINGS → Proxy tab](/assets/2021-09-23/04-postman-settings-proxy-tab.png)

_Postman → SETTINGS → Proxy tab_

2\. Go to the «General» tab and turn off _SSL certificate verification_.

![Postman → SETTINGS → General tab](/assets/2021-09-23/05-postman-settings-general-tab.png)

_Postman → SETTINGS → General tab_

## Sniff Traffic

1\. Send a GET [request](https://covid19.richdataservices.com/rds/api/catalog/int/jhu_country/metadata/json) in Postman (Charles should be started too). If the settings are made correctly, you will receive a response.

![Postman’s request](/assets/2021-09-23/06-postmans-request.png)

_Postman’s request_

2\. Open Charles. You will see an inspectable request from Postman.

![Charles’ Overview](/assets/2021-09-23/07-charles-overview.png)

_Charles’ Overview_

![Charles’ Contents](/assets/2021-09-23/08-charles-contents.png)

_Charles’ Contents_

This method will help to debug requests and responses if you doubt the consistency of headers transmitted by Postman.

It seems that Postman does not add anything extra to the request, and [cURL](https://curl.se/) is quite consistent with it.

![Comparison of requests in Postman and Charles](/assets/2021-09-23/09-comparison-of-requests.png)

_Comparison of requests in Postman and Charles_

Do not forget to turn off the proxy settings in Postman when Charles is not running.

Copy @ [Medium](https://adequatica.medium.com/scan-postman-traffic-through-charles-c266cb97914c)
