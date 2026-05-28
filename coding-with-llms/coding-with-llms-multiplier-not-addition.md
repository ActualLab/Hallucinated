# ×10, Not +10: What LLMs Actually Do to Developer Productivity

*This is the first post in a series about using LLMs in software development. Upcoming posts will cover [the Docker sandbox setup](/coding-with-llms/coding-with-llms-docker-sandbox-setup), [where routine tasks go to die](/coding-with-llms/coding-with-llms-death-of-procrastination), [bug hunting through logs](/coding-with-llms/coding-with-llms-bug-hunting-logs), [where LLMs fail spectacularly](/coding-with-llms/coding-with-llms-failures-and-limits), and [what all of this means for developer careers](/coding-with-llms/coding-with-llms-developer-careers-future).*

There's a popular narrative right now that LLMs are becoming a new team member — a "+10" to your headcount. Give them a task, they write the code, you ship it. An extra pair of hands that never sleeps.

This is wrong. Not slightly wrong — fundamentally wrong. LLMs are not an addition. They're a multiplier. And the difference between ×10 and +10 changes everything about how companies, teams, and the entire software industry should think about these tools.

## The Multiplier, Not the Extra Pair of Hands

If you're a strong developer, LLMs make you dramatically stronger. If you're a weak one, they amplify your weaknesses just as faithfully. And if you can't write code at all, multiplying zero still gives you zero.

This isn't a metaphor. It's literally how these tools behave in practice. I've been using Claude Code as my primary AI coding tool for months now, across real production projects — not toy demos, not weekend prototypes. And the pattern is unmistakable: the value you get out is proportional to what you bring in.

A senior developer who understands the codebase, knows the architecture, can smell a bad abstraction from three files away — that person can now move at a pace that would have seemed absurd two years ago. They know what to ask for, they know when the output is wrong, and they know how to course-correct. The LLM becomes a force multiplier on judgment they already have.

A developer who doesn't have that foundation? The LLM will confidently generate code that looks right, compiles, even passes tests — and is fundamentally broken in ways that only surface weeks later. I've seen it firsthand. I'll share the [gory details](/coding-with-llms/coding-with-llms-failures-and-limits) later in this series, but for now: imagine getting a pull request that looks impressive, has 70-80 files, includes tests that pass — and by evening you realize you've been handed an elaborate hallucination. That's what "+10 thinking" gets you.

## The "+10" Illusion

So where does the "+10" narrative come from? It's not entirely baseless. There genuinely are scenarios where you can hand a task to an LLM and barely look at the result. But if you examine these cases honestly, they almost always boil down to boilerplate — tasks with very little wiggle room in how they can be done.

Add this attribute to 300 types. Rename this field across the codebase. Generate serialization annotations. Write CRUD endpoints matching this schema. These are tasks where the solution space is so constrained that even a probabilistic model lands on the right answer nearly every time.

And this is precisely what creates the illusion. People see the LLM nailing boilerplate and extrapolate: "It can write code! It's like having another developer!" But the moment you give it a task with real degrees of freedom — where the right answer depends on understanding deep relationships in your codebase, maintaining invariants across dozens of files, or making architectural trade-offs — the illusion shatters.

Even in the boilerplate cases, by the way, they can behave absurdly. I have a memorable example of ChatGPT flipping its answer on a straightforward technical question five times in a single conversation. Not refining its answer — reversing it completely, back and forth. Other people on Reddit reported the same question producing eight or more reversals. When challenged with "how many times have you changed your mind now?", it counted — nine times — and then promptly changed its mind again.

This is not "+10" behavior. This is not the behavior of a reliable extra team member. This is the behavior of a powerful but unreliable tool that requires a skilled operator.

## The Death of the Procrastination Tax

Now here's where the ×10 framing gets exciting, because the biggest multiplier isn't on the hard problems. It's on the ones you've been avoiding.

Every experienced developer carries a mental list of tasks they know need doing but can't bring themselves to start. The massive refactor that touches every file. The documentation that's been "on the roadmap" for two years. The migration to a new serialization format across 300+ types. The legacy cleanup that everyone agrees is necessary but nobody wants to own.

These tasks aren't hard in the intellectual sense. They're hard because they're soul-crushing. They're mechanical, repetitive, sprawling, and boring. Senior developers — the ones best equipped to do them right — are also the ones most likely to procrastinate on them, because their brains are wired for problem-solving, not pattern-stamping.

This is what I call the procrastination tax: the cumulative cost of all the improvements your team knows it should make but never does, because the effort-to-excitement ratio is catastrophic.

LLMs obliterate this tax. Not reduce it — obliterate it.

I generated nearly all of the documentation for [Fusion](https://github.com/ActualLab/Fusion) — an open-source library I maintain — in just over a week. If I'd written it myself, it would have taken at least two months. Not because the writing was hard, but because I would have dragged my feet the entire time.

I completed a major refactoring of our application — merging two similar but divergent user profile types that had accumulated over years — in three days. By hand, that's a two-week job minimum, and one that I'd been postponing precisely because of how tedious it was.

I added MessagePack serialization support across our entire codebase — over 300 types that needed proper annotation. The LLM found every type, wrote the tests, annotated them correctly, and got the tests passing. A human doing this would have spent days on something that requires zero creativity and maximum attention to detail — the worst possible combination for a human brain.

On tasks like these, the multiplier isn't 10×. It's 20× or 30×. And it's not just because the LLM works faster. It's because these are the tasks where humans work slowest — where we procrastinate, lose focus, make careless mistakes out of boredom, and take three weeks to do what should take three days.

## The Numbers

Let me put some rough numbers on this. Before LLMs, a productive developer adding around 200 lines of meaningful code per day was considered solid — accounting for code review, meetings, debugging, and all the other overhead of real work. That was a reasonable baseline.

With LLMs — specifically with Claude Code in a proper [Docker-sandboxed setup](/coding-with-llms/coding-with-llms-docker-sandbox-setup) — I've been averaging around 5,000-10,000 lines per week of reviewed, production-quality additions. That's roughly a 5-10× increase in raw output, and it's been sustained over months, not just on a good day.

But raw lines of code are a famously bad metric. The real shift is in what kinds of tasks become feasible. The documentation project wasn't blocked on typing speed — it was blocked on willpower. The refactoring wasn't blocked on skill — it was blocked on the sheer tedium of doing it correctly across hundreds of files. LLMs don't just make you type faster. They make previously unaffordable tasks affordable.

## Software Gets More Complex — And That's Good

Here's the industry-level implication that I think most people are missing.

Every experienced developer knows the feeling of working on a codebase that's been "built with sticks and mud" — layer upon layer of tactical decisions that looked reasonable at the time but accumulated into a strategic disaster. The foundation is so deep and so fragile that nobody dares touch it. Features that should take days take weeks because everything is coupled to everything else through years of accumulated workarounds.

Most production software is like this. It's not complex because the problems are complex. It's complex because there are a million small interlocking pieces — legacy code, compatibility shims, half-finished migrations, deprecated APIs that still run in production — and reworking any of it risks bringing down the whole house of cards.

LLMs change the economics of fixing this. Those titanic refactoring efforts that were previously unaffordable? They're now within reach. The ancient foundation that everyone's afraid to touch? You can actually go down there and restructure it, because the LLM can handle the mechanical work of propagating changes across hundreds of files, and generate the tests to make sure nothing breaks.

This is a genuine win for the entire industry. For startups, it means moving faster with smaller teams. For large companies, it means their products can actually evolve instead of calcifying under the weight of their own technical debt. Both of these ultimately benefit users, because the software they use every day gets better, faster.

But — and this is the part that's both exciting and sobering — it also means software will get more complex. Not because we want complexity, but because when today's complexity becomes manageable, people will naturally tackle harder problems. The products we consider complex today will look relatively simple in a few years. The bar just moves up.

## The Fun Is Back

I want to end on something personal, because I think it resonates with a lot of experienced developers.

When you've been writing code for many years, something happens gradually. You develop an internal classifier that, over time, labels an ever-growing share of tasks as "routine." When you were a beginner, everything was a discovery. That's why people fall in love with programming in the first place — every day brings something new.

But the further you go, the less novelty there is. More and more tasks trigger the "I've seen this before" response. And as your responsibilities grow, you end up owning more and more of the routine work — migrations, upgrades, maintenance, boilerplate. The stereotypical image of a grizzled senior developer isn't an accident: someone who looks at every new task with mild disgust, because nothing is new anymore.

Here's what surprised me most about the last few months: that feeling is going away.

When the most tedious, soul-numbing tasks can be dispatched in an evening — tasks that used to loom over you for weeks — the emotional landscape of the work changes completely. The procrastination is gone. The dread is gone. What's left are the genuinely interesting problems: the architecture decisions, the subtle bugs, the design questions that actually require human judgment.

It doesn't mean routine disappears forever. Eventually, the definition of "routine" will shift upward, and a new set of tasks will feel tedious. But that adjustment will take years, and in the meantime, we're living through a period where decades of accumulated "I really should fix this but I can't face it" are suddenly solvable. That's genuinely exciting.

For companies, the message is straightforward: these tools are a multiplier on your existing talent. Invest in them aggressively. The teams that master LLM-assisted development will outpace the ones that don't — not by a small margin, but by an order of magnitude on the tasks that matter most.

---

*Next in this series: [Why I Run Claude Code in Docker — And Why You Should Too](/coding-with-llms/coding-with-llms-docker-sandbox-setup)*
