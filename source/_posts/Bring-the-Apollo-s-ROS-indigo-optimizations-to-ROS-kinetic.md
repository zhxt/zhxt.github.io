---
title: Bring the Apollo's ROS indigo optimizations to ROS kinetic
date: 2018-10-14 11:53:12
tags:
- Apollo
- ROS
- kinetic
---

Since we konw the Apollo Team made three major optimizations to the original ROS indigo, and made it open sourced, so we can 'easily' port it to ROS kinetic.

The three optimizations are:

- Add protobuf support
- Add shared memory transport
- Decentralization

We start with adding protobuf support.

### Pre-require: a minimal kinetic build.

To get a minimal kinetic build, first, create a separate workspace, generate a desktop_full version ros source list file with git format instead of tar, and select the packages we want, reduce it to kinetic-apollo-wet.git. 
Second, clone all the sources with wstool and put them into apgit dir.

copy the build.sh script from apollo-platform repo to apgit dir.

Then we have our minimal kinect source tree and can be tested with ./build.sh build, fix the dependencies if you get errors untill all the packages was built successfuly.

```
mkdir kinetic-ws
cd kinetic-ws/
rosinstall_generator desktop_full --rosdistro kinetic --deps --wet-only > kinetic-desktop-full-wet.rosinstall.git

wstool init -j8 apgit kinetic-apollo-wet.git
wstool update -j8 -t apgit

#modify to add new package, eg. octomap
wstool merge -t apgit kinetic-apollo-wet.git
wstool update  -t apgit octomap/octomap
```
### Add protobuf patches

- 1. add pb_msgs pb_msgs_example packages
- 2. patch genmsg
- 3. patch gencpp
- 4. patch genpy

- 5. patch roscpp_core/roscpp_traits
- 6. patch roscpp_core/roscpp_serialization
- 7. patch roscpp_core/cpp_common
- 8. patch ros_comm/roscpp

After adding these patches, build with:
build.sh build

### Test with talk-listener example

After build finished, we can test protobuf messages with the talk-listener example:

```
$ source ~/playground/kinetic-ws/apgit/install/ros_x86_64/setup.bash 
$ roscore &

zhxt@linux-y3ek:~/playground/kinetic-ws/apgit/install/ros_x86_64/lib/pb_msgs_example$ ./pb_listener &
[2] 8781
zhxt@linux-y3ek:~/playground/kinetic-ws/apgit/install/ros_x86_64/lib/pb_msgs_example$ ./pb_talker 
[ INFO ] [1539492583.071544874]: Hello world 0
[ INFO ] [1539492583.171751808]: Hello world 1
[ INFO ] [1539492583.271984640]: Hello world 2
[ INFO ] [1539492583.371798069]: Hello world 3
[ INFO ] [1539492583.372883499]: Time: 1539492583.371691627
[ INFO ] [1539492583.373110708]: I heard pb short message: [Hello world 3]
[ INFO ] [1539492583.373218430]: I get counter message: [3]
[ INFO ] [1539492583.471748620]: Hello world 4
[ INFO ] [1539492583.472618782]: Time: 1539492583.471656372
[ INFO ] [1539492583.472749793]: I heard pb short message: [Hello world 4]
[ INFO ] [1539492583.472834024]: I get counter message: [4]
[ INFO ] [1539492583.572173081]: Hello world 5
[ INFO ] [1539492583.572985319]: Time: 1539492583.572082529
[ INFO ] [1539492583.573119901]: I heard pb short message: [Hello world 5]
[ INFO ] [1539492583.573209616]: I get counter message: [5]
[ INFO ] [1539492583.671694881]: Hello world 6
[ INFO ] [1539492583.672237747]: Time: 1539492583.671632944
[ INFO ] [1539492583.672302738]: I heard pb short message: [Hello world 6]
[ INFO ] [1539492583.672340937]: I get counter message: [6]
[ INFO ] [1539492583.771680287]: Hello world 7
[ INFO ] [1539492583.772357171]: Time: 1539492583.771604271

```

### Troubleshooting:

- fix catkin_pkg dependency
```
pip uninstall catkin_pkg
#build catkin_pkg
PYTHONPATH=/home/zhxt/playground/kinetic-ws/apgit/install/ros_x86_64/lib/python2.7/site-packages python setup.py install --prefix=/home/zhxt/playground/kinetic-ws/apgit/install/ros_x86_64
```
Acturally, it's not that easy as we think :D

Stay tuned, the other two parts comes in next few days. Patches will be open when all the three parts finished.

