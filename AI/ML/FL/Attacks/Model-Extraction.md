# Model Extraction & IP Theft

## Overview

Model extraction aims to reproduce model functionality or steal intellectual property. In FL, exposure can occur via model snapshots, public APIs, or compromised clients that reveal behavior.

## Attack modes

- Query-based extraction: attacker queries a model (or accesses snapshots) to train a surrogate model.
- Snapshot theft: compromise server or storage to download model checkpoints.

## Defenses

- Access controls and authenticated APIs.
- Watermarking models and monitoring query behaviors.
- Rate-limiting and anomaly detection for model queries.

## Practical notes

- In federated settings, protect aggregated model checkpoints and restrict model access to vetted consumers.

## References (links)

- Tramer, F., et al., "Stealing Machine Learning Models via Prediction APIs," USENIX Security 2016 / arXiv:1609.02943. https://arxiv.org/abs/1609.02943
