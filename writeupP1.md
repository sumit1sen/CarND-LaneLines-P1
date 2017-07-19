# **Finding Lane Lines on the Road** 

## Udacity Self-Driving Car Nanodegree Project 1

### Project Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Exercise this pipeline on video
* Attempt to handle more challenging scenarios in the final video 
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/solidWhiteCurve.jpg "White curve"
[image2]: ./test_images_output/solidWhiteRight.jpg "White right"
[image3]: ./test_images_output/solidYellowCurve.jpg "Yellow curve"
[image4]: ./test_images_output/solidYellowCurve2.jpg "Yellow curve 2"
[image5]: ./test_images_output/whiteCarLaneSwitch.jpg "White lane switch"

[step1]: ./gray1.jpg "Grayscale"
[step2]: ./grayblur2.jpg "Grayscale blur"
[step3]: ./edge3.jpg "Canny edges"
[step4]: ./crop4.jpg "Crop 1"
[step5]: ./crop5.jpg "Crop 2"
[step6]: ./hough6.jpg "Hough lines"
[step7]: ./houghoverlay7.jpg "Fitted lines"
[step8]: ./final8.jpg "Final image"

<!-- [image1]: ./examples/grayscale.jpg "Grayscale" -->
[hist]: ./test_images_output/whiteCarLaneSwitch.jpghist.png "Hist"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the following steps.

* Grayscale conversion
* Gaussian blurring with 5x5 kernel
* Canny edge finding
* Hough transform to find line segments in a region of interest
* Sorting lines into 2 sets for left and right lane candidates
* Fitting each set of points to a straight line

For the multi-frame case (videos), the line fits were low-pass filtered by adding a fraction of fit coeffients from previous frames to dampen the response to rapidly changing frames and to image artifacts. 

In order to draw a single line on the left and right lanes, I replaced the draw_lines() function with a set of functions that separated the lanes, fitted them, removed outliers and overlaid them on the original image

Below you can see the final lane overlays on a series of 5 test images

![alt text][image1]
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]

The pipeline is illustrated with the following images of intermediate output

Grayscale
![alt text][step1]
Grayscale with blur
![alt text][step2]
Canny edge detection
![alt text][step3]
Cropping
![alt text][step5]
Hough transform
![alt text][step6]
Line Fit using Hough segments
![alt text][step7]
Final image
![alt text][step8]


### 2. Identify potential shortcomings with your current pipeline

Initially the pipeline performed well on the test images but did not handle some of the video frames.  My key insight was that in the real world, the lanes are not changing direction rapidly.  I first improved the selection of points going into the line fit by rejecting Hough lines with slopes that were too large or too small.  This histogram is from a typical image:

![alt text][hist]


This improved the video lane tracking but the challenge video still failed in 2 scenarios: when the road surface changed from black to gray; and when the road started to curve more sharply.

I decided that a simple fix for this would be too dampen the response of the pipeline to rapid changes in the fitted lane's slope and intercept.  I first cached the previous fit and added a fraction of this to the fit for the current frame.  This handled the curvature in the road but did not fully handle the change in road surface.

After some thought, I realized that some frames were giving erroneously large slope changes.  I then added a limit to the amount of slope change between frames to remove their effect.  This handled the changing road surface in the challenge video.

The most major shortcoming of the pipeline is that the fitting assumes that the lanes are straight.  Therefore the fit is good close to the car but gets progressively worse away from the car.

I also doubt that the pipeline will handle long stretches of light-colored road surface.  The changes I made to handle this assume only short sections of low-contrast surfaces. 

I am also concerned that I may have overtuned my pipeline to the limited set of videos (of I-280 near Stanford).  I doubt my model would work well on wiggly or bumpy roads, or under bad illumination. 

### 3. Suggest possible improvements to your pipeline
Higher order fits might be a good way to handle curves.  However, they can be unstable without further constraints.
I also think that a weighted fit that gives much higher weight to the points nearest the car would improve fitting.

Handling different road surfaces requires doing some work with color images.  I tried RGB normalization but this didn't work as expected, leading to highly saturated images with few edges.  Maybe using other color spaces would help in edge detection.

Also somehow removing parallax from the images would simplify the problem to one of finding parallel lines.  I don't know how to do this at present.