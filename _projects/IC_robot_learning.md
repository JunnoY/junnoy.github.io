---
layout: page
title: Robot learning for Navigation
img: assets/img/robot_learning.png  # TODO: add/replace thumbnail
importance: 3
category: work
related_publications: false
---

Coursework project: implement a robot learning method to **learn to cross the red line** with minimal test-time steps, with limited budgets for exploration.

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

.best-approach {
  position: relative;
  border: 3px solid #f09228;
  border-radius: 8px;
  padding: 12px;
  background: linear-gradient(135deg, rgba(240, 146, 40, 0.08) 0%, rgba(23, 114, 208, 0.05) 100%);
  box-shadow: 0 4px 16px rgba(240, 146, 40, 0.25);
  margin: 25px 0;
}

.best-badge {
  position: absolute;
  top: -14px;
  left: 20px;
  background: linear-gradient(135deg, #f09228 0%, #e8851f 100%);
  color: white;
  padding: 5px 14px;
  border-radius: 5px;
  font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.5px;
  z-index: 10;
  box-shadow: 0 2px 8px rgba(240, 146, 40, 0.4);
}

.best-badge::before {
  content: "🏆 ";
}
</style>

---
<heading>Project Overview</heading>

This coursework compares **budget-aware robot learning methods** for a navigation task in a **static but stochastic environment**: the robot must **cross the red line** using as **few test-time steps** as possible, under a strict training budget.

**Core constraints**
- Training costs money: time ($0.03/s), step ($0.002), reset ($5), and demos ($5/request + $0.5/demo step). Total budget: **$100**.
- The environment is **static within an episode** but changes **across episodes** (stochastic layouts).
- The robot **does not observe resistance directly**, yet resistance strongly affects progress: a **longer low-resistance route** can beat a shorter high-resistance one.

**Key challenge**  
The default reward is based on **x-distance to the goal line**, but positions with the same x can have very different resistance. We therefore need a learning signal that encourages the robot to prefer **low-resistance corridors**, using only what the robot can observe.

---

<heading>Method Design</heading>

**Baseline**  
Random actions often get the robot stuck in high-resistance areas. As a simple but strong baseline, the robot:
- always moves **right**, and
- randomly moves **up/down** to search for easier passages.

This provides a stable reference point that learned methods should beat.

**Why model-based**  
With limited training budget and a stochastic environment, we focus on **model-based RL**:
- learn a **dynamics model** to predict next observations, and
- learn a **reward model** to evaluate candidate action sequences for planning (MPC).

**Budget-aware exploration (region splitting)**  
To avoid spending the entire budget in one difficult area, we split the map into **8 regions** and allocate exploration **evenly** across them. If the robot reaches the goal early in a region, leftover budget is carried forward to the remaining regions.

**Resistance-aware reward shaping (without using resistance as input)**  
Although resistance is unobserved, we found a practical proxy:
- the environment’s reward reflects **distance to the goal**;
- the **change in reward between steps** indicates how much progress an action achieved;
- **smaller progress per step** suggests higher resistance.

We compute a **composite reward** combining:
- the original distance-to-goal reward, and
- a progress-based term derived from reward differences (encouraging “easy-progress” areas),

and use this target to train the reward model.

**Behaviour cloning (demo usage)**  
We also tested demos + BC: the robot requests a demonstration when stuck in a high-resistance region and cannot escape within 20 steps. In practice, demos are expensive, which reduces coverage of the 8 regions under the $100 cap—often hurting generalisation.

---

<heading>Training Phase</heading>

<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe class="embed-responsive-item"
          src="https://drive.google.com/file/d/1IW1zrJyWPhf5ccjFB1qzRP9CYwav93_k/view?usp=sharing"
          allow="autoplay; encrypted-media"
          allowfullscreen>
  </iframe>
</div>
<em class="figure-caption">Training: the environment is divided into 8 regions, and the robot explores each region under a fixed budget allocation.</em>

---

<heading>Testing Videos</heading>

<div class="best-approach">
  <span class="best-badge">Best Performance</span>
  <div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
    <iframe class="embed-responsive-item"
            src="https://drive.google.com/file/d/1XkH-TH6_WU-Xi-3QYVIjm8ibzhx-65zU/view?usp=sharing"
            allow="autoplay; encrypted-media"
            allowfullscreen>
    </iframe>
  </div>
  <em class="figure-caption">Model-based RL with CEM planning (best overall).</em>
</div>

<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe class="embed-responsive-item"
          src="https://drive.google.com/file/d/1V_eb4azQ-bvdp1eNU0--GzozSP4trpbI/view?usp=sharing"
          allow="autoplay; encrypted-media"
          allowfullscreen>
  </iframe>
</div>
<em class="figure-caption">Baseline: always move right with random vertical moves.</em>

<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe class="embed-responsive-item"
          src="https://drive.google.com/file/d/1dlQIjJ8M315dxQX-ok3tORA1InrOgxo5/view?usp=sharing"
          allow="autoplay; encrypted-media"
          allowfullscreen>
  </iframe>
</div>
<em class="figure-caption">Behaviour cloning with limited demonstrations.</em>

---

<heading>Evaluation</heading>

We compared three methods over multiple random seeds to measure **speed, success, and stability**.

**Metrics**
- **Success rate**: % episodes crossing the red line  
- **Test time (seconds)**: proportional to steps-to-cross  
- **Consistency**: variability across random seeds  

**Setup**
- Seeds: **1, 20, 40, 60, 80, 100**
- Multiple test episodes per seed
- Identical evaluation conditions across methods

<p align="center">
  <img src="/assets/img/rl_evaluation.png" alt="Results" width="80%">
  <br>
  <em class="figure-caption">Evaluation across Baseline, Model-Based + CEM, and Behaviour Cloning.</em>
</p>

**Results (summary)**
1. **Baseline (Always Right)**
   - Success: **100%**
   - Median test time: **~7s**
   - Very stable across seeds

2. **Behaviour Cloning**
   - Success: **83.3%**
   - Median test time: **~25s**
   - High variance across seeds, with extreme slow cases (up to ~100s)

3. **Model-Based + CEM**
   - Success: **100%**
   - Median test time: **~4s** (fastest)
   - Stable across seeds (tight distribution)

**Takeaways**
- **Model-Based + CEM** is best overall: fastest median time with **perfect reliability** and **high consistency**.
- **Behaviour cloning** underperforms mainly because demonstrations are expensive: the budget only supports a small number of demos, which reduces exploration coverage and harms generalisation.
- The region-based exploration strategy plus the resistance-aware reward proxy improves robustness under stochastic layouts.