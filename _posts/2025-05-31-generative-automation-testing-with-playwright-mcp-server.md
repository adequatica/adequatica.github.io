---
layout: post
title: "Generative Automation Testing with Playwright MCP Server"
date: 2025-05-31 05:07:41 +0200
tags: testing
---

As a test engineer, you cannot ignore AI trends and hyped tools. You have to give them a try.

![GitHub Copilot agent creates an autotest through the Playwright MCP server](/assets/2025-05-31/00-cover.jpg)

_GitHub Copilot agent creates an autotest through the Playwright MCP server_

A while ago, Playwright introduced **[Playwright MCP server](https://github.com/microsoft/playwright-mcp)** — a tool for generating end-to-end automation tests through a vibe coding experience.

I decided to give it a try with several intentions:

- Get to know the tool as quickly as possible — go through the path from installation to code as straightforward as possible, without diving into details;
- Check my use case, as if I had used this tool myself for real work, to convert the existing manual test case (user scenario) into an automated test;
- Are generated end-to-end tests worthwhile?

### What is MCP?

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/introduction) is a universal interpreter between LLMs and real-world applications (in this case, it is a Playwright). If you need to solve a clear problem (in this case, testing a web application with Playwright), MCP allows you to give the LLM tasks focused on your goals, without including excessive explanations about your tooling in the prompt, then the LLM may control the application as an AI agent.

So, the Playwright MCP server handles browser integrations, chooses locators, and writes tests, as it gains access to the page’s code and already knows how Playwright works, among other things.

### Installing Playwright MCP

Before you start, you need to prepare the tooling. In my example, I have:

- **VS Code** (​​1.100.2)
- LLM: [GitHub Copilot](https://github.com/features/copilot)
- It is not mandatory, but it will be handy to have an [existing testing project based on Playwright](https://github.com/adequatica/ui-testing) for testing and running generated code (tests).

First, you need to set up Playwright MCP within VS Code. This can be done by adding the MCP server config in `settings.json` (Settings → Open Settings JSON (the icon in the top right corner)):

```json
{
  "chat.mcp.enabled": true,
  "mcp": {
    "servers": {
      "playwright": {
        "command": "npx",
        "args": ["@playwright/mcp@latest"]
      }
    }
  }
}
```

![This config also enables the use of MCP in the chat](/assets/2025-05-31/01-settings-json.png)

_Fig. 1. This config also enables the use of MCP in the chat_

Once you have added an MCP server, you can start it and see the log in the output (CMD+SHIFT+U).

![Running Playwright MCP server in VS Code](/assets/2025-05-31/02-running.png)

Fig. 2. Running Playwright MCP server in VS Code

Then, the Playwright MCP server will be available for use within [GitHub Copilot agent mode](https://code.visualstudio.com/blogs/2025/02/24/introducing-copilot-agent-mode) in VS Code.

### Generating autotests

Before giving tasks to LLM, you still need to have a basic prompt. I got one from [Debbie O’Brien’s example](https://github.com/debs-obrien/generate-test-with-copilot):

```
​- You are a playwright test generator.
- You are given a scenario and you need to generate a playwright test for it.
- DO NOT generate test code based on the scenario alone.
- DO run steps one by one using the tools provided by the Playwright MCP.
- Only after all steps are completed, emit a Playwright TypeScript test that uses @playwright/test based on message history
- Save generated test file in the tests directory
- Execute the test file and iterate until the test passes
```

You need to add this prompt to the chat’s context, which will be reused during the next conversation with LLM.

![Adding prompt to the context](/assets/2025-05-31/03-add-context.png)

Fig. 3. Adding prompt to the context

Now, you can give LLM (in my case, it is Copilot) a task to write a test in natural language.

I took an ordinary website of [CERN’s shop](https://visit.cern/shop), and came up with a pretty common test case:

1. Navigate to [https://visit.cern/shop](https://visit.cern/shop)
2. Filter shop items by category = Postcards
3. Verify that «Atlas postcard» item is filtered
4. Open «Atlas postcard» item
5. Verify that the price is 1 CHF

Giving Copilot a task to write a test based on a presented test case will trigger commands, which will ask for your permission to execute it by the Agent. For simplicity, you can add an auto-approve for this action in the VS Code settings: `"chat.tools.autoApprove": true`

![Ready to send prompt](/assets/2025-05-31/04-prompt.png)

_Fig. 4. Ready to send prompt_

When it runs, the Playwright MCP server runs the browser, examines the page’s code, determines what to do, writes the test accordingly, and provides you with the file with the autotest. It not only creates the test file, but also offers a command to execute it immediately.

![Playwright MCP agent execution](/assets/2025-05-31/05-chat.png)

_Fig. 5. Playwright MCP agent execution. You can notice two unsuccessful commands — LLM tried a few variants and at least found the right way to pass the given scenario_

Execution of the `npx` command also works for me (for some reason, it run all the tests). And at the same time, when I run autotest directly in the terminal, it passes on the first run — I am impressed!

![Generated autotest passed](/assets/2025-05-31/06-passed.png)

_Fig. 6. Generated autotest passed_

```javascript
import { test, expect } from "@playwright/test";

test("Atlas postcard price is 1 CHF after filtering by Postcards", async ({
  page,
}) => {
  // 1. Navigate to https://visit.cern/shop
  await page.goto("https://visit.cern/shop");

  // 2. Filter shop items by category = Postcards
  await page.getByLabel("Category").selectOption("Postcards");
  await page.getByRole("button", { name: "Filter" }).click();

  // 3. Verify that «Atlas postcard» item is filtered
  await expect(
    page.getByRole("heading", { name: "Atlas postcard" }),
  ).toBeVisible();

  // 4. Open «Atlas postcard» item
  // The click is intercepted, so navigate directly
  await page.goto("https://visit.cern/node/609");

  // 5. Verify that the price is 1 CHF
  await expect(page.getByText("Price")).toBeVisible();
  await expect(page.getByText("1 CHF")).toBeVisible();
});
```

Of course, there is plenty of room for improvement in the generated test:

- Split the whole test into [steps](https://playwright.dev/docs/api/class-test#test-step);
- Add [custom expect messages](https://playwright.dev/docs/test-assertions#custom-expect-message);
- Add more intermediate expectations during the test;
- A human test engineer can definitely find a way to click on the item, instead of navigating to it directly (it is the main AI’s mistake);
- Rewrite the verification of the price.

Anyway, the test works; most elements [get by roles](https://playwright.dev/docs/api/class-framelocator#frame-locator-get-by-role) and labels, and verification steps have expectations. For a person who is not a professional test automation engineer, this test may appear fine because it solves the problem of increasing test coverage for regression testing.

If I test this «server» for a longer time, on a large number of different and complicated scenarios, I would face issues. However, I do not see any obstacles to using this AI tool if the test cases are simple enough, the project is not too serious, and the team does not have dedicated automation test engineers.

---

In conclusion, I answered all my questions:

- This tool works almost out of the box and requires minimal settings at the start;
- It works for my use case — it can convert user scenarios into autotests;
- These autotests are mediocre, but surprisingly, they are workable.

Just half a year ago, LLMs were generating awful tests. They looked fine, but in practice, they would not run and required a lot of manual refactoring. However, it is a whole different story now.

- AI still would not replace the test automation engineer, but now the same engineer may perform twice as well, because editing tests is easier than writing them from scratch.
- Manual testers, who never write code but write test cases, can now process their cases into somehow working autotests. This is a good starting point for anyone who wants to develop their skills in automation testing but has hesitated or does not know how to begin.
- Frontend developers who do not have enough time and resources to deal with UI testing can now quickly increase the test coverage of their interfaces. Some tests are always better than nothing.

Let’s see where this promising technology will lead us in the near future.

Read more:

- [How to Generate Playwright Tests using MCP + Copilot](https://www.youtube.com/watch?v=AaCj939XIQ4);
- **[Vibe testing with Playwright](https://timdeschryver.dev/blog/vibe-testing-with-playwright)** (the article provides an extended description of the Playwright MCP server experience);
- [Modern Test Automation with AI and Playwright MCP](https://kailash-pathak.medium.com/modern-test-automation-with-ai-llm-and-playwright-mcp-model-context-protocol-0c311292c7fb) (the article shows how to set up Playwright MCP for different IDEs);
- [A Visual Guide To MCP Ecosystem](https://block.github.io/goose/blog/2025/04/10/visual-guide-mcp/).

Copy @ [Medium](https://adequatica.medium.com/generative-automation-testing-with-playwright-mcp-server-45e9b8f6f92a)
