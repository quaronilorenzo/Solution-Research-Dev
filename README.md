# Solution Research Dev

An Agent Skill for Claude that refuses to write your code.

It takes a bug, an error or a feature you don't know how to build, researches the
current best practices, and hands you back **2-4 labelled options (A, B, C)** with what
each one takes to implement, why it makes sense architecturally, and where it hurts.
It does not tell you which one to pick. When you pick one, it walks you through the
implementation step by step — dependencies, classes, responsibilities, method
signatures, common mistakes, how to verify it works — but the code stays yours to write.

Available in [English](skills/solution-research-dev/SKILL.md) and
[Italian](skills/ricerca-soluzioni-dev/SKILL.md).

## Why I built this

I'm a junior developer, and I started my career at a moment where an AI can produce, in
thirty seconds, code that would take me a day. The obvious move is to let it. The
problem with the obvious move is that after six months of it you have shipped a lot and
learned almost nothing: you can't tell a good architectural decision from a bad one,
because you have never made one.

So I stopped asking "can the AI do this for me?" and started asking "what is this task
worth to me?". That question has four answers, and only two of them involve this skill.

**Case A — I know how to do it, but I don't feel like writing it.**
Usually frontend, usually mechanical. I describe the task to the AI in as much detail as
I can and let it write the code. Nothing to learn here; the detailed spec *is* my
contribution.

**Case B — I know how to do it, and it's interesting.**
I write it by hand. If it's the part of the job I actually want to get good at, handing
it over is a bad trade.

**Case C — I don't know how to do it, and I'm short on time (or pairing with someone).**
Break the problem into smaller parts. Give the first part to Claude and make it lay out
the possible architectural solutions, with the reasoning behind each one and where it
would lean. I choose the path. *Then* it writes the code, and I do a focused code review
of that part only — not of everything.

**Case D — I don't know how to do it, and I have time to work on it properly.**
Same start as Case C: break it down, get the options, choose. But then I do the deep
dive and write the code myself.

This skill is the formalisation of the first half of C and all of D. That first half is
the part I kept doing badly by hand: I'd ask "how do I do X?", get one confident answer,
implement it, and never find out that there were three other ways and that mine was the
wrong one for my constraints. Comparing real alternatives is where the learning is —
that's the part I wanted turned into a repeatable process instead of a good day.

In Case C, you use Phase 1 and then explicitly ask for the code. The skill will push
back; tell it you're in a hurry and taking the trade-off knowingly, and review what it
produces. In Case D, you use the whole thing as it is.

## The two rules

Everything in the skill exists to enforce two constraints:

1. **It doesn't choose for you.** It will tell you "this is worth it if the project
   scales, this is a bad fit for a small team on a deadline" — context-dependent
   trade-offs, all of them. The decision stays yours, because the point is to build the
   judgement, not to borrow it.
2. **It doesn't write the implementation.** It names the dependency, the class, the
   responsibility, the method signature. You write the body. Naming an annotation to
   explain a concept is fine; a ready-to-paste block is not.

It also holds the line when you push. If you say "just tell me which one" or "just write
it", it acknowledges the time pressure, explains in one sentence why the choice and the
writing stay yours (you're the one who will have to maintain that code), and then
compensates by getting more granular — steps almost at pseudocode specificity, or a
sharper read on which trade-off dominates in your case. That's a more useful compromise
than dropping the reason the skill exists.

## How it works

**Phase 1 — Research and options.** Checks it has the technical context (language,
framework, version) and asks if it doesn't. If it's a bug, it finds the probable cause
*first*, instead of jumping to design patterns for a problem nobody has diagnosed yet.
Then it searches the web for current best practices — library versions and security
recommendations age badly — and presents the options in a fixed format so they're
comparable at a glance:

- **What it does** — 2-3 sentences
- **What it takes to implement it** — dependencies, classes, methods, configuration
- **Why it makes sense architecturally** — the mechanism, not just "it's faster"
- **Security** — only when there's a real trade-off
- **Other factors** — performance, learning curve, maintainability, community
- **Cons / when it is NOT worth it**

If there's genuinely only one standard approach, it says so instead of inventing filler
alternatives. If the question is simple, it answers simply instead of forcing the format.

**Phase 2 — Deep dive.** Triggered when you name your choice ("B", "let's go with the
second one"). It confirms what that solution does, then gives you ordered implementation
steps, the mistakes juniors typically make with *that specific* solution, and how to
verify it works. It knows the difference between "does B handle high traffic?" (still
Phase 1, you're still deciding) and "I'll go with B" (Phase 2).

Both phases live in the same conversation — no re-invoking needed.

## Installation

**Claude Code** — personal (all projects):

```bash
mkdir -p ~/.claude/skills
cp -r skills/solution-research-dev ~/.claude/skills/solution-research-dev
```

Or project-scoped, committed with the repo:

```bash
mkdir -p .claude/skills
cp -r skills/solution-research-dev .claude/skills/solution-research-dev
```

For the Italian version, use `skills/ricerca-soluzioni-dev` instead. Don't install both:
the two descriptions overlap and either one may trigger. Each version answers in the
language you write in, so pick one.

**Claude apps** — upload the skill folder in Settings → Capabilities → Skills, where
skill creation is enabled for your account.

## Usage

You don't invoke it. Just describe the problem the way you normally would:

> I need to handle file uploads in my Spring Boot app, files can be up to 2GB — what's
> the best way to do this?

> I'm getting `LazyInitializationException` when I serialize this entity and I don't
> understand where it comes from.

> I have to cache the results of an expensive API call in a Next.js app. What are my
> options?

Then read the options, pick a letter, and write the code.

## What it is not

- Not for syntax questions with one right answer ("how do I reverse a list in Python").
  There's nothing to compare; just ask normally.
- Not a code generator. If you want code, this skill is in your way — that's Case A, and
  a plain detailed prompt serves you better.
- Not a substitute for reading the documentation. It's a way of finding out *which*
  documentation is worth your afternoon.

## Contributing

Issues and PRs welcome, particularly: translations into other languages, sharper option
formats, and cases where the skill triggers when it shouldn't (or doesn't when it
should). If you change the behaviour, change both language versions so they stay in
sync.

## License

MIT.
