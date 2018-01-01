---
layout: post
title:  "Project RoadRunner: A Journey Through the Nitty-Gritty of Self-Driving Cars"
date:   2017-12-31 12:00:00 -0700
comments: true
tags:
  - Machine Learning
  - Artificial Intelligence
  - Autonomous Driving
  - Simulation
  - AirSim
  - Unreal
---

# Introduction

Self-driving cars are one of the hottest topics in the robotics community. It's one of the most difficult challenges faced by engineers today, posing difficult problems in mechanical engineering, data science, and computer engineering. The difficulty lies in the complexity of the task, combined with the small allowable margins of error - a failure of any piece of the pipeline could easily result in disaster. Although it's a difficult problem, the reward for solving this problem will be great. Accordingly, every major automobile company, as well as a slew of startups, have devoted tons of effort into solving the self driving problem. It's a race to see who can solve the problem first. Up until this point, it's a race that I haven't been in - and it's time to fix that! This post will detail the plight of Project RoadRunner - an attempt by a few software engineers to build a self-driving car from the ground up.

# Beginnings

Our team began with the goal of replicating the [NVIDIA Lane-Assist paper](https://arxiv.org/pdf/1604.07316.pdf) on a hardware platform. In this paper, the goal of the NVIDIA team was to predict the correct steering angles of the car from a front-facing webcam placed on the dashboard. The NVIDIA team showed that it was possible to get good results using a deep neural network as the predictor. Although the NVIDIA team used a standard automobile for their work, we were not able to acquire one, so we settled on a hacked RC car as our hardware testbed. In addition to being a testbed, we wanted to use the hardware platform to gather training data. Therefore, we also needed to build in the capability for manual control. With this goal in mind, we began working.

# The Hardware Platform

![Testbed]({{site.url}}/content/Project-Roadrunner/TestBed.jpg)

The hardware platform was a pretty straightforward build. The RC car came with an ESC that handled control of the motor via PWM. We knew that we were going to have to run our deep learning models in real time, so we wanted a computational platform that had a GPU on it. Luckly, we were able to acquire a Jetson TX2 to use for testing. The Jetson is one of the most advanced boards out of NVIDIA's lab (at the time of this writing :P), containing a powerful quad-core ARM processor as well as an embedded GPU with 192 CUDA cores and 4GB of memory. It's a powerful board, but there were a few software kinks to work out:

* Turns out, the TTL USART ports don't work properly. It's a known issue. This is a problem because we were planning on using an embedded microcontroller to handle the PWM signals (and other eventual sensors that we'd want to integrate), and the easiest method of communication would be via serial. 
* Using a USB to Serial adapter was also a nonstarter initially; when plugged in, it did not show up under /dev. Turns out, with the default version of the kernel, the TTYACM drivers are not enabled (?!). Fortunately, the kind folks at JetsonHacks have scripted up the kernel build process, so rebuilding the kernel to enable this feature is actually pretty painless. 

Once the hardware issues were worked out, the coding was a straightforward exercise in ROS development. Nodes were created for grabbing webcam images, running the ML model, communicating to set the microcontroller to set the PWM signal, and for a simple TCP/IP server to allow manual control. A python client was then used to read the inputs from an XBOX controller and push them to the car over a TCP/IP connection. Here is a short video of the car in action:

<video width="680" height="480" controls>
  <source src="{{site.url}}/content/Project-Roadrunner/RoadrunnerDriving.mp4" type="video/mp4">
  Oops, your browser doesn't support direct video playback!
</video>

The prototype was functional, but we quickly ran into some problems:

* We could collect very limited amounts of training data. There was no way to safely take the car out on to the interstate. There was no way to take the car out in a rainstorm. Basically, the only dataset we could collect was the car in a parking lot or on a small road.
* The wifi-based solution was convenient, but we ran into network range issues. This could potentially be solved by switching to a different radio technology for control (such as FM), but then there would be bandwidth issues transmitting the camera feed back to the driver.
* The footprint for the testbed is very different than the footprint of an actual car. While we successfully managed to raise the webcam up to a reasonable height, it would be difficult to mount multiple cameras or additional sensors (e.g. LIDAR, sonar, etc) in a location that would both be similar to an actual car and allow the testbed to maintain stability. 
* The testbed handled very differently from an actual car; much differently than we expected. Turning was a lot tighter, and start / stop times were much less than those of an actual car. 

# The Move to Simulation

![UdacityScreenshot]({{site.url}}/content/Project-Roadrunner/UdacityScreenshot.png)

With all of the problems we encountered with the hardware testbench, it became apparent that we would need to fundamentally rethink our approach. Generating the data via hardware means seems out of the question, as it was unlikely that we'd be able to get our hands on an actual vehicle to use for experimentation. Thus, the most promising path forward appeared to be simulation in software. As it turns out, there have been a variety of simulators released with the intent of allowing researchers to develop self-driving algorithms. Of these simulators, the most promising at the time was the one released by [Udacity](https://github.com/udacity/self-driving-car-sim). This simulator is built using the Unity video game engine, and allows the user to drive a car around one of two provided tracks via a python API. The code is open-sourced, so we could modify it to support more involved experiments. This simulator seemed to be just what we needed!

We began working with the simulator. In just a few hours, we were able to generate datasets orders of magnitude larger than the datasets were able to grab from the hardware testbed. We were able to train a deep neural network to drive the car successfully through the course and demonstrated it at the Microsoft MLADS Spring 2017 conference (a Microsoft-internal conference for data scientists). While this was a major accomplishment for our group, we realized that there was still a lot of work to do on the simulation side:

* The images generated from the simulator were not very realistic. Shown below is a sample image pulled from the simulator. These images are low-quality, and do not look like anything that would be encountered in the real world. This means that algorithms developed in the simulator are likely to need retraining on actual data (or worse, will not work at all!).

![UdacityImage]({{site.url}}/content/Project-Roadrunner/UdacityImage.jpg)

* There are few maps supported in the simulator. At the time we experimented with the simulator, there were only two courses - a racetrack in the desert and a road through a jungle. This is not sufficient to train any general-purpose algorithm. More importantly, we would be unable to work on the most difficult area of the self-driving car problem: city driving. Any additional maps would need to be created from scratch.
* The code base is not being actively developed. The simulator was created for an assignment in the [Udacity self-driving car nanodegree](https://www.udacity.com/course/intro-to-self-driving-cars--nd113?gclid=Cj0KCQiAsqLSBRCmARIsAL4Pa9Svyx7MZMj4wTaugA2SA-6RiVDAGlxbWsNEooqsDra8lRILsaukWAkaAhpPEALw_wcB), and thus only needs a limited featureset. 

So, while our team could have continued working with the Udacity simulator, we began exploring other options. It was during this phase that we discovered AirSim.

# AirSim

![AirsimScreenshot]({{site.url}}/content/Project-Roadrunner/AirsimScreenshot.png)

[AirSim](https://github.com/Microsoft/AirSim) began its life as an open-source simulator for drones. Specifically, it was designed as a tool to allow researchers to develop drone control algorithms in the simulator and directly translate them to real-world drones. Lots of care had gone into the design to support interesting features such as custom physics models for each drone type and hardware-in-the-loop simulation. Unlike the Udacity simulator, AirSim is built around the [Unreal](https://www.unrealengine.com/en-US/what-is-unreal-engine-4) game engine. While more difficult to use, this engine offers two distinct advantages over the Unity engine:

* It is much easier to create high-quality assets using the Unreal framework. For example, compare the below screenshot with the screenshot from the Udacity simulator presented earlier. This screenshot has much more detail, meaning that algorithms developed against datasets derived from the Unreal engine are more likely to work on real-world datasets without modification.

![AirsimImage]({{site.url}}/content/Project-Roadrunner/AirsimImage.png)

* Unreal has an awesome assets store. Lots of effort has gone into developing high-quality assets for video games. Rather than needing to develop assets from scratch, we can re-use these assets in our simulations, allowing us to quickly generate new worlds. Although Unity has an asset store, the Unreal asset store is much more comprehensive, sometimes containing entire maps that can be plugged in out-of-the-box!

AirSim seemed promising, but there was one catch - currently, the only vehicle type supported was a drone. Automobiles were not supported in the simulator. Fortunately, the AirSim team was open to collaborating with our team, and we worked together to add the vehicle to the simulator. Once we added the car, we began to see much more interest in AirSim. It turns out that there is a lot of interest for an open-source simulator for self-driving cars, and AirSim was one of the highest-quality freely available simulators. We saw [multiple articles](http://www.wired.co.uk/article/microsoft-airsim-crash-drone-car) on [high-traffic news sites](https://economictimes.indiatimes.com/news/international/business/microsoft-extends-ai-research-to-self-driving-vehicles/articleshow/61816724.cms) showing interest in the automobile aspect of AirSim. We officially released AirSim at the 2017 SuperCompute conference in Denver, CO. For that conference, we also repeated our DNN experiment that we performed on the Udacity simulator. Below is our demonstration booth (I'm at the right! Left is one of my teammates, [Aditya Sharma](https://www.linkedin.com/in/adityasharmacmu/))

![SuperCompute]({{site.url}}/content/Project-Roadrunner/SuperCompute.jpg)

# The Autonomous Driving Cookbook

One of the issues with AirSim as compared to other simulators on the market is the lack of material for novice users. Because AirSim was initially developed as a research tool, the instructions and tutorials on the repo are designed for the audience of seasoned researchers. This makes it less accessible for those who want to begin learning about self-driving cars. To fill this gap, our team decided to develop the "[Autonomous Driving Cookbook](https://github.com/Microsoft/AutonomousDrivingCookbook)." The vision for this cookbook is a set of tutorials that would cover all of the basic building blocks of Autonomous Driving in an easy-to-digest fashion for users. This would drive interest in AirSim and the autonomous driving field in general. The starting point for this cookbook was obvious - the DNN that we have demonstrated for SuperCompute would make a very good initial tutorial. After writing up the tutorial, we released the Cookbook in December of 2017. The initial reaction was great - within the first week, we received over 20 stars, and many downloads of the dataset for the tutorial.

# Future Work

The immediate goal for our group is to add additional scenarios to the cookbook. Already, we are internally working on three different tutorials that should be available soon! In addition to augmenting the cookbook, our team will continue to work on adding additional functionality to the AirSim simulator to allow us to increase realism and add additional functionality (e.g. simulated 3-d LIDAR scans). Finally, once we are also to looking to bring our experimentation back to the hardware realm by modeling our RC testbench inside AirSim, training a model based off of simulated data, and transferring the trained model to the testbench. The future looks bright for Project RoadRunner!
