---
layout: post
title: "Layers of Defense Against Data Modification"
date: 2022-12-22 17:45:12 +0100
tags: testing
---

Sometimes, as a test engineer, you have to perform testing under a knotty environment and not affect production with your activity. Autotests can help, but they require thorough preparations.

Before diving into the topic of autotests, I need to explain the architecture of my project and its testing environments. Without this context, it will be unclear what problem is solved here:

- I have a cluster of hundreds of IoT devices;
- They send data and receive commands from «Control service»;
- Data passes through «Control service» and is stored in the in-memory database;
- Our backend gets devices’ data from the database, transforms it according to business logic, and provides it to our frontend;
- Our frontend displays transformed data to users and triggers commands to devices through «Control service».

![Architecture](/assets/2022-12-22/01-architecture.png)

Subtotal, IoT devices send their telemetry (coordinates, fuel, temperature, etc.) to «Control service» → «Control service» stores this data in the database → backend takes this data and returns into frontend by HTTP API → frontend, as a web application, shows data in the interface in a browser → users can see devices’ status and hit commands to any device → commands go to «Control service» which triggers devices to some action (beep, move, reboot, etc.).

From a bird’s eye view, the project does not look complicated in addition, **«Control service» and behavior of IoT devices are out of testing scope**, but testing environments are tricky:

- A normal testing environment has test versions of frontend, backend, database, «Control service» and a small cluster of emulated devices — _by default_ testing environment is isolated from production;
- Prestable (or pre-production) environment has test versions of frontend, backend, and database — prestable environment is not isolated from production;
- Prestable environment uses production «Control service» ⇒ the prestable database contains the same data as the production one, and the prestable frontend will trigger commands to production devices!
- Developers can switch testing and presentable backends/databases, which breaks the default setup. For example, frontend in testing environment can start using a prestable’s backend/database, and because of this commands to testing devices will fail.

![Environments](/assets/2022-12-22/02-environments.png)

Of course, many testing problems can be solved by having a few real devices for testing purposes in production, but unfortunately it is not possible due to some business reasons.

---

Next, I will cover only a part of automation testing of this project, namely **frontend automation testing based on [Playwright](https://playwright.dev/).**

One of the purposes of my autotests, besides regression testing, is **to prevent accidental commands on production devices from any non-production environment**, but at the same time, autotests should run in both testing and prestable environments (ideally also in production).

Thus my automation strategy comes from the «zero trust» concept, that any mocks or aborted requests can be overcome. I have to distrust any environment setup and expose additional layers of protection against data modification on IoT production devices.

## Layer 1: environmental restrictions — skip tests in undesirable environment

Some tests should be allowed to run only in a certain environment. This may be connected to limitations of the environment or severity of tests.

> For example of my project: if I have a command for turning the device off in the autotest, then I really do not want it to be accidentally executed outside the testing environment.

For this case, [Playwright allows skip certain tests based on the condition](https://playwright.dev/docs/test-annotations#conditionally-skip-a-test) (my condition is an environment based on the URL):

```JavaScript
test('Should turn off the device', async ({ page, HOST }) => {
  test.skip(HOST !== 'https://testing.domain', 'Skip test in non-testing env');
  …
});
```

As I mentioned earlier, I do not trust the environment setup, because the testing environment can be changed and not be isolated from production data. Therefore, I need more safety net options.

## Layer 2: user restrictions — forbid actions from test users

If you can limit access to certain features for certain users — be sure to do it for your test users. Do not grant full access to test uses if they are used in autotests. Each test user should have only one particular role for the execution of only one particular action in a limited set of autotests.

> For example of my project: if I have a command for turning the device off in the autotest by a test user, then I limit the access to trigger such kinds of commands by that test user on the level of «Control service». It turns out that the test user will send the command, but it will not be executed.

This case applies not only to autotests — it is a good practice to prevent the execution of critical actions by random users or hide functionality from them, but it can be implemented only in applications with a complex role-based access control model.

## Layer 3: test objects restrictions — choose idle testable objects

This item does not refer to restricting, but sanitary measures. To reduce the impact of unexpected commands or data modification, if they do happen, it is worth using less important or popular/loaded objects for testing. Сhoose the least noticeable objects for testing in a prestable/production environment.

> For example of my project: I try to select devices that are offline or hidden, not just randomly.

## Layer 4: mock responses — use synthetic data

Actually, your UI autotests can be completely based on mocks, and you probably do not need anything at all that I wrote about earlier.

> But it is not that simple.

For example of my project, I have two cases:

- I mock responses from the backend for a random non-existent device with required parameters. Due to this, autotests will not be able to trigger a command on a real device;
- Layers №2 «user restrictions», №3 «test object restrictions» and mocks from that layer can cause undesirable responses after producing actions (clicking on buttons) which I want to avoid for passing a test.

If a test user does not have access to a certain command, but still tries to call it (or the call is directed to a non-existent device), he gets a response with [403 Forbidden](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403) status code — so, I need to mock [200 OK](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200) response for successful pass of the autotest.

[Playwright’s API for modification network traffic](https://playwright.dev/docs/network#modify-responses) is perfectly fit for that (override status code and response body of a specified response):

```JavaScript
await page.route('https://testing.domain/handler', (route) =>
  route.fulfill({
    status: 200,
    body: `${mockedDevice}`,
  }),
);
```

## Layer 5: request restrictions — prevent requests from frontend to backend

The final restriction layer is to prevent requests to protected application or service. As an ultimate solution, requests can be terminated on a network level through proxy settings, but it is not an option if you still need to make requests from time to time in some cases.

> For example of my project: after all listed layers, my assumed autotests will run only in a testing environment, test users will be forbidden to trigger commands, and commands will be sent to an offline or non-existent device, but what if all of the above will fail?

For this case, [Playwright can abort HTTP requests](https://playwright.dev/docs/network#abort-requests):

```JavaScript
await page.route('https://control.servise/handler', (route) => {
  if (route.request().method() === 'POST') {
    route.abort();
    return;
  }
});
```

I am placing the sample code in the [beforeAll](https://playwright.dev/docs/api/class-test#test-before-all) hook of my autotests and none of the specified requests reach the server during a test-runs. Which means none of the actions lead to undesirable consequences.

---

If you have a similar problem with testing environments, you can build your own defensive layers against data modification. Just do not trust your test data, test users and so on.

Copy @ [Medium](https://adequatica.medium.com/layers-of-defense-against-data-modification-d73e9e93bdf7)
