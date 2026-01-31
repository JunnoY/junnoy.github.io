---
layout: page
title: Robot Learning for Crossing a Red Line
description: Learn a policy (in `robot.py`) that crosses the red line efficiently under a limited training budget.
img: assets/img/robot_learning.png  # TODO: add/replace thumbnail
importance: 4
category: work
related_publications: false
# github: https://github.com/USERNAME/REPO  # optional
---

Coursework project: implement a robot learning method in `robot.py` to **learn to cross the red line** with minimal test-time steps.

<!-- Optional: match the typography style used in IC_robot_teleoperation_project.md -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@600;700&family=Source+Serif+Pro:wght@400;600&display=swap" rel="stylesheet">

<style>
.page-content p,
.page-content li,
.page-content td,
.page-content th,
.page-content {
  font-family: 'Source Serif Pro', Georgia, 'Times New Roman', serif;
  font-size: 18px;
  line-height: 1.7;
  color: #2b2b2b;
}

a { color: #1772d0; text-decoration: none; }
a:hover { color: #f09228; }

heading {
  display: block;
  font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
  font-weight: 700;
  font-size: 24px;
  letter-spacing: -0.02em;
  margin-top: 2.2em;
  margin-bottom: 0.6em;
  color: inherit;
}

.figure-caption {
  font-family: 'Times New Roman', 'Source Serif Pro', serif;
  font-size: 15px;
  color: #666;
  text-align: center;
  margin-top: 0.5em;
}
</style>

---

<heading>Project Overview</heading>

This project develops a **budget-aware robot learning method** that trains during a limited-cost training phase and then executes a fast, no-training policy during testing to **cross a red line** as quickly as possible.

**Core constraints:**
- During training, you may step, reset, or request demonstrations (each has a cost).
- Training ends when the budget is exhausted; testing begins immediately after.
- During testing, the robot starts from random left-side placements via `environment.random_reset()` and must reach/cross the line quickly.

---

<heading>The Task</heading>

**Goal:** Learn to cross the red line (or get as close as possible if it fails).

**Scoring:** Based on
- how quickly the robot crosses the line during testing, or
- how close it gets if it cannot reach it.

---

<heading>Training & Testing Protocol</heading>

#### Training Phase
- The main loop (see `robot-learning.py`) repeatedly calls `Robot.training_action()`.
- Your method chooses between stepping, resetting, requesting demonstrations, or ending training.
- **Important:** avoid expensive computation that can push budget below zero.

#### Testing Phase
- No further learning/training is allowed.
- The robot is evaluated on new episodes with random initial states (left side).
- Objective: cross the line in minimal steps.

---

<heading>Operations Available (Actions)</heading>

In training, `Robot.training_action()` must return `(action_type, action_value)`:

1. **Step**  
   - `action_type = 1`  
   - `action_value = action` (the control action executed in the environment)

2. **Reset**  
   - `action_type = 2`  
   - `action_value = state` (the state to reset to)

3. **Request Demonstration**  
   - `action_type = 3`  
   - `action_value = (demo_start, demo_length)`  
     - `demo_start = 0` means “start from the robot’s current state”
     - `demo_length` is the number of demo steps

4. **End Training**  
   - `action_type = 4`  
   - `action_value` ignored

---

<heading>Budget & Cost Model</heading>

Training is constrained by a **money budget**; costs include:
- **Environment steps**
- **Resets**
- **Demonstrations**
- **Computation time** (planning / training overhead)

Cost values are defined in `constants.py`.

**Key rule:** Do not let money go below zero (penalty risk). Ensure the method returns to the main loop frequently and keeps per-iteration compute bounded.

---

<heading>Learning Method (High-Level Design)</heading>

This section describes the approach implemented in `robot.py`.

#### 1) Observations (No Access to State)
- The robot receives **observations** (high-dimensional vectors), not true states.
- Strategy: feature extraction / normalization (e.g., running mean/variance) and compact representation.

#### 2) Data Collection Strategy (Budget-Aware)
- When to step vs reset vs request demonstrations.
- How to select `demo_start` and `demo_length` to maximize learning per unit cost.

#### 3) Policy / Controller Representation
- Example options (choose what you actually implemented):
  - model-free RL policy (e.g., tabular / linear / small neural net)
  - imitation + fine-tuning
  - model-based planning with learned dynamics from observations
  - hybrid: demonstrations bootstrap + lightweight online improvement

#### 4) Training Objective & Update Rule
- Reward definition proxying “progress toward the red line”.
- Update frequency and compute budgeting (e.g., small batches, early stopping, time caps).

#### 5) Testing-Time Execution (No Training)
- Deterministic policy execution (or limited stochasticity).
- Safety checks / action clipping.
- Reset handling and recovery behaviors if stuck.

---

<heading>Implementation Notes (`robot.py`)</heading>

**You only submit `robot.py`.** Keep all required function names and signatures unchanged because `robot-learning.py` calls them.

Checklist:
- Functions required by the starter interface (do not rename / change args).
- Any additional helper classes/functions must live inside `robot.py`.
- Avoid dependencies on modified versions of other files.

---

<heading>Environment & Dynamics (What Matters)</heading>

Summarize what you learned from `environment.py`:
- action space (dimensionality, bounds)
- episode termination conditions
- observation format
- how the red line condition is detected / measured (if available)

---

<heading>Evaluation</heading>

Report the metrics you used:
- **Test success rate** (% episodes crossing the line)
- **Steps-to-cross** (mean / median)
- **Closest distance** when failing
- **Training cost breakdown** (steps vs resets vs demos vs compute)

---

<heading>Results</heading>

<p align="center">
  <img src="/assets/img/robot_learning_results.png" alt="Results" width="80%">  <!-- TODO -->
  <br>
  <em class="figure-caption">Learning curve / test performance summary (placeholder).</em>
</p>

- Training budget used: `TODO`
- Best test performance: `TODO`
- Failure cases: `TODO`

---

<heading>Demo (Optional)</heading>

<!-- Replace with your own hosted video (Drive/YouTube/etc.) if you have one -->
<!--
<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe class="embed-responsive-item"
          src="TODO_PREVIEW_LINK"
          allow="autoplay; encrypted-media"
          allowfullscreen>
  </iframe>
</div>
<em class="figure-caption">Policy execution during testing (placeholder).</em>
-->

---

<heading>References / Acknowledgements</heading>

- Starter code: `robot-learning.py`, `robot.py`, `environment.py`, `constants.py`
- Any algorithms referenced (brief citations/links)