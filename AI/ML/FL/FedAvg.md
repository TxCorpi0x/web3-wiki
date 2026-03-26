# FedAvg - Federated Averaging

## Short Description

FedAvg (Federated Averaging) is a communication-efficient federated learning algorithm in which many clients perform multiple local SGD updates on private data and the server aggregates only the resulting model parameters by a weighted average. This trades local computation for far fewer communication rounds and is the canonical practical baseline for federated optimization.

## Notation and Problem Setup

- Global dataset size: $n$ (total examples).
- Clients: $K$ clients, client $k$ holds $n_k$ examples with $\sum_{k=1}^{K} n_k = n$.
- Local empirical loss on client $k$:

$$
F_k(w) = \frac{1}{n_k} \sum_{i \in D_k} \ell(w; x_i, y_i).
$$

- Global objective:

$$
f(w) = \frac{1}{n} \sum_{i=1}^{n} \ell(w; x_i, y_i) = \sum_{k=1}^{K} \frac{n_k}{n} F_k(w).
$$

- Round index: $t = 0, 1, \ldots, T-1$.
- Global model at round $t$: $w_t$.
- Client return after local training: $w_k^{t+1}$.
- Client delta: $\Delta_k^t = w_k^{t+1} - w_t$.
- Client sampling fraction: $C$ (or absolute sampled clients $m = \max(1, \lfloor CK \rfloor)$).
- Local epochs: $E$.
- Local minibatch size: $B$.
- Local learning rate: $\eta$.
- Approximate local steps per round for client $k$:

$$
u_k \approx E \cdot \frac{n_k}{B}.
$$

All algorithmic descriptions below follow the original FedAvg formulation.

## FedAvg Algorithm Formulation

Server round (for $t = 0, \ldots, T-1$):

1. Sample clients: choose subset $S_t$ of clients of size $m_t$ (fraction $C$).
2. Broadcast: send $w_t$ to each $k \in S_t$.
3. Local training: each selected client $k$ initializes $w \leftarrow w_t$ and runs local SGD for $E$ epochs with minibatch $B$:

$$
w \leftarrow w - \eta \, \nabla \ell(w; B)
$$

repeated for each minibatch $B$ in local passes, producing $w_k^{t+1}$.

4. Aggregate: server computes the weighted average:

$$
w_{t+1} = \frac{1}{\sum_{k \in S_t} n_k} \sum_{k \in S_t} n_k \, w_k^{t+1}.
$$

Equivalently:

$$
w_{t+1} = w_t + \frac{1}{\sum_{k \in S_t} n_k} \sum_{k \in S_t} n_k \, \Delta_k^t.
$$

### Remarks

- Weighted averaging by $n_k$ ensures clients contribute proportionally to their dataset sizes.
- FedAvg generalizes FedSGD: setting $E=1$ and $B=\infty$ (one full-data gradient per client) recovers a single-step-per-round baseline.

## Pseudocode (Concise)

```python
# Server
w = w0
for t in range(T):
	S = sample_clients(C)  # |S| = m_t
	send_w_to_clients(S, w)
	collected = []
	for k in S:
		wk = ClientUpdate(k, w, E, B, eta)
		collected.append((n_k[k], wk))
	total = sum(nk for nk, _ in collected)
	w = sum(nk * wk for nk, wk in collected) / total


# ClientUpdate
def ClientUpdate(k, w_init, E, B, eta):
	w = w_init
	for epoch in range(E):
		for batch in iterate_minibatches(D_k[k], B, shuffle=True):
			g = grad(loss, w, batch)
			w = w - eta * g
	return w
```

This pseudocode mirrors the algorithm and experiments described in the FedAvg paper.

## Key Parameters and Practical Effects

| Parameter | Typical values        | Primary effect                                                                                   |
| --------- | --------------------- | ------------------------------------------------------------------------------------------------ |
| $C$       | 0.01, 0.1, 0.5        | Sampling fraction; larger $C$ reduces aggregation variance but increases per-round communication |
| $E$       | 1, 5, 10              | Local epochs; larger $E$ reduces communication rounds but can cause client drift on non-IID data |
| $B$       | 16, 32, 128, $\infty$ | Minibatch size; smaller $B$ increases local updates per epoch                                    |
| $\eta$    | task dependent        | Local learning rate; tune jointly with $E$ and $B$                                               |

Effective local work per round:

$$
u \approx E \cdot \frac{\mathbb{E}[n_k]}{B}.
$$

Increasing $u$ (via larger $E$ or smaller $B$) is the main mechanism by which FedAvg reduces communication rounds. Empirical results show orders-of-magnitude reductions in rounds for many tasks when tuning $u$.

## Convergence Intuition and Practical Tuning

- Communication vs compute tradeoff: mobile clients often have cheap compute but expensive uplink; FedAvg trades extra local computation for fewer communication rounds.
- Non-IID sensitivity: on strongly non-IID partitions, large $E$ can cause client drift (local models move toward client-specific minima), slowing or destabilizing convergence.
- Remedies include smaller $\eta$, smaller $E$, or algorithmic corrections (see variants).

Heuristics:

- Start with $C \approx 0.1$.
- Choose $B$ to fit client memory; prefer reducing $B$ before increasing $E$ if client hardware allows.
- If training oscillates or diverges, reduce $\eta$ or $E$.
- Consider decaying local computation (reduce $E$ or increase $B$) late in training to stabilize fine-tuning.

## Common Extensions and Formulas

### FedProx

Add a proximal term to stabilize local updates:

$$
\min_w \; F_k(w) + \frac{\mu}{2} \lVert w - w_t \rVert^2,
$$

where $\mu > 0$ penalizes deviation from the global model.

### SCAFFOLD

Use control variates to correct client drift:

$$
w \leftarrow w - \eta \left(\nabla \ell(w; B) - c_k + c\right).
$$

### DP-FedAvg

- Clip each client update $\Delta_k$ to norm $S$.
- Aggregate and add Gaussian noise $\mathcal{N}(0, \sigma^2 S^2 I)$.
- Track privacy loss with an accountant.

### Secure Aggregation

Use a cryptographic protocol so the server obtains $\sum_k n_k w_k$ without seeing individual $w_k$.

## Minimal Experiment Log Template

```text
# Experiment: FedAvg on <dataset>
Model: <model architecture>
Clients: K=<total>, sample C=<0.1>
Local epochs E=<>, batch B=<>, local lr eta=<>
Aggregation: weighted by n_k
Privacy: <none | DP params>
Results:
- Round 50: val acc XX.X%
- Round 100: val acc YY.Y%
Notes: (observed divergence, tuning steps, best hyperparameters)
```

Record communication rounds to reach target accuracy and the effective $u$ value for reproducibility. Empirical FedAvg results show large speedups in rounds versus FedSGD for many tasks; include such tables in experiments for clarity.

## Implementation Checklist for Production

- Secure transport for model parameters.
- Secure aggregation if individual updates must remain hidden.
- Client selection policy handling availability and fairness.
- Update clipping and normalization to bound influence of large clients.
- Monitoring and logging for divergence, stragglers, and communication failures.
- Privacy accounting if using DP.

## Canonical Citation

McMahan B., Moore E., Ramage D., Hampson S., Aguera y Arcas B. Communication-Efficient Learning of Deep Networks from Decentralized Data (FedAvg). AISTATS 2017.
