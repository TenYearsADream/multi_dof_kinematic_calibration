# Note

We are in the process of publishing our software for multi-DOF kinematic calibration. While the software is basically usable, it may still have some rough edges in the documentation. Please give us a couple of days to sort those out.

# Overview
This software allows the accurate calibration of geometric/kinematic transformation hierarchies from sensor data. It is possible to calibrate/estimate

* unknown robot locations,
* unknown relative poses within the transformation hierarchy (for example the relative pose between two cameras),
* and unknown 1-DOF hinge joints, including the rotational axes as well as a mapping from raw encoder values to angles.

Prior to optimization, hierarchies have to be modeled in a simple JSON file format, which is passed to the optimizer. The latter will then try to optimize the problem my minimizing reprojection errors to known configurations of markers, or point-to-plane metrical errors of laser points to (planar) marker surfaces.

For both, the calibration of cameras and the calibration of laser range finders, known 3D reference geometry is required. We establish this reference making use of our [visual_marker_mapping](https://github.com/cfneuhaus/visual_marker_mapping) toolkit. It allows the accurate estimation of the 3D poses of configurations of optical markers given a set of camera images (from a DSLR camera for example). It uses [AprilTags](https://april.eecs.umich.edu/papers/details.php?name=olson2011tags) by Olson, that can simply be printed out and attached to walls or objects. To perform the reconstruction, you need a calibrated camera with a fixed focal length (no auto focus). The camera needs to be calibrated using the typical OpenCV camera calibration model. 

# Installation

## Dependencies

* [Ceres Solver](http://ceres-solver.org/)
* [OpenCV](http://opencv.org/)
* [Eigen 3.0](http://eigen.tuxfamily.org/)
* [Boost](http://www.boost.org/)
* [Visual Marker Mapping Tool](https://github.com/cfneuhaus/visual_marker_mapping)

In Ubuntu, the first three dependencies can be installed using the command

* `apt-get install libceres-dev libsuitesparse-dev libopencv-dev libeigen3-dev libboost-dev libboost-filesystem-dev libboost-program-options-dev libboost-system-dev python-pip`

In Arch Linux, use:

* `pacman -S eigen opencv boost python-pip`

* Ceres is available as an AUR package called [ceres-solver](https://aur.archlinux.org/packages/ceres-solver/).

The [visual_marker_mapping](https://github.com/cfneuhaus/visual_marker_mapping) dependency is automatically pulled in as a git submodule.

## Building

You need at least
* [CMake 3.1](https://cmake.org/)
* GCC 4.7 (?)
* Flep (see https://github.com/cfneuhaus/flep)

If you are using Ubuntu, this means that you need at least Ubuntu 14.04.

### Linux/Mac

1. Install flep:
```
git clone https://github.com/cfneuhaus/flep
pip install --user ./flep
```

2. Install multi_dof_kinematic calibration and deps:

```
mkdir flep_ws && cd flep_ws && flep init
cd src
git clone https://github.com/cfneuhaus/multi_dof_kinematic_calibration.git
flep get_deps
flep cmake -DCMAKE_BUILD_TYPE=Release
flep make -j5
```

In order to get easy access to the built binaries, we provide a ROS-style setup.sh file, that adds the flep_ws/build/bin directory to your PATH environment variable. Use
```
source flep_ws/build/setup.sh
```
to run it. If you prefer not to do this step, you obviously need to provide the full path to the binaries, when running the applications.

### Windows

It should be possible to build our software on Windows, given that we do not use any platform specific features, but so far we have not attempted to build it there. Please let us know if you run into problems doing this.

# Usage

## Preliminaries

Our software works on *project paths*. A project path typically has a layout similar to to this:
```
my_project/.
my_project/calibration_data.json
my_project/my_reconstruction/reconstruction.json
my_project/my_camera/camera_calibration.json
my_project/my_camera/marker_detections.json
my_project/my_camera/images/your_image_1.jpg
my_project/my_camera/images/...
my_project/my_laser/scan1.dat
my_project/my_laser/...
...
```

* The file *reconstruction.json* contains the marker-based 3D reference geometry. It needs to be created using the [visual_marker_mapping](https://github.com/cfneuhaus/visual_marker_mapping) tool. Please refer to the [README file](https://github.com/cfneuhaus/visual_marker_mapping/blob/master/README.md) in that project for information about this process.
* The file *calibration_data.json* is the main file that describes the calibration problem to be solved. We provide a number of  [examples](#example) that show how to create this one.
* The folders *my_camera* and *my_laser* depend on the concrete sensor setup that is being optimized. The given folder- and file names only serve as an example.
* After completion, our tool write out two files: *calibration_result.json* and *calibration_result_visualization.json*. The first one contains the actual calibration results. The second one contains special information that is read by our visualization tool to visualize the results.

## Running

Our software consists of a command-line tool called *multi_dof_kinematic_calibration* located in the *build/bin* folder, and a python script that can optionally be used to visualize the results in 3D.

multi_dof_kinematic_calibration:
* `--help`: Shows a help text.
* `--project-path`: Path to the aforementioned project directory.

For visualization of the results in 3D, we also include a Python (2.7/3.0) script called "visualize_results.py". It is based on *pygame*, *OpenGL*, *GLU*, *GLUT*, *numpy*, and you may need to install the corresponding Python packages for your distribution in order to be able to run it.

The tools only parameter is the project path. The camera can be controlled using W, S, A, D. The mouse can be used to look around by holding the left mouse button. The camera speed can be increased by holding space.

## Example

We provide a test dataset, that you can use to test our tools. It is available [here](https://agas.uni-koblenz.de/data/datasets/multi_dof_kinematic_calibration/ptu_d47_w_xtion_ir_WRT_calibration_room_1.zip).

Use the following steps to perform the marker detection and 3D reconstruction (assuming you are in the root folder of this repository):

```
wget https://agas.uni-koblenz.de/data/datasets/multi_dof_kinematic_calibration/ptu_d47_w_xtion_ir_WRT_calibration_room_1.zip
unzip ptu_d47_w_xtion_ir_WRT_calibration_room_1.zip
cd ptu_d47_w_xtion_ir_WRT_calibration_room_1
visual_marker_detection --project_path ir_cam/ --marker_width 0.1285 --marker_height 0.1295
multi_dof_kinematic_calibration --project_path .
```

As an alternative, the dataset includes a Makefile that performs the necessary steps. Use
```
cd ptu_d47_w_xtion_ir_WRT_calibration_room_1
make
```
to run it. For this to work, you need to have to have sourced the *build/setup.sh* file, as explained in the [Building](#building) section.

### Output:

```
Solving full optimization problem...
    Full optimization returned termination type 0
    Full training reprojection error RMS: 0.840787 px
Solving full optimization problem...done!

Resulting Parameters:
    Tick2Rad for all joints:
        pan_joint:0.000895116, tilt_joint:0.000919349, 
    Joint poses:
        pan_joint:    1.10492          0          0    0.78117  0.0077221 -0.0024885  -0.624265
        tilt_joint:   1.29675         0         0  0.550994 -0.439112 -0.557578  0.438968
        ir_cam_joint:  0.00844504   0.0594558 -0.00874979    0.515369    0.489033    0.505573    0.489529
Test Reprojection Error RMS: 2.51105 px
Test Reprojection Error RMS for camera 1: 2.51105 px
```

The tool prints the estimated transformations for all of the joints/transformations. All poses transform from a joint's parent to joint space. Poses a given in the order *x*, *y*, *z*, *qw*, *qx*, *qy*, *qz*.

If you want to visualize the results, simply run:
```
python visualize_results.py .
```

###  Analysis

The sample contains a calibration_data.json file, which looks as follows:
```
{
  "world_reference" : "calibration_room_1/reconstruction.json",
  "hierarchy" : [
    {
      "name" : "pan_joint",
      "type" : "1_dof_joint",
      "ticks_to_rad" : 0.00089760538,
      "angular_noise_std_dev" : 0.00089760538,
      "parent_to_joint_pose_guess" : [0, 0, 0, 1, 0, 0, 0],
      "parent" : "base"
    },
    {
      "name" : "tilt_joint",
      "type" : "1_dof_joint",
      "ticks_to_rad" : 0.00091879199999999998,
      "angular_noise_std_dev" : 0.00091879199999999998,
      "parent_to_joint_pose_guess" : [0, 0, 0, 1, 0, 0, 0],
      "parent" : "pan_joint"
    },
    {
      "name" : "ir_cam_joint",
      "type" : "pose",
      "parent_to_joint_pose_guess" : [0, 0, 0, 1, 0, 0, 0],
      "parent" : "tilt_joint"
    }
  ],
  "sensors" : [
    {
      "sensor_id" : "1",
      "sensor_type" : "camera",
      "parent_joint" : "ir_cam_joint",
      "camera_path" : "ir_cam"
    }
  ],
  "calibration_frames" : [
    {
      "location_id" : -1,
      "camera_image_path_1" : "images/img_0.png",
      "pan_joint_ticks" : "-1783",
      "tilt_joint_ticks" : "613"
    },
    {
      "location_id" : -1,
      "camera_image_path_1" : "images/img_1.png",
      "pan_joint_ticks" : "-1640",
      "tilt_joint_ticks" : "613"
    },
    ..
   ]
}
```

TODO: Add more detailed description here.

All poses transform from a joint's parent to joint space. Poses a given in the order *x*, *y*, *z*, *qw*, *qx*, *qy*, *qz*. The local coordinate systems are defined as follows:
* A cameras *x*-axis points to the right, *y*-axis down, and the *z*-axis in viewing direction.


###  More examples

We provide a number additional examples that can be used to understand the capabilities of the system.

#### Synthetic Dataset with Ground Truth

In order to evaluate our approach, we created a synthetic dataset with simulated robotic arms with 3 and 4 DOF. We simulate varying amounts of noises (0 deg, 0.025 deg, 0.05 deg, 0.1 deg and 1 deg) on the actuator positioning, and render a 3D marker configuration from the simulated camera's pose. 

We provide two variants of our dataset:
* [This one](https://agas.uni-koblenz.de/data/datasets/multi_dof_kinematic_calibration/synthetic_full.zip) contains all of the mentioned data, including the actual renderings.
* In [this one](https://agas.uni-koblenz.de/data/datasets/multi_dof_kinematic_calibration/synthetic_detections_only.zip), markers have already been extracted from the camera images using our *visual_marker_detection* tool. The images itself are not included.

To run our software on any of the datasets, you can proceed in exactly the same way as it is done in the previous example. Each dataset in the archive contains a Makefile that performs the processing.

# Authors

* Frank Neuhaus (fneuhaus_AT_uni-koblenz.de)
* Stephan Manthe

# Copyright and License

[GPL 3.0](http://www.gnu.org/licenses/gpl-3.0.en.html)

# Citing

If you use our work, please cite us:

Practical Calibration of Actuated Multi-DOF Camera Systems. International Conference on Pattern Recognition Systems (ICPRS-17), 2017, IET. Juli 2017

# References

Ed Olson, [AprilTag: A robust and flexible visual fiducial system](http://april.eecs.umich.edu/papers/details.php?name=olson2011tags), Proceedings of the IEEE International Conference on Robotics and Automation (ICRA), 2011
