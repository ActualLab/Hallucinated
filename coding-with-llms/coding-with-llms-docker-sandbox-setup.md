# Why I Run Claude Code in Docker — And Why You Should Too

*Part 2 of a series on coding with LLMs. Previously: [×10, Not +10: What LLMs Actually Do to Developer Productivity](/coding-with-llms/coding-with-llms-multiplier-not-addition).*

Here's a question that determines whether you'll get a 10× multiplier or a 2× multiplier from AI coding tools: can you let the LLM run without asking permission?

Every time Claude Code pauses to ask "Can I run this command?" or "Can I edit this file?", you lose momentum. Not just the seconds it takes to click "yes" — you lose the ability to walk away, to batch tasks, to run things overnight. The permission prompts turn an autonomous agent into a chatbot that needs babysitting.

The fix is obvious: give it full permissions. The problem is also obvious: giving an LLM unrestricted access to your machine is insane.

Docker solves both problems at once.

## The Permission Prompt Problem

I've used most of the available AI coding tools at this point — Cursor, Windsurf, Copilot, various ChatGPT integrations, and Claude Code. Across all the IDE-integrated options, the workflow is fundamentally the same: the LLM suggests changes, you approve them, it suggests more, you approve those. It's interactive by design.

This interaction model has a fatal flaw that becomes apparent the moment you try to use these tools on real-world tasks. A refactoring that touches 50 files means 50 approval prompts. A task that involves running tests, reading output, fixing code, and re-running means you're sitting there approving every shell command in the loop.

You can, in theory, give some of these tools broader permissions. But IDE integrations don't have a clean sandboxing story. When Cursor or Windsurf runs a command, it runs on your machine, with your permissions, against your actual files. There's no isolation boundary. One bad `rm` command, one mangled git operation, one PowerShell command interpreted as bash — and you're dealing with real damage.

Claude Code, running in a terminal, has one critical advantage: you can put the entire thing inside a Docker container.

## Why CLI Beats IDE

The reason CLI-based tools win here isn't about the interface. It's about the isolation boundary.

An IDE plugin runs inside your IDE, which runs on your host OS. There's no clean way to sandbox it. You can restrict which files it can edit, but you can't prevent it from running arbitrary shell commands with your user's permissions — or if you do, you're back to the permission prompt problem.

A CLI tool runs in a shell. Shells run in containers trivially. When Claude Code runs inside a Docker container, it gets:

- Full read/write access to your project files (mounted as volumes)
- Full permission to run any command it wants
- Zero ability to damage anything outside the container

This is the only setup I've found where you can genuinely tell the LLM "do whatever you need to do" and mean it. No permission prompts, no babysitting, no risk.

## The Windows Problem

If you're on Windows — and I am — there's an additional reason Docker isn't optional: it's required.

Claude Code (and in fact every LLM I've tested) defaults to Unix shell syntax. It will write `cat file.txt | grep pattern`, use forward slashes in paths, chain commands with `&&`, and use single quotes where PowerShell expects double quotes. On a good day, these commands fail with a clear error and the LLM retries with the correct syntax. On a bad day, the command gets partially parsed by PowerShell, does something entirely different from what the LLM intended, and you get subtle corruption that's much harder to diagnose than an outright failure.

Running inside a Linux Docker container eliminates this entirely. The LLM thinks it's on Linux — because it is. The commands work as expected. No syntax mismatch, no partial parsing, no surprises.

## The Setup

The first thing I did when I started using Claude Code seriously was ask it to generate a Dockerfile for itself. The irony isn't lost on me.

The setup is straightforward:

1. A Dockerfile based on a standard Linux image with Node.js (for Claude Code) and whatever build tools your project needs (.NET SDK in my case, but adapt as needed)
2. Volume mounts that map your project directories into the container
3. Claude Code installed inside the container, configured with your API key
4. A wrapper script to launch it

The key design decisions:

- **Mount multiple project directories** if you work across repos. I map several of our projects into the container so Claude Code can navigate between them as needed.
- **Don't mount your home directory.** Mount only what the LLM needs access to. This is defense in depth — even though the container already provides isolation, there's no reason to expose files the LLM doesn't need.
- **Use a PowerShell/bash wrapper** for ergonomics. I have a wrapper that lets me launch Claude Code in different modes — fresh session, resume previous session, specific project context. It handles the `docker run` invocation with the right volume mounts and environment variables.

The result: I type one command, Claude Code starts inside a fully isolated Linux environment with access to my projects, and I can tell it to do whatever it wants. No permission prompts. No risk. Full autonomy.

## Why Cursor, Windsurf, and Friends Are Overvalued

This is going to be a spicy take, but I'll say it directly: from the perspective of a developer who actually uses LLMs for production work, Cursor, Windsurf, Copilot, and the rest of the IDE-integrated AI tools are massively overvalued — both as companies and as products.

The reason is exactly what I've described above. The core value proposition of an AI coding tool is autonomy — the ability to hand it a task and let it work. IDE integrations fundamentally can't deliver this because they can't be sandboxed. You're either restricting permissions (and killing productivity) or running unrestricted on your host machine (and accepting real risk).

There's a second issue: IDE integrations tend to be focused on the code editing experience — autocomplete, inline suggestions, chat-about-this-function. These are useful features, but they're incremental. They make you type faster. They don't change what's feasible.

The tasks where LLMs deliver 10-30× improvements — the [large refactorings](/coding-with-llms/coding-with-llms-death-of-procrastination), the [overnight bug hunts](/coding-with-llms/coding-with-llms-bug-hunting-logs), the codebase-wide migrations — require the LLM to work autonomously for extended periods. To navigate files, run tests, read output, adjust, and iterate. That's an agent workflow, not an autocomplete workflow. And it only works when the agent has full permissions in a safe environment.

Maybe IDE-integrated tools will solve the sandboxing problem eventually. But right now, the only production-grade setup I've found is a CLI tool in a Docker container. Everything else is either too restricted to be useful or too dangerous to be safe.

## The Overnight Pattern

Once you have the Docker setup working, a pattern emerges that would be impossible with IDE-integrated tools: the overnight task.

Before going to bed, I launch Claude Code on 2-3 tasks — each in a separate container, each working on a different branch or project. I describe the task, provide any necessary context, and let it run.

In the morning, I review what happened. Sometimes everything worked perfectly. Sometimes it went sideways (and I'll have [more to say about that](/coding-with-llms/coding-with-llms-failures-and-limits)). But even when it fails, it usually produces useful partial results — identified root causes, narrowed down bugs, or attempted approaches that inform my next move.

This pattern only works because:
1. The Docker container means nothing can go wrong outside the sandbox
2. Full permissions mean the LLM doesn't block waiting for approval at 3 AM
3. The CLI interface means no IDE needs to be running

The compounding effect is significant. While I sleep, the LLM is exploring, testing, iterating. I wake up to results instead of starting from zero. Even at a conservative success rate, that's hours of productive work happening while I'm unconscious.

That's the real reason Docker isn't optional. It's not a nice-to-have for security-conscious developers. It's the enabler of the entire autonomous agent workflow that makes LLMs a multiplier instead of a slightly faster autocomplete.

---

*Next in this series: [The Death of Developer Procrastination](/coding-with-llms/coding-with-llms-death-of-procrastination)*
