# MSCKF\_VIO


The `MSCKF_VIO` package is a stereo version of MSCKF. The software takes in synchronized stereo images and IMU messages and generates real-time 6DOF pose estimation of the IMU frame.

The software is tested on Ubuntu 16.04 with ROS Kinetic.

Video: [https://www.youtube.com/watch?v=OXSB8Bze0cY](https://www.youtube.com/watch?v=OXSB8Bze0cY)  
Paper Draft: [https://arxiv.org/abs/1712.00036](https://arxiv.org/abs/1712.00036)

## License

Penn Software License. See LICENSE.txt for further details.

## Dependencies

Most of the dependencies are standard including `Eigen`, `OpenCV`, and `Boost`. The standard shipment from Ubuntu 16.04 and ROS Kinetic works fine. One special requirement is `suitesparse`, which can be installed through,

```
sudo apt-get install libsuitesparse-dev
```

## Compling
The software is a standard catkin package. Make sure the package is on `ROS_PACKAGE_PATH` after cloning the package to your workspace. And the normal procedure for compiling a catkin package should work.

```
cd your_work_space
catkin_make --pkg msckf_vio --cmake-args -DCMAKE_BUILD_TYPE=Release
```

## Calibration

An accurate calibration is crucial for successfully running the software. To get the best performance of the software, the stereo cameras and IMU should be hardware synchronized. [Kalibr](https://github.com/ethz-asl/kalibr) can be used to calibrate the transformantion between the stereo cameras and IMU. The yaml file generated by Kalibr can be directly used in this software with slight modifications. See calibration files in the `config` folder for details. The two calibration files in the `config` folder should work directly with the EuRoC and [fast flight](https://github.com/KumarRobotics/msckf_vio/wiki) datasets.

The filter requires 200 IMU messages to initialize gyro bias, acc bias, and initial orientation. Therefore, the robot is required to start from static in order to initialize the VIO successfully.

## Example Usage

There are launch files prepared for the EuRoC and fast flight dataset separately. Upon launching the `msckf_vio_*.launch`, two ros nodes are created:
* `image_processor` takes the stereo images to detect and track features.
* `vio` takes the feature measurements and tightly fuses them with the IMU messages to estimate pose.

### `image_processor` node

**Subscribed Topics**

`imu` (`sensor_msgs/Imu`)

IMU messages is used for compensating rotation in feature tracking, and 2-point RANSAC.

`cam[x]_image` (`sensor_msgs/Image`)

Synchronized stereo images.

**Published Topics**

`features` (`msckf_vio/CameraMeasurement`)

Records the feature measurements on the current stereo image pair.

`tracking_info` (`msckf_vio/TrackingInfo`)

Records the feature tracking status for debugging purpose.

`debug_stereo_img` (`sensor_msgs::Image`)

Draw current features on the stereo images for debugging purpose. Note that this debugging image is only generated upon subscription.

### `vio` node

**Subscribed Topics**

`imu` (`sensor_msgs/Imu`)

IMU measurements.

`features` (`msckf_vio/CameraMeasurement`)

Stereo feature measurements from the `image_processor` node.

**Published Topics**

`odom` (`nav_msgs/Odometry`)

Odometry of the IMU frame including a proper covariance.

`feature_point_cloud` (`sensor_msgs/PointCloud2`)

Shows current features in the map which is used for estimation.
