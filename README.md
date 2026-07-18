# cloud-itonami-isco-7536

Open Occupation Blueprint for **ISCO-08 7536**: Shoemakers and Related
Workers.

This repository designs a forkable OSS business for a shoemaking workshop
scheduling and logistics coordination practice: a workshop
scheduling and supply-coordination robot manages crew/task records under a
governor-gated actor, so a shoemaking crew keeps its own operating
records instead of renting a closed workforce-management SaaS.

**Maturity: `:implemented`.** `src/shoecoord/` implements the
`ShoeCoordActor` as a `langgraph.graph/state-graph`
(`shoecoord.actor`) wired to a `Shoemaking Workshop Coordination
Advisor` (`shoecoord.advisor`) and an independent `ShoeCoordGovernor`
(`shoecoord.governor`), following the itonami actor pattern
(ADR-2607121000): `:intake -> :advise -> :govern -> :decide -+-> :commit
(:ok? true) +-> :request-approval (:escalate? true, human-in-the-loop
interrupt) +-> :hold (:hard? true)`. HARD invariants (always hold, never
overridable): maker provenance, workshop provenance, no-actuation (`:effect`
must be `:propose`), a closed op-allowlist (`:log-work-record`,
`:schedule-crew-operation`, `:flag-safety-concern`,
`:coordinate-supply-order` — nothing else may ever be proposed), and a
permanent, unconditional block on any proposal that would directly finalize
a shoemaking-execution decision (e.g. deciding to proceed with a specific
cutting operation) or a workshop-safety-clearance decision (e.g. declaring
the workshop safety cleared for operation), or that would override a shop
safety officer's judgment. Always-escalate paths (human sign-off regardless
of confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)): `:flag-safety-concern`
(always) and `:coordinate-supply-order` above the registered cost
threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a workshop scheduling/logistics
coordination robot performs crew scheduling, job/commission/progress-record
logging and leather/sole/adhesive-materials supply-order coordination for a
shoemaking crew, under an actor that proposes actions and an
independent **Shoemaking Workshop Coordination Governor** that gates
them. The governor never dispatches hardware itself, never performs
shoemaking work on the workshop floor, and never finalizes a
shoemaking-execution decision or a workshop-safety-clearance decision, and
never overrides a shop safety officer's judgment; `:high`/`:safety-critical`
actions (such as a flagged cutting-tool-hazard/adhesive-fume-exposure/
equipment-condition concern, or an above-threshold supply order) require
human sign-off. **This actor coordinates WORKSHOP SCHEDULING/LOGISTICS
ONLY — it never performs shoemaking work itself, and it never makes a
workshop-safety-clearance decision itself.**

## Core Contract

```text
crew roster + workshop registration + safety-reporting policy
        |
        v
Shoemaking Workshop Coordination Advisor -> ShoeCoordGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a shoemaking-execution decision, finalize a workshop-safety-
clearance decision, override a shop safety officer's judgment, suppress an
operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `7536`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
