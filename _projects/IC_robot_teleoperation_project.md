---
layout: page
title: Real-Time Vision-Based Hand Tracking for Robot Teleoperation and Learning
description: "Imperial MSc. Robotics Project, supervised by <a href=\"https://www.doc.ic.ac.uk/~ajd/\" target=\"_blank\">Prof. Andrew Davison</a>"
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
- **Purpose:** Map human hand movements to the robot’s two-finger gripper intuitively and stably.

- **Concept:** The thumb corresponds to the left gripper finger, and the index finger (or closed four fingers) represents the right finger.  
  In this project, only the **thumb–index pair** is used for simplicity and stability.

##### Human Hand Position and Orientation
- **Keypoints:** $\mathcal{M} = \{ M[0], \dots, M[20] \}$ — 21 3D hand keypoints from HaMeR.  
- **Gripper Centre:** \(p_\text{mid} = \frac{M[2] + M[5]}{2}, \quad p_\text{gc} = \frac{p_\text{mid} + M[4] + M[8]}{3}\).  
- **Orientation Axes:** \(\hat{x} = \frac{p_\text{index} - p_\text{thumb}}{\|p_\text{index} - p_\text{thumb}\|}, \quad \hat{z} = \frac{(p_\text{index} - p_\text{mid}) \times (p_\text{thumb} - p_\text{mid})}{\|(p_\text{index} - p_\text{mid}) \times (p_\text{thumb} - p_\text{mid})\|}, \quad \hat{y} = \frac{\hat{z} \times \hat{x}}{\|\hat{z} \times \hat{x}\|}\).

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
- **Axis Mapping:** $\hat{x}_R = -\hat{z}_H, \quad \hat{y}_R = \hat{x}_H, \quad \hat{z}_R = -\hat{y}_H$. 
  - Matrix form: $A_{R \leftarrow H} = \begin{bmatrix} 0 & 0 & -1 \\ 1 & 0 & 0 \\ 0 & -1 & 0 \end{bmatrix}$.  
- **Position Mapping:** $p_R = A_{R \leftarrow H} \, p_H$. 
  - Maps relative displacements as $(z_H, x_H, y_H) \mapsto (-x_R, y_R, -z_R)$.  
- **Orientation Mapping:** $R_R = A_{R \leftarrow H} \, R_H$.

<p align="center">
  <img src="/assets/img/rotation_full.png" alt="Rotation Full" width="50%">
  <br>
  <em class="figure-caption">Illustration of the human-to-robot position and orientation transformation.</em>
</p>

##### End-Effector Position and Orientation
- **Position Computation:** $\Delta p_t = p_t^{meas} - p_0, \quad \widetilde{\Delta p}_t = \Pi_t \Delta p_t, \quad \widehat{\Delta p}_t = \mathcal{K}[\widetilde{\Delta p}_t], \quad \tilde{p}^{ee}_t = g(p_0 + \widehat{\Delta p}_t)$. 
  - Purity weighting and Kalman filtering smooth motion and suppress noise.  
- **Orientation Computation:** $R_\Delta = R_R \, R_0^\mathsf{T}$. 
  - Convert to Euler angles → unwrap → apply purity weighting and Kalman filter → convert to quaternion $q^{ee}_t = \operatorname{Quat}_{xyz}(\widehat{\theta}_t)$.  
- **Notes:** Angle unwrapping prevents discontinuities. Kalman filtering ensures smooth motion. Euler angles are intermediate; quaternions are used for IK.

##### Gripper State
- **Definition:** $d_{grip} = \|p_{thumb} - p_{index}\|$, $S_{grip} = \begin{cases} \text{Closed}, & d_{grip} < 0.08~\text{m} \\ \text{Open}, & d_{grip} > 0.10~\text{m} \end{cases}$.  
- **Remarks:** The gripper state is determined from the thumb–index fingertip distance (in metres, camera frame).


#### Purity-Weighted Decoupling of Translation and Rotation
- **Purpose:** Reduce unintended coupling between translation and rotation during hand-controlled robot teleoperation.  
- **Problem:** Human hand motion naturally links position and orientation changes — when the user intends to rotate the hand, small translations often occur (and vice versa), leading to non-intuitive robot end-effector (EE) motion.  
- **Solution:** Introduce a *motion-purity mechanism* that computes translation–rotation purity scores at each time step and uses them as weights to decouple the two motion modes.

##### Purity Metrics
- **Translational and rotational magnitudes:** $m_t^{trans}=\|\Delta\mathbf{p}_t\|_2,\quad m_t^{rot}=\|\Delta\boldsymbol{\theta}_t\|_2$
- where 
  - $\Delta\mathbf{p}_t$: position increment
  - $\Delta\boldsymbol{\theta}_t$ : orientation increment (roll–pitch–yaw form)

##### Purity Scores
- **Definition:** $\pi_t^{trans}=\dfrac{m_t^{trans}}{m_t^{trans}+m_t^{rot}+\epsilon},\quad \pi_t^{rot}=\dfrac{m_t^{rot}}{m_t^{trans}+m_t^{rot}+\epsilon}$
- $\epsilon$ prevents division by zero and $\pi_t^{trans}+\pi_t^{rot}\approx1$

##### Weighted Decoupling
- **Mechanism:** If $\pi_t^{rot}\ge0.8$, translation increment $\Delta\mathbf{p}_t$ is **suppressed**, indicating primarily rotational motion. 
  - If $\pi_t^{trans}$ dominates, the rotational increment $\Delta\boldsymbol{\theta}_t$ is **down-weighted** but not fully suppressed.  
- **Effect:** Purity weights act as dynamic filters that separate translation and rotation, reducing cross-contamination.

##### Remarks
Enhances precision and intuitiveness in teleoperation, prevents small hand jitters from mixing translation–rotation motions, and complements Kalman filtering for smooth control performance.

---

#### Kalman Filter for Smoothed Motion
- **Purpose:** Apply Kalman filtering to smooth robot position and orientation, reducing noise from visual measurements.  
- **Background:** The **Kalman filter** (Kalman, 1960) estimates unobservable states from noisy observations by combining predictions from a motion model with new sensor data. 
  - It is widely used in robot vision to fuse uncertain visual inputs over time for accurate, stable motion estimates.  
- **Implementation:** A **constant-velocity Kalman filter** is applied separately to translation and rotation, operating on *incremental changes* rather than absolute poses.

##### Kalman Filtering for Relative Motion
- **Parameters:** Covariance matrices: 
  - **P** (state covariance) controls trust in current estimate
  - **Q** (process noise) controls model uncertainty
  - **R** (measurement noise) controls observation uncertainty
  - Larger **Q** → faster but noisier; larger **R** → smoother but slower
- **Purity weighting:** Translation and rotation are first decoupled via purity scores: 
  - $\widetilde{\Delta\mathbf{p}}_t=[\widetilde{\Delta p}_x,\widetilde{\Delta p}_y,\widetilde{\Delta p}_z]^\mathsf{T},\ \widetilde{\Delta\boldsymbol{\theta}}_t=[\widetilde{\Delta\phi},\widetilde{\Delta\theta},\widetilde{\Delta\psi}]^\mathsf{T}$.

##### Prediction Step
State vectors: $\mathbf{x}^p_t=[\widetilde{\Delta p}_x,v_x,\widetilde{\Delta p}_y,v_y,\widetilde{\Delta p}_z,v_z]^\mathsf{T}$ and $\mathbf{x}^r_t=[\widetilde{\Delta\phi},\omega_\phi,\widetilde{\Delta\theta},\omega_\theta,\widetilde{\Delta\psi},\omega_\psi]^\mathsf{T}$. Constant-velocity model: $\hat{\mathbf{x}}_{t|t-1}=\mathbf{F}\hat{\mathbf{x}}_{t-1|t-1},\ \mathbf{P}_{t|t-1}=\mathbf{F}\mathbf{P}_{t-1|t-1}\mathbf{F}^\mathsf{T}+\mathbf{Q}$, where $\mathbf{F}_{1D}=\begin{bmatrix}1&\Delta t\\0&1\end{bmatrix},\ \Delta t=0.01\,\text{s}$.

##### Correction Step (Direct Perception Input)
Measurement model: $\mathbf{z}_t=\mathbf{H}\mathbf{x}_t+\mathbf{v}_t,\ \mathbf{v}_t\!\sim\!\mathcal{N}(\mathbf{0},\mathbf{R})$, where $\mathbf{z}_t$ are increments from vision. Correction update: $\hat{\mathbf{x}}_{t|t}=\hat{\mathbf{x}}_{t|t-1}+\mathbf{K}_t(\mathbf{z}_t-\mathbf{H}\hat{\mathbf{x}}_{t|t-1}),\ \mathbf{P}_{t|t}=(\mathbf{I}-\mathbf{K}_t\mathbf{H})\mathbf{P}_{t|t-1}$.

##### Kalman Gain
$\mathbf{K}_t=\mathbf{P}_{t|t-1}\mathbf{H}^\mathsf{T}(\mathbf{H}\mathbf{P}_{t|t-1}\mathbf{H}^\mathsf{T}+\mathbf{R})^{-1}$. The Kalman gain adaptively balances prediction (motion model) and correction (sensor data), producing smooth, responsive position and orientation estimates.

##### Results
The filtered trajectories closely follow ground truth while removing high-frequency jitter from visual input.

<p align="center">
  <img src="/assets/img/position_x_kf.png" alt="Position X" width="32%">
  <img src="/assets/img/position_y_kf.png" alt="Position Y" width="32%">
  <img src="/assets/img/position_z_kf.png" alt="Position Z" width="32%">
  <br>
  <em class="figure-caption">Kalman filter smoothing of EE positions (X, Y, Z).</em>
</p>

<p align="center">
  <img src="/assets/img/roll_kf.png" alt="Roll" width="32%">
  <img src="/assets/img/pitch_kf.png" alt="Pitch" width="32%">
  <img src="/assets/img/yaw_kf.png" alt="Yaw" width="32%">
  <br>
  <em class="figure-caption">Kalman filter smoothing of EE orientation (Roll, Pitch, Yaw).</em>
</p>

##### Remarks
The filter reduces noise while maintaining responsiveness. Increasing **Q** speeds reaction but adds noise; increasing **R** smooths output but slows response. The tuned balance yields stable, natural robot motion.

---

#### Adaptive Motion Scaling Strategy
- **Purpose:** Present the adaptive motion-scaling strategy for the **Interbotix VX300s** robot arm. 
  - It integrates **orientation clamping** and a **dynamic hand-to-robot mapping algorithm** that scales end-effector (EE) positions for both precision and workspace coverage. 
  - Though tailored to the VX300s, the method generalises to other robot platforms.

##### Orientation Clamping
- The EE rotation limits are $\text{roll}\!\in\![-180^\circ,180^\circ],\ \text{pitch}\!\in\![-107^\circ,130^\circ],\ \text{yaw}\!\in\![-180^\circ,180^\circ]$. 
- Near these bounds (e.g. high pitch), the IK solver may produce abrupt joint changes. 
- Orientation clamping restricts roll, pitch, yaw to [−160°, +160°], [−85°, +85°], [−85°, +85°], preventing unsafe discontinuities while maintaining natural fidelity.

##### Dynamic Hand-to-Robot Mapper
Scales human hand displacements into robot EE positions with dynamic responsiveness — precise near the ground, faster when raised.

<p align="center">
  <img src="/assets/img/mapping_x.png" alt="Mapping X" width="32%">
  <img src="/assets/img/mapping_y.png" alt="Mapping Y" width="32%">
  <img src="/assets/img/mapping_z.png" alt="Mapping Z" width="32%">
  <br>
  <em class="figure-caption">Mapping human hand displacements to robot EE positions along the X, Y, and Z axes.</em>
</p>

##### Dynamic Hand-to-Robot Mapping Algorithm
The mapping from hand position $h_t$ to EE command $p_t$ proceeds through:  
1. **Smoothed Motion Magnitude:** $m_t=\|h_t-h_{t-1}\|,\ \tilde{m}_t=(1-\beta)\tilde{m}_{t-1}+\beta m_t$. Small β→smooth, large β→responsive.  
2. **Height-Dependent Range:** $z_{\text{cur}}=z_h+(h_{t,z}-h_{0,z}),\ \tau_z^{\text{clamp}}=\min(\max(\tfrac{z_{\text{cur}}-z_{\min}^R}{z_r-z_{\min}^R},0),1)$, then interpolate $\Delta x^H,\Delta y^H,\Delta z^H$ using `lerp(a,b,t)=(1-t)a+tb`.  
3. **Displacement Clamping:** $\delta_x=\text{clip}(h_{t,x}-h_{0,x},\Delta x^H),\ \delta_y=\text{clip}(h_{t,y}-h_{0,y},\Delta y^H),\ \delta_z=\text{clip}(h_{t,z}-h_{0,z},\Delta z^H)$ with `clip(x,[a,b])=min(max(x,a),b)`.  
4. **Base Axis Scaling:** $d_x=s_x\delta_x,\ d_y=s_y\delta_y,\ d_z=s_z\delta_z$.  
5. **Height-Dependent Scaling:** $\hat{s}_x=\min(s_x e^{(z_{\text{cur}}-z_r)},s_{xz,\max}),\ \hat{s}_y=\min(s_y e^{(z_{\text{cur}}-z_r)},s_{y,\max}),\ \hat{s}_z=\min(s_z e^{(z_{\text{cur}}-z_r)},s_{xz,\max})$.  
6. **Pitch-Dependent Scaling:** $b_\theta=\text{clip}(1+k_\theta\tfrac{\theta-\theta_{\text{start}}}{50},1,k_{\theta,\max})$ for $\theta>\theta_{\text{start}}$.  
7. **Ground Slowdown:** $\sigma_z=\begin{cases}1-\lambda_s(1-\tfrac{z_{\text{cur}}-(z_s-\Delta z_s)}{\Delta z_s}),&z_{\text{cur}}<z_s\\1,&z_{\text{cur}}\ge z_s\end{cases}$.  
8. **Motion Thresholding:** $\hat{m}_t=\min(\max(\tilde{m}_t,m_{\min}),m_{\max})$.  
9. **Resistance Multiplier:** $\rho_t=\begin{cases}\rho_{\max},&\hat{m}_t\le m_{\min}+\delta_\rho\\\rho_{\min},&\hat{m}_t\ge m_{\max}-\delta_\rho\\\text{lerp}(\rho_{\max},\rho_{\min},\tfrac{\hat{m}_t-m_{\min}}{m_{\max}-m_{\min}}),&\text{otherwise}\end{cases}$.  
10. **Combined Scaling:** $r_x=\rho_t\hat{s}_x d_x,\ r_y=\rho_t\hat{s}_y d_y,\ r_z=\rho_t\hat{s}_z b_\theta\sigma_z d_z$.  
11. **Position Smoothing:** $\tilde{\mathbf{p}}_t^R=(1-\alpha)\tilde{\mathbf{p}}_{t-1}^R+\alpha\mathbf{p}_t^R$, where small α→smooth/lag, large α→fast/noisy.

##### Summary
The adaptive mapper continuously blends height-dependent scaling, pitch-aware amplification, ground slowdown, and motion resistance smoothing to achieve stable, intuitive, and safe teleoperation across the robot’s workspace.
