---
layout: post
title: "Run Newman (Postman CLI) in TeamCity (CI) with Secrets"
date: 2021-09-25 03:28:26 +0300
tags: newman postman testing
---

Sometimes, as a test engineer, you need to run your tests in CI with secret tokens.

Postman is a perfect tool to start API testing. And [Newman](https://www.npmjs.com/package/newman) is a way to [run your Postman collections on the command line](https://learning.postman.com/docs/running-collections/using-newman-cli/command-line-integration-with-newman/). With Newman, you can easily run tests in your own CI tool: [TeamCity](https://www.jetbrains.com/teamcity/), Jenkins, and so forth.

![Run Newman in TeamCity with Secrets](/assets/2021-09-25/00-cover.png)

Despite the fact that Postman and Newman allow you to keep [variables in different scopes](https://learning.postman.com/docs/sending-requests/variables/), there is a security case, when you are not allowed to store data (passwords, tokens, or any authentication credentials) in plain text.

> If you put a token in Postman’s Globals or Environment variables and export variables into a file (for further use by Newman), your token’s value will be in plain text. Everyone with access to these files could see and exploit it.

To eliminate security-related risks, you can keep secrets inside CI tools. For example, TeamCity allows to [hide the actual value of a variable through Typed Parameters](https://www.jetbrains.com/help/teamcity/typed-parameters.html#Adding+Parameter+Specification).

The idea looks simple:

1. Keep secrets in TeamCity;
2. Run TeamCity build;
3. Get secrets from the environment variable and generate `globals.json` for Newman as a build step;
4. Run Newman as a build step.

![alt_text](/assets/2021-09-25/01-testing-flow.png)

_Fig. 1. Testing flow_

## Create Scripts

1\. Understand the structure of `My_Workspace.postman_globals.json` file:

Postman → Environments → Globals → Export

![postman_globals.json file](/assets/2021-09-25/02-postman-globals-json-file.png)

_Fig. 2. `postman_globals.json` file — as noted above, you can see the token’s value in plain text_

This JSON you need to generate.

2\. Write a script that generates the same JSON structure, [pull the required environment variable](https://medium.com/the-node-js-collection/making-your-node-js-work-everywhere-with-environment-variables-2da8cdf6e786) and, add it to the JSON, create a file:

```JavaScript
const fs = require('fs');

let date = new Date();
let dateIso = date.toISOString();

// The structure of postman_globals.json file
let postmanGlobals = {
  "id": "146e52f0-fd32-4814-8e58-8a3c0f4d5eb7",
  "values": [
	  {
  	  "key": "token",
  	  "enabled": true
	  }
  ],
  "name": "My Workspace Globals",
  "_postman_variable_scope": "globals",
  "_postman_exported_at": dateIso,
  "_postman_exported_using": "Postman/9.0.3"
}

// Access to environment variable and add it to object postmanGlobals
postmanGlobals.values[0].value = process.env.TOKEN;

// Create globals.json file containing object postmanGlobals as a string
fs.writeFile("globals.json", JSON.stringify(postmanGlobals), (err) => {
  if (err) throw err;
});
```

3\. [Export Postman Collection](https://learning.postman.com/docs/running-collections/intro-to-collection-runs/#sharing-collection-runs) (`*.postman_collection.json` file).

My test collection is based on one handler of [OpenWeather API](https://openweathermap.org/api). It requires a token to respond with 200 OK.

![Test collection](/assets/2021-09-25/03-test-collection.png)

_Fig. 3. Test collection_

4\. Write a script which runs [Newman as a library](https://www.npmjs.com/package/newman#using-newman-as-a-library):

```JavaScript
// https://www.npmjs.com/package/newman#using-newman-as-a-library
const newman = require('newman');

newman.run({
  collection: require('./my.postman_collection.json'),
  globals: require('./globals.json'),
  reporters: 'cli'
}, (err) => {
  if (err) throw err;
});
```

5\. Test your scripts locally before running them in CI.

![Newman’s local testing](/assets/2021-09-25/04-newmans-local-testing.png)

_Fig. 4. Newman’s local testing_

To test local access to the environment variable, you need to [add a token to the shell environment](https://askubuntu.com/questions/58814/how-do-i-add-environment-variables):

```
export TOKEN={your_secret_token}
```

## Create Build

[Creating a build configuration in TeamCity](https://www.jetbrains.com/help/teamcity/creating-and-editing-build-configurations.html) is a quite nontrivial process. I will show only the parts related to the Newman run.

### Add Token

In TeamCity build configuration → Parameters → [Add new parameter]

![Add new parameter](/assets/2021-09-25/05-add-new-parameter.png)

_Fig. 5. Add new parameter_

Fill in the fields:

- Name = env.TOKEN
- Kind = Environment variable (env.)
- Value = {your_secret_token}
- Spec → click [Show raw value] = `password display='hidden' readOnly='true'`

After [Save], your variable’s value will be hidden.

![Environment Variable and hidden value](/assets/2021-09-25/06-environment-variable-and-hidden-value.png)

_Fig. 6. Environment Variable (env.) and hidden value_

### Add Build Steps

On each step I run one script file by Node.js.

![Build Step (2 of 3): Generate globals for Newman](/assets/2021-09-25/07-build-step-2-of-3.png)

_Fig. 7. Build Step (2 of 3): Generate globals for Newman_

![Build Step (3 of 3): Run Newman](/assets/2021-09-25/08-build-step-3-of-3.png)

_Fig. 8. Build Step (3 of 3): Run Newman_

![Build Steps](/assets/2021-09-25/09-build-steps.png)

_Fig. 9. Build Steps_

When you [Run] build, everything should work.

![Success](/assets/2021-09-25/10-success.png)

_Fig. 10. Success_

For the reason that all private data is separate from the code, I [post the example on GitHub](https://github.com/adequatica/postman-newman-ci) without fear of token leaks.

Copy @ [Medium](https://adequatica.medium.com/run-newman-postman-cli-in-teamcity-with-secrets-d3f06d7199bf)
