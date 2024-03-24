---
layout: post
title: "Accessibility Manual Testing"
date: 2021-08-29 07:16:23 +0300
tags: accessibility testing
---

Step-by-step guide for testers on how to deal with the accessibility.

There are many articles for web developers about accessibility and hundreds of pages of detailed documentation. But when the task of accessibility testing turns out to be on the side of the Quality Assurance department, this information should be transformed into the requirements of users with disabilities.

This article covers some testing vectors if a test engineer has to perform an accessibility audit.

Disclaimers:

- I assume that the reader is familiar with the concept of [accessibility](https://en.wikipedia.org/wiki/Accessibility) (A11y) and technologies of its implementation (see references at the end of the article);
- The article does not cover automated testing;
- The basic checks listed below execute without using a [screen reader](https://en.wikipedia.org/wiki/Screen_reader);
- As an example, [a page of an album on Yandex Music](https://music.yandex.com/album/10575898) was selected.

![Test page](/assets/2021-08-29/00-test-page.png)

_[Test page](https://music.yandex.com/album/10575898)_

## Keyboard Navigation

The first set of tests is aimed at what can be seen with the eyes and reproduced without additional tools.

### 1. Focus on Interactive Elements

[Try to navigate through the page by TAB key](https://developers.google.com/web/fundamentals/accessibility/how-to-review#start_with_the_keyboard). Pay attention to the selection of active elements during navigation — interactive elements should be focusable. Users should clearly understand where they are on the site.

Actual result:

- All buttons and links do not have focus;
- Most of the elements have the property outline: 0px, [which is an antipattern](http://www.outlinenone.com/).

![Focus on interactive elements](/assets/2021-08-29/01-1-focus-on-interactive-elements.png)

_1.1. Focus on interactive elements — there is no focus on the marked items, the users do not understand where they are located_

Expected: if the element is interactable, it will have a visible indication.

![Focus on interactive elements](/assets/2021-08-29/01-2-focus-on-interactive-elements.png)

_1.2. Focus on interactive elements — the [contemporary jazz] link has a default focus style_

Wherefore? The highlighted focus is visual feedback for the users.

This is 2.4.7 «[Focus Visible](https://webaim.org/standards/wcag/checklist#sc2.4.7)» WCAG requirement ([more info](https://www.w3.org/WAI/WCAG21/Understanding/focus-visible.html)).

It is recommended to provide an obvious focus style (especially if the default one is removed):

- [Accessibility concerns outline](https://developer.mozilla.org/en-US/docs/Web/CSS/outline#Accessibility_concerns);
- [Introduction to Focus](https://developers.google.com/web/fundamentals/accessibility/focus).

### 2. Sequence of Elements During Tabbing

[Navigate through the page by TAB key](https://developers.google.com/web/fundamentals/accessibility/how-to-review#start_with_the_keyboard). Interactive elements should be selected from left to right and from top to bottom.

The actual result matches with expected: navigation takes place in a logical sequence. The visual order of the elements corresponds to the real order in the DOM-tree and [CSS properties do not change the order of elements](https://developers.google.com/web/fundamentals/accessibility/focus/dom-order-matters).

This is 1.3.2 «[Meaningful Sequence](https://webaim.org/standards/wcag/checklist#sc1.3.2)» and 2.4.3 «[Focus order](https://webaim.org/standards/wcag/checklist#sc2.4.3)» WCAG requirements ([more info](https://www.w3.org/WAI/WCAG21/Understanding/focus-order.html)).

### 3. TAB Order

[Navigate through the page by TAB key](https://developers.google.com/web/fundamentals/accessibility/how-to-review#start_with_the_keyboard). All [interactive content](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Content_categories#Interactive_content) should be intractable.

Actual result:

- Search button is missing for keyboard navigation. Users without a mouse or touchpad are unable to make a search request on the site.
- It is impossible to play an arbitrary track from the album. Interactive elements ([play], [like], and [dislike] buttons) are not a part of the tab order, and they appear only when the user hovers a mouse over them.

![TAB order](/assets/2021-08-29/03-tab-order.png)

_3. TAB order — [play] buttons are not available for keyboard navigation_

Expected: interactive content is intractable.

Wherefore? To use all functions of the website.

This is 2.1.1 «[Keyboard](https://webaim.org/standards/wcag/checklist#sc2.1.1)» WCAG requirement ([more info](https://www.w3.org/WAI/WCAG21/Understanding/keyboard.html)).

Elements can be included or excluded from the tab order by [tabindex value](https://developers.google.com/web/fundamentals/accessibility/focus/using-tabindex).

## 4. Tabindex Values

Check the values of the tabindex attributes in the page’s source code.

Actual result: multiple elements with tabindex > 0.

Expected: the only one element with tabindex > 0.

Wherefore? In order not to confuse the browser or to specify the first focus element (when using only one tabindex = 1).

Despite that the tabindex can be greater than 0 ([but not exceed 32767](https://www.w3.org/WAI/WCAG21/Techniques/html/H4.html)), it is recommended to have a single element with tabindex > 0, because [it will be the first one to receive focus from the keyboard by TAB](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex).

## Semantics

The second set of tests is aimed at how the site is ready to work with [assistive technologies](https://www.who.int/news-room/fact-sheets/detail/assistive-technology).

### 5. Language Declaration

Yandex Music has [localisation](https://en.wikipedia.org/wiki/Language_localisation) — the page language could be changed into Russian, Armenian, Uzbek, Kazakh, Ukrainian, or Azerbaijani. Therefore, you need to check [the declaration of the page language](https://www.w3.org/International/questions/qa-html-language-declarations).

Actual result: `<html>` tag does not have any attributes.

Expected: `<html lang="en">`.

Wherefore? To help a speech synthesizer accurately determine the language of the page.

Each language locale must correspond to the value from the [IANA registry](https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry): ru, hy, uz, kk, uk, or azb.

## 6. Document Structure

Check for the appropriate [section heading elements](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements).

The actual result matches with expected: titles are marked with the correct tags.

![Document structure](/assets/2021-08-29/06-document-structure.png)

_6. Document structure — the main title is `<h1>Round one</h1>`_

Wherefore? [A well-thought-out structure of the document](https://developers.google.com/web/fundamentals/accessibility/semantics-builtin) will help users to navigate through the page, because screen readers allow fast switching between headings.

This is 1.3.1 «[Info and Relationships](https://webaim.org/standards/wcag/checklist#sc1.3.1)» WCAG requirement ([more info](https://www.w3.org/WAI/WCAG21/Understanding/info-and-relationships.html)).

If the layout consists of only `<div>`s, it is still possible to include them into [A11y-tree](https://developers.google.com/web/fundamentals/accessibility/semantics-builtin/the-accessibility-tree) by adding the corresponding [ARIA roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/heading_role): `role= "heading" aria-level= "1"`.

### 7. Landmarks

Check for [ARIA Landmark Roles](https://www.w3.org/TR/wai-aria-1.2/#landmark_roles) attributes in the page’s source code.

Actual result: nothing.

Expected: page areas are marked up by the main ARIA roles in accordance with their content: [banner, complementary, contentinfo, main, navigation, search](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques#Landmark_roles).

![Landmarks](/assets/2021-08-29/07-landmarks.png)

_7. Landmarks — example of ARIA Landmarks Roles_

Wherefore? Landmarks, same as semantic headers, will help users to navigate through the page, because screen readers allow fast switching between landmarks.

This is 1.3.6 «[Identify Purpose](https://webaim.org/standards/wcag/checklist#sc1.3.6)» WCAG requirement ([more info](https://www.w3.org/WAI/WCAG21/Understanding/identify-purpose.html)).

Landmarks determine the purpose of the page areas and, in a way, duplicate [content sectioning HTML5 elements](https://developer.mozilla.org/en-US/docs/Web/HTML/Element#Content_sectioning): `<header>`, `<aside>`, `<footer>`, `<main>`, `<nav>`. However, it is still necessary to specify HTML5 tags by additional ARAI roles to support older browsers and [bypass bugs](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/footer#accessibility_concerns).

### 8. Interactive Content Description

Check the presence and clarity of the description of interactive elements (buttons and links). It is useful to try a screen reader for a better understanding of how the text will be read.

Actual result: there are no descriptions or they are incomprehensible.

Expected: descriptions reflect the logical meaning of interactive elements. For example:

- Instead of the standard name of the [contemporary jazz] link, it could be `aria-label="Open contemporary jazz genre page"`;
- Instead of the standard name of the [play] button, it could be `aria-label="Play album"`;
- Instead of missing the […] button description, it could be `aria-label="Open album menu"`.

Wherefore?

- When the interface is not visible, it can and should be described: what exactly the buttons and links do;
- Aria-label attribute is a text [which will be read by a screen reader](https://developers.google.com/web/fundamentals/accessibility/semantics-aria/aria-labels-and-relationships).

This is 2.4.4 «[Link Purpose (In Context)](https://webaim.org/standards/wcag/checklist#sc2.4.4)» WCAG requirement ([more info](https://www.w3.org/WAI/WCAG21/Understanding/link-purpose-in-context.html)).

### 9. Image Description

Check for alt attribute for images.

Actual result: images do not have alternative text.

![Image description](/assets/2021-08-29/09-image-description.png)

_9. Image description — `<img>` tag should have alt attribute, but it does not_

Expected:

- Image has attribute `alt="Album cover"`;
- Site logo has a title. Despite the fact that the logo is not being placed through `<img>` tag, [the link should have an alternative text](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Understanding_WCAG/Text_labels_and_names#Interactive_elements_must_be_labeled): `aria-label="Yandex Music"`.

Wherefore? It is necessary to provide a descriptive alternative text for contextual images, so that the users with a screen reader can get an idea of the images or image buttons.

This is 1.1.1 «[Non-text Content](https://webaim.org/standards/wcag/checklist#sc1.1.1)» WCAG requirement ([more info](https://www.w3.org/WAI/WCAG21/Understanding/non-text-content.html)).

### 10. Color Vision

Check:

- Displaying the site in simulating color blindness modes by using [color vision simulation built-in Firefox](https://developer.mozilla.org/en-US/docs/Tools/Accessibility_inspector/Simulation) or by third-party browser extension (see references at the end of the article);
- Hover on links in each of the color blindness modes;
- Text contrast ratio by using [Accessibility Inspector in Firefox](https://developer.mozilla.org/en-US/docs/Tools/Accessibility_inspector#Check_for_accessibility_issues).

Actual result:

- Non-contrast text. «ALBUM», «Artist», «2020», and «contemporary jazz» are too light — the color contrast between background and text should be great enough to ensure legibility — [minimum ratio is 4,5:1](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Understanding_WCAG/Perceivable/Color_contrast) ([more info](https://www.getstark.co/blog/accessible-contrast-ratios-and-a-levels-explained));

![Color vision](/assets/2021-08-29/10-1-color-vision.png)

_10.1. Color vision — color contrast should have a minimum ratio 4,5:1, but it does not_

![Contrast issue](/assets/2021-08-29/10-2-contrast-issue.png)

_10.2. Color vision — contrast issue in Firefox Developer Tools_

- Hover and active states of the [contemporary jazz] link are not visible in achromatopsia mode.

![Color blindness](/assets/2021-08-29/10-3-color-blindness.png)

_10.3. Color vision — monochromacy or achromatopsia (color blindness)_

![Blurred vision](/assets/2021-08-29/10-4-blurred-vision.png)

_10.4. Color vision — far-sightedness or blurred vision_

![Absence of red](/assets/2021-08-29/10-5-absence-of-red.png)

_10.5. Color vision — protanopia (absence of red)_

![Absence of blue](/assets/2021-08-29/10-6-absence-of-blue.png)

_10.6. Color vision — tritanopia (absence of blue)_

Expected:

- [Contrast text](https://accessibility.blog.gov.uk/2016/06/17/colour-contrast-why-does-it-matter/);
- The links are distinguishable from the text;
- [Underlined links](https://adrianroselli.com/2019/01/underlines-are-beautiful.html) (at least when hovered, so that they can be easily distinguished from the normal text).

Wherefore?

- In order not to use color exclusively for differences between page elements;
- To take care of [users with color vision deficiency](https://en.wikipedia.org/wiki/Color_blindness) and make the text and links more readable.

This is 1.4.1 «[Use of Color](https://webaim.org/standards/wcag/checklist#sc1.4.1)» and 1.4.3 «[Contrast (Minimum)](https://webaim.org/standards/wcag/checklist#sc1.4.3)» WCAG requirements ([more info](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html)).

### 11. Resize Text

Zoom page to 200%.

![Resize text](/assets/2021-08-29/11-resize-text.png)

_11. Resize text — zoom page to 200%_

The actual result matches with expected: the layout has adapted to the proportions of the text.

This is 1.4.4 «[Resize text](https://webaim.org/standards/wcag/checklist#sc1.4.4)» WCAG requirement ([more info](https://www.w3.org/WAI/WCAG21/Understanding/resize-text.html)).

### 12. Self-check in Lighthouse

Run [Accessibility Audit in Chrome Dev Tools](https://developers.google.com/web/tools/lighthouse).

Look what was missed during manual tests.

![Self-check in Lighthouse](/assets/2021-08-29/12-lighthouse.png)

_11. Self-check in Lighthouse — some of the issues were already covered_

If you do not know what to check, then an automatic audit could be started at the beginning of the testing session. [Accessibility Inspector in Firefox](https://developer.mozilla.org/en-US/docs/Tools/Accessibility_inspector#Features_of_the_Accessibility_panel) is usable to identify A11y issues in specific elements.

For further testing, you should start using a screen reader.

## References

### Similar Articles

- [How To Do an Accessibility Review](https://developers.google.com/web/fundamentals/accessibility/how-to-review)
- [Accessibility audits](https://web.dev/lighthouse-accessibility/)
- [Testing for accessibility](https://www.gov.uk/service-manual/helping-people-to-use-your-service/testing-for-accessibility)
- [Auditing For Accessibility Problems With Firefox Developer Tools](https://hacks.mozilla.org/2019/10/auditing-for-accessibility-problems-with-firefox-developer-tools/)
- [Beyond automatic accessibility testing](https://www.matuzo.at/blog/beyond-automatic-accessibility-testing-6-things-i-check-on-every-website-i-build/)

### Resources

- [The A11Y Project](https://a11yproject.com/)
- [A11y Weekly](https://a11yweekly.com/)
- [Accessible to all](https://web.dev/accessible/)
- [Web Accessibility Checklist](https://webaccessibilitychecklist.com/)
- [Accessibility Developer Guide](https://www.accessibility-developer-guide.com/)
- [Accessibility resources, guides, communities, and more](https://www.getstark.co/library/)
- [Accessibility Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/accessibility/overview/introduction/) — Apple Developer

### Standards

- [Accessibility](https://www.w3.org/standards/webdesign/accessibility)
- [Web Content Accessibility Guidelines (WCAG) 2.1](https://www.w3.org/TR/WCAG21/)
- [Understanding Requirements](https://www.w3.org/TR/UNDERSTANDING-WCAG20/conformance.html) (Level A/AA/AAA)
- [WCAG Checklist](https://webaim.org/standards/wcag/checklist) (WebAIM)

### W3C about ARIA

- [WAI-ARIA Overview](https://www.w3.org/WAI/standards-guidelines/aria/)
- [Accessible Rich Internet Applications (WAI-ARIA)](https://www.w3.org/TR/wai-aria/)
- [Using ARIA](https://www.w3.org/TR/using-aria/)
- [ARIA in HTML](https://www.w3.org/TR/html-aria/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/TR/wai-aria-practices/)
- [ARIA Design Pattern Examples](https://www.w3.org/TR/wai-aria-practices/examples/)

### From MDN

- [Accessibility tutorials](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [WAI-ARIA basics](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/WAI-ARIA_basics)
- [WAI-ARIA Roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles)
- [Using ARIA: Roles, states, and properties](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques)

### From Google

- [Accessibility @ Web Fundamentals](https://developers.google.com/web/fundamentals/accessibility/)
- [Accessibility @ Material Design](https://material.io/design/usability/accessibility.html)
- [Introduction to ARIA](https://developers.google.com/web/fundamentals/accessibility/semantics-aria/)

### Tools

- [Accessibility Inspector](https://developer.mozilla.org/en-US/docs/Tools/Accessibility_inspector) in Firefox Developer Tools
- [Accessibility In Chrome DevTools](https://www.smashingmagazine.com/2020/08/accessibility-chrome-devtools/)
- [axe](https://www.deque.com/axe/)
- [axe-core](https://github.com/dequelabs/axe-core)
- [Testing Website Accessibility with Axe](https://www.youtube.com/watch?v=6gwtqYCxNwQ) and[ WebdriverIO](https://webdriver.io)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse/)

### Screenreaders

- [VoiceOver](https://help.apple.com/voiceover/mac/10.15/) for Mac (part of[ macOS](https://developer.apple.com/accessibility/macos/))
- [Using VoiceOver to Evaluate Web Accessibility](https://webaim.org/articles/voiceover/)
- [NVDA](https://www.nvaccess.org) for Windows (free)
- [JAWS](https://www.freedomscientific.com/products/software/jaws/) for Windows (paid)
- [Orca](https://help.gnome.org/users/orca/stable/) for Linux (Gnome)

### Browser Extensions

- [ChromeVox](http://www.chromevox.com/) — a screen reader for Chrome
- [axe — Web Accessibility Testing](https://chrome.google.com/webstore/detail/axe/lhdoppojpmngadmnindnejefpokejbdd) — accessibility checker for developers
- [Equal Access Accessibility Checker](https://chrome.google.com/webstore/detail/ibm-equal-access-accessib/lkcagbfjnkomcinoddgooolagloogehp) — accessibility checker by IBM
- [Siteimprove Accessibility Checker](https://chrome.google.com/webstore/detail/siteimprove-accessibility/efcfolpjihicnikpmhnmphjhhpiclljc) — evaluates accessibility issues
- [WAVE Evaluation Tool](https://chrome.google.com/webstore/detail/wave-evaluation-tool/jbbplnpkjmmeebjpijfedlgcdilocofh) — evaluates web accessibility
- [Accessibility Insights for Web](https://chrome.google.com/webstore/detail/accessibility-insights-fo/pbjjkligggfmakdaogkfomddhfmpjeni) — highlights accessibility issues
- [Visual ARIA](https://chrome.google.com/webstore/detail/visual-aria/lhbmajchkkmakajkjenkchhnhbadmhmk) — displays ARIA attributes
- [Web Disability Simulator](https://chrome.google.com/webstore/detail/web-disability-simulator/olioanlbgbpmdlgjnnampnnlohigkjla) — simulates color blindness, low vision, dyslexia, and more
- [NoCoffee](https://chrome.google.com/webstore/detail/nocoffee/jjeeggmbnhckmgdhmgdckeigabjfbddl) — simulates visual disturbances associated with fatigue
- [High Contrast](https://chrome.google.com/webstore/detail/high-contrast/djcfdncoelnlbldjfhinnjlhdjlikmph) — changes color contrast
- [Leonardo](https://leonardocolor.io/) — [contrast-based color generator](https://medium.com/@NateBaldwin/leonardo-an-open-source-contrast-based-color-generator-92d61b6521d2)

### Useful Articles

- [101 Digital Accessibility (a11y) tips and tricks](https://dev.to/inhuofficial/101-digital-accessibility-tips-and-tricks-4728)
- [Screen Reader User Survey #9 Results](https://webaim.org/projects/screenreadersurvey9/)
- [Accessible design from the get-go](https://evilmartians.com/chronicles/accessible-design-from-the-get-go)
- [A Complete Guide To Accessibility Tooling](https://www.smashingmagazine.com/2021/06/complete-guide-accessibility-tooling/)
- [A Complete Guide To Accessible Front-End Components](https://www.smashingmagazine.com/2021/03/complete-guide-accessible-front-end-components/)
- [Web Accessible Code Libraries and Design Patterns](http://www.webaxe.org/web-accessible-code-library-design-systems-patterns/)
- [Accessible to some](https://www.matuzo.at/blog/accessible-to-some/)
- [What a Year of Learning and Teaching Accessibility Taught Me](https://www.24a11y.com/2019/what-a-year-of-learning-and-teaching-accessibility-taught-me/)
- [How accessibility trees inform assistive tech](https://hacks.mozilla.org/2019/06/how-accessibility-trees-inform-assistive-tech/)
- [Building the most inaccessible site possible with a perfect Lighthouse score](https://www.matuzo.at/blog/building-the-most-inaccessible-site-possible-with-a-perfect-lighthouse-score/)
- [Pragmatic rules of web accessibility that will stick to your mind](https://www.freecodecamp.org/news/pragmatic-rules-of-web-accessibility-that-will-stick-to-your-mind-9d3eb85a1a28/)
- [Writing HTML with accessibility in mind](https://medium.com/alistapart/writing-html-with-accessibility-in-mind-a62026493412)
- [Things I learned by pretending to be blind for a week](https://silktide.com/blog/2013/things-i-learned-by-pretending-to-be-blind-for-a-week)
- [What we found when we tested tools on the world’s least-accessible webpage](https://accessibility.blog.gov.uk/2017/02/24/what-we-found-when-we-tested-tools-on-the-worlds-least-accessible-webpage/)

### Articles about ARIA

- [What Is ARIA Even For](https://briefs.video/#pilot)
- [Web Accessibility — ARIA](https://medium.com/hacktive-devs/web-accessibility-aria-437b2c0ba31a)
- [Why, How, and When to Use Semantic HTML and ARIA](https://css-tricks.com/why-how-and-when-to-use-semantic-html-and-aria/)
- [ARIA Landmark Roles](https://medium.com/accessibility-a11y/aria-landmark-roles-459bc1a9e483)
- [Accessible Landmarks](https://www.scottohara.me/blog/2018/03/03/landmarks.html)
- [Using WAI-ARIA Landmarks](https://developer.paciellogroup.com/blog/2013/02/using-wai-aria-landmarks-2013/)

### Courses

- Free PDF book: [Giving a damn about accessibility](https://uxdesign.cc/giving-a-damn-about-accessibility-6caf90be5a40) by Sheri Byrne-Haber
- [Web Accessibility](https://www.udacity.com/course/web-accessibility--ud891) — free course for web developers; it partially overlaps with the videos of [A11ycasts with Rob Dodson](https://www.youtube.com/playlist?list=PLNYkxOF6rcICWx0C9LVWWVqvHlYJyqw7g)

Copy @ [Medium](https://adequatica.medium.com/accessibility-manual-testing-85826e161071)
