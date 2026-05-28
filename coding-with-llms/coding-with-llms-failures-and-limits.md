# The Elaborate Hallucination: Where LLMs Fail Spectacularly

*Part 5 of a series on coding with LLMs. Previously: [Let It Read Your Logs: Bug Hunting with LLMs](/coding-with-llms/coding-with-llms-bug-hunting-logs).*

I need to balance the optimism of the previous articles with a clear-eyed look at where these tools fall apart. And they fall apart badly — not gracefully, not with helpful error messages, but with the confident delivery of code that looks right, compiles, passes its own tests, and is fundamentally broken.

## The Fusion TypeScript Port

This is the story I promised in the [first article](/coding-with-llms/coding-with-llms-multiplier-not-addition). It's the most instructive failure I've had with LLMs, because the failure wasn't obvious at all — for most of a day, it looked like a stunning success.

I wanted to port [Fusion](https://github.com/ActualLab/Fusion) — my open-source .NET library for real-time applications — to TypeScript. This isn't a trivial task. Fusion's core involves computed values with automatic dependency tracking, invalidation propagation, and a caching layer that all need to work together precisely. I spent about an hour dictating a detailed specification to ChatGPT, then distilled it into structured summaries, and fed everything to Claude Code with a comprehensive plan.

It worked overnight. In the morning, I opened the project and — I'm not exaggerating — I was amazed. 70-80 files. Core abstractions implemented. Dependency tracking, computed values, invalidation. Tests written and passing. My first reaction was that this had actually worked. That a task I expected to take weeks had been done in a night.

By lunch, the amazement was fading. As I reviewed more carefully, I started finding issues. Not cosmetic issues — structural ones. The dependency tracking wasn't actually tracking anything. It was going through the motions of tracking — calling the right methods, maintaining data structures that looked like dependency graphs — but the actual propagation logic was wrong in ways that would make it useless under real conditions.

By evening, I was looking at what I can only describe as an elaborate hallucination. The code was a *simulation* of Fusion, not a *port* of Fusion. It had the right shape — the right file structure, the right class names, the right method signatures — but the internal logic was broken in ways that no human developer would produce. Not broken in a "off by one" or "missed an edge case" way. Broken in a "this fundamentally doesn't understand what it's implementing" way.

The worst part? The tests passed. Because the tests were hallucinated too. They tested the code's actual (wrong) behavior, not the specified (correct) behavior. The LLM had generated a self-consistent but wrong system — code that does something coherent, just not what it was supposed to do, and tests that prove the wrong behavior works correctly.

This is the kind of error that even a beginner programmer wouldn't make. A beginner would get the implementation wrong but would at least try to test against the specification. The LLM generated the implementation and the tests as a unit, which meant the tests were precisely calibrated to the implementation's bugs.

## The Context Window Wall

Why does this happen? The answer is in the math of context windows.

Current models have a stated context window of around 200,000 tokens. That sounds like a lot until you account for everything else that has to fit in there:

- **System prompt and tool definitions:** Several thousand tokens before your conversation even starts
- **Tool call overhead:** Every file read, command execution, and response takes tokens
- **The LLM's own reasoning:** On complex tasks, the model's internal chain-of-thought can easily consume tens of thousands of tokens
- **Generated code:** The new code being written takes tokens too

What's actually left for your source code? Roughly 100,000 tokens in the best case. That translates to about 20-40 files of typical source code. For a library like Fusion — which has around a thousand files — the model can only see a small fraction at any time.

Other models claim larger windows. GPT models have wider context. Gemini claims even more. But in practice, larger context windows don't help proportionally. These models tend to pay less attention to information that's far from the current focus — the "lost in the middle" problem. A 1-million-token context window where the model effectively ignores 80% of it isn't much better than a 200K window where it uses most of it.

And for a porting task, the math gets worse. You need the *source* code in context (to understand what to port) and the *target* code in context (to maintain consistency in what you're generating). That immediately halves your effective capacity. Add in test code, build output, error messages — and you're down to maybe 10-20 files of useful source context for a task that requires understanding hundreds.

## The Global Invariant Problem

The context window limitation manifests as a specific failure mode: inability to maintain global invariants.

A global invariant is a property that must hold true across your entire codebase — not just in one file, but everywhere. Examples:

- "Every serializable type must have a MessagePack annotation" (this is the kind of task LLMs handle *well* — it's mechanical and verifiable)
- "Dependency invalidation must propagate transitively through the entire computed value graph" (this is the kind of invariant LLMs *can't* maintain — it requires understanding how 20 different components interact)

When an LLM writes a large feature, it's essentially working file-by-file, keeping a lossy summary of the broader context. It can maintain local invariants perfectly — things that need to be true within a single file or a small cluster of files. But invariants that span the whole system gradually decay as the LLM's attention moves to different parts of the code.

In the Fusion port, this manifested as each component being internally consistent but incompatible with the others at the integration layer. The computed value implementation was fine in isolation. The dependency tracker was fine in isolation. But they didn't actually work together correctly, because the integration contract between them required understanding both components simultaneously — and the model couldn't hold both in focus at the same time.

## The Voice Message Edit: A Smaller Example

Not every failure is dramatic. Here's a smaller one that illustrates how LLMs miss things that are obvious in hindsight.

Our app has voice messages that are transcribed to text. I asked Claude Code to make these transcribed messages editable. The requirements were nuanced: if the user makes small edits (fixing typos in the transcription), keep it as a voice message with updated text. If the edits are substantial (basically a rewrite), convert it to a regular text message.

Claude Code implemented this. It worked — for the basic case. But it completely missed that editing might introduce markup (bold, italic, links). A voice message with markup doesn't make sense — it should automatically convert to a regular text message. This was visible in the existing code that handles message rendering. The information was *right there*, in files that Claude Code had read, and it still missed the implication.

This is a pattern I see repeatedly. The LLM can follow a specific chain of logic — "user edits message → check edit distance → if substantial, convert" — but it doesn't organically notice tangential implications. A human developer, while implementing the edit feature, would naturally think "wait, what about markup?" because humans maintain ambient awareness of the broader feature set. LLMs don't have ambient awareness. They have focused attention on the current task, and anything outside that focus is invisible.

## When LLMs Work vs. When They Don't

After months of using these tools, I have a reasonable heuristic for predicting success:

**LLMs work well when:**
- The task is primarily mechanical with clear, verifiable constraints
- The required context fits in 20-40 files
- Success can be validated by running tests
- The task doesn't require understanding global invariants across the codebase
- You can describe the expected output precisely

**LLMs fail when:**
- The task requires maintaining coherence across a large codebase
- Multiple new components need to integrate at a deep architectural level
- The "correct" solution depends on understanding subtle domain-specific contracts
- The task is iterative — each step depends on understanding all previous steps, and the cumulative context exceeds what fits in the window
- You would struggle to fully specify the expected behavior even in a detailed document

The dividing line roughly maps to: can you decompose this into subtasks that each fit within the context window, where the interfaces between subtasks are explicit and well-defined? If yes, the LLM can probably handle each subtask well. If the task is inherently holistic — where correctness requires simultaneously considering more information than fits in the context window — it will fail.

For practical purposes: **routine refactorings, migrations, annotation tasks, bug hunting, documentation — yes.** Large new features, architectural redesigns, porting complex libraries — not yet.

The "not yet" is important. Context windows are growing. Models are getting better at attending to long contexts. But we're probably years away from a model that can genuinely reason about a 1,000-file codebase as a coherent whole. Until then, knowing where the cliff edge is — and stopping before you fall off it — is one of the most important skills for LLM-assisted development.

---

*Next in this series: [What LLMs Mean for Developer Careers](/coding-with-llms/coding-with-llms-developer-careers-future)*
