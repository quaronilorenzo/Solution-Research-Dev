# Solution Research Dev

An Agent Skill for Claude that hands you the tools instead of the answer, and keeps a
written record of what you decided.

You bring a bug, an error or a feature you don't know how to build. The skill:

1. **Researches and compares.** It finds the cause if it's a bug, checks current best
   practices, and gives you **2-4 labelled options (A, B, C)**: what each one takes, why
   it makes sense architecturally, and where it hurts. If you have an idea of your own,
   it judges that honestly too. It does not pick for you.
2. **Explains the one you pick and writes it down.** It saves your decisions to a README
   for that feature inside your project, then walks you through the implementation step
   by step, without writing it. The code stays yours.
3. **Writes the code only if you ask.** It then turns that README into a workflow of the
   finished code: what happens at each step, why, a link to the exact lines, and what to
   expect at deploy and startup. You open the code when you hit a bug, not to understand
   it.

The skill file is written in Italian, but it answers in whatever language you write in,
feature READMEs included.

## What this is for

The point is not to be a pair of hands typing what the AI dictates. The point is to
understand how to build the thing. The workflow the skill is designed around:

1. **Do your own homework first, and write it down.** Read up, work out what you need to
   build and what the hard parts are. Come in with a problem you understand, not one
   you've outsourced. See [Keep a notepad](#keep-a-notepad).
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

## Keep a notepad

The best way to use this skill is to think before you ask, in a plain notepad. Any text
file will do. Mine has three headings: the steps as I imagine them (*Step by step*), the
choices I made on purpose (*Scelte che ho voluto fare*), and what I don't know (*Cose che
non so*).

```text
Step by step (memo):

Scelte che ho voluto fare:

Cose che non so:
```

Real notes from when I had to build a login, before asking anything:

```text
Step by step :
1) Prendere la request per loggarsi -> fare checks da controller come se è un email e un regex per la password valida
2) Portarla a login service, nuova classe
3) Prendere l'email, se non esiste nel db scartarla direttamente
4) Cryptare la password con BCrypt (come faccio in userservice) -> confrontarla con quella del db

Scelte che ho voluto fare:
-1 la request deve essere email e password e basta
-2 controllare prima dal controller -> service nuovo (separare la login dal resto del utente mi sembrava sensato, dimmi tu)
-3 cercare prima l'email se esiste
-4 e poi cryptare con lo stesso metodo con cui registriamo e checkare se è giusto

Cose che non so:
1) Ok dopo che ho checkato che l'utente è lui cosa posso fare? come faccio a darli un jwt ?
e non so come fare da qui
2) se non è lui tiro anche un exception? c'è un exception che crea…
```

Then paste the notes into your message, or just use them to ask a sharper question. The
skill doesn't read your notepad by itself. Each heading does a different job:

- **Step by step** shows how far you got on your own. Even if you never send it, you've
  already found where your understanding stops.
- **Scelte (your choices)** are what the skill judges. It treats them as hypotheses:
  it builds its options as if you hadn't mentioned them, so you don't get your own idea
  back dressed up as a comparison, and then opens its answer with a verdict on them. If
  something is a hard fact about the project ("we're on Java 17", "no new dependencies"),
  say it's a constraint, because anything unclear gets treated as an idea.
- **Cose che non so (what you don't know)** are the gaps to research. In the notes above,
  that's how to issue a JWT and what to do on a failed login: the actual subject of the
  options.

Being wrong in the notes is useful. Step 4 above has a real mistake: you can't hash the
incoming password again and compare the two results, because BCrypt salts every hash
with a random value, so the same password gives a different hash each time. You call the
library's match function, which reads the salt out of the stored hash. Write down what
you believe, and you find out exactly which belief was wrong. That sticks better than a
correct answer you never had to question.

## What changed in this version

- **Your idea gets a verdict.** Mentioning how you'd solve it no longer steers the
  research. The options are built independently, then a short *On your idea* block opens
  the answer: makes sense, partly makes sense, or doesn't hold, naming the mechanism. If
  it holds, it names the closest option and how they differ. It won't hunt for the "least
  wrong" option out of politeness, and the closest option isn't moved to the top.
- **Decisions are saved to a file.** Picking an option now creates
  `.claude/features/<feature-name>/README.md` in your project, so long conversations
  don't lose details and a later conversation can pick the feature up where it stopped.
- **You can correct it in writing.** That README has a *Developer notes* section that's
  yours. The skill re-reads the file before writing code, your notes take precedence over
  what it said earlier, and it never rewrites them. If a note clashes with the chosen
  option, it tells you instead of quietly following or ignoring it.
- **Phase 3 now explains itself without making you read the code.** Code goes directly
  into your files. Afterwards the README gets a step-by-step workflow with links to the
  exact lines, plus startup and deploy notes and a symptom-to-location troubleshooting
  list.
- **It reports what it actually verified.** If it could build or run the tests, you get
  the real result. If it couldn't, it says so: code nobody has run isn't presented as
  working.

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

Everything in the skill exists to enforce four constraints:

1. **It doesn't choose for you.** It will tell you "this is worth it if the project
   scales, this is a bad fit for a small team on a deadline" — context-dependent
   trade-offs, all of them. The decision stays yours, because the point is to build the
   judgement, not to borrow it. This one doesn't bend under time pressure: it will give
   you a sharper read on which trade-off matters most in your case, and you still decide.
2. **By default it doesn't write the implementation.** It names the dependency, the
   class, the responsibility, the method signature. You write the body. Naming an
   annotation to explain a concept is fine; a ready-to-paste block is not.
3. **Unless you ask.** Explicitly, in so many words — "write it for me", not a generally
   impatient tone. Then it writes the code without a lecture, because you asked for it
   knowingly.
4. **Your idea is a hypothesis, not a direction.** Constraints narrow the options; ideas
   and suspicions don't. "It's the cache's fault" gets checked, and if it's wrong you
   hear so, with the evidence.

## How it works

**Phase 1 — Research and options.** Checks it has the technical context (language,
framework, version) and asks if it doesn't. If the feature already has a README in
`.claude/features/`, it reads that first so it doesn't contradict decisions already made.
If it's a bug, it finds the probable cause *first*, instead of jumping to design patterns
for a problem nobody has diagnosed yet. Then it searches the web for current best
practices — library versions and security recommendations age badly — and presents the
options in a fixed format so they're comparable at a glance:

- **What it does** — 2-3 sentences
- **What it takes to implement it** — dependencies, classes, methods, configuration
- **Why it makes sense architecturally** — the mechanism, not just "it's faster"
- **Security** — only when there's a real trade-off
- **Other factors** — performance, learning curve, maintainability, community
- **Cons / when it is NOT worth it**

If you shared an idea, the verdict on it comes before all of this. If there's genuinely
only one standard approach, it says so instead of inventing filler alternatives. If the
question is simple, it answers simply instead of forcing the format.

**Phase 2 — Deep dive.** Triggered when you name your choice ("B", "let's go with the
second one"). It confirms what that solution does, saves the feature README, then gives
you ordered implementation steps, the mistakes juniors typically make with *that
specific* solution, and how to verify it works. It knows the difference between "does B
handle high traffic?" (still Phase 1, you're still deciding) and "I'll go with B"
(Phase 2). If a serious problem surfaces at this point, like a deprecated library, it
tells you before going on. If you come back with your own code for a check, it corrects
and explains without rewriting whole sections.

**Phase 3 — The code, if you ask for it.** Only on an explicit request. It re-reads the
feature README, your notes included, writes the full implementation into your project
files using the tools it named in Phase 2, comments the key points, and builds or runs
tests if it can. Last, it writes the workflow into the README and closes with a short
summary: files touched, the workflow in one line, and the two or three things you need
before starting the app. If you skip straight here it picks the most sensible option
itself and says which, so you know what the code is based on. Without access to your
project, the code and the README come back in the reply instead.

All three phases live in the same conversation — no re-invoking needed. To resume later,
the feature README carries the decisions across conversations.

## The feature README

One file per feature, at `.claude/features/<feature-name>/README.md`, in plain language
and with no code in it. It's written in two stages:

| | After Phase 2 (300-500 words) | Added after Phase 3 (1200 words at most) |
|---|---|---|
| **Decisions** | Problem, chosen option and stack, architectural idea, planned classes | Classes updated if the code changed the plan |
| **Workflow** | One line with the planned steps | Each step explained and linked to its lines |
| **Running it** | — | *Startup and deploy*, *If something goes wrong* |
| **Developer notes** | Yours from the start | Still yours, never touched |

After Phase 3, the workflow reads like this. The one-line chain at the top includes what
happens before and after the main action, but only if the code really does it:

```markdown
1. Receives the request → 2. Checks the format → 3. Looks up the user → 4. Verifies the password → 5. Answers

### 4. Verifies the password
**What happens:** compares the password sent with the hash saved at registration.
**Why this way:** BCrypt salts every hash, so two hashes can't be compared directly.
**Code:** [`LoginService.login`](../../../src/main/java/com/acme/LoginService.java#L42-L58)
```

Line numbers are taken from the finished files, never from memory, and rechecked whenever
the skill changes that code. If one has still drifted, search for the method by name.

The rest of the file covers what a person who won't read the code can't discover alone:

- **Startup and deploy:** dependencies and properties to set, what starts by itself
  (jobs, migrations), what gets written in production, how it behaves with several
  instances, what was and wasn't verified, and what it assumed (table names, defaults).
- **If something goes wrong:** symptom → which step and class to open, or which log or
  table to check.

Whether to commit `.claude/features/` is up to you. Committing it shares the decisions
with your team.

## Installation

The skill works best with web search, so the best practices it finds are current, and
with write access to your project, for the feature README. Without write access, it puts
the README in the reply and asks you to save it.

**Claude Code** — personal (all projects):

```bash
mkdir -p ~/.claude/skills/ricerca-soluzioni-dev
cp SKILL.md ~/.claude/skills/ricerca-soluzioni-dev/SKILL.md
```

Or project-scoped, committed with the repo:

```bash
mkdir -p .claude/skills/ricerca-soluzioni-dev
cp SKILL.md .claude/skills/ricerca-soluzioni-dev/SKILL.md
```

**Claude apps** — upload the skill folder in Settings → Capabilities → Skills, where
skill creation is enabled for your account.

## Usage

You don't invoke it. Just describe the problem the way you normally would, with your
notes if you have them:

> I need to handle file uploads in my Spring Boot app, files can be up to 2GB — what's
> the best way to do this?

> I'm getting `LazyInitializationException` when I serialize this entity and I don't
> understand where it comes from.

> I have to build the login in my Spring Boot app. Here are my notes: what I'd do, what I
> chose and what I don't know. What are my options?

Then read the options, pick a letter, and either write the code or say you want it
written. Remembered something you forgot to mention? Add it to *Developer notes* in the
feature README and tell the skill.

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
