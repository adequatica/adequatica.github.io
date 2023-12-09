---
layout: post
title: "Vectors of Testing Pop-up Overlays"
date: 2020-09-04 19:43:27 +0300
tags: testing
---

How to manually test modal windows.

A few years ago, I was working on a web project full of pop-ups. After this experience, I collected a list of insights for testing pop-up overlays or [modal](https://dribbble.com/tags/modal) windows.

![A perfect example of a pop-up overlay](/assets/2020-09-04/00-example.jpg)

_A perfect example of a pop-up overlay (modal window)_

### 1. Pay attention to the location of the X close cross

(_Set by the design._) If the X close cross is outside of the pop-up area — it’s a 100% identifier of the complete closing of the pop-up. But in some rare cases, when it’s located inside the pop-up, it can close interactive content inside the pop-up (of course, it’s better to avoid such kind of UX). The main point is to make sure that it’s easy and obvious to dismiss the pop-up (modal window).

![Outside X close cross — OK](/assets/2020-09-04/01-outside-x.png)

_Fig. 1. Outside X close cross — OK_

![Inside X close cross — OK](/assets/2020-09-04/02-inside-x.png)

_Fig. 2. Inside X close cross — OK_

![This is unobvious. Does X cross close an interactive content or the whole pop-up?](/assets/2020-09-04/03-unobvious-x.png)

_Fig. 3. This is unobvious. Does X cross close an interactive content or the whole pop-up?_

### 2. Close pop-up in different ways

- X close cross;
- [ESC] button;
- Click on overlay out of the pop-up — this case is set by the design and marketing. It’s preferable to give a user the ability to close pop-up in any way, but some managers are afraid to reduce the conversion rate if users will have more room for rejection.

### 3. Scroll inside pop-up

(_Set by the design and defined by the amount of content._) All content should fit into the pop-ups area or be able to view. There are a few common bugs related to this case:

- Content falls out of the pop-up area;
- Content is not scrollable inside the pop-up.

![Text falls out of the pop-up area](/assets/2020-09-04/04-bug.png)

_Fig. 4. Bug: text falls out of the pop-up area_

![The text is cropped, and there is no way to scroll it](/assets/2020-09-04/05-bug.png)

_Fig. 5. Bug: the text is cropped, and there is no way to scroll it_

### 4. Maximum scroll down to the bottom

Overlay or «vail» should not pull away from the screen.

![Overlay was not fixed](/assets/2020-09-04/04-bug.png)

_Fig. 6. Bug: overlay was not fixed_

### 5. Background scroll behind overlay

(_Set by the design._) Despite the fact that scrollability is determined by the design, it’s preferable to lock a page scroll of background during the opened pop-up, because it could prevent possible problems after closing the pop-up.

### 6. Browser scrollbar during opened pop-up

Auto-hide of the browser scrollbar may lead to twitching of the whole page. It’s better to have a useless scrollbar, than an irritable user experience.

![Twitching content](/assets/2020-09-04/07-twitching.gif)

_Fig. 7. Bug: twitching content_

### 7. Open pop-up with and without a connected mouse

Connected device (mouse) effects on the display of the scrollbar!

### 8. Open pop-up from different parts of the page

In some cases, opening a pop-up from the bottom of the page could lead to scrolling of the entire page to the top on the background.

### 9. Pay attention to the place of opening/closing of pop-up on the page

The viewport of the main page should not change after opening/closing of the pop-up.

### 10. Pop-up content should be idempotent

([Idempotence](https://en.wikipedia.org/wiki/Idempotence) is set by the documentation.) In a perfect world, repeatable operation of «open > close > open» should lead to the same result.

### 11. Try to search for something on the page

The browser will search everything on the page (inside pop-up and under overlay), and the overlay with pop-up should not break if something is found in the background.

![CTRL+F highlights found keywords](/assets/2020-09-04/08-ctrl-f.png)

_Fig. 8. CTRL+F highlights found keywords_

### 12. Multiple pop-ups on the page at the same time

(_Prioritization is set by documentation._) If a few pop-ups could be opened simultaneously, this case must be tested. The display priority is usually set by [z-index](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index), and it’s a very fragile piece.

![Bug: minor pop-up overlays major pop-up](/assets/2020-09-04/09-bug.png)

_Fig. 9. Bug: minor pop-up overlays major pop-up_

### 13. Dropdowns with pop-up

Try to combine opened [dropdown menus](https://dribbble.com/tags/dropdown_menu) with pop-up. It’s a very frequent case when dropdown selects turn out on top of the overlay.

![Bug: dropdown overlays pop-up](/assets/2020-09-04/10-bug.png)

_Fig. 10. Bug: dropdown overlays pop-up_

### 14. Open pop-up by direct link

(_Set by the design or defined by the documentation._) The most common ways of opening pop-up by direct link you could face are:

- **[Path](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_URL):** http://foo.bar/pop-up
- **[Query string](https://en.wikipedia.org/wiki/Query_string):** http://foo.bar/xyzzy?pop-up=true
- **URL with [UTM parameters](https://en.wikipedia.org/wiki/UTM_parameters):** http://foo.bar/pop-up?utm_source=medium — special case of a query string, but make sure the parameters don’t disappear after opening;
- **[Hash](https://developer.mozilla.org/en-US/docs/Web/API/URL/hash) parameter:** http://foo.bar/xyzzy#pop-up — this is an edge case for [Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=645075).

### 15. Navigation by [Back] browser button

(_Set by the design and the ability to work with direct links._) The description of how this should work is almost always absent in documentation. What should happen when we click [Back] after opening the pop-up? Should it close pop-up or bring back a previous page? — opening a previous page is more expected in terms of global browser behavior, but it’s always discussible and depends on the particular UI.

### 16. Keyboard navigation

Try to open pop-up by keyboard keys and interact with it by [TAB] and [ENTER]. The main problem of this case is that developers forget to switch focus to the opened pop-up, and a user navigation focus stays under pup-up. It’s an [accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility) issue that corresponds to WCAG criteria [2.1.1 Keyboard](https://www.w3.org/TR/WCAG21/#keyboard) and [2.1.2 No Keyboard Trap](https://www.w3.org/TR/WCAG21/#no-keyboard-trap).

### 17. Compare pop-up with design mockup

(_Set by the design._) Dimensions of pop-up should match the mockup. The percentage of the overlay’s transparency should match the mockup — the transparency could be set in different ways. For example, by [opacity](https://developer.mozilla.org/en-US/docs/Web/CSS/opacity). The same opacity in the browser may differ from the same value in the designer’s mockup and depends on the graphics editor ([Sketch](https://www.sketch.com/) and [Figma](https://www.figma.com/) are preferable to avoid such collisions).

![Compare different opacity](/assets/2020-09-04/11-compare-different-opacity.png)

_Fig. 11. Compare different opacity: 70% vs 80%_

By the way, the visual percentage of opacity can solve the case when two pop-ups are opened simultaneously on top of each other — a rear pop-up will be hidden, but the decrease in transparency due to two overlays will reveal it.

### 18. Check touch version on a physical mobile device

Sometimes responsive design mode in desktop browsers may not show the real pop-up’s behavior on a touch device.

### 19. Check in incognito mode (private window) in Safari

This is the edge case. If you have an external widget (or iframe) that requires authorization inside the pop-up, most likely, there will be [problems](https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/).

### 20. Check in IE 11 and Edge 14/15

Only if you have a significant amount of users in this browser.

---

The above list is not complete and can be supplemented with new cases depending on the context of the particular project.

Free fonts used in illustrations: [Calluna](https://www.myfonts.com/fonts/exljbris/calluna/) and [Calluna Sans](https://www.myfonts.com/fonts/exljbris/calluna-sans/).

Copy @ [Medium](https://adequatica.medium.com/vectors-of-testing-pop-up-overlays-modal-windows-65c2ede32344)
