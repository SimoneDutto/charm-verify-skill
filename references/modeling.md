# Source-grounded modeling

## Discover the lifecycle before selecting abstractions

Start with metadata/charmcraft, configuration, actions, entry point, event
registrations, and existing specs. Use symbols and text search to build a
mechanism/interaction map. Inspect helpers, decorators, peer/relation state, workload adapters,
and relevant local or vendored libraries, including `lib/` when needed.
Do not automatically load all dependencies or assume unread library semantics.
Tests can inform intended behavior, but mocks do not prove side effects succeed.

For a broad verification request, target the charm's interacting lifecycle at a
finite abstraction level. Do not first find a suspicious handler and then make
that handler the entire model. The discovery catalogue below prompts coverage;
it is not a mandatory list of features every charm must implement.

Record each discovered mechanism, source entry points, state it reads/writes,
interacting mechanisms, and coverage: modeled, abstracted with justification, or
omitted. In particular, identify shared peer data, pending work, desired/applied
state, workload roles, and coordination ownership. State when omissions exclude
interleavings or limit a property's meaning. Abstraction is not a claim that the
omitted implementation is correct.

Honor an explicitly requested subsystem or property. Otherwise, a narrow slice
may be an incremental construction step, not a replacement for the broad target.
Separate modules/models can aid development, but independent passes do not check
composition. Preserve the relevant shared state and cross-mechanism transitions
in the integrated model.

For each modeled action, record:

| Action/event | Handler and helpers | Reads/writes | Guards and failure outcomes |
|---|---|---|---|
| Concrete source-backed action | Source paths and lines | Local/shared/workload state | Evidence or explicit assumption |

Use actual source locations, never invented placeholders in a delivered model.
Distinguish desired, persisted, applied, and advertised state where they differ.
Do not equate Juju leadership with database/workload primary status.

## Construct the model for discovery

1. Define finite state for persistent local/shared data, workload state, roles,
   pending/deferred work, and desired/applied/advertised state as relevant.
2. Encode source-backed actions and real scheduling boundaries. Include event
   replay, repeated reconciliation, role changes, and dependency outcomes where
   supported; do not constrain their order to an anticipated reproduction.
3. Compose mechanisms over shared state. Derive interactions from read/write
   dependencies and event delivery, rather than an arbitrary list of scenarios.
4. Parse and check small integrated instances while extending coverage. Prefer
   reducing identity/value/history domains over deleting lifecycle interactions.
   Keep enough identities and generations to express the properties being checked.
5. If coverage cannot be completed within the budget, identify the remaining
   mechanisms and affected properties. Do not describe a partial model as
   verification of the whole charm.

External failures must have documented or source-supported meanings, including
whether they raise, return an error, partially apply work, or leave it unchanged.
Label unverified outcomes as assumptions and validate them when triaging traces.
Do not introduce a convenient rejection merely to manufacture a counterexample,
or require a deployed reproducer before exploring source-supported behavior.

## Derive properties independently of suspected defects

Build a named property catalogue from the charm's intended obligations, not a
list of bugs already noticed. For each property record its meaning, source or
intent basis, observation boundary, required mechanisms, and any environment or
fairness assumptions. Use the catalogue to drive exploration even when no
violating execution is known.

Applicable examples include:

- disruptive operations require current ownership and valid health prerequisites;
- acknowledgments/publication reflect applied work, not only desired state;
- leadership transfer and deferred replay preserve pending-work obligations;
- cleanup or rotation of one relation preserves another relation's valid state;
- restore, membership, and configuration transitions preserve their shared
  prerequisites;
- pending work eventually progresses under explicitly justified recovery and
  event-delivery assumptions.

These are prompts, not universal policies. Establish the intended contract before
encoding it, check enabling reachability, and avoid tautological properties whose
truth is guaranteed by an abstraction that simply assumes the desired behavior.
Keep specification obligations distinct from the implementation actions being
checked; do not repair a source defect by putting the property into an action guard.

## Abstraction discipline

- Use finite identities, revisions, roles, and environment states. Explain bounds
  and what behaviors they exclude. Reduce irrelevant detail, not the suspected
  failure path. Do not silently add state constraints that hide bad states.
- Specify source-supported initial conditions; a convenient impossible startup
  can invalidate many otherwise plausible findings.
- Juju hook execution is sequential within a unit. Do not split synchronous calls
  into arbitrarily interleavable hooks. Model multi-unit/external interleavings
  where justified; partial outcomes require a supported failure/crash boundary.
- A failed side effect must preserve the state it did not actually change.
  Never define success as "make every desired invariant true."
- Every action assigns or explicitly leaves unchanged every state variable on
  every branch. Parenthesize conditional expressions inside conjunctions.
- Safety properties apply at specified boundaries. A transient internal state
  is not automatically a source defect visible between hooks.
- Liveness requires explicit environment and retry assumptions. Do not add
  fairness on successful side effects, dependency recovery, or manual rescue
  merely to make a property pass.
- For each property, identify a reachable enabling scenario and a meaningful
  failure condition. Report whether reachability was mechanically checked or
  only argued. Zero initial states or an unreachable trigger is not assurance.
- Import the standard module defining each operator. Keep module name and
  filename consistent and provide a proper header and `====` terminator.
- Prefer named readable predicates/actions over a JSON state-machine encoding.
  Include concise source mappings and assumptions in the spec itself.

## Event labels for explanations

For new models, include a diagnostic STRING variable `lastEvent`, initialized to
`"initial"`. Every real transition sets a concrete event and source location,
for example `"config_changed | src/charm.py:312"`. Use actual locations.
Parameterized actions must label each event meaningfully. Environment actions
must say they are environment actions rather than invent a Python handler.

Include `lastEvent` in the state tuple, but never use it in operational guards,
updates to operational state, or checked properties. It observes behavior, not
repairs it. TLC action locations alone are acceptable for existing specs; do not
silently rewrite a supplied model just to add labels. Distinguish diagnostic
labels (AI-authored mappings) from verified Python execution.

## Discovery catalogue

Use these as prompts, not universal policies or a required finding count:

- **Lifecycle/configuration:** desired vs written vs loaded revisions; startup,
  reconnect, reload/restart failures; new containers losing applied state.
- **Relations:** joined vs data-ready; version changes, removal, and rejoin;
  departed unit vs whole relation; removal of obsolete effective configuration.
- **Leadership/peers:** partial work transferred to a new leader; events before
  peer state exists; local vs shared writes; primary/leader disagreement.
- **Idempotency:** stable identity, duplicate resources, repeated unchanged
  hooks, preservation of another relation's valid state.
- **Rotation:** publish/activate ordering, overlap, retirement, rejoin, bounded
  generations without preventing repeated rotations.
- **Failure/recovery:** pending-work persistence, partial writes, retries,
  workload replacement, and an unavailable external dependency.
- **Rolling operations:** ownership, health/quorum prerequisites, acknowledgement
  after failure, and abandoned coordination.
- **Membership/storage/restore:** self vs peer departure, failed member removal,
  destructive bootstrap prerequisites, storage detach, and action failure.
- **Publication/readiness:** actual applied state vs desired state; publication
  before availability; stability after repeated reconciliation.
- **Composition:** multiple consumers/providers, cleanup of only the departing
  consumer, endpoint/TLS/config fallback consistency.

Favor distinct useful properties across evidenced mechanisms and their composition.
Do not invent behavior or encode an unconfirmed preference as established policy.
Label hypotheses and ambiguous intent explicitly.
