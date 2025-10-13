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
  color: #1a1a1a;
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

---

<heading>Motion Retargeting</heading>

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
