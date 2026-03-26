# Communication & Infrastructure Attacks

## Overview

These attacks target the networking, storage, or orchestration layers of FL: intercepting updates, tampering with messages, replaying updates, or compromising servers.

## Attack types

- Eavesdropping / MITM: intercept model uploads/downloads if transport is insecure.
- Replay & tampering: replay previous updates or modify messages in transit.
- Availability attacks (DoS): flood coordinator or clients to disrupt rounds.
- Supply-chain compromise: alter server code or checkpoint storage to inject malicious behavior.

## Defenses

- TLS + mutual authentication, HSTS, and certificate management.
- Signed updates and integrity checks (MACs) to detect tampering.
- Redundant deployment and rate-limiting for DoS mitigation.
- Secure CI/CD and signed binaries to reduce supply-chain risk.

## References (links)

- Bonawitz, K., et al., "Practical Secure Aggregation for Privacy-Preserving Machine Learning," ePrint 2017/281. https://eprint.iacr.org/2017/281.pdf
## Operational notes

- Include forensic logging and cryptographic evidence for audits.
