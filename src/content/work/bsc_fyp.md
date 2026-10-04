---
title: B.Sc. Final Year Project
publishDate: 2021-09-21 00:00:00
img: /assets/images/bsc_fyp/self_driving_simple.png
img_alt: RV32I pipeline processor architecture
description: |
  Road Sign, Traffic Light and Static Object Detection for Self-Driving
start_date: "2021/09"
end_date: "2022/02"
tags:
  - ROS
  - NVIDIA Jetson AGX Xavier
  - Deep Learning
  - Computer Vision
  - Python
---

### Road Sign, Traffic Light and Static Object Detection for Self-Driving

📄 **[Read the paper (IEEE Xplore)](https://ieeexplore.ieee.org/document/10460046)**

💻 **[Code & dataset (GitHub)](https://github.com/harinduravin/DualCam)**

**In one line:** a real-time, end-to-end perception system that detects and tracks traffic lights and lane markings on Sri Lankan roads, and turns them into driver warnings. It runs on an NVIDIA Jetson AGX Xavier inside a car and uses two Sri Lankan datasets we built ourselves.

---

#### At a glance

| | |
|---|---|
| **Goal** | Detect static objects (traffic lights, lanes, lane markings) while driving at normal speed on Sri Lankan roads, in real time and with high accuracy |
| **Approach** | Dual-camera traffic light detection (YOLOv5), lane detection with marking classification (UltraFast / LaneATT), Kalman-filter tracking, TensorRT optimisation |
| **Hardware** | Narrow + wide camera pair, NVIDIA Jetson AGX Xavier, vehicle battery with power inverter, mobile app for warnings |
| **Headline results** | **52.65 Hz** traffic light detection and **48.12 Hz** lane detection on the Jetson with TensorRT; **80.89 %** lane accuracy on our CeyLane test set |
| **Datasets** | **DualCam** (Sri Lankan traffic lights) and **CeyLane** (Sri Lankan lane markings) |
| **Tech** | YOLOv5 · UltraFast Lane Detection · LaneATT · Kalman filter / SORT · TensorRT · ROS · NVIDIA Jetson AGX Xavier |

![Project at a glance](/assets/images/bsc_fyp/images/project_at_a_glance.png)

---

#### The problem

Self-driving and driver-assistance systems depend on reliably spotting static road features. Doing that in Sri Lanka is harder than it looks:

- Models trained on foreign datasets perform poorly here because road conditions are different.
- Traffic lights are small, change with the lighting, look like other objects, and change state as you drive. That leads to many false negatives.
- Lane detection and tracking is difficult on diverse and often unmarked or worn roads.
- Several detection models have to run together on a resource-constrained device and still keep up in real time.

**Our aim:** design an end-to-end static object detection system that does real-time inference with high accuracy on Sri Lankan roads.

---

#### System overview

![System pipeline](/assets/images/bsc_fyp/images/system_pipeline.png)

Frames from the camera system go through two parallel branches on the Jetson AGX Xavier. One finds and tracks traffic lights, the other finds, classifies and tracks lane markings. The results are sent to a mobile app that shows the driver warnings.

##### Hardware

| Component | Details |
|---|---|
| Narrow-angle camera | Evetar lens, 36° × 60° field of view (vertical × horizontal) |
| Wide-angle camera | Theia lens, 86° × 120° field of view (vertical × horizontal) |
| Camera sensor | Basler daA1920-30uc, 30 FPS |
| Compute | NVIDIA Jetson AGX Xavier |
| Power | Vehicle battery, with a power inverter supplying 240 V |
| Output | Mobile app connected to the Jetson over USB |

| Camera unit | In-vehicle setup | Enclosure design |
|:---:|:---:|:---:|
| ![Camera unit](/assets/images/bsc_fyp/images/camera_unit.png) | ![Camera mounted in the vehicle](/assets/images/bsc_fyp/images/in_vehicle_setup.png) | ![Camera enclosure CAD](/assets/images/bsc_fyp/images/camera_enclosure_cad.png) |

| Jetson AGX Xavier | Power inverter |
|:---:|:---:|
| ![NVIDIA Jetson AGX Xavier](/assets/images/bsc_fyp/images/jetson_agx_xavier.png) | ![Power inverter](/assets/images/bsc_fyp/images/power_inverter.png) |

---

#### Datasets

Because foreign datasets did not transfer well, we collected and annotated two Sri Lankan datasets on urban and suburban roads around Colombo.

| | **DualCam** | **CeyLane** |
|---|---|---|
| Purpose | Traffic light detection | Lane and lane-marking detection |
| Classes | 10 traffic light classes | 8 lane-marking classes |
| Training set | 1,032 images (776 narrow-angle, 256 wide-angle) | 1,729 images |
| Test set | 1,626 images (813 simultaneous narrow + wide pairs) | 740 images |
| Annotation format | n/a | TuSimple format, compatible with modern SOTA lane detectors |

**DualCam classes:** green, red, yellow, green-up, green-left, green-right, red-yellow, count-down, empty-count-down, empty.

| DualCam samples | CeyLane samples |
|:---:|:---:|
| ![DualCam sample 1](/assets/images/bsc_fyp/images/dataset_dualcam_1.jpg) | ![CeyLane sample 1](/assets/images/bsc_fyp/images/dataset_ceylane_1.jpg) |
| ![DualCam sample 2](/assets/images/bsc_fyp/images/dataset_dualcam_2.jpg) | ![CeyLane sample 2](/assets/images/bsc_fyp/images/dataset_ceylane_2.jpg) |

---

#### Traffic light detection

##### Method

1. A synchronised **narrow-angle** camera (sees far) and **wide-angle** camera (sees near) feed one **YOLOv5** detector as a batch of two frames.
2. Detections from the narrow view are mapped into the wide frame using a **planar homography** and merged with the wide-camera detections.
3. The combined detections are tracked with a **Kalman-filter based SORT** tracker.
4. The tracked output drives the driver indications.

Using two cameras lets the system pick up traffic lights from much farther away than a single wide camera would.

![Traffic light detection example 1](/assets/images/bsc_fyp/images/traffic_light_detection_1.jpg)
![Traffic light detection example 2](/assets/images/bsc_fyp/images/traffic_light_detection_2.jpg)

##### Results

- Detects lights from far away, down to around 1-pixel-sized bounding boxes, with high recall.
- The dual-camera technique significantly reduced false negatives.
- The post-processing step adds only about 1 ms on an RTX 2080 Ti and about 5 ms on the Jetson AGX Xavier, so overall speed is barely affected.

The precision-recall curves below compare the narrow camera, the wide camera and the combined output. Combining both cameras reaches clearly higher recall.

![Precision-recall curves for narrow, wide and combined detection](/assets/images/bsc_fyp/images/precision_recall.png)

**Class-wise F1 scores on the test set**

| Class | YOLOv5s (FP16) | YOLOv5l (FP16) |
|---|---:|---:|
| Red | 72.03 | 83.56 |
| Green-arrows | 65.03 | 74.5 |
| Yellow | 59.39 | 72.36 |
| Count-down | 59.39 | 68.62 |
| Empty-count-down | 52.65 | 65.05 |
| Green | 49.55 | 62.04 |
| Red-yellow | 36.36 | 55.56 |
| Empty | 38.25 | 48.59 |

Count-down and empty-count-down classes score lower because they are smaller than a normal traffic light. Classes such as empty, red-yellow, green-left, green-right and green-up have few examples, which also lowers their F1.

**Speed: combined frame vs individual frames**

| Model | RTX 2080 Ti, individual | RTX 2080 Ti, combined | Jetson AGX Xavier, individual | Jetson AGX Xavier, combined |
|---|---:|---:|---:|---:|
| YOLOv5s | 416.7 Hz | 256.4 Hz | 57.5 Hz | 42.8 Hz |
| YOLOv5l | 196.1 Hz | 117.6 Hz | 29.5 Hz | 16.5 Hz |

---

#### Lane detection and tracking

##### Method

1. **UltraFast** and **LaneATT** lane detectors (ResNet-18 backbone) were extended with a **classifier** that labels the type of each lane marking (solid, dashed, double, and so on).
2. Lane lines are tracked over time with a **Kalman-filter** based algorithm.
3. The tracker output is used to build a **lane departure warning**.

| Lane tracking example 1 | Lane tracking example 2 |
|:---:|:---:|
| ![Lane tracking 1](/assets/images/bsc_fyp/images/lane_tracking_1.jpg) | ![Lane tracking 2](/assets/images/bsc_fyp/images/lane_tracking_2.jpg) |

##### Results

Training on our own CeyLane dataset gives clearly better results on Sri Lankan roads than training on the CuLane dataset:

| Detector | Backbone | Trained on | Accuracy | False positives | False negatives |
|---|---|---|---:|---:|---:|
| UltraFast | ResNet-18 | CuLane | 72.38 % | 56.38 % | 49.09 % |
| UltraFast | ResNet-18 | **CeyLane** | 77.22 % | 48.58 % | 38.72 % |
| LaneATT | ResNet-18 | CuLane | 70.32 % | 18.09 % | 39.10 % |
| LaneATT | ResNet-18 | **CeyLane** | **80.89 %** | **14.02 %** | **24.36 %** |

*Evaluated on the CeyLane test set.*

**Lane-marking classification** (LaneATT, class-wise F1 on the TuSimple dataset)

| Lane marking class | F1 score |
|---|---:|
| Continuous yellow | 97.33 % |
| Continuous white | 96.67 % |
| Dashed | 98.60 % |
| Double dashed | 84.96 % |
| Botts' dots | 98.45 % |
| Double continuous yellow | 98.06 % |
| **Overall (simple average)** | **95.67 %** |

Two rare classes (double dotted-solid and double solid-dotted) are left out of the results because the dataset has too few examples of them.

---

#### Running in real time on the Jetson

Both models were optimised with **TensorRT**. On the Jetson AGX Xavier this gives roughly **1.5×** speed-up for lane detection and **2.8×** for traffic light detection, with no loss in accuracy.

![TensorRT speed-up on the Jetson AGX Xavier](/assets/images/bsc_fyp/images/tensorrt_speedup.png)

| Model | Jetson AGX Xavier (no TensorRT → TensorRT) | RTX 2080 Ti (no TensorRT → TensorRT) | Accuracy (no TensorRT → TensorRT) |
|---|---|---|---|
| Lane detector (UltraFast) | 31.12 Hz → **48.12 Hz** | 144.42 Hz → 218.81 Hz | 76.65 % → 76.64 % |
| Traffic light detector | 18.70 Hz → **52.65 Hz** | 109.90 Hz → 117.6 Hz | Overall F1 66.34 % → 66.38 % |

##### End-to-end integration

The traffic light, lane and road-marking models were integrated into one pipeline using **ROS**, all running on the Jetson AGX Xavier. Running them together costs some speed per model but stays in real-time territory:

| Detector | Before integration | After integration |
|---|---:|---:|
| Traffic light detector | 56.52 Hz | 39.51 Hz |
| Lane detector | 48.12 Hz | 39.48 Hz |
| Road marking detector | 37.51 Hz | 25.21 Hz |

---

#### The driver experience

The Jetson connects to a mobile app over USB. The app shows two warnings:

- **Traffic light ahead**
- **Lane departure**

Traffic light images and their predicted labels are also uploaded to a database in real time so they can be collected later and annotated through a web tool, which makes it easy to grow the dataset.

<p align="center">
  <img src="/assets/images/bsc_fyp/images/mobile_app.png" alt="Mobile app showing traffic light ahead and lane departure warnings" width="260">
</p>

---

#### Conclusions

- Our dual-camera technique detects traffic lights from a long distance in real time and significantly reduces false negatives.
- Our lane detection pipeline detects and tracks lanes with good accuracy in the complex conditions found on Sri Lankan roads.
- Together, the traffic light and lane pipelines give the driver useful lane departure and traffic light warnings, similar to an Advanced Driver Assistance System (ADAS).
- Training on our **Sri Lankan datasets** improved detection accuracy over models trained on foreign data.

---

#### Team and acknowledgements

- **Group members:** [Harindu Jayarathne](https://www.linkedin.com/in/harindu-jayarathne/), [Tharindu Samarakoon](https://www.linkedin.com/in/tharindusamare/), [Hasara Koralege](https://www.linkedin.com/in/hasara-koralege/), [Asitha Divisekara](https://www.linkedin.com/in/divisekara/)
- **Supervisor:** [Dr. Peshala Jayasekara](https://www.linkedin.com/in/peshala-jayasekara-756b7240/)
- **Co-supervisor:** [Dr. Ranga Rodrigo](https://www.linkedin.com/in/rangarodrigo/)
- **External collaborator:** Creative Software (Pvt) Ltd.