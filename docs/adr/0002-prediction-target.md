# 0002 — Prediction target

- **Status:** Accepted
- **Date:** 2026-11-21

## Context

Public F1 data has no direct measurement of tyre wear. Any target is therefore derived, and the choice of derivation determines whether the model learns something real or something spurious.

The obvious target — the number of laps until the observed pit stop — is wrong. A pit stop is a **team decision**, taken for undercut, overcut, traffic or safety-car reasons, often well before the tyre is actually finished. Training on it teaches strategy, not tyre life.

The second obvious target, raw lap time, is also contaminated: fuel load, track evolution, traffic and driver input dominate the signal. A model fitted on it predicts the car and the race situation, not the tyre.

## Decision

Model the **remaining useful life of a stint as a survival problem with right-censoring**.

- The event of interest is the tyre reaching the end of its useful life, defined as degradation beyond a threshold relative to the reference lap of the same stint.
- A stint that ends in a pit stop **before** that threshold is **censored**, not complete. Censoring is the correct representation of "we stopped observing before the event".
- The model outputs a distribution over remaining laps, not a point estimate.

The exact definition of the threshold is a modelling decision. It is deliberately deferred and must be documented, with a sensitivity analysis over its value.

## Consequences

- Uncertainty becomes a first-class output, which is precisely what the downstream race simulator needs for counterfactual reasoning.
- Evaluation requires survival metrics (concordance index, integrated Brier score) rather than a plain error metric.
- Tooling must support censored observations; not every gradient-boosting library does.
- The label depends on a threshold, so the model card must state it explicitly and show how sensitive the conclusions are to it.

## Alternatives considered

- **Raw lap-time regression** — learns fuel load, traffic and car performance. Requires heavy correction before it says anything about tyres.
- **Binary classification of "pit next lap"** — directly encodes team strategy; high apparent accuracy, no predictive meaning.
- **Regression on laps-to-pit** — the censoring problem hidden behind a point estimate; systematically biased toward early stops.
- **Direct cliff detection per stint** — elegant, but too noisy on short stints and not comparable across circuits. Best used as a sensitivity check rather than the primary label.
