---
layout: post
title: "How I Collected Bumps During the Migration to ESLint 9"
date: 2024-09-27 17:02:57 +0200
tags: linting
---

As a software engineer, you must maintain and keep your infrastructure up-to-date.

![DALL-E 3 prompt](/assets/2024-09-27/00-cover-dall-e-3.jpg)

_DALL-E 3 prompt: draw an engineering computer console with a config file on a terminal, synth-wave 80s-style_

I don’t consider myself a programmer but a QA engineer, so I use some part of the JavaScript tooling as a helpful feature for my primary job: autotests. Tools like [ESLint](https://eslint.org/) and [Prettier](https://prettier.io/) help write clean code — they are good and easy to set up with most default settings. Maximum profit without having to dive deep into the details. However, when the time came to migrate from ESLint v8 to ESLint v9, it forced me to figure out how everything works there.

First of all, [ESLint](https://eslint.org/) is a tool used to analyze and ensure code quality in JavaScript (and TypeScript). The ESLint v9.0.0 [was released](https://eslint.org/blog/2024/04/eslint-v9.0.0-released/) in April 2024, and support for the v8.x version [ends](https://eslint.org/blog/2024/09/eslint-v8-eol-version-support/) on the 5th of October 2024. Half a year for migration sounds like enough time, but the ninth version completely breaks backward compatibility with a new «[flat config](https://eslint.org/blog/2023/10/flat-config-rollout-plans/)» configuration system. The need to completely rewrite the config file became the reason for postponing migration at the very last moment. It is so painful for developers that ESLint even introduced special tools to make the process more accessible: [Configuration Migrator](https://eslint.org/blog/2024/05/eslint-configuration-migrator/) and [Config Inspector](https://eslint.org/blog/2024/04/eslint-config-inspector/).

---

Before even touching configs on any production repository at work, I decided to experiment with my own [boilerplate testing repository](https://github.com/adequatica/ui-testing).

ESLint v8’s `.eslintrc` config file was quite simple:

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "project": ["tsconfig.json"]
  },
  "plugins": ["@typescript-eslint", "simple-import-sort"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-floating-promises": ["error"],
    "comma-dangle": ["error", "always-multiline"],
    "simple-import-sort/imports": "error",
    "simple-import-sort/exports": "error"
  }
}
```

And the whole «ESLint infrastructure» required only four packages:

- `eslint`
- `eslint-plugin-simple-import-sort`
- `@typescript-eslint/eslint-plugin`
- `@typescript-eslint/parser`

My initial idea was just to take the [documentation](https://eslint.org/docs/latest/use/configure/configuration-files) and rewrite a new configuration file from scratch. It looked pretty straightforward: create `eslint.config.mjs` instead of `.eslintrc` and copy-paste the existing rules, but it turned out to be not so simple.

Because for each config’s key, you have to check pages of documentation to match the new format (new way of [extending by «recommended» configs](https://eslint.org/docs/latest/use/configure/combine-configs), new way of [plugins connection](https://eslint.org/docs/latest/use/configure/plugins), and so on), I immediately decided to try **[Configuration Migrator](https://eslint.org/blog/2024/05/eslint-configuration-migrator/)**.

After executing the migration script:

```
npx @eslint/migrate-config .eslintrc
```

I got a new config, twice as long as the previous one:

```javascript
import typescriptEslint from "@typescript-eslint/eslint-plugin";
import simpleImportSort from "eslint-plugin-simple-import-sort";
import tsParser from "@typescript-eslint/parser";
import path from "node:path";
import { fileURLToPath } from "node:url";
import js from "@eslint/js";
import { FlatCompat } from "@eslint/eslintrc";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
const compat = new FlatCompat({
  baseDirectory: __dirname,
  recommendedConfig: js.configs.recommended,
  allConfig: js.configs.all,
});

export default [
  ...compat.extends(
    "eslint:recommended",
    "plugin:@typescript-eslint/eslint-recommended",
    "plugin:@typescript-eslint/recommended"
  ),
  {
    plugins: {
      "@typescript-eslint": typescriptEslint,
      "simple-import-sort": simpleImportSort,
    },

    languageOptions: {
      parser: tsParser,
      ecmaVersion: 5,
      sourceType: "script",

      parserOptions: {
        project: ["tsconfig.json"],
      },
    },

    rules: {
      "@typescript-eslint/no-floating-promises": ["error"],
      "comma-dangle": ["error", "always-multiline"],
      "simple-import-sort/imports": "error",
      "simple-import-sort/exports": "error",
    },
  },
];
```

To make it work, I also had to install two more packages: `@eslint/js` and `@eslint/eslintrc`, and add `eslint.config.mjs` to `tsconfig.json`.

The overhead with `export default [...compat.extends` and a bunch of constants with file paths did not look trustworthy, especially since anything like this is not presented in the documentation.

The first launch attempt immediately returned an error:

```
Parsing error: "parserOptions.project" has been provided for @typescript-eslint/parser.
The file was not found in any of the provided project(s): eslint.config.mjs
```

OK, the migration did not go smoothly. In order to fix it, I decided to stay in the config only lines that look comprehensible and fill [`ignores`](https://eslint.org/docs/latest/use/configure/migration-guide#ignoring-files) and `files` keys to get rid of the `.eslintignore` file.

Then, I decided to manually set up `typescript-eslint` plugin by their [documentation](https://typescript-eslint.io/getting-started/), which also presents a different way of configuration:

For a new ESLint, I also changed [`@typescript-eslint/eslint-plugin`](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin) package to [`typescript-eslint`](https://www.npmjs.com/package/typescript-eslint) as a new proper way to use this tooling.

Somehow, I combined the config, but it did not work until I put the main config’s object above any «recommended» ones:

```javascript
export default [
 {
   ignores: [
     '**/node_modules/*',
     …
   ],
   files: ['**/*.js', '**/*.ts'],
   …
 },
 eslint.configs.recommended,
 ...tseslint.configs.recommended,
];
```

This lore, I dug up the [issues](https://github.com/eslint/eslint/discussions/18304).

Then, with **[Config Inspector](https://eslint.org/blog/2024/04/eslint-config-inspector/)**, I found that my only rule `comma-dangle` is [deprecated](https://eslint.org/docs/latest/rules/comma-dangle), and I need to use a [corresponding rule from the additional package](https://eslint.style/rules/js/comma-dangle): `@stylistic/eslint-plugin` (thank goodness that ESLint Stylistic’s [documentation](https://eslint.style/packages/ts) is excellent).

![ESLint Config Inspector is a fairly useful tool](/assets/2024-09-27/01-eslint-config-inspector.png)

_Fig. 1. ESLint Config Inspector is a fairly useful tool_

The only thing I did not get was why the migrator chose the [language option](https://eslint.org/docs/latest/use/configure/language-options) as `ecmaVersion: 5`, while in` tsconfig.json` I had `es2020`. To make everything smooth, I switched `ecmaVersion` to the default value = `latest` and updated `tsconfig.json` — rechecked that ain’t broken, and pleased that my small repository allowed me to make such a fundamental change.

At least ESLint was working, but when I tried to test how it would catch code errors, I noticed that Prettier started to remove TypeScript’s generic annotations on file save.

Function like this:

```javascript
async getLang(): Promise<Locator> {
  return await this.lang;
}
```

Turned to this:

```javascript
async getLang(): Promise {
  return await this.lang;
}
```

That was a complete disaster, and [StackOverflow did no help](https://stackoverflow.com/questions/78757481/prettier-is-removing-typescript-generic-annotation-from-react-class-component). I assumed that Prettier started to conflict with new ESLint rules and decided to fix it with [`eslint-plugin-prettier`](https://www.npmjs.com/package/eslint-plugin-prettier) package, but it was no use.

**Reboot VS Code editor fixed the last problem.**

---

Finally, I got the working configuration, working linters, and working project.

ESLint v9’s `eslint.config.mjs` config started to look like this:

```javascript
import eslint from "@eslint/js";
import stylisticTs from "@stylistic/eslint-plugin-ts";
import tsParser from "@typescript-eslint/parser";
import eslintPluginPrettierRecommended from "eslint-plugin-prettier/recommended";
import simpleImportSort from "eslint-plugin-simple-import-sort";
import tseslint from "typescript-eslint";

export default [
  {
    ignores: [
      "**/node_modules/*",
      "**/test-results/*",
      "**/playwright-report/*",
    ],
    files: ["**/*.js", "**/*.ts"],
    plugins: {
      "@stylistic/ts": stylisticTs,
      "simple-import-sort": simpleImportSort,
    },
    languageOptions: {
      parser: tsParser,
      ecmaVersion: "latest",
      sourceType: "script",
      parserOptions: {
        project: "./tsconfig.json",
      },
    },
    rules: {
      "@stylistic/ts/comma-dangle": ["error", "always-multiline"],
      "@typescript-eslint/no-floating-promises": ["error"],
      "simple-import-sort/imports": "error",
      "simple-import-sort/exports": "error",
    },
  },
  eslint.configs.recommended,
  ...tseslint.configs.recommended,
  eslintPluginPrettierRecommended,
];
```

Unfortunately, the number of required packages for «ESLint infrastructure» has doubled:

- `@eslint/js`
- `@stylistic/eslint-plugin-ts`
- `@types/eslint__js`
- `@typescript-eslint/parser`
- `eslint`
- `eslint-config-prettier`
- `eslint-plugin-prettier`
- `eslint-plugin-simple-import-sort`
- `typescript-eslint`

But I simplified npm scripts from this:

```
"lint": "eslint '**/*.{js,ts}'",
"lint:fix": "eslint --fix '**/*.{js,ts}'",
```

To this:

```
"lint": "eslint",
"lint:fix": "eslint --fix",
```

---

Despite the small initial config, I ran into too many bumps. I have no idea how people with multiple plugins and dozens of rules will do the migration without problems ([some of them have already given up](https://medium.com/@loic_4084/how-to-migrate-to-eslint-9-x-9d4137b0b939)), but I hope the benefits of [future versions of ESLint](https://eslint.org/blog/2024/07/whats-coming-next-for-eslint/) will exceed the cost of migration.

Related articles:

- [Eslint v9: Migrate from Older Versions](https://javascript.plainenglish.io/eslint-v9-migrate-from-older-versions-da30b74372be);
- [Embrace the Future: Navigating the New Flat Configuration of ESLint](https://www.raulmelo.me/en/blog/migration-eslint-to-flat-config);
- [How does eslint-config-prettier works](https://allalmohamedlamine.medium.com/how-does-eslint-config-prettier-works-b5f926be6418)?

P.S. When trying to update ESLint in production projects, it turned out that some plugins were simply not ready to work with ESLint 9, and it was impossible to switch to a new version without changing the habitual dev environment. That means we will use the eighth version ([v8.57](https://eslint.org/blog/2024/02/eslint-v8.57.0-released/)) for a long time.

Copy @ [Medium](https://adequatica.medium.com/how-i-collected-bumps-during-the-migration-to-eslint-9-973ec54fe254)
