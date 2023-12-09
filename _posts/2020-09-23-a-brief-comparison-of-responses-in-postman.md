---
layout: post
title: "A Brief Comparison of Responses in Postman"
date: 2020-09-23 08:26:55 +0300
tags: api testing postman
---

Sometimes, as a test engineer, you need to quickly compare two API responses.

Moreover, if you don’t need to look for differences and just want to get YES (they are the same) or NO (they are not) — [Postman](https://www.postman.com/) is a way to do that.

Let’s assume that you have a REST API with success code = 200 OK and the response comes in JSON format. I took, for example, one handler of the [SpaceX REST API](https://github.com/r-spacex/SpaceX-API/blob/master/docs/v4/README.md):

```
https://api.spacexdata.com/v4/capsules/{capsuleId}
```

For the first request in the [Tests tab](https://learning.postman.com/docs/writing-scripts/script-references/test-examples/), you need:

- Save response data into a variable.

![Tests tab](/assets/2020-09-23/01-tests-tab.png)

```JavaScript
pm.test("Status is OK", function () {
pm.response.to.have.status(200);
    });
let jsonData = pm.response.json();
pm.globals.set("response", jsonData);
```

For the second request in the Tests tab, you need:

- Get previous response data from a variable;
- Compare current response data with the previous one. I use [deep-eql Chai assert](https://www.chaijs.com/api/bdd/#method_equal) for comparing objects.

![Tests tab](/assets/2020-09-23/02-tests-tab.png)

```JavaScript
pm.test("Status is OK", function () {
    pm.response.to.have.status(200);
});
let jsonData = pm.response.json();
let firstResponse = pm.globals.get("response");
pm.test("Responses are equal", function () {
    pm.expect(jsonData).to.deep.equal(firstResponse);
});
```

In the case of a FAIL test, you have two different data.

For more detailed research of the cause of the failure or the place of data discrepancy, you will need a more advanced test script or a different tool.

Copy @ [Medium](https://adequatica.medium.com/brief-comparison-of-responses-in-postman-aea23ee9d342)
