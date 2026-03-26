# SCAFFOLD — Stochastic Controlled Averaging

## Short description

SCAFFOLD introduces control variates (correction terms) at clients and server to reduce client drift in federated learning. Each client maintains a local control variate $c_k$ and the server maintains a global control variate $c$. During local updates, gradients are corrected by subtracting the client's control variate and adding the server's control variate, which helps align local updates with the global objective.

## Notation

- Global model at round $t$: $w_t$.
- Server control variate: $c_t$.
- Client control variate for client $k$: $c_k^t$.
- Local gradient: $\nabla \ell(w; B)$.

## Local update rule with control variates

Each client performs corrected SGD steps:

$$
w \leftarrow w - \eta \left( \nabla \ell(w; B) - c_k^t + c_t \right).
$$

After local training, client $k$ updates its control variate (typically using the difference between local and server gradients or aggregated updates) and sends the model/metadata back to the server.

## Algorithm (high-level)

1. Server broadcasts $(w_t, c_t)$.
2. Each client initializes $w \leftarrow w_t$ and runs local SGD with control variate correction.
3. Client returns $w_k^{t+1}$ and a control-variates delta to server.
4. Server aggregates updates and updates global control variate $c_{t+1}$.

Exact update rules for $c_k$ and $c$ depend on implementation; see canonical pseudocode below for a minimal form.

## Pseudocode (concise)

```python
# Server
w = w0
c = 0
for t in range(T):
    S = sample_clients(C)
    send_model_and_control(S, w, c)
    collected = []
    delta_cs = []
    for k in S:
        wk, delta_ck = ClientUpdateScaffold(k, w, c, c_k[k], E, B, eta)
        collected.append((n_k[k], wk))
        delta_cs.append(delta_ck)
    total = sum(nk for nk, _ in collected)
    w = sum(nk * wk for nk, wk in collected) / total
    c = c + average(delta_cs)  # server control update (implementation-specific)


def ClientUpdateScaffold(k, w_init, c_server, c_k, E, B, eta):
    w = w_init
    for epoch in range(E):
        for batch in minibatches(D_k[k], B):
            g = grad(loss, w, batch)
            w = w - eta * (g - c_k + c_server)
    # compute delta for client control variate (simple form)
    delta_ck = (c_server - c_k) + (w - w_init) / (eta * local_steps)
    return w, delta_ck
```

## Key parameters and effects

- Corrective terms reduce client drift and improve convergence under severe heterogeneity.
- Requires storing and updating client control variates $c_k$ (stateful clients) unless stateless approximations are used.

## Convergence intuition and tuning

- The control variates act like variance-reduction terms, making local steps approximate global gradients more closely.
- SCAFFOLD typically converges faster in rounds when clients are heterogeneous, but introduces extra state and communication for control variates.

## When to use

- Use SCAFFOLD when client heterogeneity is large and you can maintain small client-side state (control variates).

## Caveats

- Stateful clients: requires storing `c_k` on client or server-managed mapping; brittle when clients drop or rotate frequently (cross-device).
- Extra communication: sending control-variates deltas increases per-round metadata.

## Minimal experiment log snippet

```text
Experiment: SCAFFOLD on <dataset>
Local: E=<>, B=<>, eta=<>
Results: rounds-to-target, effect on heterogeneity
```

## Implementation checklist for production

- Decide client-state strategy for `c_k`: persistent storage, reinitialization policy, or stateless approximations.
- Handle client dropout: fall back to default `c_k=0` or use server-side cached values.
- Monitor communication overhead and ensure control-variates updates are compact.

## Canonical citation

Karimireddy, S. P., Kale, S., Mohri, M., Reddi, S. J., Stich, S. U., & Suresh, A. T. (2020). SCAFFOLD: Stochastic Controlled Averaging for Federated Learning. ICML 2020.
