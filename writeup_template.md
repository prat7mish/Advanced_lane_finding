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
Please refer the location: "./CarND-Advanced-Lane-Lines/output_images/" for output images
and refer the location: "./CarND-Advanced-Lane-Lines/project_video_solution.mp4" for solution for sample video provided detecting lanes on a curved road.


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Pipeline

#### 1. Compute the camera calibration using chessboard images (refer cell 3).

The code for this step is contained in the first code cell of the IPython notebook located in "./advance_lane_finding.ipynb".  

A set of chessboard images will be used for this purpose.

I have defined the calibrate_camera function which takes as input parameters an array of paths to chessboards images, and the number of inside corners in the x and y axis.

For each image path, calibrate_camera:

It reads the image by using the matplotlib.image.imread function,
converts it to grayscale usign cv2.cvtColor,
find the chessboard corners usign cv2.findChessboardCorners
Finally, the function uses all the chessboard corners to calibrate the camera by invoking cv2.calibrateCamera.

The values returned by cv2.calibrateCamera will be used later to undistort our video images.

#### 2. Apply a distortion correction to raw images (refer cell 4).

cv2.undistort is then used with the values returned by cv2.calibrateCamera to undistort the image.

The sample output of the same could be found at (./CarND-Advanced-Lane-Lines/output_images/undistorted_image.png)


#### 3. Use color transforms, gradients, etc., to create a thresholded binary image (refer cells 5-16).

In this step, we will define the following funtions to calculate several gradient measurements (x, y, magnitude, direction and color).

Calculate directional gradient: abs_sobel_thresh().
Calculate gradient magnitude: mag_thres().
Calculate gradient direction: dir_thresh().
Calculate color threshold: col_thresh().
Then, combine_threshs() will be used to combine these thresholds, and produce the image which will be used to identify lane lines in later steps.

The sample outputs of the step could be found at below 
1  ./CarND-Advanced-Lane-Lines/output_images/undistorted_image.png
2  ./CarND-Advanced-Lane-Lines/output_images/threshold_gradient_x.png
3  ./CarND-Advanced-Lane-Lines/output_images/threshold_gradient_y.png
4  ./CarND-Advanced-Lane-Lines/output_images/thresholded_direction_image.png
5  ./CarND-Advanced-Lane-Lines/output_images/thresholded_magnitude_image.png
6  ./CarND-Advanced-Lane-Lines/output_images/combined_thresholded.png
7  ./CarND-Advanced-Lane-Lines/output_images/color_thresholded.png



#### 4. Apply a perspective transform to rectify binary image ("birds-eye view") (refer cells 17-18).

The next step in our pipeline is to transform our sample image to birds-eye view.

The process to do that is quite simple:

First, you need to select the coordinates corresponding to a trapezoid in the image, but which would look like a rectangle from birds_eye view.
Then, you have to define the destination coordinates, or how that trapezoid would look from birds_eye view.
Finally, Opencv function cv2.getPerspectiveTransform will be used to calculate both, the perpective transform M and the inverse perpective transform Minv.
M and Minv will be used respectively to warp and unwarp the video images.

The sample output for this step could be found at ./CarND-Advanced-Lane-Lines/output_images/warped_image.png


#### 5. Detect lane pixels and fit to find the lane boundary (refer cell 20-22).

In order to detect the lane pixels from the warped image, the following steps are performed.

First, a histogram of the lower half of the warped image is created.

Then, the starting left and right lanes positions are selected by looking to the max value of the histogram to the left and the right of the histogram's mid position.

A technique known as Sliding Window is used to identify the most likely coordinates of the lane lines in a window, which slides vertically through the image for both the left and right line.

Finally, using the coordinates previously calculated, a second order polynomial is calculated for both the left and right lane line. Numpy's function np.polyfit will be used to calculate the polynomials.

Once you have selected the lines, it is reasonable to assume that the lines will remain there in future video frames. detect_similar_lines() uses the previosly calculated line_fits to try to identify the lane lines in a consecutive image. If it fails to calculate it, it invokes detect_lines() function to perform a full search.

The sample output for this step could be found at:
1  ./CarND-Advanced-Lane-Lines/output_images/sliding_window.png
2. ./CarND-Advanced-Lane-Lines/output_images/lane_lines_detected.png

#### 6. Determine the curvature of the lane, and vehicle position with respect to center (refer cell 23-24).

Here, following metrics will be calculated: the radius of curvature and the car offset.


#### 7. Warp the detected lane boundaries back onto the original image (refer cells 27-28).

The next step will be to draw the lanes on the original image:

First, we will draw the lane lines onto the warped blank version of the image.
The lane will be drawn onto the warped blank image using the Opencv function cv2.fillPoly.
Finally, the blank will be warped back to original image space using inverse perspective matrix (Minv).
This code is implemented in the draw_lane() function.

The sample output for this step could be found at ./CarND-Advanced-Lane-Lines/output_images/lane_with_green_markings.png

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

I checked on my video and noticed the wobbly nature of the detected lanes in below conditions:
1. When there was some shadow variations in the video on the road.
2. During the car turning towards right hand side.

Such incidents can be ignored by taking proper thresholds and value of coordinates.
