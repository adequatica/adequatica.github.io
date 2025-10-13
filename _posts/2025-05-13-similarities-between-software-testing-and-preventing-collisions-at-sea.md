---
layout: post
title: "Similarities Between Software Testing and Preventing Collisions at Sea"
date: 2025-05-13 04:30:32 +0200
tags: testing
---

Sometimes, as a test engineer, you may notice a close connection between software testing practices and something completely different.

![GPT-image-1 prompt](/assets/2025-05-13/00-colregs-art.jpg)

_GPT-image-1 prompt: A modern sailing yacht with three tall white sails, sailing in the open sea at sunrise or sunset. In the background, a massive container ship is loaded with cargo containers. The lighting is soft, casting reflections on the water. Make it is a marine art painting. Show the beauty of the sea waves._

If you are a sailing enthusiast, you must learn a set of special rules for marine navigation. These rules turn out to be closely related to the day-to-day work of a QA engineer. [The International Regulations for Preventing Collisions at Sea](https://en.wikipedia.org/wiki/International_Regulations_for_Preventing_Collisions_at_Sea) (COLREGs) are surprisingly similar to software engineering. Designed to keep vessels safe in unpredictable conditions, these rules neatly map onto the realities of testing, release processing, updating and upgrading systems, and incident management. This post cherry-picks and reinterprets some of the most relevant COLREGs rules through the lens of software testing.

## Rule 2 Responsibility

> (a) Nothing in these Rules shall exonerate any vessel, or the owner, master, or crew thereof, from the consequences of any neglect to comply with these Rules or of the neglect of any precaution which may be required by the ordinary practice of seamen, or by the special circumstances of the case.
>
> (b) In construing and complying with these Rules due regard shall be had to all dangers of navigation and collision and to any special circumstances, including the limitations of the vessels involved, which may make a departure from these Rules necessary to avoid immediate danger.

Interpretation of these articles of the Rules to «normal» language means that if a collision does occur, neither captain can say, «I was right». In maritime collisions, both vessels are considered responsible. Item (b) explains why: if the only way to avoid a collision is to break the Rules, then the Rules must be broken.

The same principle should apply to review of incidents in production. Responsibility should not be pinned solely on developers, testers, ops teams, or management who approved the release. Everyone shares responsibility. Instead of finding someone to blame, you should focus on learning from the incident on how to prevent such a situation in the future. This may mean adding more tests, improving release processes, clarifying task requirements, and developing the soft skills necessary to communicate problems early and effectively.

However, _sometimes_ you may break established procedures to meet important product requirements (or goals). _Sometimes_ you have to be extremely agile, which means knowing when and how you can bend your own rules — like shipping with failing tests or releasing on weekends — if the circumstances demand it.

## Rule 7 Risk of collision

> (a) Every vessel shall use all available means appropriate to the prevailing circumstances and conditions to determine if risk of collision exists. If there is any doubt such risk shall be deemed to exist.

In the context of software testing, this article on the Rules can be read as you must use every available metric to monitor the system — if some parameter can be monitored, it should be. If it looks safe to run tests in production, do it. If you can set up alerts that track critical values of your application, please do so as well. Do not rely on a single tool or one golden metric — use a variety of them.

If one metric on a dashboard starts spiking while others remain flat, immediately assume that something is wrong; do not wait for a failure of another system’s metric because many signals lag behind real issues.

Keep in mind that just because our dashboards are green does not mean everything is fine.

## Rule 8 Action to avoid collision

> (a) Any action taken to avoid collision shall, if the circumstances of the case admit, be positive, made in ample time and with due regard to the observance of good seamanship.

Here, «positive and made in ample time» may apply to testing for a case when you have doubts about the quality of a task; you should not release it. If you are unsure about the stability of the released version, you should roll it back immediately.

«Good seamanship» can be interpreted as [good engineering practices](https://en.wikipedia.org/wiki/Good_engineering_practice): writing unit tests, maintaining clear documentation, or following settled dogmas like not deploying on a Friday or right before a holiday season.

> (b) Any alteration of course and/or speed to avoid collision shall, if the circumstances of the case admit, be large enough to be readily apparent to another vessel observing visually or by radar; a succession of small alterations of course and/or speed should be avoided.

For software engineering this rule suggests that if a developer needs to make changes to isolate or identify a bug, she should make one significant change rather than many small ones.

A clear, noticeable change is more effective for debugging the root cause — it is easier to observe, reason about, and measure. Small tweaks in multiple places can obscure the signal, making it harder to tell what actually had an effect.

## Rule 13 Overtaking and rule 14 Head-on situation

> (c) When a vessel is in any doubt as to whether she is overtaking another, she shall assume that this is the case and act accordingly.

And

> (c) When a vessel is in any doubt as to whether such a situation exists she shall assume that it does exist and act accordingly.

In quality assurance, just like at sea, doubt is not allowed. **Any uncertainty must be treated as a worst-case scenario.**

If you think (or feel) there might be a bug, assume it exists. There is no harm in filing an invalid ticket. If it turns out to be nothing, you will close it with a «Won’t fix» resolution. That is a much safer outcome than ignoring a real issue because you were not sure enough.

## Rule 17 Action by stand-on vessel

> (b) When, from any cause, the vessel required to keep her course and speed finds herself so close that collision cannot be avoided by the action of the give-way vessel alone, she shall take such action as will best aid to avoid collision.

This is a great example of thoughtful mentorship. If you see a junior engineer about to take a risky and inappropriate action, sometimes it is valuable to let her proceed so that she can learn firsthand from the consequences. But at the same time, you should be ready to step in and prevent the actual failure of the production system. This is not about micromanagement, this is about intervening in exceptional situations.

## Rule 18 Responsibilities between vessels

> (d) (i) Any vessel other than a vessel not under command or a vessel restricted in her ability to manoeuvre shall, if the circumstances of the case admit, avoid impeding the safe passage of a vessel constrained by her draught.
> (ii) A vessel constrained by her draught shall navigate with particular caution having full regard to her special condition.

In terms of testing, «exhibiting the signals» means that during release, the current status of the release (or its main task) should be explicit. Proactively informing the team and stakeholders about an upcoming or ongoing release — one of the riskiest moments in the lifecycle of any system — is simply a good practice. Visibility helps everyone navigate more safely.

«With particular caution having full regard to her special condition» translates as not all releases are the same. Some may depend on the timing of other deployments, align with specific business events, or intentionally break from standard procedures. If a release has special constraints, it must be handled with special care.

## Rule 19 Conduct of vessels in restricted visibility

> (b) Every vessel shall proceed at a safe speed adapted to the prevailing circumstances and conditions of restricted visibility. A power-driven vessel shall have her engines ready for immediate manoeuvre.

You must be prepared for things to go wrong — whether it is a failed release, a critical bug, or a flawed task that slipped through.

In uncertain or high-risk conditions, you should operate at a safe pace and be ready to act immediately. That means knowing how to quickly roll back a release, revert a change, or disable a broken feature in a production environment, and having the tools and processes in place to do so without hesitation.

---

The final decision always belongs to the captain, who is usually a senior software engineer, team lead, or C-level manager. No matter how democratic the team is, someone with experience, knowledge, skills, and the willingness to take responsibility should be placed in charge when it comes to crucial choices, like rolling out a long-awaited but problematic release, prioritizing of equally important tasks, or changing the technology stack.

Copy @ [Medium](https://adequatica.medium.com/correlation-between-software-testing-and-preventing-collisions-at-sea-3902a934d327)
