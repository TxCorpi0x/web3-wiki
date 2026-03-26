# FedYogi — Federated Yogi (Server-Side Yogi)

## Short description

FedYogi adapts the Yogi optimizer to federated learning by applying Yogi-style second-moment updates at the server to the aggregated client updates. Yogi modifies the second-moment estimator to prevent uncontrolled growth and improve adaptation in non-stationary settings; this can lead to improved stability over Adam in some federated regimes.

## Notation (reused)

- Global model at round $t$: $w_t$.
- Client return after local training: $w_k^{t+1}$.
- Client delta: $\Delta_k^t = w_k^{t+1} - w_t$.
- Aggregated (weighted) update: $g_t = \sum_{k\in S_t} p_k^t \Delta_k^t$ with $p_k^t = \frac{n_k}{\sum_{j\in S_t} n_j}$.
- Server first moment: $m_t$.
- Server Yogi second-moment: $v_t$.
- Hyperparameters: $\beta_1,\beta_2\in[0,1)$, $\epsilon>0$, server step-size $\alpha_s$.

## Algorithm (server-side Yogi)

Yogi updates the second moment to shrink when new squared-gradients are smaller, avoiding runaway accumulation. At each round:

1. Compute aggregated update $g_t$.
2. Update first moment (same as Adam):

$$
m_{t+1} = \beta_1 m_t + (1-\beta_1) g_t.
$$

3. Update Yogi second moment using an element-wise rule:

$$
v_{t+1} = v_t + (1-\beta_2) \; g_t^2 \odot \mathrm{sign}(g_t^2 - v_t).
$$

This rule increases $v_t$ when $g_t^2>v_t$ and decreases it when $g_t^2<v_t$, controlling growth.

4. (Optional bias-correction) compute $\hat m_{t+1}$ and $\hat v_{t+1}$ analogously to Adam.

5. Update global model:

$$
w_{t+1} = w_t + \alpha_s \frac{\hat m_{t+1}}{\sqrt{\hat v_{t+1}} + \epsilon}.
$$

Defaults: $\beta_1=0.9$, $\beta_2=0.999$, $\epsilon=10^{-8}$; tune $\alpha_s$ per task.

## Pseudocode (concise)

```python
# Server
w = w0
m = 0
v = 0
for t in range(T):
    S = sample_clients(C)
    send_model(S, w)
    collected = []
    for k in S:
        wk = ClientUpdate(k, w, E, B, eta)
        collected.append((n_k[k], wk))
    total = sum(nk for nk, _ in collected)
    g = sum(nk * (wk - w) for nk, wk in collected) / total

    m = beta1 * m + (1 - beta1) * g
    v = v + (1 - beta2) * (g * g) * np.sign(g * g - v)
    m_hat = m / (1 - beta1 ** (t+1))
    v_hat = v  # Yogi typically uses uncorrected v or an appropriate correction if desired
    w = w + alpha_s * m_hat / (np.sqrt(v_hat) + eps)
```

## Key parameters and practical effects

- Yogi's second-moment update prevents excessive growth in $v_t$, often improving long-term stability compared to Adam.
- Useful when aggregated updates vary in scale across rounds or when Adam's v accumulates too quickly.

## Convergence intuition and tuning

- The sign-based adaptation makes the second moment responsive but bounded, which helps in non-stationary federated regimes.
- Start with Yogi defaults and tune $\alpha_s$ and $\beta_2$ if adaptation is too slow or too aggressive.

## When to use

- Prefer FedYogi over FedAdam when you observe that Adam's second moment increases without sufficient decay, or when long-run stability is a concern.

## Caveats

- The sign-based update is element-wise and must be implemented robustly for sparse tensors.
- As with other adaptive methods, check for fairness and bias when client sampling is skewed.

## Minimal experiment log snippet

```text
Experiment: FedYogi on <dataset>
Server: beta1=0.9, beta2=0.999, eps=1e-8, alpha_s=0.5
Local: E=<>, B=<>, eta=<>
Results: rounds-to-target, wall-clock, stability notes
```

## Implementation checklist for production

- Persist `m_t` and `v_t` across restarts; ensure numerical stability for `v_t` updates.
- Clip or normalize aggregated updates before adaptation if client magnitudes vary widely.
- Use secure aggregation; apply Yogi to the decrypted aggregate.

## Canonical citation

Reddi, S. J., Charles, Z., Suresh, A. T., et al. Adaptive Federated Optimization. ICLR 2021.
