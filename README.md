# 🖥️ Raspberry Pi Handwritten Digit Recognition System
[![Python](https://img.shields.io/badge/Python-3.9+-3776AB?logo=python)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.x-5C3EE8)](https://opencv.org/)
[![TensorFlow Lite](https://img.shields.io/badge/TensorFlow-Lite-FF6F00?logo=tensorflow)](https://www.tensorflow.org/lite)
[![RPi.GPIO](https://img.shields.io/badge/Raspberry-Pi-CF0000?logo=raspberrypi)](https://www.raspberrypi.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

> Repository Name: mnist-handwritten-digit-recognition
> Short Description: Embedded handwritten digit recognition based on Raspberry Pi, KNN & CNN, fused MNIST + self-collected dataset with camera & nixie tube hardware display

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [Hardware Composition](#-hardware-composition)
- [File Structure](#-project-file-structure)
- [Core Module Introduction](#-core-module-introduction)
- [Algorithm Comparison](#-algorithm-comparison-knn-vs-cnn)
- [Deployment & Running Guide](#-quick-start-deployment-guide)
- [Team Division](#-team-division-of-labor)
- [Project Documentation](#-project-documents)
- [Problems & Solutions](#-common-bugs-and-solutions)
- [Optimization Suggestions](#-future-optimization-suggestions)
- [Demo Resource](#-demo-video-link)
- [Acknowledgements](#-acknowledgements)

---

## 🎯 Project Overview
This is a software-hardware integrated embedded computer vision project developed by Group 8.
Based on Raspberry Pi as the main control chip, the system realizes end-to-end handwritten digit recognition:
1. Physical button trigger CSI camera to capture handwritten images
2. Image preprocessing: grayscale, binarization, denoising, morphological stroke repair
3. Horizontal & vertical projection segmentation to extract single digits
4. Two recognition algorithms: traditional KNN & lightweight CNN
5. Nixie tube hardware display for recognition results

### Core Features
✅ Dual algorithm comparison: KNN baseline + CNN optimized model
✅ Dataset fusion: Standard MNIST + self-shot real handwritten samples
✅ Complete embedded hardware interaction: Camera, button, common-anode nixie tube, LED
✅ Modular code design, reusable image processing & GPIO tool functions
✅ Complete exception handling & automatic GPIO resource release

---

## 🔌 Hardware Composition
### Hardware List
Raspberry Pi mainboard, CSI camera, common-anode nixie tube, LED indicator, push button, breadboard, DuPont wires, current-limiting resistors

### Key BOARD Pin Mapping
| Device | Physical Pin | Function |
|--------|-------------|----------|
| Camera Trigger Button | 38 | Camera capture switch |
| Nixie Tube Segment A | 18 | Digital segment control |
| Nixie Tube Segment B | 16 | Digital segment control |
| Nixie Tube Segment C | 13 | Digital segment control |
| Nixie Tube Segment D | 12 | Digital segment control |
| Nixie Tube Segment E | 11 | Digital segment control |
| Nixie Tube Segment F | 22 | Digital segment control |
| Nixie Tube Segment G | 29 | Digital segment control |
| Nixie Tube DP | 15 | Decimal point |

> Circuit rule: Common-anode device, GPIO LOW = light on, HIGH = light off

---

## 📂 Project File Structure
```
mnist-handwritten-digit-recognition/
├── README.md                 # Project introduction document
├── CreateTrainSet.py         # Build dataset & train KNN classifier
├── ProcessRealImage.py       # Camera capture & real image inference
├── main.py                   # Hardware main program, nixie tube display
├── my_function.py            # Encapsulated image & GPIO tool library
├── train_cnn_colab.ipynb     # CNN training script for Google Colab
├── dataset/                  # Training data storage
│   ├── mnist_standard.npz
│   └── user_collect.npz
├── model/                    # Saved offline models
│   ├── knn_model.npz
│   └── mnist_cnn.tflite
├── docs/                     # Project report folder
│   └── HNR Project Report（Group8）.docx
└── LICENSE
```

---

## 🧩 Core Module Introduction
<details>
<summary>1. CreateTrainSet.py (KNN Dataset Training)</summary>
- Load standard MNIST digit samples
- Unify image to 20×20, flatten to 400D feature vector
- Generate 0~9 matching labels
- Save dataset as npz to avoid repeated preprocessing
- Train OpenCV KNN as baseline recognition algorithm
</details>

<details>
<summary>2. my_function.py (Core Tool Library)</summary>
### Image Processing
- Grayscale conversion, binarization, small noise component removal
- Morphology dilation/opening/closing to repair broken strokes
- Foreground crop, proportional resize to MNIST 28×28 standard format
### Segmentation Algorithm
- Horizontal projection split digit rows; vertical projection split single digits
### GPIO Hardware Control
- Camera button initialization, software anti-shake, photo preview & capture
- Nixie tube initialization, single/multi-row cyclic display, resource cleanup
### Dataset Management
- Save & load self-collected handwritten data, merge with MNIST for CNN training
</details>

<details>
<summary>3. ProcessRealImage.py & main.py (Hardware Inference & Display)</summary>
1. Initialize camera GPIO, open 10s preview window
2. Block and wait for button press to capture pictures
3. Complete preprocessing & segmentation pipeline
4. Call KNN / TFLite CNN to get recognition result list `rowsNumberList`
5. Nixie tube cyclic display rules:
   - Same row digits: 1s interval with screen off between digits
   - Different rows: 2s blank interval before switching
6. try-except-finally to safely release GPIO pins after exit
</details>

---

## 📊 Algorithm Comparison: KNN VS CNN
| Item | KNN (Traditional Machine Learning) | CNN (Deep Learning) |
|------|------------------------------------|---------------------|
| Feature Extraction | Manual preprocessing rules | Automatic convolution feature extraction |
| Dataset | Single MNIST 20×20 vector | MNIST + self-collected 28×28 images |
| Performance | Poor for complex digits (8,9), sensitive to handwriting deformation | High accuracy, anti-interference for irregular handwritten numbers |
| Deployment Cost | Lightweight, no GPU required | Need TFLite lightweight quantization for Raspberry Pi |
| Limitation | Over-reliance on manual image processing logic | Longer training time (Colab offline training) |

### Optimization Idea
Fuse official MNIST data with self-shot real handwritten samples to reduce the distribution gap between training data and actual shooting scenes, greatly improving recognition accuracy of complex digits.

---

## 🚀 Quick Start Deployment Guide
### 1. Hardware Wiring
Connect nixie tube, camera, button to Raspberry Pi according to pin mapping table, check short circuit.

### 2. Environment Installation
```bash
# Enable camera in raspberry config
sudo raspi-config
sudo apt update && sudo apt install python3-pip python3-opencv libatlas-base-dev
pip3 install numpy RPi.GPIO tensorflow pillow
```

### 3. Generate KNN Training Dataset
```bash
python3 CreateTrainSet.py
```

### 4. Train CNN Model
Upload `train_cnn_colab.ipynb` to Google Colab, run all cells, download `mnist_cnn.tflite` and put into `/model` folder.

### 5. Run Full System
```bash
python3 main.py
```
1. Camera preview pops up for 10 seconds to adjust shooting angle
2. Press physical button to take photos of handwritten digits
3. Automatic segmentation and recognition
4. Nixie tube circulate display prediction results
5. Press `Ctrl + C` to exit, GPIO automatically release resources

---

## 📑 Project Documents
📄 [[完整项目报告 Word文档](./docs/HNR Project Report.docx)](https://github.com/Hua-Haoyue/mnist-handwritten-digit-recognition/blob/main/docs/HNR%20Project%20Report.docx)

---

## ⚠️ Common Bugs and Solutions
### Hardware Problems
1. CSI camera loose cable → Kernel crash, SSH disconnect
   Solution: Fix cable with fixing clip, reserve debugging buffer time
2. Nixie tube display disorder → Pin wiring mismatch or segment code error
   Solution: Two-person collaborative debugging, check wiring table and print GPIO level logs
3. Button no response → Mechanical bounce / missing pull-down resistor config
   Solution: Add software debounce delay, reinitialize GPIO pull-down mode

### Software Algorithm Problems
1. Vertical stacked digits segmentation error
   Solution: Adjust projection threshold, optimize vertical segmentation logic
2. Low KNN recognition accuracy for digit 8 & 9
   Solution: Switch to CNN model, supplement self-collected training samples
3. Offline test accuracy inconsistent with real shooting effect
   Root cause: Label mismatch during dataset expansion, training labels are correct and do not affect actual inference

---

## 💡 Future Optimization Suggestions
### Hardware Optimization
- Use lock-type GPIO headers and CSI cable fixing clips to reduce poor contact
- Prepare spare Raspberry Pi and peripheral modules
- Add shockproof shell and anti-static operation protection

### Software Optimization
- Write automatic SSH reconnection script for hardware disconnection
- Build QEMU Raspberry Pi simulation environment for offline coding
- Quantify CNN model to speed up Raspberry Pi inference

### Project Management
- Pre-project hardware operation training to reduce wiring errors
- Reserve 10%~15% total cycle as hardware failure buffer time
- Remove nixie tube model marking to train independent hardware discrimination ability

---

## 🎬 Demo Video Link
Experimental test video:
https://video.weibo.com/show?fid=1034:5250434453930056
The video records the whole hardware-software joint debugging process and team online & offline collaborative development.

---

## 🙏 Acknowledgements
Special thanks to course teachers and teaching assistants for technical guidance during the project.
Thanks to the school for providing embedded practice platform.
Thanks to all team members for mutual cooperation to complete the whole software-hardware integrated system.
