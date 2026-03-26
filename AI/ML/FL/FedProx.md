# FedProx — Federated Proximal

## Short description

FedProx extends Federated Averaging by adding a proximal term to each client's local objective. The proximal term penalizes deviation from the current global model, which reduces client drift caused by heterogeneous local objectives and stabilizes convergence across clients with non-IID data.

## Notation (reused)

- Global model at round $t$: $w_t$.
- Client local objective: $F_k(w)$ (empirical risk on client $k$).
- Proximal coefficient: $\mu_{prox} \ge 0$ (often denoted $\mu$ in the paper).
- Client update after local training: $w_k^{t+1}$.

## Local objective with proximal term

Each client $k$ solves a regularized local problem during local updates:

$$
\min_w \, F_k(w) + \frac{\mu_{prox}}{2} \lVert w - w_t \rVert^2.
$$

This penalizes drifting too far from the server model $w_t$ during local optimization.

## Algorithm (high-level)

1. Server broadcasts $w_t$.
2. Each selected client $k$ initializes $w \leftarrow w_t$ and runs local optimization (SGD) on the proximal objective for $E$ epochs:

$$
w \leftarrow w - \eta \left( \nabla \ell(w; B) + \mu_{prox} (w - w_t) \right).
$$

3. Clients return $w_k^{t+1}$ and server aggregates by weighted average (as in FedAvg).

## Pseudocode (concise)

```python
# Server
w = w0
for t in range(T):
    S = sample_clients(C)
    send_model(S, w)
    collected = []
    for k in S:
        wk = ClientUpdateProx(k, w, E, B, eta, mu_prox)
        collected.append((n_k[k], wk))
    total = sum(nk for nk, _ in collected)
    w = sum(nk * wk for nk, wk in collected) / total


def ClientUpdateProx(k, w_init, E, B, eta, mu_prox):
    w = w_init
    for epoch in range(E):
        for batch in minibatches(D_k[k], B):
            g = grad(loss, w, batch)
            w = w - eta * (g + mu_prox * (w - w_init))
    return w
```

## Key parameters and effects

| Parameter | Typical values | Effect |
| --- | --- | --- |
| $\mu_{prox}$ | 0.001 — 1.0 | Controls strength of proximal penalty; larger values keep local models closer to global model |
| Local LR $\eta$ | task dependent | Interacts with $\mu_{prox}$; may reduce effective local step magnitude |
| Local epochs $E$ | 1 — 10 | More local epochs still useful but drift is reduced by proximal term |

Practical tip: start with small $\mu_{prox}$ (e.g., 1e-2) and increase if client drift causes divergence.

## Convergence intuition and tuning

- The proximal term regularizes local updates towards the global model, reducing divergence when clients have strongly different local minima.
- FedProx provides theoretical and empirical stabilization in heterogeneous settings; however, too-large $\mu_{prox}$ can under-utilize local data (clients make little progress).

## When to use

- Use FedProx when FedAvg exhibits slow or unstable convergence on non-IID data.

## Caveats

- Proximal regularization adds an extra hyperparameter to tune.
- For cross-silo settings with large per-client datasets, smaller $\mu_{prox}$ may suffice.

## Minimal experiment log snippet

```text
Experiment: FedProx on <dataset>
mu_prox=0.01, Local: E=<>, B=<>, eta=<>
Results: rounds-to-target, stability notes
```

## Implementation checklist for production

- Implement proximal gradient step efficiently on clients.
- Monitor per-client progress to ensure proximal term not preventing useful learning.
- Use secure aggregation and clipping as in FedAvg if needed.

## Canonical citation

Li, T., Sahu, A. K., Talwalkar, A., & Smith, V. (2020). Federated Optimization in Heterogeneous Networks (FedProx). MLSys 2020.
