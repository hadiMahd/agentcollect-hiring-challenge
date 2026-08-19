# AgentCollect UX Bug Detection — Merged Plan

## Goal

Detect suspicious UX failures before users report them without treating frustration as proof of a bug.

Core model:

> **User action → expected response → observed response**

Confidence increases when several forms of evidence agree:

> **Missing outcome + frustration + recurrence + business impact − valid explanations**

## Decision gates

**Question / decision:** Can PostHog managed AI process AgentCollect replays?

**Why it matters:** PostHog Self-driving reduces maintenance but processes sensitive replay data.

**Default if unanswered:** Do not enable AI analysis.

**What changes depending on the answer:** If approved, trial Self-driving; otherwise use PostHog events, insights, alerts, and manual replay confirmation.

**Tradeoff:** Lower maintenance versus privacy and vendor dependence.

**Question / decision:** Which flow should be piloted?

**Why it matters:** Payment has greater impact but may involve redirects and third-party providers.

**Default if unanswered:** Do not select a flow until observability and business priority are confirmed.

**What changes depending on the answer:** The selected flow determines outcome events, latency expectations, and severity.

**Tradeoff:** Highest impact versus easiest reliable validation.

**Question / decision:** Can sessions be correlated with backend outcomes?

**Why it matters:** Backend and UI success can disagree.

**Default if unanswered:** Detect frustration only; do not claim outcome failure.

**What changes depending on the answer:** Reliable correlation enables outcome-aware detection and silent frontend-failure detection.

**Tradeoff:** Better precision versus instrumentation and privacy work.

**Question / decision:** What alert volume can be reviewed?

**Why it matters:** Thresholds must match human capacity.

**Default if unanswered:** Run silently without team notifications.

**What changes depending on the answer:** Review capacity determines thresholds, digest frequency, and real-time escalation.

**Tradeoff:** Recall versus alert fatigue.

## Phase 1: Audit before architecture

Inspect the current PostHog configuration and representative sessions:

- Captured events, properties, exceptions, rage clicks, navigation, and network timing.
- Replay and event sampling rates.
- Stable action identifiers, route templates, releases, and feature flags.
- Existing payment, dispute, report, and case-action outcome events.
- Whether frontend attempts can join safely to backend results.
- Validation, permission, pending, duplicate, and external-redirect states.
- Masking, retention, access, residency, and AI-processing restrictions.
- Expected alert ownership and review capacity.

Do not rely on DOM changes unless the audit proves they are available as queryable metadata. Avoid replay snapshot parsing.

## Phase 2: Establish expected behavior

Use this hierarchy:

1. Backend business outcome.
2. Frontend acknowledgement or state transition.
3. Product requirements, application logic, and automated tests.
4. Healthy historical sessions for timing and sequence baselines.
5. Human product or engineering confirmation.

Reuse existing events where possible. Add generic instrumentation only when necessary:

- `ux_action_attempt`: surface, stable action name, random operation ID.
- `ux_action_result`: matching fields, outcome, and duration.
- Outcomes: `success`, `validation`, `denied`, `pending`, or `failure`.

Do not capture entered values or business identifiers.

## Phase 3: Managed-first detection

Keep PostHog as the system of record. Do not initially build an exporter, replay parser, feature store, anomaly service, or separate alerting pipeline.

### If managed AI is approved

Trial PostHog's session-replay signal source on a restricted, masked scope. It can identify confusion, abandonment, failures, and exceptions and cluster them into human-reviewed reports. See [PostHog Session Replay](https://posthog.com/docs/session-replay).

### If managed AI is not approved

Use PostHog events, insights, dashboards, and aggregate alerts. Replays remain restricted evidence for human confirmation.

### Candidate evidence

Prioritize:

- Missing or failed outcome followed by retry or abandonment.
- Blocking exception adjacent to an action.
- Abnormally slow response followed by repeated attempts.
- No observable response after a valid action.
- The same pattern recurring across users.
- A significant increase after a release or feature-flag change.

Never trust rage clicks, dead clicks, refreshes, navigation loops, long waits, or abandonment alone.

Segment debtor payment, debtor dispute, and client-dashboard behavior separately.

## Incident grouping and alerting

Group related evidence by:

> surface + action + missing outcome + failure pattern + release or feature flag

Produce one incident per pattern, not one alert per session.

Each incident should contain:

- Expected and observed behavior.
- Evidence contributing to confidence.
- Affected sessions and users.
- Business severity.
- Release, feature flag, browser, and device context.
- A small set of masked replay links.
- Plausible non-bug explanations.

Reviewer labels:

- `confirmed`
- `ux-friction`
- `expected`
- `performance`
- `telemetry-gap`
- `duplicate`
- `insufficient-evidence`

Run in shadow mode first. Derive alert thresholds and digest frequency from observed precision and agreed review capacity rather than inventing fixed numbers.

## Privacy controls

- Mask inputs and sensitive text before capture.
- Exclude payment widgets, authentication screens, documents, and dispute narratives.
- Strip query strings and disable network body capture.
- Never capture payment data, account numbers, debtor details, tokens, or permissions.
- Restrict replay access and audit usage.
- Use short retention appropriate to operational needs.
- Verify masking with synthetic sensitive values.

Keep Replay Vision disabled initially because it sends rendered video and raw events to Google Gemini. Consider sampled use only after privacy approval and evidence that structured detection misses valuable visual failures. See [PostHog Replay Vision](https://posthog.com/docs/replay-vision).

## Validation

Evaluate using healthy sessions, historical incidents, injected failures, and ambiguous examples:

- Submit action that produces no response.
- Backend failure after submission.
- Slow success causing retries.
- Valid validation error.
- Valid permission denial.
- Normal repeated pagination clicks.
- Successful backend result with broken UI acknowledgement.
- Masking failure canary.

Measure:

- Precision among top findings.
- Recall on known and injected failures.
- Time from first occurrence to detection.
- Alerts per 1,000 sessions.
- Duplicate incident rate.
- False-positive reasons by surface.
- Performance on held-out routes, releases, and later time periods.
- Reviewer effort.

## Scaling and exit criteria

Process structured metadata across all sessions and open replays only for selected incidents. Let PostHog handle storage, querying, aggregation, clustering, and notification where supported.

Consider custom infrastructure only when measured evidence shows a material limitation in:

- Precision or recall.
- Privacy controls.
- Query capability.
- Detection latency.
- Cost at AgentCollect's volume.
- Managed-service reliability.

Until then, custom replay processing is unnecessary.
