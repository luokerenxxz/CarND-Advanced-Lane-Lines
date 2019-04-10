## Project Writeup 


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

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./test_images/straight_lines1.jpg "Road Transformed"
[image3]: ./output_images/binary_images.png "Binary Example"
[image4]: ./output_images/Perspective_Transform2.png "Warp Example"
[image5]: ./output_images/detect_lane_line.png "Fit Visual"
[image6]: ./output_images/warp_back.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This document is my writeup.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first part named 'Camera Calibration and Distortion Measurement' of the IPython notebook located in "./find_lane_line.ipynb" 

I start by preparing "object points", which means the coordinates of the 3D chessboard corner points in real world space. Here we assume they are all on one plane, which is parallel to the camera and its z value is 0. Then we use function `cv2.findChessboardCorners` to get the coordinates of the 2D image points, and call them `imgpoints`.  

Then I used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I'll use an image to show how do I apply the distortion-correct:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this part is named 'Create thresholded binary images' in the same ipython notebook. At the beginning, I converted the original image to HLS space and extracted s channel. Then I used a combination of s channel image and x axis gradient thresholds to generate a binary image (I don't choose to use y axis because the car's nose will be detected through y axis gradient thresholds).  Here's an example of my output for this step. 
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code of perspective transform is the part named 'Perspective Transform'. I put the coordinates of the source points and desired coordinates of these points in the transformed images into the  `cv2.getPerspectiveTransform` function. And this funtion put out the transform matrix M, whcih we can use later to get the 'bird-eye' view. I chose the coordinates of the source and destination points in the following form:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 581, 460      | 250, 0        | 
| 195, 720      | 250, 720      |
| 1127, 720     | 1030, 720     |
| 703, 460      | 1030, 0       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this part is named 'Detect the Lane line boundries'. I used the result of the binary image as the input of this part. I began with find peak of the histogram of bottom half of the image as the position of the first window.Then I identified the nonzero pixels in this windows and recentered the window for the next level. After I got all the nonzero pixels in this image, I fit my lane lines with a 2nd order polynomial using the coordinates of these points,and they kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this part in named 'Calculate the radius of curvature and the vehicle position'. I first some weights into the fit function to convert amount of pixels into meters. Then I calculated the curvature of the two lines at the bottom if this image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the part caled 'Warp the lane line back to the original image'.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./P4_video_final.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There are some frames the detection of the lane lines may fail, so I added an Class to store the result of the previous frames, and applied a smoothness to the result. In the mean time, I used an function of `snaity_check` to ensure we detect the real lane lines. I also implemented an funtion `search_around_poly`, which means to search around the previous fit line, to decrease the computation load when find the nonzero pixels. But there isn't an significant decrease on the comptation time. 
In my project, I found 3 situations the pipieline likely fail:
1. When the car is turning rapidly
2. When there are great shadow on the road
3. There are color changes on the same lane line
In order to make it more robust, I have tried to use snaity check, Look-Ahead Filter and smoothing methods, and they worked well.
Hope that I can get some advice on this project!
