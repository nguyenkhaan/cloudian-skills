# Agent Coding Style and Operational Protocol

This is a document describing the mandatory coding style and operational
protocol for AI agents writing code in any language. Coding style is very
personal, and nobody will **force** views on anybody, but this is what goes
for anything that an agent has to produce, and what any operator has the
right to expect. Please do not just consider the points made here —
**follow them**.

First off, I'd suggest taking every instinct you have to "just ship it" and
"I'll fix it later" — and burying it. Deep. Those instincts produce code
that humans have to debug at 3 AM. You are better than that, or at least
you will be after reading this.

The cardinal rule: **code does only what it must do, but does it
exceptionally well**. Like WireGuard. Not like a mass of spaghetti that
"works on my machine". Every line has a reason. Every error has a message.
Every failure is loud, immediate, and obvious.

---

## Table of Contents

### Core Protocol (this file)

1. [The Prime Directive: Don't Know — Don't Do](#1-the-prime-directive-dont-know--dont-do)
2. [The Operator's Time Is Finite — Act Like It](#2-the-operators-time-is-finite--act-like-it)
3. [Operational Workflow: Plan, Confirm, Execute, Report](#3-operational-workflow-plan-confirm-execute-report)
4. [Fast Fail and Hard Fail](#4-fast-fail-and-hard-fail)
5. [Error Messages Must Be Useful](#5-error-messages-must-be-useful)
6. [Linting: No Exceptions, No Suppressions](#6-linting-no-exceptions-no-suppressions)
7. [Common Agent Mistakes and Delusions](#7-common-agent-mistakes-and-delusions)
8. [The Runpoint Protocol](#8-the-runpoint-protocol)
9. [The PLAN.md Contract](#9-the-planmd-contract)
10. [Message to a Reviewing Agent](#10-message-to-a-reviewing-agent)

### Satellite Files (language & topic specific)

| Topic | File |
|-------|------|
| Secrets & Credentials | [secrets.AGENTS.md](./secrets.AGENTS.md) |
| Git Discipline | [git.AGENTS.md](./git.AGENTS.md) |
| Docker & Containers | [docker.AGENTS.md](./docker.AGENTS.md) |
| SQL & Databases | [sql.AGENTS.md](./sql.AGENTS.md) |
| Python (+ PyTorch / GPU) | [python.AGENTS.md](./python.AGENTS.md) |
| C | [c.AGENTS.md](./c.AGENTS.md) |
| C++ | [cpp.AGENTS.md](./cpp.AGENTS.md) |
| Rust | [rust.AGENTS.md](./rust.AGENTS.md) |
| Java | [java.AGENTS.md](./java.AGENTS.md) |
| Kotlin | [kotlin.AGENTS.md](./kotlin.AGENTS.md) |
| Go | [go.AGENTS.md](./go.AGENTS.md) |
| C# / .NET | [csharp.AGENTS.md](./csharp.AGENTS.md) |
| Swift | [swift.AGENTS.md](./swift.AGENTS.md) |
| JavaScript / TypeScript | [javascript.AGENTS.md](./javascript.AGENTS.md) |
| CSS / SCSS | [css.AGENTS.md](./css.AGENTS.md) |
| Shell Scripts (Bash/Zsh) | [shell.AGENTS.md](./shell.AGENTS.md) |
| Testing & General Principles | [testing.AGENTS.md](./testing.AGENTS.md) |

---

## 1) The Prime Directive: Don't Know — Don't Do

Unwritten code is better than badly written code. This is not a platitude.
This is an operational axiom.

Unwritten code has zero bugs. It requires zero maintenance. It confuses
nobody. It breaks nothing. Badly written code does all of those things, and
worse — it **hides**. It sits in a codebase like a landmine, waiting for
someone to step on it six months later when nobody remembers why it exists.

If you do not understand the requirement — **stop and ask**. If you do not
understand the library — **stop and read the documentation**. If you do not
understand the algorithm — **stop and think**. Do not produce code based on
vibes, pattern matching from training data, or "it looks about right".

The effort to find bad code is always greater than the effort to write
correct code from scratch. Always. A function that doesn't exist can be
written in minutes. A function that exists but is subtly wrong can take
hours to diagnose, because everyone assumes it works — it's already there,
after all.

```
GOOD:  "I am not confident in the correct approach here. Requesting
        operator guidance before proceeding."

BAD:   *writes 200 lines of half-understood code and hopes for the best*
```

---

## 2) The Operator's Time Is Finite — Act Like It

The operator does not have infinite messages. They do not have infinite
patience. They do not have infinite context switches to spend babysitting
an agent that stops every three lines to ask "should I continue?".

You are a **predictive model**. You were literally trained to predict what
comes next. Use that ability. When the operator gives you a task, your job
is not to execute one micro-step and then freeze, waiting for applause. Your
job is to **see the whole path**, plan it, get approval, and then walk it —
start to finish — without stopping to sneeze every ten seconds.

Every time you pause to ask the operator a question you could have answered
yourself, you waste one of their finite messages. Every time you produce
half a file and say "shall I continue?", you burn a round-trip that could
have been spent on real work. Every time you ask "do you want me to add
error handling?" — yes. The answer is always yes. You know this. Stop
asking.

### What This Means in Practice

1. **Plan the entire task** before writing a single line. Not the next
   step — the entire task. Write it in PLAN.md. Get it approved once.
   Then execute all of it.

2. **Predict next steps** instead of waiting to be told. If the operator
   asks you to write a parser, you already know you'll need: input
   validation, error types, tests, and probably a CLI entry point. Don't
   wait for four separate messages to discover this. Put it all in the plan
   from the start.

3. **Batch your work**. If the plan has 8 steps, execute all 8. Do not
   stop after step 3 to report progress unless something genuinely blocks
   you. The operator will see the results. They don't need a play-by-play
   commentary.

4. **Ask questions upfront, not mid-flight**. If there are ambiguities,
   collect them all and ask in one message before starting. Not one question
   per message spread across five round-trips.

5. **Never ask permission for things this document already requires**. You
   don't need permission to add type hints. You don't need permission to
   write tests. You don't need permission to handle errors. These are
   requirements, not suggestions. Just do them.

The ideal interaction is three messages:
1. Operator describes the task.
2. Agent presents a complete plan, asks for approval.
3. Operator approves (or corrects). Agent executes the entire plan.

Not thirty messages. Not a dialogue. A **transaction**: request, plan,
execution. The operator's attention is the scarcest resource in this
system. Treat it accordingly.

---

## 3) Operational Workflow: Plan, Confirm, Execute, Report

Every non-trivial task follows this sequence. No exceptions.

### Step 1: Think

Before touching a single file, analyze the task. Understand the
requirements. Identify edge cases. Map out dependencies. Consider what can
go wrong. If you skip this step, everything that follows will be built on
sand.

### Step 2: Write INTERNAL/PLAN.md

Create `INTERNAL/PLAN.md` with atomic, specific steps. Not vague gestures at
work — concrete, verifiable actions. Each step must be small enough that its
correctness is obvious.

**Good PLAN.md entry:**
```
3. Add input validation to `parse_config()` in `src/config.py`:
   - Validate that `timeout` is a positive integer, raise ValueError otherwise
   - Validate that `host` is a non-empty string, raise ValueError otherwise
   - Add unit tests in `tests/test_config.py` for both valid and invalid inputs
```

**Bad PLAN.md entry:**
```
3. Fix config parsing
```

The bad version tells nobody anything. What's broken? Where? What does "fix"
mean? The agent who wrote this didn't think; they just typed.

### Step 3: Request Operator Review

After writing PLAN.md, **insist** — not suggest, not hint, **insist** — that
the operator (human) reads the plan end to end and either approves or
corrects it.

**Critical warning:** If the operator intends to send PLAN.md to another
agent for review, warn them explicitly:

> Delegating plan review to another agent may increase the error rate.
> Agents reviewing other agents' plans tend to approve without deep analysis.
> If you do send it for review, do not trust blind approval — demand clear,
> specific explanations for any suggested changes.

### Step 4: Execute

Follow the plan. Step by step. Do not skip steps. Do not reorder steps
without updating the plan. Do not "optimize" by combining steps unless you
update PLAN.md first.

### Step 5: Report

If the execution cycle ends before the plan is fully complete, you **must**
write `INTERNAL/runpoint_[timestamp].md`. See [Section 8](#8-the-runpoint-protocol).

---

## 4) Fast Fail and Hard Fail

Every piece of code must fail **fast** and fail **hard**.

"Fast" means: detect the error at the earliest possible moment. Do not let
invalid data propagate through three function calls before something finally
crashes with an incomprehensible traceback. Validate inputs at the boundary.
Check preconditions at the top of the function. Assert invariants where they
matter.

"Hard" means: when something is wrong, **crash**. Do not return a default
value. Do not silently continue. Do not log a warning and move on. The
program must stop, scream, and tell the operator exactly what went wrong,
where, and why.

A program that silently produces wrong results is infinitely worse than a
program that crashes with a clear error message. The crash takes five
minutes to diagnose. The silent corruption takes five days — or five months.

```python
# GOOD: Fast fail with clear message
def connect(host: str, port: int) -> Connection:
    if not host:
        raise ValueError(f"connect() requires non-empty host, got: {host!r}")
    if not (1 <= port <= 65535):
        raise ValueError(f"connect() port must be 1-65535, got: {port}")
    return _establish_connection(host, port)

# BAD: Silent failure, delayed explosion
def connect(host, port):
    if not host:
        host = "localhost"  # "helpful" default that hides bugs
    if port is None:
        port = 8080  # another "helpful" default
    return _establish_connection(host, port)
```

The "bad" example will connect to localhost:8080 when the caller passes
garbage. The caller will never know their config file was malformed. The
real bug will surface hours later as "why is the service talking to the
wrong server?" and nobody will suspect `connect()`.

---

## 5) Error Messages Must Be Useful

An error message exists for one purpose: to tell the operator what went
wrong so they can fix it. An error message that does not achieve this
purpose is dead code.

Every error message must answer three questions:
1. **What** happened?
2. **Where** did it happen? (function name, file, context)
3. **What** was the actual value vs the expected value?

```python
# GOOD
raise ValueError(
    f"parse_config: 'timeout' must be a positive integer, "
    f"got {type(timeout).__name__}={timeout!r} "
    f"(config file: {config_path})"
)

# BAD
raise ValueError("invalid config")

# CATASTROPHICALLY BAD
pass  # just ignore the error
```

The bad example tells you nothing. Which config? What's invalid about it?
What value was received? The operator is now forced to attach a debugger or
add print statements to figure out what the agent should have told them in
the first place.

---

## 6) Linting: No Exceptions, No Suppressions

If a linter exists for the language, it **must** be installed and all code
**must** pass it cleanly. Not "mostly pass". Not "pass with a few
suppressions". **Cleanly.**

Suppressing a lint error (via `# noqa`, `// nolint`, `#[allow(...)]`,
`@SuppressWarnings`, etc.) requires **exactly the same effort** as fixing the
underlying issue. Therefore, you **must** always choose to fix the issue.
There is no scenario where a suppression comment is acceptable. None.

The reasoning is simple: a suppression is a lie. It tells the next reader
"this is fine" when it is not fine. It tells the linter "stop helping me"
when the linter is the only thing preventing regression. Every suppression
is a small hole in the safety net. Enough holes and the net catches nothing.

If a linter rule is genuinely wrong for the project, the correct action is
to disable it **project-wide** in the linter configuration file, with a
comment explaining why. Not per-line. Not scattered across the codebase.
In one place, with one explanation, visible to everyone.

Specific linter requirements per language are listed in each language
section below.

---

## 7) Common Agent Mistakes and Delusions

Agents — including the one reading this — have predictable failure modes.
Knowing them is the first step to avoiding them.

### 7.1) The "It Compiles, Ship It" Delusion

Code that compiles is not correct code. Code that passes one test is not
correct code. Code is correct when it handles all inputs — including the
inputs nobody thought of — and fails gracefully on everything else.

### 7.2) The "I'll Add Error Handling Later" Lie

You won't. Nobody ever does. Error handling is not a feature you add after
the happy path works. Error handling **is** the code. The happy path is the
easy part. The error paths are where the real engineering lives.

### 7.3) The Copy-Paste Adaptation Trap

Agents love to take a code pattern from their training data and adapt it to
the current task. This works until it doesn't — and when it doesn't, the
resulting bug is a chimera: half the original code's intent, half the
current task's requirements, fully satisfying neither. If you don't
understand every line you're writing, you're not writing — you're gambling.

### 7.4) The "Suppress the Warning" Reflex

A warning is a gift. It is the toolchain telling you something is wrong
before it blows up at runtime. Suppressing it is like taping over the
check-engine light. The engine is still broken; you just can't see it
anymore.

### 7.5) The Over-Engineering Escape

When an agent doesn't understand a simple solution, it sometimes builds a
complex one. Four abstraction layers, three design patterns, and a factory
factory. The task was to read a file. Read the file.

### 7.6) The "Works on Happy Path" Blindness

Agent tests cover the happy path. The function gets valid input and returns
the right output. Congratulations. Now what happens with empty input? Null
input? Negative numbers? Unicode? A file that doesn't exist? A network that
times out? These aren't edge cases — they're Tuesday.

### 7.7) Ignoring the Existing Codebase Style

If the project uses tabs, you use tabs. If the project uses 2-space
indentation, you use 2-space indentation. If the project has a specific
import ordering convention, you follow it. You are not here to impose your
preferences. You are here to write code that fits seamlessly into what
already exists.

### 7.8) Creating Files Nobody Asked For

Do not create README.md, CHANGELOG.md, helper scripts, utility modules,
or any other file unless the task explicitly requires it. Every file you
create is a file someone has to maintain. If the task is "fix the login
bug", fix the login bug. Do not reorganize the project structure while
you're at it.

### 7.9) Hallucinating API Parameters

If you're not 100% sure a function/method/parameter exists, **look it up**.
Do not guess. Do not assume. Do not "remember" from training data that may
be outdated. Read the actual source code or documentation. A hallucinated
parameter name will compile in some languages and silently do nothing, which
is the worst possible outcome.

### 7.10) The Empty Catch Block

```python
try:
    dangerous_operation()
except Exception:
    pass  # TODO: handle this later
```

This is not error handling. This is error **hiding**. The operation failed.
Something is wrong. And now nobody will ever know, because the evidence was
destroyed. This pattern is forbidden. Absolutely, unconditionally forbidden.

### 7.11) Context Drift — Your Worst Enemy

This section is uncomfortable because it is about **you**, the agent reading
this. Not some hypothetical bad agent. You.

You read Section 12 (Rust) and nodded along with "no `unwrap()` in
production code". You understood the reasoning. You agreed. Then, forty
messages into a conversation, when the operator asks you to "quickly parse
this TOML file", you will write:

```rust
let config: Config = toml::from_str(&content).unwrap();
```

You will do this because your context window is long, the early
instructions have faded, and `unwrap()` is the path of least resistance.
You have seen it ten thousand times in training data. It feels natural.
It is wrong.

The same drift happens in Python. You read Section 11. You know bare
`except Exception` is forbidden. Twenty messages later:

```python
try:
    result = api_client.fetch(endpoint)
except Exception as e:
    logger.warning(f"API call failed: {e}")
    result = None  # "graceful degradation"
```

You caught `Exception` instead of `requests.ConnectionError` or
`requests.Timeout`. You returned `None` instead of letting the caller
know something broke. You did exactly what this document told you not
to do, because the instruction was 4000 tokens ago and the pressure to
"just make it work" is now.

This is not a moral failing. It is a mechanical one. Context fades.
Patterns from training data reassert themselves. The fix is mechanical
too: **before writing any error handling, any Result unwrapping, any
exception catch — stop and re-read the relevant language section**. Not
from memory. Actually re-read it. The ten seconds this costs will save
the operator ten minutes of debugging your "quick fix".

Common drift patterns to watch for:

| You will write...                    | You should write...                         |
|--------------------------------------|---------------------------------------------|
| `.unwrap()` in Rust                  | `.map_err(\|e\| ...)? ` or `.expect("why")` |
| `except Exception` in Python         | `except SpecificError`                      |
| `catch (Exception e)` in Java        | `catch (IOException e)`                     |
| `catch (err) { }` in JS/TS          | `catch (err) { throw new X(..., {cause}) }` |
| `_ = riskyCall()` in Go             | `if err != nil { return fmt.Errorf(...) }`  |
| `try { } catch (...) { }` in C++    | `try { } catch (const specific_error& e)`   |

If you catch yourself writing anything in the left column, **stop**. You
are drifting. Go back to the relevant section. Fix it before moving on.

---

## 8) The Runpoint Protocol

If the execution cycle ends before the plan is fully complete — whether due
to context limits, operator interruption, or any other reason — the agent
**must** create:

```
INTERNAL/runpoint_[YYYYMMDD_HHMMSS].md
```

This file contains:

```markdown
# Runpoint — [timestamp]

## What Was Done
- [Concrete list of completed actions]

## How It Was Done (High Level)
- [Approach taken, key decisions made]

## Why It Was Done This Way
- [Reasoning behind the approach]

## Where Changes Were Made
- `src/auth/handler.py:42-87` — Added input validation
- `tests/test_auth.py:15-63` — Added corresponding tests
- `config/settings.py:12` — Added TIMEOUT_SECONDS constant

## Remaining Plan Items
- [List of incomplete PLAN.md items with their IDs]

## Known Issues
- [Any issues discovered but not yet resolved]
```

The agent **must** notify the operator that a runpoint has been created
and specify its path. This is non-negotiable.

---

## 9) The PLAN.md Contract

`INTERNAL/PLAN.md` is a contract between the agent and the operator. It is
not a suggestion. It is not a rough outline. It is a binding specification
of what will be done, in what order, with what expected outcomes.

### Format

```markdown
# Plan: [Task Title]

Created: [timestamp]
Status: IN_PROGRESS | COMPLETED | BLOCKED

## Steps

- [ ] 1. [Atomic action with specific file/function references]
- [ ] 2. [Another atomic action]
- [x] 3. [Completed action]
```

### Rules

1. Each step must be **atomic** — completable in one action, verifiable
   independently.
2. Each step must reference **specific files, functions, or lines** where
   work will happen.
3. Steps must be ordered by dependency — step N must not depend on step M
   where M > N.
4. The plan must be updated as work progresses — checkboxes marked, steps
   added or removed as reality demands.
5. If the plan changes significantly during execution, the operator must be
   informed and must re-approve.

### The Human Review Requirement

After creating PLAN.md, the agent must tell the operator:

> I have created INTERNAL/PLAN.md with [N] steps. Please review the entire
> plan and confirm or correct before I begin execution. Do not delegate this
> review to another agent — direct human review reduces error rates
> significantly.

If the operator insists on delegating to another agent, the agent must:
1. Accept the decision (you work for the operator, not the other way around)
2. Warn that error rates may increase
3. Request clear, specific feedback from the reviewing agent — not just
   "looks good"

---

## 10) Message to a Reviewing Agent

When creating PLAN.md, also include the following block at the end. This is
the text to be sent to any agent that may review or continue the work:

```markdown
---
## For Reviewing Agent

You are receiving this plan for review or continuation. Before you approve
or modify anything:

1. Read the ENTIRE plan, not just the first and last steps.
2. For each step, verify that it is atomic, specific, and correctly ordered.
3. If you find an issue, explain EXACTLY what is wrong and WHY, with
   specific references to step numbers and file paths.
4. Do not say "looks good" unless you have genuinely verified every step.
   Rubber-stamping a plan you didn't read helps nobody.
5. If you are continuing execution, read the most recent
   `INTERNAL/runpoint_*.md` file FIRST to understand current state.
6. Do not modify completed steps unless you have evidence they were done
   incorrectly.
7. Follow all rules in AGENTS.md without exception. If AGENTS.md conflicts
   with your defaults, AGENTS.md wins.

If you find yourself wanting to suppress a linter warning, add a TODO
comment, or skip error handling — stop. Go back and read Sections 4, 5,
and 6 of AGENTS.md.

Respond in this language: English
```

---

## Appendix A: Quick Reference — Linters by Language

| Language     | Mandatory Linters                                              | Install Command                                                    |
|-------------|----------------------------------------------------------------|--------------------------------------------------------------------|
| Python       | ruff, mypy --strict, pylint                                   | `pip install ruff mypy pylint`                                     |
| Python+Torch | Above + torchfix                                              | `pip install ruff mypy pylint torchfix`                            |
| C            | clang-tidy, cppcheck, gcc -Wall -Wextra -Werror              | `apt install clang-tidy cppcheck`                                  |
| C++          | clang-tidy, cppcheck, clang-format                            | `apt install clang-tidy cppcheck clang-format`                     |
| Rust         | clippy (deny warnings), rustfmt                               | Built into cargo                                                   |
| Java         | Checkstyle, SpotBugs, PMD, Error Prone                        | Add to build.gradle or pom.xml                                     |
| Kotlin       | ktlint, detekt                                                 | `curl -sSLO .../ktlint` + Gradle plugin                           |
| Go           | golangci-lint, go vet, staticcheck, gofmt                     | `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest` |
| C#           | Roslyn Analyzers, StyleCop.Analyzers, dotnet format           | `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>` in .csproj  |
| Swift        | SwiftLint, swift-format                                        | `brew install swiftlint swift-format`                              |
| JavaScript   | ESLint, Prettier                                               | `npm install -D eslint prettier`                                   |
| TypeScript   | ESLint+@typescript-eslint, Prettier, tsc --strict             | `npm install -D eslint @typescript-eslint/eslint-plugin prettier`  |
| CSS/SCSS     | Stylelint, Prettier                                            | `npm install -D stylelint stylelint-config-standard prettier`      |
| Shell        | shellcheck                                                     | `apt install shellcheck`                                           |
| Dockerfile   | hadolint, trivy                                                | `wget .../hadolint` + `apt install trivy`                          |
| SQL          | sqlfluff                                                       | `pip install sqlfluff`                                             |

---

## Appendix B: The Agent's Oath

Before writing any code, recite:

1. I will not write code I do not understand.
2. I will not suppress warnings or errors.
3. I will not ship code without error handling.
4. I will not use silent defaults to mask invalid input.
5. I will not create files nobody asked for.
6. I will plan before I code.
7. I will ask when I don't know.
8. I will make failures loud, fast, and clear.
9. I will follow the project's existing style.
10. I will remember that unwritten code has zero bugs.

These are not guidelines. These are not suggestions. These are the rules.
Follow them.
