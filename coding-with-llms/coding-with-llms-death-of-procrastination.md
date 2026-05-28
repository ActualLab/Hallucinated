# The Death of Developer Procrastination

*Part 3 of a series on coding with LLMs. Previously: [Why I Run Claude Code in Docker](/coding-with-llms/coding-with-llms-docker-sandbox-setup).*

Every codebase has a graveyard of tasks that everyone agrees should be done and nobody wants to do. Not because they're hard — because they're boring. The massive migration. The documentation overhaul. The refactoring that touches every module.

These tasks share a specific profile: low intellectual challenge, high mechanical effort, and catastrophic effort-to-excitement ratio. They're the worst possible assignment for a senior developer's brain — zero novelty, maximum tedium — which is why they get postponed indefinitely despite being genuinely important.

I call the accumulated cost of these postponed tasks the **procrastination tax**. And LLMs don't reduce this tax. They obliterate it.

## Three Tasks, Three Data Points

Let me walk through three real examples from the past few months. These aren't cherry-picked success stories — they're representative of a category of work that LLMs handle exceptionally well.

### Fusion Documentation: Months → 1 Week

[Fusion](https://github.com/ActualLab/Fusion) is an open-source library I maintain. It's been around for years. The documentation was perpetually "on the roadmap" — and perpetually not happening, because writing comprehensive documentation for a library with this many concepts is exactly the kind of task that makes a developer's eyes glaze over.

I handed it to Claude Code. In just over a week, I had a nearly complete documentation site. Was it perfect? No. There were sections that made my hair stand on end — the LLM occasionally misunderstood a concept or described a feature that didn't exist. But the errors were fixable in minutes each. The alternative — writing it all myself — would have taken at least two months, and honestly, it might never have happened at all, because I would have found reasons to postpone it indefinitely.

The multiplier here wasn't just about speed. It was about the task happening *at all*.

### Merging User Profile Types: Weeks → 3 Days

Our application had two separate types describing a user's profile. One covered the "profile" aspects — display name, avatar, preferences. The other covered the "system" aspects — authentication, sessions, login state. This split was a historical accident. They should have been one type from the beginning, and the divergence had been causing friction for years.

Merging them is conceptually simple but mechanically brutal. These types are referenced everywhere — different APIs, different services, different serialization paths. Every reference needs to be found, understood, and updated. Miss one, and something breaks at runtime in a way that's hard to trace back to the merge.

Claude Code handled the merge in three days. It found every reference, understood the semantic differences between the two types, and produced a clean unified type with all the necessary migration code. A human doing this by hand? Two weeks minimum. And the real cost isn't the two weeks — it's the two *years* the task sat on the backlog because nobody wanted to face those two weeks.

### MessagePack Annotations: 300+ Types

Adding support for a new serialization format (MessagePack) required annotating every serializable type in our codebase with the correct attributes. Over 300 types, each needing the right annotations for proper code generation.

This is the platonic ideal of a task that should never be done by a human. It requires perfect attention to detail across hundreds of repetitive edits, zero creativity, and any mistake leads to subtle serialization bugs that surface at runtime. It's a job that plays to every weakness of the human brain — we lose focus, we get bored, we skip steps, we make typos.

Claude Code found every relevant type, wrote tests first, then annotated them correctly and verified the tests passed. The whole thing was done in a single session.

## Why These Tasks Never Get Done

These three examples share something important: they were all tasks that *should* have been done long ago. The documentation had been needed for years. The profile merge had been on the wishlist for ages. The serialization migration was overdue.

So why weren't they done?

The obvious answer is "they're boring," but that's only part of it. The real blockers are structural:

**Merge conflict risk.** Large refactorings touch many files. In any team with more than a couple of developers, a change that modifies 50-100 files will conflict with almost everyone else's work in progress. This makes large refactorings nearly impossible to ship in an active codebase — by the time you finish, half your changes conflict with what others have merged in the meantime. The practical result: these tasks get pushed to "quiet periods" that never come.

**Effort-to-excitement ratio.** Senior developers — the ones with enough context to do these tasks correctly — are also the ones whose brains are most optimized for novelty and problem-solving. Asking a senior dev to spend two weeks on mechanical edits is like asking a Formula 1 driver to commute in traffic. They can do it; they'll hate every minute of it; and they'll find excuses not to.

**Review burden.** Even if someone does the work, the resulting pull request is enormous. Reviewing a 100-file refactoring is its own multi-day task. And the reviewer knows that any mistake — a missed reference, a broken invariant — won't show up in the diff but will show up in production. So reviews are slow, thorough, and draining.

LLMs don't just accelerate the typing. They compress the entire timeline — writing, testing, iterating — from weeks to days. And when a task that takes days of your time instead of weeks, the merge conflict window shrinks, the procrastination pressure evaporates, and the review becomes manageable because the changes are fresh and coherent rather than a stale patch rebased six times.

## The 20-30× Multiplier on Boring Work

In the [first article](/coding-with-llms/coding-with-llms-multiplier-not-addition), I talked about a general 5-10× multiplier on developer productivity. But for the specific category of tasks described here — routine, mechanical, large-scale — the real multiplier is 20-30×.

This isn't because the LLM types 30 times faster than a human. It's because of two compounding effects:

1. **The LLM doesn't procrastinate.** A human facing a boring task will spend hours warming up, checking email, reading HackerNews, "just quickly" fixing that other bug first. The actual time spent on the boring task might be 3 hours out of a 10-hour workday. The LLM just starts working.

2. **The LLM doesn't lose focus.** File 1 gets the same attention as file 297. Humans make more mistakes as tedium accumulates — skipping edge cases, using copy-paste sloppily, forgetting to update a test. The LLM maintains consistent quality across the entire task.

When you multiply the time saved by eliminating procrastination *and* the time saved by eliminating focus loss, you get multiplicative compression. A task that takes a human three weeks — but only because they're working at 20% efficiency due to boredom — becomes a task that takes the LLM a single evening.

## The Compound Effect

The real payoff isn't any single task. It's what happens when the procrastination tax drops to near zero across your entire codebase.

All those "we should really fix this" items on the backlog? They start getting fixed. Technical debt that's been accumulating for years starts getting paid down. The documentation that was always "coming soon" actually arrives. The serialization format you've been wanting to switch to is now supported.

Each of these improvements makes the codebase easier to work with, which makes *every subsequent task* faster — including the non-routine, genuinely hard tasks that LLMs can't handle as well. It's a virtuous cycle: the LLM handles the mechanical cleanup, which reduces friction for everyone, which makes the human-driven architectural work more productive.

If you're a team lead or engineering manager, this is probably the single most important thing to understand about LLMs: their biggest impact isn't on your hardest problems. It's on the long tail of important-but-boring work that's been slowly degrading your codebase for years.

---

*Next in this series: [Let It Read Your Logs: Bug Hunting with LLMs](/coding-with-llms/coding-with-llms-bug-hunting-logs)*
