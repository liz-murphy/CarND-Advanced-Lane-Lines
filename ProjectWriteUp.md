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

[//]: # (Image References)

[image1]: ./output_images/test_undist.jpg "Undistorted"
[image2]: ./output_images/undisored_img.jpg "Undistorted image"
[image3]: ./output_images/original_img.jpg "Original image"

[image4]: ./output_images/combined_threshold.jpg "Color and Gradient Thresholding"

[image5]: ./output_images/perspective_source.png "Perspective Transform"

[video1]: ./project_video.mp4 "Video"

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

![Undistored checkerboard][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

- I first apply the distortion correction learned from the calibration to the test images. This can be found in Section 3 of the ipython notebook. Here is an example output image shown next to the original image.
![Original image][image3]
![Undistorted test image][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. These functions can be found in section 4 of the ipython notebook. 

The gradient thresholding selects:
- pixels that have a strong gradient in x and y, or a directional gradient of a certain magnitude

The color thresholding selects:
- pixels that have hue and saturation values common to white/yellow pixels.

The output at this stage looks like this:

![Color and Gradient Thresholding][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

 The `perspective_rectify()` function is in Section 5 of the iPython notebook. It takes as input an image (`img`).  I chose to the hardcode the source and destination points - these are detailed in the function itself.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Perspective Transform][image5]

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


![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

- The pipeline is obviously nowhere near real time.
- It 
