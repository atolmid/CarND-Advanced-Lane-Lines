##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/calibrate.png "Images with Corners"
[image2]: ./output_images/undistort_output.png "Undistorted"
[image3]: ./output_images/test1.png "Road Transformed"
[image4]: ./output_images/gradx.png "Threshold Grad. X"
[image5]: ./output_images/grady.png "Threshold Grad. Y"
[image6]: ./output_images/mag.png "Threshold Magnitude"
[image7]: ./output_images/direction.png "Threshold Direction"
[image8]: ./output_images/grads.png "Threshold Combined Grads"
[image9]: ./output_images/hls.png "Threshold HLS"
[image10]: ./output_images/combined_thresh.png "All Thresholds Applied"
[image11]: ./output_images/pipeline.png "Pipeline Output"
[image12]: ./output_images/undistorted_warped.png "Perspective transform"
[image13]: ./output_images/warped_straight_lines.png "Perspective transform - Straight Lines"
[image14]: ./output_images/lines_on_thresh.png "perspective transform 3"
[image15]: ./output_images/draw_lanes.png "Draw lanes"
[image16]: ./output_images/lane_drawn_challenge.png "Draw lanes Challenge"
[image17]: ./output_images/lane_drawn_test.png "Draw lanes Test"
[image18]: ./output_images/thresholds_shade.png "Thresholds Shade"
[image19]: ./output_images/warped_shade.png "Warped Shade"
[image20]: ./output_images/pipeline_shade.png "Pipeline Output with Shade"
[video1]: ./project_video.mp4 "Video"
[video2]: ./output_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. 
Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  
Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  
`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  

There are two camera calibration functions. The difference between them is that calibrate\_camera returns objpoints and imgpoints while drawing the input images, while calibrate\_camera1 takes only one image as input, and returns in addition the modified image, without drawing anything.
I initially used the first function, and drew the images with corners.
An example can be seen in the image below:

![alt text][image1]

Then, I applied distortion correction to the test images.
I created a function called 'undist' that uses `cv2.undistort()` function and outputs the corrected image, as well as mtx and  dist, that can later be used by other functions.
The output for one of the test images can be seen in the following figure:

![alt text][image2]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
the process for applying the distortion correction to the test images was described in the previous section.
A second example of the result can be seen in the following image:
![alt text][image3]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (cells 6-p in the iPython notebook).  
In cell No 10, I created a function where you can select an area of interest in an image using a parameter.
After the image is converted to gray, the image where pixels in this area are not zero is returned
Then, in cell No 11, I combine this area mask with the threshold masks.
Then I define a pipeline, that produces the result of applying the masks mentioned above.

Here's an example of my output for this step.  (note: this is not actually from one of the test images)
* Threshold Grad. X:
![alt text][image4]

* Threshold Grad. Y:
![alt text][image5]

* Threshold Magnitude:
![alt text][image6]

* Threshold Direction:
![alt text][image7]

* Threshold Combined Grads:
![alt text][image8]

* Threshold HLS:
![alt text][image9]

* All Thresholds Applied:
![alt text][image10]

* Pipeline Output:
![alt text][image11]

* Things are of ourse a biyt more complicated, when there is a lot of shadow in the image:
![alt text][image20]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `corners_unwarp(img, corners, nx, ny, mtx, dist)`.
I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[550, 460],[740, 460], [1280, 700],[0, 700]])
dst = np.float32([[0, 0], [1280, 0], [1280, 720], [0, 720]])
```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550, 460      | 0, 0          | 
| 740, 460      | 1280, 0       |
| 1280, 700     | 1280, 720     |
| 0, 700        | 0, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

I initially verified that the images are transformed.
An example can be seen in the following images:
* One of the test images:
![alt text][image12]

* One of the straight lines images
![alt text][image13]

* And one using the images that are harder to detect the lines in, due to the existence of shadow
![alt text][image19]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I created a function called 'get\_line\_positions', that gets as input a binary warped image, the number of sliding windows, and minimum pixels and the margins,
and returns the x and y coordinates of the left and right line.
It uses the method that creates a histogram of the input image, and finds the peak of the left and right half.
Then it goes through the sliding windows, and if the pixels assumed to belong to the line are more than the minimum, it centers the next window on their mean position.
A method called 'get\_line\_fit' takes the points and fits them to a polynomial.
At the end there is a method called 'draw\_lane' that takes as input an image, and draws the lines on it.
An example of an image which first underwent perspective transform and was thresholded, and finally the lanes were drawn on the processed image, can be seen below:

![alt text][image14]

In addition, a function called 'project\_lines' was created, that takes the undistorted and the warped image, and outputs the image with the lines drawn.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 34 through 36 of the iPython notebook, using code provided in the lectures.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cells 37 through 43 in my iPython notebook.
I initially created a pipeline function called 'draw\_lanes', that takes the path of an image, and detects the lane lines, and at the end draws the original image, the output of the previously created function 'pipeline', and the original image, with the lane drawn.
  
Here is an example of my result on a test image:

![alt text][image15]

Then, as suggested in the lectures, I created a class called 'Line', and a function to update its variables.
Additionally, a function 'check\_lane', that performs sanity checks (curvature, horisontal distance and slope) on the lines detected, to see if the values make sense.
Then, a function to skip sliding windows, as suggested in the lectures, and finally, a pipeline method, called 'draw\_lanes\_with\_skip'.
This function takes as input images, and after performing sanity checks on the detected lines, it decides whether the sliding window should be skipped, or there will be a line points detection from scratch.
It then computes in addition the curvature and the distance from the center of the lane, which are then embedded on the top left corner of the image, and the image is returned.

this function was used to process video.

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In general it took some time to make everything work.
Finding working source and destination points was also something that took some time, and required trial and error.
Several other of the parameters are defined for these specific images, and will probably not work well on much different ones.
Thus, one of the things that need to be done, is consider how this could be changed.
The lanes in the video are mostly correctly detected and drawn, however in the challenge videos the results were not equally good, which means I have to revisit the process at a latter ti,e in order to make improvements.

The main problem was in the case of shadow, and light reflection.
In the shaded area for example, the HSV threshold included almost all shaded region, which extremely distorted the final outcome.

An example can be seen in the next image:

![alt text][image18]

A way of dealing with it was first to include a threshold using red from RGB, and also limiting the H in HSV to (170, 190)
This improved things, however there is still plenty of room for improvement:

![alt text][image16]
![alt text][image17]