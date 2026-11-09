# MCP execution and durable evidence

## Discover capabilities

Find and inspect the available tools before invoking them. This installation was
tested with these TLA+ MCP capabilities:

| Capability | Example tool | Important inputs |
|---|---|---|
| Syntax and level checking | `tlaplus_mcp_sany_parse` | Absolute `fileName` |
| Symbol inspection | `tlaplus_mcp_sany_symbol` | Absolute `fileName` |
| Exhaustive checking | `tlaplus_mcp_tlc_check` | `fileName`, `cfgFile`, `extraJavaOpts`, `extraOpts` |
| Random smoke testing | `tlaplus_mcp_tlc_smoke` | Same categories; simulation only |
| Saved trace replay | `tlaplus_mcp_tlc_trace` | `fileName`, `cfgFile`, `traceFile` |

Names may have server prefixes or different parameter names. Use the discovered
schema. Do not assume a server is installed because this guide names it. If
missing, report the blocker and preserve modeling work; do not simulate a pass or
install an alternative silently.

The tested server rejects paths outside its open workspace. Personal skill files
may live elsewhere, but inputs and attempt directories must be in the workspace.
Use absolute paths. Respect access exclusions and workspace boundaries.

## Prepare one immutable attempt

Suggested investigation layout (adapt to an existing project layout):

```text
tla/
  CharmModel.tla                 # editable working model
  CharmModel.cfg                 # full intended check set
  verification/<scope>/
    latest.md                    # current attempt, disposition, next step
    findings.md                  # stable finding IDs across refinements
    attempt-001/                 # choose a new unused directory each time
      CharmModel.tla
      CharmModel.cfg
      manifest.json
      source/                    # relevant exact source snapshots
      parse.txt
      check-00-baseline/
        CharmModel.tla
        CharmModel.cfg
        output.txt
      check-01-<property>/
        CharmModel.tla
        CharmModel.cfg
        output.txt
        <server-generated trace files, if available>
      results.json
      report.md
```

Snapshot all local TLA+ imports needed by the model, preserving module resolution.
Run against these snapshots, not the editable working file. Copy the same module
closure into each isolated check directory so parallel or subsequent checks do
not share mutable artifacts. Use safe directory labels, not unchecked path input.

In `manifest.json` record UTC time, source root, commit/dirty state if available,
hashes of relevant source and model/config files, module imports, scope, model
bounds, assumptions, and checker options. Copy relevant non-secret source evidence
so uncommitted edits and future line shifts do not destroy traceability. If a
dependency cannot be snapshotted, record its identity and the resulting limitation.
Preserve the mechanism coverage inventory and property catalogue with the attempt
and link them from the manifest. Record which interactions were excluded by
bounds or omissions so subsequent checks cannot silently change the target.
Capture/check hashes before and after snapshotting; do not claim a consistent
snapshot if inputs changed during collection.

Create `results.json` with the full planned check list marked `not_run` before
running tools. Update it after every check, not just at the end.
Attempts are write-once inputs with append-only execution evidence: never edit
their spec/config/source to repair a model. Start another attempt instead.

## Isolate checks without changing the model

1. Read the entire config and enumerate every named `INVARIANT(S)` and
   `PROPERTY/PROPERTIES`, including multiline lists. For configs authored by this
   skill, use named operators and `\*` line comments for clarity.
2. Form a shared base by removing only those check declarations. Preserve
   constants, model values, INIT/NEXT or SPECIFICATION, state/action constraints,
   views, symmetry, overrides, and other settings verbatim. Constraints, symmetry,
   and fairness may affect assurance; document their implications, especially for
   liveness. If parsing is ambiguous, ask or consult tooling; never guess and
   silently drop settings or properties.
3. Baseline: use the base config with deadlock checking enabled, unless the input
   explicitly disables it. If disabled, call this baseline **reachability**, not
   a deadlock pass. Do not add duplicate contradictory CHECK_DEADLOCK directives.
4. For each property, use the same base plus exactly its original declaration.
   Disable deadlock detection in property runs so a deadlock cannot mask their
   results: replace an existing CHECK_DEADLOCK setting, or add
   `CHECK_DEADLOCK FALSE`. This is a checking-only difference; record it.
5. Preserve every requested check in the results, including checks not attempted.
   If there are no declared properties, explain the limited baseline result and
   propose source-grounded properties; do not call a baseline a full verification.

Never narrow the initial states, strengthen guards, or add fairness while
isolating checks. Independent checks operate on one identical behavior model.

## Parse, check, and bound resources

Parse the snapshot with SANY first and save its response. A parse failure marks
the attempt inconclusive and the planned checks `not_run`; fix a working copy.

Default to one worker, 512 MB Java heap, and a 60-second exploration limit per
check when the tool supports it:

```json
{
  "fileName": "/absolute/workspace/path/check-01/CharmModel.tla",
  "cfgFile": "/absolute/workspace/path/check-01/CharmModel.cfg",
  "extraJavaOpts": ["-Xmx512m", "-Dtlc2.TLC.stopAfter=60"],
  "extraOpts": ["-workers", "1"]
}
```

This is an invocation shape, not a command to run with placeholder paths.
The stopAfter setting requests a TLC exploration limit, not a guaranteed wall-clock
kill during parsing or blocked operations. Inspect the returned termination reason.
Never infer exhaustive completion from exit code zero alone. Increase limits only
with a reason and appropriate user approval for substantial resource increases.
Do not start unbounded parallel runs in a shared environment.

Run the baseline first. A shared parse/initialization/evaluation error makes the
model invalid; preserve the error and mark remaining checks `not_run` rather than
repeating it for every property. A deadlock counterexample does not invalidate
property runs. A timeout does not prove a shared model error; continue the other
bounded checks if resources permit.

After a violation, continue all remaining planned properties. Do not stop at one
finding. Record each result independently, even if failures appear related.
Simulation is optional early feedback, labeled `simulation`; it never substitutes
for an exhaustive pass.

## Preserve outputs and classify

The tested server returned full TLC text and exit status; it created a binary
`.tlc` counterexample beside the checked module. Its tool description mentions
`.vscode/tlc/`, but actual locations vary: use the path reported by the invocation.
Do not assume it also saves a durable stdout log.

Save the returned checker text verbatim to `output.txt` using available harness
file tools. Keep explanatory prose separate. Save invocation arguments, tool name,
exit status when exposed, checker version, explored-state counts, and termination
reason. If output is truncated/offloaded, retrieve the full available evidence
through supported tools. Do not reconstruct missing "raw output" from memory:
record the limitation, preserve any supplied artifact, and label excerpts as such.
If evidence cannot be saved, report that failure rather than claiming persistence.

| Status | Required evidence |
|---|---|
| `passed` | Exhaustive model-check completion, no reported violation/error, meaningful initial states; only within declared bounds/constraints |
| `counterexample` | Explicit invariant/temporal/deadlock violation; retain trace or diagnostic even if a full witness is unavailable |
| `checker_error` | Syntax, configuration, evaluation, resource failure, or tool execution error |
| `timeout` | Explicit time/resource-budget stop before exhaustive completion |
| `interrupted` | Cancellation or lost completion; outcome not established |
| `not_run` | Not invoked; include reason |

Keep mode separately: `modelcheck` vs `simulation`. A simulation with no violation
is `no_violation_observed`, not `passed`. Missing/contradictory termination evidence
is inconclusive, not success. Never label every nonzero exit as a source bug.

When replay is useful, use the preserved spec, exact check config, and reported
trace path. Replay is interpretation of an existing witness, not a fresh exhaustive
pass. For a new ALIAS, make a derived analysis copy/config and preserve its lineage;
do not modify original attempt inputs. Preserve raw states alongside any ALIAS view.
