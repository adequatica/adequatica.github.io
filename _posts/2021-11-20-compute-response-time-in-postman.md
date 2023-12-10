---
layout: post
title: "Compute Response Time in Postman"
date: 2021-11-20 15:48:24 +0300
tags: testing api postman
---

Sometimes, as a test engineer, you need to perform a non-function testing.

When developers start to speed up an API, the QA department usually gets two types of tasks:

1. Сheck that handlers work the same way as they worked before (that could be tested by [comparison of responses](https://adequatica.medium.com/brief-comparison-of-responses-in-postman-aea23ee9d342));
2. Check that handlers start work faster.

«Start work faster» means starting to respond in less time. [Response time](<https://en.wikipedia.org/wiki/Response_time_(technology)>) is a common [non-functional requirement](https://en.wikipedia.org/wiki/Non-functional_requirement) for the API or its particular handlers or [methods](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods).

[Postman allows to check the response time](https://learning.postman.com/docs/writing-scripts/script-references/test-examples/#testing-response-times), but one check is often not enough. For complex testing of response time, we need to measure:

- Average response time, based on a large number of requests;
- Maximum (peak) response time to find out the extremes.

![Postman’s run summary](/assets/2021-11-20/01-postmans-run-summary.png)

_Fig. 1. Postman’s run summary_

To perform such testing, we should:

1. Run required request multiple times through [Postman collection runner](https://learning.postman.com/docs/running-collections/intro-to-collection-runs/);
2. Save each response time into a variable;
3. Calculate average and maximum timings.

First of all, we need to create a [variable](https://learning.postman.com/docs/sending-requests/variables/) without value.

![Global variable = timings](/assets/2021-11-20/02-global-variable-timings.png)

_Fig. 2. Global variable = timings_

Then, write a few lines of code in the Tests tab of the request (I took, for example, one handler of [The COVID Tracking Project API](https://covidtracking.com/data/api)). For convenience, I have output the calculated data into the test name.

```JavaScript
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});

const currentTiming = pm.response.responseTime;

let totalTiming = pm.globals.get("timings");

let array;

// If postman variable is empty it is an empty string
if (totalTiming.length === 0) {
  // This is the case for the first or single request
  totalTiming = currentTiming;
  array = [totalTiming];
} else {
  totalTiming = `${totalTiming},${currentTiming}`;
  array = totalTiming.split(',');
}

pm.globals.set("timings", totalTiming);

let sumOfArray = 0;

for (let i = 0; i < array.length; i++) {
  sumOfArray += Number(array[i]);
}

const average = sumOfArray / (array.length);

pm.test(`Response timings: Average = ${average.toFixed(2)}, Max = ${Math.max(...array)}, Min = ${Math.min(...array)}`, function () {
  pm.expect(pm.response.responseTime).to.be.above(0);
});
```

The only edge case is the first or single request, which should be taken into consideration and properly handled.

![Single request](/assets/2021-11-20/03-single-request.png)

_Fig. 3. Single request_

Now, we should clear the variable’s value (otherwise, it will affect our subsequent results) and run the collection with a significant amount of iterations.

![Collection runner](/assets/2021-11-20/04-collection-runner.png)

_Fig. 4. Collection runner_

On the last request we will get average and maximum response timings.

![Response timings](/assets/2021-11-20/05-response-timings.png)

_Fig. 5. Response timings_

When we perform the same test (run this collection with the same number of iterations) for two handlers: the benchmark and the new one, we can compare our metrics.

Of course, such kind of non-functional testing could be made via more professional tools like [LoadRunner](https://www.microfocus.com/en-us/products/loadrunner-professional/overview) or [JMeter](https://jmeter.apache.org).

Copy @ [Medium](https://adequatica.medium.com/compute-response-time-in-postman-89ff3edd093e)
