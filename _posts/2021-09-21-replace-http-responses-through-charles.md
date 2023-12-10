---
layout: post
title: "Replace HTTP Responses Through Charles"
date: 2021-09-21 09:50:18 +0300
tags: testing charles
---

Sometimes, as a test engineer, you need to rewrite HTTP/HTTPS requests for your web application

A default browser’s developer tools do not support a replacement of requests or responses, but third-party HTTP proxies, like [Charles](https://www.charlesproxy.com/), do. Unfortunately, Charles’ interface is not obvious to users who need this operation a couple of times a year.

![Test page: Charleroi](/assets/2021-09-21/01-test-page.png)

_[Test page: Charleroi](https://openweathermap.org/city/2800482)_

Disclaimers:

- There are a few different approaches to change response in Charles. I will cover only [Rewrite Tool](https://www.charlesproxy.com/documentation/tools/rewrite/);
- Your client (site) should have open backend requests. If your web application has a server side rendering, then most of the requests will be hidden;
- As an example, the page of a [city on OpenWeather](https://openweathermap.org/city/2800482) was selected because all requests are inspectable in the browser’s developer tools (Network tab);
- I use Charles 4.5.6 on macOS Catalina 10.15.7.

## Set up Charles

Fortunately, a [free version of Charles](https://www.charlesproxy.com/download/) is enough to complete our discovery.

1\. At the first boot, you will be prompted with a message for automatically configuring network settings. Do not hesitate to click [Grant privileges].

![Automatic macOS Proxy Proxy Configuration](/assets/2021-09-21/02-automatic-configuration.png)

_Automatic macOS Proxy Proxy Configuration_

2\. Then go to _Help → SSL Proxying → Install Charles Root Certificate_ and add the certificate to Keychain Access.

![Install Charles Root Certificate](/assets/2021-09-21/03-install-root-certificate.png)

_Install Charles Root Certificate_

3\. Then go to Keychain Access, find the recently added Charles’ certificate, double click to open the certificate window, and make it trusted — change the Trust settings to _Always Trust_.

![Always Trust to Charles Proxy CA](/assets/2021-09-21/04-always-trust.png)

_Always Trust to Charles Proxy CA_

4\. Then go to _Proxy → SSL Proxying Settings…_ and check the box _Enable SSL Proxying_ in the «SSL Proxying Setting» window.

![SSL Proxying Settings](/assets/2021-09-21/05-ssl-proxy-settings.png)

_SSL Proxying Settings_

5\. Restart Charles.

6\. Finally, turn on _Proxy → macOS Proxy._

![macOS Proxy](/assets/2021-09-21/06-macos-proxy.png)

_macOS Proxy_

## Inspect Traffic

1\. Reload the website in a browser. You will see multiple requests in Charles.

2\. Choose the required domain and select _Enable SSL Proxying_ in the menu.

![Enable SSL Proxying](/assets/2021-09-21/07-enable-ssl-proxing.png)

_Enable SSL Proxying_

3\. Reload the website in a browser. Now, you will see fully inspectable requests in this domain.

![Request overview](/assets/2021-09-21/08-request-overview.png)

_Request overview_

## Replace Responses

### Replace Status Code

Let’s try to break the web page by an error code in response.

1\. Choose an inspectable request and select _Tools → Rewrite…_

![Rewrite](/assets/2021-09-21/09-rewrite.png)

_Rewrite…_

2\. Click [Add] in the «Rewrite Setting» window and click [Add] again to fill the location — it is a pattern of the request:

```
Protocol = https

Host = openweathermap.org

Port = *

Path = /data/2.5/onecall

Query = *
```

![Edit Location](/assets/2021-09-21/10-edit-location.png)

_Edit Location_

3\. Now click the last [Add] to fill the rewrite rule and choose type = _Response Status_. Specify which value should be replaced (Match Value = 200) with a new one (Replace Value = 500). Click [OK] and close the window.

![Rewrite rule](/assets/2021-09-21/11-rewrite-rule.png)

_Rewrite rule_

4\. Reload the website. The web page crashed because the endpoint with data responded 500 — this is what we expected.

![Status Code: 500](/assets/2021-09-21/12-status-code.png)

_Status Code: 500_

### Replace Body

Let’s change the temperature as an arbitrary part of the response body.

1\. In the «Rewrite Rule» window, choose type = _Body_ and mark the checkbox _Response_. Specify which value should be replaced (Match Value = `24.`) with a new one (Replace Value = `35.`). Click [OK] and close the window.

![_Rewrite rule](/assets/2021-09-21/13-rewrite-rule.png)

_Rewrite rule_

2\. Reload the website. The temperature has changed to a substitute value because the endpoint with data responded with an artificial body — this is what we expected.

![Dev Tools → Network → request Preview](/assets/2021-09-21/14-dev-tools-network-request-preview.png)

_Dev Tools → Network → request Preview_

For more examples of how to use Charles in this case you can follow this links:

- [How to use Charles Proxy to rewrite HTTPS traffic for web applications](https://deliveroo.engineering/2018/12/04/how-to-use-charles-proxy-to-rewrite-https-traffic-for-web-applications.html);
- [How to change response body with Charles](https://stackoverflow.com/questions/31681518/how-to-change-response-body-with-charles)?

Copy @ [Medium](https://adequatica.medium.com/replace-http-responses-through-charles-f2954c372b40)
