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

[image1]: ./output_images/chess_board.png "Chessboard"
[image2]: ./test_images/test4.jpg "Test Road Image"
[image3]: ./output_images/undist.png "Undistorted"
[image4]: ./output_images/combined_binary.png "Binary Example"
[image5]: ./output_images/warped.png "Warp Example"
[image6]: ./output_images/fit.png "Fit Visual"
[image7]: ./output_images/final_result.png "Output"
[image8]: ./output_images/undist_example.png
[image9]: ./output_images/dist_example.png
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook Advanced_Finding_Lanes.ipynb.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
The distortion-corrected version of above image is shown below:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of HLS color scheme and x-gradient thresholds to generate a binary image (see the function `threshold()` defined in the third cell of Advanced_Finding_Lanes.ipynb). Here's an example of my output for this step.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is a function called `perspective_transform()` defined in the third cell of  Advanced_Finding_Lanes.ipynb.  The `perspective_transform()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[575,464],[710,464], [1093,714], [218,714]])
img_size = (img.shape[1], img.shape[0])
dst = np.float32([[300,0],[950,0], [950,img_size[1]], [300,img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 300, 0        | 
| 710, 464      | 950, 0        |
| 1093, 714     | 950, 720      |
| 218, 714      | 950, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image8]
![alt text][image9]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identifying lane-line pixels and fit their positions with a 2nd polynomial is a function `fit_binary_warped()` defined in the third cell of Advanced_Finding_Lanes.ipynb. I used the silding window method introduced in the class and I fitted my lane lines with a 2nd order polynomial like this:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for calculating the radius of curvature of lane is a funciton `curvature()` defined in the third cell of Advanced_Finding_Lanes.ipynb. I used the math equaiton for radius of curvature introduced in class and the unit is also transformed in meter.

The code for calculating the position of the vehicle with respect to center is a function `distance_from_center()` defined in the third cell of Advanced_Finding_Lanes.ipynb. The unit is also tranformed into meter. The radius of curvature and position information are displayed in the final video result.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function `drawing()` defined in the third cell of Advanced_Finding_Lanes.ipynb.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One porblem I faced in the implementation is in the stage of fitting the bianry warped picture of lanes. Since I use the sliding window method, I need to properly select the margin parameter of the sliding window. When I first set the margin as 90, for test4.jpg, I didn't identify the correct right lane line, so I increase the margin to 110 to absorb more data and then the results become correct.

Two factors that will affect the robustness and might make pipeline fail are the quality of road binary image and how you fit the lane lines data. In this case, I combine the x-gradient and HSL color to pick up the lane pixels. It works fine for project_video.mp4, but when I test is on challenge_video.mp4, it just fails to test the correct the line. Apprently, there are other lane-line-like noises in challenge_video.mp4 such as the shadow of middle curb and the marks of road repairing. So we might need to choose better thresholding method and tuning its parameter to pick out the real lane line pixels effectively and also tune the fitting algorithm accordingly. 
