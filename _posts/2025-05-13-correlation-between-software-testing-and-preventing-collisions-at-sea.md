---
layout: post
title: "Correlation Between Software Testing and Preventing Collisions at Sea"
date: 2025-05-13 04:30:32 +0200
tags: testing
---

Sometimes, as a test engineer, you may notice connections between software testing practices and other seemingly unrelated areas.

![GPT-image-1 prompt](/assets/2025-05-13/00-colregs.jpg)

_GPT-image-1 prompt: A modern blue and white sailing yacht with three tall white sails, sailing in the open sea at sunrise or sunset. In the background, a massive container ship is loaded with cargo containers. The lighting is soft, casting reflections on the water._

As a sailing enthusiast, I have had to learn a set of special rules, which I found closely related to my day-to-day work as a QA engineer. [The International Regulations for Preventing Collisions at Sea](https://en.wikipedia.org/wiki/International_Regulations_for_Preventing_Collisions_at_Sea) (COLREGs) turn out to be surprisingly related to software engineering. These rules, designed to keep vessels safe in unpredictable conditions, map neatly onto the realities of testing, release processing, updating systems, and incident responding. In this post, I cherry-picked the most relevant COLREGs and reinterpreted them through the lens of software testing.

## Rule 2 Responsibility

> (a) Nothing in these Rules shall exonerate any vessel, or the owner, master, or crew thereof, from the consequences of any neglect to comply with these Rules or of the neglect of any precaution which may be required by the ordinary practice of seamen, or by the special circumstances of the case.
>
> (b) In construing and complying with these Rules due regard shall be had to all dangers of navigation and collision and to any special circumstances, including the limitations of the vessels involved, which may make a departure from these Rules necessary to avoid immediate danger.

In other words, if a collision does occur, neither captain can simply say, «I was right». In maritime collisions, both vessels are considered responsible. Item (b) explains why: if the only way to avoid a collision is to break the Rules, then the Rules must be broken.

The same principle applies to software production incidents. Responsibility should not be pinned solely on developers, testers, ops teams, or management who approved the release. Everyone shares responsibility. Instead of finding someone to blame, we should focus on learning from the incident on how to prevent such a situation in the future. That means adding more tests, improving release processes, clarifying task requirements, and developing soft skills necessary to communicate problems early and effectively.

And _sometimes_ we must deviate from established procedures to meet important product requirements (or goals). _Sometimes_ we have to be extremely agile, which means knowing when and how we can bend our own rules — like shipping with failing tests or releasing on weekends — if the circumstances demand it.

## Rule 7 Risk of collision

> (a) Every vessel shall use all available means appropriate to the prevailing circumstances and conditions to determine if risk of collision exists. If there is any doubt such risk shall be deemed to exist.

We must use every available monitoring of the system — if some parameter can be monitored, it should be. If it looks safe to run tests in production, do it. If we can set up alerts that track critical values of our application, do that too. Do not rely on a single, universal tool or one golden metric — use a bunch of them.

If one metric on a dashboard starts to spike while others remain flat, immediately assume that something is wrong. Please do not wait for another system component to fail before we start to act. Many signals lag behind real issues.

Keep in mind, just because our dashboards are green does not mean everything is fine.

## Rule 8 Action to avoid collision

> (a) Any action taken to avoid collision shall, if the circumstances of the case admit, be positive, made in ample time and with due regard to the observance of good seamanship.

«Positive and made in ample time» may mean that if we have doubts about the quality of a task, we should not release it. If we are unsure about the stability of the released version, we roll it back immediately.

«Good seamanship» can be interpreted as **good engineering practices**: writing unit tests, maintaining clear documentation, or following team norms like not deploying on a Friday or right before a holiday season.

## Rule 8 Action to avoid collision

> (b) Any alteration of course and/or speed to avoid collision shall, if the circumstances of the case admit, be large enough to be readily apparent to another vessel observing visually or by radar; a succession of small alterations of course and/or speed should be avoided.

In the context of software testing, this rule suggests that if we need to make changes to isolate or identify a bug, we should make one significant change rather than many small ones.

A clear, noticeable change is more effective for debugging the root cause — it is easier to observe, reason about, and measure. Small tweaks in multiple places can obscure the signal, making it harder to tell what actually had an effect.

## Rule 13 Overtaking

> (c) When a vessel is in any doubt as to whether she is overtaking another, she shall assume that this is the case and act accordingly.

## Rule 14 Head-on situation

> (c) When a vessel is in any doubt as to whether such a situation exists she shall assume that it does exist and act accordingly.

In quality assurance, just like at sea, doubt is not allowed. **Any uncertainty must be treated as a worst-case scenario.**

If we think (or feel) there might be a bug, assume it exists. There is no harm in filing a ticket — if it turns out to be nothing, we will close it with a «Won’t fix» resolution. That is a much safer outcome than ignoring a real issue because we were not sure enough.

## Rule 17 Action by stand-on vessel

> (b) When, from any cause, the vessel required to keep her course and speed finds herself so close that collision cannot be avoided by the action of the give-way vessel alone, she shall take such action as will best aid to avoid collision.

This is a great example of thoughtful mentorship. If you see a junior engineer about to take a risky and inappropriate action, sometimes it is valuable to let her proceed so that she can learn firsthand from the consequences. But at the same time, you should be ready to step in and prevent the actual failure of the production system. This is not to micromanage or override every decision all the time; this is about intervening in exceptional situations.

## Rule 18 Responsibilities between vessels

> (d) (i) Any vessel other than a vessel not under command or a vessel restricted in her ability to manoeuvre shall, if the circumstances of the case admit, avoid impeding the safe passage of a vessel constrained by her draught, exhibiting the signals in Rule 28.
>
> (ii) A vessel constrained by her draught shall navigate with particular caution having full regard to her special condition.

«Exhibiting the signals» is a reminder that during releases, the current status of release or the main task should be explicit. Proactively informing the team and stakeholders about an upcoming or ongoing release — one of the riskiest moments in the lifecycle of any system — is simply good practice. Visibility helps everyone navigate more safely.

«With particular caution having full regard to her special condition» translates as not all releases are the same. Some may depend on the timing of other deployments, align with specific business events, or intentionally break from standard procedures. If a release has special constraints, it must be handled with special care.

## Rule 19 Conduct of vessels in restricted visibility

> (b) Every vessel shall proceed at a safe speed adapted to the prevailing circumstances and conditions of restricted visibility. A power-driven vessel shall have her engines ready for immediate manoeuvre.

We must be prepared for things to go wrong — whether it is a failed release, a critical bug, or a flawed task that slipped through.

In uncertain or high-risk conditions, we should operate at a safe pace and be ready to act immediately. That means knowing how to quickly roll back a release, revert a change, or disable a broken feature in a production environment, and having the tools and processes in place to do so without hesitation.

---

The final word always belongs to the captain — in our world, that is the senior software engineer, team lead, or some C-level manager. No matter how democratic the team may be, and regardless of how decisions are usually made, when it comes to crucial choices and decisions, someone with experience, knowledge, and skills (and willing to take the responsibility) should be placed in charge.

Copy @ [Medium](https://adequatica.medium.com/correlation-between-software-testing-and-preventing-collisions-at-sea-3902a934d327)
