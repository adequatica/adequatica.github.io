---
layout: post
title: "Developing UI with Testing in Mind"
date: 2024-05-13 10:38:08 +0100
tags: testing
---

In web development, testing is not just about ensuring functionality; it is also about guaranteeing a seamless user experience and accessibility. This article outlines essential practices for upholding these standards at the foundation stage.

![DALL-E 3 prompt](/assets/2024-05-13/00-cover-dall-e-3.jpg)

_DALL-E 3 prompt: make the interface for a complex modern website with accessibility features, buttons, alerts, notifications, backdrops, hints, loaders, progress bars, and skeletons. Use material UI and use bright web-safe color palette. 2d view from the top and ready to code_

On my current project, I work not only as a test engineer but also as a web developer, creating some UI components that I had to test by myself simultaneously. Suddenly, taking into account the tester’s background, you have to consider many things that are missing from the design mockups but which will be revealed at the testing stage.

The following tips will help deliver an intuitive and seamless user experience when developing UI components:

1. Providing a meaningful visual clue;
2. Do not hide the outline;
3. Enhancing interactive elements;
4. Feedback on user actions;
5. Implementing error handling from the backend;
6. Closeable pop-ups and notifications;
7. Ensuring forms usability;
8. Maintaining visual stability;
9. Maintaining overall consistency;
10. Testability of elements.

## 1. Providing a meaningful visual clue

Color-coded pictograms and icons should not rely solely on color to convey information. They must have a title or an alt text description with its value because:

- Users may not distinguish color (accessibility issue);
- Users may not trust a color code (some technical users want to see real data instead of a «friendly interpretation» of the data).

![On the left: something’s value is only color-coded; on the right: the icon has a title with the value and the value is specified in the description](/assets/2024-05-13/01-smth.png)

_Fig. 1. On the left: something’s value is only color-coded; on the right: the icon has a title with the value and the value is specified in the description_

Keep in mind, that all users, including those with color vision deficiencies, should effectively «read» the interface.

## 2. Do not hide the outline

Unfortunately, mostly due to ignorance, some developers and designers argue about hiding outline styles by saying that this is «ugly». However, [Retaining an outline on the focused element](https://adequatica.medium.com/automated-accessibility-testing-of-keyboard-navigation-on-tab-89d30087c111#f3a3) aids users in understanding their current place on the page, offering valuable context and improving overall usability.

![The default outline](/assets/2024-05-13/02-outline.png)

_Fig 2. The default outline_

Moreover, the **outline style can be changed, **and its behavior can be configured separately for mouse and keyboard navigation by [`focus` and `focus-visible`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible#focus_vs_focus-visible) CSS classes.

```
/* It draws the focus ring around the button when the user navigates to it with the keyboard */
button:focus-visible {
 outline: 2px solid #0069c2;
}
```

Keep in mind, that outlines serve an essential function in user navigation, particularly in multi-button interfaces.

## 3. Enhancing interactive elements

Interactive elements should be intuitively highlighted in some way when in focus:

- Buttons should change color or shape;
- Links should change color and/or underlining;
- The mouse [cursor](https://developer.mozilla.org/en-US/docs/Web/CSS/cursor) should change the shape on clickable elements to a pointer. This point is contradictory for buttons but unambiguous for URLs. According to the [definition of the pointer](https://www.w3.org/TR/CSS22/ui.html#cursor-props): «the cursor is a pointer that indicates a link».

![Yes button changed color after hover and the mouse shape has the pointer hand](/assets/2024-05-13/03-pointer.png)

_Fig. 3. [Yes] button changed color after hover and the mouse shape has the pointer hand_

Keep in mind, that interactivity provides visual feedback to users navigating the interface via keyboard or mouse.

Read more:

- [Into to UI Cursors](https://app.uxcel.com/courses/ui-components-n-patterns/basics-834);
- Two opinions: «[Buttons shouldn’t have a hand cursor](https://medium.com/simple-human/buttons-shouldnt-have-a-hand-cursor-b11e99ca374b)» and «[On the web, maybe buttons should have a hand cursor](https://thepascalheynol.medium.com/on-the-web-maybe-buttons-should-have-a-hand-cursor-1e4498c42b3e)».

## 4. Feedback on user actions

Every user action should evoke a corresponding feedback response, **indicating that the action has been initiated or is in progress.** This ensures users are not left in the dark, providing reassurance and clarity throughout their interaction with the interface.

It can be a special state of the button or the whole intractable component, or some kind of alert or notification after click. There are a lot of feedback components that can be used: alerts, notifications, backdrops, hints, loaders, progress bars, and skeletons.

![Example of feedback Notification in Mantine components library](/assets/2024-05-13/04-notification.png)

_Fig. 4. Example of feedback Notification in Mantine components library_

Apple Human Interface Guidelines has [Feedback’s best practices](https://developer.apple.com/design/human-interface-guidelines/feedback#Best-practices) which apply to the web.

## 5. Implementing error handling from the backend

This issue is similar to the previous one but here it is expanded beyond the client side.

It is a good practice to alert users of any problems that may arise by backend responses. Valuable bad requests ([4xx](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) and [5xx](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses) status codes) from the backend should be processed and shown in some way (as hints or notifications) on the frontend.

![Example of React Alert in Material UI components library](/assets/2024-05-13/05-refresh.png)

_Fig. 5. Example of React Alert in Material UI components library_

Keep in mind, that consistent communication between the backend and frontend ensures transparency and empowers users to navigate potential disruptions effectively.

## 6. Closeable pop-ups and notifications

All pop-ups, overlays, and notifications, even if they are auto-closable, should have a closure cross or [Close] button and be in an accustomed place. [Close] button and [X] in one element are not redundant because different users have different habits and experiences.

![Example of Modal window in Material UI components library](/assets/2024-05-13/06-close.png)

_Fig. 6. Example of Modal window in Material UI components library_

In addition, closing the pop-up by [ESC] key will not be superfluous 😉

Read further:

- [Vectors of Testing Pop-up Overlays](https://adequatica.medium.com/vectors-of-testing-pop-up-overlays-modal-windows-65c2ede32344).

## 7. Ensuring forms usability

There are a lot of articles about forms’ usability that my recommendations will be a drop in the bucket. During developing and testing forms, I often focus on the required fields. There can be two main cases with them:

1. Disable [Save]/[Send] button until all the required fields are filled in. In this case, it should be an indication of why the submit button is disabled and all required fields should be marked.
2. Upon attempted submission, highlight the blank or required fields. In this case, all previously filled fields should not be erased. And of course, all required fields should be marked.

![Example of a form in Mantine components library. Required fields are marked and the [Register] button is disabled while required fields are empty](/assets/2024-05-13/07-register.png)

_Fig. 7. Example of a form in Mantine components library. Required fields are marked and the [Register] button is disabled while required fields are empty_

Read further:

- [An Extensive Guide To Web Form Usability](https://www.smashingmagazine.com/2011/11/extensive-guide-web-form-usability/);
- [Designing Efficient Web Forms: On Structure, Inputs, Labels And Actions](https://www.smashingmagazine.com/2023/02/guide-accessible-form-validation/);
- [How to Design a User-Friendly Form](https://blog.hubspot.com/marketing/form-ux);
- [How to test forms for usability](https://web.dev/learn/forms/usability-testing).

## 8. Maintaining visual stability

To ensure a smooth user experience, elements, particularly buttons and blocks of text, should not jump or shift unexpectedly when their state changes.

![Avoid unintended jumps and shifts](/assets/2024-05-13/08-open.gif)

_Fig. 8. Avoid unintended jumps and shifts_

Keep in mind, that divert shifts of UI elements distract the user’s attention.

## 9. Maintaining overall consistency

Almost last but not least, maintain the consistency of the components and their logic. If you have native checkboxes and selects on one page, do not invent a custom one for another page. If you use buttons to perform some actions, do not use buttons as links. All interactive elements must be combined with each other and perform the same function throughout the whole app.

![On the left: the «Read more» looks like a button, but behaves as a link; on the right: «Read more» looks and behaves as a link](/assets/2024-05-13/09-read-more.png)

_Fig. 9. On the left: the [Read more] looks like a button, but behaves as a link; on the right: «Read more» looks and behaves as a link_

Keep in mind, that your interface should provide clear feedback when users misuse tools or features, guiding them toward correct usage and preventing frustration. A tool that users use incorrectly should tell them that they are doing it wrong.

## 10. Testability of elements

This one is a controversial item and highly depends on the processes within the development team: who writes autotests? which framework is used? and so on.

Based on my experience, it is much easier (and cost-efficient) to commit to UI testing of each component at the development stage. Adding «anchors» of test IDs for locators for interactable elements (buttons, inputs, selects, etc.) is better in advance than later — your test automation engineers will say «Thank you» for that later.

```
<Button
 onClick={onOpenSmth}
 variant="light"
 title="Open something"
 data-testid="button-open-smth"
>
 Open
</Button>
```

Read further:

- [Locate by test id](https://playwright.dev/docs/locators#locate-by-test-id);
- [Testing Recipes](https://legacy.reactjs.org/docs/testing-recipes.html) (take a look at data-testid attributes at examples in code snippets);
- [Why Your Development Team Should Use data-testid Attributes](https://medium.com/@automationTest/why-your-development-team-should-use-data-testid-attributes-a83f1ca27ebb).

---

If none of the listed items were considered during the web application’s development, then QA engineers will report dozens of [bugs related to A11Y](https://adequatica.medium.com/accessibility-manual-testing-85826e161071), UI, UX, and the app’s logic. Meanwhile, most of them can be resolved in advance by adhering to the practices discussed above.

Copy @ [Medium](https://adequatica.medium.com/developing-ui-with-testing-in-mind-741d9fe8e3b3)
