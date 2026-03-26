# FedAdam — Federated Adam (Server-Side Adam)

## Short description

FedAdam adapts the Adam optimizer to the federated setting by applying Adam-style adaptive updates at the server to the aggregated client updates. Instead of a plain weighted average, the server maintains first and second moment estimates of the aggregated updates and uses them to scale and adapt the applied step. This often improves robustness to heterogeneity and reduces sensitivity to learning-rate tuning.

## Notation (reused)

- Global model at round $t$: $w_t$.
- Client return after local training: $w_k^{t+1}$.
- Client delta: $\Delta_k^t = w_k^{t+1} - w_t$.
- Aggregated (weighted) update: $g_t = \sum_{k\in S_t} p_k^t \Delta_k^t$ with $p_k^t = \frac{n_k}{\sum_{j\in S_t} n_j}$.
- Server first moment: $m_t$.
- Server second moment: $v_t$.
- Adam hyperparameters: $\beta_1,\beta_2\in[0,1)$, $\epsilon>0$, server step-size $\alpha_s$.

## Algorithm (server-side Adam)

Treat the aggregated update $g_t$ as the quantity to which Adam is applied. At each round:

1. Compute aggregated update $g_t$.
2. Update first and second moment estimates:

$$
m_{t+1} = \beta_1 m_t + (1-\beta_1) g_t,
$$

$$
v_{t+1} = \beta_2 v_t + (1-\beta_2) (g_t \odot g_t),
$$

Optionally apply bias-correction:

$$
\hat m_{t+1} = \frac{m_{t+1}}{1-\beta_1^{t+1}},\qquad \hat v_{t+1} = \frac{v_{t+1}}{1-\beta_2^{t+1}}.
$$

3. Update the global model:

$$
w_{t+1} = w_t + \alpha_s \frac{\hat m_{t+1}}{\sqrt{\hat v_{t+1}} + \epsilon}.
$$

Common defaults: $\beta_1=0.9$, $\beta_2=0.999$, $\epsilon=10^{-8}$. Typical server step-size $\alpha_s\in[0.1,1.0]$ depending on scale of aggregated updates.

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
    v = beta2 * v + (1 - beta2) * (g * g)
    m_hat = m / (1 - beta1 ** (t+1))
    v_hat = v / (1 - beta2 ** (t+1))
    w = w + alpha_s * m_hat / (np.sqrt(v_hat) + eps)
```

## Key parameters and practical effects

- $\beta_1,\beta_2$: momentum terms for first/second moments (default 0.9, 0.999).
- $\alpha_s$: server step-size; controls how strongly adaptive steps move the global model.
- $\epsilon$: numerical stability; also affects effective step scaling for very small second moments.

Practical guidance: adaptive scaling reduces sensitivity to raw magnitudes of $g_t$, making FedAdam easier to tune across tasks than plain FedAvg.

## Convergence intuition and tuning

- Adam rescales coordinates inversely by an estimated root-mean-square of past updates, which helps when different parameters have different effective learning rates across clients.
- Start with canonical Adam defaults and tune $\alpha_s$ if steps are too aggressive. If training is unstable, reduce $\alpha_s$ or increase $\epsilon$.
- Because Adam can adapt quickly, re-check local learning rates $\eta$ and local epochs $E$ after enabling FedAdam.

## When to use

- Use FedAdam when vanilla FedAvg is unstable or difficult to tune due to heterogeneous per-parameter scales.
- Particularly helpful in encoder/embedding layers or sparse models where per-coordinate scaling matters.

## Caveats

- Adam-style adaptivity can mask biased aggregates from skewed client sampling; monitor for bias and fairness effects.
- When combined with DP (noisy, clipped aggregates), second-moment estimates may be biased—re-tune $\beta_2$ and $\epsilon$ accordingly.

## Minimal experiment log snippet

```text
Experiment: FedAdam on <dataset>
Server: beta1=0.9, beta2=0.999, eps=1e-8, alpha_s=0.5
Local: E=<>, B=<>, eta=<>
Results: rounds-to-target, wall-clock, stability notes
```

## Implementation checklist for production

- Maintain stable accumulation of $m_t$ and $v_t$ across rounds; persist checkpointed optimizer state if restarting.
- Clip or normalize aggregated updates before applying Adam if client magnitudes vary widely.
- Use secure aggregation when needed; apply Adam on the aggregated value after secure decryption.

## Canonical citation

Reddi, S. J., Charles, Z., Suresh, A. T., et al. Adaptive Federated Optimization. ICLR 2021. (See FedAdam, FedYogi, FedAdagrad variants.)
