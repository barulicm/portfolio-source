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

{{< highlight bash "linenos=yes" >}}
sudo apt update && sudo apt upgrade
sudo apt install build-essential cmake pkg-config libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libgtk2.0-dev libgtk-3-dev libatlas-base-dev gfortran python2.7-dev python3-dev
cd ~
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.0.1.zip
unzip opencv && cd opencv
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
cmake --build . -- -j4
sudo cmake --build . --target install
sudo ldconfig
{{< / highlight >}}

If you're working in C++, you'll need raspicam to interact with the camera. You can install it with these commands:

{{< highlight bash "linenos=yes" >}}
# Download raspicam-0.1.6.zip from https://sourceforge.net/projects/raspicam/files/
unzip raspicam-0.1.6.zip && cd raspicam-0.1.6
mkdir build && cd build
cmake ..
cmake –build .
sudo cmake –build . –target install
{{< / highlight >}}

Raspicam doesn't seem to install itself totally correctly, so if CMake complains that it can't find the raspicam_cv project when you're building your project, try pointing CMake at the raspicam build directory like this:

{{< highlight bash >}}
cmake -Draspicam_DIR=/home/pi/raspicam-0.1.6/build/ ..
{{< / highlight >}}

If you're using Python, instead of getting raspicam, you'll need to get picamera. This can be done with the pip package manager:

{{< highlight bash >}}
pip install picamera
{{< / highlight >}}

Finally, install PiPCA9685 to get C++ and Python APIs for controlling our motors:

{{< highlight bash "linenos=yes" >}}
git clone https://github.com/barulicm/PiPCA9685.git
cd PiPCA9685
mkdir build && cd build
cmake ..
cmake –build . 
sudo cmake –build . –target install
sudo cmake –build . –target python_install
{{< / highlight >}}

![](../../images/projects/racecar_20.gif)

### Writing the Software

Alright! It's time to start writing some self-driving car code! Both the C++ and Python exmaples below implement the same approach, so I'll explain it once for the C++ example.

You can download both the C++ and Python code [here](../../projectfiles/racecar_code.zip).

The overall structure of this code is a single loop which grabs the latest image from the camera, uses color matching to identify orange cones, and steers toward the area it can see with the least amount of cones.

#### C++

For context, here's the entire main.cpp C++ example file.

{{< highlight CPP "linenos=yes" >}}
#include <iostream>
#include <opencv2/opencv.hpp>
#include <raspicam/raspicam_cv.h>
#include <PiPCA9685/PCA9685.h>
#include <unistd.h>

int main() {
	const auto WIDTH = 320;
	const auto HEIGHT = 240;
	const auto SPEED = 1;
	const auto STEER = 0;
	
	raspicam::RaspiCam_Cv Camera;
	Camera.set(CV_CAP_PROP_FRAME_WIDTH, WIDTH);
	Camera.set(CV_CAP_PROP_FRAME_HEIGHT, HEIGHT);
	if(!Camera.open()) {
		std::cerr << "Couldn't open camera." << std::endl;
		return 1;
	}
	
	PCA9685 pca;
	pca.set_pwm_freq(60.0);
	
	cv::namedWindow("Preview", CV_WINDOW_NORMAL);
	
	cv::Mat frame;
	cv::Mat frame_HSV;
	cv::Mat filtered;
	
	cv::Scalar minVal(0, 100, 100);
	cv::Scalar maxVal(20, 255, 255);
	
	cv::Rect left_roi(0, 0, WIDTH/3, HEIGHT);
	cv::Rect center_roi(WIDTH/3, 0, WIDTH/3, HEIGHT);
	cv::Rect right_roi(2*WIDTH/3, 0, WIDTH/3, HEIGHT);
	
	std::array<cv::Rect, 3> ROIs = {left_roi, center_roi, right_roi};
	std::array<double, 3> PWMs = {1.5, 1.7, 2.0};
	std::array<int, 3> nonZeroCounts = {0,0,0};
	
	pca.set_pwm_ms(SPEED, 1.7);
	usleep(1'000'000);
	pca.set_pwm_ms(SPEED, 1.76);
	
	while(Camera.grab() && cv::waitKey(30) != ' ') {
		Camera.retrieve(frame);
		
		cv::cvtColor(frame, frame_HSV, cv::COLOR_BGR2HSV);
		
		cv::inRange(frame_HSV, minVal, maxVal, filtered);
		
		auto countFunc = [&filtered](cv::Rect &roi) {
			return cv::countNonZero(filtered(roi));
		};
		
		std::transform(ROIs.begin(), ROIs.end(), nonZeroCounts.begin(), countFunc);
		
		auto minIter = std::min_element(nonZeroCounts.begin(), nonZeroCounts.end());
		auto idx = std::distance(nonZeroCounts.begin(), minIter);
		
		cv::rectangle(frame, ROIs[idx], cv::Scalar(0,255,0), 3);
		
		pca.set_pwm_ms(STEER, PWMs[idx]);							
		
		cv::imshow("Preview", frame);
	}
	
	pca.set_pwm_ms(SPEED, 1.7);
	pca.set_pwm_ms(STEER, 1.7);

	return 0;
}
{{< / highlight >}}

And now for a quick section-by-section break down of what this code is doing.

{{< highlight CPP "linenos=yes" >}}
const auto WIDTH = 320;
const auto HEIGHT = 240;
const auto SPEED = 1;
const auto STEER = 0;
	
raspicam::RaspiCam_Cv Camera;
Camera.set(CV_CAP_PROP_FRAME_WIDTH, WIDTH);
Camera.set(CV_CAP_PROP_FRAME_HEIGHT, HEIGHT);
if(!Camera.open()) {
	std::cerr << "Couldn't open camera." << std::endl;
	return 1;
}
	
PCA9685 pca;
pca.set_pwm_freq(60.0);
	
cv::namedWindow("Preview", CV_WINDOW_NORMAL);
	
cv::Mat frame;
cv::Mat frame_HSV;
cv::Mat filtered;
	
cv::Scalar minVal(0, 100, 100);
cv::Scalar maxVal(20, 255, 255);
	
cv::Rect left_roi(0, 0, WIDTH/3, HEIGHT);
cv::Rect center_roi(WIDTH/3, 0, WIDTH/3, HEIGHT);
cv::Rect right_roi(2*WIDTH/3, 0, WIDTH/3, HEIGHT);
	
std::array<cv::Rect, 3> ROIs = {left_roi, center_roi, right_roi};
std::array<double, 3> PWMs = {1.5, 1.7, 2.0};
std::array<int, 3> nonZeroCounts = {0,0,0};
{{< / highlight >}}

This first section simply defines some useful constants, initializes our camera and PCA9685 (the PWM hat), and declares the arrays we'll be working with.

{{< highlight CPP "linenos=yes" >}}
pca.set_pwm_ms(SPEED, 1.7);
usleep(1'000'000);
pca.set_pwm_ms(SPEED, 1.76);
{{< / highlight >}}

This starts the car driving forward at a constant speed. PWM is a digital signal that communicates a "duty cycle". We can treat this like a percentage control over the car's throttle. On the wire, PWM is a square wave where the on time varies from 1.0ms to 2.0ms. The Turnigy truck maps this range as follows.

1.5 -> Full Reverse.

1.7 -> Stopped

2.0 -> Full Forward

As you can see in the code, I'm only setting the forward speed to 1.76, or just 20% of the cars max speed. This car can go fast!

The 1 second delay gives the car's ESC time to initialize. It waits for a certain length of neutral (1.7ms) signal before it enables the vehicle.

{{< highlight CPP "linenos=yes" >}}
while(Camera.grab() && cv::waitKey(30) != ' ') {
	Camera.retrieve(frame);						
	// ...
	cv::imshow("Preview", frame);
}
{{< / highlight >}}

Here is our main loop, without all of the image processing parts for now. Every iteration, we grab a new image from the camera, check if the user has pressed the spacebar to stop the app, do some processing on the image, and then show it in the preview window.

{{< highlight CPP >}}
cv::cvtColor(frame, frame_HSV, cv::COLOR_BGR2HSV);
{{< / highlight >}}

This line converts the image from the BGR colorspace to the HSV colorspace. HSV will make it easier for us to do simple thresholding to highlight all of the orange cones in the track.

{{< highlight CPP >}}
cv::inRange(frame_HSV, minVal, maxVal, filtered);
{{< / highlight >}}

And this line does the actual thresholding by filling the "filtered" image with 255s where the HSV color was in the range given, and 0 where the color was out of that range.

{{< highlight CPP "linenos=yes" >}}
auto countFunc = [&filtered](cv::Rect &roi) {
	return cv::countNonZero(filtered(roi));
};
		
std::transform(ROIs.begin(), ROIs.end(), nonZeroCounts.begin(), countFunc);
{{< / highlight >}}

Here, we define a lambda function which calls countNonZero on the given region of interest (ROI) of the filtered image. Then we use transform to fill the nonZeroCounts array with the number of non-zero pixels in each ROI.

{{< highlight CPP "linenos=yes" >}}
auto minIter = std::min_element(nonZeroCounts.begin(), nonZeroCounts.end());
auto idx = std::distance(nonZeroCounts.begin(), minIter);

cv::rectangle(frame, ROIs[idx], cv::Scalar(0,255,0), 3);

pca.set_pwm_ms(STEER, PWMs[idx]);
{{< / highlight >}}

Now that we have the counts for each ROI, we can get the index of the smallest count using min_element and distance. The next two lines draw the rectangle back on the original image for previewing and set the steering PWM to the value corresponding to the direction we want to turn towards.

Similarly to the drive motor, here our PWM values map as follows:

1.5 -> Full left

1.7 -> Straight ahead

2.0 -> Full right

{{< highlight CPP >}}
pca.set_pwm_ms(SPEED, 1.7);
pca.set_pwm_ms(STEER, 1.7);
{{< / highlight >}}

Finally, after the loop, we reset the PWM hat to stop the drive motor and set the steering servo to the straight ahead, neutral position. This just makes sure our car doesn't keep running away from us when we stop the app.

And that's all the code it takes to get this car driving around the demo track in the video above.

#### Python

And here's the entire Python version of the code.

{{< highlight Python "linenos=yes" >}}
from picamera import PiCamera
from picamera.array import PiRGBArray
import time
import cv2
from PiPCA9685.PiPCA9685 import PCA9685
import numpy as np

def main():
    WIDTH = 320
    HEIGHT = 240
    STEER = 0
    SPEED = 1
    
    camera = PiCamera()
    camera.resolution = (WIDTH,HEIGHT)
    rawCapture = PiRGBArray(camera, size=(WIDTH,HEIGHT))

    # camera warmp up time
    time.sleep(0.1)

    pca = PCA9685()
    pca.set_pwm_freq(60)

    cv2.namedWindow('Preview', cv2.WINDOW_NORMAL)

    minVal = (0, 100, 100)
    maxVal = (20, 255, 255)

    ROIs = [ [(0,WIDTH//3), (0,HEIGHT)],
             [(WIDTH//3, 2*WIDTH//3), (0, HEIGHT)],
             [(2*WIDTH//3, WIDTH), (0, HEIGHT)] ]

    PWMs = [1.5, 1.7, 2.0]

    pca.set_pwm_ms(SPEED, 1.7)
    time.sleep(1)
    pca.set_pwm_ms(SPEED, 1.76)

    for capture in camera.capture_continuous(rawCapture, format='bgr', use_video_port=True):
        frame = capture.array

        frame_HSV = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        filtered = cv2.inRange(frame_HSV, minVal, maxVal)

        minCount = float('inf')
        minIdx = 0
        
        for i in range(len(ROIs)):
            ROI = ROIs[i]
            count = cv2.countNonZero(filtered[ROI[1][0]:ROI[1][1],ROI[0][0]:ROI[0][1]])
            if count < minCount:
                minCount = count
                minIdx = i

        minROI = ROIs[minIdx]

        cv2.rectangle(frame, (minROI[0][0],minROI[1][0]), (minROI[0][1],minROI[1][1]), color=(0, 255, 0), thickness=3)

        pca.set_pwm_ms(STEER, PWMs[minIdx])

        cv2.imshow('Preview', frame)

        rawCapture.truncate(0)

        if cv2.waitKey(10) == ord(' '):
            break

    pca.set_pwm_ms(SPEED, 1.7)
    pca.set_pwm_ms(STEER, 1.7)
    cv2.destroyAllWindows()

if __name__ == '__main__':
    main()
{{< / highlight >}}
