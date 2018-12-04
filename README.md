Advanced Lane Detection Project
===
In the second project of Udacity Autonomous nano programs, Advanced Lane detection, I practiced actual implementation of advanced image processing methods to identify the lane on the road, and calculate the car offset between lanes and curvature of the lanes.

![Sample Lane Detection Results](https://raw.githubusercontent.com/masih84/CarND-Advanced-Lane-Lines/master/test_videos_output/gif_exampel.gif)


The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

A jupyter/iPython notebook was used  on github [Full Project Repo](https://github.com/masih84/CarND-Advanced-Lane-Lines) - [Advanced Lane Finding Project Notebook](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/examples/Advanced_Lane_Code.ipynb). The project is written in python and its utilises such as [numpy](http://www.numpy.org/) and [OpenCV](http://opencv.org/).

## Camera Calibration
To fix the camera distortion and identify the calibration factors, images of chessboard were used and camera is calibrated . In this process, test images are converted to gray and the chessboard corners is identified by `cv2.findChessboardCorners()`. The calibration factors `mtx` and `dist` are found with `cv2.calibrateCamera()`.  The following image shows the comparison between original chessboard and calibrated result after finding the chess corners and fixing the perspective.

![Original and Undistorted image](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/Output_calibration_9.jpg?raw=true)

### Example of undistorted calibration image 
The `cv2.undistort()` takes an image and defaults the `mtx` and `dist` variables from the previous camera calibration before returning the undistorted image. Here is example of original and undistorted camera image:
![test image distorted and undistorted](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/test_image_undistorted_2.jpg?raw=true)


### Threshold binary images
 After correction of the image distortion, combination of HSL color transforms, gradients in x direction `cv2.Sobel`, was used for filtering the image in `thereshold_image()`. This function receives the image and thresholds limits and filtered result is provided as an Binary image.  The threshold limits are tuned to provide a good lane detection for project video. Here are two examples of images after filtering with `thereshold_image()`. Right and left lanes are clear in both binary images.


![original image and Threshold binary](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/test_image_thereshold_1.jpg?raw=true)

![original image and Threshold binary](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/test_image_thereshold_5.jpg?raw=true)

### Perspective transform 
Perspective transformation has been done using `warp_image()`.Four points are selected as sourse on straight-line test image to match the road lanes, and destination points are selcted to create a rectangle as destination image. Transformation matrix `M` and its inverse `Minv` are calculated using `cv2.getPerspectiveTransform`.
![Distorted with bird's eye view](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/test_image_perspective_transformed_2.jpg?raw=true)


### Lane-line pixel identification and polynomial fit
Having a bird-eye view, I used sliding windows method to find right and left lanes pixels using `find_lane_pixels`. The start point for sliding window is selected based on two picks on histogram found using `hist` function. Then using  `np.polyfit`, we can easily find the fitted polynomials on both lanes. Following images show example of identified lanes using sliding windows and fitted polynomials. For visualization purpose, the area between polynomials are highlighted green.

![Warped box seek and new polynomial fit](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/identified_lane_2.jpg?raw=true)
For visualization purpose, the are between polynomials are highlighted green.

### Radius of curvature calculation and vehicle from center offset
Having lane polynomials, `measure_curvature_real` measure the curvature with equation described here:  (https://www.intmath.com/applications-differentiation/8-radius-curvature.php). The identified polynomials in pixel domain is converted to meter with following conversion factors are selected as `xm_per_pix` = 10 *3/700  and `ym_per_pix` = 3.7/700 based on U.S. regulations regarding  minimum lane width of 12 feet or 3.7 meters, and the dashed lane lines are 10 feet or 3 meters long each. 

I used `find_car_offse` in order to find car offset where `offset = ((rx - xcenter) - (xcenter - lx))*xm_per_pix` where `xcenter` and `ycenter` is image ceneter, and `rx` and `ry` are location of fitted polynomials at the bottom of image.


### Final pipeline
In the pipeline, we analyze  the image to find the road lanes and keep track of past ten lane polynomials. Input to the pipeline is array of images that found by reading the video file using `load_video_imgs`.

For the first image, we use ` find_lane_pixels` (sliding windows method) to find lane pixels. However, after having average polynomial for past 10 lanes, the new lane is identified by ` search_around_poly` only searching round the previous average lanes. 

The difference between previous average polynomial and new polynomial is calculated to ensure new polynomials are valid. If that difference is greater than a selected threshold, we ignore the result, and re-do the lane finding based on sliding windows.

#### Sample result images
The final result, including highlighted area between lanes, right and left curvature radius, car offset, are created as images. Here are two sample of final pipline results:
![`lane.result_decorated`](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/Final_result_2.jpg?raw=true)

![Lane.warped_decorated](https://github.com/masih84/CarND-Advanced-Lane-Lines/blob/master/output_images/Final_result_5.jpg?raw=true)

## Result Video

I used [moviepy](http://zulko.github.io/moviepy/) to process the project video.  The [Project Video Lane mp4 on GitHub](https://raw.githubusercontent.com/masih84/CarND-Advanced-Lane-Lines/blob/master/project_video_lane.mp4?raw=true), has the result ([YouTube Link](https://youtu.be/lVQZknhh-Bs))

## Discussion
This project was a great practice to see how the image processing can be used for lane identifications. The moving averaging technique has a big impact on smoothing my result. Identifying outlier result and resting my code using sliding windows was very effective to avoid any jumps and non-sense lanes.

### Problems/Issues faced
I am quite new to Python, so most of my time was spend on developing python code and reading different library to learn how various functions of `numpy` or `Moviepy` works. I tried to process tha challenges video and found few issues to improve but ran out of time.
