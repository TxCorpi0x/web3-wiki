# Privacy & Inference Attacks

## Overview

These attacks aim to extract private information from model updates: membership of examples, properties of the dataset, or even reconstruct raw training data.

## Membership inference

- Goal: determine whether a particular example or user appears in training data.
- Mechanism in FL: analyze model updates across rounds; small clients or unique examples increase vulnerability.

## Property inference

- Goal: infer aggregate properties of a client's dataset (e.g., average age, presence of disease markers).

## Model inversion and gradient reconstruction

- Gradient inversion: reconstruct inputs by inverting gradients or updates (works best when batch sizes are small and model architecture yields informative gradients).
- Techniques include optimization-based reconstruction and analytic inversion for simple models.

## Defenses

- Differential privacy (user-level): add calibrated noise and clipping to bound leakage.
- Secure aggregation: hides individual updates from the server, reducing direct leakage risk.
- Larger batch sizes and local accumulations reduce granularity of information per update.

## Practical evaluation

- Run threat-modeled reconstructions in controlled tests to measure leakage.
- Measure privacy-utility trade-offs when enabling DP.

## References

- Melis et al., Exploiting Unintended Feature Leakage in Collaborative Learning.
- Gradient inversion papers (Zhu et al., 2019; subsequent improvements).

## References (links)

- Melis, L., et al., "Exploiting Unintended Feature Leakage in Collaborative Learning," arXiv:1805.04049 / IEEE S
tl; https://arxiv.org/abs/1805.04049
- Zhu, L., Liu, Z., Han, S., "Deep Leakage from Gradients", arXiv:1906.08935 / NeurIPS 2019. https://arxiv.org/abs/1906.08935
