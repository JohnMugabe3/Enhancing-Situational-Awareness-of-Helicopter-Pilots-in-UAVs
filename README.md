# Enhancing Situational Awareness of Helicopter Pilots in UAVs

This repository is the top-level thesis workspace for the codebases associated with the paper:

**John Mugabe, Mariusz Wisniewski, Adolfo Perrusquia, and Weisi Guo.**  
*Enhancing Situational Awareness of Helicopter Pilots in Unmanned Aerial Vehicle-Congested Environments Using an Airborne Visual Artificial Intelligence Approach*.  
Sensors, 2024, 24(23), 7762.  
DOI: `10.3390/s24237762`

Paper link: <https://www.mdpi.com/1424-8220/24/23/7762>

## Purpose

The project develops an airborne visual AI pipeline to improve helicopter pilot situational awareness in UAV-congested environments. The end-to-end workflow combines:

- object detection to find nearby UAVs,
- stereo depth estimation to estimate their distance,
- path prediction to estimate likely future motion,
- a threshold-based alert system to warn the pilot of likely collisions.

This umbrella repository groups the thesis work into one entry point while preserving the original component repositories as independent codebases.

## Repository Structure

The component repositories are included here as git submodules:

- `repos/STEREONET-MODEL-TRAINING`: StereoNet data generation, training, and evaluation code for stereo depth estimation.
- `repos/COLLISION-PREDICTION-ALGORITHMS`: trajectory preprocessing and recurrent-model training code for collision prediction.
- `repos/GENERAL-IMPLEMENTATION-CODE`: integrated PyBullet simulation, stereo perception, object detection, distance estimation, path prediction, and alert generation.

## End-to-End Project Workflow

The project is organized as a four-stage situational-awareness pipeline.

### 1. Object Detection

The integrated implementation captures left and right RGB images from a stereo camera mounted on the simulated helicopter and runs a YOLO-based UAV detector on the images.

Main steps:

1. Stereo images are captured from left and right virtual cameras in PyBullet.
2. Each image is converted to a detection blob using OpenCV DNN preprocessing.
3. A YOLO detector runs forward inference to identify UAV candidates.
4. Bounding boxes are filtered using confidence thresholding and Non-Maximum Suppression.
5. Final boxes are drawn on the left and right camera views for operator visualization.

Algorithms used:

- YOLO object detector loaded from custom `yolo-drone.cfg` and `yolo-drone.weights`
- OpenCV DNN inference pipeline
- confidence thresholding at `0.5`
- Non-Maximum Suppression using `cv2.dnn.NMSBoxes` with NMS threshold `0.4`

In the paper, the detection stage is described as a YOLOv3-based UAV detector for real-time airborne perception.

### 2. Depth Estimation

After detection, the system estimates scene depth from the left and right stereo images using a learned StereoNet model.

Main steps:

1. A synthetic stereo dataset is generated in PyBullet using multiple camera settings and motion patterns.
2. Left, right, and ground-truth depth images are saved from simulation.
3. A StereoNet model is trained to map stereo image pairs to dense depth maps.
4. During inference, the trained model predicts a dense depth map from the current stereo pair.
5. The depth values inside each detected bounding box are averaged to estimate UAV distance.

Algorithms used:

- StereoNet-style convolutional stereo depth network
- convolution + batch normalization + ReLU feature extractor
- feature concatenation between left and right image streams
- refinement network with bilinear upsampling to `640x480`
- synthetic stereo data generation in PyBullet
- depth-buffer conversion from simulated camera geometry
- average valid depth pooling inside each detection bounding box

Training and evaluation methods used:

- Mean Squared Error loss
- Adam optimizer
- `ReduceLROnPlateau` learning-rate scheduler
- train/validation split using `train_test_split`
- SSIM for structural depth-quality evaluation
- PSNR for reconstruction-quality evaluation
- MAE and RMSE for runtime error reporting in the integrated implementation

The StereoNet training code also varies camera field-of-view and near/far clipping planes, and generates different motion patterns including:

- circular motion
- zigzag motion
- random motion

### 3. Path Prediction

The next stage predicts likely future UAV trajectories so the system can assess collision risk before a drone becomes critical.

Main steps:

1. Raw UAV trajectory logs are read from the UAV delivery dataset.
2. Relevant motion features are extracted: latitude, longitude, and altitude.
3. The features are normalized using Min-Max normalization.
4. Sliding temporal windows of length `10` are created.
5. Recurrent neural networks are trained to predict the next future position.
6. In the integrated runtime system, the LSTM model uses the last 10 positions of each detected UAV to predict future positions relative to the helicopter.

Algorithms used:

- MinMaxScaler normalization
- sequence-window creation with a 10-step input history
- Long Short-Term Memory (LSTM)
- Gated Recurrent Unit (GRU)
- Bidirectional LSTM (BiLSTM)

Training settings used in the collision-prediction repository:

- input features: `lat`, `lon`, `alt`
- hidden size: `25`
- one recurrent layer
- output size: `3`
- batch size: `64`
- `50` epochs
- Adam optimizer
- Mean Squared Error loss
- MAE and RMSE for predictive evaluation

Important implementation detail:

- The research repository trains and compares LSTM, GRU, and BiLSTM predictors.
- The integrated alert pipeline currently deploys the LSTM model for runtime collision prediction.

### 4. Alert System

The final stage converts estimated distances and predicted motion into pilot-facing warnings.

Main steps:

1. The system computes predicted future UAV positions using the trained LSTM.
2. Predicted distance to the helicopter is computed from the estimated future coordinates.
3. Relative speed is estimated from the difference between predicted and latest positions.
4. Time-to-collision is estimated using distance divided by velocity.
5. A warning is triggered if the UAV is close enough and the time-to-collision is finite.
6. The system classifies the warning direction as front-left or front-right based on the predicted lateral position of the UAV.
7. The GUI displays detections, depth map, tracked distances, and textual collision alerts.

Algorithms and rules used:

- Euclidean distance for predicted UAV-to-helicopter separation
- linear time-to-collision estimate `time = distance / velocity`
- threshold-based alert generation
- direction-of-approach classification from relative lateral position

Current runtime alert conditions from the integrated implementation:

- alert threshold: `3000 cm`
- example alert messages:
  - `Drone Detected - Front Left! Collision in T seconds`
  - `Drone Detected - Front Right! Collision in T seconds`

## Algorithm Summary

Across the full thesis project, the following algorithms and methods are used:

- YOLO-based UAV object detection
- Non-Maximum Suppression
- StereoNet for stereo depth estimation
- synthetic stereo dataset generation in PyBullet
- recurrent neural networks for trajectory prediction:
  - LSTM
  - GRU
  - BiLSTM
- Min-Max feature normalization
- Mean Squared Error loss
- Adam optimization
- `ReduceLROnPlateau` scheduling
- SSIM
- PSNR
- MAE
- RMSE
- Euclidean distance estimation
- threshold-based collision alerting
- time-to-collision estimation

## How The Repositories Map to the Workflow

1. `STEREONET-MODEL-TRAINING`
   - generates synthetic stereo data in PyBullet
   - trains StereoNet
   - evaluates depth quality using SSIM and PSNR

2. `COLLISION-PREDICTION-ALGORITHMS`
   - preprocesses UAV trajectory logs
   - normalizes motion data
   - trains LSTM, GRU, and BiLSTM predictors
   - evaluates path-prediction error using MAE and RMSE

3. `GENERAL-IMPLEMENTATION-CODE`
   - loads the trained StereoNet and LSTM models
   - runs real-time UAV detection
   - estimates depth for detected drones
   - predicts future paths
   - generates front-left/front-right collision warnings in the GUI

## Clone With Submodules

Clone this umbrella repository together with all linked thesis repositories:

```bash
git clone --recurse-submodules git@github.com:JohnMugabe3/Enhancing-Situational-Awareness-of-Helicopter-Pilots-in-UAVs.git
```

If you already cloned it without submodules:

```bash
git submodule update --init --recursive
```

## Notes

- The submodule layout is intentional so each research component keeps its own history and can still be maintained independently.
- Some component repositories contain trained model artifacts and environment assets required by the original experiments.
- The paper states that the proposed framework combines object detection, depth estimation, collision prediction, and threshold-based pilot alerts into a single airborne visual AI system.
