## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

-----------------------------------

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Pipeline(single images)

#### 1. Compute the camera calibration using chessboard images (refer cell 3).

The code for this step is contained in the ----- code cell of the IPython notebook located in "./.ipynb".  

The OpenCV functions findChessboardCorners and calibrateCamera are the backbone of the image calibration. A number of images of a chessboard, taken from different angles with the same camera, comprise the input. Arrays of object points, corresponding to the location (essentially indices) of internal corners of a chessboard, and image points, the pixel locations of the internal chessboard corners determined by findChessboardCorners, are fed to calibrateCamera which returns camera calibration and distortion coefficients. 

#### 2. Apply a distortion correction to raw images (refer cell 4).

The values returned by the above step are used by the OpenCV undistort function to undo the effects of distortion on any image produced by the same camera. Generally, these coefficients will not change for a given camera (and lens). 

The below image depicts the corners drawn onto twenty chessboard images using the OpenCV function drawChessboardCorners:
-----give the images path---------

#### 3. Use color transforms, gradients, etc., to create a thresholded binary image (refer cells 5-16).

In this step, we will define the following funtions to calculate several gradient measurements (x, y, magnitude, direction and color).

Then, combine_threshs() will be used to combine the thresholds, and produce the image which will be used to identify lane lines in later steps.

The sample outputs of the step could be found at below 
1  ./CarND-Advanced-Lane-Lines/output_images/undistorted_image.png

#### 4. Apply a perspective transform to rectify binary image ("birds-eye view") (refer cells 17-18).

The next step in our pipeline is to transform our sample image to birds-eye view.

The process to do that is quite simple:

First, you need to select the coordinates corresponding to a trapezoid in the image, but which would look like a rectangle from birds_eye view.
Then, you have to define the destination coordinates, or how that trapezoid would look from birds_eye view.
Finally, Opencv function cv2.getPerspectiveTransform will be used to calculate both, the perpective transform M and the inverse perpective transform Minv.
M and Minv will be used respectively to warp and unwarp the video images.

The sample output for this step could be found at -------------------


#### 5. Detect lane pixels and fit to find the lane boundary (refer cell 20-22).

In order to detect the lane pixels from the warped image, the following steps are performed.

First, a histogram of the lower half of the warped image is created.

Then, the starting left and right lanes positions are selected by looking to the max value of the histogram to the left and the right of the histogram's mid position.

A technique known as Sliding Window is used to identify the most likely coordinates of the lane lines in a window, which slides vertically through the image for both the left and right line.

Finally, using the coordinates previously calculated, a second order polynomial is calculated for both the left and right lane line. Numpy's function np.polyfit will be used to calculate the polynomials.

Once you have selected the lines, it is reasonable to assume that the lines will remain there in future video frames. detect_similar_lines() uses the previosly calculated line_fits to try to identify the lane lines in a consecutive image. If it fails to calculate it, it invokes detect_lines() function to perform a full search.

The sample output for this step could be found at:
1  -----


#### 6. Determine the curvature of the lane, and vehicle position with respect to center (refer cell 23-24).

The radius of curvature is based upon this website and calculated in the code cell titled "Radius of Curvature and Distance from Lane Center Calculation" using this line of code (altered for clarity):

curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])

In this example, fit[0] is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). y_meters_per_pixel is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:

lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
r_fit_x_int and l_fit_x_int are the x-intercepts of the right and left fits, respectively. This requires evaluating the fit at the maximum y value (719, in this case - the bottom of the image) because the minimum y value is actually at the top (otherwise, the constant coefficient of each fit would have sufficed). The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).


#### 7. Warp the detected lane boundaries back onto the original image (refer cells 27-28).

The next step will be to draw the lanes on the original image:

First, we will draw the lane lines onto the warped blank version of the image.
The lane will be drawn onto the warped blank image using the Opencv function cv2.fillPoly.
Finally, the blank will be warped back to original image space using inverse perspective matrix (Minv).
This code is implemented in the draw_lane() function.

The sample output for this step could be found at ----------

#### 8. Display lane boundaries and numerical estimation of lane curvature and vehicle position (refer cells 29-30).

The next step is to add metrics to the image.The method add_metrics() receives an image and the line points and returns an image which contains the left and right lane lines radius of curvature and the car offset.

This function makes use of the previously defined curvature_radius() and car_offset() function.

The sample output for this step could be found at ./CarND-Advanced-Lane-Lines/output_images/lane_with_metrics.png

#### 9. Run pipeline in a video (refer cells 31-32).

In this step, we will use all the previous steps to create a pipeline that can be used on a video.

Create the ProcessImage class which allows to calibrate the camera when initializing the class and also keep some track of the previously detected lines.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_solution.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration, etc.
Also there would definitely be an issue in snow or in a situation where, for example, a bright white car were driving among dull white lane lines. 

I've considered a few possible approaches for making my algorithm more robust. These include more dynamic thresholding (perhaps considering separate threshold parameters for different horizontal slices of the image, or dynamically selecting threshold parameters based on the resulting number of activated pixels), designating a confidence level for fits and rejecting new fits that deviate beyond a certain amount (this is already implemented in a relatively unsophisticated way) or rejecting the right fit (for example) if the confidence in the left fit is high and right fit deviates too much (enforcing roughly parallel fits).
