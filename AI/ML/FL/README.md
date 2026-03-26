# Federated Learning

## Short Description

Federated Learning (FL) is a distributed machine learning paradigm where many clients collaboratively train a shared model under server orchestration while keeping raw training data on-device or in local silos. The server aggregates model updates rather than collecting source data, enabling privacy-aware learning under communication and systems constraints [2][3].

## Purpose and Scope

This document is a ready-to-commit reference for federated learning in general. It covers:

- problem setup and notation,
- canonical training loop,
- system/data challenges unique to FL,
- practical tuning and evaluation guidance,
- privacy and security mechanisms,
- common algorithm families and when to use them,
- a minimal experiment log template.

Where possible, claims are grounded in canonical sources and recent surveys [1][2][3].

## What Makes FL Different

Compared with centralized ML, FL is constrained by:

- decentralized data ownership and governance,
- partial client participation each round,
- heterogeneous hardware/network availability,
- strongly non-IID and unbalanced client data,
- strict communication budgets and failure handling.

These constraints are central in modern FL formulations and evaluations [2][3].

## Notation and Problem Setup

- Number of clients: $K$.
- Client $k$ has dataset $D_k$ with size $n_k$.
- Total examples: $n = \sum_{k=1}^{K} n_k$.
- Model parameters: $w \in \mathbb{R}^d$.
- Local objective on client $k$:

$$
F_k(w) = \frac{1}{n_k} \sum_{i \in D_k} \ell(w; x_i, y_i).
$$

- Global federated objective:

$$
f(w) = \sum_{k=1}^{K} \frac{n_k}{n} F_k(w).
$$

- Communication round index: $t = 0,1,\ldots,T-1$.
- Global model at round $t$: $w_t$.
- Sampled client subset: $S_t$ with $|S_t|=m_t$.
- Local return from client $k$: $w_k^{t+1}$.
- Client delta: $\Delta_k^t = w_k^{t+1} - w_t$.

This objective form is the standard baseline used across FL optimization work [1][3].

## Canonical FL Training Loop

At each server round:

1. Server samples participating clients $S_t$.
2. Server broadcasts global parameters $w_t$.
3. Each selected client performs local optimization on private data.
4. Server aggregates updates into a new global model.

A generic weighted aggregation form is:

$$
w_{t+1} = w_t + \sum_{k \in S_t} p_k^t \Delta_k^t,
$$

where $p_k^t \ge 0$ and $\sum_{k \in S_t} p_k^t = 1$.

For [FedAvg](FedAvg.md)-style weighting, $p_k^t = \frac{n_k}{\sum_{j \in S_t} n_j}$ [1].

## Pseudocode (Framework-Level)

```python
# Server
w = w0
for t in range(T):
	S = sample_available_clients(C)   # fraction or fixed count
	send_model(S, w)
	updates = []
	for k in S:
		wk, meta = client_update(k, w)
		updates.append((k, wk, meta))
	w = aggregate(updates, strategy="weighted")


# Client
def client_update(k, w_init):
	w = w_init
	for epoch in range(E):
		for batch in iterate_minibatches(D_k[k], B, shuffle=True):
			g = grad(loss, w, batch)
			w = w - eta * g
	return w, {"num_examples": len(D_k[k])}
```

## System Regimes

### Cross-Device FL

- Very large client populations.
- Small sampled fraction per round.
- Intermittent connectivity and high straggler/dropout rates.
- Typical for mobile/edge personalization use cases [1][2].

### Cross-Silo FL

- Smaller number of reliable organizations.
- More stable participation and larger per-client datasets.
- Strong governance, auditability, and policy constraints [2].

## Key Challenges and Practical Implications

| Challenge | Why it matters | Typical mitigation |
| --- | --- | --- |
| Statistical heterogeneity (non-IID) | Local models drift toward client-specific optima | reduce local steps, proximal/control-variate methods |
| Systems heterogeneity | Uneven compute/network causes stale or missing updates | deadline-aware sampling, partial-work support |
| Communication bottleneck | Rounds dominate wall-clock in many deployments | increase local work carefully, compression, better client sampling |
| Privacy leakage risk | Updates can reveal sensitive patterns | secure aggregation, DP, policy controls |
| Fairness across clients | Global average can under-serve minority clients | fairness-aware objectives and per-group evaluation |

Non-IID instability and client drift are explicitly analyzed in SCAFFOLD and related work [5]. Systems/statistical heterogeneity are central in FedProx and survey treatments [4][2].

## Key Hyperparameters and Effects

| Parameter | Typical range | Primary effect |
| --- | --- | --- |
| Client fraction $C$ | 0.01 to 0.2 (cross-device), higher in cross-silo | variance vs per-round cost |
| Local epochs $E$ | 1 to 10 | more local progress, more drift risk |
| Batch size $B$ | 16 to 256 | smaller batch gives noisier but more update steps |
| Local LR $\eta$ | task dependent | dominant stability knob |
| Server optimizer/LR | [FedAvg](FedAvg.md)/[FedAvgM](FedAvgM.md)/[FedAdam](FedAdam.md)/[FedYogi](FedYogi.md)/[FedAdagrad](FedAdagrad.md) | can improve robustness under heterogeneity |

Practical pattern: tune $\eta$ jointly with local work ($E$, $B$), then tune client fraction and server optimizer [3][6].

## Privacy and Security Stack

### Secure Aggregation

Secure aggregation allows the server to recover only aggregate sums (for example, summed model updates), not each client contribution [7].

### Differential Privacy (User-Level)

A common DP-FL pattern:

1. Clip each client update to norm $S$.
2. Aggregate clipped updates.
3. Add Gaussian noise calibrated to target privacy.
4. Track cumulative privacy loss with an accountant.

Adaptive clipping can reduce manual tuning burden in DP-FL [8].

## Common Algorithm Families

### [FedAvg](FedAvg.md) (Baseline)

Local SGD plus weighted model averaging; strong practical baseline [1].

### FedProx

Adds a proximal local objective term:

$$
\min_w \; F_k(w) + \frac{\mu}{2}\lVert w - w_t \rVert^2,
$$

which can stabilize training in heterogeneous settings [4].

### SCAFFOLD

Adds control variates to correct client drift:

$$
w \leftarrow w - \eta \left(\nabla \ell(w;B) - c_k + c\right),
$$

improving convergence under data heterogeneity [5].

### Adaptive FedOpt (FedAdagrad/FedAdam/FedYogi)

Uses adaptive server-side optimization and often improves tuning robustness and convergence behavior in practice [6].

### FedAvgM

Apply server-side momentum to the aggregated model update (see [FedAvgM](FedAvgM.md)). Server momentum can stabilize and accelerate convergence under sampling noise and heterogeneity with minimal protocol changes.

### FedAdam

Apply server-side Adam-style adaptive updates to the aggregated client updates (see [FedAdam](FedAdam.md)). FedAdam rescales coordinates by estimated RMS of past aggregates and often reduces sensitivity to learning-rate tuning.

### FedYogi

Apply server-side Yogi adaptive updates to the aggregated client updates (see [FedYogi](FedYogi.md)). FedYogi uses a sign-based second-moment update to prevent uncontrolled growth of the second moment and can improve long-run stability versus Adam.

### FedAdagrad

Apply server-side Adagrad updates to the aggregated client updates (see [FedAdagrad](FedAdagrad.md)). FedAdagrad scales steps by accumulated squared aggregates and helps when parameters require different effective learning rates.

### Security and Attacks

See the security overview for federated learning in [Attacks](Attacks/README.md) — taxonomy, examples, and mitigations for poisoning, inference, Byzantine, Sybil, communication, and infrastructure attacks.

## Comparison of Methods

The table below summarizes the key idea, recommended use-case, and trade-offs for the algorithms covered in this folder.

| Method | Main idea | When to use | Pros | Cons |
| --- | --- | --- | --- | --- |
| FedAvg | Local SGD + weighted averaging | Default baseline | Simple, communication-efficient | Sensitive to heterogeneity |
| FedAvgM | Server-side momentum on aggregated updates | Noisy/oscillatory FedAvg training | Smooths updates, faster convergence | May accumulate bias under skewed sampling |
| FedAdam | Server-side Adam on aggregated updates | Per-parameter scale differences | Adaptive coordinate-wise scaling, easier tuning | Requires optimizer state; can mask biased aggregates |
| FedYogi | Server-side Yogi (modified second moment) | When Adam's second moment grows too quickly | More stable long-run adaptation | More complex update rule |
| FedAdagrad | Server-side Adagrad (accumulator) | Parameters with different effective LRs | Simple adaptivity | Accumulator can shrink steps over long runs |
| FedProx | Proximal term in client objective | Strong client heterogeneity | Reduces client drift | Extra hyperparameter; may limit local progress |
| SCAFFOLD | Client/server control variates (correction) | Severe heterogeneity and client-state available | Reduces drift and variance | Requires client state and extra metadata |



## Evaluation Checklist

Always report:

- global metric vs round and vs wall-clock,
- number of rounds to target quality,
- participation statistics (eligible, sampled, successful clients),
- communication volume (uplink/downlink bytes),
- client distribution diagnostics (label skew, quantity skew),
- fairness slices (for example worst-group accuracy),
- privacy/security settings if enabled.

This aligns with best-practice guidance for federated optimization evaluation [2][3].

## Minimal Experiment Log Template

```text
# Experiment: Federated Learning on <dataset>
Setting: <cross-device | cross-silo>
Model: <architecture>
Clients: K=<>, sampled fraction C=<>
Local training: E=<>, B=<>, eta=<>
Server optimization: <FedAvg (see FedAvg.md) | FedAvgM | FedAdam | ...>
Privacy/Security: <none | secure aggregation | DP parameters>
System assumptions: <availability, drop rate, deadline>
Results:
- Round 100: metric=<>
- Round 300: metric=<>
- Time-to-target: <>
Notes:
- instability/drift observations
- best hyperparameters
- fairness and failure analysis
```

## Implementation Checklist for Production

- Robust client eligibility, sampling, and retry logic.
- Explicit handling of stragglers and dropouts.
- Secure transport and authenticated update channels.
- Secure aggregation for update confidentiality.
- Update validation, clipping, and anomaly monitoring.
- End-to-end observability for rounds, failures, and drift.
- Privacy accounting and governance documentation when using DP.

## References

[1] McMahan et al., Communication-Efficient Learning of Deep Networks from Decentralized Data, AISTATS 2017. https://proceedings.mlr.press/v54/mcmahan17a.html

[2] Kairouz et al., Advances and Open Problems in Federated Learning, Foundations and Trends in ML, 2021. https://arxiv.org/abs/1912.04977

[3] Wang et al., A Field Guide to Federated Optimization, 2021. https://arxiv.org/abs/2107.06917

[4] Li et al., Federated Optimization in Heterogeneous Networks (FedProx), MLSys 2020. https://arxiv.org/abs/1812.06127

[5] Karimireddy et al., SCAFFOLD: Stochastic Controlled Averaging for Federated Learning, ICML 2020. https://arxiv.org/abs/1910.06378

[6] Reddi et al., Adaptive Federated Optimization, ICLR 2021. https://arxiv.org/abs/2003.00295

[7] Bonawitz et al., Practical Secure Aggregation for Privacy Preserving Machine Learning, Cryptology ePrint 2017/281. https://eprint.iacr.org/2017/281

[8] Andrew et al., Differentially Private Learning with Adaptive Clipping, NeurIPS 2021. https://proceedings.neurips.cc/paper/2021/hash/91cff01af640a24e7f9f7a5ab407889f-Abstract.html
