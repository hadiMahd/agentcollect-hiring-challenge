# agentcollect-hiring-challenge

Planning exercise for detecting broken or frustrating UX from AgentCollect's PostHog session replays before users report it.

## Submission

- **Final deliverable:** [PLAN3.md](./PLAN3.md)
- **Approach:** managed-first, outcome-aware, explainable, privacy-conscious, and human-reviewed
- **Focus:** debtor payment and dispute flows plus the internal client dashboard

The prompt journey below is intentionally preserved to show how the initial idea was clarified before the final plan was produced.

---

## 🧠 PROMPT JOURNEY

* **Raw idea** – the messy thoughts straight from my brain cells
* **Polished** – ChatGPT refined the prompt
* **Current** – Now working with Codex to implement

---

## 🔍 MY RAW PROMPT

> hi codex, i have a debt payment platform where users pay their debt, there are buttons and forms to submit in the ui, and my main goal is to try to catch ux bugs by analyzing posthog replays before users report them.

what im thinking about is multiple clicks without results, multiple refreshes, going back and forth, abandoning after trying to do something, errors if they are available, and also any patterns that tell me there is a problem in the ux or that the user is frustrated before he even finishes using the app.

but i also dont want to assume something like multiple clicks automatically means there is a bug. it could be a slow network, validation, an intentionally disabled button, permissions, or just normal user behavior.

the mental model im thinking about is basically: **"is the user's want/need satisfied?"** and **"did his action/request get the appropriate response?"**

so i think one important part is figuring out what the expected behavior actually is after an action. for example if someone clicks submit, what is supposed to happen after that, and how can we reliably know it happened? im thinking the ground truth could come from successful sessions, backend success events, product requirements, tests, or the application logic, but i dont know yet what data is actually available.

some things i know i need to understand before making decisions are what posthog captures reliably, whether elements have stable identifiers, what represents a successful payment/dispute/action, whether sessions can be connected to backend outcomes, what normal user behavior looks like, and whether debtor flows and the internal dashboard should be treated differently.

i also want the solution to generalize to ux bugs we dont already know about instead of hardcoding known buttons/pages or rules like "5 clicks = bug".

i want something scalable, efficient, feasible to actually run in production, and not something that needs an expensive llm or vision model analyzing every replay if structured events can solve most of the problem.

privacy is also important because payment/dispute replays can contain pii, so i would prefer to work with masked or extracted behavioral data and avoid sending raw replays to third-party models unless there is a very good reason.

i want you to help me brainstorm this before we choose an architecture or write code.

tell me:

* what important things i still dont know and should figure out first
* where you would get the expected-behavior ground truth from
* which signals you would start from and which ones you would not trust alone
* where false positives could happen
* what assumptions could slap us later in production if we get them wrong
* what questions i should ask the product/engineering team and why each question matters
* how you would think about scalability, efficiency and feasibility
* the important tradeoffs between a simple rule-based approach, learned behavioral baselines, and heavier ai/llm/replay analysis
* how you would avoid alert fatigue
* and what privacy/security concerns i should think about

have a lot of scrutiny when making decisions. dont jump straight into a fancy architecture. challenge my assumptions and tell me where my thinking could be wrong.

for important decisions or questions, tell me **why it matters, what you would assume by default if we dont know the answer yet, and what would change depending on the answer.**

---

## 📝 CHATGPT ADJUSTED PROMPT

> # Refined Prompt

You are helping me plan a production-grade system for detecting UX bugs from PostHog session replays on a debt-payment platform.

The main product surfaces are:

* debtor payment flows,
* debtor dispute flows,
* and an internal client dashboard with forms, buttons, reports, and case-management actions.

My goal is to detect broken or frustrating UX **before users report it**, especially issues that do not necessarily produce a crash or exception.

My current mental model is:

**user action → expected outcome → observed outcome**

In other words:

* did the user get what they were trying to achieve?
* did the product respond appropriately to their action?
* if not, is there enough evidence to consider the interaction suspicious?

The initial behavioral signals I am thinking about are:

* repeated clicks on the same element,
* clicks with no visible result,
* repeated refreshes,
* back-and-forth navigation,
* abandonment after an important action,
* long waits followed by retries,
* rage clicks or dead clicks if available,
* errors or exceptions when available,
* and repeated similar patterns across different sessions.

I do **not** want to assume that one signal automatically means a bug.

For example, repeated clicks could also be caused by:

* slow network responses,
* validation errors,
* intentionally disabled controls,
* permission restrictions,
* user confusion,
* or normal impatient behavior.

One of the biggest things I need to figure out is the **expected-behavior ground truth**.

For example, if a user clicks “Submit dispute”, I need to know what should happen next and how to verify that it actually happened.

Possible sources of ground truth might include:

* backend success events,
* successful historical sessions,
* product requirements,
* automated tests,
* application logic,
* frontend state changes,
* or input from the product/engineering team.

I do not yet know which of these are available or reliable.

Other unknowns I already know I need to clarify include:

* exactly what PostHog events and properties are captured reliably,
* whether DOM elements have stable identifiers,
* whether PostHog sessions can be safely connected to backend business outcomes,
* what normal behavior looks like for each product surface,
* which actions are intentionally blocked by validation or permissions,
* whether debtor and client-dashboard behavior require separate baselines,
* and how much alert volume the team can realistically review.

I want the solution to:

* generalize to UX bugs we have never seen before,
* avoid hardcoding known buttons, pages, or incidents,
* be scalable and computationally efficient,
* be feasible to operate in production,
* minimize false positives and alert fatigue,
* remain explainable to engineers and product teams,
* and handle privacy carefully because payment/dispute sessions may contain PII.

I would prefer not to use an expensive LLM or vision model on every replay if most detection can be done from structured event metadata. Heavier analysis should only be used if it clearly improves detection enough to justify the cost and privacy tradeoff.

Before proposing architecture or implementation, **challenge my reasoning first**.

I want you to do the following:

1. Identify the most important things I still do not know.
2. Explain which unknowns must be resolved before implementation and which ones can safely remain assumptions for an MVP.
3. Identify the best sources of expected-behavior ground truth and explain the strengths and weaknesses of each.
4. Evaluate the behavioral signals I listed:

   * which are strong,
   * which are weak,
   * which should only be trusted when combined with other signals.
5. Give realistic false-positive examples.
6. Identify cases where a real UX bug could be missed by my current thinking.
7. Propose high-value questions I should ask the product/engineering team.
8. Compare the tradeoffs between:

   * rule-based detection,
   * learned behavioral baselines,
   * anomaly detection,
   * LLM-based reasoning,
   * and visual replay analysis.
9. Recommend the smallest feasible first version that could realistically run in production.
10. Explain how that approach would scale as session volume grows.
11. Explain how to prevent alert fatigue and duplicate alerts.
12. Identify privacy/security risks and how to minimize exposure of PII.
13. Red-team the design and identify assumptions that could create serious problems later in production.

For every important question or architectural decision, use this format:

**Question / decision**
**Why it matters:**
**Default assumption if unanswered:**
**What changes depending on the answer:**
**Tradeoff:**

Do not jump immediately to a complex architecture.

Be skeptical, challenge my assumptions, and optimize for a solution that is **generalizable, explainable, scalable, efficient, privacy-conscious, and actually feasible to ship.**

---

## How I approached the problem

I treated this as an uncertainty-reduction exercise rather than jumping directly to an architecture.

1. **Separate symptoms from bugs.** Repeated clicks, refreshes, abandonment, and long waits can indicate frustration, but each also has legitimate explanations.
2. **Define observable contracts.** The useful question is whether a valid action received an appropriate frontend and backend outcome—not whether the user merely looked frustrated.
3. **Combine independent evidence.** Confidence should increase when a missing outcome, retry behavior, errors, recurrence, and business impact agree.
4. **Segment unlike behavior.** Debtors completing a payment and trained agents using a dashboard should not share one behavioral baseline.
5. **Prefer existing capabilities.** PostHog should handle replay storage, querying, aggregation, and managed analysis where its capabilities and privacy terms fit.
6. **Keep humans responsible for conclusions.** The system ranks investigation candidates; product and engineering reviewers confirm whether an incident is a bug.

## Why the final plan is managed-first

The simplest maintainable solution is not a new replay-processing platform. A custom exporter, event processor, feature store, anomaly model, and alerting service would add operational work before proving that PostHog cannot meet the need.

The final plan therefore starts with:

- A telemetry and privacy audit.
- Existing PostHog events and replay signals.
- Minimal action/outcome instrumentation only where ground truth is missing.
- Structured metadata for broad detection and restricted replay access for confirmation.
- A shadow-mode pilot before team notifications.
- Explicit exit criteria before considering custom infrastructure.

This is a deliberate build-versus-buy decision, not a rejection of custom detection. If the managed approach shows a measurable gap in precision, recall, privacy, latency, cost, or query capability, that evidence defines what a custom component must solve.

## How I would validate the direction

The pilot should include healthy sessions, historical incidents, injected failures, and intentionally ambiguous behavior. Useful measures include:

- Precision among the highest-ranked findings.
- Recall on known and injected failures.
- Time from first occurrence to detection.
- Duplicate incident rate and alerts per session volume.
- False-positive reasons for each product surface.
- Reviewer effort and whether findings are actionable.
- Performance on routes and releases not used during calibration.

Thresholds and alert frequency should come from the observed data and the team's review capacity, not arbitrary numbers chosen during planning.

## What this submission demonstrates

- Product reasoning grounded in user outcomes rather than isolated analytics signals.
- Skepticism about false positives, historical baselines, and inferred intent.
- Production pragmatism through managed services and explicit build criteria.
- Privacy-first handling of payment, dispute, and debtor replay data.
- Explainable incident evidence and human review instead of opaque automation.
- A path from unknowns to a small pilot, measurable validation, and controlled scaling.

See [PLAN3.md](./PLAN3.md) for the complete decision gates, implementation sequence, privacy controls, and validation plan.
