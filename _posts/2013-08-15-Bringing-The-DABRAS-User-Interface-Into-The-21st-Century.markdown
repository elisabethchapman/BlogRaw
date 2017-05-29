---
layout: post
title:  "Bringing the DABRAS User Interface Into the 21st Century"
date:   2013-08-15 19:30:55 -0700
comments: true
tags:
  - project
  - c sharp
  - professional
---

The field of particle physics is one of the most fascinating active areas of physical research today. One of the unfortunate side effects of playing with atoms, however, is the generation of radioactive particles. When splitting atoms, it is possible to contaminate an area with radiation; radiation that is hazardous to human health. Despite the depictions in the movies, it can be difficult to detect whether a room has been contaminated by radioactive particles. There generally isn't a green glowing blob! There have been many means developed to detect radiation, but one of the most ingenious methods has been the DABRAS, designed by health physicists at Argonne National Laboratory.

![DABRAS cover image]({{site.url}}/content/Bringing-The-DABRAS-User-Interface-Into-The-21st-Century/CoverImage.jpg)

The Dual Alpha and Beta Radioactive Assay System (DABRAS), is a machine designed to detect extremely small amounts of alpha and beta radiation. Up to six samples taken from the field can be placed inside of the machine at one time, speeding up the process of determining whether a room has been contaminated or not. The DABRAS system was originally developed in the early 1980s, and has been in constant use since then.

![Eww. Dos]({{site.url}}/content/Bringing-The-DABRAS-User-Interface-Into-The-21st-Century/DOS.jpg)

Unfortunately, one side-effect of being developed in the 1980s is that the user interface software that runs with the DABRAS only runs on the DOS operating system. It's clunky, difficult to use, an inefficient, but works. Lately, the machines that the DABRAS had been running on have begun to break due to old age, and it was becoming increasingly difficult to find replacement parts. Emulation was not an option, because the old code extensively relied on system calls that could not be emulated properly. Thus, the decision was made to re-write the user interface, making it more user-friendly an adding functionality.</p>

![C#]({{site.url}}/content/Bringing-The-DABRAS-User-Interface-Into-The-21st-Century/CSharp.jpg)

The new DABRAS user interface client is a C# windows forms application. In addition to allowing the user to perform the statistical counts needed to determine whether the room is clear of radiation, it also adds other useful features such as automatic background calibration, data logging and visualization, and internet connectivity. The new software is currently being used by health physicists at Argonne, and will continue to be used for as long as the .NET 4.0 environment will be supported by Microsoft (i.e. a long time!)</p>
		
