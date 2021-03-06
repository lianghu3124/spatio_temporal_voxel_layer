# Spatio-Temporal Voxel Layer

This is a drop in replacement for the voxel_grid voxel representation of the environment. This package does a number of things to improve on the voxel grid package and extend the capabilities offered to the users, under a LGPL v2.1 license. Developed and maintained by [Steven Macenski](https://www.linkedin.com/in/steven-macenski-41a985101/) at [Simbe Robotics](http://www.simberobotics.com/). Much of the documentation below describes features to be added later.

This package sits on top of [OpenVDB](http://www.openvdb.org/), an open-source C++ library built by Dreamworks "comprising a novel hierarchical data structure and a suite of tools for the efficient storage and manipulation of sparse volumetric data discretized on three-dimensional grids. It is developed and maintained by DreamWorks Animation for use in volumetric applications typically encountered in feature film production."

Leveraging OpenVDB, we have the ability to efficiently maintain a 3 dimensional voxel-representative world space. We wrap this with ROS tools and interfaces to the [navigation stack](http://wiki.ros.org/navigation) to allow for use of this layer in standard ROS configurations. It is certainly possible to utilize this package without ROS/Navigation and I invite other competing methodologies to develop here and create interfaces. 

## **Spatio**-
The Spatio in this package is the representation of the environment in a configurable `voxel_size` voxel grid stored and searched by OpenVDB. 

## -**Temporal**
The Temporal in this package is the novel concept of `voxel_decay` whereas we have functions that associate to voxels and their information expires over time. In development, exist infrasture to store times in each voxel after which the voxel will disappear from the map. The goal is also to query a static map and determine which connected components belong to the map, not in the map, or moving. Each of these three classes of blobs will have configurable models to control the time they persist, and if these things are reported to the user.    

## Raytracing
This package utilizes OpenVDB's raytracing and intersector technologies to raytrace out voxels to be cleared in the levelset. This is true raytracing, unlike the 2D->3D attempt the traditional voxel grid uses. 

In development is an additional method of tracing where frustums of cameras are projected into the levelset and cleared. This may be combined in the future with the temporal features above to accelerate the decay of those voxels that do not remain marked rather than simply clearing them naively.

## Installation
Required dependencies ROS Kinetic, navigation, OpenVDB, TBB.

### ROS

[See install instructions here.](http://wiki.ros.org/kinetic/Installation)

### Navigation

`sudo apt-get install ros-kinetic-navigation`

### OpenVDB

`sudo apt-get install libopenvdb3.1 libopenvdb-doc libopenvdb-dev libopenvdb-tools`

### TBB

`sudo apt-get install libtbb-dev libtbb2 libtbb-doc`

## Configuration and Running

### costmap_common_params.yaml

An example fully-described configuration is shown below.

```
rgbd_obstacle_layer:
  enabled:               true
  voxel_decay:           20 #seconds
  voxel_decay_static:    1.
  voxel_size:            0.05
  track_unknown_space:   true
  observation_persistence: 0.0
  max_obstacle_height:   2.0
  unknown_threshold:     15
  mark_threshold:        0
  combination_method:    1
  obstacle_range:        3.0
  raytrace_range:        15.0
  origin_z:              0.0
  publish_voxel_map:     true
  observation_sources: rgbd_clear rgbd_clear
  rgbd_mark:
    data_type: PointCloud2
    topic: camera1/depth/points
    marking: true
    clearing: false
    min_obstacle_height: 0.3
    max_obstacle_height: 2.0
  rgbd_clear:
    data_type: PointCloud2
    topic: camera1/depth/points
    marking: false
    clearing: true
    min_obstacle_height: -15.
    max_obstacle_height: 15.
```

### local/global_costmap_params.yaml

Add this plugin to your costmap params file. 

`- {name: rgbd_obstacle_layer,     type: "spatio_temporal_voxel_layer/SpatioTemporalVoxelLayer"}`


### Running

`roslaunch [navigation_pkg] move_base.launch`

**This is a highly experimental package under heavy development. Functionality is not guaranteed until the first full release.**