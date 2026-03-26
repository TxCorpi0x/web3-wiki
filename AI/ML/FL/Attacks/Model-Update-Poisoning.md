# Model / Update Poisoning Attacks

## Overview

Model-update poisoning refers to direct manipulation of the model update (gradient or parameter delta) sent by a malicious client. Compared with data-level attacks, attackers can craft precise updates to maximize effect and evade simple detectors.

## Attack vectors

1. Scaled / boosted updates
   - Attacker multiplies its update by a large scalar to dominate the average.

2. Sign-flip and gradient manipulation
   - Invert sign of gradients or craft updates orthogonal to true descent direction.

3. Model replacement
   - Attacker crafts an update that replaces the server model with an attacker-chosen model in a small number of rounds.

4. Targeted backdoor via crafted updates
   - Instead of poisoning training data, craft updates that guide parameters to produce a backdoor while keeping global validation stable.

5. Stealthy low-norm perturbations
   - Optimize small-norm updates that produce large behavioral changes at targeted inputs.

## Attack mechanics and optimization

- Attackers can solve an inner optimization (e.g., minimize distance to desired model under norm/stealth constraint) to produce updates.
- With knowledge of aggregation rules, attackers can craft updates that survive robust aggregators or exploit gaps (e.g., Krum assumes limited fraction of Byzantine clients).

## Detection and defenses

- Update clipping and per-client contribution limits deny scaled-update attacks.
- Robust aggregation rules (median, trimmed mean, Krum, Bulyan) reduce effect of outliers — evaluate under non-IID data.
- Validation and holdout: evaluate candidate updates on holdout dataset (if available) before applying them.
- Model snapshot diffs and per-parameter anomaly detection (e.g., sudden shift in coordinates) can flag malicious updates.

## Practical trade-offs

- Robust aggregators may discard helpful but legitimately large updates; tune thresholds carefully.
- Combining secure aggregation and robust detection is non-trivial because secure aggregation hides individual updates; consider hybrid approaches (e.g., selective reveal under attestation).

## References

- Blanchard et al., Byzantine-tolerant aggregation (Krum) and follow-ups.
- Recent work on model replacement and targeted backdoors explores optimization formulations for stealthy updates.

## References (links)

- Blanchard, P., et al., "Byzantine-Tolerant Machine Learning," arXiv:1703.02757. https://arxiv.org/abs/1703.02757
- Bagdasaryan, E., et al., "How To Backdoor Federated Learning," Proc. of PMLR 108 (AISTATS), 2020. https://proceedings.mlr.press/v108/bagdasaryan20a.html
