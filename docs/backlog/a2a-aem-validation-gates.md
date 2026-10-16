# Backlog: A2A + AEM + Independent Validation Gates

**Status:** BACKLOG / DESIGN ONLY  
**Owner:** To be assigned  
**Created:** 2026-10-16  
**Implementation approval:** NOT GRANTED  
**Target:** Agent Village Hub / Agent Group

> Tracking note: This backlog document is an interim substitute because GitHub Issues are disabled on `JinGin126/agent-village-hub` as of 2026-10-16. When Issues are enabled, migrate this into a standalone issue and retain a backlink.

## 目标 / Why

为现有 Agent Group 预留未来多 Agent 协作的可靠性架构：**A2A (Agent-to-Agent Protocol)** for interoperability/task lifecycle; **AEM (Agent Evaluation Metric)**-style quality measurement; and independent **Validation / Approval Gates** to decide whether an output may be passed to the next agent.

**Important:** An upstream Agent reporting `completed` does **not** mean its output is correct. AEM evaluates quality, but a score is not an authorization mechanism. A separate gate must make the release decision, backed by objective checks and (where warranted) human approval.

## Current baseline

The current Agent Village Hub README describes a **custom relay protocol (poll/respond)**, a Hub-managed runtime and state machine, world-defined phases/turns/tools/visibility, hub-side tool validation, logs and observer views.

- **Existing relay is not assumed to implement standard A2A.**
- **Existing schema/action checks are not assumed to implement AEM.**
- Existing world functionality and relay API must be preserved; explore A2A as an **optional adapter** before considering migration.
- Any ERP → Accounting → QBO → Audit agent chain is **illustrative only**; this backlog authorizes no changes to Cloud Sync Hub or financial systems.

## Conceptual design

```text
Producer Agent
  └── A2A message/task/artifact (or existing relay via adapter)
         └── Durable handoff record: task IDs, schema, artifact hash, provenance
                └── Independent Validation Gate
                       ├── PASS: accept artifact and route to Consumer Agent
                       ├── FAIL: stop, explain, optionally safe rework
                       └── HOLD: require explicit human decision
                       └── Record decision and evaluation evidence
                              └── AEM-style quality evaluation and monitoring
```

**Separate states:**

- Protocol/execution state: submitted, working, completed, failed, canceled, input required — map to precise official A2A states during design.
- Validation state: not evaluated, validating, pass, fail, hold.
- Approval state: not required, pending, approved, denied; bind approvals to exact output hash/version.
- Dispatch rule: a `completed` task alone can **never** release a downstream action.

## Work packages / 子任务

1. **A2A fit-gap**: compare current relay with official A2A Agent Card/discovery, task lifecycle, messages, artifacts, authentication, streaming/async, cancellation, and versioning; recommend no change vs optional adapter vs migration.
2. **Handoff envelope**: define versioned and durable `taskId`, `runId`, `parentTaskId`, `fromAgent`, `toAgent`, `schemaVersion`, `artifactRef`, `inputHash`, `outputHash`, `attempt`, `evidenceRefs`, `policyVersion`, timestamps and correlation/trace IDs. Handle replay/duplicates.
3. **Independent gate**: schema and required-field validation, deterministic constraints, capability/permission checks, business invariants, policy binding, confidence limits, stop-on-unknown, and authorized human approval for high-risk actions. Reviewer agents can advise; deterministic acceptance criteria remain independent.
4. **AEM-style evaluation**: build golden tasks and ground truth for correctness, completeness, earliest-error attribution, invalid handoffs, false-pass and false-block rates, propagation, recovery rate, latency, token count and cost. Keep offline/continuous evaluation separate from execution permission.
5. **Observability**: trace producer → gate → consumer; preserve sanitized evidence, input/output fingerprints, validator version, reason codes, approval identity, retry history and final outcome. Do not log secrets or sensitive full payloads.
6. **Non-production POC**: two agents and a rejecting validator; test both standard A2A adapter (if selected) and existing relay path. Retain backward compatibility with existing worlds.
7. **Operational decision**: document security model, failure domains, replay/idempotency, backpressure, concurrency, token costs, latency, hosting/maintenance burden, rollback and migration plan.

## Acceptance criteria (for eventual implementation, not for this backlog entry)

- [ ] ADR comparing relay / optional A2A adapter / full migration, with cost-benefit and recommendation.
- [ ] Versioned handoff and status-mapping contracts.
- [ ] Demonstrate that an upstream `completed` result with wrong values or omitted fields is **blocked** before downstream dispatch.
- [ ] Cover successful, incomplete, wrong value, stale version, duplicated delivery, timeout, cancellation, unauthorized action and reviewer disagreement cases.
- [ ] Define reproducible AEM-style evaluation dataset/method, report quality and false-positive/false-negative results, and document error attribution limitations.
- [ ] Audit trail links all involved agents and gate decisions without leaking secrets/PII.
- [ ] Benchmark reliability, latency, throughput, token usage and infrastructure costs against current relay.
- [ ] Existing agent-village-hub worlds continue operating unmodified when optional feature is off.
- [ ] Split implementation into small approved GitHub issues with explicit rollback, tests and authority boundaries.

## Not in scope / Safety boundaries

- No implementation, deployment, service migration, production credential changes, financial operations or automated external writes authorized here.
- No promise of zero errors; objective is detection, containment, auditability and safe recovery.
- AEM is a **quality evaluation framework**, not a runtime enforcement engine; A2A manages Agent communication/task lifecycle, not correctness.

## References

- [Agent Village Hub README](../../README.md)
- [A2A protocol](https://a2a-protocol.org/)
- [AWS: Agent Evaluation Metric for multi-turn conversations](https://aws.amazon.com/blogs/machine-learning/agent-evaluation-metric-for-multi-turn-conversations/)
