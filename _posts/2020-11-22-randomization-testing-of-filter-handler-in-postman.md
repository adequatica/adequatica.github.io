---
layout: post
title: "Randomization Testing of Filter Handler in Postman"
date: 2020-11-22 13:07:20 +0300
tags: api testing postman
---

Sometimes, as a test engineer, you need to make multiple requests of a single API method with different parameters.

A special case of such an API method could be a filter or a search handler, which makes requests with pre-selected parameters. In the interface, it might look like this:

![Tests tab](/assets/2020-11-22/01-show.png)

_Fig. 1. You select parameters and press [Show]_

Each parameter has predefined values. They could be hardcoded on the frontend or got from other handlers.

Thus, to make a desired request, we first need somehow to collect the parameters.

After that, we can proceed to test and apply different techniques of test design:

- [Pairwise testing](http://pairwise.org/);
- [Combinatorial methods](https://en.wikipedia.org/wiki/Combinatorics);
- [Equivalence partitioning](https://en.wikipedia.org/wiki/Equivalence_partitioning);
- [Random testing](https://en.wikipedia.org/wiki/Random_testing).

Let’s focus on the last one because the aim of the article is to show how to operate with handlers, variables and collections in [Postman](https://learning.postman.com/docs/introduction/overview/) rather than finding defects. [COVID-19 Rich Data Services](https://documenter.getpostman.com/view/2220438/SzYevv9u) free API was taken as an example.

Our step-by-step algorithm is as follows:

1. Decide which parameters we need to collect for filtering requests;
2. Understand how to collect those parameters;
3. Create precondition requests with [scripts](https://learning.postman.com/docs/writing-scripts/script-references/test-examples/);
4. Run preconditions;
5. Run filter handler with randomized parameters, received from previous steps.

![Testing flow](/assets/2020-11-22/02-testing-flow.png)

_Fig. 2. Testing flow_

### 1. Decide which parameters we need to collect for filtering request

[According to the documentation](https://documenter.getpostman.com/view/2220438/SzYevv9u#ca3edafd-5683-4cc1-aab1-0c497d1186dc), the filter handler can accept a lot of parameters, but we will limit ourselves to a few:

```
http://{host}/rds/api/query/{catalogId}/{productId}/select?collimit={number}&count={boolean}&format={id}&inject={boolean}&limit={number}&metadata={boolean}&where={query}
```

### 2. Understand how to collect those parameters

- `catalogId` — comes from [/rds/api/catalog](https://documenter.getpostman.com/view/2220438/SzYevv9u#245ce204-95a4-41cc-ac53-afd10c5a87b2), it can be collected by parsing the response body;
- `productId` — comes from [/rds/api/catalog](https://documenter.getpostman.com/view/2220438/SzYevv9u#245ce204-95a4-41cc-ac53-afd10c5a87b2) and depends on `catalogId`, it can be collected by parsing the response body;
- `collimin` — number, set arbitrarily;
- `count` — boolean, set arbitrarily;
- `format` — a limited set of values based on documentation, we will predefine all available formats into a variable;
- `inject` — boolean, set arbitrarily;
- `limit` — number, set arbitrarily;
- `metadata` — boolean, set arbitrarily;
- `where` — SQL-like string, for simplicity, we will use only one pair of parameters — classification and classification code — they are connected and can be collected by parsing the response body from [/rds/api/query/{catalogId}/{productId}/classifications](https://documenter.getpostman.com/view/2220438/SzYevv9u#b763ba93-d3dd-419e-83a7-85b1ea5e49ed) and [/rds/api/query/{catalogId}/{productId}/classification/{classificationId}/codes](https://documenter.getpostman.com/view/2220438/SzYevv9u#d9b1a38d-e08c-4e91-b04f-e0aef91d7b22).

### 3. Create precondition requests with scripts

For making a real filter request `catalogId` and `productId` must match, but since our goal is to make a synthetic request with any combinations, it is OK to collect these parameters without matching. To overcome inconsistency, we will run a collection of preconditions a few times.

After parsing the requested data from the response, we need to set them to environment variables:

```JavaScript
let jsonData = pm.response.json();
// Collect and store data only if it exists
if (jsonData.catalogs.length > 0) {
  let allCatalogs = [];
  let allDataProducts = [];

  jsonData.catalogs.forEach(
    element => allCatalogs.push(element.id)
  );
  // Method of parsing data depends on requested data schema
  for (var i = 0; i < allCatalogs.length; i++) {
    jsonData.catalogs[i].dataProducts.forEach(
      element => allDataProducts.push(element.id)
    );
  }

  pm.environment.set("allCatalogs", allCatalogs);
  pm.environment.set("allDataProducts", allDataProducts);
}
```

For the `format` parameter, we will set all available values into an environment variable. Unfortunately, if we manually set data into Postman variables as an array type, it will be set as a string (data type saves into variable’s value only by `pm.environment.set()` script). We should take this behavior into account in the next steps.

### 4. Run preconditions

We should run preconditions separately from our main request to reduce the load and avoid extra queries not affecting the final result.

Before starting, we have no data in our environment:

![Environment quick look](/assets/2020-11-22/03-environment-quick-look.png)

_Fig. 3. Environment quick look_

In the [Collection Runner](https://learning.postman.com/docs/running-collections/intro-to-collection-runs/), we check only precondition handles:

![We set a few iterations (10) to overcome some inconsistencies in the data](/assets/2020-11-22/04-as-mentioned-above.png)

_Fig. 4. As mentioned above, we set a few iterations (10) to overcome some inconsistencies in the data_

The result of the run will fill environment variables with data:

![Environment variables](/assets/2020-11-22/05-environment-variables.png)

_Fig. 5. Environment variables_

### 5. Run filter handler with randomized parameters

Part of the randomized parameters will get from environment variables and the other part we will calculate in the precondition script. For boolean and integer parameters, we are lucky to use Dynamic variables instead of writing our own code:

To randomize parameters, we can just take a random item from the array which gets from the environment variable:

![Params tab](/assets/2020-11-22/06-params-tab.png)

_Fig. 6. Params tab_

```JavaScript
let preconditionItemFromArray = preconditionArray => {
  let tempArray = pm.environment.get(`${preconditionArray}`);
  // https://stackoverflow.com/a/5915122
  return tempArray[Math.floor(Math.random() * tempArray.length)];
};
```

Now, when you run requests in the Collection Runner, combination values of Iterations and Delay could simulate primitive load testing (especially considering random requests):

![Especially considering random requests](/assets/2020-11-22/07.png)

![Run’s results](/assets/2020-11-22/08-runs-results.png)

_Fig. 8. Run’s results_

I highly recommend setting «[Request timeout in ms](https://learning.postman.com/docs/getting-started/settings/#request)» in Postman settings to prevent the runner’s freezing if some requests execute unexpectedly long.

Copy @ [Medium](https://adequatica.medium.com/randomization-testing-of-filter-handler-in-postman-5cc37432602c)
