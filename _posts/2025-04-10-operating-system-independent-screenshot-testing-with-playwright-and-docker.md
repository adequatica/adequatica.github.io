---
layout: post
title: "Operating System Independent Screenshot Testing with Playwright and Docker"
date: 2025-04-10 04:58:53 +0200
tags: testing
---

Sometimes, as a test engineer, you might be dazed when all your dev team needs to perform visual regression testing.

![Operating System Independent Screenshot Testing with Playwright and Docker](/assets/2025-04-10/00-cover.jpg)

One of the causes of flaky tests are Operating Systems (OS) and platforms (e.g., browsers) [1]. This is especially true for visual regression testing, which is why screenshot tests may be very fragile.

> Terminology clarification: visual regression testing and screenshot testing are the same type of testing here.

## Challenge

The core problem is dug into different font rendering methods used by various operating systems and browsers [[2](https://www.inkwell.ie/typography/os-rendering.html)]. The same text with the same font looks slightly different on Windows, macOS [[3](https://damieng.com/blog/2007/06/13/font-rendering-philosophies-of-windows-and-mac-os-x/)], and Linux. Thus, on closer inspection, it appears that the text pixels are very different, and for visual comparison this «same» text will be totally opposite.

![Different rendering in Chromium on Ubuntu and Safari on macOS](/assets/2025-04-10/01-rendering-cromium-ubuntu-safari-macos.png)

_Fig. 1. Different rendering in Chromium on Ubuntu and Safari on macOS_

This problem multiplies if your dev team members run visual tests on local machines with different configurations and in a cloud with its own setting. For example, when one developer turned on the [anti-aliasing](https://www.oreilly.com/library/view/web-design-in/0596009879/ch28s04s05.html) setting, it would make reference screenshots, and then visual tests would fail for another developer who turned off this setting on his machine. More simply, developers on Windows, macOS, and Linux cannot just run the same visual regression test suite locally.

Read more about font rendering:

- [Type rendering on the web](https://blog.typekit.com/2010/10/05/type-rendering-on-the-web/) by Tim Brown;
- [A Closer Look At Font Rendering](https://www.smashingmagazine.com/2012/04/a-closer-look-at-font-rendering/) by Tim Ahrens.

## Solution

**The solution to the challenges is running tests inside Docker.** All tests and screenshots inside Docker will always be executed on the same OS and the same browser.

Additional requirements include seamless running with a single command that is unified for all developers. Reference screenshots and test reports should be available locally, and Docker should only be needed for running Playwright tests on demand.

**Let’s start by assuming that you already have a Playwright’s test infrastructure** for running visual regression tests (if not, take a look at [this article](https://www.browsercat.com/post/ultimate-guide-visual-testing-playwright) or [the other one](https://css-tricks.com/automated-visual-regression-testing-with-playwright/)). In addition, you will need a few things:

1. `Dockerfile` for building a Docker build;
2. `Makefile` script for running dockerfile;
3. `Bash` script for running Docker image — running this script will run tests too.

> I suppose it can be done simply, but the proposed approach has proven itself as a production solution in several projects I’ve worked on.

### 1. Dockerfile for building a Docker build

[Microsoft already provides Docker images for Playwright](https://playwright.dev/docs/docker), and it remains to supplement it with the necessary settings for accepting Playwright commands:

```bash
FROM mcr.microsoft.com/playwright:v1.51.1

# Use a safe working directory to avoid conflicts in the root
WORKDIR /app

RUN npm install @playwright/test@1.51.1

CMD ["npx", "playwright"]
```

In the presented case, `:v1.51.1` container is based on Ubuntu 24.04 LTS (Noble Numbat) and contains the Playwright v1.51.1 release with Chromium 134.0.6998.35 browser. All test runs, regardless of who and where they run it (locally by any dev team member or CI/CD), will be executed on this «stack».

### 2. Make script for running dockerfile

A script for building Docker image through universal [Make](<https://en.wikipedia.org/wiki/Make_(software)>) will allow any dev team members to use it without bothering about any other settings:

```bash
.PHONY: init-playwright-screenshot-tests
init-playwright-screenshot-tests:
   @docker buildx build -t playwright-screenshot-tests ./docker/screenshot-tests
```

### 3. Bash script for running Docker image

Another script, this time on Вash, will run Docker image to execute Playwright tests inside the container:

```bash
set -ex

if [ -t 0 ] ; then
   ARGS="-i"
else
   ARGS=""
fi

docker run ${ARGS} --rm \
 -p 9323:9323 \
 -v "$PWD/tests/:/app/tests" \
 -v "$PWD/playwright.screenshot.config.ts:/app/playwright.screenshot.config.ts" \
 -v "$PWD/playwright-report/:/app/playwright-report" \
 -e "CI=${CI}" \
 --add-host=host.docker.internal:host-gateway \
 -t playwright-screenshot-tests \
 npx -y playwright "$@"
```

Where:

- `-p 9323:9323` option maps port 9323 on the local machine to port 9323 inside the container. This allows to open Docker’s resources on a local machine, for example, for examining test reports [http://0.0.0.0:9323/](http://0.0.0.0:9323/) (the same port should also be set for HTML [reporter in Playwright’s config](https://playwright.dev/docs/api/class-testconfig#test-config-reporter)).
- `-v` options mount directories from the current working directory (`$PWD`) into `/app` directory inside the container. This allows the container to access local files.
- `-t playwright-screenshot-tests` specifies the name of the Docker image to use (it was a given name in step №2).
- After the container starts, the script will run `npx -y playwright "$@"` command inside the container; where `-y` flag automatically confirms any prompts that `npx` might display, and `"$@"` option allows to execute Playwright with any additional arguments passed to the script.

This `playwright-screenshot.sh` script should run through the npm script ⇒ `package.json` should contain this command:

```json
 "scripts": {
   "test:screenshots": "./playwright-screenshot.sh test --config playwright.screenshot.config.ts"
 }
```

![Result of running npm script command](/assets/2025-04-10/02-npm-run-test-screenshots.png)

_Fig. 2. Result of running npm script command_

Now, when all dev team members run tests (or update screenshots) through Docker, you will be working with guaranteed identical images.

**For a better understanding, [check out the repository](https://github.com/adequatica/visual-regression-testing).**

---

### More tips to follow:

- **Do not capture and compare whole pages.** Capture solely elements on the page and make them as logically small as possible.
- **Set some [`maxDiffPixels`](https://playwright.dev/docs/test-snapshots#maxdiffpixels)** (an acceptable amount of pixels that could be different); a few pixels will not change the game but bring robustness against flakiness.
- Update screenshots with each browser update because [browsers may change text rendering processes](https://developer.chrome.com/blog/better-text-rendering-in-chromium-based-browsers-on-windows).
- **Do not mix end-to-end tests and screenshot tests.** Store and run these tests separately. Think of them as two absolutely different kinds of tests despite using the same test framework.
- Go through the [Playwright’s documentation on visual comparisons](https://playwright.dev/docs/test-snapshots).

References:

1. Negar Hashemi, Amjed Tahir, and Shawn Rasheed, “An Empirical Study of Flaky Tests in JavaScript,” in 38th IEEE International Conference on Software Maintenance and Evolution (ICSME), 2022. [https://doi.org/10.48550/arXiv.2207.01047](https://doi.org/10.48550/arXiv.2207.01047)
2. [OS & Browser text rendering](https://www.inkwell.ie/typography/os-rendering.html) by Matt Mc Donagh;
3. [Font rendering philosophies of Windows & Mac OS X](https://damieng.com/blog/2007/06/13/font-rendering-philosophies-of-windows-and-mac-os-x/) by Damien Guard.

Copy @ [Medium](https://adequatica.medium.com/operating-system-independent-screenshot-testing-with-playwright-and-docker-6e2251a9eb32)
