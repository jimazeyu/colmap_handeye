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
3. Use `calibration/cam_to_base/calibration.ipynb` to calibrate both the wrist and side cameras. The output will include transformation matrices as calibration results.

**Example results:**

Wrist camera to end effector:
```python
Rotation matrix: 
[[-0.01106385 -0.99964285  0.02432601]
 [ 0.99967782 -0.01161345 -0.02256943]
 [ 0.02284388  0.02406847  0.99944928]]
Translation vector: 
[[ 0.07624674]
 [-0.03747215]
 [-0.09101364]]
```

Virtual colmap coordinates to the arm base(can be directly copied to GraspSplats):
```python
world2base = np.array([
[-0.4089165231525215, -0.8358961766325012, 0.3661486842582114, 0.42083348316217706],
[-0.9105881302403995, 0.34730407737749247, -0.22407394962882685, 0.20879287837427596],
[0.060137626808399375, -0.4250381861999404, -0.9031755123527864, 0.5594013590398528],
[0.0, 0.0, 0.0, 1.0],
])
```

Left camera to base:

```python
left2base = np.array([
[-0.9976893814818404, 0.016141059503606225, -0.06599518373702401, 0.4802522351597585],
[0.05878010270198073, 0.6921878285221563, -0.7193197547489758, 0.6467787114729435],
[0.03407047996032865, -0.7215368848810733, -0.6915372196428687, 0.5627649333032588],
[0.0, 0.0, 0.0, 1.0],
])
```

Right camera to base:

```python
right2base = np.array([
[0.9998830560352922, 0.004162496284347904, 0.01471556584733334, 0.4926358949473875],
[-0.010079531484636053, -0.5442897813649162, 0.8388367164989752, -0.6617851283408656],
[0.011501186833307645, -0.8388869456168774, -0.544184173947581, 0.46742609681882763],
[0.0, 0.0, 0.0, 1.0],
])
```

## Acknowledgements

We have adapted the COLMAP rescaling code from the [aruco-estimator repository](https://github.com/meyerls/aruco-estimator).