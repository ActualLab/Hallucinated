# Let It Read Your Logs: Bug Hunting with LLMs

*Part 4 of a series on coding with LLMs. Previously: [The Death of Developer Procrastination](/coding-with-llms/coding-with-llms-death-of-procrastination).*

There's a class of bugs where you know something went wrong, you have the logs proving it, and you just need someone to sit down and trace through hundreds of lines of output to figure out exactly where the sequence broke. It's detective work — tedious, time-consuming, and entirely mechanical once you know what to look for.

LLMs are unreasonably good at this.

## The Distributed Locking Bug

Let me describe a specific bug that Claude Code found, because it illustrates the pattern well.

Our application uses a shard-based routing mechanism for remote calls. The design is roughly this: services that look local are actually distributed across a cluster of machines, with a guarantee that each shard is owned by exactly one machine at any given time. This means the state living on that machine is authoritative — no synchronization needed, which is what makes the whole thing fast.

The implementation involves several moving parts. Every machine registers in a registry of active nodes. Shards are divided using consistent hashing. On top of that, machines explicitly acquire distributed locks for their shards — belt and suspenders, because consistent hashing alone doesn't provide strong enough guarantees during cluster membership changes.

The tricky part: during transitions — when a node joins or leaves the cluster — there's a window where multiple machines might think they own the same shard. The consistent hash ring says machine A owns shard 7, but machine B still holds the distributed lock for shard 7 from before the ring updated. And machine C might see yet another view of the ring because its membership information is slightly stale. These race conditions need to be handled correctly, and we have stress tests that provoke them at high frequency.

One of these stress tests was failing intermittently. The failure was a consistency violation — two machines briefly believed they had exclusive ownership of the same shard, leading to conflicting state updates.

I pointed Claude Code at the test output — several hundred lines of log entries showing lock acquisitions, lock releases, shard map updates, and the eventual consistency violation. The log format wasn't anything special: timestamps, thread IDs, component names, and structured messages about lock state transitions.

Claude Code traced the entire sequence correctly. It identified which machine acquired the lock first, when the shard map update propagated to each node, at what point the second machine believed the shard was unowned (because its view of the map was stale), and exactly where the lock acquisition logic failed to check for the stale map condition. It found a subtle ordering bug in the code that handles the gap between "consistent hash says this shard is mine" and "distributed lock confirms this shard is mine."

I would have found this bug eventually. But "eventually" means hours of staring at log output, manually correlating timestamps across multiple nodes, and building a mental model of the state machine transitions. Claude Code did it in minutes, and its explanation was precise enough that I could verify the diagnosis and write the fix immediately.

## Why Logs Are a Force Multiplier

This wasn't a one-off. I have dozens of similar examples. The pattern is consistent: if the bug leaves a trace in your logs, and your logs are detailed enough to reconstruct what happened, an LLM can usually find the bug faster than you can.

This has a practical implication that I don't think enough teams have internalized: **your logging infrastructure just became a force multiplier for AI-assisted debugging.**

Before LLMs, the main audience for logs was humans — either reading them in real time during an incident or searching through them after the fact. The quality bar was "good enough for a human to understand." Structured logging was nice to have but often didn't justify the effort.

Now there's a second audience, and it's an audience that can process vastly more log data than a human, has infinite patience for correlating timestamps, and never loses track of which thread was doing what. The ROI calculation for investing in logging quality has changed fundamentally.

In practice, this means:

- **Log state transitions explicitly.** Don't just log "lock acquired" — log which lock, by which node, with what view of the shard map at the time of acquisition. The more context in each log line, the less the LLM needs to infer.
- **Log at the boundaries.** Entry and exit of key operations, with the relevant parameters. If a request comes in, log its ID. If a lock attempt starts, log what it's trying to lock and why.
- **Make sure tests emit logs.** This seems obvious, but many test frameworks suppress log output by default. When your test output includes the full log trace, you can hand the entire thing to an LLM and say "find the bug."

## Stress Tests as Bug Reproduction Engines

The distributed locking bug wasn't found in production. It was found by a stress test that deliberately provokes cluster membership changes at high frequency — events that happen rarely in production but, when they do, need to be handled correctly.

We have a battery of these stress tests. They take scenarios that occur infrequently under normal operation and run them at compressed timescales — thousands of shard transitions in minutes instead of a handful per day. When something goes wrong, the test catches it, and the log output provides the full trace of what happened.

This combination — stress tests that provoke rare conditions, plus detailed logging, plus an LLM that can parse the output — is extremely powerful. The workflow is:

1. A stress test fails (or you suspect a production bug and write a new stress test to reproduce it)
2. You hand the test output to Claude Code
3. Claude Code reads the logs, identifies the failure sequence, and often diagnoses the root cause
4. You verify the diagnosis and write the fix

Step 3 is where humans traditionally spend most of their time, and it's exactly the step that LLMs can compress from hours to minutes.

You can take this further. If you have a production bug that isn't covered by existing stress tests, you can ask Claude Code to *write* a stress test that reproduces it. Describe the scenario — "users are seeing stale data after a shard migration" — and the LLM can write a test that simulates rapid shard migrations with concurrent reads. Once the test reliably fails, you're back in the workflow above.

This also works with UI-level bugs. Claude Code can drive a browser via Playwright, so "reproduce this user-reported issue" can mean "write a Playwright script that performs the user's actions repeatedly under load and check for the reported inconsistency." The LLM handles the mechanical work of writing the reproduction script, and the test framework handles running it thousands of times until the race condition surfaces.

## The Overnight Debug Pattern

In the [Docker sandbox article](/coding-with-llms/coding-with-llms-docker-sandbox-setup), I mentioned the overnight task pattern. Bug hunting is where this pattern pays off the most.

The workflow: before going to bed, I describe the bug to Claude Code. I point it at the relevant logs, the failing test, or both. I describe what the expected behavior should be and let it work.

In the morning, one of three things has happened:

1. **It found the bug and wrote the fix.** This happens more often than you'd expect for bugs that are primarily about tracing through complex execution sequences. I review the fix, verify it makes sense, run the tests, and merge.

2. **It found the bug but the fix is wrong or incomplete.** The diagnosis is still valuable — it narrows the search space from "something is wrong somewhere in this subsystem" to "this specific ordering assumption is violated in this specific code path." I finish the job in minutes instead of hours.

3. **It went down the wrong path entirely.** Even here, the attempt is informative. The approaches it tried and the reasons they didn't work tell me something about where the bug *isn't*, which is still useful information. And it cost me nothing — I was asleep.

The expected value of this workflow is strongly positive. Even a 30% success rate on fully solving the bug means I wake up to solved problems one morning out of three. And the partial successes on the other mornings still save significant time.

## What Good Logging Looks Like Now

To make this work, you need to rethink logging slightly. Not radically — most of the practices that make logs good for LLMs also make them good for humans. But the emphasis shifts:

**Structured messages > free-form strings.** Instead of `"Lock operation completed successfully"`, log `"Lock acquired: shard=7, node=A, ring_version=42, lock_id=xyz"`. The LLM can parse both, but the structured version lets it correlate events much more precisely.

**Causality chains.** If operation B happened because of event A, log that relationship. "Shard map updated (trigger: node C joined at ring_version=43)" gives the LLM explicit causal links instead of forcing it to infer them from timestamps.

**Test log output should be verbose.** In production, you might throttle logging for performance. In tests — especially stress tests — turn everything on. The LLM can handle thousands of log lines. You can't, but that's the point.

The developers who invested in good logging infrastructure years ago are now reaping an unexpected dividend. Their logs aren't just useful for incident response — they're training data for an AI debugger that works while they sleep.

---

*Next in this series: [The Elaborate Hallucination: Where LLMs Fail Spectacularly](/coding-with-llms/coding-with-llms-failures-and-limits)*
