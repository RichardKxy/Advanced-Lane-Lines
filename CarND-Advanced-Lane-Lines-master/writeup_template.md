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

[image1]: ./output_images/camera_calibration.png "Camera Calibration"
[image2]: ./output_images/undistort_output.png "Undistorted"
[image3]: ./output_images/distortion_corrected_color_image.png "Undistorted Color Image"
[image4]: ./output_images/RGB_HSV.png "RGB HSV"
[image5]: ./output_images/HLS_LAB.png "HLS LAB"
[image6]: ./output_images/binary_color_image.png "Binary Image"
[image7]: ./output_images/gradient.png "Gradient"
[image8]: ./output_images/warp.png "Warped Image"
[image9]: ./output_images/sliding_window.png "Sliding Window"
[image10]: ./output_images/histogram.png "Histogram"
[image11]: ./output_images/region.png "Rigion"
[image12]: ./output_images/function.png "Radius Function"
[image13]: ./output_images/plot.png "Plot Region"

[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced Lane Lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]

I used the distortion coefficients from Camera Calibration to accomplish the distortion correction with the `cv2.undistort()` function.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

At first, I tried four different color space to observe the performance of each channel of color image.

![alt text][image4]

![alt text][image5]

Then I observed R-channel of RGB and B-channel of LAB had more clear lane lines. Therefore, I combined binary R-channel and B-channel using the `binary()` function with different threshold.

![alt text][image6]

About gradients, I combined gradient in x direction and gradient in y direction with the `cv.sobel()` function as well as gradient direction and magnitude.

![alt text][image7]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`.  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[600,480],[720,480],[280,660],[1030,660]])
dst = np.float32([[450,0],[width-450,0],[450,height],[width-450,height]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 600, 480      | 450, 0        | 
| 680, 480      | 830, 0        |
| 280, 660      | 450, 720      |
| 1030, 660     | 830, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image8]

In addition, the code called `unwarp()` is to get inverse perspective transform.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First, I used the `sliding_window()` function to a 2nd order polynomial and histogram as follows:

![alt text][image9]

![alt text][image10]

Then the `region()` function is to use previous information to fit a 2nd order polynomial, which is useful for video

![alt text][image11]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines with `measure_curvature`.

First, I used below function to calculate the radius of curvature.

![alt text][image12]

Then, I used the polynomial function to calculate the position of vehicle.

The result of the test image as follows:

| Radius of left curve | Radius of right curve | The position of vehicle | 
|:--------------------:|:---------------------:|:-----------------------:| 
| 578.138541956 m      | 1199.9480261 m        | 0.0502552509988 m       |

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines in the function `draw()`. Here is an example of my result on a test image:

![alt text][image13]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I built a new Python notebook, "./Advanced Lane Lines Video.ipynb".

I added a class called `Line()` to modify the absence of lane lines.

Here's a [link to my video result](./project_video_out.mp4)

I tried to deal with the challenge video and got a not bad performance.

Here's a [link to my video result](./challenge_video_out.mp4)

About the harder challenge video, the performance is bad. But I didn't have much time to solve it.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The code finds the lane lines in binary images with histogram. Because of the perspective transformation, there exists much disturbance at the left and right side of the images. Therefore, I used left and right offset to eliminate the influence of the disturbance.

In the harder challenge video, the lumination, sharp curve, other vehicles and etc. have a great influence on the performance of this video. It's hard for the code to adjust itself quickly. Therefore, the performance is bad.