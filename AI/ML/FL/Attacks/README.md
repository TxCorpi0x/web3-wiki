# Attacks on Federated Learning

## Short description

This document surveys attacks that can occur in federated learning (FL), organized by adversary goals and attack vectors, and summarizes common defenses. It is intended as an operational checklist for researchers and engineers designing FL systems.

## Threat model

- Adversary capabilities: a subset of clients may be malicious or compromised; the network or server may be partially compromised; adversary may observe model updates, participate in training, or control many fake clients (Sybil).
- Adversary goals: degrade global model (untargeted poisoning), insert targeted backdoors, extract private information (membership/property inference, reconstruction), steal model IP, or cause denial-of-service and supply-chain compromise.
- Assumptions vary: number of compromised clients, whether secure aggregation or DP is used, and whether clients carry persistent state.

## Taxonomy of attacks

1. Data-level poisoning
   - Label flipping: change labels on poisoned local data (e.g., map class A→B).
   - Additive/data perturbation: inject malicious training examples to bias the learned classifier.
   - Backdoor/data-filter attacks: insert triggers into training inputs so the model misbehaves on triggered inputs while preserving overall accuracy.

2. Model/update poisoning
   - Scaled/boosted updates: malicious client scales its update to dominate aggregation.
   - Sign-flipping and gradient manipulation: invert or modify update direction to hinder convergence.
   - Model replacement: craft an update that replaces the global model with an attacker-chosen model in one or few rounds.
   - Targeted backdoor via update manipulation: design updates that implant a backdoor while keeping validation metrics stable.

3. Byzantine and Sybil attacks
   - Byzantine failures: arbitrary (malicious) client behavior, including random, stale, or crafted updates.
   - Sybil attacks: adversary controls many fake clients to increase influence over aggregation.

4. Privacy and inference attacks
   - Membership inference: determine whether a specific example/user was in a client's dataset from model updates.
   - Property inference: recover aggregate properties of client data (e.g., demographic attributes).
   - Model inversion / gradient reconstruction: reconstruct training samples from gradients or model updates (gradient inversion attacks).
   - Label leakage: infer labels from intermediate updates, especially with small batch sizes.

5. Model extraction / IP theft
   - Query-based extraction: reproduce model behavior via API access (more common in centralized setting but relevant when model snapshots are exposed).

6. Communication and infrastructure attacks
   - Eavesdropping / MITM: intercept model uploads/downloads if transport is not secure.
   - Replay and tampering: replay old updates or modify in-flight messages.
   - Denial-of-service and availability attacks: flood coordinator or clients to disrupt rounds.
   - Supply-chain/server compromise: attacker gains access to the server or model storage and modifies global logic or checkpoints.

7. Incentive and free-riding attacks
   - Free-riders: clients avoid local work but attempt to collect rewards or benefits.
   - Rational adversaries: clients that strategically manipulate updates to maximize short-term gain.

8. Attacks against defenses
   - DP-robustness attacks: craft updates that exploit noise added by DP to stealthily poison models.
   - Adaptive poisoning: design payloads that evade anomaly detectors or robust aggregators.

---

## Per-attack deep-dives

See the detailed pages for each attack class:

- [Data-level Poisoning](Data-Poisoning.md)
- [Model / Update Poisoning](Model-Update-Poisoning.md)
- [Byzantine & Sybil](Byzantine-Sybil.md)
- [Privacy & Inference Attacks](Privacy-Inference-Attacks.md)
- [Gradient Reconstruction & Inversion](Gradient-Reconstruction.md)
- [Model Extraction & IP Theft](Model-Extraction.md)
- [Communication & Infrastructure Attacks](Comm-Infrastructure-Attacks.md)
- [Incentive, Free-riding & Rational Adversaries](Incentive-FreeRiding.md)
- [Attacks Against Defenses / Adaptive Poisoning](Defense-Evasion-Adaptive-Poisoning.md)

## Attack examples (brief)

- Label-flipping on image classification: flip a fraction of client labels for class 1→7 to bias global classifier.
- Backdoor triggered by pixel pattern: train clients to associate a small pixel patch with target label while retaining clean accuracy.
- Gradient inversion on small-batch update: reconstruct a client's image from gradient snapshots when B is small and model layer gradients are revealed.

## Defenses and mitigations

1. Secure transport & authentication
   - Use TLS, mutual authentication, and proper certificate management to prevent MITM and tampering.

2. Secure aggregation and privacy
   - Secure aggregation protocols (e.g., Bonawitz et al.) prevent the server from seeing individual updates while still allowing aggregate computation.
   - Differential privacy (user-level) limits information leaked by updates; combine with careful clipping and noise calibration.

3. Robust aggregation
   - Coordinate-wise median, trimmed mean, Krum, Bulyan: aggregators designed to reduce influence of Byzantine clients.
   - Limitations: many assume bounded fraction of malicious clients and can be sensitive to non-IID data.

4. Update clipping and normalization
   - Clip update norms, scale updates, or normalize per-client contributions to prevent scaling attacks.

5. Anomaly detection and validation
   - Hold-out validation on server-side datasets or cross-client validation to detect malicious updates.
   - Monitor update norms, cosine similarity, and per-round metrics; flag outliers for manual review or automated rejection.

6. Client authentication, attestation and reputation
   - Device attestation (TEEs), signed updates, and reputation systems to limit Sybil and supply-chain risks.

7. Stateful defenses and control variates
   - Methods like SCAFFOLD, client-reweighting, or adaptive clipping can reduce drift and make certain poisoning attacks harder.

8. Combining DP with robust aggregation
   - Carefully combine DP noise with robust aggregation and clipping; be aware that DP noise can reduce the effectiveness of anomaly detectors.

9. Operational controls
   - Rate limits, participant vetting, incremental rollout, canary models, model validation pipelines, and rollback mechanisms.

10. Certified and provable defenses (research frontier)
   - Certified robustness under bounded perturbations, formal verification for backdoors, and certified aggregation are active research areas.

## Defense trade-offs and deployment guidance

- Secure aggregation protects privacy but makes per-client anomaly detection harder (server cannot inspect individual updates).
- DP provides privacy guarantees but degrades utility and can mask poisoning; balance via tuned clipping and noise schedules.
- Robust aggregation reduces poisoning risk but may be brittle under extreme non-IID distributions; test aggregators on realistic client heterogeneity.
- Attestation and TEEs add hardware trust but increase deployment complexity and cost.

## Implementation checklist

- Secure transport (TLS) and authentication for all channels.
- Consider secure aggregation where strong privacy is required; benchmark effect on anomaly detection.
- Implement update clipping and per-client contribution limits.
- Add robust aggregation (median/trim/Krum/Bulyan) and test under synthetic poisoning scenarios.
- Monitor per-round statistics (norms, cosine similarity, validation loss) and alert on anomalies.
- Maintain a client registry, vetting, and reputation scoring to reduce Sybil risk.
- Provide canary/holdout datasets and automated rollback for suspicious model changes.
- Persist optimizer state, audit logs, and cryptographic evidence where relevant for forensic analysis.

## Further reading

- Bonawitz et al., Practical Secure Aggregation for Privacy Preserving Machine Learning (2017).
- Bagdasaryan et al., How To Backdoor Federated Learning (2018).
- Melis et al., Exploiting Unintended Feature Leakage in Collaborative Learning (2019).
- Truex et al., A Hybrid Approach to Privacy-Preserving Federated Learning (2020).

## Canonical links

- Bonawitz, K., et al., "Practical Secure Aggregation for Privacy-Preserving Machine Learning," ePrint 2017/281. https://eprint.iacr.org/2017/281.pdf
- Bagdasaryan, E., et al., "How To Backdoor Federated Learning," Proc. of PMLR 108 (AISTATS), 2020. https://proceedings.mlr.press/v108/bagdasaryan20a.html
- Melis, L., et al., "Exploiting Unintended Feature Leakage in Collaborative Learning," arXiv:1805.04049. https://arxiv.org/abs/1805.04049
- Zhu, L., Liu, Z., Han, S., "Deep Leakage from Gradients," arXiv:1906.08935 / NeurIPS 2019. https://arxiv.org/abs/1906.08935
- Truex, S., et al., "A Hybrid Approach to Privacy-Preserving Federated Learning," arXiv:1812.03224. https://arxiv.org/abs/1812.03224
