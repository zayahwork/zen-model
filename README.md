# ZEN Model — Reinforcement-Learned Fusion Plasma Control

**An AI that learns to control a simulated fusion reactor, trained on a world model built from real MAST tokamak data.**

Built by **Zayah Nelson** (19, self-taught) as an end-to-end machine-learning proof of concept: predict real plasma → respect physics → detect disruptions → learn the dynamics → control the plasma.

> **Honest scope up front:** This is a *proof of concept, in simulation, on one retired experimental reactor* (MAST, UK). The plasma dynamics are learned from real data; the instability and control layer are simplified. It demonstrates the *method* real fusion labs use — prediction plus reinforcement-learned control — but it is **not** a deployable reactor controller. The honesty is deliberate: overclaiming would make it less credible, not more.

---

## What it does

Fusion is how the sun makes energy. On Earth, the barrier is the **plasma** — superheated matter that wants to go unstable and crash (a "disruption"). ZEN Model is an AI that learns to keep that plasma hot (more energy) while steering it away from collapse.

Two operating modes emerged from training:

| Mode | Behavior | Survival | Avg temp |
|------|----------|----------|----------|
| **Safe** | Holds plasma stable the entire run | **200/200 (100%)** | ~470 eV |
| **Risk** | Pushes hotter for more energy, eventually disrupts | 132/200 | ~533 eV |

---

## Key results (all on held-out data / repeated runs)

**Plasma forecasting (Transformer):**
- **R² 0.836** predicting core temperature 5 steps ahead on **unseen shots**, using by-shot train/test splits to prevent leakage.
- Beat every baseline: persistence (0.770), LSTM (0.774), MLP (0.808), linear regression (0.810).
- **Honest finding:** adding plasma current and heating as extra inputs did *not* help — recent temperature already encodes their effect. Simpler won, tested 4+ ways.
- Forecast skill degrades gracefully with horizon: R² 0.92 (1 step) → 0.83 (5) → 0.76 (10) → 0.48 (50).

**Physics-informed model (PINN):**
- Adding a smoothness/continuity physics loss kept accuracy the same (0.835 vs 0.833) but produced **measurably smoother, more physically-consistent** forecasts (jumpiness −3.8%) — the real purpose of a PINN.

**Disruption detection:**
- A naive temperature-based detector flagged 95% of shots (broken). Switching to **plasma-current collapse** — the true disruption signature — gave clean labels.

**Control (PPO reinforcement learning inside a learned world model):**
- The trained forecaster became a "dream reactor"; a PPO controller trained inside it.
- **Traps caught and fixed honestly:** an early controller *exploited world-model errors* (driving temperature negative to fake survival); another *reward-hacked* (grabbing heat points then crashing). Both were diagnosed and fixed.
- Final controller: **the only strategy with 100% survival**, holding ~2× the temperature of safe-but-cold play. Verified against baselines (random, always-heat, always-calm all crash 49–96% of the time).

---

## How it was built (the pipeline)

1. **Real reactor data** — 326 shots from the MAST tokamak (temperature + current over time), pulled from the public archive. No synthetic data.
2. **Forecast the plasma** — LSTM then Transformer, validated only on unseen shots.
3. **Add physics** — a physics-informed loss for smoother, more physical predictions.
4. **Detect disruptions** — from plasma-current collapse (after catching a broken first attempt).
5. **Build a world model** — turn the forecaster into a learned "dream reactor."
6. **Learn to control it** — PPO reinforcement learning inside the dream, with reward/enviroment fixes to force genuine control.

---

## Repo contents

| File | What it is |
|------|-----------|
| `notebook.ipynb` | Clean, top-to-bottom notebook — the winning path only |
| `zen_model.html` | Interactive site visualizing the trained controller (open in a browser) |
| `requirements.txt` | Python dependencies |
| `README.md` | This file |

## Running it

```bash
pip install -r requirements.txt
jupyter notebook notebook.ipynb
```

The notebook downloads MAST data (public, no login), trains the models, and reproduces the results above. Open `zen_model.html` directly in any browser for the interactive visualization.

## Data & credits

- Data: **MAST tokamak** Level-2 shots via the public archive (`mastapp.site`), CC-BY-SA 4.0.
- Built with PyTorch, stable-baselines3, gymnasium, xarray.
- Methods (learned world models + RL control for tokamaks) are inspired by published fusion-ML research; this is an independent student-scale reimplementation, not affiliated with those groups.

---

*Zayah Nelson · ZEN Model · a self-taught proof of concept.*
