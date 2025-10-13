---
layout: page
title: Real-Time Vision-Based Hand Tracking for Robot Teleoperation and Learning
description: Imperial MSc. Robotics Project, supervised by Prof. Andrew Davison
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
T和he figure below illustrates the hand tracking pipeline:

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
  \( \mathcal{M} = \{ M[0], \dots, M[20] \} \) — 21 3D hand keypoints from HaMeR.

- **Gripper Centre:**  
  \[
  p_\text{mid} = \frac{M[2] + M[5]}{2}, \quad
  p_\text{gc} = \frac{p_\text{mid} + M[4] + M[8]}{3}
  \]

- **Orientation Axes:**  
  \[
  \hat{x} = \frac{p_\text{index} - p_\text{thumb}}{\|p_\text{index} - p_\text{thumb}\|}, \quad
  \hat{z} = \frac{(p_\text{index} - p_\text{mid}) \times (p_\text{thumb} - p_\text{mid})}{\|(p_\text{index} - p_\text{mid}) \times (p_\text{thumb} - p_\text{mid})\|}, \quad
  \hat{y} = \frac{\hat{z} \times \hat{x}}{\|\hat{z} \times \hat{x}\|}
  \]

- **Remarks:**  
  These form a right-handed coordinate frame representing the hand’s local orientation.

---

##### Human-to-Robot Transformation
- **Axis Mapping:**  
  \[
  \hat{x}_R = -\hat{z}_H,\;
  \hat{y}_R = \hat{x}_H,\;
  \hat{z}_R = -\hat{y}_H
  \]
  Matrix form:  
  \[
  A_{R \leftarrow H} =
  \begin{bmatrix}
  0 & 0 & -1\\
  1 & 0 &  0\\
  0 & -1 &  0
  \end{bmatrix}
  \]

- **Position Mapping:**  
  \[
  p_R = A_{R \leftarrow H}\, p_H
  \]
  Maps relative displacements as  
  \((z_H, x_H, y_H) \mapsto (-x_R, y_R, -z_R)\).

- **Orientation Mapping:**  
  \[
  R_R = A_{R \leftarrow H}\, R_H
  \]

<p align="center">
  <img src="/assets/img/rotation_full.png" alt="Rotation_Full" width="80%">
  <br>
  <em class="figure-caption">Illustration of the human-to-robot position and orientation transformation</em>
</p>

---

##### End-Effector Position and Orientation
- **Position Computation:**  
  \[
  \Delta p_t = p_t^{meas} - p_0,\quad
  \widetilde{\Delta p}_t = \Pi_t \Delta p_t,\quad
  \widehat{\Delta p}_t = \mathcal{K}[\widetilde{\Delta p}_t],\quad
  \tilde{p}^{ee}_t = g(p_0 + \widehat{\Delta p}_t)
  \]
  Purity weighting and Kalman filtering smooth motion and suppress noise.

- **Orientation Computation:**  
  \[
  R_\Delta = R_R\, R_0^\mathsf{T}
  \]
  Convert to Euler angles → unwrap → apply purity weighting and Kalman filter →  
  convert to quaternion \( q^{ee}_t = \operatorname{Quat}_{xyz}(\widehat{\theta}_t) \).

- **Notes:**  
  - Angle unwrapping prevents discontinuities.  
  - Kalman filtering ensures smooth motion.  
  - Euler angles are intermediate; quaternions are used for IK.  

---

##### Gripper State
- **Definition:**  
  \[
  d_\text{grip} = \|p_\text{thumb} - p_\text{index}\|
  \]
  \[
  S_\text{grip} =
  \begin{cases}
  \text{Closed}, & d_\text{grip} < 0.08\text{ m}\\
  \text{Open}, & d_\text{grip} > 0.10\text{ m}
  \end{cases}
  \]

- **Remarks:**  
  The gripper state is determined from the thumb–index fingertip distance (in metres, camera frame).

#### Purity-Weighted Decoupling of Translation and Rotation
- Purpose:

#### Kalman Filter For Smoothed Motion
- Purpose:

#### Adaptive Motion Scaling Strategy

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
