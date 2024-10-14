# colmap_handeye
This repository provides tools for simultaneous hand-in-eye and hand-to-eye calibration using COLMAP. It is naturally compatible with COLMAP-based NeRF/3D Gaussian splatting tasks like [GraspSplats](https://github.com/jimazeyu/GraspSplats).

## Installation

First, follow the [COLMAP installation guide](https://colmap.github.io/). Then, install the required dependencies with the following commands:

```bash
micromamba create -n colmap_calibration python=3.10 -c conda-forge
# For calibration
pip install opencv-python opencv-contrib-python==4.6.0.66 matplotlib open3d colmap-wrapper numpy==1.24.4 pycolmap==0.4.0 ipykernel 
# For data collection (ensure panda-python version matches your arm)
pip install pyrealsense2 panda-python
```

## Data Collection

Our setup includes the following hardware, but you can adapt it to use your own robot arm and camera. We use an RGB-D camera, though an RGB camera suffices for calibration tasks:
- Realsense D435 camera (1 wrist camera, 2 side cameras)
- Franka Research 3 robot arm
- Aruco code board, generated using the [Aruco Generator](https://chev.me/arucogen/)

If you are using a similar hardware setup (Realsense + Franka), you can use the provided notebooks: `data_collection/wrist_cam_shoot.ipynb` and `data_collection/side_cam_shoot.ipynb` for image capturing. Predefined shooting poses are available in `data_collection/shooting_pose_tabletop.txt`.

## Custom Data Preparation

An example dataset is provided to help you prepare your own data. Follow this format:
- Files in `poses`: 4x4 numpy arrays representing the pose of the end effector relative to the robot base.
- Files in `images`: RGB images (1280x720 resolution in our dataset).
- Files in `depth`: Aligned depth images (1280x720 resolution in our dataset).

**Note:** Ensure the scene is sufficiently complex to provide enough feature points for COLMAP, as this will improve calibration accuracy.

## Calibration

Follow these steps sequentially:

1. Use `calibration/camera_align/sfm.ipynb` to compute the camera poses in world coordinates (virtual coordinates) and rescale them using the Aruco board.
2. Use `calibration/cam_to_base/tsdf_initialization.ipynb` to check alignment and generate a 3D point cloud using TSDF (this can serve as initialization for [GraspSplats](https://github.com/jimazeyu/GraspSplats)).
3. Use `calibration/cam_to_base/calibration.ipynb` to calibrate both the wrist and side cameras. The output will include transformation matrices, such as `world2base` (original coordinate system) and side cameras to base.

Example results:

```python
world2base = np.array([
    [-0.45036495018819156, -0.8136286436826232, 0.36766811368564656, 0.4199064849740805],
    [-0.8862783766217048, 0.3575391129093141, -0.2944085968052156, 0.20789427908058333],
    [0.10808353609492491, -0.45844761196803585, -0.8821245582716889, 0.5606234792839725],
    [0.0, 0.0, 0.0, 1.0],
])
```

Left camera to base:

```python
np.array([
    [-0.99767353,  0.01652426, -0.06613986,  0.48034451],
    [0.05913856,  0.6924143,  -0.71907237,  0.64605187],
    [0.03391405, -0.72131089, -0.69178063,  0.56220322],
    [0., 0., 0., 1.]
])
```

Right camera to base:

```python
np.array([
    [0.99990103,  0.00362503,  0.01359375,  0.49349123],
    [-0.0094344, -0.54400889,  0.83902641, -0.66249925],
    [0.01043662, -0.83907162, -0.54392085,  0.46702689],
    [0., 0., 0., 1.]
])
```

## Acknowledgements

We have adapted the COLMAP rescaling code from the [aruco-estimator repository](https://github.com/meyerls/aruco-estimator).