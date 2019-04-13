# Stereo Vision With ROS
This respository contains the launch files to calibrate a stereo vision cameras with ROS and OpenCV and computing a PCL pointcloud from observed data. 
You need a pair of cameras, I bought a pair of this USB webcam which is okay for this task.

## Publishing Camera Data
Save the following text under “stereo_usb_cam_stream_publisher.launch”
```
<launch>

<arg name="video_device_right" default="/dev/video1" />
<arg name="video_device_left" default="/dev/video2" />

<arg name="image_width" default="640" />
<arg name="image_height" default="480" />

<arg name="right_image_namespace" default="stereo" />
<arg name="left_image_namespace" default="stereo" />

<arg name="right_image_node" default="right" />
<arg name="left_image_node" default="left" />

<arg name="image_topic_name_right" default="image_raw" />
<arg name="camera_info_topic_name_right" default="camera_info" />

<arg name="image_topic_name_left" default="image_raw" />
<arg name="camera_info_topic_name_left" default="camera_info" />



<arg name="left_camera_name" default="left" />
<arg name="right_camera_name" default="right" />

<arg name="left_camera_frame_id" default="left_camera" />
<arg name="right_camera_frame_id" default="right_camera" />

<arg name="framerate" default="30" />

<arg name="camera_info" default="false" />
<arg name="camera_info_path" default="stereo_camera_info" />
	  
<node ns="$(arg right_image_namespace)" name="$(arg right_image_node)" pkg="usb_cam" type="usb_cam_node" output="screen" >
	<param name="video_device" value="$(arg video_device_right)" />
	<param name="image_width" value="$(arg image_width)" />
	<param name="image_height" value="$(arg image_height)"/>
	<param name="pixel_format" value="yuyv" />
	<param name="io_method" value="mmap"/>
	<remap from="/usb_cam/image_raw" to="$(arg image_topic_name_right)" />
	<param name="framerate" value="$(arg framerate)"/> 
	<!-- if camera_info is available, we will use it-->
	<param name="camera_frame_id" value="$(arg right_camera_frame_id)" if="$(arg camera_info)" /> 
	<param name="camera_info_url" value="file://${ROS_HOME}/$(arg camera_info_path)/right.yaml" if="$(arg camera_info)"/>
	<param name="camera_name" value="narrow_stereo/$(arg right_camera_name)" if="$(arg camera_info)"/> 
	<remap from="/usb_cam/camera_info" to="$(arg camera_info_topic_name_right)" if="$(arg camera_info)"/>

</node>

<node ns="$(arg left_image_namespace)" name="$(arg left_image_node)" pkg="usb_cam" type="usb_cam_node" output="screen" >
	<param name="video_device" value="$(arg video_device_left)" />
	<param name="image_width" value="$(arg image_width)" />
	<param name="image_height" value="$(arg image_height)"/>
	<param name="pixel_format" value="yuyv" />
	<param name="io_method" value="mmap"/>
	<param name="framerate" value="$(arg framerate)"/> 
	<remap from="/usb_cam/image_raw" to="$(arg image_topic_name_left)"/>
	<!-- if camera_info is available, we will use it-->
	<param name="camera_frame_id" value="$(arg left_camera_frame_id)" if="$(arg camera_info)"/> 
	<param name="camera_name" value="narrow_stereo/$(arg left_camera_name)" if="$(arg camera_info)"/> 
	<param name="camera_info_url" value="file://${ROS_HOME}/$(arg camera_info_path)/left.yaml" if="$(arg camera_info)"/> 
	<remap from="/usb_cam/camera_info" to="$(arg camera_info_topic_name_left)" if="$(arg camera_info)"/>
</node>
</launch>
```

Then run the following node to publish both cameras.
```
roslaunch stereo_usb_cam_stream_publisher.launch video_device_right:=/dev/video1 video_device_left:=/dev/video2 image_width:=640 image_height:=480 left_camera_name:=left right_camera_name:=right
```
Calling the Calibration Node
```
rosrun camera_calibration cameracalibrator.py --size 8x6 --square 0.108 right:=/stereo/right/image_raw left:=/stereo/left/image_raw right_camera:=/stereo/right left_camera:=/stereo/left  --no-service-check --approximate=0.1
```

If you have USB cam with some delays you should add the following “–no-service-check –approximate=0.1”


The result gonna be store at /tmp/calibrationdata.tar.gz. Unzip the file and save it under “/home/<username>/.ros/stereo_camera_info“
  
  [![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/omqbmmUX1E/0.jpg)](https://www.youtube.com/watch?v=omqbmmUX1E)




![alt text](https://img.shields.io/badge/license-BSD-blue.svg)
