---
layout: post
title: "Security Testing for Manual Testers"
date: 2022-05-06 16:52:12 +0400
tags: security testing xss
---

Sometimes, as a test engineer, you need to check that your web application is not vulnerable.

I am not going to describe what [security testing](https://en.wikipedia.org/wiki/Security_testing) is because there are many introductory articles about it. In some companies, security testing or [vulnerability assessment](<https://en.wikipedia.org/wiki/Vulnerability_assessment_(computing)>) is performed by penetration testers from a special team or by the information security department, but it happens that such kind of activities have to be done by ordinary manual testers.

I would like to list **practical actions** of a brief security testing that **can be performed manually and right away without additional tooling**.

The main testing landmarks are:

- HTTPS
- XSS (Cross-site scripting)
- SQL Injection
- CSP (Content Security Policy)
- CORS (Cross-Origin Resource Sharing)
- CSRF (Cross-Site Request Forgery). Here, in terms of defense
- Open Redirect

## HTTPS

Manual testing of your web application over [HTTPS](https://en.wikipedia.org/wiki/HTTPS) means **performing a regression run with HTTPS in URLs**.

If all functionality works as expected, then it is OK, but here are a few tips:

1\. **URLs with HTTP protocol should redirect to HTTPS.** This is a server’s side setting and a part of nonfunctional requirements — ask your developers about it.

If there is no redirect, then use a browser extension for automatic redirection from HTTP to HTTPS:

- [HTTPS Everywhere](https://addons.mozilla.org/en-US/firefox/addon/https-everywhere/) or [Smart HTTPS](https://addons.mozilla.org/en-US/firefox/addon/smart-https-revived/) for Firefox;
- [HTTPS Everywhere](https://chrome.google.com/webstore/detail/https-everywhere/gcbommkclmclpchllfjekcdonpmejbdp) for Chrome.

Some browsers (Firefox since [version 83](https://blog.mozilla.org/security/2020/11/17/firefox-83-introduces-https-only-mode/) and since [version 91 in Private mode](https://blog.mozilla.org/security/2021/08/10/firefox-91-introduces-https-by-default-in-private-browsing/)) do it by default without any extensions.

2\. **Check the certificate for validity.**

Firefox: under the «protection» icon in the URL bar (since [version 70, it is not colored green](https://blog.mozilla.org/security/2019/10/15/improved-security-and-privacy-indicators-in-firefox-70/)):

![Firefox Connection security](/assets/2022-05-06/01-https-firefox.png)

_Fig. 1. Firefox Connection security_

Chrome: under the same «protection» icon or in the [Security Panel in DevTools](https://blog.chromium.org/2016/01/introducing-security-panel-in-devtools.html):

![Chrome DevTools Security Panel](/assets/2022-05-06/02-https-chrome.png)

_Fig. 2. Chrome DevTools Security Panel_

3\. **Watch out for mixed content!**

[All your data should be loaded over HTTPS](https://web.dev/what-is-mixed-content/). If your page is loaded over secure HTTPS, but other resources (images, videos, scripts) are loaded over an insecure HTTP, you’ll be notified about it in a browser’s Console. In some cases, «[mixed content](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content)» may be the reason that content may not be displayed on the web page.

4\. **All URLs inside `<head>` tag should have HTTPS protocol.**

Check `href`, `content`, and `src` attributes in `<link>`, `<meta>`, and `<script>` tags:

![All URLs inside <head> tag have HTTPS protocol](/assets/2022-05-06/03-https-head.png)

_Fig. 3. All URLs inside `<head>` tag have HTTPS protocol_

5\. **All sharing widgets should provide HTTPS URLs.**

Check pop-ups of Facebook, Twitter, or «Copy link» buttons: all shared links should have HTTPS protocol. Click on it and check what link is going to be shared.

![Twitter’s widget shares HTTPS link](/assets/2022-05-06/04-https-shares.png)

_Fig. 4. Twitter’s widget shares HTTPS link_

Further reading:

- [Why HTTPS matters](https://web.dev/why-https-matters/);
- [The Complete Guide To Switching From HTTP To HTTPS](https://www.smashingmagazine.com/2017/06/guide-switching-http-https/).

## XSS (Cross-site scripting)

[Cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting) is the most common security vulnerability that can be found in web applications. It enables the attacker to put his client-side script into a web page, and this script will be executed when another user opens a spoofed page.

![Examples of XSS execution](/assets/2022-05-06/05-xss-example.png)

_Fig. 5. Examples of XSS execution_

![Examples of XSS execution](/assets/2022-05-06/06-xss-example.png)

_Fig. 6. Examples of XSS execution_

The most common types of XSS are:

- [Reflected XSS](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.html), when script executes immediately (in search suggestions or by following a link with XSS in URL);
- [Stored XSS](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/02-Testing_for_Stored_Cross_Site_Scripting.html), when script saves (as a user’s comment) and executes when another user opens the page.

For both types of XSS we will test how developers coped with escaping special special characters to prevent XSS attacks. Some developers argue that [escaping](https://javascript.info/regexp-escaping) should be done on frontend, and others argue that it should be done on backend, but for safety, it must be done both on frontend and backend.

Based on the above, the principles of testing XSS are:

1. **Take the XSS vector → put it in all forms and inputs → press [Enter] → see what happens;**
2. **Take the XSS vector → put it URL → press [Enter] → see what happens.**

The main thing in these cases is to have a set of universal XSS vectors, for example:

- `{% raw %}&lt;XSS>'':!--"&lt;hr/>&{()}{% endraw %}`

- `{% raw %}&lt;img/src='x'onerror=alert(1)>{% endraw %}`

- `{% raw %}"&lt;img/src='x'onerror=alert(1)>"@gmail.com{% endraw %}`

- `{% raw %}&lt;%2Ftitle>&lt;svg%2Fonload%3Dprompt'XSS'>{% endraw %}`

- `{% raw %}javascript:/*-->&lt;/marquee>&lt;/script>&lt;/title>&lt;/textarea>&lt;/noscript>&lt;/style>&lt;/xmp>">[img=1]&lt;img -/style=-=expression&#40&#47;&#42;'/-/*&#39;,/**/eval(name)//&#41;;width:100%;height:100%;position:absolute;behavior:url(#default#VML);-o-link:javascript:eval(title);-o-link-source:current name=alert(1) onerror=eval(name) src=1 autofocus onfocus=eval(name) onclick=eval(name) onmouseover=eval(name) background=javascript:eval(name)//>"{% endraw %}`

- For URL: `{% raw %}#'">&lt;script>prompt(document.domain)&lt;/script>1{% endraw %}`

More examples:

- [XSS Filter Evasion Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html);
- [Cross-site scripting (XSS) cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet).

If your web application is based on a popular UI framework, then with a high probability, it has protection for XSS attacks «out of the box», but the manual check will not be superfluous.

3\. **Check for an X-XSS-Protection header.**

[This header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) stops pages from loading when they detect reflected cross-site scripting attacks. This feature only partially protects against XSS attacks because [it is supported only by a few browsers](https://caniuse.com/mdn-http_headers_x-xss-protection). At least, expect to see (or ask developers to implement) this value:

```
x-xss-protection: 1; mode=block
```

Further reading:

- [XSS Game](https://xss-game.appspot.com/);
- [What is the http-header "X-XSS-Protection"](https://stackoverflow.com/questions/9090577/what-is-the-http-header-x-xss-protection)?
- [XSS Prevention Cheat Sheet for Penetration Testers](https://www.hackingloops.com/xss-prevention-cheat-sheet-for-penetration-testers/).

## SQL Injection

For brief manual testing without special knowledge, [SQL injection](https://en.wikipedia.org/wiki/SQL_injection) may be the same as XSS.

You should put [SQL injection payloads](https://github.com/payloadbox/sql-injection-payload-list) in the forms and inputs in the hope they will not crash your web application.

Further reading:

- [SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) and [SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html);
- [SQL injection](https://portswigger.net/web-security/sql-injection) and [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet);
- [SQL Injection Cheat Sheet](https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/).

## CSP (Content Security Policy)

According to [Content Security Policy](https://developers.google.com/web/fundamentals/security/csp), the source code on the site could be loaded only by the site itself ([same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy)) or from allowed sites (resources). This is one of the ways to protect against XSS attacks — if the attacker somehow implemented a link to his script into your site, this script will not be loaded from the attacker’s resource because this resource is not in the allowlist.

For example, scripts, images, and other content on `https://developer.mozilla.org` can be loaded only from `https://developer.mozilla.org` OR from the allowed sites.

The allowlist of [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) resources can be found in response headers:

![Content-Security-Policy response header](/assets/2022-05-06/07-csp-response-header.png)

_Fig. 7. Content-Security-Policy response header_

The testing vectors are:

1. **All allowed resources are loading;**
2. **Watch out for errors of blocked content in a browser’s Console.**

Common case: developers forget to put some allowed resources in a white list, and due to this, this resource does not load.

![Content Security Policy](/assets/2022-05-06/08-csp-blocked.png)

_Fig. 8. Content Security Policy: The page’s settings blocked the loading of the resource_

3. **All data from not allowed resources is blocked.**

To test that third-party resource does not load, you can run this script in the browser’s Console:

```
fetch("https://example.com/").then(response => response.json()).then(data => console.log(data));
```

If you see the Content Security Policy error: «_The page’s settings blocked the loading of the resource…_» — then the test passed.

![Content Security Policy](/assets/2022-05-06/09-csp-blocked.png)

_Fig. 9. Content Security Policy: The page’s settings blocked the loading of the resource_

## CORS (Cross-Origin Resource Sharing)

According to [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (based on HTTP-headers), a site can give permission to third-party sites to have access to its content.

How to test:

1\. **Make an HTTP request:**

```
curl -i 'https://example.com/' -H 'Origin: http://evil.com/'
```

If you will see the Access-Control-Allow-Origin response header — it is a bug. It means that our site (`https://example.com`) gives access to an absolutely unknown resource.

2\. **Watch out for errors of blocked content in a browser’s Console:**

![Cross-Origin Request Blocked](/assets/2022-05-06/10-cors-blocked.png)

_Fig. 10. Cross-Origin Request Blocked_

Further reading:

- [A quick introduction to web security](https://www.freecodecamp.org/news/a-quick-introduction-to-web-security-f90beaf4dd41);
- [What is the difference between CORS and CSPs](https://stackoverflow.com/questions/39488241/what-is-the-difference-between-cors-and-csps)?

## CSRF (Cross-Site Request Forgery). Here, in terms of defense

[CSRF](https://developer.mozilla.org/en-US/docs/Glossary/CSRF) is a name for a type of attack when an unauthorized command is submitted from a trusted user. In most cases, this command includes in the URL as a malicious parameter, for example:

```
https://example.com/?action=delete_session_id&id=123
```

[To prevent such kinds of actions](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html), sites can implement unique tokens in each truthful request (in GET parameter, in the header, or as a cookie), for example:

```
curl -X POST 'https://example.com/' -H 'X-CSRF-Token: pebesbmvvrac9r7ieaq0rxf5sqz49ti8'
```

There are two test cases:

1\. If you know that your site has CSRF tokens, you can:

**Make a request without a CSRF token or with a spoiled CSRF token** — in this case, you should not get a valid response.

2\. Or, if your site does not have CSRF tokens, you still can **try to make an unauthorized request**:

Steps:

1\. Find POST request on your site;

2\. Make an HTML file with a button that will run this request, but without any headers:

```
<html><head></head><body>
<form method="POST" action="https://accounts.firefox.com/metrics">
<input type="submit" name="button" value="OK"></form>
</body></html>

```

3\. Open your HTML file and click on the button (run the request).

If you see response code = 400, 401, 403, or 404 — it is OK, at least your site does not allow unauthorized requests.

![Bad Request](/assets/2022-05-06/11-csrf-bad-request.gif)

_Fig. 11. Bad Request_

Further reading:

- [Cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery);
- [What is a CSRF token and how does it work](https://brightsec.com/blog/csrf-token/)?

## Open Redirect

If your web application supports Open Redirect, then it has potentially vulnerable functionality.

[How does it work](https://www.acunetix.com/blog/web-security-zone/what-are-open-redirects/)? If you pass in the URL a GET parameter with the link, then a passed link will be opened. The GET parameter for redirection in URL could be arbitrary (as developers decide, usually: `url`, `link`, `retpath`, or `continue`), for example:

```
https://example.com/destination?retpath=http://example.com/redirection
```

Instead of open `/destination`, it will redirect to `/redirection`.

Open Redirect is commonly used during authorization when the user should be returned to the original page after logging in ⇒ the Open Redirect links [should be strictly validated](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html) to prevent redirects to third-party sites.

The testing vectors are:

1. **Redirect works only for allowed sites** (ideally, only for URLs of the same domain);
2. **Redirect does not work for third-party sites.**

If you try to open `https://example.com/?url=https://mozilla.org`, only example.com site should open:

![No redirection through GET parameter in URL](/assets/2022-05-06/12-open-redirect.png)

_Fig. 12. No redirection through GET parameter in URL_

---

As I pointed out at the beginning, I have listed only brief checks for manual execution. Of course, there are a bunch of tools for deep security testing, like [Burp Suite](https://portswigger.net/burp).

Further reading:

- [Types of attacks](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks);
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/stable/).

Copy @ [Medium](https://systemweakness.com/a-brief-security-testing-for-manual-testers-f2f59d56fbb5)
