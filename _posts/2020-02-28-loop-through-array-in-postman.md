---
layout: post
title: "Loop through array in Postman"
date: 2020-02-28 08:44:05 +0300
tags: testing postman
---

Sometimes, as a test engineer, you need to iterate over a set of elements and call the same API method with each of them.

If you do not have ready testing infrastructure, you can use [Postman Collection Runner](https://learning.postman.com/docs/collections/running-collections/intro-to-collection-runs/) for a quick solution.

Let’s assume that you have an array (or any collection) of items, each of which you need to pass into the same request.

As an example, I decided to find which of 10 books from [Google Books](https://www.google.com/search?tbm=bks&q=dostoevsky) has a preview (it will be an array of items). As in [my previous article](https://adequatica.medium.com/use-postman-collection-runner-as-vulnerability-scanner-aff7471c94fb), I chose [Google Books API](https://developers.google.com/books/docs/v1/using#RetrievingVolume) for that:

```
{% raw %}https://www.googleapis.com/books/v1/volumes/{{varVolume}}?key={{yourAPIKey}}{% endraw %}
```

In the Postman desktop application, you need to create:

- Request with a variable;
- Variable with initial value = 0 — it will incrementally increase ([does not matter in Globals or Environment scope](https://learning.postman.com/docs/sending-requests/variables/)).

![The initial and current values must be 0](/assets/2020-02-28/01-initial-and-current-values.png)

_Fig. 1. The initial and current values must be 0_

In the Request on the «[Pre-request Script](https://learning.postman.com/docs/writing-scripts/pre-request-scripts/)» tab, you need:

- Specify an array of items;
- Get an index for the current request from the variable;
- Set an item according to the index into another variable for the current request.

```
const dataArray = [
  "TWiYDwAAQBAJ",
  "8ahgioXQ9g8C",
  "K98hhw0IEHgC",
  "mDKphT8_XLsC",
  "XfDOcmJisn0C",
  "vxX2JGsN7PoC",
  "5x4uDwAAQBAJ",
  "mmlgAAAAMAAJ",
  "jetfAAAAMAAJ",
  "38xQHS4h0yEC"
];
let item = pm.globals.get("itemOfArray");
pm.globals.set("varVolume", dataArray[item]);
```

![{varVolume} variable will be created automatically](/assets/2020-02-28/02-varVolume-variable.png)

_Fig. 2. {varVolume} variable will be created automatically_

In the Request on «[Tests](https://learning.postman.com/docs/postman/scripts/test-scripts/)» tab, you need:

- Increment index and save a new value after the current request. It will be used to request a new item on the next iteration;
- Case 1: Output target data from the response [into the console](https://learning.postman.com/docs/sending-requests/troubleshooting-api-requests/);
- Case 2 (advanced): Save target data into a variable.

```JavaScript
let item = pm.globals.get("itemOfArray");
pm.globals.set("itemOfArray", Number(item) + 1);

let jsonData = pm.response.json();
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});

// Output only in case of condition
if (jsonData.accessInfo.viewability !== "NO_PAGES") {
  // Case of output №1 - in Postman console
  console.log(`${jsonData.id} is ${jsonData.accessInfo.viewability}`);

  // Case of output №2 - in Postman variable
  // Additional variable to avoid "undefined" previous data on the first iteration
  let previousResponse = (pm.globals.get("resposeData") === undefined) ? '' : `${pm.globals.get("resposeData")}, `;

  // Represent output as a key:value data
  const keyInQuotes = `"${jsonData.id}"`;
  const valueInQuotes = `"${jsonData.accessInfo.viewability}"`;
  pm.globals.set("resposeData", `${previousResponse}${keyInQuotes}: ${valueInQuotes}`);
}
```

![{responseData} variable will be created automatically](/assets/2020-02-28/03-responseData-variable.png)

_Fig. 3. {responseData} variable will be created automatically_

Run your single request in [Collection Runner](https://learning.postman.com/docs/running-collections/intro-to-collection-runs/). Before starting, you need:

1. Set a number of iterations = quantity of your data items;
2. And open Postman’s console.

![Iterations = array.length](/assets/2020-02-28/04-iterations.png)

_Fig. 4. Iterations = array.length_

![Example of the run](/assets/2020-02-28/05-example-of-the-run.png)

_Fig. 5. Example of the run_

Open Postman’s console and select «Hide network». You’ll see all your target data in a defined format (output case 1);

![Hide the network from the console log](/assets/2020-02-28/06-hide-the-network.png)

_Fig. 6. Hide the network from the console log_

Open Postman’s environments manager and check the value of the variable with response data (output case 2).

![Data stores as the variable value in a copy-pastable format](/assets/2020-02-28/07-copy-pastable-format.png)

_Fig. 7. Data stores as the variable value in a copy-pastable format_

After the current run you should reset the iterable variable to its default value (0) and delete temporary created variables to prevent any failures at the next run.

---

You can also [loop through a data file in the Postman Collection Runner](https://blog.postman.com/looping-through-a-data-file-in-the-postman-collection-runner/).

Copy @ [Medium](https://adequatica.medium.com/loop-through-array-in-postman-f944a5265d62)
