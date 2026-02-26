---
layout: page
title: Real-Time Vision-Based Hand Tracking for Robot Teleoperation and Learning
img: /assets/img/robot_teleoperation/robot_teleoperate.png
importance: 3
category: work
related_publications: false
---

Imperial MSc. Robotics Project, supervised by <a href="https://www.doc.ic.ac.uk/~ajd/" target="_blank">Prof. Andrew Davison</a>

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

<em class="figure-caption">
  Demonstration of teleoperation tasks performed with the physical VX300s robot arm by RGB-based hand tracking in real time.
</em>

---

<heading>Robot Imitation Learning</heading>

<div class="embed-responsive embed-responsive-16by9 mt-3 mb-3">
  <iframe class="embed-responsive-item"
          src="https://drive.google.com/file/d/1xn9Pt2lOfbbGdhPWujNqPAoEyKmrKA7e/preview"
          allow="autoplay; encrypted-media"
          allowfullscreen>
  </iframe>
</div>

<em class="figure-caption">
  One-shot imitation learning demonstration showing the physical robot replicating the human-demonstrated task in 10 different scenes.
</em>

---

<heading>System Design</heading>
The overall workflow of the proposed **real-time robot teleoperation system** is as follows:

1. **Hand Tracking** — Captures RGB images in real time and computes 3D hand keypoints using **MediaPipe** and **HaMeR**.  
2. **Motion Retargeting** — Transforms hand keypoints from the **MANO frame** to the **Robot frame**, applies **Kalman filtering** for temporal smoothing, and performs **adaptive motion scaling** for natural control.  
3. **Robot Manipulation & Visualization** — Executes the mapped motions on the **physical robot arm**, runs simulations in **Gazebo**, and renders visual feedback using **Rerun**.

<p align="center">
  <img src="/assets/img/robot_teleoperation/project_framework.png" alt="System Framework" width="80%">
  <br>
  <em class="figure-caption">Project Framework</em>
</p>

The architecture consists of three main components:

1. **Vision Processor** — Handles RGB frame analysis and hand keypoint extraction.  
2. **Robot Controller** — Generates joint commands via a **dynamic hand-to-robot mapper** for smooth and precise control.  
3. **Scale Tuning GUI** — Allows real-time adjustment of motion scaling for user-specific responsiveness.

This design achieves an overall data transmission rate of 30 Hz, satisfying the requirements for real-time teleoperation.

<p align="center">
  <img src="/assets/img/robot_teleoperation/multi_process_thread.png" alt="Multiprocessing and Multithreading Design" width="80%">
  <br>
  <em class="figure-caption">Multiprocessing and Multithreading in the system design</em>
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
  <img src="/assets/img/robot_teleoperation/hamer_pipeline.png" alt="Hamer Pipeline" width="80%">
  <br>
  <em class="figure-caption">Fig. 1: Hand Tracking Pipeline</em>
</p>

<p align="center">
  <img src="/assets/img/robot_teleoperation/hand_structure.png" alt="Hand Structure" width="30%">
  <br>
  <em class="figure-caption">Fig. 2: Illustration of the MANO hand skeleton model, with labelled joints. This project uses joints 2, 4, 5, and 8.</em>
</p>

**Real-Time Multiprocessing** — **MediaPipe** and **HaMeR** run in separate processes to ensure **real-time performance** and low-latency execution.

---

<heading>Motion Retargeting</heading>

The motion retargeting pipeline translates human hand motion—captured from 3D keypoints—into corresponding robot end-effector (EE) commands for intuitive, stable, and responsive teleoperation. It integrates hand pose extraction, coordinate transformation, motion decoupling, temporal filtering, and adaptive scaling.

---

#### **1. Human-to-Robot Cartesian Position and Orientation Transformation**

**Purpose:**  
To map human hand movements and orientations to the robot’s gripper in a physically meaningful and intuitive way.

**Process Overview:**  
- The thumb and index fingertips correspond to the robot’s left and right gripper fingers.  
- The midpoint between them defines the gripper centre.  
- A local hand coordinate frame is constructed using keypoints, forming a right-handed basis representing the hand’s orientation.  
- A fixed transformation matrix maps this hand frame to the robot’s Cartesian frame, ensuring consistent spatial interpretation.  
- Both position and orientation are transformed accordingly so that hand motion directly drives the robot’s end-effector (EE).

**Outcome:**  
A stable and intuitive spatial mapping between human gestures and robot motion.

---

#### **2. End-Effector Position and Orientation Estimation**

**Position:**  
- Compute displacement relative to the initial reference frame.  
- Apply purity weighting and Kalman filtering to smooth trajectories and suppress noise.  
- Combine filtered displacements with the starting pose for stable EE positioning.

**Orientation:**  
- Estimate rotation changes from the hand’s local frame relative to its initial orientation.  
- Convert to Euler angles, unwrap discontinuities, filter, and then convert to quaternions for IK input.

**Outcome:**  
Smooth, continuous, and physically consistent EE trajectories.

---

#### **3. Gripper State Estimation**

**Definition:**  
- The gripper state (open or closed) is inferred from the distance between the thumb and index fingertips.  
- Thresholds classify gestures into open or closed gripper commands.

**Outcome:**  
Natural, fingertip-based control of the robot’s gripper.

---

#### **4. Purity-Coupled Translation and Rotation**

**Motivation:**  
Human motion couples translation and rotation, which can cause unintended EE motion.

**Approach:**  
- Compute translation and rotation magnitudes per frame.  
- Derive purity scores representing the dominant motion type.  
- Use dynamic weighting: suppress translation during rotational gestures and downweight rotation during translational ones.

**Effect:**  
Prevents cross-contamination between motion modes, improving precision and intuitiveness.

---

#### **5. Kalman Filtering for Smoothed Motion**

**Purpose:**  
To smooth noisy visual input and produce stable EE motion.

**Method:**  
- Apply separate Kalman filters for translation and rotation under a constant-velocity model.  
- Fuse predicted motion and measured increments adaptively using tuned covariance matrices.  
- Apply purity weighting to independently filter translation and rotation.

**Outcome:**  
Clean, temporally consistent trajectories suitable for real-time control.

<p align="center">
  <img src="/assets/img/robot_teleoperation/position_x_kf.png" alt="Position X" width="32%">
  <img src="/assets/img/robot_teleoperation/position_y_kf.png" alt="Position Y" width="32%">
  <img src="/assets/img/robot_teleoperation/position_z_kf.png" alt="Position Z" width="32%">
  <br>
  <em class="figure-caption">Kalman filter smoothing of EE positions (X, Y, Z).</em>
</p>

<p align="center">
  <img src="/assets/img/robot_teleoperation/roll_kf.png" alt="Roll" width="32%">
  <img src="/assets/img/robot_teleoperation/pitch_kf.png" alt="Pitch" width="32%">
  <img src="/assets/img/robot_teleoperation/yaw_kf.png" alt="Yaw" width="32%">
  <br>
  <em class="figure-caption">Kalman filter smoothing of EE orientation (Roll, Pitch, Yaw).</em>
</p>

---

#### **6. Adaptive Motion Scaling Strategy**

**Objective:**  
To map human hand movements to the robot’s workspace dynamically, ensuring
    - precise near the ground and faster at higher elevations
    - maintaining smooth, safe motion.

##### **(a) Orientation Clamping**  
Limits roll, pitch, and yaw within safe ranges to prevent unstable IK solutions and abrupt joint changes.

##### **(b) Dynamic Hand-to-Robot Mapping Algorithm**

The hand-to-EE mapping follows eleven adaptive stages:

1. **Smoothed Motion Magnitude** – Stabilise frame-to-frame hand motion using exponential smoothing.  
2. **Height-Dependent Range** – Dynamically adjust workspace range based on hand height.  
3. **Displacement Clamping** – Restrict hand displacement within safe bounds.  
4. **Base Axis Scaling** – Scale clamped hand motion per Cartesian axis.  
5. **Height-Dependent Scaling** – Increase responsiveness with elevation for larger workspace coverage.  
6. **Pitch-Dependent Scaling** – Adjust Z-axis sensitivity based on wrist pitch angle.  
7. **Ground Slowdown** – Dampen vertical motion near the table or base for safety.  
8. **Motion Thresholding** – Filter out negligible motion and cap excessive input.  
9. **Resistance Multiplier** – Apply adaptive damping based on motion intensity for stability.  
10. **Combined Scaling** – Integrate all scaling effects into final per-axis motion commands.  
11. **Position Smoothing** – Apply temporal smoothing for steady and continuous EE motion.

<p align="center">
  <img src="/assets/img/robot_teleoperation/mapping_x.png" alt="Mapping X" width="32%">
  <img src="/assets/img/robot_teleoperation/mapping_y.png" alt="Mapping Y" width="32%">
  <img src="/assets/imgrobot_teleoperation/mapping_z.png" alt="Mapping Z" width="32%">
  <br>
  <em class="figure-caption">Mapping human hand displacements to robot EE positions along the X, Y, and Z axes.</em>
</p>

**Outcome:**  
The mapping achieves precise and adaptive control—stable near the ground, responsive mid-air, and safely bounded within the robot’s workspace.

---

#### **Overall Summary**

The complete motion retargeting pipeline combines geometric mapping, motion decoupling, temporal filtering, and adaptive scaling.  
It ensures that human hand gestures are translated into **smooth, intuitive, and physically consistent robot end-effector motion**, enabling natural teleoperation and robust data collection for learning-based manipulation.

