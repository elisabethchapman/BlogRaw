---
layout: post
title:  "Extracting Images from a RosBag"
date:   2017-05-20 19:30:55 -0700
comments: true
tags:
  - ROS
  - Bash
---

[Ros](http://www.ros.org/) is the framework powering most complex robots today. It comes with a lot of benefits - one of the most important being the [rosbag](http://wiki.ros.org/rosbag). Rosbags are incredibly versitile and useful data storage containers, being able to store any sort of data that can be passed between nodes within a system. Using rosbags, it is possible to save data from a test run, make modifications to the code, and "replay" the test run with the new code using the collected data. This powerful capability reduces the need for manual testing, decreasing implementation time and improving code quality. 

This flexibility has a price, however. It's not trivial to convert the data from the rosbags into other data formats, such as csvs or tsvs. The data is stored in a binary format, so it can't be copy-pasted or visualized directly. If you want to get the data, you have two options:

* Write some code using the rosbag API to extract the data. This allows you the most flexability, but comes at the price of having to write a program for each custom datatype. 
* Write a node that saves topic data to disk. Then, pipe the data into the node

Both of these take time and are kind of annoying. One of the common scenarios that I keep running into regards rosbags that are composed of [images](http://docs.ros.org/api/sensor_msgs/html/msg/Image.html). Frequently, I have a rosbag of these messages (say, saved from a webcam), and I want to extract the images to use to build a ML model. Or, I'd like to visualize the images in the bag as a video. Previously, doing this would require a few steps:

* Create a temporary directory
```
$ mkdir tmp
```
* Start roscore in a terminal window 
```
$ roscore
```

* Open a separate terminal window. Cd into the temp directory. Start ros's image_view package in another terminal window, to save to disk. 
```
$ rosrun image_view extract_images _sec_per_frame:=$SEC_PER_FRAME image:=${TOPIC_NAME}
```

* Open yet another terminal window. Pipe the images in the rosbag to the topic
```
$ rosbag play ${BAG_PATH}
```

* Wait for that to finish. Kill the terminal windows running rosbag and roscore. 
* You now have the images in the temp directory. Next, run the images through mencoder (ffmpeg can be used as well)
```
$ mencoder "mf://*.jpg" -mf type=jpg:fps=${FRAMES_PER_SECOND} -o ${OUTPUT_FILE_NAME} -speed 1 -ofps ${FRAMES_PER_SECOND} -ovc lavc -lavcopts vcodec=mpeg2video:vbitrate=2500 -oac copy -of mpeg
```

This process works beautifully, producing an output .mpg video of the images that are in the topic data. However, there are a few obvious disadvanatages:

* Three terminal windows need to be opened. Yes, you can ctrl+z the processes, but then you need to go through and kill the running roscore and rosbag processes at the end.
* The commands are kinda long, and it's easy to make a typo
* One of the commands reqires "frames per second" as an input parameter, and another requries "seconds per frame." 

I've found myself having to do this task so frequently, I went ahead and made a bash script that performs the above tasks. It takes care of managing the ROS processes, and cleans up the running ROS processes. It can even clean up the temporary directory used to extract the images, saving disk space. It can be used as follows:
```
$ ./Bag2Vid.sh <flags> Bagname
```

The following flags are available
* **-p**: This flag will allow the specification of the temporary file path to use. If you want the extracted images to live in a specific directory, you can use something like `-p /path/to/images`. If not set, then the script will create a directory called 'tmp' in the current directory.
* **-o**: This flag will allow you to change the output video file name, with something like `-o filename.mpg`. If not set, the filename defaults to 'output.mpg'
* **-s**: If -s is passed, then the images will be saved. This can be useful if you are trying to feed the images into a Machine Learning model. By default, the image directory will be deleted after the video is created.
* **-f**: This parameter sets the frames per second of the camera. For example, to use this script with a camera that runs at 20 fps, use `-f 20`. By default, the script assumes that the camera runs at 30 fps.
* **-t**: The topic name for the rosbag. So, if the rosbag recorded from topic '/foobar', pass `-t /foobar`. This parameter is required, as there is no real way to set a sensible default.
* **-n**: If this parameter is passed, then the mencoder command will be skipped, and the video will not be created. This is useful with the -s flag.

The script can be pulled [from this github repo](https://github.com/mitchellspryn/useful-ros-scripts){:target="_blank"}.
