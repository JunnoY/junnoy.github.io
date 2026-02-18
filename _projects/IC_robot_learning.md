---
layout: page
title: Robot learning for Navigation
img: /assets/img/robot_learning/image.png
importance: 3
category: work
related_publications: false
---

<!-- Import fonts -->
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@600;700&family=Source+Serif+Pro:wght@400;600&display=swap" rel="stylesheet">

<style>
/* --- Global Typography --- */
.page-content p,
.page-content td,
.page-content th,
.page-content tr,
.page-content {
  font-family: 'Source Serif Pro', Georgia, 'Times New Roman', serif;
  font-size: 18px;
  line-height: 1.7;
  color: #2b2b2b;
}


/* --- Links --- */
a {
  color: #1772d0;
  text-decoration: none;
}
a:hover {
  color: #f09228;
}

/* --- Section Headings (like "Method") --- */
heading {
  display: block;
  font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
  font-weight: 700;
  font-size: 24px;
  letter-spacing: -0.02em;
  margin-top: 2.5em;
  margin-bottom: 0.6em;
  color: inherit; /* ← inherit from parent / theme */
}

/* --- Optional Page Title --- */
name {
  display: block;
  font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
  font-size: 42px;
  font-weight: 700;
  text-align: center;
  margin-bottom: 0.5em;
}

/* --- Paper-style Subtitles --- */
papertitle {
  display: block;
  font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
  font-size: 20px;
  font-weight: 600;
  margin-top: 1.2em;
}

/* --- Captions --- */
.figure-caption {
  font-family: 'Times New Roman', 'Source Serif Pro', serif;
  font-size: 15px;
  color: #666;
  text-align: center;
  margin-top: 0.5em;
}


/* --- Tables --- */
table {
  margin: 0 auto;
  border-collapse: collapse;
  width: 90%;
}
td {
  vertical-align: top;
  padding: 10px;
}

.champion-box {
  border: 3px solid #FFD700;
  border-radius: 10px;
  padding: 15px;
  margin: 20px 0;
  background: linear-gradient(135deg, #fff9e6 0%, #fffbf0 100%);
  box-shadow: 0 4px 8px rgba(255, 215, 0, 0.3);
}

.champion-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
  font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif;
  font-weight: 700;
  font-size: 20px;
  color: #B8860B;
}

/* --- Video Containers --- */
.video-container {
  margin: 20px 0;
  text-align: center;
  width: 100%;
  max-width: 900px;
  margin-left: auto;
  margin-right: auto;
}

.video-container iframe {
  width: 100%;
  max-width: 900px;
  height: 506px; /* 16:9 aspect ratio */
  border: none;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

@media (max-width: 768px) {
  .video-container iframe {
    height: 360px;
  }
}
</style>

<heading>Project Overview</heading>
This coursework aims to implement a robot learning method to learn to cross the red line with minimal test-time steps, with limited budgets for exploration. It compares budget-aware robot learning methods for a navigation task in a static but stochastic environment: the robot must cross the red line using as few test-time steps as possible, under a strict training budget.

**Core constraints**
- Training costs money: time (`$0.03/s`), step (`$0.002`), reset (`$5`), and demos (`$5/request + $0.5/demo step`). Total budget: `$100`.
- The environment is static within an episode but changes across episodes (stochastic layouts).
- The robot does not observe resistance directly, yet resistance strongly affects progress: a longer low-resistance route can beat a shorter high-resistance one.

<heading>Resistance-aware Reward Shaping</heading>
The default reward is based on x-distance to the goal line, but positions with the same x value can have very different resistance. It is highly unlikely for the robot to reach the goal under time limits when it is stuck in a high resistant region. We therefore need a learning signal that gives higher rewards to the low resistant regions that encourage the robot to walk towards.

Although resistance is not observable in this setting, the environment's reward reflects distance to the goal, which the change in reward between steps indicates the speed of the robot in this region. We therefore know the resistance of the robot's current region.

The speed of the robot is computed by: 

$$\text{Speed} = \frac{\Delta R}{\sqrt{(\Delta x)^2 + (\Delta y)^2}}$$

where $\Delta R$ is the change in reward, and $\Delta x$, $\Delta y$ are the components of the robot's action in the x and y directions.

We compute a composite reward combining:

- the original distance-to-goal $R_1$, and
- a progress-based term derived from reward differences $R_2$ (encouraging "fast" areas),

$$R = 0.2 R_1 + 0.8 R_2$$

where $R$ is the composite reward.

<heading>Algorithms</heading>
This section introduces 6 different algorithms that are implemented to solve this problem. [Algorithm 1](#algorithm-1-baseline-always-right) is the baseline. [Algorithm 2](#algorithm-2-model-based-rl-policy--cem-planning) and [Algorithm 3](#algorithm-3-behaviour-cloning-bc) are implemented from scratch. [Algorithm 4](#algorithm-4-ppo-proximal-policy-optimization), [Algorithm 5](#algorithm-5-sac-soft-actor-critic), and [Algorithm 6](#algorithm-6-td3-bc-twin-delayed-ddpg-with-behavior-cloning) are implemented using the [TianShou RL library](https://github.com/thu-ml/tianshou).

##### **Algorithm 1: Always Right (Baseline)** {#algorithm-1-baseline-always-right}
Random actions often get the robot stuck in high-resistance areas. As a simple but strong baseline, the robot:

- always moves **right**, and
- randomly moves **up/down** to search for easier passages.

This provides a stable reference point that learned methods should beat.

##### **Algorithm 2: Model-Based RL policy + CEM Planning** {#algorithm-2-model-based-rl-policy--cem-planning}
With limited data collection budget and a stochastic environment, my initial instinct is to let the robot have a thorough exploration of the environment rather than asking for the expensive demos.

- At the **training stage**, to ensure thorough exploration, we split the environment to 8 regions, and allow the robot to randomly explore in each region. This prevents the robot accidentally enters a high resistant region during testing that it has not explored before.
- The robot learns a **dynamics model** to predict next observations, and **reward model** to evaluate candidate action sequences for planning
- At the **testing stage**, we use **CEM planning**, which the robot simulates some potential paths with several possible actions, and choose the action that leads to the path with highest reward. Notice **CEM planning** is only valid when the **dynamics model** and **reward model** learn the environment very well.
- A **waypoint library** is created while training to store all the highly efficient transitions. During testing, the robot will use the waypoint library to find the nearest waypoints and use **CEM planning** to find an action sequence that reaches the waypoint.

##### **Algorithm 3: Behaviour Cloning (BC)** {#algorithm-3-behaviour-cloning-bc}
In [Algorithm 2](#algorithm-2-model-based-rl-policy--cem-planning), when the robot gets stuck in a high resistant region, it will be reset to a new region and explore again. However, this method does not teach the robot how to escape when it gets stuck. We improve the policy by combining model-based RL with behaviour cloning, which when the robot gets stuck, we request a demo of 20 steps and save it to the dataset with the highest rewards (**the robot does not execute these demo steps, it continues exploring randomly until the it runs out of steps in that region**), so the robot can learn how to act properly when it gets stuck. Noted that in order to save budget, if we have already asked for a demo in certain observation previously, we will not ask for a demo again.

Note that the training and testing stages are the same as [Algorithm 2](#algorithm-2-model-based-rl-policy--cem-planning), and it also uses CEM-planning and the waypoint library to find an action sequence to reach the nearest waypoint with high efficiency. With the demos, the training dataset and the waypoint library have more high quality data for the robot the learn better.

##### **Algorithm 4: PPO (Proximal Policy Optimization)** {#algorithm-4-ppo-proximal-policy-optimization}
PPO is an **on-policy, online** algorithm that uses **clipped objective functions** to ensure stable policy updates.

The general process of PPO is as follows:

1. A actor-critic policy is initialised
2. For every iteration, the robot samples from the current policy (which is a continuous probabilistic distribution) to explore the environment and collect data
3. After data collection in each iteration, the current policy is saved as the old policy, and a new policy is updated using the current policy and the new collected data
4. When updating the policy, we must ensure the **KL-Divergence** between the old policy and the new policy is within a threshold. PPO uses a mechanism named [LCLIP](#lclip) to limit the update of the new policy, to make sure the actions sampled from the new policy are better than average
5. After update, all training data will be removed, the subsequent policies are only updated by the data collected using their last policy

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/PPO.png" alt="PPO Algorithm" style="max-width: 80%; height: auto;">
  <div class="figure-caption">PPO Algorithm. Adapted from <a href="https://arxiv.org/abs/1707.06347" target="_blank">Schulman et al., 2017, Proximal Policy Optimization Algorithms</a>.</div>
</div>

##### **Algorithm 5: SAC (Soft Actor-Critic)** {#algorithm-5-sac-soft-actor-critic}
SAC is a off-policy, online algorithm that maximizes both expected return and entropy, encouraging exploration while maintaining sample efficiency.

The differences between SAC and PPO:

- SAC is off-policy which it randomly explores first (with a policy) and use all the data collected to train a new policy (the data can be reused to train the policy multiple times); unlike PPO (on-policy), which it updates the policy using the data collected by the policy itself. They are both online learning since they all use the data that are collected by interacting with the environment
- PPO's policy update mechanism is like fine-tuning the policy with higher quality data each time. You might wonder whether the data used by SAC are high quality enough. SAC has a special mechanism which maximizes both expected return and entropy. It chooses the action with the greatest expected return, and also ensures that this action has high entropy, which means the action is more diverse from the previous actions to encourage more exploration. This ensures the quality of the SAC's training data.

SAC alternates between two processes:
1. interacting with the environment and storing data in a replay buffer;
2. sampling mini-batches from the buffer and using stochastic gradient descent to simultaneously update the Q-function, policy network, and temperature parameter ${\alpha}$.

it has two critics networks to prevent overestimation bias (by taking the minimum Q value).

The [temperature parameter ${\alpha}$](#sac-temp) is used to control the entropy, it can be auto-adjusted by per iteration (of the policy updates).

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/SAC.png" alt="SAC Algorithm" style="max-width: 80%; height: auto;">
  <div class="figure-caption">SAC Algorithm. Adapted from <a href="https://arxiv.org/abs/1801.01290" target="_blank">Haarnoja et al., 2018, Soft Actor-Critic Algorithms and Applications</a>.</div>
</div>

<heading>Training Phase</heading>
<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe src="https://drive.google.com/file/d/1IW1zrJyWPhf5ccjFB1qzRP9CYwav93_k/preview" width="640" height="480"></iframe>
</div>
<div class="figure-caption">
  Training: the environment is divided into 8 regions, and the robot explores each region under a fixed budget allocation.
</div>


<heading>Testing Videos</heading>

<div class="champion-box">
  <div class="champion-header">
    <span style="font-size: 28px;">🏆</span>
    <span>1st Place (Tied): Behavioural Cloning - Success Rate: 93.3%</span>
  </div>
  <div class="video-container">
    <iframe src="https://drive.google.com/file/d/1hG9-6-ewOImfFDlqkZhN62jB1E-geEIr/preview" width="640" height="480"></iframe>
    <div class="video-caption">Behavioural Cloning: Achieves highest success rate (93.3%, 56/60 successful runs) with efficient median testing time of ~7 seconds. Combines model-based exploration with demonstration learning.</div>
  </div>
</div>

<div class="champion-box">
  <div class="champion-header">
    <span style="font-size: 28px;">🏆</span>
    <span>1st Place (Tied): SAC - Success Rate: 93.3%</span>
  </div>
  <div class="video-container">
    <iframe src="https://drive.google.com/file/d/1UKTfQahUi96KGXU0gjy4HLYmeFOUaUHo/preview" width="640" height="480"></iframe>
    <div class="video-caption">SAC (Soft Actor-Critic): Ties for highest success rate (93.3%, 56/60 successful runs) with median testing time of ~14 seconds. Demonstrates fast and stable convergence during training with off-policy learning.</div>
  </div>
</div>

<div class="video-container">
  <div style="font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif; font-weight: 600; font-size: 18px; color: #2b2b2b; margin-bottom: 10px;">2nd Place: PPO - Success Rate: 90.0%</div>
  <iframe src="https://drive.google.com/file/d/1fLtzu4TBePmrxIF-ecLc-pCeGGnOtqYZ/preview" width="640" height="480"></iframe>
  <div class="video-caption">PPO (Proximal Policy Optimization): Achieves competitive success rate (90.0%, 54/60 successful runs) with highest path efficiency (0.93) among learning methods. Shows more variable testing times with bimodal distribution.</div>
</div>

<div class="video-container">
  <div style="font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif; font-weight: 600; font-size: 18px; color: #2b2b2b; margin-bottom: 10px;">3rd Place: Always Right (Baseline) - Success Rate: 73.3%</div>
  <iframe src="https://drive.google.com/file/d/13IL0Al5HN_pB_8KfSPw0CKFYSyRR8nHg/preview" width="640" height="480"></iframe>
  <div class="video-caption">Always Right: Baseline approach achieving 73.3% success rate (44/60 successful runs). Moves directly rightward with random vertical exploration. Shows perfect path efficiency (1.0) but lower success rate due to stochastic environment challenges.</div>
</div>

<div class="video-container">
  <div style="font-family: 'Inter', 'Helvetica Neue', Arial, sans-serif; font-weight: 600; font-size: 18px; color: #2b2b2b; margin-bottom: 10px;">4th Place: Model-Based + CEM - Success Rate: 68.3%</div>
  <iframe src="https://drive.google.com/file/d/1hS3KFojTCoX2VKp9NbHpVGXXqacxLTYe/preview" width="640" height="480"></iframe>
  <div class="video-caption">Model-Based + CEM: Achieves lowest success rate (68.3%, 41/60 successful runs) with highest testing times (median ~60 seconds). Suggests learned dynamics and reward models may not capture environment complexity sufficiently for effective CEM planning.</div>
</div>

<heading>Evaluation</heading>

##### **Experimental Setup**

To ensure robust and statistically meaningful comparisons across all approaches, we conducted a comprehensive evaluation using **20 different random seeds** (ranging from 10 to 200 in increments of 10), with **3 independent runs per seed** for each approach. This design choice serves several important purposes:

1. **Seed diversity**: The 20 seeds cover a wide range of environment initializations and stochastic layouts, ensuring that performance is evaluated across diverse scenarios rather than being biased toward specific configurations. This is particularly important given that the environment is static within an episode but stochastic across episodes.

2. **Multiple runs per seed**: The 3 runs per seed account for the inherent stochasticity in both the learning algorithms (e.g., random action sampling in PPO and SAC, random exploration in model-based methods) and the environment dynamics. This allows us to capture the variance in performance and provides more reliable statistical estimates.

3. **Total sample size**: With 20 seeds × 3 runs = **60 total runs per approach**, we achieve sufficient statistical power to detect meaningful differences between methods while maintaining computational feasibility within the budget constraints.

This evaluation protocol results in **300 total test runs** across all 5 approaches (Always Right, Behavioural Cloning, SAC, Model-Based + CEM, and PPO), providing a comprehensive basis for comparison.

##### **Evaluation Metrics and Results**

**1. Success Rate**

Success rate is the primary metric for task completion, defined as the proportion of runs that successfully cross the red line within the time limit. This binary metric directly reflects whether each approach can solve the core navigation task. We report both point estimates and 95% confidence intervals using normal approximation for proportions, allowing us to assess statistical significance of differences between approaches.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/07_success_rate_with_ci.png" alt="Success Rate with Confidence Intervals" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Success rate by approach with 95% confidence intervals.</div>
</div>

The evaluation reveals clear performance differences across approaches. **Behavioural Cloning** and **SAC** achieve the highest success rates at **93.3%** (56 successful runs out of 60), with overlapping 95% confidence intervals [0.87, 0.99], indicating no statistically significant difference between these two top performers. **PPO** follows closely with a **90.0%** success rate (54/60), with confidence interval [0.83, 0.97] that overlaps with the top two methods, suggesting competitive performance. The baseline **Always Right** achieves **73.3%** (44/60), while **Model-Based + CEM** shows the lowest success rate at **68.3%** (41/60), with confidence intervals [0.63, 0.83] and [0.58, 0.78] respectively.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/06_success_failure_breakdown.png" alt="Success vs Failure Breakdown" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Success vs failure count by approach (60 total runs per approach).</div>
</div>

These results demonstrate that learning-based approaches (Behavioural Cloning, SAC, PPO) significantly outperform both the simple baseline and the model-based planning approach, with the imitation learning and off-policy RL methods showing particular strength.

**2. Testing Time (Time Elapsed)**

Testing time measures the time taken to complete the task (or reach the time limit) during testing. This metric is crucial because the objective explicitly requires minimizing test-time steps. We analyze both the mean and distribution (via box plots and violin plots) to understand not just average performance but also consistency and variability. Lower testing times indicate more efficient policies that reach the goal faster.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/01_testing_time_boxplot.png" alt="Testing Time Distribution" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Testing time distribution by approach (box plot).</div>
</div>

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/04_testing_time_violin.png" alt="Testing Time Violin Plot" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Testing time distribution by approach (violin plot showing probability density).</div>
</div>

Testing time distributions reveal important efficiency characteristics. **Behavioural Cloning** and **SAC** exhibit the most efficient performance with median testing times of approximately **7 seconds** and **14 seconds** respectively, and tight distributions concentrated at low values. Both approaches show occasional outliers reaching up to 100 seconds (timeout), but the majority of successful runs complete quickly.

**PPO** shows a **bimodal distribution** with median around **16 seconds**, but with significant variability: many runs complete in under 25 seconds, while others approach the 100-second timeout. This suggests less consistent performance compared to Behavioural Cloning and SAC.

**Model-Based + CEM** demonstrates the highest testing times with a median of approximately **60 seconds** and a wide distribution spanning the full time range. The interquartile range extends from 22 to 100 seconds, indicating that even when successful, this approach takes substantially longer to reach the goal.

**Always Right** also shows a bimodal distribution with median around **33 seconds**, but with high variability—some runs complete very quickly (near 0 seconds) while others timeout at 100 seconds, reflecting the stochastic nature of its random vertical exploration strategy.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/02_testing_time_by_seed.png" alt="Testing Time by Seed" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Mean testing time by approach and seed, showing seed-dependent variability.</div>
</div>

Analysis across different seeds reveals significant variability in performance. For certain seeds (e.g., 10, 30, 70, 100, 150, 170, 190, 200), Behavioural Cloning, PPO, SAC, and Always Right all achieve very low testing times, while Model-Based + CEM remains relatively high. For other seeds (e.g., 20, 40, 50, 60, 90, 110, 130), multiple approaches show high testing times, often exceeding 60 seconds. This seed-dependent performance suggests that environment layout characteristics strongly influence which approaches succeed, with learning-based methods showing better generalization across diverse scenarios.

**3. Distance to Goal**

Distance to goal measures the final Euclidean distance to the goal line at the end of each run. This continuous metric provides finer-grained information than binary success/failure, revealing how close unsuccessful runs come to completion and allowing us to distinguish between approaches that fail catastrophically versus those that nearly succeed. A distance of 0.0 indicates successful goal achievement.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/05_distance_to_goal.png" alt="Distance to Goal Distribution" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Distance to goal distribution by approach. The green dashed line at y=0.0 indicates the goal.</div>
</div>

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/13_avg_distance_to_goal.png" alt="Average Distance to Goal" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Average distance to goal (all runs) with error bars showing standard deviation.</div>
</div>

When examining the final distance to goal across all runs (including failures), **SAC** and **Behavioural Cloning** achieve the lowest average distances at **0.05** and **0.06** respectively, with relatively small standard deviations. This indicates that even when these approaches fail, they typically come very close to the goal, suggesting robust navigation capabilities.

**PPO** shows an average distance of **0.12**, while **Always Right** and **Model-Based + CEM** have higher average distances of **0.22** and **0.28** respectively. The box plot analysis reveals that Behavioural Cloning, SAC, and PPO have their interquartile ranges centered at 0.0 (goal achievement), with only occasional outliers at higher distances, whereas Always Right and Model-Based + CEM show wider distributions with medians above 0.0.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/08_time_vs_distance.png" alt="Time vs Distance to Goal" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Scatter plot of time elapsed vs distance to goal, showing successful (circles) and failed (X marks) trials for each approach.</div>
</div>

The scatter plot reveals that all successful trials cluster at distance 0.0 across the full time range, while failures primarily occur at the 100-second timeout with varying distances to goal. This pattern indicates that most failures are due to time limits rather than catastrophic navigation errors.

**4. Path Efficiency**

Path efficiency is calculated as the ratio of shortest path distance to actual path distance traveled. This metric, derived from trajectory analysis, quantifies how directly each approach navigates toward the goal. A value of 1.0 indicates perfect efficiency (straight-line path), while lower values indicate more circuitous routes. This is particularly relevant for understanding whether policies learn to navigate efficiently or waste steps exploring unnecessarily.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/15_path_efficiency.png" alt="Path Efficiency" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Path efficiency (shortest path / actual path) by approach. Higher values indicate more direct navigation.</div>
</div>

Path efficiency analysis reveals interesting navigation patterns. **Always Right** achieves near-perfect efficiency (**1.0**) as expected, since it moves directly rightward. Among learning-based methods, **PPO** shows the highest path efficiency at **0.93**, indicating it learns to navigate relatively directly toward the goal. **SAC** and **Behavioural Cloning** achieve **0.90** and **0.89** respectively, showing good but slightly lower efficiency, possibly due to more exploratory behavior. **Model-Based + CEM** has the lowest path efficiency at **0.77**, suggesting more circuitous navigation paths, which aligns with its longer testing times.

**5. Action Smoothness**

Action smoothness measures the continuity of actions over time, computed as $1 / (1 + \text{average action change rate})$. Higher values (closer to 1.0) indicate smoother, more stable control policies, which are generally preferred for real-world robot deployment as they reduce mechanical wear and improve predictability.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/16_action_smoothness.png" alt="Action Smoothness" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Action smoothness by approach. Higher values indicate smoother control policies.</div>
</div>

All approaches demonstrate high action smoothness (0.97-0.99), with minimal variability. **Always Right**, **SAC**, and **PPO** achieve the highest smoothness at **0.99**, while **Behavioural Cloning** and **Model-Based + CEM** are slightly lower at **0.97**. These high values indicate that all learned policies produce stable, continuous control signals suitable for real-world deployment, with no approach showing problematic jerky behavior.

**6. Training Performance (PPO vs SAC)**

For the two online RL algorithms (PPO and SAC), we analyze training dynamics to understand learning efficiency and stability. Learning curves show sample efficiency and convergence behavior, loss curves reveal training stability, sample efficiency indicates how quickly each algorithm learns, and training time is relevant for computational cost considerations.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/09_learning_curves_rewards.png" alt="Learning Curves" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Learning curves showing episode rewards over training for PPO and SAC (mean ± std).</div>
</div>

**Learning Curves**: SAC demonstrates faster and more stable convergence, reaching near-optimal performance (mean reward ≈ 0) by episode 50-60 and maintaining it consistently. PPO shows more variability during early training (episodes 0-80) with a wider standard deviation band, and experiences a temporary performance dip around episode 200 before recovering. Both eventually converge to similar final performance.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/12_loss_curves.png" alt="Loss Curves" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Training loss curves showing actor loss over training steps for PPO and SAC (mean ± std).</div>
</div>

**Loss Curves**: PPO maintains a stable, near-zero actor loss throughout training with minimal variance, indicating consistent policy updates. SAC exhibits highly dynamic actor loss values (ranging from -20 to -180) with large standard deviations, reflecting its different optimization objective that maximizes both return and entropy.

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/10_sample_efficiency.png" alt="Sample Efficiency" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Sample efficiency: episodes required to reach target reward (-50) for PPO and SAC.</div>
</div>

**Sample Efficiency**: Both PPO and SAC reach the target reward threshold (-50) in approximately **4.1-4.3 episodes** on average, with PPO showing slightly higher variability (error bar extending from 0 to 8.8 episodes) compared to SAC (0.8 to 7.4 episodes).

<div style="text-align: center; margin: 2em 0;">
  <img src="/assets/img/robot_learning/11_training_time.png" alt="Training Time" style="max-width: 90%; height: auto;">
  <div class="figure-caption">Training time comparison between PPO and SAC.</div>
</div>

**Training Time**: Both methods require similar computational time, with PPO averaging **1.85 minutes** and SAC **1.9 minutes**, with overlapping error bars indicating no significant difference in training duration.

**Summary and Key Findings**

1. **Best Overall Performance**: Behavioural Cloning and SAC tie for highest success rate (93.3%) with efficient testing times, making them the top-performing approaches overall.

2. **Efficiency Trade-offs**: While Always Right achieves perfect path efficiency, its lower success rate (73.3%) and variable testing times make it inferior to learning-based methods for this stochastic environment.

3. **Model-Based Limitations**: Model-Based + CEM shows the lowest success rate and highest testing times, suggesting that the learned dynamics and reward models may not capture the environment complexity sufficiently for effective CEM planning.

4. **PPO Strengths**: PPO achieves competitive success rate (90.0%) with the highest path efficiency among learning methods (0.93), indicating it learns efficient navigation strategies, though with more variable testing times.

5. **Training Insights**: SAC's faster convergence and more stable learning curves suggest better sample efficiency during training, though both PPO and SAC achieve similar final test performance.

6. **Robustness**: The high action smoothness across all approaches (0.97-0.99) indicates that all learned policies are suitable for real-world deployment from a control stability perspective.


### **References**
##### **PPO: LCLIP** {#lclip}
PPO first defines $r_t(\theta) = \frac{\pi_{\theta_{\text{old}}}(a_t|s_t)}{\pi_\theta(a_t|s_t)}$, which is the probability difference between the old policy and the new policy of sampling the same action under the same observation. When $r_t(\theta) = 1$, the new policy is the same as the old policy

PPO then defines the $L^{\text{CLIP}}$ function as $L^{\text{CLIP}}(\theta) = \mathbb{E}_t[\min(r_t(\theta)\hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)\hat{A}_t)]$, where $\hat{A}_t$ is the **advantage estimation** function that estimate how good an action is compared to the average

$L^{\text{CLIP}}$ aims to limit the probability ratio $r_t(\theta)$ to the range $[1-\epsilon, 1+\epsilon]$, where $\epsilon$ is usually **0.2**. By taking the minimum value between the unclipped/actual probability ratio $r_t(\theta)\hat{A}_t$ and the clipped probability ratio, $L^{\text{CLIP}}$ becomes the lower bound of the unclipped probability ratio, which is a pessimistic estimation.

**Pessimistic estimation:**
- When $\hat{A}_t > 0$: it means the actions sampled from the new policy are better than average. So when $r_t(\theta) > 1+\epsilon$, $L^{\text{CLIP}} = (1+\epsilon)\hat{A}_t$, ensuring the new policy would not deviate from the old policy so much
- When $\hat{A}_t < 0$: it means the actions sampled from the new policy are worse than average. So when $r_t(\theta) < 1-\epsilon$, $L^{\text{CLIP}} = (1-\epsilon)\hat{A}_t$, ensuring the new policy still have a stable gradient signal for updates

For example:
- Unclipped: $-0.06$ is a very small gradient signal. The policy already reduced the action probability from 0.5 to 0.01, so the update signal is weak and may not guide further improvement effectively.
- Clipped: $-2.4$ provides a stronger, more stable gradient signal, allowing the policy to continue learning from this bad action.

##### **SAC: the temperature parameter $\alpha$** {#sac-temp}
**Problem:** The temperature parameter $\alpha$ balances entropy against reward. Reward scales vary across tasks, making manual $\alpha$ tuning difficult and error-prone.

**Solution:** SAC formulates this as a constrained optimization: maximize expected return while maintaining a minimum expected entropy constraint. By solving the dual problem, the algorithm derives a **gradient rule** that automatically updates $\alpha$ each iteration.

**Mechanism:** If the policy's entropy falls below a target entropy $\bar{\mathcal{H}}$, $\alpha$ increases to encourage exploration; if it exceeds the target, $\alpha$ decreases to make the policy more deterministic.

**Benefit:** This automatic adaptation improves robustness and usability across tasks without task-specific hyperparameter tuning.