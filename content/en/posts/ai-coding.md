+++
title = "AI Coding"
date = 2026-06-04T00:20:32Z
draft = false

#summary = "How AI tools, agents, and engineering discipline are reshaping software development — and what it means for engineers."
tags = ["AI", "coding", "agentic", "engineering"]
categories = ["engineering"]
+++

## From Text Editors to AI Pair Programmers

I still remember my first job writing C code with a very simple text editor back to my college time. Then I moved to vim. Later, I switched to source insight. Then VSCode, with plugins for linting, formatting, and debugging, became my daily driver.

Today I am using a powerful AI coding assistant inside my IDE, which can suggest entire functions, explain codebases, write tests, and even refactor across files. This evolution isn't just incremental — it is structural. The unit of "programming" has changed. I even just use "TUI" (kiro-cli/opencode) to do coding with AI coding assistant. I spend less time on typing code, instead of planning and reviewing.

## Agentic Coding: When AI Can Act Autonomously

**Agentic coding** means AI agents that take initiative. They don't just suggest the next token — they plan, execute, review, and iterate.

- Separate the code to different files autonomously
- Write tests for edge cases
- Refactor existing code based on your requirements
- Debug by reading logs and proposing fixes

The key shift is from *autocomplete* to *autonomy*. The engineer is now a reviewer and architect, not a typist.

## AI Harness: Giving AI the Right Context

"Harness" means control. AI is powerful, but it needs structure to be useful. Harnessing AI well means:

- **Context boundaries** — giving the AI only the relevant files and docs, not the entire repo
- **Structured prompts** — not vague requests, but clear contracts of what you want
- **Tools access** - use different tools to effectively extract the related context
- **Human-in-the-loop** — the engineer always signs off on the AI's work
- **Verification steps** — tests, linting, review gates

Simplfy it, AI Agent = Model + Harness.

This is less flashy than agentic coding, and I think it is actually more important. A powerful agent with bad context is worse than a dumb autocomplete.

## Engineering Habits That Still Matter (and New Ones That Don't)

Some classic engineering practices are timeless, but others need rethinking in the AI era.

- **Readability still wins** — AI-generated code should be readable by humans, not just by other AI
- **Testing becomes even more critical** — AI can hallucinate bugs. Strong tests are your safety net
- **Code reviews change** — reviews are now about intent and architecture, not syntax
- **Onboarding accelerates** — AI can explain unfamiliar codebases in minutes
- **But understanding still matters** — if you don't understand what the AI wrote, you can't maintain it

## What I Think Engineers Should Focus On

If I have to summarize, this is where I would spend my effort:

- **Prompt engineering as a skill** — writing good instructions for AI is now a core engineering competency
- **Agent skills** - here skills are resuable, procedural instructions that teach AI agents how to execute complex, multi-step tasks independently
- **Context design** — structuring repos and docs so AI can actually help
- **Judgment** — knowing when to trust AI, when to ignore it, and when to discard and rewrite
- **How to verify** - verification is critical important ever, always consider how to test on what AI have done.
- **Systems thinking** — as AI changes the granularity of work, understanding the bigger picture becomes even more valuable

The engineers who thrive won't be those who reject AI, nor those who let AI do everything. It will be those who know how to *harness* AI — give it the right constraints, the right context, and the right oversight.

---

*The tools will keep changing. The core skill is still clear thinking.*
