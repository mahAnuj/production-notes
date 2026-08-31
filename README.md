# Production Notes

Architecture case studies of systems I built, deployed and operate — written
from the source, with the measurements that justified each decision.

**Every one runs against a hard cost ceiling, and none of them can be scaled by
spending more.** One lives entirely on cloud free tiers; another runs on a
machine I already own with model spend inside a flat-rate rolling cap. Different
ceilings, same consequence — *add a bigger instance* is not an available move,
so the design has to absorb the constraint instead.

**What they cost to run: a domain name.** No servers, no managed databases, no
paid model APIs — the only recurring infrastructure cost is domain registration,
on the order of ₹1,700 (~US$20) a year. The other recurring cost is the AI coding
subscription used to build them, which for one of these systems doubles as the
runtime that generates its content.

That is the through-line of this collection.

These are not tutorials or pattern catalogues. Each one describes a system that
serves real users, including the decisions that were wrong, the incidents that
corrected them, and what I would do differently. Every number is a real
configured value or a real measurement.

---

## The studies

### [Keeping 500 stocks analysed on 100 LLM runs a night](vivatrades-nightly-analysis.md)
**VivaTrades** — a batch pipeline that refreshes research reports for 100 of the
NIFTY 500 each night, rotating the full universe in five days.

Splitting cheap work from expensive work so a rate-limit error in the fetch
stage cannot waste model spend. Using deterministic code wherever judgement is
not required. Guarding an agent-orchestrated pipeline with an output-count check
rather than an exit code, because agents fail by doing *less* rather than by
erroring — and splitting the run in two so a dead session costs half instead of
everything. Includes the timeout I set from the happy path, which quietly
destroyed 95% of coverage for five nights.

### [Running a production web app on nothing](shubh-bhagya-free-tier.md)
**Shubh Bhagya** — a Vedic astrology site with 68 routes and 76 long-form
articles, on a container that scales to zero.

Why only shared object storage counts as state when containers come and go. How
thirty sequential round-trips made a 13-second homepage, and why the fix was
removing them rather than speeding them up. Cutting completion tokens 85% while
*increasing* output quality. Why the effective timeout in a layered stack is
never the one in your own config — and what a 21-day silent failure teaches.

---

## Recurring lessons

Themes that showed up in both systems:

**Use a model only where judgement is genuinely required.** Both systems get
their biggest cost and correctness wins from *not* calling one — deterministic
assembly for anything mechanical, bounded reasoning for the rest.

**Silent failure is the expensive failure mode.** A pipeline that completes and
writes less than it should, a job that stops producing, a blank response cached
as success, a timeout firing on 95% of inputs while the job reports healthy.
Most of the costly incidents in both systems were silent, and the loud ones were
cheap by comparison. Health checks have to measure *output*, not process status.

**Hard ceilings have odd shapes, and the shape is the design.** A fixed number
of scheduler slots. Storage that is free in one region only — and not the one
the app runs in. Quotas keyed per model rather than per key. A rolling usage cap
you cannot buy past. These are not inconveniences to route around; they are what
determines the architecture.

**A rule that lives only in prose is not a rule.** A test suite whose docstring
promised it made no network calls, a code comment describing a timeout value that
had already been reverted — documented invariants decay silently until something
enforces them. I re-learned this writing these very documents.

---

## About

Built and operated by [Anuj Mahajan](https://github.com/mahAnuj) — staff-level
engineer, 15 years in distributed systems and cloud-native backends. These
systems are built solo with AI-assisted development; the architecture, data
pipelines and infrastructure decisions are mine.

**[anujmahajan.com](https://anujmahajan.com)** — case studies on all three
platforms, including VeoCabs which is not covered here, plus writing on AI
engineering and architecture.

Source repositories are private. These documents describe the architecture and
the reasoning, not the implementation.
