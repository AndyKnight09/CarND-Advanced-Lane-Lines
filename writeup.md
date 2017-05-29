# Advanced Lane Finding Project

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

[img1]: ./output_images/distorted.png "Distorted"
[img2]: ./output_images/undistorted.png "Undistorted"
[img3]: ./output_images/distorted2.png "Distorted"
[img4]: ./output_images/undistorted2.png "Undistorted"
[img5]: ./output_images/binary_combined.png "Binary Combined"
[img6]: ./output_images/source.png "Source Points"
[img7]: ./output_images/destination.png "Destination Points"
[img8]: ./output_images/region_of_interest.png "Region of Interest"
[img9]: ./output_images/window1.png "Windowing without apriori"
[img10]: ./output_images/window2.png "Windowing with apriori"
[img11]: ./output_images/final.png "Final output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first three code cells of the project [Jupyter notebook](./project.ipynb).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to one of the test images using the `cv2.undistort()` function and obtained this result: 

![distorted][img1]![undistorted][img2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the results of the camera calibration the first step in the data pipeline was to undistort each image using the following code `undist = cal_undistort(img, mtx, dist)`. Here is an example of this process using one of the test images:
![distorted][img3]![undistorted][img4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps are defined in code cells 4-6 in `project.ipynb`). This includes different color spaces, channels and different spacial gradients. Here's an example of my final output for this step which is a combination of multiple thresholding techniques:

![thresholding][img5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in code cell 7 of the Jupyter notebook. The `warp()` function takes as inputs an image (`img`) and uses a global set of source (`src`) and destination (`dst`) points. I chose the source and destination points using one of the test images:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 578, 460      | 320, 0        | 
| 185, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 704, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![source points][img6]
![destination points][img7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Once I had a thresholded binary image I applied the following region of interest (in code cell 8) so that rough terrain in the centre of the lane didn't cause issues for the line detection. (Note: the example shows a non-binary image to help illustrate the mask used).

![region of interest][img8]

I then used a histogram and windowing function to classify points as coming from right or left lane lines. Two different techniques were used; one where we don't have an existing estimate (code cell 11) for the lane lines and one where we do have a prior estimate (code cell 12). In each case once I had classified the points I used a simple second-order polynomial fit through each set of points to determine the lane lines. Here images showing the two techniques.

![window without apriori][img9]
![window with apriori][img10]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculate the radius of curvature of the lane by evaluating the radius of curvature of my two lane line polynomial fits for the point nearest to the car (i.e. at the bottom of the warped image). When doing this I needed to take into account the fact that the ratio of x:y in the warped image wasn't 1:1 in the world frame. Based on the perspective transformation and some knowledge about the scale of the lane markings a suitable x/y scaling could be used to convert the lane pixels back into world coordinates to get a sensible curvature. The `curvature` member function is defined in code cell 10 in the `Line` class.

In addition to the curvature I also calculated the position of the vehicle in the lane by assuming the camera is positioned in the centre of the vehicle. Therefore the position of the vehicle could be found by subtracting the centre of the lane (average of the two lane line fits evaluated at the bottom of the warped image) from the centre of the warped image. 

Having calculated these two values I then rendered the information over the top of the image using `cv2.putText()`. In addition I fitted a polygon to the lane fits in the warped space which was then transformed back into the camera frame using `unwarp()`.

![final output][img11]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my project video result](./test_videos_output/project_output.mp4) which manages to track the lane lines successfully.

Here's a [link to my challenge video result](./test_videos_output/challenge_output.mp4) which does reasonably at tracking the lane lines but has a couple of significant failures.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline derived during this project works reasonably well on the easiest of the project videos but doesn't manage to tackle the more challenging videos. I found that I needed to use a tracker to keep hold of my latest lane estimates in case I lost the lane lines for a couple of frames (which happened in the easiest project video).

I think that given more time I could probably get a slightly better result from the thresholding techniques which would be required for clips that have more varied lighting conditions (like the hardest challenge video). Also the tracker could be improved to store more information about how the lane lines are changing rather than just the last good lane  estimate. This would help the tracker to predict the lane position more accurately during dropouts.

A final area that I think could have been improved is validation of the lane estimates made. Currently I check that the lane curvature is sensible and that we have managed to match a certain number of points for each lane. However, I am not checking that lane estimates for left/right are consistent with each other (similar radius, parallel, expected width etc.).