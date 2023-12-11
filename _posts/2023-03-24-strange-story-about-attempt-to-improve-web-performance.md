---
layout: post
title: "Strange Story About Attempt to Improve Web Performance"
date: 2023-03-24 07:30:32 +0100
tags: performance
---

Sometimes, as a test engineer, you have to argue with product owners about seemingly obvious things.

It was at the time when I was testing a media web service with around 16 million average monthly visitors.

There was a legacy page with a list of snippets of some content (images and descriptions) and a [Show more] button at the bottom.

![Mockup](/assets/2023-03-24/01-mockup.png)

_Mockup of a service’s page with performance issues._

The problem with such a simple page was its performance — it took over 10 sec to load just a bunch of snippets. Why?

The investigation showed:

1. [Lighthouse performance audit](https://developer.chrome.com/docs/lighthouse/performance/performance-scoring/) gave a score = 1!
2. Frontend requested only one backend handler for data, but
   1. this handler responded with a 9 Mb body (JSON file), which took to load for nearly 1 sec. A client (user’s browser) had to parse this file to render it, which, of course, took a long time due to the large amounts of data;
   2. JSON data contained too much unnecessary information. Frontend used only a fraction of the data for the snippet, but dozens of key-value pairs remained unused.
3. And it turned out to be much worse: if the user clicked on the [Show more] button, the page opened with about three thousand snippets (that is all snippets contained in a 9 Mb body without pagination). 3. And for all thousands of snippets, image thumbnails began to load simultaneously. User could get around 100 Mb with no reason.

That sounds terrifying, especially for users with poor and not unlimited internet and low-performance devices.

A development team quickly found a number of the easiest ways of improvements (less efforts / maximum profit):

1. Load only necessary data from backend (even without pagination we could reduce size of the body for 50%);
2. Add real pagination on frontend and backend (load data by chunks and only when it required);
3. Implement [lazy loading](https://web.dev/lazy-loading-images/) for images.

We (the engineering team) asked product owners to give us time to fix those issues (we came up with the described tasks and time estimates), but got the answer «_no_».

Strangely, maybe we did not explain the severity of the problem well?

Then, we provided a list of examples of the relationship between page load speed and conversion:

- [Pinterest](https://medium.com/pinterest-engineering/driving-user-growth-with-performance-improvements-cfc50dafadd7) increased search engine traffic and sign-ups by 15% when they reduced perceived wait times by 40%;
- [COOK](https://blog.eggplantsoftware.com/case-studies/cook-increases-conversions-by-seven-percent-thanks-to-faster-load-time) increased conversions by 7%, decreased bounce rates by 7%, and increased pages per session by 10% when they reduced average page load time by 850 milliseconds;
- [BBC](https://www.creativebloq.com/features/how-the-bbc-builds-websites-that-scale) found they lost an additional 10% of users for every additional second their site took to load;
- [DoubleClick by Google](https://www.thinkwithgoogle.com/consumer-insights/consumer-trends/mobile-site-load-time-statistics/) found 53% of mobile site visits were abandoned if a page took longer than 3 seconds to load;
- [AutoAnything](https://www.digitalcommerce360.com/2010/08/19/web-accelerator-revs-conversion-and-sales-autoanything/) reduced page load time by half and saw a boost of 12% to 13% in sales;
- Mobify found that decreasing their homepage’s load time by 100 milliseconds resulted in a 1,11% uptick in session-based conversion;
- Walmart discovered that improving page load time by one second increased conversions by 2%;
- 1 second of load lag time would cost [Amazon](https://www.fastcompany.com/1825005/how-one-second-could-cost-amazon-16-billion-sales) $1,6 billion in sales per year;
- Amazon increased revenue by 1% for every 100 ms of improvement;
- A lag time of 400 ms results in a decrease of 0,44 % traffic;
- For every 100 ms of improvement, they grew incremental revenue by up to 1%;
- 47% of consumers expect a web page to load in 2 seconds or less;
- 40% of people abandon a website that takes more than 3 seconds to load;
- Yahoo increased traffic by 9% for every 400 ms of improvement
- Mozilla got 60 million more Firefox downloads per year, by making their pages 2,2 seconds faster;
- Shopzilla says, that reducing page load time from 6 seconds to 1,2 seconds delivers a 12% increase in revenue and a 25% increase in pages viewed;
- 79% of customers who run into any kind of performance issues on your website are less likely to buy from you again;
- 0–4 second load time is best for conversion rates;
- The first 5 seconds of page load time have the highest impact on conversion rates;
- [A site that loads](https://www.portent.com/blog/analytics/research-site-speed-hurting-everyones-revenue.htm) in 1 second has a conversion rate 5x higher than a site that loads in 10 seconds.

Did it convince product owners that performance matters? — No!

The answer was like: «_We understand your concern, but so far our user base is growing linearly, we are not going to invest resources into fixing technical debts_».

![Down with technical debt, while there is growth](/assets/2023-03-24/02-graph-growth.png)

_What this graph does not say is that if there are no problems now, it does not mean that they will not appear in the future._

It was incredibly frustrating (why not taking care of a technical debt is bad is not the topic of this story). What engineers saw as a problem and thought that it affects the product and users, managers did not even consider it as an issue requiring attention.

I suppose that product owners were not conscious enough, but developers were — they fixed most of the performance issues in their spare time within a couple of months.

Further reading:

- [Why does site speed matter](https://developers.google.com/web/fundamentals/performance/why-performance-matters)?
- [WPO Stats](https://wpostats.com/) — case studies and experiments demonstrating the impact of web performance optimization (WPO) on user experience and business metrics;
- [The Psychology of Web Performance](https://www.oreilly.com/library/view/time-is-money/9781491928783/ch01.html) — «Time Is Money» by Tammy Everts;
- [How long is too long](https://developer.mozilla.org/en-US/docs/Web/Performance/How_long_is_too_long)?

Font for mockup illustration: [Norum Ipnum](https://www.ideo.com/blog/use-these-dummy-numbers-when-prototyping-with-data).

Copy @ [Medium](https://adequatica.medium.com/strange-story-about-attempt-to-improve-web-performance-7f699afbe126)
