# What LLMs Mean for Developer Careers

*Part 6 of a series on coding with LLMs. Previously: [The Elaborate Hallucination: Where LLMs Fail Spectacularly](/coding-with-llms/coding-with-llms-failures-and-limits).*

Throughout this series I've argued that LLMs are a multiplier, not an addition. In this final article, I want to trace what that multiplier effect means for developers at different career stages, and for the industry overall.

The short version: this is good news for developers, but the distribution of that good news is very uneven.

## Senior Developers Are the New Bottleneck

If LLMs multiply your existing ability, then the people with the most ability to multiply get the most benefit. In practice, this means senior developers — the people who understand the codebase deeply, can spot bad abstractions, know when the LLM's output is subtly wrong — are becoming dramatically more productive.

A senior developer with Claude Code can now produce in a week what used to take a month. Not because they type faster, but because the [tedious parts](/coding-with-llms/coding-with-llms-death-of-procrastination) — the mechanical refactorings, the boilerplate, the migrations — no longer consume their time. They spend their hours on architecture, review, and decision-making, while the LLM handles the rest.

This creates a new bottleneck: **review capacity**.

When a senior developer was writing 200 lines of code per day by hand, review throughput wasn't a constraint. Now, when LLMs produce thousands of lines per day, someone needs to verify all of it. And "someone" can only be another senior developer — because the [failure modes](/coding-with-llms/coding-with-llms-failures-and-limits) I described earlier are often invisible to less experienced eyes. The code compiles, the tests pass (sometimes because the tests are as wrong as the code), and the bugs are architectural rather than syntactic.

Senior developers will become the critical path in every engineering organization. Their ability to review, validate, and course-correct LLM output will determine the team's actual throughput. Companies that understand this will invest heavily in their senior talent. Companies that don't will discover that 10× raw output with 0.5× review capacity produces worse results than no LLMs at all.

## The Problem for Mid-Level and Junior Developers

This is the uncomfortable part.

The traditional model of developer career growth works like this: you hire juniors, assign them tasks at the edge of their ability, invest in mentoring and code review, and over a few years they become mid-level developers. You do the same with mids, and eventually they become seniors. It's expensive and slow, but it works.

LLMs disrupt this pipeline in two ways.

**First, the easy tasks are gone.** The tasks that were traditionally assigned to junior and mid-level developers — well-scoped features, bug fixes in familiar code, mechanical refactorings — are exactly the tasks that LLMs handle best. A senior developer can now hand these to an LLM instead of a junior, and get the result back in hours instead of days, without needing to block on code review of a junior's output.

**Second, the review cost equation has flipped.** Previously, reviewing a junior's code was an investment — you were teaching them, building their skills, and getting work done (slowly) in the process. Now, a senior developer's review bandwidth is their scarcest resource. Spending it on reviewing a junior's code competes directly with reviewing LLM output — and the LLM produces more, faster, with fewer back-and-forth cycles.

I want to be clear: I don't think this means companies will stop hiring junior developers. The industry still needs to grow its senior talent pipeline, and that requires investing in juniors. But the calculus has changed. The investment period before a junior developer becomes net-positive is getting longer, because the bar for "net positive" has risen — you're now comparing their output to what a senior developer could produce by delegating to an LLM.

For mid-level developers, the situation is arguably tougher. Juniors are cheap enough that the investment argument still works. But a mid-level developer is expected to be independently productive — and "independently productive" now means "more productive than a senior developer with an LLM for the tasks in question." That's a significantly higher bar than it was two years ago.

## The Skill Premium Gets Steeper

The multiplier framing makes the career math very clear.

If LLMs are a 5-10× multiplier on your effective output, then the difference between developers isn't additive anymore — it's multiplicative. A developer who produces 2× the quality decisions per hour doesn't just get 2× the LLM benefit. They get 2× times the LLM multiplier — so 10-20× the effective output of a developer with half their skill.

This means the return on investment in your own skill development has gone up dramatically. Every increment in your ability to:

- Recognize bad architecture in LLM output
- Decompose problems into LLM-appropriate subtasks
- Write precise specifications that minimize hallucination
- Spot the gap between "tests pass" and "code is correct"

...gets multiplied by the LLM's amplification factor. The gap between "good developer" and "great developer" was already significant. Now it's an order of magnitude.

If you're early in your career, this is actually encouraging news — provided you take it seriously. The payoff for genuine skill development has never been higher. But "skill" in this context means deep understanding of systems, architecture, and failure modes — not familiarity with a specific framework or the ability to write boilerplate quickly. The latter is exactly what LLMs commoditize.

## Software Gets More Complex — And That's Good

I touched on this in the [first article](/coding-with-llms/coding-with-llms-multiplier-not-addition), but it bears expanding here because it has direct career implications.

Most production software is built on foundations that nobody wants to touch. Layer after layer of tactical decisions — each reasonable at the time, collectively a strategic disaster. The codebase works, but it works slowly and painfully, because the accumulated technical debt makes every change harder than it needs to be. Every senior developer knows the feeling: "this should take a day but it'll take a week because of the way the foundation was built five years ago."

These foundations have been untouchable because the cost of restructuring them was prohibitive. A rewrite of the storage layer that touches every service endpoint? That's a quarter of engineering time for a quarter, with significant regression risk. No reasonable company would approve it.

LLMs change this math. The [mechanical work](/coding-with-llms/coding-with-llms-death-of-procrastination) of propagating changes across hundreds of files — the thing that made these refactorings prohibitively expensive — is now cheap. The test generation that makes such refactorings safe is also cheap. What remains expensive is the *judgment* — deciding what the new architecture should look like, which invariants must be maintained, how to sequence the migration. And that judgment is exactly what experienced developers provide.

The implication: a lot of software that has been limping along on "sticks and mud" foundations is about to get rebuilt. Not rewritten from scratch — that's usually a mistake — but incrementally restructured in ways that were previously unaffordable. This is a genuine win. For startups, it means building better foundations faster. For large companies, it means products that can evolve instead of calcifying.

It also means that "today's complexity" becomes "tomorrow's baseline." When current hard problems become tractable, people tackle harder ones. The products we consider complex now will look relatively straightforward in a few years. The overall complexity of software will increase — not because we like complexity, but because we can now manage more of it. And this means sustained demand for developers who can navigate and reason about that complexity.

## The Fun Is Coming Back

I'm going to end this series where I ended the first article — on a personal note, because I think it captures something real.

When you've been programming for many years, you develop an internal classifier that labels tasks as "routine" or "interesting." Over time, the "routine" bucket grows and the "interesting" bucket shrinks. The stereotypical senior developer — the slightly cynical one who looks at every new task with mild disgust — exists because, after enough years, almost everything is something you've seen before. You end up owning more and more of the routine work (migrations, upgrades, maintenance, boilerplate) because you're the one who can do it right, and it slowly crushes the joy out of the work.

Over the past few months, something has changed. The most tedious, soul-numbing tasks — the ones that used to loom over me for weeks — now take an evening. The procrastination is gone. The dread is gone. What's left are the genuinely interesting problems: architecture decisions, subtle bugs, design questions that actually require human judgment.

This isn't permanent, of course. Eventually the definition of "routine" will shift upward, and new categories of tasks will feel tedious. But that adjustment will take years. In the meantime, we're living through a period where decades of accumulated "I really should fix this but I can't face it" are suddenly tractable.

For every experienced developer who's been slowly losing enthusiasm — who remembers when every day was a discovery and misses that feeling — I think the next few years are going to be unexpectedly good.

The tools are a multiplier. Invest in the skills that get multiplied. Use the tools aggressively. The developers who master this workflow will outpace the ones who don't by an order of magnitude — and for the first time in a long time, the work will be fun again.

---

*This is the final article in the "Coding with LLMs" series. Start from the beginning: [×10, Not +10: What LLMs Actually Do to Developer Productivity](/coding-with-llms/coding-with-llms-multiplier-not-addition).*
