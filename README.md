# Solution Research Dev

An Agent Skill for Claude that hands you the tools instead of the answer.

It takes a bug, an error or a feature you don't know how to build, researches the
current best practices, and hands you back **2-4 labelled options (A, B, C)** with what
each one takes to implement, why it makes sense architecturally, and where it hurts.
It does not tell you which one to pick. When you pick one, it walks you through the
implementation step by step — dependencies, classes, responsibilities, method
signatures, common mistakes, how to verify it works — and the code stays yours to write,
unless you explicitly ask it to write it.

**English only.** Earlier versions shipped an Italian copy alongside; the skill now
answers in whatever language you write in, so a second copy was just two files to keep
in sync. Write to it in Italian and it replies in Italian.

## What this is for

The point is not to be a pair of hands typing what the AI dictates. The point is to
understand how to build the thing. The workflow the skill is designed around:

1. **Do your own homework first.** Read up, work out what you actually need to build and
   what the hard parts are. Come in with a problem you understand, not one you've
   outsourced.
2. **Ask the skill what the possible approaches are.** This is what it's good at: laying
   out the real alternatives with their trade-offs, including the ones you didn't know
   existed.
3. **Make the architectural call yourself.** The choice depends on your project, your
   constraints, your team — things you know and the model doesn't. Don't delegate it.
   The skill deliberately won't make it for you.
4. **Then decide who writes the code.** By hand if you want to learn the thing down to
   the details. By the AI if you're short on time, or you're senior enough that writing
   it teaches you nothing — in which case you ask, and it writes it (Phase 3).

Step 4 is a real choice, not a failure. What's not a choice is skipping steps 1 and 3.

## What changed in this version

The skill used to have two phases and a hard line against ever writing code. It now has
three:

- **Phase 3 exists.** Ask explicitly — "write it for me", "I'm in a rush, give me the
  finished code" — and it writes complete, working code for the option you chose, with
  the key points commented and the common mistakes still flagged. No lecture, no
  push-back ritual: you asked for it knowingly.
- **Rule 2 is now a default, not an absolute.** "Don't write the implementation code" is
  how it behaves in Phases 1 and 2; Phase 3 is a separate mode you turn on, not an
  exception you have to argue for.
- **Rule 1 did not move.** Asking for the code is fine. Asking it to make the
  architectural choice for you is still refused — briefly, without moralising, and it
  compensates with a sharper read on which trade-off dominates in your case. You still
  decide.
- **The framing is explicit about tools.** The job is to name the APIs, methods, classes,
  annotations, security patterns and optimization techniques you may not know exist in
  your stack — a kind of documentation cut to your specific problem.

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

This skill is Case C and Case D, end to end: Phases 1 and 2 are the part they share,
Phase 3 is where C splits off. That shared part is the one I kept doing badly by hand:
I'd ask "how do I do X?", get one confident answer, implement it, and never find out
that there were three other ways and that mine was the wrong one for my constraints.
Comparing real alternatives is where the learning is — that's the part I wanted turned
into a repeatable process instead of a good day.

## The rules

Everything in the skill exists to enforce three constraints:

1. **It doesn't choose for you.** It will tell you "this is worth it if the project
   scales, this is a bad fit for a small team on a deadline" — context-dependent
   trade-offs, all of them. The decision stays yours, because the point is to build the
   judgement, not to borrow it. This one doesn't bend, even under time pressure.
2. **By default it doesn't write the implementation.** It names the dependency, the
   class, the responsibility, the method signature. You write the body. Naming an
   annotation to explain a concept is fine; a ready-to-paste block is not.
3. **Unless you ask.** Explicitly, in so many words — then it writes the code, commented
   where a relevant tool comes into play, and still tells you where it could break.

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

**Phase 3 — The code, if you ask for it.** Only on an explicit request. It writes the
full implementation for the option you chose, reusing the exact tools it named in
Phase 2 so the code matches the explanation, with comments on the parts worth
understanding. If you skip straight here it picks the most sensible option itself and
says which, so you know what the code is based on.

All three phases live in the same conversation — no re-invoking needed.

## Installation

**Claude Code** — personal (all projects):

```bash
mkdir -p ~/.claude/skills/solution-research-dev
cp SKILL.md ~/.claude/skills/solution-research-dev/SKILL.md
```

Or project-scoped, committed with the repo:

```bash
mkdir -p .claude/skills/solution-research-dev
cp SKILL.md .claude/skills/solution-research-dev/SKILL.md
```

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

Then read the options, pick a letter, and either write the code or say you want it
written.

## What it is not

- Not for syntax questions with one right answer ("how do I reverse a list in Python").
  There's nothing to compare; just ask normally.
- Not a code generator you reach for first. Phase 3 exists, but it's the last step of a
  process, not a shortcut past it. If all you want is code, that's Case A, and a plain
  detailed prompt serves you better.
- Not a substitute for reading the documentation. It's a way of finding out *which*
  documentation is worth your afternoon.

## Contributing

Issues and PRs welcome, particularly: sharper option formats, and cases where the skill
triggers when it shouldn't (or doesn't when it should).

## License

MIT.
