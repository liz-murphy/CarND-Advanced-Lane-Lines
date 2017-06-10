## Advanced Lane Finding Writeup

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

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.    

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./Project.ipynb" ..  

- I declare the "object points" which are the known (x,y,z) coordinates of the chessboard. I assume the board is fixed at z=0,
and the coordinates are on a regular grid. 
- The 2D image points are found using OpenCV's findChessboardCorners(...) function, which identifies all the corners in the supplied image. To get a good calibration, we need a number of images containing these chessboards taken from different angles.
I loop through the supplied calibration images and add the image point and object points (which doesn't change) to a list.
- Now that I know the correspondence between the object points and the image points, I can make use of OpenCV's calibrateCamera(...) function to find the camera matrix and the distortion coefficients.

I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
<img src="/camera_cal/calibration3.jpg" alt="alt text" width="300" >
<img src="/output_images/test_undist.jpg" alt="alt text" width="300" >

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

- I first apply the distortion correction learned from the calibration to the test images. This can be found in Section 3 of the ipython notebook. Here is an example output image shown next to the original image.
<img src="/output_images/keep/undistored_img.jpg" width="300"><img src="/output_images/keep/original_img.jpg" width="300">

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. These functions can be found in section 4 of the ipython notebook. 

The gradient thresholding selects:
- pixels that have a strong gradient in x and y, or a directional gradient of a certain magnitude

The color thresholding selects:
- pixels that have hue and saturation values common to white/yellow pixels.

The output at this stage looks like this:

<img src="/output_images/combined_threshold.jpg">

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

 The `perspective_rectify()` function is in Section 5 of the iPython notebook. It takes as input an image.  I chose to hardcode the source and destination points - these are detailed in the function itself.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

<img src="/output_images/perspective_source.png">

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used a sliding-window convolutional technique to identify lane-line pixels. This is described in Section 8 of the iPython notebook, in the comments of find_lane_lines(...). Some things to note:
- this function takes left and right lane objects as parameters
- it uses the position of previously detected lines to narrow down the initial search window on a new frame
- if the convolution returns the left-most (smallest index) as the highest scoring pixel, the corresponding value is checked against the center pixel of the template window being evaluated. This is to add some robustness in the case of all values being near zero and equal, when np.convolve will return the smallest index. Instead we reset it to the center of the template window.

The raw output of the lane line detection is then passed to polyfit_lane_lines(...).
- this function also takes the line objects as input
- it fits a 2D polynomial to the raw output of the lane-line finder on the most recent image.
- it computes the squared error of each point to the new fit, and uses 1/err as a weight for each point
- the points are then combined with the output of the last n (n=5) images, which are stored in the Lane object in a deque.
- the combined points and their weights are fitted with a second "best-fit" polynomial. It is this line that I use to display the lane lines on the final image.

<img src="/output_images/perspective_warp_img.jpg" width=300><img src="/output_images/lane_finding_img.jpg" width=300>

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I used the equations [here|http://www.intmath.com/applications-differentiation/8-radius-curvature.php] to find the radius of curvature for each lane line.
- I made a minor modification to deal with divide-by-zero issues on straight lines
- I eyeballed some warped images to come up with a pixel-to-m conversion. My lane width tended to average 890 pixels (3.7m in the real world), and the dashed lane lines were around 110 pixels (3m in the real world).

To find the lane position:
- I used the x-intercept of my best fit line with the bottom of the image. The average of the left and right lines gives the center of the lane. The difference between that, and the midpoint of the image, is the distance of the vehicle from the center of the lane. (This all assumes the camera is mounted in the center of the vehicle).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here's an image which details the lane curvature, width and fills in the detected lane.

<img src="/output_images/total_pipeline.png" width=300>

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Difficulties:
- sensitivity to color, lighting, detecting the railing as a lane line!

When will it fail:
- different times of day when the color and lighting changes
- rain, snow
- different road geometries
- occlusions in heavy traffic
- dirt on the camera lens
- changes to calibration, i.e. you can't move this camera very far without breaking the assumptions that underpin the perspective warping.
- it's also nowhere near real time.

To make it more robust:
- first step would be to look at how I create the birds-eye view. I would pick the source points for the perspective warp
dynamically, based on the current image properties and a rudimentary estimate of where the lines are from a first pass of the lane detector.
- implement multiple color/gradient filters and dynamically choose the best one.
- more sophisticated tracking. Right now I just weight individual points by their distance to the fitted line.


