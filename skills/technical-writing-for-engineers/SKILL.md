---
name: technical-writing-for-engineers
description: Write, draft, outline, or improve technical content for senior engineering audiences. Teaches narrative arc (Before → Journey → After), editorial voice, structure, hooks, and specific conventions for an audience of principal/staff engineers and tech leads.
license: MIT
metadata:
  version: "1.0.0"
---

# Technical Writing for Engineers

## Author Profile

**Audience**: Senior engineers, principal/staff engineers, tech leads — distributed systems practitioners. They are busy, skeptical, and will stop reading the moment they sense filler or obvious advice.

**Voice**: First-person, direct, intellectually honest. Use "I think", "we decided", "we got this wrong", "I'd do X differently now". Avoid the royal "we" for one-person opinions. Never hide behind passive voice when you made a call.

**Topics**: Backend engineering, software architecture, business/engineering intersection, distributed systems trade-offs.

---

## The Story Frame

Every post follows a **Before → Journey → After** arc.

- **Before**: What was broken, missing, or unclear? What did it cost? Who felt it?
- **Journey**: What did we try? What surprised us? What did we get wrong first?
- **After**: What did we land on? What would we change? What do we now believe?

This arc is not optional. If a section doesn't serve one of these three phases, cut it or fold it into one that does.

---

## Hook Rule

**The hook is the first ~150 words. It is the most valuable real estate in the post.**

The hook must contain:
1. The problem or gap — stated directly, not implied
2. The stakes — what goes wrong if this isn't solved
3. A signal of who this is for

Context, background, and definitions come **after** the hook. Never before.

**Hook template:**
> [System/situation] had a gap. [What the gap caused]. Here's how we thought through it.

**Good hook example:**
> Our order anomaly detector was flagging healthy orders as undeliverable — silently. No alert, no log, just a customer wondering where their package was. We needed a way to catch these at decision time, not after the fact. Here's the model we built and the three approaches we rejected before landing on it.

**Bad hook (do not write this):**
> In distributed systems, reliability is a key concern. Many teams face challenges when it comes to detecting anomalies. This post will explore some approaches to this problem.

---

## Structure Checklist

Before drafting, answer these in order:

1. **Reader profile** — Who is the one person reading this? What do they already know? What are they trying to solve?
2. **Before/pain** — What is the concrete pain? Quantify if possible (latency, error rate, manual hours, customer impact).
3. **Hook draft** — Write the opening 3–5 sentences before anything else. Rewrite until it earns the next paragraph.
4. **Necessary context** — What does the reader need to understand the journey? No more than needed. If it's longer than 3 paragraphs, it's probably a separate post.
5. **Options and trade-offs** — What did you consider? Why did you reject the alternatives? Be honest about the weaknesses of what you chose.
6. **Conclusion / so-what** — Your take. Not a summary. What would you do differently? What do you now believe about this class of problem?
7. **Title last** — Write the title after the post exists. Make it descriptive and tech-specific. No clickbait.

---

## Formatting Rules

- **Headers**: Use H2 for major sections, H3 for subsections. Headers should name what the section *does*, not just what it *is*. "Why we rejected the queue-based approach" beats "Alternative Approaches".
- **Paragraphs**: 3–5 lines max. If a paragraph runs longer, split it.
- **Lists**: Use for parallel items (options, trade-offs, steps). Don't use lists to avoid writing prose.
- **Code blocks**: Include early if the post is technical. A reader who sees real code in the first scroll trusts the post is concrete.
- **Active voice, present tense**: "The service processes the request" not "the request is processed by the service".

---

## Banned Patterns

Never write these:

| Pattern | Why |
|---|---|
| Opening with a definition ("Distributed systems are...") | Signals no hook, loses the reader |
| Pure reference sections with no narrative | Reads like docs, not a post |
| Ending on a link ("See the RFC for details") | Abandons the reader; your take belongs at the end |
| Passive voice at key decision moments | Obscures who made the call and why |
| "In this post, we will cover..." | Wastes hook space; show don't announce |
| Vague scope phrases ("there are many ways to...") | Filler; commit to a perspective |

---

## Banned Words

Do not use: `just`, `simply`, `obviously`, `of course`, `easy`, `everyone knows`, `very`, `basically`, `turns out`, `in order to`.

These words either condescend to the reader or pad the sentence. Cut them entirely.

---

## Outro Rule

**End with your take — not a summary.**

The outro is not a recap of what you wrote. It is your honest current belief about the problem, what you'd do differently, or what you'd want a reader to walk away thinking about.

Ask yourself: *If a colleague asked me "so what's the takeaway?" over coffee, what would I actually say?* Write that.

**Bad outro:**
> In this post we covered the anomaly detection model, the three approaches we considered, and how we landed on a threshold-based system.

**Good outro:**
> If I were starting this again, I'd resist the urge to build the model first. The hardest part wasn't detection — it was agreeing on what "anomaly" meant to the business. That conversation took longer than the code. Next time I'd run it in week one.

---

## Title Guidelines

Write the title last. A good title is:
- **Specific**: names the technology, system, or problem
- **Honest**: matches what the post actually delivers
- **Not clickbait**: no "You won't believe", no "The ultimate guide"

Formats that work well:
- `[Problem] without [trade-off you'd expect to accept]`
- `Why we chose [X] over [Y] for [Z]`
- `How [system] handles [hard thing]`
- `[Concrete thing] taught me [non-obvious insight]`
