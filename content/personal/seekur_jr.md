---
title: "Seekur Jr."
date: 2019-08-06T19:12:47-07:00
draft: false
summary: "This project is from my internship at GTRI ELSYS in 2012. The project uses a kinect to autonomously map and navigate its environment."
tileImage: "images/projects/SeekurJr.jpg"
year: "2012"
sortYear: "2012-02"
topImage: "../../images/projects/Seekur_Jr_headerimage.png"
---

{{< youtube CHovZuMBuj4 >}}

This project is the result of my work at GTRI ELSYS during the summer of 2012. The project equips Mobile Robot's Seekur Jr robot with a kinect, which it uses to autonomously generate a map of its environment while navigating successfully to a user-defined destination.

The [code](../../projectfiles/KinectAndAStar_WithVoiceControl_RawCode.zip) and a [summary](../../projectfiles/KinectandASummary.pdf)  are available.

The robot uses the [Point Cloud Library (PCL)](http://www.pointclouds.org) to process the 3D data from the kinect.

[Mobile Robot's ARIA library](http://www.mobilerobots.com/Software/ARIA.aspx) was used for communication with Seekur Jr's microcontroller.

Voice commands and feedback were added using [Microsoft's Speech API (SAPI)](http://www.microsoft.com/en-us/download/details.aspx?id=10121).

The only additional sensor added to the robot was [Microsoft's Kinect sensor](https://developer.microsoft.com/en-us/windows/kinect).

This project was developed using Visual C++ 2010 Express Edition (now part of [Visual Studio](https://www.visualstudio.com/).
