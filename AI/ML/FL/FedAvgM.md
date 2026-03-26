# FedAvgM — Federated Averaging with Server Momentum

## Short description

FedAvgM is a simple and practical variant of FedAvg that applies momentum on the server during aggregation. Instead of directly applying the weighted average of client updates to the global model, the server maintains and updates a velocity (momentum) and moves the global model along that velocity. This smooths noisy aggregated updates and often accelerates convergence in non-IID settings or when communication is infrequent.

## Notation (reused)

- Global model at round $t$: $w_t$.
- Client return after local training: $w_k^{t+1}$.
- Client delta: $\Delta_k^t = w_k^{t+1} - w_t$.
- Aggregated (weighted) update: $g_t = \sum_{k\in S_t} p_k^t \Delta_k^t$, where typically $p_k^t = \frac{n_k}{\sum_{j\in S_t} n_j}$.
- Server momentum coefficient: $\mu \in [0,1)$.
- Server learning rate (optional): $\alpha_s$ (often set to 1).

## Algorithm (server-side momentum)

Maintain a server velocity $v_t$ initialized to $0$. At round $t$:

1. Sample clients $S_t$ and broadcast $w_t$.
2. Each selected client performs local training and returns $w_k^{t+1}$.
3. Server computes the aggregated (weighted) update

$$
g_t = \sum_{k\in S_t} p_k^t \Delta_k^t,\qquad p_k^t=\frac{n_k}{\sum_{j\in S_t} n_j}.
$$

4. Update the server velocity and the global model with momentum:

$$
v_{t+1} = \mu v_t + g_t,
$$
$$
w_{t+1} = w_t + \alpha_s v_{t+1}.
$$

Interpretation:
- $v_t$ accumulates past aggregated updates and acts like Polyak/SGD momentum on the server.
- $\alpha_s$ is a server step-size (often 1). Choosing $\alpha_s<1$ is equivalent to dampening the applied momentum.

Common defaults used in practice: $\alpha_s=1$, $\mu\in[0.8,0.99]$.

## Pseudocode (concise)

```python
# Server
w = w0
v = 0
for t in range(T):
    S = sample_clients(C)
    send_model(S, w)
    collected = []
    for k in S:
        wk = ClientUpdate(k, w, E, B, eta)
        collected.append((n_k[k], wk))
    total = sum(nk for nk, _ in collected)
    g = sum(nk * (wk - w) for nk, wk in collected) / total  # weighted avg delta
    v = mu * v + g
    w = w + alpha_s * v
```

## Key parameters and practical effects

| Parameter | Typical values | Primary effect |
| --- | --- | --- |
| $\mu$ (server momentum) | 0.8 — 0.99 | Smooths aggregated updates; higher values bias updates toward historical directions |
| $\alpha_s$ (server step) | 0.1 — 1.0 | Scales how strongly the velocity affects the global model |
| Client fraction $C$ | 0.01 — 0.5 | As in FedAvg; affects aggregation variance |
| Local epochs $E$ | 1 — 10 | More local progress per round, interacts with momentum magnitude |
| Local LR $\eta$ | task dependent | Tuning interacts with server momentum; often reduced when using high $\mu$ |

Practical note: momentum reduces per-round variance but may amplify a persistent bias introduced by uneven client sampling; pair with clipping/normalization if client magnitudes vary.

## Convergence intuition and practical tuning

- Momentum filters high-frequency variance in aggregated updates, helping with noisy client sampling and heterogeneous data.
- For cross-device FL with small sampled fractions, server momentum often stabilizes learning and reduces oscillations due to sampling noise.
- Tuning guidance:
    - Start with $\mu=0.9$, $\alpha_s=1$. If training overshoots or drifts, reduce $\alpha_s$ or $\mu$.
    - When increasing $\mu$, consider reducing local learning rate $\eta$ or local epochs $E$ to avoid compounding aggressive local steps.
    - Combine with per-round normalization of $g_t$ (divide by its norm) if some rounds have unusually large aggregated updates.

Empirical behavior: momentum can reduce rounds-to-target in many workloads, but benefits depend on sampling regime and heterogeneity.

## Practical effects and tuning

- Momentum smooths noisy aggregated updates and can reduce oscillation from heterogeneity.
- Typical server momentum $\mu$: 0.8–0.99. Higher values bias more toward past directions.
- `alpha_s` (server step) is useful when aggregated updates are large; values $\le 1$ are common.
- When using momentum, re-tune local learning rate $\eta$ and local epochs $E$ as interactions change.

## When to use

- Useful when FedAvg shows slow or oscillatory convergence due to non-IID data or aggressive local work.
- Lightweight: requires only maintaining a server-side velocity and no extra client state.

## Caveats

- Momentum can accumulate biased directions if client sampling is highly skewed; monitor for unintended drift.
- Combine with clipping or normalization of client deltas if client magnitudes vary widely.

## Minimal experiment log snippet

```text
Experiment: FedAvgM on <dataset>
Server momentum mu=<0.9>, alpha_s=<1.0>
Local: E=<>, B=<>, eta=<>
Results: rounds-to-target, wall-clock, stability notes
```

## Further reading

- FedAvgM is a practical server-side heuristic used in several federated optimization studies and engineering implementations. Consider comparing with server optimizers (FedAdam, FedYogi) when tuning robustness under heterogeneity.

## Common extensions and interplay with other methods

- FedProx / proximal terms: combining server momentum with local proximal regularizers can help reduce drift while preserving momentum benefits.
- SCAFFOLD: control variates address client drift at the client level; server momentum is complementary and can be used together in some designs.
- DP-FedAvg (differential privacy): when clipping client deltas and adding noise for DP, apply momentum to the noisy, clipped aggregates — but account for the bias and increased variance introduced by DP noise.
- Secure aggregation: FedAvgM is compatible with secure aggregation; compute the aggregated sum inside the secure protocol, then update server velocity normally.

## Implementation checklist for production

- Secure transport of model parameters and authentication.
- Secure aggregation when needed to hide individual client contributions.
- Clipping and/or normalization of client deltas before aggregation to bound influence.
- Monitor velocity norm `||v_t||` and aggregated update norms to detect runaway momentum.
- Re-tune local learning rates and epochs when enabling server momentum.
- Logging of participation statistics and per-round diagnostics (norms, cosine similarity between successive `g_t`).

## Canonical citation

McMahan B., Moore E., Ramage D., Hampson S., Aguera y Arcas B. Communication-Efficient Learning of Deep Networks from Decentralized Data (FedAvg). AISTATS 2017.
