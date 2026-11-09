# Explained results and iterative refinement

## Persist structured outcomes

Write `results.json` at each attempt and update it after each tool result.
Use this shape (replace example values with evidence; do not invent counts):

```json
{
  "attempt": "attempt-001",
  "scope": "actual investigated scope",
  "overall_status": "inconclusive",
  "manifest": "manifest.json",
  "parse": {
    "status": "not_run",
    "output": null
  },
  "checks": [
    {
      "name": "ActualNamedOperator",
      "kind": "invariant",
      "mode": "modelcheck",
      "status": "not_run",
      "reason": "Awaiting execution",
      "spec": "check-01/CharmModel.tla",
      "config": "check-01/CharmModel.cfg",
      "output": null,
      "trace": null,
      "exit_code": null,
      "distinct_states": null,
      "finding_ids": []
    }
  ]
}
```

Populate the full real list, including baseline (`deadlock` or `reachability`),
all invariants, and temporal properties. Optional unknown values remain null.
Use paths relative to the attempt. Include tool arguments and termination details
in the manifest or per-check metadata. Link artifacts only after they exist.

Overall status precedence:

1. `inconclusive` if parsing failed, evidence is insufficient, any requested check
   errored/timed out/was interrupted/not run, or only simulation was performed.
   State any counterexamples separately; incomplete work never erases them.
2. Otherwise `counterexample` if at least one check violated its property.
3. Otherwise `passed` only when all requested exhaustive checks completed.

Keep source-fidelity confidence separate from checker status. A genuine TLC
violation of a bad model is still a checker counterexample, not a Python bug.

## Explain every finding

Assign stable IDs (`F001`, `F002`, ...), preserving IDs across attempts when the
same underlying candidate reappears. Do not silently merge away failing checks.
For each failed check, include:

1. **Property and meaning:** exact operator and the behavior it expresses.
2. **Initial conditions:** relevant initial values, environment, assumptions,
   and bounded identities/revisions.
3. **Witness:** a numbered table of actual TLC states/transitions, event/action,
   changed values, and implementing source locations.
4. **Failure:** exact violating predicate or liveness obligation. For temporal
   failures show the loop/stuttering/back-edge marker and what never happens.
5. **Source validation:** registration, guards, helpers/decorators, data readiness,
   side-effect success/failure, and subsequent event delivery along the whole
   enabling path. Check real return/error semantics, not just method names.
6. **Disposition:** source-supported candidate, modeling artifact, or unresolved
   assumption/intent; explain confidence and missing evidence.
7. **Next action:** targeted model repair, missing-source investigation, or
   concrete source regression-test scenario.
8. **Discovery and added value:** identify whether the execution was first found
   by TLC, noticed during source inspection, supplied by the user, or already
   known before modeling. Explain which ordering or composition TLC exposed or
   explored beyond an existing example. Do not relabel an incidental review
   finding or a formalized known defect as a model-discovered bug.

Keep evidence levels explicit: a model counterexample, source-supported
reachability, an isolated execution with mocked operations, and deployed behavior
are different claims. Validate the guards and dependency semantics that enable
the full trace; an injected HTTP error does not establish its natural incidence,
and a violated coordination invariant does not by itself establish an outage.

Never fill gaps with a plausible imagined reproduction. TLC may omit a violating
successor or render only part of a trace: say what is missing. Event labels in the
spec are authored source mappings, not proof Python executed.

If one false initialization/transition invalidates several findings, identify
every dependent finding. Preserve both the trace and the invalidation rationale.
Several different failed properties may share a root cause; explain that without
hiding the individual results.

## Report format and chat UX

Write an attempt `report.md` with:

```text
# Verification: <scope> / <attempt>
## Outcome
Overall checker status, separate source-fidelity summary, incomplete work.
## Scope and assumptions
Mechanism coverage inventory, shared-state interactions, source revision/dirty
state, bounds, omissions and their consequences, fairness.
## Property catalogue
Named obligations, intent/source basis, observation boundaries, required
mechanisms, environment assumptions, and enabling-reachability evidence.
## Checks
| Check | Kind | Status | Evidence |
## Findings
For each ID: property, initial conditions, witness, source validation,
discovery origin, model-checking contribution, disposition, confidence, and
regression-test suggestion.
## Changes since previous attempt
Model changes and why; finding IDs retained, superseded, or invalidated.
## Next iteration
Concrete remaining work and links to editable inputs and immutable evidence.
```

In chat, lead with a concise outcome and all-checks table, then useful witness
explanations and artifact links. Do not dump raw logs or present AI interpretation
as checker output. Explain timeouts/errors as explicitly as violations.
Do not claim "no bugs" after a pass. Say which bounded abstraction passed.
For broad requests, lead with integrated coverage achieved and remaining gaps,
not just the first failing slice. Do not use state counts, failed-property counts,
or report volume as substitutes for useful, source-validated discoveries.

Maintain investigation-root `findings.md` as a small ledger:

| ID | Property/check | First attempt | Latest evidence | Disposition | Reason/next action |
|---|---|---|---|---|---|

Use `candidate`, `model_artifact`, `unresolved`, `confirmed_by_test`, or
`fixed_and_tested` as appropriate. A model pass alone cannot confer the last two.
Record older dispositions/evidence when changing a finding's classification.

Maintain `latest.md` with the latest completed/incomplete attempt, working input
paths, key decisions, pending work, and links to the ledger/report. This is a
handoff aid, not a second conversation/session store.

## Refine without erasing evidence

- Edit working inputs, never a checked snapshot. Create a fresh attempt.
- Record before/after rationale for assumptions, bounds, guards, initialization,
  properties, and fairness. Do not silently remove requested checks.
- Correct source-inaccurate behavior; do not "repair" source-supported violations
  by construction. Ambiguous intended policies require user guidance.
- Recheck the full relevant property set on the revised model. Compare finding
  dispositions; disappearance under a smaller bound is not a resolution.
- If asked to fix source, add a focused regression test using repository tooling,
  show the original failure when feasible, make the surgical fix, and rerun it.
  Update the model only if necessary to represent the changed source behavior,
  then rerun the model checks. State separately if a real test is unavailable.
- Do not run arbitrary repository code or install dependencies just for discovery.
  Source tests/fixes follow the user's authorization and normal harness safeguards.
- Iterate on clear modeling errors, but stop and explain if repeated attempts
  make no progress or require an unapproved major budget/scope change.
