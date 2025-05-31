---
layout: post
title: "Benefits of Test Reports for Single Tasks Even for Positive Results"
date: 2024-03-10 07:49:00 +0100
tags: testing
---

In this article, the term «Test Report» refers to a QA’s comment after checking (testing) a single task (ticket).

As a test engineer, you have to test many things. The work’s result is a verified task, but sometimes it can be closed without visible work. So, it is a good practice to leave comments about the job done.

![DALL-E 3 prompt](/assets/2024-03-10/00-cover-dall-e-3.jpg)

_DALL-E 3 prompt: make the interface of a website for managing tickets with the comments section. It should have minimalistic UI, light and bright, very engineering look. Make it fullscreen._

## 1. Confirmation of the accomplished work

The result (outcome) of the developer’s work is Pull Requests and changes in the code; these are traceable artifacts of their activities, and results are countable and tangible.

The result (outcome) of the QA’s work (especially if it is manual testing) is challenging to quantify. Eventually, it only confirms that the feature works or a bug is fixed, but confirmation should have proof.

For instance, if a ticket comes for testing → something happens on the QA side → ticket closes (or returns for improvements). But what exactly has happened? What have the QAs done/tested/verified? Where is a presentation of the work done? Why not close the ticket without any testing?

Thus, **a test report is a valuable artifact of QA’s work.** A screenshot of a new web page or just one button, an HTTP response of a new or fixed handler — all of this matters.

In startups and small teams pursuing speed and agility, QAs can immediately close tickets without any comments if «everything is fine». This is handy for a short period of time, after which you have to refer to the old tickets from time to time.

_I encountered tickets in the Tested/Closed state without mentioning what had happened in the Testing state. Even the assignee and test engineer did not remember what they had done there, but another task depended on this ticket, and nobody knew what to do._

![Ticket without any evidence of testing work](/assets/2024-03-10/01-ticket-without-any-evidence-of-testing-work.png)

_Fig. 1. Ticket without any evidence of testing work_

![Ticket with QA’s comment](/assets/2024-03-10/02-ticket-with-qas-comment.png)

_Fig. 2. And the same ticket with QA’s comment_

This point can be counterargumented: Since everything is fine in the release and there are no new bugs in production, this indicates good testing work. But it could have happened if there had been no testing at all.

## 2. All team members can synchronize their expectations

Guided by the text report, all stakeholders will be able to verify the correctness of the task before it is released.

The developer will receive confirmation that she did everything right, the manager or analyst will receive confirmation that her task was completed according to specification, and the designer will receive confirmation that her mockup is pixel-perfect.

At least, the test report will be seen from a new angle (from the point of the task’s author) and can be interpreted differently than QA expected. This is why it can help double-check the result and prevent a misunderstood task from being released if someone considers the testing results incorrect.

_I have had cases when developers came after reading the test report and explained that everything was implemented differently than what was tested. Or managers would explain that they assumed something completely different from the final implementation. Both cases can be attributed to miscommunication problems or insufficient task descriptions. Still, such issues cannot be avoided even with strict development regulations._

## 3. Test reports may come in handy in the future

In 9 out of 10 tickets, no one will look at test reports, the attached screenshots, or examples of HTTP handler calls and responses, but in the remaining last one, these screenshots and examples may become crucial. The test report shows how the feature/system worked at the time of testing (see point №1), and this «time stamp from the past» may be wanted in the future.

I faced a few cases on this point:

_1. The successfully tested feature was broken in production. In the incident postmortem meeting, some team members tried to see a lack of testing, but the test report showed that the testing was performed flawlessly. In the end, it turned out to be an infrastructure incident, and no doubt fell on the QA team._

This is not as rare as it seems. This is not as rare as it seems. You never know when you will need these reports (or if you will ever need them), **but when you do, you will thank God that you did it.**

_2. A few months after the project change, a manager from a previous project came to me and asked me to explain how one particular functionality works. Of course, I had already forgotten the technical details. But thanks to the test report, we restored the feature’s functionality using screenshots and QA comments._

_3. A week ago, the entire team of one of our customers changed. The old team did not keep any documentation of their processes, and new people were trying to figure out the functionality of their legacy. The only documentation of the features they needed turned out to be test reports in tickets (and test cases in TMS) written more than a year ago.
Both cases above may represent poorly documented projects, but this does not diminish the value of test reports._

Both cases above may represent poorly documented projects, but this does not diminish the value of test reports.

### One more thing

Test report should not be just a single «OK» (it is no better than an immediately closed ticket; see point №1).

Examples of HTTP handler calls should show the URL of a testing environment, and screenshots should show the address bar to identify the URL of a testing environment and a browser - all this proves that the task was deployed and tested in the right place.

![Ticket without any evidence of testing work](/assets/2024-03-10/03-ticket-with-a-useless-comment.png)

_Fig. 3. Ticket with a useless comment_

![Ticket with QA’s comment](/assets/2024-03-10/04-ticket-with-a-helpful-test-report.png)

_Fig. 4. And the same ticket with a helpful test report_

---

On one of the projects, I commented about successful checks in each of my tickets, but one developer asked me not to do so: «If everything works, then why else comment on the ticket? This is an unnecessary action and distracts me.»

I answered: «I used to work in this way. I’m fully confident in why I do so, and it was profitable on all my previous projects. It doesn’t take much time. But I’ll stop doing it for your tickets if you don’t like it.»

After a while, he asked me to resume making test reports in his tickets because the rest of the team, including him, found such reports extremely valuable.

Copy @ [Medium](https://adequatica.medium.com/benefits-of-test-reports-for-single-tasks-even-for-positive-results-73bfd61b1d2e)
