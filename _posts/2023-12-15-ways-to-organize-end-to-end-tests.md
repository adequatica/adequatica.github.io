---
layout: post
title: "Ways to Organize End-to-End Tests"
date: 2023-12-15 10:00:25 +0100
tags: automation testing
---

Sometimes, test engineers and developers have opposite views on the way to organize a file structure of end-to-end tests. However, there are ways that can be considered «best practices».

![Midjourney prompt](/assets/2023-12-15/00-cover.jpg)

_Midjourney prompt: programming files structure, 2d scheme, engineering drawing, white background_

Some web developers usually see end-to-end tests as overcomplicated utility scripts, which can be managed as unit tests. However, test engineers strive to organize them in an image and likeness of the structure of their Test Management System (TMS).

I was lucky enough to test and organize autotests on different projects with various file structures and found that there is no ideal way to do it. It is just some ways are much better than others, depending on the application’s architecture and team’s workflows.

- End-to-end tests are spread across the project (BAD);
- End-to-end tests’ specs are located in a separate directory (GOOD);
- End-to-end tests are located in a separate directory (GOOD);
- End-to-end tests are located in a separate repository (SO-SO).

Disclaimer:

- Examples of a file structure will be based on a «common» frontend application and the [Playwright](https://playwright.dev/) testing framework.
- End-to-end tests in this article = autotests that implement user stories or replicate «manual» test cases by QA team. It doesn’t matter how fast or slow they are; the main thing is that they have to open a tested website and do something to check something.

---

## End-to-end tests are spread across the project

First of all, **this is definitely a bad way to handle end-to-end tests.**

```
.
├── public/
├── src/
│   ├── apps/
│   │   ├── pages/
│   │   │   ├── about/
│   │   │   │   ├── page-object-models/
│   │   │   │   │   └── about.ts
│   │   │   │   └── about.tsx
│   │   │   └── home/
│   │   │       ├── page-object-models/
│   │   │       │   └── home.ts
│   │   │       └── home.tsx
│   │   └── stores/
│   ├── components/
│   │   ├── component-one/
│   │   │   ├── one.css
│   │   │   ├── one.tsx
│   │   │   └── one.spec.ts
│   │   └── component-two/
│   │       ├── two.css
│   │       ├── two.tsx
│   │       └── two.spec.ts
│   └── features/
├── .eslintrc
├── .gitignore
├── package.json
├── playwright.config.ts
├── README.md
└── tsconfig.json
```

Characteristics:

- The [test configuration](https://playwright.dev/docs/test-configuration) file (`playwright.config.ts`) is located on the root level, along with all the application settings.
- Based on the previous point, all dependencies of the project will be installed to run tests.
- Tests follow code style and linters from the parent project.
- End-to-end tests are located near tested components.

Developers who do this argue that tests check corresponding components, but what if there is more than one test per component?, if a functional test affects a few components?, if a test doesn’t affect any particular component (some general check for performance of a11y)?, then where should you put such a file?

Page object models’ files are also become spread across different directories. Someone puts them inside components’ directories near tests; someone puts them inside pages’ directories; someone puts models in a separate directory. Anyway, it turns into a mess.

**It is inconvenient for QAs** to find and navigate through autotests. It is also difficult to combine test cases from TMS with autotests if they are spread across multiple directories of a project.

- A good point is that features and tests can be done/fixed simultaneously in one Pull Request.

**The described way is suitable for unit tests, not end-to-ends.** Similar opinions:

- _[There is a fundamentally different strategy for how these two types of tests are written](https://www.jimlynchcodes.com/blog/why-i-recommend-unit-tests-in-the-src-folder)._
- _[I don’t think the approach with unit test files works for these tests. Instead, I like having a top-level directory named “integration-tests” that contains these tests](https://www.coreycleary.me/where-to-put-your-tests-in-a-node-project-structure)._

It is also okay to put screenshot tests next to its components (of course, if the screenshot is only taken of a single component and doesn’t include complicated logic).

## End-to-end tests’ specs are located in a separate directory

This one is a good approach to organizing autotests.

```

.
├── public/
├── src/
│   ├── apps/
│   ├── components/
│   └── features/
├── tests/
│   ├── page-object-models/
│   │   ├── about.ts
│   │   └── home.ts
│   ├── test-one.spec.ts
│   └── test-two.spec.ts
├── .eslintrc
├── .gitignore
├── package.json
├── playwright.config.ts
├── README.md
└── tsconfig.json
```

Characteristics:

- The test configuration file is located on the root level, along with all the application settings.
- Based on the previous point, all dependencies of the project will be installed to run tests.
- Tests follow code style and linters from the parent project.
- End-to-end tests are located in a separate directory (`/tests` or `/e2e`).

Here, all the tests are put together in one directory, and page object models (and possible utils) are located nearby. QAs can easily find and navigate through autotests because the file structure is explicit.

- Features and tests can be done/fixed simultaneously in one Pull Request.

## End-to-end tests are located in a separate directory

This way is not much different from the previous one, except test infrastructure becomes independent.

```
.
├── public/
├── src/
│   ├── apps/
│   ├── components/
│   └── features/
├── tests/
│   ├── page-object-models/
│   │   ├── about.ts
│   │   └── home.ts
│   ├── tests/
│   │   ├── test-one.spec.ts
│   │   └── test-one.spec.ts
│   ├── .eslintrc
│   ├── .gitignore
│   ├── package.json
│   ├── playwright.config.ts
│   ├── README.md
│   └── tsconfig.json
├── .eslintrc
├── .gitignore
├── package.json
├── README.md
└── tsconfig.json
```

Characteristics:

- The test configuration file and and-to-end tests are located in a separate directory. Autotests become an «application», and its directory should contain additional files and dependencies.
- Based on the previous point, only autotests’ dependencies will be installed to run tests. **It will speed up the preparation of the build in CI.**
- Tests _may_ follow code style and linters from the parent project.

Here, all the tests and page object models are put together in one directory. QAs can easily find and navigate through autotests because the file structure is explicit.

Because test infrastructure becomes more flexible and «independent», QAs have more opportunities to manage the tests’ structure within the scope of the parent project.

- Features and tests can still be done/fixed simultaneously in one Pull Request.

A similar case of implementation of this organization is taking out `/tests` directory on the root level of the app if the app consists not just of one client service.

```
.
├── .infrastructure/
├── client/
│   ├── public/
│   └── src/
│       ├── apps/
│       ├── components/
│       └── features/
├── server/
└── tests/
   ├── client/
   │   ├── page-object-models/
   │   └── tests/
   └── server/
```

Here, `/tests` directory can contain separate/independent directories for each test project for any subprojects in the root level directory (like `/client` and `/server` in the example above).

## End-to-end tests are located in a separate repository

This approach is quite controversial, but it is suitable for large or independent QA teams.

```
.
├── public/
├── src/
│   ├── apps/
│   ├── components/
│   └── features/
├── .eslintrc
├── .gitignore
├── package.json
├── README.md
└── tsconfig.json

.
├── page-object-models/
│   ├── about.ts
│   └── home.ts
├── tests/
│   ├── test-one.spec.ts
│   └── test-one.spec.ts
├── .eslintrc
├── .gitignore
├── package.json
├── playwright.config.ts
├── README.md
└── tsconfig.json
```

Characteristics:

- Full independence from a tested project. QA team can implement and organize autotests in whatever way they want (maybe repeat the structure of test cases from TMS).
- Only autotests’ dependencies will be installed to run tests. This is an excellent option.
- The test project has to copy/paste code style and linters from the parent project if the QA team wants to follow these rules.
- Features and tests still can’t be done/fixed simultaneously in one Pull Request. Tests for new features or fixes should be done in a separate Pull Request in a separate repository  — **it is not handy for developers. Tests in a separate repository are convenient only for QA teams.** This can’t be considered a bad thing, because with a large number of tests (hundreds or thousands), it is faster to develop and fix them in a separate repository by QAs.

---

Thanks to [tree.nathanfriend.io](https://tree.nathanfriend.io/) for generating ASCII folder structure diagrams.
