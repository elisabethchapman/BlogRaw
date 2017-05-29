---
layout: post
title:  "Automated Pest Deterrent System"
date:   2015-05-23 19:30:55 -0700
comments: true
tags:
  - embedded systems
  - robotics
  - project
  - machine learning
  - c plus plus
  - image processing
  - computer vision
---

		
One of the most insidious problems faced by the United States has gone largely unnoticed: super deer. Not just your average deer, but super deer. These deer are actively sabotaging every attempt I, and many of my other neighbors attempts' to grow vegetables and plant flowers. Although it may seem as if I'm being a bit hyperbolic, there is merit to investigating this problem: deer account for over $350 million in crop destruction annually. 

While most people attempt to scare the deer by using pepper sprays or flashy lights, I found that the most effective method of deterring deer was by "...stimulating..." them with a pellet gun. While the deer happily ignored other deterrent methods, they would run at the slightest hint of contact from a pellet gun. Although it was quite fun chasing after deer, it does not make sense to have a human monitoring the gardens 24/7. Thus, my design team decided to attempt to automate this process, creating what we dubbed the "Deerinator."

![Cover Image]({{site.url}}/content/Automated-Pest-Deterrent-System/CoverImage.jpg)

The Deerinator mechanically consists of an electric airsoft gun mounted atop a PVC base. A webcam is mounted on the gun in line with the barrel. In the center of the base, there is a waterproof bucket in which two servos are mounted, allowing the weapon to actuate. Additional electronics are housed inside of the bucket: a beaglebone black for control and communications, a wi-fi router for wireless connection, and a repurposed computer power supply used to provide regulated power for all onboard devices. 

![Electronics]({{site.url}}/content/Automated-Pest-Deterrent-System/Electronics.jpg)

There are two modes of operation for the device. In the first mode, the user connects to the device using a GUI-client. This client establishes a TCP/IP connection to the beaglebone black. Once the connection has been established, the client listens for user input from the keyboard and sends the user's commands to the beaglebone. The beaglebone receives the commands and actuates the hardware appropriately, as well as streams the webcam data back to the user interface client. 

The other mode of operation of this device is an autonomous mode. In this mode, the beaglebone black processes the video feed in order to determine if there is a deer in the range of fire of the device. If a valid target exists, then the gun is actuated to face the center of the target, and then it is fired. To determine if there is a deer in the video feed, the video feed is first processed to separate the foreground image from the background image using a Mixture of Gaussians (MOG) background subtraction method (second image below). Then, Scale Invariant Feature Transform (SIFT) keypoints are evaluated at a subset of the foreground datapoints (third image below). Finally, the SIFT keypoints are then run through a Support Vector Machine (SVM) classifer to determine if the data points represent a deer or not (fourth image below. Green circles are hits, red are misses). If enough points are identified as "deer," then the firing sequence is initiated.

![Stages]({{site.url}}/content/Automated-Pest-Deterrent-System/deerStages.jpg)

Although the algorithm managed to identify deer, it seemed to have high false positive rate when presented with sharply contrasting textures. If this project were to be repeated, this failure mode could be mitigated by examining clusters of points rather than singular points, allowing for the development of richer feature sets. Also, the algorithm runs slowly on the beaglebone black; one may need to upgrade to a board with more processing power such as a cubieboard. 

This project utilized low-level linux c++ programming as well as image processing using the OpenCV API. In addition, there was much work done with system integration, as many different components needed to be integrated together in order to ensure that the final system performed as expected. The code for the daemon can be downloaded from the [github repo](https://github.com/mitchellspryn/Deerinator).
