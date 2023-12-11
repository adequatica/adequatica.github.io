---
layout: post
title: "JSON Data Validation Levels and its Usage in Postman"
date: 2022-07-19 05:58:02 +0400
tags: json postman testing
---

Testing JSON could be a part of contract testing — you check whether the response body corresponds to specification (or contract or terms of reference).

![alt_text](/assets/2022-07-19/00-cover.jpg)

The most appropriate way to do that is through [JSON Schema](https://json-schema.org/understanding-json-schema/) — the method of description of the structure of the JSON data.

Besides JSON Schema, each individual key/value pair can be checked for compliance with a number of criteria from the specification.

These criteria can be divided into validation levels:

1. JSON validation — low-level check;
2. Required keys;
3. Data types;
4. Default values;
5. Required values;
6. Additional criteria — high-level check.

Based on these levels, you can adjust the depth of testing of your JSON data.

---

### 1. JSON validation

This is an initial check that answers the question: _Is the body parsable?_

Validation could be done by JavaScript's standard method — [JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse) will throw an error in case of invalid JSON.

By the way, according to [JSON documentation](https://www.json.org/json-en.html), the empty object `{}` is a valid JSON.

### 2. Required keys

_Does a JSON object contain required key/value pairs?_

### 3. Data types

_What data type should have the value of a given key? Number, string, boolean, array, object, or null?_

### 4. Default values

This point is rarely specified in the documentation, but for some JSON keys, you have to figure out what is considered the correct value.

For example, suppose you get the response body:

```
{
  name: "Marcus Aurelius",
  school: "Stoicism"
}
```

But, _what if you get a philosopher without a particular school? What should be the default value? `school: ""` or the key should be missing?_

### 5. Required values

It could be anything related to documentation: formats of the dates, numerical boundaries, etc.

A good question to ask at this level: _Could the value be `undefined`?_ (preferably — not).

Another example: suppose you get the response body:

```
{
  name: "Marcus Aurelius",
  school: "Stoicism",
  occupation: ["philosopher", "emperor"]
}
```

If the required value of `occupation` key is `philosopher`, then each body should be checked for that the array `occupations[]` includes an item `"philosopher"`.

### 6. Additional criteria

This level is strongly tied up to specification and can not be universal for equivalent requests.

For example, suppose you get the response body:

```
{
  name: "Marcus Aurelius",
  school: "Stoicism",
  reign: [161, 180]
}
```

Exactly for this body, the `reign` array must include only 161 and 180 values. But for other cases, this particular check will be incorrect.

So, if you decide not to engage in high-level validation checks (additional criteria or required values), then your test will be more universal and less prone to false positives. But otherwise, you are more likely to miss a bug. It is always a trade-off.

---

## Use cases of validation levels

Each key/value pair in the response body could be checked both manually and automatically.

Sometimes, even if you do not need to write autotests for JSON validation, it is easier to write a script test than to validate the body with your own eyes. [Postman test scripts](https://learning.postman.com/docs/writing-scripts/test-scripts/) are the best solution for such kinds of tasks.

For the examples above, I took a simple response from an [OpenWeather API](https://openweathermap.org/api)’s request: `https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&units=metric&exclude=minutely,hourly,daily,current,allerts&APPID={api_key}`

```
{
  "lat": 40.1811,
  "lon": 44.5136,
  "timezone": "Asia/Yerevan",
  "timezone_offset": 14400
}
```

### How JSON data validation levels MAY look in a script

If you take each validation level and blindly impose it on the data structure, the test script may look like this:

```JavaScript
pm.test("1. JSON validation", function () {
  pm.response.to.be.json;
});

pm.test("2. Required keys", function () {
  let jsonData = pm.response.json();
  pm.expect(jsonData.lat).to.not.equal(undefined);
  pm.expect(jsonData.lon).to.not.equal(undefined);
  pm.expect(jsonData.timezone).to.not.equal(undefined);
  pm.expect(jsonData.timezone_offset).to.not.equal(undefined);
});

pm.test("3. Data type of each value", function () {
  let jsonData = pm.response.json();
  pm.expect(jsonData.lat).to.be.a('number');
  pm.expect(jsonData.lon).to.be.a('number');
  pm.expect(jsonData.timezone).to.be.a('string');
  pm.expect(jsonData.timezone_offset).to.be.a('number');
});

pm.test("4. Default values", function () {
  let jsonData = pm.response.json();
  // timezone string should contain something
  pm.expect(jsonData.timezone).to.have.lengthOf.above(0);
});

pm.test("5. Required values", function () {
  let jsonData = pm.response.json();
  pm.expect(jsonData.lat).to.be.within(-90, 90);
  pm.expect(jsonData.lon).to.be.within(-180, 180);
});

pm.test("6. Аdditional criteria", function () {
  let jsonData = pm.response.json();
  pm.expect(jsonData.timezone).to.have.string('Yerevan');
});
```

![How JSON data validation levels MAY look in a script](/assets/2022-07-19/01-may.png)

Such a script can be significantly improved by getting rid of redundant checks and refactoring the structure.

### How JSON data validation levels SHOULD look in a script

A more correct way is an individual checks of each value:

```JavaScript
let jsonData;

pm.test("Response body should be JSON", function () {
  // 1. JSON validation
  // Invalid JSON body will not be parsed
  pm.response.to.be.json;
  // Assigning the variable if JSON body is parsable
  jsonData = pm.response.json();
});

pm.test("lat key should be correct", function () {
  // 2-3. Required keys and data types
  // If the key is missing, the test will fail
  pm.expect(jsonData.lat).to.be.a('number');
  // 4-5. Default and required values
  pm.expect(jsonData.lat).to.be.within(-90, 90);
});

pm.test("lon key should be correct", function () {
  pm.expect(jsonData.lon).to.be.a('number');
  pm.expect(jsonData.lon).to.be.within(-180, 180);
});

pm.test("timezone key should be correct", function () {
  pm.expect(jsonData.timezone).to.be.a('string');
  // 6. Additional criteria
  pm.expect(jsonData.timezone).to.have.string('Yerevan');
});

pm.test("timezone key should be correct", function () {
  pm.expect(jsonData.timezone_offset).to.be.a('number');
});
```

![How JSON data validation levels SHOULD look in a script](/assets/2022-07-19/02-should.png)

### JSON data validation through JSON Schema

JSON Schema allows to test all validation levels at once!

Postman has a built-in tool for validation JSON Schema — [Tiny Validator](https://www.npmjs.com/package/tv4). To use it, you just need to define your JSON Schema as an object:

```JavaScript
pm.test("Schema is valid", function () {
  // 1. JSON validation
  // Invalid JSON body will not be parsed
  let jsonData = pm.response.json();

  let schema = {
    "type": "object",
    "properties": {
      "lat": {
        // 3. Data types
        "type": "number",
        // 4-5. Default and required values
        "minimum": -90,
        "maximum": 90
      },
      "lon": {
        "type": "number",
        "minimum": -180,
        "maximum": 180
      },
      "timezone": {
        "type": "string",
        // 6. Additional criteria
        "pattern": "Yerevan"
      },
      "timezone_offset": {
        "type": "number"
      }
    },
    // 2. Required keys
    "required": [
      "lat",
      "lon",
      "timezone",
      "timezone_offset"
    ]
  };

  pm.expect(tv4.validate(jsonData, schema)).to.be.true;
});
```

![Pass](/assets/2022-07-19/03-pass.png)

With a JSON Schema you can check the body in a fast and efficient way.

Unfortunately, in case of an error, you will not get an idea of what exactly is broken:

![Fail](/assets/2022-07-19/04-fail.png)

Actually, Tiny Validator is an outdated tool. If you are going to use schema validation in autotests, it is better for you to choose [Ajv](https://ajv.js.org/).

---

If your response body consists of XML format, all definitions of validation levels can be applied to it too.

Further reading:

- [Validating APIs](https://learning.postman.com/docs/designing-and-developing-your-api/validating-elements-against-schema/);
- [Json Schema and Json Validation](https://dev.to/anilkulkarni87/json-schema-and-json-validation-52f7);
- [Validating Data With JSON-Schema](https://code.tutsplus.com/tutorials/validating-data-with-json-schema-part-1--cms-25343).

Copy @ [Medium](https://adequatica.medium.com/json-data-validation-levels-and-its-usage-in-postman-7e110aff60c7)
