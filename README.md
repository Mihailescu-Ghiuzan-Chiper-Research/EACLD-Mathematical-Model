# EACLD-E0: reproducible active-diagnosis experiment

**MDPI Information – Evidence-Adaptive Cross-Layer Diagnosis (EACLD)**  
**Research artifact:** differential diagnosis of sensor faults and telemetry attacks (EACLD-E0)  
**Date:** 4 November 2026

This package implements the first computational version of the research specification *Specificația științifică a metodei EACLD – experimentul E0*, reported in the MDPI Information research manuscript.

The core scientific object is a finite-state sequential decision algorithm that chooses diagnostic observations based on Bayes risk reduction, action costs, and latent shared trust-domain failures. Its intervention-based simulation distinguishes physical changes, sensor faults, post-authentication tampering, and replay.

## Experiment design

There are four causal hypotheses (`H1`–`H4`), two network modes (`nominal`, `degraded`) and four diagnostic actions. Each intervention block contains the four causes with the same **P0 passive temperature trajectory**. P1 contains whatever security verification was already performed by background monitoring. A probe already observed successfully in P1 is not invoked again. A passive verification reported `unavailable` can be retried actively.

The experiment generates 640 training episodes, 160 validation episodes arranged in 40 paired blocks, and 160 primary test episodes grouped into 40 blocks. Model emission probabilities are fitted using the *training* interventions, not copied from the generating configuration. Only the evaluation process accesses causal labels on validation/test; policy engines receive the public telemetry and probe responses requested explicitly through an environment interface.

**Interpretation:** this is a controlled *synthetic experiment*, not field data. Physical probes and latency metrics are simulated. No cryptographic forgery is implemented: `H3` is a simulated change after a source authenticator is produced; if the verifier is healthy it correctly reports invalidity. `H4` reuses a valid old frame and is flagged by freshness checks when those checks function correctly.

## Methods and outputs

The seven comparison policies are B0 (P0 only), B1 (P1 only), B2 (all nonredundant checks), B3 (random selection), B4 (greedy entropy per cost), B5 (nonmyopic decision optimization with independent-evidence approximation) and B6 (EACLD with a common latent verifier-health variable). B5 and B6 use Bellman optimization for the four-probe finite horizon; B4 is the deliberately myopic information-theoretic comparator.

The decision space includes exact labels, two two-cause classes and an unresolved verdict. Decisions minimize a declared asymmetric loss; incomplete causal attribution is not silently counted as a correct exact diagnosis. Main metrics are exact macro-F1, exact coverage, realized decision loss, decision loss plus normalized probe cost, probe count, latency, bytes and multiclass Brier score.

`RAPORT_REZULTATE_E0.md` contains the complete comparison tables, matched-block 95% bootstrap intervals, findings, limitations and commands. Reproducing the experiments generates the `results/` output and the corresponding `results/`, `data/` and `figures/` folders under `sensitivity/`; these generated output directories are not committed to this repository.

## Reproduction

From this directory:

```bash
python -m pip install -r requirements.txt
python -m pytest -q tests
python run_experiment.py --config config/e0_config.json --out .
python run_experiment.py --config config/e0_stress_verifier_only.json --out sensitivity/verifier_prevalence_only
python run_experiment.py --config config/e0_stress_config.json --out sensitivity/shared_verifier_stress
python make_report.py
```

The command output is deterministic for fixed library versions and seeds; every policy consumes the same pre-sampled potential outcome for a requested probe on the same episode. The joint state is inferred from permitted observations; exact causal truth does not appear in public episode identifiers or policy input. The `data/private/` files are retained solely for calibration/evaluation and are not supplied to any policy decision routine.

## File layout

The source code and configurations below are tracked in Git; the output directories are created by experiment execution.

- `config/`: complete deterministic experiment configurations.
- `eacld/core.py`: finite-state Bayesian inference, loss function and exact Bellman planner.
- `eacld/simulator.py`: matched intervention blocks, data generation, supervised probability calibration.
- `eacld/evaluation.py`: metrics and stratified paired bootstrap.
- `run_experiment.py`: end-to-end experiment and plots.
- `make_report.py`: rebuild the research report from saved CSV files.
- `tests/`: automated property and functional tests.
- `data/public/`: information exposed to inference.
- `data/private/`: controlled intervention truth and potential probe responses.
- `results/`: fitted parameters, model manifests, per-episode results, decision traces, intervals and summaries.
- `figures/`: six publication-oriented diagnostic figures.
- `sensitivity/`: isolated shared-verifier prevalence change and compound-stress scenario.

## Method boundaries

The prototype assumes one dominant cause per episode and a static latent verifier state per episode. Probe outcomes are conditionally independent given the cause, verifier health and network mode; this model does **not** assume unconditional independence of security evidence. Exact lookahead is appropriate to four probes; a larger probe catalog requires bounded search, approximate inference or action pruning. The separate five-VM distributed experimental evaluation is documented in the EACLD-Testbed repository.
