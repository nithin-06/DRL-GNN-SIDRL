# DRL+GNN Reproduction: Project Summary

## Objective

Reproduce the work of Almasan et al., **"Deep Reinforcement Learning meets Graph Neural Networks: Exploring a Routing Optimization Use Case"** (*Computer Communications*, 2022).

The project focuses on reproducing a **GNN + DQN agent** that learns to route network traffic. The original paper trains the model on a single topology, **NSFNet**, and claims that the learned policy can generalize to previously unseen network topologies.

---

## Setup

| Component             | Details                                                                   |
| --------------------- | ------------------------------------------------------------------------- |
| Original work         | Almasan et al., *Deep Reinforcement Learning meets Graph Neural Networks* |
| Official repository   | DRL-GNN                                                                   |
| Environment           | Windows                                                                   |
| Hardware              | CPU only                                                                  |
| GPU required          | No                                                                        |
| Framework             | TensorFlow                                                                |
| Original dependencies | 2022-era versions                                                         |
| Current dependencies  | Modernized for current Python/TensorFlow                                  |
| Training topology     | NSFNet                                                                    |
| Training iterations   | 1,000                                                                     |

The original repository was modernized to run with current Python and TensorFlow versions.

During the reproduction, the shipped repository was also compared against the hyperparameters stated in the paper. Several discrepancies were identified and corrected before conducting the main experiments.

---

# Experiments

## 1. Baseline Feasibility Run

The first experiment was performed to determine whether the full reproduction was computationally feasible on a laptop without a GPU.

### Results

* Average training time: **~6.8 seconds per iteration**
* Target training length: **1,000 iterations**
* Estimated total training time: **~2–4 hours**
* Hardware: **CPU only**

This confirmed that the complete experiment could be reproduced locally without requiring dedicated GPU resources.

---

## 2. Architecture Correction

The public repository was compared against the hyperparameters explicitly stated in the paper.

Several differences were identified:

| Hyperparameter        | Repository | Paper | Correction |
| --------------------- | ---------: | ----: | ---------: |
| Hidden state size     |         20 |    27 |         27 |
| Message-passing steps |          4 |     7 |          7 |
| Dropout rate          |       0.01 |   0.1 |        0.1 |

These values were corrected to match the paper before running the main reproduction experiments.

This distinction is important because reproducing the repository's default configuration would not constitute a direct reproduction of the architecture described in the paper.

---

# 3. Baseline Reproduction on NSFNet

The corrected model was trained on **NSFNet**, the topology used for training in the original paper.

Each experiment was run for **1,000 iterations** using three independent random seeds:

* Seed 37
* Seed 73
* Seed 101

The trained DQN agent was evaluated against two routing baselines:

* **SAP:** Shortest Available Path
* **LB:** Load Balancing with random routing

### Results

| Seed |  DQN |  SAP |   LB |
| ---: | ---: | ---: | ---: |
|   37 | 13.2 | 15.3 | 11.2 |
|   73 | 14.6 | 15.3 | 11.2 |
|  101 | 13.5 | 15.3 | 11.2 |

### Observations

The DQN agent consistently:

* **Outperforms LB** across all three seeds.
* **Underperforms SAP** by approximately 1–2 points.
* Produces relatively consistent results across independent runs.

The training curves plateau approximately between **iteration 600 and 1,000**, which is consistent with the convergence behavior reported in the paper, where convergence occurs around iteration 600.

### Reproduction assessment

The training behavior was successfully reproduced in two important respects:

1. The learned policy consistently improves over the random load-balancing baseline.
2. The approximate convergence point is consistent with the original paper.

However, the DQN did not reach the performance of SAP in any of the three runs.

---

# 4. Generalization to Unseen Topologies

The paper's main claim is that a model trained on one topology can generalize to previously unseen network topologies.

To test this claim, each model trained on NSFNet was evaluated **without retraining** on two unseen topologies:

| Topology | Nodes | Relative to NSFNet   |
| -------- | ----: | -------------------- |
| NSFNet   |    14 | Training topology    |
| GBN      |    17 | Similar size         |
| GEANT2   |    24 | Substantially larger |

The two test topologies were selected to examine whether generalization changes as the size of the unseen topology increases.

---

## GBN Results

GBN contains **17 nodes**, making it relatively close in size to NSFNet's 14 nodes.

| Seed |  DQN |   LB | DQN vs LB           |
| ---: | ---: | ---: | ------------------- |
|   37 | 10.0 | 10.4 | DQN slightly better |
|   73 | 11.6 | 10.5 | DQN slightly worse  |
|  101 | 10.7 | 10.5 | DQN slightly better |

### Observation

The results on GBN are mixed but approximately tied with the LB baseline.

The model therefore demonstrates **reasonable generalization** to a topology that is similar in size to the training topology.

---

## GEANT2 Results

GEANT2 contains **24 nodes**, substantially larger than the 14-node NSFNet training topology.

| Seed |  DQN |   LB | DQN vs LB |
| ---: | ---: | ---: | --------- |
|   37 |  8.4 | 12.0 | DQN worse |
|   73 | 11.4 | 12.1 | DQN worse |
|  101 | 10.4 | 12.0 | DQN worse |

### Observation

The DQN model underperforms the LB baseline on GEANT2 for **all three independent seeds**.

Unlike the GBN experiment, there is no seed in which the DQN clearly exceeds the baseline.

---

# Key Findings

The reproduction produced three main findings.

### 1. Training behavior was successfully reproduced

The corrected DQN model consistently outperformed the random load-balancing baseline on NSFNet.

The training curves also showed convergence around **iteration 600–1,000**, broadly consistent with the behavior reported in the original paper.

### 2. The SAP comparison was only partially reproduced

The DQN agent remained approximately **1–2 points behind SAP** across all three independent seeds.

Because the gap appears consistently across multiple seeds, it is unlikely to be explained by a single unfavorable random initialization.

### 3. Generalization appears to be conditional on topology size

The generalization experiment suggests that the model's ability to transfer to unseen topologies depends on how different the target topology is from the training topology.

| Topology | Nodes | Generalization Result    |
| -------- | ----: | ------------------------ |
| NSFNet   |    14 | Training topology        |
| GBN      |    17 | Approximately matches LB |
| GEANT2   |    24 | Consistently below LB    |

The model generalizes reasonably to **GBN**, which has a topology size similar to NSFNet.

However, performance deteriorates on **GEANT2**, which is substantially larger than the training topology.

This suggests that the generalization claim may hold under certain topology conditions rather than uniformly across unseen topologies.

---

# Reproduction vs. New Finding

An important distinction is made between reproducing the original results and extending the analysis.

### Reproduced

* DQN training on NSFNet
* Convergence behavior
* Improvement over random routing / LB
* Generalization experiment methodology

### Partially reproduced

* Comparison against SAP

The DQN consistently remains below SAP in the reproduced experiments.

### New observation

The experiments indicate a potential relationship between **topology size and generalization performance**:

> A model trained on NSFNet can transfer reasonably to a similarly sized unseen topology, but its performance can deteriorate substantially when evaluated on a significantly larger topology.

This behavior was observed consistently across three independent random seeds on GEANT2.

The observation should therefore be presented as a **finding from this reproduction**, rather than as a direct claim made by the original paper.

---

# Current Status

| Component                  | Status   |
| -------------------------- | -------- |
| Environment setup          | Complete |
| Dependency modernization   | Complete |
| Architecture verification  | Complete |
| Hyperparameter correction  | Complete |
| NSFNet training            | Complete |
| Three-seed reproduction    | Complete |
| SAP comparison             | Complete |
| GBN generalization         | Complete |
| GEANT2 generalization      | Complete |
| Main reproduction result   | Complete |
| Presentation-ready results | **Yes**  |

## Optional Follow-up Experiments

If additional time is available, two experiments remain useful:

### 1. Longer NSFNet Training

Run an extended NSFNet training experiment beyond 1,000 iterations to determine whether the DQN-SAP performance gap decreases with additional training.

### 2. `readout_units` Verification

The `readout_units` hyperparameter used by the implementation has not been identified in the paper's stated hyperparameters.

This value should be investigated further to determine whether:

* it is specified elsewhere in the original implementation,
* it is implicit in the architecture,
* or it is an implementation-specific choice not documented in the paper.

---

# Overall Reproduction Status

**Core reproduction: Complete**

The reproduction successfully demonstrates the main training behavior reported in the original work and provides additional evidence about the limitations of cross-topology generalization.

The most notable result is the difference between **GBN and GEANT2**: the model transfers reasonably to the similarly sized GBN topology but consistently underperforms the LB baseline on the substantially larger GEANT2 topology across all three random seeds.
