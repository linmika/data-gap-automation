# Closing data gaps without a data pipeline

*A design note on automating the "look it up by hand" work that sits between a data warehouse and
an operations tool. Concept only: no code, no system details.*

## The problem

Operations teams often work from two places at once:

- **The warehouse** knows *which* records need attention: every exception, every reason code,
  every day's list.
- **The operational tool** holds *the detail someone actually needs to act*: the evidence, the
  attachment, the latest note.

Two gaps keep them apart:

| Gap | What it looks like |
|---|---|
| **Incomplete tables** | The warehouse has the record and its status, but not the field the team needs. That field only exists inside the tool. |
| **Freshness** | The table arrives the next day; the decision is needed today. |

The usual bridge is a person opening record after record and copying values into a sheet. It
works at fifty records a day. At thousands, it doesn't get done, and the decisions that depended
on it get made blind.

## The idea

Automate the person, not the system.

A small browser tool does exactly what the person was doing, in the person's own signed-in
browser, with exactly the access they already have:

```
  shared sheet                 the user's browser                  the operational tool
  ─────────────               ────────────────────                ────────────────────
  list of records   ───────►  for each record:          ───────►  the same view the
  to look up                  look it up, as the user             user would open
                              pick the relevant entry
  filled-in detail  ◄───────  write the result back     ◄───────
  + who filled it
```

It adds no new access path, no service account, no stored passwords. If the user can't see a
record by hand, the tool can't either.

## Design principles

**Same person, end to end.** The tool runs only when the person signed in to the sheet and the
person signed in to the tool are the same. Every row it writes records who wrote it.

**No shared secrets.** Access to the sheet is decided by the organisation's own sign-in, not by a
key pasted into a config file that someone will eventually forward.

**Pick, don't grab.** Records usually have history: the same exception can happen twice. The sheet
carries enough context (a reason, a time) to choose the right entry, and the result says plainly
when it had to guess, so a person can check.

**Slow on purpose.** One lookup at a time, at a human-ish pace. Fast enough to clear a day's list
in the background; gentle enough that nobody on the tool's side would notice.

**Off by default.** Nothing runs on a timer until someone switches it on, and nothing sends
automatic messages. A job that keeps running after people have stopped watching it creates alarm,
not value. (Learned first-hand: an early version sent alert e-mails long after the person who
set it up had moved on to something else.)

**The sheet is the access boundary.** Whatever the tool writes into the sheet is readable by
everyone the sheet is shared with. Treat it with the same care as the source.

## When not to do this

- **The field is already in the warehouse** and next-day is good enough: write a query.
- **The tool has an export or an official interface**: use that.
- **You need volume or speed beyond a careful human's pace**: ask the owning team for a proper
  interface. This approach is a bridge, not a replacement for one.
- **You're unsure whether it fits your organisation's rules**: ask the system owner first.
  Automating your own access is still automation, and it is better discussed before it scales
  than after.

## What it took to get right

- **Agreeing on identity was the real design work.** Not the lookup itself: deciding that "who ran
  it" must be one verifiable person, and building every check around that.
- **Verification beats confidence.** Each result was checked against the tool by hand on a sample
  before anything ran unattended. A tool's own log saying "done" proves nothing.
- **Hand-offs need an off switch.** Anything meant to be passed to another team needs a clearly
  documented way to stop it, not just to start it.

## Status

Built and tested end to end on a small scale for an internal operations use case, then paused
deliberately while its fit with internal policy is confirmed. The implementation is kept private.

---

Text licensed under [CC BY 4.0](LICENSE).
