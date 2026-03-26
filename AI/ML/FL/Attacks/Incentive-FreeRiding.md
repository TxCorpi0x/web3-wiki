# Incentive, Free-riding, and Rational Adversaries

## Overview

Incentive attacks focus on economic or participation incentives: adversaries try to gain rewards without contributing, or act strategically to gain advantage.

## Attack modes

- Free-riding: clients attempt to avoid local computation (submit stale or random updates) but claim credit.
- Rational manipulation: clients manipulate updates to maximize short-term rewards (e.g., reputation systems) at cost to global utility.

## Defenses

- Verifiable computation or lightweight checks to ensure clients perform work (e.g., randomized audits, challenge-response).
- Reputation systems that weight clients by historical helpfulness.
- Economic design: align rewards to long-term contribution (escrow, delayed payouts).

## Practical notes

- Verification must be lightweight to avoid defeating FL's efficiency benefits.

## References (links)

- Kairouz, P., et al., "Advances and Open Problems in Federated Learning," Foundations and Trends in Machine Learning, 2021 (survey including incentives). https://arxiv.org/abs/1912.04977
