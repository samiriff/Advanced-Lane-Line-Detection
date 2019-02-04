

## Writeup 

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

[image1]: ./output_images/calibration.png "Undistorted"
[image2]: ./output_images/distortion_correction.png "Distortion Correction"
[image3]: ./output_images/interactive_slider_unwarp_src.png "Interactive Slider Unwarp Src"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./output_images/interactive_slider_gradient_threshold.png "Interactive Slider Gradient Threshold"
[image6]: ./output_images/gradient_thresholding.png "Gradient Thresholding"
[image7]: ./output_images/color_thresholding_rgb.png "Color Thresholding RGB"
[image8]: ./output_images/color_thresholding_hls.png "Color Thresholding HLS"
[image9]: ./output_images/color_thresholding_hsv.png "Color Thresholding HSV"
[image10]: ./output_images/combined_color_gradient_thresholding.png "Combined Color and Gradient Thresholding"
[image11]: ./output_images/warp_and_threshold.png "Warp and Threshold Image"
[image12]: ./output_images/histogram.png "Histogram Peaks"
[image13]: ./output_images/fit_polynomial.png "Fit Polynomial"
[image14]: ./output_images/curvature.png "Curvature"
[image15]: ./output_images/lane_center.png "Lane Center"
[image16]: ./output_images/unwarp.png "Unwarp"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the `Camera Calibration` section of the IPython notebook located in "./P2.ipynb".

In the `calibrate_camera()` method, I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. Also, only those chessboards with `9x6` inner corners were considered for calibration 

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  In the `undistort_images()` method, I applied this distortion correction to all the images in the `./camera_cal` folder using the `cv2.undistort()` function and obtained the following result, which depicts the original image alongside the undistorted image with the chessboard corners coloured, wherever applicable. 

![Undistorted][image1]

To prevent myself from re-running this step every time I opened this notebook, I saved the camera matrix and distortion coefficients in a file using the `pickle` package.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images:
![distortion_corrected][image2]

The code for this step can be found in the `Distortion Correction` section of the notebook.

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform can be found in the `Warp Perspective` section of the notebook, which includes a function called `corners_unwarp_lane()`.  The `corners_unwarp_lane()` function takes as inputs an image (`img`), calibrated camera matrix (`mtx`), distortion coefficients(`dist`) as well as source (`src`) and destination (`dst`) points. There is also a boolean parameter(`overlay_line_markings`) to control whether a polygon should be drawn to highlight the selected `src` points in the given image. 

To determine the `src` points, I used a test image which contained straight lane lines so that they could be warped into parallel lines in the result. I used interactive slider widgets from the `IPyWidgets` library to adjust coordinates applied on the test image, as depicted below:

![interactive_slider_unwarp_src][image3]

For the destination points, I chose a rectangle with left and right boundaries at 1/4th and 3/4th of the width, to accommodate curved lines as well (I found this after a few curved lines in the video weren't getting highlighted properly)

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 251, 685      | 320, 720        | 
| 595, 450      | 320, 0      |
| 686, 450      | 960, 0      |
| 1054, 685     | 960, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![warped_straight_lines][image4]

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image, as outlined in the `Thresholding` section of the notebook, which is divided into 3 parts:

 1. Gradient Thresholding: 
 
	 I used the Sobel Operator on grayscale images to calculate the gradients in the X and Y directions separately to identify pixels that fell within a given threshold. Then, I found the overall magnitude and direction of the gradient in both X and Y directions to identify pixels that fell within a given threshold.
	 To identify the appropriate threshold values and kernel size, I made use of interactive sliders from the `IPyWidget` library, as shown below:
	 
	![interactive_slider_gradient_threshold][image5]
	
	The values I identified are as follows:

	|  Parameter      | Threshold Values | 
	|:-------------:|:-------------:| 
	|  Kernel Size     | 5        | 
	|  Gradient in X and Y Directions     | Min: 45, Max: 120      |
	|  Magnitude of Gradient     | Min: 21, Max: 221      |
	|  Direction of Gradient   | Min: 0.60, Max: 1.30      |

	A sample of the output with these values on a test image is as follows:
	![gradient_thresholding][image6]
	Note that the `combined` image combines the Gradient in X and Y directions with the Magnitude and direction of the gradient. 

 2. Color Thresholding:
 
	I experimented with images in all 3 color spaces viz., RGB, HSL and HSV. For each color space, I extracted each channel and applied thresholds to them. To identify the appropriate threshold values, I made use of interactive sliders from the `IPyWidget` library, and obtained the following values:

	|  Parameter      | Threshold Values | 
	|:-------------:|:-------------:| 
	|  R Channel     | Min: 39, Max: 215       | 
	|  G Channel     | Min: 47, Max: 222      |
	|  B Channel     | Min: 44, Max: 132      |
    |  H Channel     | Min: 15, Max: 100      |
  	|  L Channel     | Min: 15, Max: 200      |
	|  S Channel     | Min: 90, Max: 255      |
    |  H Channel     | Min: 15, Max: 100      |
  	|  S Channel     | Min: 15, Max: 200      |
	|  V Channel     | Min: 90, Max: 255      |
	
	A sample of the output with these values on a test image is as follows:
	![color_thresholding_rgb][image7]
	![color_thresholding_hls][image8]
	![color_thresholding_hsv][image9]

 3. Color + Gradient Thresholding:
 
	Finally, in the `combined_color_gradient_thresholding()` method, I combined the gradient thresholds and color thresholds obtained above by using the H and S channels from the HLS color space + R and B channels from the RGB color space + Gradients in the X and Y directions + Magnitude and Direction of the Gradient, in the following manner:
	`((h_channel == 1) & (s_channel == 1)) |
	((r_channel == 0) & (b_channel == 1)) |
	((mag_binary == 1) & (dir_binary == 1)) |
	((gradx_binary == 1) & (grady_binary == 1))`

	The following output was obtained:
	![combined_color_gradient_thresholding][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the `Sliding Window` section, I first applied the combined threshold obtained above to a warped image to check if the output was proper (`warp_and_threshold()` method). As seen in the image below, the left and right lane lines can clearly be seen against the black background
![warp_and_threshold][image11]

Then, to determine which pixels in the resulting image were part of the left or right lane lines, I plotted a histogram of the bottom half of the image using the `find_histogram()` method. The left and right peaks in the histogram indicate the bottom portions of the left and right lanes respectively (`find_histogram_peaks()` method)
![histogram][image12]

I used the sliding window approach outlined in the lecture to fit the positions of these pixels with a polynomial, by first identifying the X and Y positions of all non-zero pixels in the image, followed by iterating through the windows and centering each window around the mean position if the number of pixels within that window was greater than the minimum number of pixels hyper-parameter. The hyperparameters were set as follows:

|  Parameter      | Values | 
|:-------------:|:-------------:| 
|  Number of sliding windows     | 9       | 
|  Margin of sliding window (pixels)     | 100      |
|  Minimum number of pixels required to recenter window      | 50      |

After determining all the relevant lane pixels, 2 second order polynomials were fitted to these pixels in the left and right halves. The figure below depicts these 2 polynomials with the left and right halves coloured in red and blue respectively, and each window is outlined in green.
![fit_polynomial][image13]

#### 5. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the `Unwarp` section, I used the code snippet provided in the `Tips and Tricks` topic which uses the X and Y coordinates obtained from the left and right polynomials plotted earlier, to plot a filled green polygon onto a blank image. This image is then unwarped using the inverse of the perspective transform matrix obtained earlier and the result is overlaid on the original image as shown below:
![unwarp][image16]

#### 6. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the `Measuring Curvature` section, I used the left and right polynomials obtained above in pixel space to derive 2 new polynomials in world space, assuming that a 30 meter long lane is represented by 720 pixels and a 3.7 meter wide lane is represented by 700 pixels. 

First, 2 lists of X coordinates for the left and right lanes are obtained from each polynomial in pixel space for each value of Y from 0 to 720. These lists are used to fit 2 new polynomials in world space, and the radii of curvature of the left and right lanes closest to the vehicle are obtained using the following formula, with `y` set to the bottom of the image:
![curvature][image14]

To calculate the lane center, I used the 2 polynomials in pixel space to determine X coordinates of the left and right lanes with `y` set to the bottom of the image. Then, I found the mid-point of these 2 X- coordinates in pixel space and subtracted the center of the image (640) from it. The result was converted into meters. 
![lane_center][image15]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://www.youtube.com/watch?v=1BVLh1tesGM&feature=youtu.be)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The `Pipeline` section uses all the methods defined in the previous sections to process every frame of the final video to highlight lane lines. In addition, it makes use of a class called `Line` which keeps track of a previous frame's left and right lane polynomials. This helps in removing outliers by assuming that any new X coordinate determined by a polynomial at a given Y coordinate should lie within 50 pixels of the X coordinate determined by a polynomial of the previous frame at that same Y coordinate. I found that this technique helped in frames where slight shadows or road texture changes tended to alter the lane line markings. 

There is also a `Frame Extraction for Debugging` section which helps in extracting frames of the video in which lane lines are not being highlighted properly, which was useful for debugging purposes. 

The `IPyWidget` library helped me immensely in experimenting with different threshold values. However, determining suitable thresholds for gradient and colors proved to be a task with a lot of trial and error. I believe I found the threshold values by chance, and it works properly only for the above video. For other videos, I would have to find some other threshold values. 

My pipeline would likely fail in regions with continuous shadows or if any lane is covered by another object such as a car or other lines formed by the texture of the road, as seen in the challenge video. My pipeline would also fail on roads where the lane lines curve drastically, due to the logic I am relying on in the `Line` class. 

To make this pipeline more robust, I will have to keep track of the polynomials detected in at least a few frames of the video, to validate a polynomial fit in a new frame. I would also have to experiment to find better gradient and color thresholds that can be applied to the challenge videos. To improve the time taken by the pipeline to process the video input, I shouldn't be running the sliding window logic afresh for each and every frame, but instead rely on previous frames. I could also use the curvature values of previous frames to fine-tune the highlighted lanes.