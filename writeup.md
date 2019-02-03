
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
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the `Camera Calibration` section of the IPython notebook located in "./P2.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. Also, only those chessboards with `9x6` inner corners were considered for calibration 

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to all the images in the `./camera_cal` folder using the `cv2.undistort()` function and obtained the following result, which depicts the original image alongside the undistorted image with the chessboard corners coloured, wherever applicable. 

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

For the destination points, I chose a rectangle with left and right boundaries at 1/4th and 3/4th of the width, to accommodate curved lines as well (I discovered this after a few curved lines in the video weren't getting highlighted properly)

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

 1. Gradient Thresholding
	 Using grayscale images, I used the Sobel Operator to calculate the gradients in the X and Y directions separately to identify pixels that fell within a given threshold. Then, I found the overall magnitude and direction of the gradient in both X and Y directions to identify pixels that fell within a given threshold.
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

 2. Color Thresholding
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

 3. Color + Gradient Thresholding
	Finally, I combined the gradient thresholds and color thresholds obtained above by using the H and S channels from the HLS color space + R and B channels from the RGB color space + Gradients in the X and Y directions + Magnitude and Direction of the Gradient, in the following manner:
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

I used the sliding window approach outlined in the lecture to fit the positions of these pixels with a polynomial. The hyperparameters were set as follows:
|  Parameter      | Values | 
|:-------------:|:-------------:| 
|  Number of sliding windows     | 9       | 
|  Margin of sliding window (pixels)     | 100      |
|  Minimum number of pixels required to recenter window      | 50      |

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

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
