# 🌱 NEB

**Mining that trains a neural network.** One mining attempt is one exact integer training step of a model that the chain owns, and the shares of each round become one step of that model.

## 🌱 One attempt

```
key    = BLAKE3(header)                          header: 80 bytes
X      = 16 samples of N = 60,000, picked by ChaCha8(key)
y1 = rshift(X W1^T, 5);  a1 = clip(y1, -8, 7);  y2 = clip(rshift(a1 W2^T, 5), -8, 7);  E2 = clip(y2 - T, -8, 7)
G2 = E2^T a1;  G1 = (clip(rshift(E2 W2, 3), -8, 7) where -8 <= y1 <= 7)^T X      int32, exact, clipped to +-256
ticket = BLAKE3(fold(G1, G2) || key)            block if ticket <= target, share if ticket <= 32 x target
```

One step of the model per block, from the shares of the previous round:

```
R <- R + sum of the clipped G;   D = clip(trunc(R / 2^SU), -1, 1);   W <- clip(W - D, -8, 7)
```

**Exact.** Weights are 4-bit integers and sums are int32, with no floating point. The C reference, the CUDA kernels and an independent Python oracle return the same digest, bit for bit, so every node rebuilds the same model from the blocks.

**The miner chooses no input.** The dataset is fixed by the chain and pinned by hash. Each share is clipped, so the influence of one miner on a step is bounded.

## ⚡ Numbers

| | |
|---|---|
| Model accuracy, MNIST, simulation, 96,000 shares | 96.0 %, and 95.7 % with the one-block delay of the protocol |
| Node software, 10 pools, 300 blocks | 89.9 %, the accuracy of a single pool |
| Verification | about 0.5 ms per attempt on one CPU core, 17 ms for a typical block |
| AVX-512 miner, one 16-core CPU | 1.3 to 2.0 million attempts/s |

Methods and raw outputs: [experiments](https://github.com/SilasEigen/neb/blob/main/docs/experiments.md).

## 📐 Read

**[Whitepaper](https://github.com/SilasEigen/neb/blob/main/docs/whitepaper.md)** | **[Proof of work and shares](https://github.com/SilasEigen/neb/blob/main/spec/shares.md)** | **[Node and chain](https://github.com/SilasEigen/neb/blob/main/spec/node.md)** | **[Network guide](https://github.com/SilasEigen/neb/blob/main/docs/testnet.md)**

🌱 Runs on a public test network with two relays. Code: [SilasEigen/neb](https://github.com/SilasEigen/neb) · Site and explorer: [neuralenginebyte.org](https://neuralenginebyte.org)
