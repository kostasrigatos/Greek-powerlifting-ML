# The Greek Strength Calculator — A Plain-English Guide

This page explains the tool in this project in everyday language — no
formulas, no statistics jargon. If you're a coach or athlete and want to
understand what the tool tells you and where the numbers come from, this is
for you.

*(Looking for the technical details — the actual methods, code, and data
behind this? See [`README.md`](README.md).)*

---

## How to use the calculator, and how to read your result

*This section will be completed once the interactive tool is finished —
check back soon.*

**What you'll enter:**
- Your bodyweight
- Your sex
- Your age
- Your federation (or a default if you're not sure)

**What you'll get back:** two predicted totals — one calculated directly,
and one built up from separate squat, bench, and deadlift predictions added
together. They won't match exactly, and that's expected — think of the
second one as a sanity check on the first, not a second opinion that has to
agree perfectly.

Only the first number — the direct prediction — comes with a range attached.
The second number doesn't get its own range, since combining three separate
ranges honestly is more complicated than it sounds (your squat, bench, and
deadlift results tend to rise and fall together, so simply adding up three
ranges would overstate how uncertain the combined number really is). Treat
the second number as a useful comparison point, not a fully ranged
prediction in its own right.

**About the range you'll see next to your number:** the tool doesn't just
give you a single guess — it gives you a range, like "570 kg, likely
somewhere between 410 and 730 kg." That range isn't the tool being vague;
it's the tool being honest. It's telling you how much it actually knows
based on lifters similar to you in the data, rather than pretending to more
precision than it has.

**A caveat worth knowing:** the range tends to run wider for women than for
men. This isn't a flaw specific to you — it's because the dataset simply
has fewer female competitors than male ones, so the tool has less to learn
from for that group. We checked this carefully and it holds no matter which
method we used to calculate the range, which tells us it's a real limit of
the data, not a mistake in the tool.

---

## How this was built

This tool is the result of an eight-part analysis of Greek powerlifting
competition data, going back to 2015.

We started with real competition results pulled from OpenPowerlifting, the
largest public database of powerlifting meets in the world. The raw data
had a hidden problem: it mixed together results from full meets (squat,
bench, and deadlift) with results from bench-only and other partial events,
all lumped into one "total" column. That made the numbers misleading — a
bench-only result isn't comparable to a full meet total. Once we separated
these out and kept only full meets, the data told a much cleaner story.

We also made a point of not letting the tool "cheat." Competitors already
know their Wilks or Dots score from the scoreboard — but those scores are
*calculated from* your total in the first place. Letting the tool use them
to predict your total would be like letting it peek at the answer before
guessing, so we left them out entirely.

From there, we built and compared several ways of predicting an athlete's
total — starting with simple, transparent formulas and working up to more
flexible, modern machine-learning methods. At every step, we checked our
work by comparing against real results we'd deliberately held back, so the
numbers you see reflect genuine predictive ability, not just a good fit to
data the tool had already seen.

The last stretch of the project was about honesty, not just accuracy. A
prediction is only as useful as your ability to trust it — so we spent real
effort making sure the tool's confidence ranges are calibrated: when it
says "80% confident," it's actually right about 80% of the time when
checked against real results, not just a number that sounds reassuring.

**Curious about the Greek scene itself?** One of the earliest findings in
this project was that Greek raw powerlifters perform comparably to lifters
in nearby countries — Italy, Spain, Portugal, Bulgaria, Romania, and others
— once the data is cleaned up properly. The full comparison, with charts,
is in the [technical README](README.md#three-findings-in-pictures).

---

*Questions, corrections, or feedback? This project is open source — see the
[main repository](README.md) for details.*
