# Attacks Against Defenses & Adaptive Poisoning

## Overview

Adaptive attackers tailor payloads to evade deployed defenses (robust aggregators, anomaly detectors, DP noise). These attacks are research-active and require careful threat modeling.

## Techniques

- Optimized stealth: solve constrained optimization to maximize attack objective while keeping anomaly scores below detection thresholds.
- DP-aware poisoning: craft updates that exploit the added DP noise to hide malicious signal.
- Aggregator-aware attacks: craft updates that survive robust rules (e.g., mimic median-like updates across many coordinates).

## Defenses and countermeasures

- Adversarial testing: evaluate defenses against realistic adaptive adversaries (red-team exercises).
- Ensemble defenses: combine multiple detectors and aggregators to widen attack surface for attacker and reduce single-point failures.
- Randomized defenses: randomize aggregation subsets or detector thresholds to make optimization harder for attackers.

## References (links)

- Carlini, N., et al., "A Wolf in Sheep's Clothing: Evading Defenses to Transferable Attacks on Image Classifiers" (example of adaptive attacks vs defenses). https://arxiv.org/abs/1905.00008
- Bagdasaryan, E., et al., "How To Backdoor Federated Learning," Proc. of PMLR 108 (AISTATS), 2020. https://proceedings.mlr.press/v108/bagdasaryan20a.html
## Practical guidance

- Resistance to adaptive attackers is limited; prefer layered defenses and operational controls (auditing, canary deployments).
