# Detection-as-Code (DaC) CI/CD Pipeline

[![CI/CD Validation](https://github.com/mariatdonadio/Detection-as-Code/actions/workflows/validate.yml/badge.svg)](https://github.com/mariatdonadio/Detection-as-Code/actions)
[![Deployment](https://github.com/mariatdonadio/Detection-as-Code/actions/workflows/deploy.yml/badge.svg)](https://github.com/mariatdonadio/Detection-as-Code/actions)

## Executive Summary
Modern Security Operations Centers (SOCs) suffer from alert fatigue due to the deployment of static, unvalidated detection rules. This repository demonstrates a robust **Detection-as-Code (DaC)** infrastructure that applies software engineering principles to threat intelligence. 

It ensures that no detection rule reaches production without automated syntax validation, unit testing against historical logs, and strict false-positive threshold checks via a CI/CD pipeline.

## Architecture and CI/CD Workflow
The infrastructure is orchestrated using GitHub Actions and the RunReveal CLI. The pipeline strictly enforces the following lifecycle:

1. **Syntax Linting:** Validates the structural integrity of all Sigma and SQL rules.
2. **Matrix Testing (Unit Tests):** Uses `jq` to dynamically parse `.github/detection-tests.json` and iterate through test cases.
3. **Exit Code Evaluation:** The Bash runtime explicitly evaluates expected exit codes to ensure the rule triggers on malicious data (`Exit Code 1`) and remains silent on benign data (`Exit Code 0`).
4. **Dry-Run Deployment:** Simulates the synchronization of validated rules to the detection engine.

## Threat Intelligence Applied
Instead of generic placeholders, this pipeline is tested against specific, high-fidelity threat vectors mapped to the **MITRE ATT&CK** framework.

### Active Rule: T1558.003 - Kerberoasting
* **Location:** `detections/sigma/windows/kerberoasting.yml`
* **Architectural Blindspot Addressed:** A naive rule searching for RC4 ticket requests (EventID 4769) generates massive noise in a legacy Active Directory environment. This rule is engineered with explicit logical exclusions to filter out machine accounts (ending in `$`) and the native `krbtgt` service, isolating true brute-force attempts on vulnerable user service accounts.

## Stress Testing Strategy
Testing the "happy path" is insufficient for detection engineering. The pipeline evaluates contradictory scenarios to guarantee a controlled false-positive rate:

* **True Positive Alert (`samples/kerberoast_positive.json`):** Simulates an RC4 ticket request for a standard SQL service account. The pipeline expects and validates an `Exit Code 1`.
* **True Negative Silence (`samples/kerberoast_negative.json`):** Simulates a noisy but benign request from a machine account (`PC-001$`). The pipeline ensures the rule's exclusion logic holds by expecting an `Exit Code 0`.

## Operations Guide (How to Contribute)
To introduce a new detection rule to this repository, you must fulfill the CI/CD requirements. Pushing a rule without its corresponding test data will intentionally break the build.

1. Add your Sigma/SQL rule to the appropriate directory under `detections/`.
2. Generate a positive JSON log sample (malicious) and a negative JSON log sample (benign) in the `samples/` directory.
3. Update `.github/detection-tests.json` to include both test cases with their respective `expected_exit_code`.
4. Commit and push to trigger the validation workflow.