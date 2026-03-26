# Byzantine and Sybil Attacks

## Overview

Byzantine attacks model arbitrary or malicious behavior by clients. Sybil attacks occur when an adversary controls many identities/clients to amplify influence.

## Attack modes

- Byzantine (arbitrary) behavior: send random, stale, or strategically malicious updates.
- Sybil: create many fake clients (often trivial in open systems) to change the effective fraction of attackers.

## Impact

- Break assumptions of aggregators that bound fraction of malicious clients.
- Amplify poisoning attacks or create effective denial-of-service by overwhelming honest clients.

## Defenses

- Authentication and attestation to curtail fake clients.
- Rate limits, per-entity quotas, and reputation systems.
- Robust aggregation algorithms with provable guarantees under bounded Byzantine fraction.
- Client partitioning and cross-checks: require diverse client sets per round.

## Deployment notes

- For cross-device scenarios (large population), Sybil resistance is crucial — require device attestation or identity vetting.
- For cross-silo (fewer clients), strict governance and client audits mitigate Byzantine risks.

## References

- Byzantine-resilient aggregation literature (Krum, Bulyan, coordinate median) and Sybil-resistance literature in distributed systems.

## References (links)

- Blanchard, P., et al., "Byzantine-Tolerant Machine Learning," arXiv:1703.02757. https://arxiv.org/abs/1703.02757
- El Mhamdi, E. M., et al., "The Hidden Vulnerability of Distributed Learning in Byzantium" (Bulyan), arXiv:1802.07927. https://arxiv.org/abs/1802.07927
