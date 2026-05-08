# Enhancing Situational Awareness of Helicopter Pilots in UAVs

This repository is the top-level thesis workspace for the codebases associated with the paper:

**John Mugabe, Mariusz Wisniewski, Adolfo Perrusquia, and Weisi Guo.**  
*Enhancing Situational Awareness of Helicopter Pilots in Unmanned Aerial Vehicle-Congested Environments Using an Airborne Visual Artificial Intelligence Approach*.  
Sensors, 2024, 24(23), 7762.  
DOI: `10.3390/s24237762`

Paper link: <https://www.mdpi.com/1424-8220/24/23/7762>

## Purpose

The thesis project combines stereo depth estimation, collision-risk prediction, and integrated simulation/deployment code into one research pipeline aimed at improving airborne visual situational awareness in UAV-congested environments.

This repository groups the project into a single entry point while preserving the original component repositories as independent codebases.

## Repository Structure

The component repositories are included here as git submodules:

- `repos/STEREONET-MODEL-TRAINING`: training code for the StereoNet-based stereo depth estimation model.
- `repos/COLLISION-PREDICTION-ALGORITHMS`: data preparation and recurrent-model training code for UAV collision prediction.
- `repos/GENERAL-IMPLEMENTATION-CODE`: integrated implementation code that combines simulation assets, stereo vision, object detection, and prediction logic.

## How The Parts Fit Together

1. `STEREONET-MODEL-TRAINING` trains the stereo depth model used to infer scene geometry from airborne stereo imagery.
2. `COLLISION-PREDICTION-ALGORITHMS` prepares UAV trajectory data and trains temporal models such as LSTM, GRU, and bidirectional LSTM variants for collision-risk prediction.
3. `GENERAL-IMPLEMENTATION-CODE` brings the main system together in simulation, combining stereo perception, learned model artifacts, and environment assets.

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
