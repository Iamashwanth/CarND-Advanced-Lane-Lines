## Project writeup

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

[image1]: ./p_bin.png "Undistorted"
[image2]: ./p_undist.png "Road Transformed"
[image3]: ./p_bin.png "Binary Example"
[image4]: ./p_warped.png "Warp Example"
[image5]: ./p_lines.png "Fit Visual"
[image6]: ./p_result.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I am using the same code from the lectures.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistorted image][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Undistorted image][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used `cv2.inRange()` function to find yellow and white lines in the image.
`OR` operation is performed on these two lines image outputs and passed down to rest of the pipeline.

![Thresholded image][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I have selected below source and destination points for perspective transform.
`cv2.getPerspectiveTransform()` is used to get transformation matrix.
`cv2.warpPerspective()` is used to transformation the image.

`perspective_trans()` is used to implement this funcionality.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 720,  450     | 1100, 100     | 
| 1080, 650     | 1100, 720     |
| 200,  650     | 100,  720     |
| 560,  450     | 100,  100     |

![Perspective transform][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Plotting a histogram of the bottom half of the image and looking for the peaks helped in finding the base of the lines. Once the lane bases are found, I used sliding window approach to find the rest of the lane by adjusting the window center with every iteration by computing an average on the pixels found in the current window.

I used `np.polyfit()` on the pixel positions that belong to the left and right lanes to fit a line through them.

Code to perform these operations is part of `find_lanes()` and `fit_lanes()` functions.

![Sliding window image][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

First I converted the left and right lane position from pixels to meters.
After this I used the formula mentioned in the lectures to compute the lane curvature.

`lane_curvature()` is used to compute the lane curvature.

Since the camera is mounted at the center of the car, we can find the position of the vehicle with respect of center by computing lane center and comparing it with the image center.

`car_offset()` is used to compute the vehicle offset

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of my result on a test image:

![Result image][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

`run_pipeline()` is used to run a frame through the pipeline.

Here's a [link to my video result](./output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I have used `inRange()` to filter out the lane lines, but this is prone to errors under varying light conditions. To improve this, information from S and H channels of a HSV image can be used.

My pipeline uses sliding window approach on every single frame of the video. But the data from the previous frames can be used to make this computation less intensive.
