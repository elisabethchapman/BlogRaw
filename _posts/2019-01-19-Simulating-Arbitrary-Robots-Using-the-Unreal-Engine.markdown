---
layout: post
title:  "Simulating Arbitrary Robots Using the Unreal Engine"
date:   2019-01-19 19:30:55 -0700
comments: true
tags:
  - Python
  - C++
  - AirSim
  - Simulation
  - Unreal Engine
---

# Introduction
One of the challenges associated with building robots on a production scale is verification of robot behavior in a variety of environments. Robots that are built not only need to function in a laboratory setting, but also in a wide variety of environments that are difficult and expensive to recreate. For example, an engineer designing a robot to mine coal would like to be able to test his creations inside of the actual coal mines in which they will be running. Robots designed to work on nuclear reactors would ideally be tested in the environments they will be working. Autonomous vehicles need to be tested in a variety of driving conditions. All of these tests are necessary to ensure a robust product is developed, but are too difficult to efficiently create in a laboratory setting.

In lieu of physical testing, many engineers have turned to simulation solutions. The value proposition is easy to recognize - rather than create physical prototypes and test them in the real world, we can create virtual representations of the robot and the environment, and use this to test our algorithms. Although the simulated world will never match the real world exactly, it will allow for rapid iteration and testing in tons of different environments, many of which would be impractical or even impossible to create in the real world. In addition to the verification capabilities, a simulator will allow for the generation of rich datasets which can be used for machine learning and model development. It can even enable new algorithms to be developed, such as deep reinforcement learning-based solutions, which would be infeasible with a physical robot.

The problem with simulation is that it is extremely difficult and expensive to create an accurate model of reality. Some of the difficulty is obvious - creating a performant physics engine that can handle thousands of objects is difficult. Accurately modeling noise in sensors is difficult. Creating a real-time rendering engine is difficult. Some of the difficulty, however, is not so obvious. As a prime example, consider asset creation. Every mundane object that will appear in the simulated world needs to be designed. That tree in the background? Someone needs to draw the visual mesh, create the collision mesh, create the texture maps, and export all of the files into a usable format. This can take days for even the most skilled technical artist. As the simulations become more and more realistic, more and more of these assets need to be created.

Fortunately, this problem, and a lot of other problems, have already been solved by a different vertical - the gaming vertical. For decades, the video game industry has poured extensive amounts of time, effort, and money into developing the most realistic artificial worlds possible. Tricky technical problems have been solved and wrapped in easy-to-use libraries, and tons of assets have been created for use (free or otherwise). This naturally leads to the question of how we can re-use this work to solve the simulation problem. Attempting to answer this question led to the AirSim project.

# AirSim 
AirSim is a project that sprang out of the Microsoft Research (MSR) labs. The initial goal of AirSim was to build a simulator for drones. Rather than re-write the simulator from the ground up, the AirSim team used the Unreal Engine as the backbone of their solution. The Unreal Engine, maintained and developed by Epic Games Corp, is an open-source game engine. Although the learning curve is steep, using the Unreal engine for a solution gives a lot of benefits:

* Access to the Unreal Store and Unreal Editor, which makes including content developed by third parties extremely easy.
* A robust, efficient rendering engine capable of rendering realistic textures in real-time.
* A robust, efficient physics engine (in the form of PhysX), which supports many advanced features such as constraints.
* A c++ interface and SDK that simplifies a lot of the mechanics and details of game programming. Chances are, most tasks that need to be done can be represented by a library function call.
* A robust and efficient standard library.
* Garbage collection.
* A cross-platform build system which allows binaries to be compiled for a variety of systems (most notably, Windows and Linux).
* Visual Studio integration, which helps speed development.

The Unreal Engine is not perfect, however, and comes with some drawbacks:
* Slow compile times. Include What You Use (IWYU) helps, but builds can still take minutes on a beefy machine.
* A complex build process. One needs to build a program (the Unreal Header Tool, or UHT) to build their program.
* Sparse documentation. One frequently needs to read the source code to figure out what functions do. To Epic's credit, the unreal source is very well maintained and easy to read.
* Some rough edges around the SDK, which can lead to unexpected results. For example, garbage collection will occasionally free UObjects that reside within built-in collection types that are not TArray&lt;>.This is documented one one of the tutorials, but somewhat unexpected.

In the end, however, Unreal turned out to be a very useful engine, and with it, AirSim began to thrive. The ability to bootstrap a simulation by using content developed by third parties allowed users to get up and running much quicker than with other simulation solutions. Using an open source simluation engine also allowed contributions to be received from many parties (like the automobile pawn from yours truly :) ), and AirSim quickly grew into a capable simulator. Although all sorts of impressive features were being added, AirSim at its core was a drone and automobile simulator. While these are two of the most popular types of robots in the public perception, there are many more robots that are in use today. For example, robotic arms are used in factories all over the world to assemble automobiles and package goods. Robotic forklifts are in development by a variety of different startups. Autonomous security robots are currently being deployed to replace traditional security guards. All of these use cases would benefit from simulation, but could not be served by AirSim. This led to the question - is it possible to create a simulator that can simulate any type of robot? With this goal, I set out to develop UrdfSim.

# UrdfSim
Many developers working with robots leverage the Robotic Operating System (ROS) framework. This framework has a variety of tools for working with robots, and even has its own simulation solution (Gazebo). While Gazebo does not allow one to introduce third party content in an easily of a manner as AirSim, it is designed for working with arbitrary robots, so it pays to study their design. Their design revolves around the Universal Robot Description File (URDF), which is an XML which describes the robot's architecture. This file is a joint-and-link architecture, which describes robots by rigid bodies, called "links", which attach to each other via "joints." There are a variety of different joint types, which control the physics of the robot. After learning this, the goal became to see if we can augment AirSim with capabilities of parsing the URDF file. 

One of the interesting properties of the URDF file is the ability to specify custom geometry for the links. On the fly, the user can specify the different geometry types that make up the robot, which then get parsed by the engine and used in the solution. In order to recreate this functionality in Unreal, there are a variety of challenges that need to be solved:
* Generally, mesh data is statically baked at compile time. This means that when the game is created, the data behind each of the meshes are computed and compiled into the final executable. Creating a static mesh on the fly is quite difficult. Fortunately, there are plugins that have done a lot of the heavy lifting, and (with a bit of work) allow for the creation of these meshes at run time. 
* Like many other physics engines, Unreal can only properly simulate physics on convex meshes. This is due to efficiency reasons, among other things. For example, Unreal would be unable to simulate physics on a doughnut, as there is a hole in the middle which makes the shape non-convex. In order to simulate the mesh, one needs to divide up the single concave mesh into a series of convex meshes, in a process called "Convex Mesh Decomposition." Solving this problem exactly is a NP hard problem, but approximate solutions exist. 
A future post will cover dynamic mesh creation in much more detail, but UrdfSim solves it. 

After 9 months of development, the URDF-based simulation is complete. It supports an impressive feature set that is not present in other solutions:
* Arbitrary robot creation via URDF file. This gives a clear workflow for exporting robots designed in 3-d modeling programs like Blender or Solidworks and importing them into Unreal.
* Dynamic mesh generation allows for complex geometries to be simulated with minimal overhead from the user.
* Because the system leverages the Unreal Engine, we can take advantage of all of the engine features for developing accurate simulations such as advanced material properties.
* Because the system leverages AirSim, we have access to all of the advanced sensor models such as segmentation cameras and LIDARs. 
* The user does not ever have to open the Unreal Editor to use UrdfSim. Everything can be compiled down into a single exe that can be distributed to end users for consumption.

Here are a few GIFs of some sample robots in action!

![Lunabot]({{site.url}}/content/Simulating-Arbitrary-Robots-Using-the-Unreal-Engine/Lunabot.gif)
A recreation of the Lunabot from the 2012 competition. In addition to showing off the capabilities of the system to accurately capture the motion of a complex robot, it also shows off the debug mode, which makes it easier to construct the necessary simulation files. 

![Arm]({{site.url}}/content/Simulating-Arbitrary-Robots-Using-the-Unreal-Engine/Arm.gif)
A 4-DOF robotic arm.

![CameraOnArm]({{site.url}}/content/Simulating-Arbitrary-Robots-Using-the-Unreal-Engine/CameraOnArm.gif)
The same robot as above, but with the camera attached to the arm. 

[Here is a link to the repository for the plugin](https://github.com/mitchellspryn/UrdfSim).

# Analysis of solution
After every project, it's important to look back on the project and determine what went well, what went poorly, and what knowledge can be transferred to new projects. First, let's take a critical look at the simulator. We have already discussed the highlights of the simulator in the previous paragraphs, but we should analyze some of the shortcomings:

* **The simulator is not fully URDF compatable**. There are some tags which are not easy to emulate in the engine. For example, it is not easy to have a object that is a spherical visual mesh, but has rectangular collision - it's simpler if they match. Lots of time and complexity was spent on an effort for full compatibility; this led to a difficult to develop solution. Many of the advanced features are rarely used, however, so the solution could have been simplified by reducing the number of supported features. This is a key insight - when developing a new product, always look for the absolute minimum feature set that can be deployed and be useful. Users will complain about missing features, and then it will be clear which features need to be implemented.
* **Some physics modes are difficult to support generically**. For example, floating isn't implementable generically in the Unreal engine. If someone wants to make a boat, for example, there is a lot of custom set-up that must be done. This means that it's probably not possible to create a 100% generic simulator. However, covering the majority of use cases is still a valid outcome.
* **PhysX is a physics engine designed for speed, not for accuracy**. This can become clear when attempting to develop complicated robots with lots of constraints. In particular, "chaining" constraints among multiple meshes seems to lead to poor results, as the simulation will oscillate (and in extreme cases, break apart). This could potentially be mitigated with a few mechanisms:
    * **Careful URDF design**: Instead of chaining together a bunch of boxes to make an "A" shape with a fixed constraint, instead make a single mesh. This takes some of the load off the engine at the expense of usability.
    * **Switching to Actuation instead of joints**: In the Unreal Engine, joints are ultimately represented by PxJoint objects via PhysX. In the latest versions of PhysX, there is support for an object type called "Actuation", which theoretically handles these types of workloads (complex series of constraints) better. However, this object type is not currently supported as a first-class citizen in the engine (i.e. there is no UClass wrapper). Perhaps someday, the engine will support this type and it will become easier to use. 
    * **Using a different physics engine**: Something like Bullet, which is designed for more accurate physics. However, PhysX is deeply integrated with Unreal, and using a separate physics engine is a massive undertaking. 
For now, the current solution should fit most use cases, but there is definitely room for improvement
* **Some of the sensor models can be improved**. In particular, the LIDAR model is a simple ray cast LIDAR. Is it possible to make a more accurate one, perhaps with MC-MC simulations? If so, could it be made real-time?

So although the simulator is quite good, there is still work that can be done to improve it. Even with these shortcomings, however, UrdfSim is still a useful tool that solves a lot of problems. What about more general development principals that can be learned from developing UrdfSim?

* **Don't be afraid of reading the source code**. Initially, development with the Unreal Engine was a frustrating experience, as the documentation is very sparse (especially when compared with other engines like Unity). After struggling, I realized that I actually had a much better version of the documentation right in front of me - the actual source code. Reading the source code trumps documentation, as it is the actual code that is being executed. Docs can be wrong, but the source doesn't lie. This is not to say that having good documentation is useless - far from it. But after awhile, I found myself looking at the source code before the documentation as it became quicker to find the answer.
* **Determine what is a minimal viable product and *WRITE IT DOWN* before beginning work**. Feature creep, especially on products designed to solve a "general" problem like this one, is difficult to combat. The easiest way to fight it is to determine at the outset what features will be supported, and write them down. Don't change it unless there is a very good reason. Writing down the feature definition is critical, otherwise it's easy to argue that a feature was "always part of the plan."
* **Finding a way to visualize a process is the quickest way to debug it**. Initially, there was a lot of effort expended into the creation of the coordinate transformations between different robot parts. Lots of sheets of paper were spent attempting to solve the problems, but the transformations were still not right. It wasn't until I implemented a "debug mode" which drew the different coordinate axes until the logic fixes became clear. This pattern repeated itself throughout the project - if it wasn't possible to visualize a component, it was much more difficult to debug. It's worth it to spend the time up-front to create nice debug visualizations before implementing the complex system logic for a component.
* **Be careful about includes**. Unreal's IWYU does a good job of excluding irrelevant portions of the engine from the build, but it can backfire if every engine file starts to get included haphazardly. This can bloat build times dramatically. This isn't just an Unreal problem, though - C++ include bloat is a problem everywhere. 

# Summary
In this post, we introduced a new simulator, UrdfSim, which allows for the simulation of arbitrary robots. This simulator leverages AirSim and the Unreal Engine to create a system which allows users to rapidly develop realistic simulations for a variety of applications. This will allow for users to develop more robust, reliable robotic systems that can operate in a variety of different environments that are difficult or expensive to recreate in the laboratory, making simulation more accessible to end users.
