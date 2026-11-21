# 0003 — Inference contract

- **Status:** Accepted
- **Date:** 2026-11-21

## Context

The long-term goal is race-day assistance: the model is queried while a race is unfolding, using only what is known at that moment. Historical data and live data are different sources, but the model must not care which one it is talking to.

If the first version is built around a batch dataset — "give me the whole race and I return predictions" — then adding live inference later means rewriting the feature layer, the training pipeline and the API. This ADR exists to prevent that rewrite.

## Decision

**The unit of inference is a stint state, not a dataset.**

A prediction request carries:

- circuit and event,
- driver and car context,
- compound, normalised by event,
- the laps completed in the current stint, with causal fields only,
- optional current weather and track status.

A prediction response carries:

- the survival curve over remaining laps,
- median and credible interval,
- provenance: model version, data version, source commit.

Three rules follow from this contract:

1. **Features are computed incrementally.** Every feature must be computable from the laps already observed. Stint-level aggregates are forbidden, by construction, in the feature layer.
2. **Ingestion is an adapter.** A single interface with a historical implementation today and a live implementation later. The rest of the system depends on the interface, never on the source.
3. **Batch and live share the contract.** Historical replay is a client that replays a real race, lap by lap, through the same endpoint.

## Consequences

- Future leakage is excluded structurally rather than caught by review.
- The replay demo becomes a thin client with no modelling logic of its own.
- The feature library must be written for incremental computation, which constrains some convenient aggregate features.
- The API must be versioned from the start: adding fields is safe, changing the meaning of a field is not.
- Live ingestion remains out of scope for the first version, but its seam already exists.

## Alternatives considered

- **Batch dataset API** — simpler to build first, but blocks the stated end goal and forces a rewrite of the feature layer.
- **Offline model with no serving layer** — acceptable for a notebook, useless as a platform, and gives no answer to "how is this operated?".
- **Feature store (for example Feast)** — solves online/offline consistency at a scale and team size this project does not have. A versioned feature module with tests provides the same guarantees here.
