# Traffic Signal Control with Deep Q-Learning (DQN)

A reinforcement learning project that trains a DQN agent to adaptively control traffic signals at a simulated four-way intersection, using [SUMO](https://eclipse.dev/sumo/) for traffic simulation and [Stable Baselines3](https://stable-baselines3.readthedocs.io/) for the RL algorithm.

---

## The Problem

Traditional traffic signals run on fixed time cycles — green for 30 seconds, red for 30 seconds, regardless of how many cars are actually waiting. This is simple to implement but inherently wasteful: a busy lane might be forced to wait while an empty lane gets its turn.

The goal of this project is to replace that fixed logic with a learning agent that **observes the current state of the intersection and chooses which lanes to open** based on actual traffic demand.

---

## Approach

### Simulation Environment — SUMO

The intersection is a standard 4-way junction (North, South, East, West) modelled in SUMO (Simulation of Urban MObility). Each arm has 3 lanes — right, straight, and left turns. Traffic is generated dynamically across three demand patterns within each episode:

- **Morning phase** (0–150s): Higher North–South volume, lower East–West
- **Afternoon phase** (150–300s): Mixed, balanced demand
- **Evening phase** (300–450s): Shifted ratios

This forces the agent to generalise across different conditions rather than overfitting to a single pattern.

### Reinforcement Learning — DQN

The agent uses **Deep Q-Network (DQN)**, a classic RL algorithm that learns a Q-function mapping (state, action) → expected future reward. The agent:

1. **Observes** the current state — queue lengths and waiting times at each lane, provided by SUMO-RL
2. **Selects an action** — which lane group to open
3. **Receives a reward** — proportional to the negative total queue length (fewer waiting cars = better reward)
4. **Learns** by storing experiences in a replay buffer and updating the neural network via Q-learning

### Action Space Design

Rather than using the default 6 SUMO signal phases, we designed a custom **15-action space** giving the agent much finer control:

| Actions | Direction | Combinations |
|---|---|---|
| 0–6 | North–South | Right only, Straight only, Left only, + all pairwise combinations, + all three |
| 7–13 | East–West | Same 7 combinations |
| 14 | Safety | All-red transition |

Each action is mapped back to one of the 6 underlying SUMO phases via bitmask lookup. This lets the agent learn, for example, that during a right-turn surge it only needs to open the right lane — rather than being forced to choose between coarser all-NS or all-EW phases.

### Compatibility Layer

SUMO-RL returns multi-agent dict outputs. Since we're running single-agent, a `SB3CompatWrapper` flattens these dicts into NumPy arrays compatible with Stable Baselines3, and handles both old (4-tuple) and new (5-tuple) Gymnasium step formats.

---

## Results

After 20,000 training timesteps the agent shows clear improvement in episode reward (less negative = fewer vehicles queuing). During visual evaluation with the SUMO GUI:

- **Average vehicle wait time**: ~37 seconds
- **Max wait time observed**: ~273 seconds
- **Cumulative reward per episode**: ~-22 (compared to ~-158 in early training)

The reward trajectory during training improves steadily from around -158 to -163 by the final logged episodes, confirming the agent is learning a meaningful signal control policy.

---

## Project Structure

```
├── traffic_signal_dqn.ipynb   # Main notebook — training, evaluation, visual demo
├── intersection.net.xml        # SUMO road network (4-way intersection)
├── intersection.rou.xml        # SUMO vehicle routes (dynamic traffic demand)
├── dqn_intersection.zip        # Saved trained model (generated after running cell 2)
└── README.md
```

---

## How to Run

### Prerequisites

```bash
pip install gymnasium stable-baselines3 sumo-rl
```

You also need [SUMO installed](https://eclipse.dev/sumo/userdoc/Installing/) and the `SUMO_HOME` environment variable set.

### Steps

1. Clone the repository
2. Open `traffic_signal_dqn.ipynb` in Jupyter
3. Run **Cell 2** to train the agent (saves `dqn_intersection.zip`)
4. Run **Cell 3** for headless evaluation
5. Run **Cell 4** to open the SUMO GUI and watch the agent in action

---

## Tools & Libraries

| Tool | Purpose |
|---|---|
| [SUMO](https://eclipse.dev/sumo/) | Traffic microsimulation |
| [sumo-rl](https://github.com/LucasAlegre/sumo-rl) | Gymnasium wrapper for SUMO |
| [Stable Baselines3](https://stable-baselines3.readthedocs.io/) | DQN implementation |
| [Gymnasium](https://gymnasium.farama.org/) | RL environment interface |
| [TraCI](https://eclipse.dev/sumo/userdoc/TraCI/) | Real-time SUMO data access |

---

## What I Learned

This project gave me hands-on experience with the full reinforcement learning pipeline — from designing a custom action space and bridging incompatible APIs, to training a neural network agent and evaluating it against real simulation metrics. The biggest challenge was the wrapper engineering: getting SUMO-RL's multi-agent dict outputs into a shape that Stable Baselines3 could consume required careful handling of multiple API versions.

---

*Undergraduate project — ongoing. Contributions and feedback welcome.*
