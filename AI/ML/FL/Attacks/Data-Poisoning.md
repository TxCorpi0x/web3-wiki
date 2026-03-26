# Data-level Poisoning Attacks

## Overview

Data-level poisoning attacks mean adversaries manipulate local training data on compromised clients to bias the global model. These attacks are often simple to implement and can be effective because FL relies on local gradient steps computed on client data.

Common goals:
- Untargeted poisoning: degrade overall model utility.
- Targeted poisoning / backdoors: cause misclassification for inputs containing a trigger while preserving clean accuracy.

## Attack types

1. Label-flipping
   - Attacker flips labels for a subset of local examples (e.g., class A→B).
   - Simple, stealthy when only a small fraction of clients or examples are poisoned.

2. Additive malicious examples
   - Attacker injects poisoned inputs (with possibly realistic features) that push model parameters toward attacker-desired behavior.

3. Backdoor / Trigger-based poisoning
   - Insert a small trigger (pixel patch, text token, pattern) into training examples and label them to a target class. At inference, inputs with the trigger are misclassified.
   - Key property: maintain overall clean accuracy to remain undetected.

4. Data augmentation / feature-targeted manipulation
   - Modify or craft inputs to bias specific features or representation dimensions.

## Attack mechanics

- Poisoned clients perform local training using manipulated data and send updates that reflect the poisoned objective.
- If poisoned updates are averaged without robust aggregation or clipping, their influence accumulates on the server.

## Evaluation metrics

- Clean accuracy degradation (untargeted).
- Backdoor success rate (triggered inputs → target label).
- Number of compromised clients and rounds-to-fail.

## Defenses

- Robust aggregation (median/trim/Krum/Bulyan) to reduce influence of outliers.
- Update clipping / norm bounds to limit per-client effect.
- Holdout validation on server-side or cross-validation to detect performance shifts.
- Data provenance and client vetting to reduce poisoning surface.
- Differential privacy (user-level) reduces per-client influence but can reduce utility and complicate anomaly detection.

## Practical notes

- Backdoor attacks are particularly concerning because they can remain stealthy across rounds.
- Use ensemble detectors: monitor per-client contribution norms, cosine similarity with previous updates, and validation drops.

## References and examples

- Bagdasaryan et al., How To Backdoor Federated Learning (2018).
- Training small-scale drills locally to evaluate defense sensitivity is recommended.

## References (links)

- Bagdasaryan, E., et al., "How To Backdoor Federated Learning," Proc. of PMLR 108 (AISTATS), 2020. https://proceedings.mlr.press/v108/bagdasaryan20a.html (PDF: http://proceedings.mlr.press/v108/bagdasaryan20a/bagdasaryan20a.pdf)
