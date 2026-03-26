# Gradient Reconstruction & Model Inversion

## Overview

Gradient reconstruction attacks attempt to recover raw training inputs (or approximations) from gradients or model updates. These are particularly relevant in FL because clients send gradients/parameter deltas to the server.

## Attack techniques

- Optimization-based inversion: initialize a dummy input and optimize it so that its gradient matches the observed gradient.
- Analytic inversion: for simple linear layers or shallow models, closed-form reconstructions may be possible.
- Layer-wise exploitation: reconstruct sensitive layers first (e.g., embeddings) to increase fidelity.

## Factors affecting success

- Batch size: small batches (even size 1) leak more information.
- Model architecture: convolutional and overparameterized models leak structural cues.
- Gradient quantization and compression can reduce fidelity.

## Defenses

- Differential privacy (clipping + noise) reduces reconstruction fidelity.
- Larger batch sizes and client-side accumulation obscure individual samples.
- Gradient compression and sparsification can make inversion harder, but may not be sufficient.

## Practical guidance

- Treat small-batch, high-frequency updates as high-risk; combine DP and secure aggregation when possible.
- Use reconstruction tests during threat modeling to measure realistic leakage.

## References

- Zhu et al., Deep Leakage from Gradients (2019) and follow-up works.

## References (links)

- Zhu, L., Liu, Z., Han, S., "Deep Leakage from Gradients", arXiv:1906.08935 / NeurIPS 2019. https://arxiv.org/abs/1906.08935
