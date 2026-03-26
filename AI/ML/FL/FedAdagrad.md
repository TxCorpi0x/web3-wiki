# FedAdagrad — Federated Adagrad (Server-Side Adagrad)

## Short description

FedAdagrad adapts the Adagrad optimizer to the federated setting by maintaining a per-coordinate accumulator of squared aggregated updates at the server and scaling applied steps accordingly. It is one of the adaptive federated optimizers introduced in Reddi et al. (Adaptive Federated Optimization) and can help when parameter-wise learning-rate differences exist across clients.

## Notation (reused)

- Global model at round $t$: $w_t$.
- Client delta: $\Delta_k^t = w_k^{t+1} - w_t$.
- Aggregated update: $g_t = \sum_{k\in S_t} p_k^t \Delta_k^t$.
- Accumulator: $s_t$ (element-wise sum of squared aggregated updates).
- Server step-size: $\alpha_s$, numerical stability $\epsilon$.

## Algorithm (server-side Adagrad)

At each round:

1. Compute aggregated update $g_t$.
2. Update accumulator:

$$
s_{t+1} = s_t + g_t \odot g_t.
$$

3. Update the global model:

$$
w_{t+1} = w_t + \alpha_s \frac{g_t}{\sqrt{s_{t+1}} + \epsilon}.
$$

Typical choices: small $\epsilon$ like $10^{-8}$ and $\alpha_s$ tuned per task.

## Pseudocode (concise)

```python
# Server
w = w0
s = 0
for t in range(T):
    S = sample_clients(C)
    send_model(S, w)
    collected = []
    for k in S:
        wk = ClientUpdate(k, w, E, B, eta)
        collected.append((n_k[k], wk))
    total = sum(nk for nk, _ in collected)
    g = sum(nk * (wk - w) for nk, wk in collected) / total

    s = s + g * g
    w = w + alpha_s * g / (np.sqrt(s) + eps)
```

## Key parameters and practical effects

- `s_t` accumulates squared updates so coordinates with large historical updates get smaller future steps.
- FedAdagrad can stabilize learning when some parameters require much smaller effective learning rates.

## Convergence intuition and tuning

- Adagrad adapts per-coordinate step-sizes using historical squared gradients; in federated settings this helps handle heterogeneous parameter scales.
- As `s_t` grows, effective steps shrink; monitor for overly small steps and consider resetting or using variant accumulators (e.g., RMSProp-style decay) if necessary.

## When to use

- Use FedAdagrad when per-parameter scales differ and plain FedAvg is hard to tune.

## Caveats

- Unbounded accumulation can lead to vanishing effective steps in very long runs; consider FedAdam/FedYogi or decayed accumulators for long experiments.

## Minimal experiment log snippet

```text
Experiment: FedAdagrad on <dataset>
alpha_s=<>, eps=1e-8
Local: E=<>, B=<>, eta=<>
Results: rounds-to-target, step-scaling behavior
```

## Implementation checklist for production

- Persist `s_t` and ensure numerical stability when taking square roots.
- Consider accumulator reset schedules or decay if steps shrink excessively.
- Use secure aggregation and clipping as standard when needed.

## Canonical citation

Reddi, S. J., Charles, Z., Suresh, A. T., et al. Adaptive Federated Optimization. ICLR 2021.
