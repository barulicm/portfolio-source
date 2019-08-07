---
title: "DIY Autonomous Car"
date: 2019-08-06T18:15:55-07:00
draft: false
summary: "Raspberry Pi-based, small scale, self-driving car."
tileImage: "images/projects/diy_car_thumbnail.jpg"
year: "2019"
sortYear: "2019-03"
topImage: "../../images/projects/racecar-headerimage.png"
---

With only about $200 in parts, anyone can get started exploring the world of self-driving cars. This page provides step-by-step details about how to build this RC-car-based autonomous vehicle.

{{< youtube xRj47hFZP-0 >}}

### Parts to Buy

This table lists all of the major components I bought to put the car together. In addition to these parts, you'll need materials for making custom parts (I used a 3D printer), and some screws (listed in the second table).

#### Major Components

Part | Vendor | Price
--- | --- | ---
[Turnigy 1 / 18 4WD Mini Stadium Truck (RTR)](https://hobbyking.com/en_us/turnigy-1-18-4wd-mini-stadium-truck-rtr.html) | HobbyKing | $75.15
[Turnigy 1300 mAh 2S 20C Lipo Pack](https://hobbyking.com/en_us/turnigy-1300mah-2s-20c-lipo-pack-suit-1-18th-truck.html) | HobbyKing | $10.64
[Turnigy E3 Compact 2S/3S Lipo Charger (US Plug)](https://hobbyking.com/en_us/turnigy-e3-compact-2s-3s-lipo-charger-100-240v-us-plug.html) | HobbyKing | $13.10
[HobbyKing Lipo Voltage Checker 2S-8S](https://hobbyking.com/en_us/hobbykingtm-lipo-voltage-checker-2s-8s.html) | HobbyKing | $2.11
[Raspberry Pi - 3 Model B](https://www.adafruit.com/product/3055) | Adafruit | $35.00
[Raspberry Pi Camera Board v2](https://www.adafruit.com/product/3099) | Adafruit | $29.95
[Adafruit 16-Channel PWM / Servo Hat](https://www.adafruit.com/product/2327) | Adafruit | $17.50
[Anker PowerCore 5000](https://www.amazon.com/gp/product/B01CU1EC6Y) | Amazon | $21.99
 |Total|$205.44

#### Additional Hardware

Part | Quantity
--- | ---
6-32 x 3/8in. Pan Head Screw | 10
M2 x 8mm | 4
M2.5 x 20mm | 4
11mm Plastic Spacer for M2.5 Screw | 4

These parts can be found at any big box hardware store for a few dollars.

An alternative to the 11mm spacers and long M2.5 screws is to grab [Adafruit's brass standoffs for Pi Hats](https://www.adafruit.com/product/2336) and some shorter M2.5 screws.

### Assembling the Hardware

To prep the car for additional parts, remove the decorative body and the receiver. Install the battery now, as getting the battery in after the extra parts are added is difficult.

To mount the computing hardware onto the chassis, I designed some custom 3D printed parts which clamp onto the rim around the Turnigy truck's chassis. The parts I downloaded can fit one at a time on the [MonoPrice Mini Delta printer](https://www.monoprice.com/product?p_id=21666), which has a very small build area, so they should fit on just about any 3D printer out there. Once the parts are screwed together, the rim of the car's chassis should fit into the slots on the bottom of the support arches, and the screws next to those slots will clamp the assembly to the car.

You can download the STLs I printed [here](../../projectfiles/racecar_3dParts.zip). (These STLs are in millimeters.)

Once the 3D printed parts are secured, connect the steering servo's PWM cable to the PWM hat's channel 0, and the drive motor cable to channel 1. Pay attention to the channel markings and polarity markings (S,5,G) on the PWM hat.

### Setting up the Raspberry Pi

The Raspberry Pi is a super useful embedded computer, but it does require a bit of setup to work with the hardware we've got. The first step, if you're new to using the Pi, is to install Raspian, the default operating system. You can do this with the beginner-friendly [NOOBS installer](https://www.raspberrypi.org/documentation/installation/noobs.md).

Once you're in raspian, we need to enable the Camera and I2C kernel modules. You can do this by opening the Raspberry Pi Configuration program from the Preferences menu. On the interfaces tab, enable "Camera" and "I2C". Press OK and restart the Pi to load the modules.

We're going to use three libraries in our code. OpenCV will help us process the image and make intelligent decisions based on what the robot sees. PiPCA9685 is a library I wrote for easily interfacing with Adafruit's PWM shield to control our motors. raspicam (c++) and picamera (python) let us grab images from the camera. Before writing any code, let's get these libraries installed.

OpenCV is a great library, but it can be a bit tricky to install on the Pi. You'll find no shortage of tutorials on how to install it. Here are the commands I used to succesfully install OpenCV 3 on my Pi.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%"> 1
 2
 3
 4
 5
 6
 7
 8
 9
10</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #888888">sudo apt update &amp;&amp; sudo apt upgrade</span>
<span style="color: #888888">sudo apt install build-essential cmake pkg-config libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libgtk2.0-dev libgtk-3-dev libatlas-base-dev gfortran python2.7-dev python3-dev</span>
<span style="color: #888888">cd ~</span>
<span style="color: #888888">wget -O opencv.zip https://github.com/opencv/opencv/archive/4.0.1.zip</span>
<span style="color: #888888">unzip opencv &amp;&amp; cd opencv</span>
<span style="color: #888888">mkdir build &amp;&amp; cd build</span>
<span style="color: #888888">cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..</span>
<span style="color: #888888">cmake --build . -- -j4</span>
<span style="color: #888888">sudo cmake --build . --target install</span>
<span style="color: #888888">sudo ldconfig</span>
</pre></td></tr></table></div>

If you're working in C++, you'll need raspicam to interact with the camera. You can install it with these commands:

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%">1
2
3
4
5
6</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #c65d09; font-weight: bold">#</span> Download raspicam-0.1.6.zip from https://sourceforge.net/projects/raspicam/files/
<span style="color: #888888">unzip raspicam-0.1.6.zip &amp;&amp; cd raspicam-0.1.6</span>
<span style="color: #888888">mkdir build &amp;&amp; cd build</span>
<span style="color: #888888">cmake ..</span>
<span style="color: #888888">cmake --build .</span>
<span style="color: #888888">sudo cmake --build . --target install</span>
</pre></td></tr></table></div>

Raspicam doesn't seem to install itself totally correctly, so if CMake complains that it can't find the raspicam_cv project when you're building your project, try pointing CMake at the raspicam build directory like this:

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%"><span style="color: #888888">cmake -Draspicam_DIR=/home/pi/raspicam-0.1.6/build/ ..</span>
</pre></div>

If you're using Python, instead of getting raspicam, you'll need to get picamera. This can be done with the pip package manager:

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%">1</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #888888">pip install picamera</span>
</pre></td></tr></table></div>

Finally, install PiPCA9685 to get C++ and Python APIs for controlling our motors:

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%">1
2
3
4
5
6
7</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #888888">git clone https://github.com/barulicm/PiPCA9685.git</span>
<span style="color: #888888">cd PiPCA9685</span>
<span style="color: #888888">mkdir build &amp;&amp; cd build</span>
<span style="color: #888888">cmake ..</span>
<span style="color: #888888">cmake --build . </span>
<span style="color: #888888">sudo cmake --build . --target install</span>
<span style="color: #888888">sudo cmake --build . --target python_install</span>
</pre></td></tr></table></div>

![](../../images/projects/racecar_20.gif)

### Writing the Software

Alright! It's time to start writing some self-driving car code! Both the C++ and Python exmaples below implement the same approach, so I'll explain it once for the C++ example.

You can download both the C++ and Python code [here](../../projectfiles/racecar_code.zip).

The overall structure of this code is a single loop which grabs the latest image from the camera, uses color matching to identify orange cones, and steers toward the area it can see with the least amount of cones.

#### C++

For context, here's the entire main.cpp C++ example file.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%"> 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #557799">#include &lt;iostream&gt;</span>
<span style="color: #557799">#include &lt;opencv2/opencv.hpp&gt;</span>
<span style="color: #557799">#include &lt;raspicam/raspicam_cv.h&gt;</span>
<span style="color: #557799">#include &lt;PiPCA9685/PCA9685.h&gt;</span>
<span style="color: #557799">#include &lt;unistd.h&gt;</span>

<span style="color: #333399; font-weight: bold">int</span> <span style="color: #0066BB; font-weight: bold">main</span>() {
	<span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> WIDTH <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">320</span>;
	<span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> HEIGHT <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">240</span>;
	<span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> SPEED <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">1</span>;
	<span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> STEER <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">0</span>;
	
	raspicam<span style="color: #333333">::</span>RaspiCam_Cv Camera;
	Camera.set(CV_CAP_PROP_FRAME_WIDTH, WIDTH);
	Camera.set(CV_CAP_PROP_FRAME_HEIGHT, HEIGHT);
	<span style="color: #008800; font-weight: bold">if</span>(<span style="color: #333333">!</span>Camera.open()) {
		std<span style="color: #333333">::</span>cerr <span style="color: #333333">&lt;&lt;</span> <span style="background-color: #fff0f0">&quot;Couldn&#39;t open camera.&quot;</span> <span style="color: #333333">&lt;&lt;</span> std<span style="color: #333333">::</span>endl;
		<span style="color: #008800; font-weight: bold">return</span> <span style="color: #0000DD; font-weight: bold">1</span>;
	}
	
	PCA9685 pca;
	pca.set_pwm_freq(<span style="color: #6600EE; font-weight: bold">60.0</span>);
	
	cv<span style="color: #333333">::</span>namedWindow(<span style="background-color: #fff0f0">&quot;Preview&quot;</span>, CV_WINDOW_NORMAL);
	
	cv<span style="color: #333333">::</span>Mat frame;
	cv<span style="color: #333333">::</span>Mat frame_HSV;
	cv<span style="color: #333333">::</span>Mat filtered;
	
	cv<span style="color: #333333">::</span>Scalar minVal(<span style="color: #0000DD; font-weight: bold">0</span>, <span style="color: #0000DD; font-weight: bold">100</span>, <span style="color: #0000DD; font-weight: bold">100</span>);
	cv<span style="color: #333333">::</span>Scalar maxVal(<span style="color: #0000DD; font-weight: bold">20</span>, <span style="color: #0000DD; font-weight: bold">255</span>, <span style="color: #0000DD; font-weight: bold">255</span>);
	
	cv<span style="color: #333333">::</span>Rect left_roi(<span style="color: #0000DD; font-weight: bold">0</span>, <span style="color: #0000DD; font-weight: bold">0</span>, WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, HEIGHT);
	cv<span style="color: #333333">::</span>Rect center_roi(WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, <span style="color: #0000DD; font-weight: bold">0</span>, WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, HEIGHT);
	cv<span style="color: #333333">::</span>Rect right_roi(<span style="color: #0000DD; font-weight: bold">2</span><span style="color: #333333">*</span>WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, <span style="color: #0000DD; font-weight: bold">0</span>, WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, HEIGHT);
	
	std<span style="color: #333333">::</span>array<span style="color: #333333">&lt;</span>cv<span style="color: #333333">::</span>Rect, <span style="color: #0000DD; font-weight: bold">3</span><span style="color: #333333">&gt;</span> ROIs <span style="color: #333333">=</span> {left_roi, center_roi, right_roi};
	std<span style="color: #333333">::</span>array<span style="color: #333333">&lt;</span><span style="color: #333399; font-weight: bold">double</span>, <span style="color: #0000DD; font-weight: bold">3</span><span style="color: #333333">&gt;</span> PWMs <span style="color: #333333">=</span> {<span style="color: #6600EE; font-weight: bold">1.5</span>, <span style="color: #6600EE; font-weight: bold">1.7</span>, <span style="color: #6600EE; font-weight: bold">2.0</span>};
	std<span style="color: #333333">::</span>array<span style="color: #333333">&lt;</span><span style="color: #333399; font-weight: bold">int</span>, <span style="color: #0000DD; font-weight: bold">3</span><span style="color: #333333">&gt;</span> nonZeroCounts <span style="color: #333333">=</span> {<span style="color: #0000DD; font-weight: bold">0</span>,<span style="color: #0000DD; font-weight: bold">0</span>,<span style="color: #0000DD; font-weight: bold">0</span>};
	
	pca.set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.7</span>);
	usleep(<span style="color: #0000DD; font-weight: bold">1&#39;000&#39;000</span>);
	pca.set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.76</span>);
	
	<span style="color: #008800; font-weight: bold">while</span>(Camera.grab() <span style="color: #333333">&amp;&amp;</span> cv<span style="color: #333333">::</span>waitKey(<span style="color: #0000DD; font-weight: bold">30</span>) <span style="color: #333333">!=</span> <span style="color: #0044DD">&#39; &#39;</span>) {
		Camera.retrieve(frame);
		
		cv<span style="color: #333333">::</span>cvtColor(frame, frame_HSV, cv<span style="color: #333333">::</span>COLOR_BGR2HSV);
		
		cv<span style="color: #333333">::</span>inRange(frame_HSV, minVal, maxVal, filtered);
		
		<span style="color: #008800; font-weight: bold">auto</span> countFunc <span style="color: #333333">=</span> [<span style="color: #333333">&amp;</span>filtered](cv<span style="color: #333333">::</span>Rect <span style="color: #333333">&amp;</span>roi) {
			<span style="color: #008800; font-weight: bold">return</span> cv<span style="color: #333333">::</span>countNonZero(filtered(roi));
		};
		
		std<span style="color: #333333">::</span>transform(ROIs.begin(), ROIs.end(), nonZeroCounts.begin(), countFunc);
		
		<span style="color: #008800; font-weight: bold">auto</span> minIter <span style="color: #333333">=</span> std<span style="color: #333333">::</span>min_element(nonZeroCounts.begin(), nonZeroCounts.end());
		<span style="color: #008800; font-weight: bold">auto</span> idx <span style="color: #333333">=</span> std<span style="color: #333333">::</span>distance(nonZeroCounts.begin(), minIter);
		
		cv<span style="color: #333333">::</span>rectangle(frame, ROIs[idx], cv<span style="color: #333333">::</span>Scalar(<span style="color: #0000DD; font-weight: bold">0</span>,<span style="color: #0000DD; font-weight: bold">255</span>,<span style="color: #0000DD; font-weight: bold">0</span>), <span style="color: #0000DD; font-weight: bold">3</span>);
		
		pca.set_pwm_ms(STEER, PWMs[idx]);							
		
		cv<span style="color: #333333">::</span>imshow(<span style="background-color: #fff0f0">&quot;Preview&quot;</span>, frame);
	}
	
	pca.set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.7</span>);
	pca.set_pwm_ms(STEER, <span style="color: #6600EE; font-weight: bold">1.7</span>);

	<span style="color: #008800; font-weight: bold">return</span> <span style="color: #0000DD; font-weight: bold">0</span>;
}
</pre></td></tr></table></div>

And now for a quick section-by-section break down of what this code is doing.
            
<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%"><span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> WIDTH <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">320</span>;
<span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> HEIGHT <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">240</span>;
<span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> SPEED <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">1</span>;
<span style="color: #008800; font-weight: bold">const</span> <span style="color: #008800; font-weight: bold">auto</span> STEER <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">0</span>;
	
raspicam<span style="color: #333333">::</span>RaspiCam_Cv Camera;
Camera.set(CV_CAP_PROP_FRAME_WIDTH, WIDTH);
Camera.set(CV_CAP_PROP_FRAME_HEIGHT, HEIGHT);
<span style="color: #008800; font-weight: bold">if</span>(<span style="color: #333333">!</span>Camera.open()) {
	std<span style="color: #333333">::</span>cerr <span style="color: #333333">&lt;&lt;</span> <span style="background-color: #fff0f0">&quot;Couldn&#39;t open camera.&quot;</span> <span style="color: #333333">&lt;&lt;</span> std<span style="color: #333333">::</span>endl;
	<span style="color: #008800; font-weight: bold">return</span> <span style="color: #0000DD; font-weight: bold">1</span>;
}
	
PCA9685 pca;
pca.set_pwm_freq(<span style="color: #6600EE; font-weight: bold">60.0</span>);
	
cv<span style="color: #333333">::</span>namedWindow(<span style="background-color: #fff0f0">&quot;Preview&quot;</span>, CV_WINDOW_NORMAL);
	
cv<span style="color: #333333">::</span>Mat frame;
cv<span style="color: #333333">::</span>Mat frame_HSV;
cv<span style="color: #333333">::</span>Mat filtered;
	
cv<span style="color: #333333">::</span>Scalar minVal(<span style="color: #0000DD; font-weight: bold">0</span>, <span style="color: #0000DD; font-weight: bold">100</span>, <span style="color: #0000DD; font-weight: bold">100</span>);
cv<span style="color: #333333">::</span>Scalar maxVal(<span style="color: #0000DD; font-weight: bold">20</span>, <span style="color: #0000DD; font-weight: bold">255</span>, <span style="color: #0000DD; font-weight: bold">255</span>);
	
cv<span style="color: #333333">::</span>Rect left_roi(<span style="color: #0000DD; font-weight: bold">0</span>, <span style="color: #0000DD; font-weight: bold">0</span>, WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, HEIGHT);
cv<span style="color: #333333">::</span>Rect center_roi(WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, <span style="color: #0000DD; font-weight: bold">0</span>, WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, HEIGHT);
cv<span style="color: #333333">::</span>Rect right_roi(<span style="color: #0000DD; font-weight: bold">2</span><span style="color: #333333">*</span>WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, <span style="color: #0000DD; font-weight: bold">0</span>, WIDTH<span style="color: #333333">/</span><span style="color: #0000DD; font-weight: bold">3</span>, HEIGHT);
	
std<span style="color: #333333">::</span>array<span style="color: #333333">&lt;</span>cv<span style="color: #333333">::</span>Rect, <span style="color: #0000DD; font-weight: bold">3</span><span style="color: #333333">&gt;</span> ROIs <span style="color: #333333">=</span> {left_roi, center_roi, right_roi};
std<span style="color: #333333">::</span>array<span style="color: #333333">&lt;</span><span style="color: #333399; font-weight: bold">double</span>, <span style="color: #0000DD; font-weight: bold">3</span><span style="color: #333333">&gt;</span> PWMs <span style="color: #333333">=</span> {<span style="color: #6600EE; font-weight: bold">1.5</span>, <span style="color: #6600EE; font-weight: bold">1.7</span>, <span style="color: #6600EE; font-weight: bold">2.0</span>};
std<span style="color: #333333">::</span>array<span style="color: #333333">&lt;</span><span style="color: #333399; font-weight: bold">int</span>, <span style="color: #0000DD; font-weight: bold">3</span><span style="color: #333333">&gt;</span> nonZeroCounts <span style="color: #333333">=</span> {<span style="color: #0000DD; font-weight: bold">0</span>,<span style="color: #0000DD; font-weight: bold">0</span>,<span style="color: #0000DD; font-weight: bold">0</span>};
</pre></div>

This first section simply defines some useful constants, initializes our camera and PCA9685 (the PWM hat), and declares the arrays we'll be working with.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">pca.set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.7</span>);
usleep(<span style="color: #0000DD; font-weight: bold">1&#39;000&#39;000</span>);
pca.set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.76</span>);
</pre></div>

This starts the car driving forward at a constant speed. PWM is a digital signal that communicates a "duty cycle". We can treat this like a percentage control over the car's throttle. On the wire, PWM is a square wave where the on time varies from 1.0ms to 2.0ms. The Turnigy truck maps this range as follows.

1.5 -> Full Reverse.

1.7 -> Stopped

2.0 -> Full Forward

As you can see in the code, I'm only setting the forward speed to 1.76, or just 20% of the cars max speed. This car can go fast!

The 1 second delay gives the car's ESC time to initialize. It waits for a certain length of neutral (1.7ms) signal before it enables the vehicle.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%"><span style="color: #008800; font-weight: bold">while</span>(Camera.grab() <span style="color: #333333">&amp;&amp;</span> cv<span style="color: #333333">::</span>waitKey(<span style="color: #0000DD; font-weight: bold">30</span>) <span style="color: #333333">!=</span> <span style="color: #0044DD">&#39; &#39;</span>) {
	Camera.retrieve(frame);						
	<span style="color: #888888">// ...</span>
	cv<span style="color: #333333">::</span>imshow(<span style="background-color: #fff0f0">&quot;Preview&quot;</span>, frame);
}
</pre></div>

Here is our main loop, without all of the image processing parts for now. Every iteration, we grab a new image from the camera, check if the user has pressed the spacebar to stop the app, do some processing on the image, and then show it in the preview window.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">cv<span style="color: #333333">::</span>cvtColor(frame, frame_HSV, cv<span style="color: #333333">::</span>COLOR_BGR2HSV);
</pre></div>

This line converts the image from the BGR colorspace to the HSV colorspace. HSV will make it easier for us to do simple thresholding to highlight all of the orange cones in the track.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">cv<span style="color: #333333">::</span>inRange(frame_HSV, minVal, maxVal, filtered);
</pre></div>

And this line does the actual thresholding by filling the "filtered" image with 255s where the HSV color was in the range given, and 0 where the color was out of that range.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%"><span style="color: #008800; font-weight: bold">auto</span> countFunc <span style="color: #333333">=</span> [<span style="color: #333333">&amp;</span>filtered](cv<span style="color: #333333">::</span>Rect <span style="color: #333333">&amp;</span>roi) {
	<span style="color: #008800; font-weight: bold">return</span> cv<span style="color: #333333">::</span>countNonZero(filtered(roi));
};
		
std<span style="color: #333333">::</span>transform(ROIs.begin(), ROIs.end(), nonZeroCounts.begin(), countFunc);
</pre></div>

Here, we define a lambda function which calls countNonZero on the given region of interest (ROI) of the filtered image. Then we use transform to fill the nonZeroCounts array with the number of non-zero pixels in each ROI.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%"><span style="color: #008800; font-weight: bold">auto</span> minIter <span style="color: #333333">=</span> std<span style="color: #333333">::</span>min_element(nonZeroCounts.begin(), nonZeroCounts.end());
<span style="color: #008800; font-weight: bold">auto</span> idx <span style="color: #333333">=</span> std<span style="color: #333333">::</span>distance(nonZeroCounts.begin(), minIter);
		
cv<span style="color: #333333">::</span>rectangle(frame, ROIs[idx], cv<span style="color: #333333">::</span>Scalar(<span style="color: #0000DD; font-weight: bold">0</span>,<span style="color: #0000DD; font-weight: bold">255</span>,<span style="color: #0000DD; font-weight: bold">0</span>), <span style="color: #0000DD; font-weight: bold">3</span>);
		
pca.set_pwm_ms(STEER, PWMs[idx]);
</pre></div>

Now that we have the counts for each ROI, we can get the index of the smallest count using min_element and distance. The next two lines draw the rectangle back on the original image for previewing and set the steering PWM to the value corresponding to the direction we want to turn towards.

Similarly to the drive motor, here our PWM values map as follows:

1.5 -> Full left

1.7 -> Straight ahead

2.0 -> Full right

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><pre style="margin: 0; line-height: 125%">pca.set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.7</span>);
pca.set_pwm_ms(STEER, <span style="color: #6600EE; font-weight: bold">1.7</span>);
</pre></div>

Finally, after the loop, we reset the PWM hat to stop the drive motor and set the steering servo to the straight ahead, neutral position. This just makes sure our car doesn't keep running away from us when we stop the app.

And that's all the code it takes to get this car driving around the demo track in the video above.

#### Python

And here's the entire Python version of the code.

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%"> 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #008800; font-weight: bold">from</span> <span style="color: #0e84b5; font-weight: bold">picamera</span> <span style="color: #008800; font-weight: bold">import</span> PiCamera
<span style="color: #008800; font-weight: bold">from</span> <span style="color: #0e84b5; font-weight: bold">picamera.array</span> <span style="color: #008800; font-weight: bold">import</span> PiRGBArray
<span style="color: #008800; font-weight: bold">import</span> <span style="color: #0e84b5; font-weight: bold">time</span>
<span style="color: #008800; font-weight: bold">import</span> <span style="color: #0e84b5; font-weight: bold">cv2</span>
<span style="color: #008800; font-weight: bold">from</span> <span style="color: #0e84b5; font-weight: bold">PiPCA9685.PiPCA9685</span> <span style="color: #008800; font-weight: bold">import</span> PCA9685
<span style="color: #008800; font-weight: bold">import</span> <span style="color: #0e84b5; font-weight: bold">numpy</span> <span style="color: #008800; font-weight: bold">as</span> <span style="color: #0e84b5; font-weight: bold">np</span>

<span style="color: #008800; font-weight: bold">def</span> <span style="color: #0066BB; font-weight: bold">main</span>():
    WIDTH <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">320</span>
    HEIGHT <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">240</span>
    STEER <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">0</span>
    SPEED <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">1</span>
    
    camera <span style="color: #333333">=</span> PiCamera()
    camera<span style="color: #333333">.</span>resolution <span style="color: #333333">=</span> (WIDTH,HEIGHT)
    rawCapture <span style="color: #333333">=</span> PiRGBArray(camera, size<span style="color: #333333">=</span>(WIDTH,HEIGHT))

    <span style="color: #888888"># camera warmp up time</span>
    time<span style="color: #333333">.</span>sleep(<span style="color: #6600EE; font-weight: bold">0.1</span>)

    pca <span style="color: #333333">=</span> PCA9685()
    pca<span style="color: #333333">.</span>set_pwm_freq(<span style="color: #0000DD; font-weight: bold">60</span>)

    cv2<span style="color: #333333">.</span>namedWindow(<span style="background-color: #fff0f0">&#39;Preview&#39;</span>, cv2<span style="color: #333333">.</span>WINDOW_NORMAL)

    minVal <span style="color: #333333">=</span> (<span style="color: #0000DD; font-weight: bold">0</span>, <span style="color: #0000DD; font-weight: bold">100</span>, <span style="color: #0000DD; font-weight: bold">100</span>)
    maxVal <span style="color: #333333">=</span> (<span style="color: #0000DD; font-weight: bold">20</span>, <span style="color: #0000DD; font-weight: bold">255</span>, <span style="color: #0000DD; font-weight: bold">255</span>)

    ROIs <span style="color: #333333">=</span> [ [(<span style="color: #0000DD; font-weight: bold">0</span>,WIDTH<span style="color: #333333">//</span><span style="color: #0000DD; font-weight: bold">3</span>), (<span style="color: #0000DD; font-weight: bold">0</span>,HEIGHT)],
             [(WIDTH<span style="color: #333333">//</span><span style="color: #0000DD; font-weight: bold">3</span>, <span style="color: #0000DD; font-weight: bold">2</span><span style="color: #333333">*</span>WIDTH<span style="color: #333333">//</span><span style="color: #0000DD; font-weight: bold">3</span>), (<span style="color: #0000DD; font-weight: bold">0</span>, HEIGHT)],
             [(<span style="color: #0000DD; font-weight: bold">2</span><span style="color: #333333">*</span>WIDTH<span style="color: #333333">//</span><span style="color: #0000DD; font-weight: bold">3</span>, WIDTH), (<span style="color: #0000DD; font-weight: bold">0</span>, HEIGHT)] ]

    PWMs <span style="color: #333333">=</span> [<span style="color: #6600EE; font-weight: bold">1.5</span>, <span style="color: #6600EE; font-weight: bold">1.7</span>, <span style="color: #6600EE; font-weight: bold">2.0</span>]

    pca<span style="color: #333333">.</span>set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.7</span>)
    time<span style="color: #333333">.</span>sleep(<span style="color: #0000DD; font-weight: bold">1</span>)
    pca<span style="color: #333333">.</span>set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.76</span>)

    <span style="color: #008800; font-weight: bold">for</span> capture <span style="color: #000000; font-weight: bold">in</span> camera<span style="color: #333333">.</span>capture_continuous(rawCapture, <span style="color: #007020">format</span><span style="color: #333333">=</span><span style="background-color: #fff0f0">&#39;bgr&#39;</span>, use_video_port<span style="color: #333333">=</span><span style="color: #008800; font-weight: bold">True</span>):
        frame <span style="color: #333333">=</span> capture<span style="color: #333333">.</span>array

        frame_HSV <span style="color: #333333">=</span> cv2<span style="color: #333333">.</span>cvtColor(frame, cv2<span style="color: #333333">.</span>COLOR_BGR2HSV)

        filtered <span style="color: #333333">=</span> cv2<span style="color: #333333">.</span>inRange(frame_HSV, minVal, maxVal)

        minCount <span style="color: #333333">=</span> <span style="color: #007020">float</span>(<span style="background-color: #fff0f0">&#39;inf&#39;</span>)
        minIdx <span style="color: #333333">=</span> <span style="color: #0000DD; font-weight: bold">0</span>
        
        <span style="color: #008800; font-weight: bold">for</span> i <span style="color: #000000; font-weight: bold">in</span> <span style="color: #007020">range</span>(<span style="color: #007020">len</span>(ROIs)):
            ROI <span style="color: #333333">=</span> ROIs[i]
            count <span style="color: #333333">=</span> cv2<span style="color: #333333">.</span>countNonZero(filtered[ROI[<span style="color: #0000DD; font-weight: bold">1</span>][<span style="color: #0000DD; font-weight: bold">0</span>]:ROI[<span style="color: #0000DD; font-weight: bold">1</span>][<span style="color: #0000DD; font-weight: bold">1</span>],ROI[<span style="color: #0000DD; font-weight: bold">0</span>][<span style="color: #0000DD; font-weight: bold">0</span>]:ROI[<span style="color: #0000DD; font-weight: bold">0</span>][<span style="color: #0000DD; font-weight: bold">1</span>]])
            <span style="color: #008800; font-weight: bold">if</span> count <span style="color: #333333">&lt;</span> minCount:
                minCount <span style="color: #333333">=</span> count
                minIdx <span style="color: #333333">=</span> i

        minROI <span style="color: #333333">=</span> ROIs[minIdx]

        cv2<span style="color: #333333">.</span>rectangle(frame, (minROI[<span style="color: #0000DD; font-weight: bold">0</span>][<span style="color: #0000DD; font-weight: bold">0</span>],minROI[<span style="color: #0000DD; font-weight: bold">1</span>][<span style="color: #0000DD; font-weight: bold">0</span>]), (minROI[<span style="color: #0000DD; font-weight: bold">0</span>][<span style="color: #0000DD; font-weight: bold">1</span>],minROI[<span style="color: #0000DD; font-weight: bold">1</span>][<span style="color: #0000DD; font-weight: bold">1</span>]), color<span style="color: #333333">=</span>(<span style="color: #0000DD; font-weight: bold">0</span>, <span style="color: #0000DD; font-weight: bold">255</span>, <span style="color: #0000DD; font-weight: bold">0</span>), thickness<span style="color: #333333">=</span><span style="color: #0000DD; font-weight: bold">3</span>)

        pca<span style="color: #333333">.</span>set_pwm_ms(STEER, PWMs[minIdx])

        cv2<span style="color: #333333">.</span>imshow(<span style="background-color: #fff0f0">&#39;Preview&#39;</span>, frame)

        rawCapture<span style="color: #333333">.</span>truncate(<span style="color: #0000DD; font-weight: bold">0</span>)

        <span style="color: #008800; font-weight: bold">if</span> cv2<span style="color: #333333">.</span>waitKey(<span style="color: #0000DD; font-weight: bold">10</span>) <span style="color: #333333">==</span> <span style="color: #007020">ord</span>(<span style="background-color: #fff0f0">&#39; &#39;</span>):
            <span style="color: #008800; font-weight: bold">break</span>

    pca<span style="color: #333333">.</span>set_pwm_ms(SPEED, <span style="color: #6600EE; font-weight: bold">1.7</span>)
    pca<span style="color: #333333">.</span>set_pwm_ms(STEER, <span style="color: #6600EE; font-weight: bold">1.7</span>)
    cv2<span style="color: #333333">.</span>destroyAllWindows()

<span style="color: #008800; font-weight: bold">if</span> __name__ <span style="color: #333333">==</span> <span style="background-color: #fff0f0">&#39;__main__&#39;</span>:
    main()
</pre></td></tr></table></div>
