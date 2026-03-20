# 6dof-pose-estimation-pcd

> Point-cloud-based zero-shot 6-DoF pose estimation package for ROS.

This package estimates the 6-DoF pose of a target object from a filtered point cloud using clustering and PCA-based orientation analysis. It is designed to be integrated into an existing perception pipeline rather than used as a standalone detector.

[![ROS Noetic](https://img.shields.io/badge/ROS-Noetic-blue)](http://wiki.ros.org/noetic)
[![Paper](https://img.shields.io/badge/Paper-AEJ-blue)](https://doi.org/10.1016/j.aej.2025.12.015)
[![Demo](https://img.shields.io/badge/Demo-YouTube-red?logo=youtube)](https://youtu.be/58YqAvVOr8E)
[![Project](https://img.shields.io/badge/Project-Notion-black?logo=notion)](https://www.notion.so/Autonomous-charging-system-via-manipulator-UGV-docking-using-zero-shot-6-DoF-pose-estimation-3297602386468098bb55e4f2c04444df?source=copy_link)
[![Docker](https://img.shields.io/badge/Docker-jmk9%2Fdocking--desktop-blue?logo=docker)](https://hub.docker.com/repository/docker/jmk9/docking-desktop/general)


---

## 🎥 Demo

[![Demo Video](https://img.youtube.com/vi/58YqAvVOr8E/0.jpg)](https://youtu.be/58YqAvVOr8E)

**Demo video:** [Zero-shot 6-DoF pose estimation with filtered point cloud](https://youtu.be/58YqAvVOr8E)

---

## 📌 Overview

This package consumes a point cloud that has already been filtered to contain only the target object and estimates a stable 6-DoF pose from it.

It is intended for integration with an upstream perception stack such as:

- RGB-D camera
- 2D object detector (e.g., YOLO-based detector)
- RGB-D / point cloud fusion module
- point-cloud filtering module for extracting the target object region

A typical use case is **zero-shot drogue pose estimation for docking**, where the detector identifies the target region and this package recovers its 3D pose from the corresponding point cloud.

---

## ⚙️ Key Features

- Point-cloud-based **zero-shot 6-DoF pose estimation**
- Clustering-based object point extraction refinement
- Orientation estimation using **PCA** and **weighted PCA**
- ROS topic-based integration with existing perception stacks
- Optional visualization support for **RViz**
- Offline comparison scripts for clustering and PCA variants

---

## 🧩 System Integration

This package is **not a standalone full perception pipeline**.  
It assumes that another module already provides a point cloud containing mainly the target object.

### Expected upstream pipeline

1. An RGB-D camera publishes depth / point cloud data
2. A 2D detector publishes object detections
3. An external fusion or filtering module extracts the target-object point cloud
4. This package estimates the target 6-DoF pose from that filtered point cloud

In other words, this package focuses on **pose estimation from filtered 3D data**, not on object detection itself.

---

## 📦 Dependencies

### Environment

- ROS Noetic
- Ubuntu 20.04

### ROS dependencies

See `package.xml` for the complete dependency list. Major runtime dependencies include:

- `rospy`
- `sensor_msgs`
- `geometry_msgs`
- `std_msgs`
- `vision_msgs`
- `message_filters`
- `detection_msgs` (for `BoundingBoxes`)

### Python packages

- `numpy`
- `scipy`
- `opencv-python`
- `open3d`

---

## 🔧 Installation

```bash
cd ~/catkin_ws/src
git clone https://github.com/<your-id>/pcd-6dof-pose.git
cd ~/catkin_ws
rosdep install --from-paths src --ignore-src -r -y
catkin_make
source devel/setup.bash
```

---

## 🚀 Launch

### Main launch file

```bash
roslaunch pcd_6dof_pose pcd_6dof.launch
```

---

## 🔌 Input and Output

### Expected input

This package typically expects a filtered target point cloud such as:

- `/detection/lidar_detector/yolo_objects_pointcloud` (`sensor_msgs/PointCloud2`)

This topic can be remapped depending on your system configuration.

If your pipeline uses 2D detections as auxiliary input, those detections are generally produced upstream and are **not** the primary output of this package.

### Main output

Typical outputs may include:

- estimated target pose (`geometry_msgs/Pose` or `geometry_msgs/PoseStamped`, depending on implementation)
- visualization markers / intermediate data for RViz
- saved point cloud / orientation results for offline analysis

You should verify the exact published topics and message types in the launch file and source code used in your setup.

---

## 🧪 How to Run in an Integration Scenario

This package assumes the following modules are already running:

- an RGB-D camera node publishing depth / point cloud data
- a 2D object detector publishing detections
- an external fusion/filtering node producing a target-only point cloud

Once the filtered point cloud is available, run this package and connect the topic names to your own perception stack through remapping if necessary.

Example integration flow:

- camera point cloud → upstream filtering module
- detector result + depth fusion → target-only point cloud
- target-only point cloud → `6dof-pose-estimation-pcd`
- estimated pose → downstream docking / manipulation / tracking module

---

## 📁 Repository Structure

```text
src/
├── pose_estimator_node.py      # Main point-cloud-based 6-DoF pose estimator
├── compare_pca.py              # PCA vs weighted PCA orientation comparison
└── compare_clustering.py       # Clustering comparison (DBSCAN, Agglomerative, HDBSCAN)

launch/
└── pcd_6dof.launch            # Launch file for the pose estimation pipeline

cluster/                        # Saved clustered point clouds (.npy)
pca_orientation/                # Saved PCA orientation matrices
wpca_orientation/               # Saved weighted PCA orientation matrices
```

---

## 📝 Notes

- This package is designed to be plugged into an existing perception stack, such as a camera + detector + RGB-D fusion pipeline.
- Frame names such as `camera_depth_optical_frame` and `world` must be consistent with your TF tree.
- DBSCAN thresholds such as `eps` and `min_samples`, as well as z-filtering thresholds, were tuned for a specific setup and may need adjustment in other environments.
- The quality of the estimated pose strongly depends on the quality of the upstream filtered point cloud.
- If the input point cloud includes substantial background noise or partial target observations, clustering and orientation stability may degrade.

---

## 📄 Paper

[https://doi.org/10.1016/j.aej.2025.12.015](https://doi.org/10.1016/j.aej.2025.12.015)

---

## 📚 Citation

```
@article{JUNG2026122,
title = {Autonomous charging system via manipulator-UGV docking using zero-shot 6-DoF pose estimation},
journal = {Alexandria Engineering Journal},
volume = {134},
pages = {122-134},
year = {2026},
issn = {1110-0168},
doi = {https://doi.org/10.1016/j.aej.2025.12.015},
url = {https://www.sciencedirect.com/science/article/pii/S1110016825011871},
author = {Minkyu Jung and Andrew Jaeyong Choi},
keywords = {Autonomous docking, 6-doF pose estimation, Open-vocabulary object detection, Point cloud localization, Manipulator inverse kinematics, ROS-based integration},
abstract = {To ensure long-term field operation of Unmanned Ground Vehicles (UGVs) without human intervention, autonomous charging capability is essential. This paper presents a fully autonomous charging system that enables a UGV to perform path planning, align itself with the charging station, and a 6-DoF manipulator to locate and connect to the charging port without relying on external markers. By combining vision-language detection with geometric reasoning on RGB-D data, the system estimates a complete 6-DoF pose without relying on fiducial markers or task-specific retraining, thereby enhancing adaptability across different docking environments. Specifically, it employs the open-vocabulary object detector YOLO-World to identify the docking port in the RGB image and selects the corresponding 3D position from the depth map. To estimate orientation, the surrounding point cloud is clustered to extract the object shape, from which the without relying on fiducial markers or task-specific retraining, thereby enhancing adaptability across different docking environments. principal axis is computed, resulting in a fully generalizable and marker-free docking pipeline. This 6-DoF pose is used to compute a feasible manipulator configuration through inverse kinematics. The proposed pipeline is integrated into a ROS-based framework and enables precise docking in unstructured outdoor environments without manual supervision. Experimental results demonstrate the robustness and accuracy of the system under real-world conditions, achieving a docking success rate of 90% and an average total operation time of 34.4 s.}
}
```

---

## 👤 Author

- **Minkyu Jung** — first author
- **Andrew Jaeyong Choi** — advisor
