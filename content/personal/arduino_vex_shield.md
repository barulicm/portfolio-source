---
title: "VEX Shield for Arduino"
date: 2019-08-06T19:11:47-07:00
draft: false
summary: "This is an Arduino shield I've designed for making it easier to interface with VEX electronics (servos, sensors, etc.) from an Arduino microcontroller."
tileImage: "images/projects/VexShieldRender.png"
year: "2014"
sortYear: "2014"
topImage: "../../images/projects/Arduino_Vex_Shield_headerimage.png"
---

The [VEX robotics design system](http://vexrobotics.com) includes a lot of electronic motors and sensors that are nice and easy to use. Controlling them from an [Arduino](http://arduino.cc) microcontroller is not very complicated, but requires a lot of wire routing and power distribution that can turn a breadboard into a bird's nest. This shield takes care of routing the necessary power and signal pins to provide convenient 3-pin header connections that VEX components naturally plug in to.

![](../../images/projects/VexShieldRender.png)

The board was designed in [Eagle CAD V6.4](http://www.cadsoftusa.com/). This was my first Eagle CAD project to go from idea to finished design. The software is actually incredibly nice to use, once you learn the conventions of the interface. A compressed folder of the design files is available [here](../../projectfiles/ArduinoVexShield.zip).

The bill of materials for this board is available [here](../../projectfiles/VEX%20Shield%20BOM.pdf). Estimated cost for components is $15.39. This does not include the cost of the printed PCB.

**NOTE**: The 5.1K and 150 ohm resistors in the parts list were selected as "close enough" to the desired 5K and 140 ohm resistors.
