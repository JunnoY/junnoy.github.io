---
layout: page
title: Real-Time Vision-Based Hand Tracking for Robot Teleoperation and Learning
description: Imperial MSc. Robotics Project, supervised by <a href="https://www.doc.ic.ac.uk/~ajd/" target="_blank">Prof. Andrew Davison</a>
img: assets/img/robot_teleoperate.png
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

/* --- Section Headings (like “Method”) --- */
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
</style>

---

<heading>Project Overview</heading>
The rapid advancement of robotics and growing interest in embodied intelligence are bringing the vision of robots assisting humans in daily life closer to reality.

This project develops a real-time vision-based robot teleoperation framework that tracks human hand motions from RGB images and maps them to robot actions through a self-designed dynamic mapping mechanism. The system enables intuitive robot control and precise data collection, capturing both human hand keypoints and robot joint states.

By using this dataset, we aim to train models that generate high-quality robot trajectories directly from hand keypoints, overcoming the limitations of current motion-retargeting methods and accelerating the development of scalable, data-driven learning of generalisable robot policies.

---

<heading>Teleoperation Tasks Demo</heading>

<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe class="embed-responsive-item"
          src="https://drive.google.com/file/d/1why7RwDyutQ-Nh82tbSZvTIdnDxn6oLW/preview"
          allow="autoplay; encrypted-media"
          allowfullscreen>
  </iframe>
</div>

<div class="figure-caption">
  Demonstration of teleoperation tasks performed with the physical VX300s robot arm by RGB-based hand tracking in real time.
</div>

---

<heading>Robot Imitation Learning</heading>

<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe class="embed-responsive-item"
          src="https://drive.google.com/file/d/1xn9Pt2lOfbbGdhPWujNqPAoEyKmrKA7e/preview"
          allow="autoplay; encrypted-media"
          allowfullscreen>
  </iframe>
</div>

<div class="figure-caption">
  One-shot imitation learning demonstration showing the physical robot replicating the human-demonstrated task in 10 different scenes.
</div>

<heading>System Design</heading>
The overall workflow of the proposed **real-time robot teleoperation system** is as follows:

1. **Hand Tracking** — Captures RGB images in real time and computes 3D hand keypoints using **MediaPipe** and **HaMeR**.  
2. **Motion Retargeting** — Transforms hand keypoints from the **MANO frame** to the **Robot frame**, applies **Kalman filtering** for temporal smoothing, and performs **adaptive motion scaling** for natural control.  
3. **Robot Manipulation & Visualization** — Executes the mapped motions on the **physical robot arm**, runs simulations in **Gazebo**, and renders visual feedback using **Rerun**.

<p align="center">
  <img src="/assets/img/project_framework.png" alt="System Framework" width="80%">
  <br>
  <em class="figure-caption">Project Framework</em>
</p>

The architecture consists of three main components:

1. **Vision Processor** — Handles RGB frame analysis and hand keypoint extraction.  
2. **Robot Controller** — Generates joint commands via a **dynamic hand-to-robot mapper** for smooth and precise control.  
3. **Scale Tuning GUI** — Allows real-time adjustment of motion scaling for user-specific responsiveness.

This design achieves an overall data transmission rate of 30 Hz, satisfying the requirements for real-time teleoperation.

<p align="center">
  <img src="/assets/img/multi_process_thread.png" alt="Multiprocessing and Multithreading Design" width="80%">
  <br>
  <em class="figure-caption">Multiprocessing and multithreading in the system design</em>
</p>

---

<heading>Hand Tracking</heading>
The figure below illustrates the hand tracking pipeline:

1. **Input Acquisition** — RGB frames are captured from the **RealSense** camera (depth information is not used).  
2. **Image Enhancement** — Each frame undergoes **skin-tone guided correction**, **natural colour adjustment**, **contrast enhancement**, and **bilateral filtering** to normalise skin appearance and improve robustness for hand detection.  
3. **2D Hand Detection** — The **MediaPipe** hand detector processes enhanced frames, localising **21 2D keypoints** per hand and producing cropped images with **handedness labels**.  
4. **3D Hand Reconstruction** — Cropped hand images and bounding boxes are fed into the **HaMeR** model, which outputs **3D hand mesh vertices**, **camera parameters**, **3D joint keypoints** in the camera frame, and **2D projected keypoints**, as shown in Fig. 2.  
5. **Coordinate Alignment** — The predicted 3D keypoints and camera parameters are used to compute hand joint coordinates aligned with **RealSense** measurements.  
6. **Visualisation** — The reconstructed hand mesh is rendered in the RGB frame, displaying additional metrics such as the **thumb–index fingertip distance** and **camera proximity**.

<p align="center">
  <img src="/assets/img/hamer_pipeline.png" alt="Hamer Pipeline" width="80%">
  <br>
  <em class="figure-caption">Fig. 1: Hand Tracking Pipeline</em>
</p>

<p align="center">
  <img src="/assets/img/hand_structure.png" alt="Hand Structure" width="30%">
  <br>
  <em class="figure-caption">Fig. 2: Illustration of the MANO hand skeleton model, with labelled joints. This project uses joints 2, 4, 5, and 8.</em>
</p>

**Real-Time Multiprocessing** — **MediaPipe** and **HaMeR** run in separate processes to ensure **real-time performance** and low-latency execution.

---

<heading>Motion Retargeting</heading>
The following steps are done in the motion retargeting pipeline:

#### Human-to-Robot Cartesian Position and Orientation Transformation
- **Purpose:**  
  Map human hand movements to the robot’s two-finger gripper intuitively and stably.

- **Concept:**  
  The thumb corresponds to the left gripper finger, and the index finger (or closed four fingers) represents the right finger.  
  In this project, only the **thumb–index pair** is used for simplicity and stability.

##### Human Hand Position and Orientation
- **Keypoints:**  
  $ \mathcal{M} = \{ M[0], \dots, M[20] \} $ — 21 3D hand keypoints from HaMeR.

- **Gripper Centre:**  
  $$
  p_\text{mid} = \frac{M[2] + M[5]}{2}, \quad
  p_\text{gc} = \frac{p_\text{mid} + M[4] + M[8]}{3}
  $$

- **Orientation Axes:**  
  $$
  \hat{x} = \frac{p_\text{index} - p_\text{thumb}}{\|p_\text{index} - p_\text{thumb}\|}, \quad
  \hat{z} = \frac{(p_\text{index} - p_\text{mid}) \times (p_\text{thumb} - p_\text{mid})}{\|(p_\text{index} - p_\text{mid}) \times (p_\text{thumb} - p_\text{mid})\|}, \quad
  \hat{y} = \frac{\hat{z} \times \hat{x}}{\|\hat{z} \times \hat{x}\|}
  $$

<p align="center">
  <img src="/assets/img/hand_triangle.png" alt="Hand Triangle" width="40%">
  <br>
  <em class="figure-caption">Illustration of the human hand gripper: 
  Green points represent $ \mathbf{p}_{thumb} $, $ \mathbf{p}_{index} $, and $ \mathbf{p}_{mid} $;  
  the red point represents $ \mathbf{p}_{gc} $.</em>
</p>

- **Remarks:**  
  These vectors form a right-handed coordinate frame representing the hand’s local orientation.

---

##### Human-to-Robot Transformation
- **Axis Mapping:**  
  $$
  \hat{x}_R = -\hat{z}_H, \quad
  \hat{y}_R = \hat{x}_H, \quad
  \hat{z}_R = -\hat{y}_H
  $$
  Matrix form:  
  $$
  A_{R \leftarrow H} =
  \begin{bmatrix}
  0 & 0 & -1 \\
  1 & 0 & 0 \\
  0 & -1 & 0
  \end{bmatrix}
  $$

- **Position Mapping:**  
  $$
  p_R = A_{R \leftarrow H} \, p_H
  $$
  Maps relative displacements as  
  $(z_H, x_H, y_H) \mapsto (-x_R, y_R, -z_R)$.

- **Orientation Mapping:**  
  $$
  R_R = A_{R \leftarrow H} \, R_H
  $$

<p align="center">
  <img src="/assets/img/rotation_full.png" alt="Rotation Full" width="50%">
  <br>
  <em class="figure-caption">Illustration of the human-to-robot position and orientation transformation.</em>
</p>

---

##### End-Effector Position and Orientation
- **Position Computation:**  
  $$
  \Delta p_t = p_t^{meas} - p_0, \quad
  \widetilde{\Delta p}_t = \Pi_t \Delta p_t, \quad
  \widehat{\Delta p}_t = \mathcal{K}[\widetilde{\Delta p}_t], \quad
  \tilde{p}^{ee}_t = g(p_0 + \widehat{\Delta p}_t)
  $$
  Purity weighting and Kalman filtering smooth motion and suppress noise.

- **Orientation Computation:**  
  $$
  R_\Delta = R_R \, R_0^\mathsf{T}
  $$
  Convert to Euler angles → unwrap → apply purity weighting and Kalman filter →  
  convert to quaternion $ q^{ee}_t = \operatorname{Quat}_{xyz}(\widehat{\theta}_t) $.

- **Notes:**  
  - Angle unwrapping prevents discontinuities.  
  - Kalman filtering ensures smooth motion.  
  - Euler angles are intermediate; quaternions are used for IK.

---

##### Gripper State
- **Definition:**  
  $$
  d_{grip} = \|p_{thumb} - p_{index}\|
  $$
  $$
  S_{grip} =
  \begin{cases}
  \text{Closed}, & d_{grip} < 0.08 \text{ m} \\
  \text{Open}, & d_{grip} > 0.10 \text{ m}
  \end{cases}
  $$

- **Remarks:**  
  The gripper state is determined from the thumb–index fingertip distance (in metres, camera frame).

#### Purity-Weighted Decoupling of Translation and Rotation
- **Purpose:**  
  Reduce unintended coupling between translation and rotation during hand-controlled robot teleoperation.

- **Problem:**  
  Human hand motion naturally links changes in position and orientation.  
  When the user intends to rotate the hand, small translations often occur — and vice versa — causing non-intuitive robot EE motion.

- **Solution:**  
  Introduce a *motion purity mechanism* that computes translation–rotation purity scores at each time step and uses them as weights to decouple the two motion modes.

---

##### Purity Metrics
- **Translational and rotational magnitudes:**
  $$
  m_t^{trans} = \|\Delta \mathbf{p}_t\|_2, \qquad
  m_t^{rot} = \|\Delta \boldsymbol{\theta}_t\|_2
  $$
  where  
  - $ \Delta \mathbf{p}_t $: position increment  
  - $ \Delta \boldsymbol{\theta}_t $: orientation increment (roll–pitch–yaw form)

---

##### Purity Scores
- **Definition:**
  $$
  \pi_t^{trans} =
  \frac{m_t^{trans}}{m_t^{trans} + m_t^{rot} + \epsilon}, \qquad
  \pi_t^{rot} =
  \frac{m_t^{rot}}{m_t^{trans} + m_t^{rot} + \epsilon}
  $$
  where $ \epsilon $ is a small constant to prevent division by zero.  
  By design, $ \pi_t^{trans} + \pi_t^{rot} \approx 1 $.

---

##### Weighted Decoupling
- **Mechanism:**
  - If $ \pi_t^{rot} \ge 0.8 $ → translation increment $ \Delta \mathbf{p}_t $ is **suppressed**, indicating primarily rotational motion.  
  - If $ \pi_t^{trans} $ dominates → rotational increment $ \Delta \boldsymbol{\theta}_t $ is **down-weighted** but not fully suppressed.

- **Effect:**  
  The purity weights act as dynamic filters, separating translation and rotation to reduce cross-contamination.

---

##### Remarks
- Enhances precision and intuitiveness in teleoperation.  
- Prevents small hand jitters from causing mixed translation–rotation movements.  
- Complements Kalman filtering for smooth control performance.

#### Kalman Filter for Smoothed Motion
- **Purpose:**  
  Apply Kalman filtering to smooth robot position and orientation changes, reducing noise from visual measurements.

- **Background:**  
  The **Kalman filter** (Rudolf E. Kalman, 1960) estimates unobservable states from noisy observations by combining predictions from a motion model with incoming sensor data.  
  It is widely used in robot vision for fusing uncertain visual measurements over time, providing accurate and stable estimates of robot or target motion.

- **Implementation:**  
  This project uses a **constant-velocity Kalman filter**, applied separately to **translation** and **rotation**.  
  The filter operates on *incremental changes* (not absolute poses), as relative dynamics are better captured by the constant-velocity assumption.

---

##### Kalman Filtering for Relative Motion
- **Parameters:**  
  The Kalman filter uses tunable covariance matrices:
  - **P (State covariance):** confidence in current estimate — larger → rely more on new measurements.  
  - **Q (Process noise):** uncertainty in motion model — larger → faster response but more noise.  
  - **R (Measurement noise):** uncertainty in observations — larger → smoother but less responsive output.

- **Purity weighting:**  
  Translation and rotation are first decoupled via purity scores (Section *Purity-Weighted Decoupling*):  

  $$
  \widetilde{\Delta \mathbf{p}}_t =
  \begin{bmatrix}
  \widetilde{\Delta p}_x & \widetilde{\Delta p}_y & \widetilde{\Delta p}_z
  \end{bmatrix}^\mathsf{T}, \quad
  \widetilde{\Delta \boldsymbol{\theta}}_t =
  \begin{bmatrix}
  \widetilde{\Delta \phi} & \widetilde{\Delta \theta} & \widetilde{\Delta \psi}
  \end{bmatrix}^\mathsf{T}
  $$

---

##### Prediction Step
- **State vectors:**

  $$
  \mathbf{x}^p_t =
  \begin{bmatrix}
  \widetilde{\Delta p}_x & v_x & \widetilde{\Delta p}_y & v_y & \widetilde{\Delta p}_z & v_z
  \end{bmatrix}^\mathsf{T}, \quad
  \mathbf{x}^r_t =
  \begin{bmatrix}
  \widetilde{\Delta \phi} & \omega_\phi & \widetilde{\Delta \theta} & \omega_\theta & \widetilde{\Delta \psi} & \omega_\psi
  \end{bmatrix}^\mathsf{T}
  $$

- **Constant-velocity model:**

  $$
  \hat{\mathbf{x}}_{t|t-1} = \mathbf{F}\,\hat{\mathbf{x}}_{t-1|t-1}, \qquad
  \mathbf{P}_{t|t-1} = \mathbf{F}\,\mathbf{P}_{t-1|t-1}\mathbf{F}^\mathsf{T} + \mathbf{Q}
  $$

  where  

  $$
  \mathbf{F}_{1D} =
  \begin{bmatrix}
  1 & \Delta t \\ 0 & 1
  \end{bmatrix}, \quad \Delta t = 0.01~\text{s}
  $$

---

##### Correction Step (Direct Perception Input)
- **Measurement model:**

  $$
  \mathbf{z}_t = \mathbf{H}\mathbf{x}_t + \mathbf{v}_t, \quad \mathbf{v}_t \sim \mathcal{N}(\mathbf{0}, \mathbf{R})
  $$

  where $ \mathbf{z}_t $ are perception-based increments from the vision pipeline.

- **Correction update:**

  $$
  \hat{\mathbf{x}}_{t|t} = \hat{\mathbf{x}}_{t|t-1} + \mathbf{K}_t\left(\mathbf{z}_t - \mathbf{H}\hat{\mathbf{x}}_{t|t-1}\right), \quad
  \mathbf{P}_{t|t} = (\mathbf{I} - \mathbf{K}_t\mathbf{H})\,\mathbf{P}_{t|t-1}
  $$

---

##### Kalman Gain
$$
\mathbf{K}_t = \mathbf{P}_{t|t-1}\mathbf{H}^\mathsf{T}
\left(\mathbf{H}\mathbf{P}_{t|t-1}\mathbf{H}^\mathsf{T} + \mathbf{R}\right)^{-1}
$$

- **Interpretation:**  
  The Kalman gain $ \mathbf{K}_t $ adaptively balances:  
  - the **motion model** (prediction), and  
  - the **sensor measurements** (correction).  
  It yields smooth yet responsive position and orientation estimates.

---

##### Results
- **Observation:**  
  The filtered trajectories closely follow ground truth while removing high-frequency jitter from visual input.

| ![Position Kalman filter](/assets/img/position_x_kf.png) ![Position Y](/assets/img/position_y_kf.png) ![Position Z](/assets/img/position_z_kf.png) |
|:-------------------------------------------------------------------------------------------------------------------------------------------------:|
| *Kalman filter smoothing of EE positions (X, Y, Z).* |

| ![Roll Kalman filter](/assets/img/roll_kf.png) ![Pitch Kalman filter](/assets/img/pitch_kf.png) ![Yaw Kalman filter](/assets/img/yaw_kf.png) |
|:-------------------------------------------------------------------------------------------------------------------------------------------:|
| *Kalman filter smoothing of EE orientation (Roll, Pitch, Yaw).* |

---

##### Remarks
- The filter efficiently reduces noise while maintaining responsiveness.  
- Increasing **Q** makes the filter react faster but amplifies noise.  
- Increasing **R** yields smoother motion but slower response.  
- Ideal balance produces stable and natural robot movement.

#### Adaptive Motion Scaling Strategy
- **Purpose:**  
  Present the adaptive motion scaling strategy implemented for the **Interbotix VX300s** robot arm.  
  The method integrates **orientation clamping** and a **dynamic hand-to-robot mapping algorithm** that scales end-effector (EE) positions for both precision and workspace coverage.  
  While tailored to the VX300s, the approach generalises to other robot platforms.

---

##### Orientation Clamping

- **Concept:**  
  According to the Interbotix VX300s specifications, the EE has rotation limits:

  $$
  \text{roll} \in [-180^\circ,\, 180^\circ], \quad
  \text{pitch} \in [-107^\circ,\, 130^\circ], \quad
  \text{yaw} \in [-180^\circ,\, 180^\circ].
  $$

- **Problem:**  
  When the human hand tilts near these bounds (e.g. upward pitch), the **IK solver** may produce abrupt changes in joint configuration, causing discontinuous and unsafe robot motion.

- **Solution:**  
  Apply **orientation clamping**:
  - roll ∈ [−160°, +160°]  
  - pitch, yaw ∈ [−85°, +85°]  
  This prevents unsafe or discontinuous IK results while maintaining natural orientation fidelity.

- **Effect:**  
  Ensures continuous, stable, and safe control during teleoperation while preserving sufficient rotation for manipulation tasks.

---

##### Dynamic Hand-to-Robot Mapper

- **Objective:**  
  Scale human hand displacements into robot EE positions with dynamic responsiveness — precise near the ground, fast when elevated.

- **Illustration:**  
  The robot’s mapped motion along the X, Y, and Z axes is shown below.  
  Configured parameters are summarised in *Appendix Dynamic Mapper Configuration*.

| ![Mapping X](/assets/img/mapping_x.png) ![Mapping Y](/assets/img/mapping_y.png) ![Mapping Z](/assets/img/mapping_z.png) |
|:----------------------------------------------------------------------------------------------------------------------:|
| *Mapping human hand displacements to robot EE positions along the X, Y, and Z axes using the dynamic mapper.* |

---

##### Dynamic Hand-to-Robot Mapping Algorithm

The mapping from the measured hand position $h_t$ to the commanded EE position $p_t$ proceeds through the following steps:

---

###### Step 1: Smoothed Hand Motion Magnitude

**Purpose:**  
Estimate a smoothed motion magnitude $\tilde{m}_t$ to stabilise velocity estimation and suppress jitter.

**Formula:**
$$
m_t = \|h_t - h_{t-1}\|, \qquad
\tilde{m}_t = (1-\beta)\tilde{m}_{t-1} + \beta m_t
$$

**Parameter effect:**  
- Small β → smoother but slower response  
- Large β → more responsive but jittery

---

###### Step 2: Height-Dependent Displacement Range

**Purpose:**  
Adapt displacement ranges with hand height for precise low-level control and freer motion when raised.

**Formula:**
$$
z_{\text{cur}} = z_h + (h_{t,z} - h_{0,z})
$$
$$
\tau_z^{\text{clamp}} = \min\!\Big(\max\!\big(\tfrac{z_{\text{cur}} - z_{\min}^R}{z_r - z_{\min}^R}, 0\big), 1\Big)
$$
$$
\Delta x^H =
\big[\text{lerp}(\Delta x_{\min}^-, \Delta x^-, \tau_z^{\text{clamp}}),
     \text{lerp}(\Delta x_{\min}^+, \Delta x^+, \tau_z^{\text{clamp}})\big]
$$
$$
\Delta y^H =
\big[\text{lerp}(\Delta y_{\min}^-, \Delta y^-, \tau_z^{\text{clamp}}),
     \text{lerp}(\Delta y_{\min}^+, \Delta y^+, \tau_z^{\text{clamp}})\big]
$$
$$
\Delta z^H = [\Delta z^-, \Delta z^+]
$$
with $\text{lerp}(a,b,t) = (1-t)a + tb,\ t \in [0,1]$.

**Parameter effect:**  
- Smaller $(\Delta x_{\min}, \Delta y_{\min})$: high precision near ground  
- Larger $(\Delta x_{\min}, \Delta y_{\min})$: faster but less stable  
- Larger $z_r$: extended fine-control region  

---

###### Step 3: Hand Displacement Clamping

**Purpose:**  
Clip raw hand displacements to safe workspace limits.

**Formula:**
$$
\delta_x = \text{clip}(h_{t,x}-h_{0,x},\,\Delta x^H), \quad
\delta_y = \text{clip}(h_{t,y}-h_{0,y},\,\Delta y^H), \quad
\delta_z = \text{clip}(h_{t,z}-h_{0,z},\,\Delta z^H)
$$
with $\text{clip}(x,[a,b]) = \min(\max(x,a),b)$.

---

###### Step 4: Base Axis Scaling

**Purpose:**  
Map hand displacements to robot motions using scaling factors $s_x, s_y, s_z$.

**Formula:**
$$
d_x = s_x\,\delta_x, \quad
d_y = s_y\,\delta_y, \quad
d_z = s_z\,\delta_z
$$

---

###### Step 5: Height-Dependent Scaling Adjustment

**Purpose:**  
Increase scaling when the hand is raised above $z_r$, capped by maximum limits.

**Formula:**
$$
\hat{s}_y = \min\!\big(s_y e^{(z_{\text{cur}}-z_r)},\, s_{y,\max}\big)
$$
$$
\hat{s}_x = \min\!\big(s_x e^{(z_{\text{cur}}-z_r)},\, s_{xz,\max}\big)
$$
$$
\hat{s}_z = \min\!\big(s_z e^{(z_{\text{cur}}-z_r)},\, s_{xz,\max}\big)
$$

---

###### Step 6: Pitch-Dependent Vertical Scaling

**Purpose:**  
Boost vertical response when the hand tilts downward.

**Formula:**
$$
b_\theta =
\begin{cases}
\text{clip}\!\left(1 + k_\theta \tfrac{\theta - \theta_{\text{start}}}{50},\, 1,\, k_{\theta,\max}\right), & \theta > \theta_{\text{start}} \\
1, & \text{otherwise}
\end{cases}
$$

---

###### Step 7: Ground Slowdown Scaling

**Purpose:**  
Reduce vertical speed near the ground to prevent collisions.

**Formula:**
$$
\sigma_z =
\begin{cases}
1 - \lambda_s \left(1 - \tfrac{z_{\text{cur}} - (z_s-\Delta z_s)}{\Delta z_s}\right), & z_{\text{cur}} < z_s \\
1, & z_{\text{cur}} \ge z_s
\end{cases}
$$

---

###### Step 8: Motion Thresholding

**Purpose:**  
Ignore tiny hand jitters and limit excessive motion.

**Formula:**
$$
\hat{m}_t = \min(\max(\tilde{m}_t, m_{\min}), m_{\max})
$$

---

###### Step 9: Resistance Multiplier Computation

**Purpose:**  
Adjust resistance $\rho_t$ based on hand motion magnitude for fine vs. coarse control.

**Formula:**
$$
\rho_t =
\begin{cases}
\rho_{\max}, & \hat{m}_t \le m_{\min} + \delta_\rho \\
\rho_{\min}, & \hat{m}_t \ge m_{\max} - \delta_\rho \\
\text{lerp}(\rho_{\max}, \rho_{\min}, \tfrac{\hat{m}_t - m_{\min}}{m_{\max}-m_{\min}}), & \text{otherwise}
\end{cases}
$$

---

###### Step 10: Combined Scaling Application

**Purpose:**  
Combine all scaling, resistance, and slowdown factors into final robot displacements.

**Formula:**
$$
r_x = \rho_t \hat{s}_x d_x, \quad
r_y = \rho_t \hat{s}_y d_y, \quad
r_z = \rho_t \hat{s}_z b_\theta \sigma_z d_z
$$

---

###### Step 11: Position Smoothing

**Purpose:**  
Stabilise final robot trajectory using exponential smoothing.

**Formula:**
$$
\tilde{\mathbf{p}}_t^R = (1-\alpha)\tilde{\mathbf{p}}_{t-1}^R + \alpha \mathbf{p}_t^R
$$

**Effect:**  
- Small α → smooth but lagging  
- Large α → fast but noisy  

---

##### Summary
The adaptive mapper continuously blends:
- **Height-dependent scaling**  
- **Pitch-aware amplification**  
- **Ground slowdown**  
- **Motion resistance and smoothing**

to achieve **stable, intuitive, and safe teleoperation** across the robot’s full workspace.

