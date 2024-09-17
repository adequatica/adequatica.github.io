---
layout: post
title: "Practical Usage of Pairwise Testing"
date: 2024-09-17 19:30:49 +0100
tags: testing
---

Using pairwise test design technique to represent components in Storybook.

![DALL-E 3 prompt](/assets/2024-09-17/00-cover-dall-e-3.jpg)

_DALL-E 3 prompt: you are a software engineer. You have a set of data, battery levels: 0%, 19%, 20%, and 100%, isWireles, isConnected, and isCharging. Based on this data, render various UI components of the battery_

Recently, I’ve been mentoring a junior test engineer who asked me a question: «How do we apply pairwise testing?» I was confused because, from my point of view, it is so obvious that it does not need any explanations. But if the question was raised, it demands an answer.

According to the definition of **pairwise testing** (or [all-pairs testing](https://en.wikipedia.org/wiki/All-pairs_testing)), it is a type of **combinatorial testing** (2-way testing in combinatorial terminology) that, for each pair of input parameters to a system, tests all possible discrete combinations of those parameters.

But it does not clarify noobie questions: How do you choose pairs? How do you identify the proper set of input parameters? In which cases you should use this technique? All these questions can be answered only after a complex theoretical study.

Thus, for a brief response, I turned to a real-life example where we use pairwise testing in a current product. And there are such cases in Storybook’s components!

### Unfortunately, a little introduction to pairwise testing still be required:

Pairwise testing reduces test cases by examining all unique pairs of input variables rather than exhaustively testing every possible combination. This technique cuts down on testing time, resources, and the overall number of test cases required.

If your aim is 100% test coverage ([cover of all possible combinations](https://en.wikipedia.org/wiki/Combination)) of a system of _n_-parameters with _k_-combinations, and if the number of combinations for each parameter is fixed (which is quite rare for real-life production systems), then, according to the [multiplication principle of counting](https://openstax.org/books/contemporary-mathematics/pages/7-1-the-multiplication-rule-for-counting), you will have to perform _k^n_ tests. (For accurate combinatorial calculations, a [binomial coefficient](https://en.wikipedia.org/wiki/Binomial_coefficient) is used.)

A seemingly small system of six parameters with four values each requires the execution of 6^4 = 1296 tests for 100% coverage. Adding just one more parameter dramatically increases the number of tests: (6+1)^4 = 2401. A real-life production system with hundreds of parameters and dozens of values will demand tens of millions of tests for 100% coverage [1] — that is what it means: that exhaustive testing of all possible combinations is not always feasible [2].

Here comes all-pairs testing, which allows maximizing input coverage while minimizing the test suite size. Unhappily, with some limitations:

- You will not get 100% coverage [1] (although, in practice, you don’t need it);
- This approach is not effective for testing event sequences [1] (e.g., software with time-dependent logic) — this leads to absurdly many combinations;
- A large number of parameters made it time-consuming and effort-consuming. Again, this led to too many combinations, the execution of which would not allow you to provide any budget for testing.

Despite the flaws above, this technique is excellently applied to a system with a limited number of parameters and values. In this case, we can achieve a pretty decent test coverage with determined parameters rather than using random ones [3].

This system can be a [Storybook](https://storybook.js.org/), a development environment tool that is used as a playground for UI components. It allows the creation and testing of UI components in isolation. Each UI component (if it is not a composite one and is made just enough) usually gets a limited number of parameters ([`Props`](https://react.dev/learn/passing-props-to-a-component)) for rendering ⇒ <strong>you can predefine `Props` using a pairwise test design technique.</strong>

### A real-life example:

Our project has a component of the battery of the IoT device. The battery, in addition to the percentage of charge, can represent several other parameters: type (wireless or not), connection (connected to the charger or not), and charging status (charging or not, depending on the connection).

The next step after choosing the [system under test](https://en.wikipedia.org/wiki/System_under_test) (in our case, it is a battery component) is determining the possible parameters’ values:

- For battery level were chosen boundary values of UI’s states — `batteryLevel: 0`, `19`, `20`, and `100` (the «expert judgment» choice of values is justified and is a common practice [4]);
- Battery type — `isWireless: boolean`;
- Connection status — `isConnected: boolean`;
- Charging status — `isCharging: boolean`;
- In case both parameters `isConnected` and `isCharging` are `true`, UI should show a charging sign; if both are `false` — no sign; in other cases — an error sign.

Then, determined parameters and values are passed to a pairwise test case generator. There are a lot of [available tools](https://www.pairwise.org/tools.html), including web-based ones, such as [Pairwise Online Tool](https://pairwise.teremokgames.com/) and [Pairwise Pict Online](https://pairwise.yuuniworks.com/) powered by [Microsoft Pict](https://github.com/microsoft/pict). I chose the last one to generate test cases because of its severe mathematical base.

![Pairwise test cases](/assets/2024-09-17/01-pairwise-pict-online.png)

_Fig. 1. Pairwise test cases_

Onwards, test cases’ values are passed to the component in the story ([`battery.stories.tsx`](https://storybook.js.org/docs/writing-stories/args)) for rendering.

```JavaScript
import type { Meta, StoryObj } from '@storybook/react';

import { Battery } from './battery';

const meta: Meta<typeof Battery> = {
  component: Battery
};

export default meta;
type Story = StoryObj<typeof Battery>;

export const Primary: Story = {
  render: function Render() {
    return (
      <>
        <Battery
          batteryCharge={100}
          isWareless={true}
          isConnected={true}
          isCharging={true}
        />
        <Battery
          batteryCharge={0}
          isWareless={false}
          isConnected={false}
          isCharging={false}
        />
        {/* And so on… */}
        <Battery
          batteryCharge={20}
          isWareless={true}
          isConnected={false}
          isCharging={false}
        />
      </>
    );
  },
};
```

Now, after building a Storybook app, we can observe a battery component in different variants, conditioned by precalculated values.

![Pairwise test cases in Storybook’s story](/assets/2024-09-17/02-pairwise-battery-storybook.png)

_Fig. 2. Pairwise test cases in Storybook’s story_

Surprisingly, after specifying these values to represent a battery component, a bug was discovered in the component’s logic. So, the technique works!

---

For further development of the technique and enhancing test coverage, you can switch to full-scale combinatorial testing — instead of 2-way interactions (pair-wise), start using a tree- of four-way interactions with the IPOG algorithm [5]. But keep in mind this will significantly increase the number of combinations.

The Storybook itself is an excellent tool for [testing UIs](https://storybook.js.org/docs/writing-tests), but it is beyond the scope of this article.

### Read more:

- **[Pairwise Testing: A Best Practice That Isn’t](https://www.satisfice.com/download/pairwise-testing-a-best-practice-that-isnt) by James Bach and Patrick J. Schroeder;**
- [Introduction to Combinatorial Testing](https://personal.utdallas.edu/~ewong/SE6367/03-Lecture/29-A-Combinatorial-Testing-by-Kuhn.pdf) — presentation by Rick Kuhn, National Institute of Standards and Technology, Carnegie-Mellon University, 7 June 2011;
- [List of papers on combinatorial testing](https://csrc.nist.rip/Projects/automated-combinatorial-testing-for-software/acts-library/papers), National Institute of Standards and Technology (NIST);
- [Pairwise Testing](https://www.pairwise.org/);
- [Pairwise Testing Explained with Tools & Examples](https://www.testrail.com/blog/pairwise-testing/);
- [Pairwise Testing \| What It Is, When & How to Perform](https://testsigma.com/blog/pairwise-testing/)?
- [Pairwise Testing in the Real World: Practical Extensions to Test-Case Scenarios](<https://learn.microsoft.com/en-us/previous-versions/software-testing/cc150619(v=msdn.10)>).

### References:

1. D. Richard Kuhn, Dolores R. Wallace, Albert M. Gallo Jr., “[Software Fault Interactions and Implications for Software Testing](https://ieeexplore.ieee.org/document/1321063),” IEEE Transactions on Software Engineering, Volume: 30, Issue: 6, pp. 418–421, June 2004, DOI: 10.1109/TSE.2004.24.
2. Renée C. Bryce, Yu Lei, D. Richard Kuhn, Raghu Kacker, “[Combinatorial Testing](https://www.igi-global.com/chapter/combinatorial-testing/37033),” Ch. 14 in Handbook of Research on Software Engineering and Productivity Technologies: Implications of Globalization, edited by M. Ramachandran and R. Atem de Carvalho, IGI Global, 2010, pp. 196–208. DOI: 10.4018/978-1-60566-731-7.ch014.
3. D. Richard Kuhn, R.N. Kacker, Yu Lei, “[Random vs. combinatorial methods for discrete event simulation of a grid computer network](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=904044),” Proceedings of Modeling and Simulation, World, 2009, pp. 83–88.
4. Raghu N. Kacker, D. Richard Kuhn, Yu Lei, James F. Lawrence, “[Combinatorial testing for software: An adaptation of design of experiments](https://www.sciencedirect.com/science/article/abs/pii/S0263224113000596),” Measurement, Volume 46, Issue 9, November 2013, Pages 3745-3752.
5. Yu Lei, Raghu Kacker, D. Richard Kuhn, Vadim Okun, James Lawrence, “[IPOG: A General Strategy for T-Way Software Testing](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=50944),” 14th Annual IEEE International Conference and Workshop on Engineering of Computer Based Systems (ECBS 2007), 10 April 2007, DOI: 10.1109/ECBS.2007.47.

Copy @ [Medium](https://adequatica.medium.com/practical-usage-of-pairwise-testing-8a8d00f86b9b)
