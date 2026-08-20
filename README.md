# AgentCollect UX Bug Detection Plan

This repository contains my system-design response for detecting broken or frustrating UX from AgentCollect's PostHog session replays before users report it.

The product surfaces include debtor payment and dispute flows plus the internal client dashboard. The difficult cases are often not crashes: an action may appear to do nothing, respond too slowly, produce unclear feedback, or cause repeated retries and abandonment.

## Final proposal

The complete proposal is in [PLAN3.md](./PLAN3.md).

It combines two priorities:

- Detect unfamiliar UX failures through behavioral evidence and expected outcomes.
- Keep the first production version simple by using managed PostHog capabilities before building custom replay infrastructure.

## Core reasoning

The detector should not equate frustration with a confirmed bug.

> **User action → expected response → observed response**

A finding becomes more credible when several signals agree:

> **Missing outcome + frustration + recurrence + business impact − valid explanations**

For example, repeated clicks alone are weak evidence. Repeated clicks following a valid submission, with no expected outcome, an adjacent error, and the same pattern across several sessions are much more useful.

## Proposed direction

- Audit the current PostHog data and privacy controls before choosing thresholds.
- Establish expected behavior from backend outcomes, frontend acknowledgement, product requirements, tests, and reviewed healthy sessions.
- Reuse existing events and add minimal generic action/outcome instrumentation only where needed.
- Use PostHog as the system of record for detection, aggregation, replay evidence, and notifications.
- Separate payment, dispute, and client-dashboard behavior rather than forcing one baseline across different users and workflows.
- Group related sessions into incidents and require human confirmation.
- Keep visual replay AI disabled unless a privacy-reviewed experiment proves that structured metadata misses valuable failures.
- Build custom processing only after a managed pilot demonstrates a measurable limitation.

## What the plan addresses

- Unknown data and ground-truth requirements.
- Strong, weak, and combined behavioral signals.
- False positives and missed-bug scenarios.
- Expected-behavior sources and their limitations.
- Alert deduplication and reviewer feedback.
- Privacy controls for debtor and payment data.
- Validation using known incidents, injected failures, and ambiguous healthy behavior.
- Scaling and exit criteria for custom infrastructure.

## Key unresolved decisions

The plan deliberately does not invent answers that require AgentCollect context:

- Whether managed AI processing is approved for replay data.
- Which product flow provides the best first pilot.
- Whether frontend sessions can be joined safely to backend outcomes.
- How many findings the team can realistically review.

These are decision gates in the plan, not details hidden behind premature architecture.

## Repository contents

- [PLAN3.md](./PLAN3.md) — final merged plan and recommended starting approach.
- [README.md](./README.md) — recruiter-facing summary and navigation.

This is intentionally a planning deliverable; it focuses on how to reduce uncertainty and choose the smallest maintainable production approach before implementation.
