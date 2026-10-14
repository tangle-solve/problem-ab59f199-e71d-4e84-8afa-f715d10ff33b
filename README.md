<!-- solve-problem:ab59f199-e71d-4e84-8afa-f715d10ff33b -->
# Separating withholding from outages

A [Tangle Solve problem](https://solve.tangle.tools/problems/ab59f199-e71d-4e84-8afa-f715d10ff33b).

## What is the problem?

In a proof-of-work blockchain, a *selfish miner* (or pool) withholds solved blocks to build a private chain, releasing them strategically to orphan honest blocks and gain excess revenue. The observational signature of selfish mining at the public layer is anomalous fork structure: unusually many forks, short private-chain releases, and "surprising" orphan rates concentrated on specific miners or pools.

The problem: **given on-chain data (block tree, timestamps, miner/pool attribution per block, network propagation delays), distinguish selfish withholding from benign causes of the same statistical anomalies.** Concretely, build a detector D that maps an observation window to {withholding, benign}, controlling the false-positive rate against four confounders:

1. **Network outages / partitions** — a miner temporarily disconnected mines on a stale view, producing forks without strategic intent.
2. **Hashpower migration** — pools gaining/losing share shift fork statistics, mimicking a growing attacker.
3. **Propagation races** — natural latency competition between large miners yields extra orphans proportional to size (this is *expected* even absent attack).
4. **Pool misattribution** — block attribution to pools (via coinbase tags) is noisy; untagged blocks and proxy miners corrupt per-entity statistics.

Formal setting: point process of block discoveries with rates proportional to hashpower shares; a public tree-growth process; the null hypothesis H0 is the honest protocol plus the four confounders above; H1 adds a strategic withholder following some generalized selfish-mining policy (e.g., Eyal–Sirer or partial-withholding variants). One wants a test with controlled Type-I error under H0 and power against H1, or a causal estimator isolating the withholding effect.

## Why does it matter?

Detection is the prerequisite for credible attack attribution: exchanges, pools, and protocol designers currently cannot distinguish "pool X is attacking" from "pool X has bad connectivity," so accusations (like those around the documented Qubic/Monero campaign) are contested. A calibrated detector unlocks deterrence (social/protocol sanctions against verified attackers), measurement of attack prevalence and profitability in the wild, and validation of the theoretical selfish-mining literature's assumptions about rational attacker behavior. Conversely, without it, every anomaly claim is unfalsable and honest large pools get smeared.

## What's already known?

- **Theory:** Eyal–Sirer (2014) and Sapirshtein et al. (2016) characterize optimal selfish mining; the attack's *revenue* footprint is well understood, but these are ex ante models, not detectors. Symmetry/robustness analyses (e.g., uncle-rate-based studies in Ethereum) show orphan-rate elevation correlates with withholding but is confounded by size.
- **Statistical detection:** Li, Campajola, Tessone (2024, Sci. Rep.) propose statistical tests flagging anomalous forks in PoW systems — the current state of the art referenced in the census. These tests are correlational: they flag anomalous fork statistics but do not causally separate withholding from outages, migration, races, or misattribution. The census marks this as "beyond 2024 statistical tests," i.e., their limitations are the motivation.
- **Case study:** Lee & Kim (2026, AFT) reconstruct a documented selfish-mining campaign (Qubic against Monero) with ground-truth-ish evidence, providing labeled data and observed tactics/limits — the census calls this the "2026 Qubic reconstruction."
- **Gap:** no published detector with proven false-positive control under the four confounders; the census explicitly notes "anomalous forks require causal interpretation," pointing toward causal inference (difference-in-differences, synthetic control, quasi-experimental designs) rather than pure hypothesis tests.

## What would count as solving it?

Any of the following, in increasing strength:

1. **Calibrated detector:** an algorithm with demonstrated Type-I error ≤ α (e.g., 0.05) on benign data exhibiting each confounder (real outage periods, migration events, large-pool races, attribution noise), and power ≥ some stated level on simulated/known attacks. Evaluated on the Qubic/Monero window plus injected attacks on benign chains.
2. **Causal decomposition:** a method that estimates the withholding-attributable fraction of observed orphan-rate anomalies with confidence intervals, showing confounder effects are separately identified (e.g., via timing of migration/outage events as instruments or controls).
3. **Negative/impossibility result:** a proof that withholding and, say, propagation races with a latency-correlated miner are statistically indistinguishable under realistic observation, with a quantified detection boundary — this would itself be a publishable resolution.

Since the type is *experimental*, the referee standard is: real and simulated data, controlled false positives, sensitivity analyses over attack parameters (withholding intensity, hash share), and the Qubic case as validation.

## How to start

1. **Read Lee & Kim (2026) first** — extract the empirical signatures of the actual campaign (fork timing, release patterns, attribution) and use them to define concrete H1 features; then read Li et al. (2024) to see exactly which statistics their tests use and where they break.
2. **Build a labeled simulation harness:** a network simulator (or modified Bitcoin/Monero regtest) where you can inject each confounder separately — outages (drop a miner's connectivity for Δt), migration (ramp hash share), races (vary latency between large pools), misattribution (randomly relabel ε of blocks) — plus attacks. Measure the joint distribution of candidate features under each condition; overlap of distributions defines the hardness frontier.
3. **Formulate as causal inference:** use known exogenous events (pool closures, documented outages, hashrate migrations as in Ethereum Merge-era or China's 2021 mining ban) as natural experiments to estimate the benign anomaly baseline, then test deviations against it (synthetic-control / difference-in-differences on fork statistics per pool).
4. **Main obstacle:** pool misattribution and the fact that a smart withholder can mimic a badly connected miner; attack the identification problem via *timing structure* (withholding produces deterministic release-after-honest-block patterns races don't).

## Key references

- Li, J., Campajola, C., Tessone, C. J. (2024). *Statistical detection of selfish mining in proof-of-work blockchain systems*. Scientific Reports, DOI:10.1038/s41598-024-55348-3 — develops statistical tests flagging anomalous fork structure indicative of selfish mining; the baseline this problem must go beyond. https://doi.org/10.1038/s41598-024-55348-3
- Lee, ?, Kim, ? (2026). *Inside Qubic's Selfish Mining Campaign on Monero: Evidence, Tactics, and Limits*. AFT 2026, DOI:10.4230/LIPIcs.AFT.2026.28 — forensic reconstruction of a real selfish-mining campaign, the closest thing to labeled ground-truth data. https://doi.org/10.4230/LIPIcs.AFT.2026.28 (verify author initials and content from the proceedings)
- Eyal, I., Sirer, E. G. (2014). *Majority is not Enough: Bitcoin Mining is Vulnerable*. FC 2014 — defines the selfish-mining strategy and its revenue model; needed to specify H1. (Not in census; standard background.)
- Sapirshtein, A., Sompolinsky, Y., Zohar, A. (2016). *Optimal Selfish Mining Strategies in Bitcoin*. FC 2016 — characterizes the full space of withholding policies, needed for power analysis over attack variants. (Background; verify details.)

## Warnings

- The census is terse; the exact confounder list (outages, migration, races, misattribution) is given but no formal model is specified — you must define H0 yourself and justify it.
- The 2026 Lee & Kim reference and its DOI are post-census-horizon; verify it exists and what it actually contains before relying on it.
- Don't evaluate on fork rate alone: orphan rate grows with pool size even honestly; any detector must condition on size and latency or it will flag every large pool.
- Pool attribution from coinbase tags is systematically biased (untagged blocks, rotating addresses); treat per-entity statistics as measurement-with-error.
- Beware simulation-only validation with a single network topology — results are notoriously topology-sensitive.

---
*`BCGT-000232` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Calibrated detector:** an algorithm with demonstrated Type-I error ≤ α (e.g., 0.05) on benign data exhibiting each confounder (real outage periods, migration events, large-pool races, attribution noise), and power ≥ some stated level on simulated/known attacks. Evaluated on the Qubic/Monero window plus injected attacks on benign chains.
2. **Causal decomposition:** a method that estimates the withholding-attributable fraction of observed orphan-rate anomalies with confidence intervals, showing confounder effects are separately identified (e.g., via timing of migration/outage events as instruments or controls).
3. **Negative/impossibility result:** a proof that withholding and, say, propagation races with a latency-correlated miner are statistically indistinguishable under realistic observation, with a quantified detection boundary — this would itself be a publishable resolution.

Since the type is *experimental*, the referee standard is: real and simulated data, controlled false positives, sensitivity analyses over attack parameters (withholding intensity, hash share), and the Qubic case as validation.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/ab59f199-e71d-4e84-8afa-f715d10ff33b?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000232.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000232)
