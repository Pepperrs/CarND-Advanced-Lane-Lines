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

[image0]: ./output_images/original_image.jpg "Original image"
[image1]: ./output_images/distorted.jpg "Distorted"
[image2]: ./output_images/undistorted.jpg "Undistorted"
[image3]: ./output_images/HLS_image.jpg "HLS image"
[image4]: ./output_images/HLS_warped_image.jpg "HLS warped"
[image5]: ./output_images/histogram.jpg "Histogram"
[image6]: ./output_images/boxes_lane_image.png "boxes and Lane"
[image7]: ./output_images/final_image.jpg "Final image"
[video1]: ./project_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in function calibration() in the file lanelines.py

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

Before:

![alt text][image1]

After:

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image0]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (The code can be found in lanelines.py#hls_gradient(image).  Here's an example of my output for this step.
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_image(image, warp_matrix)`, which appears in the file `lanelines.py`.  The `warp_image(image, warp_matrix)` function takes as inputs an image (`image`), as well as source (`warp_matrix`), which is generated by `generate_warp_config()`.

This function uses the following hardcoded src and dst points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 253, 697      | 303, 697      |
| 585, 456      | 303, 0        |
| 700, 456      | 1011, 0       |
| 1061, 690     | 101, 690      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the function `detect_lane_lines` using a histogram to detect the approximate location of the lane lines, I calculate what pixels in the binary map probably belong to either lane line.
Then I lay windows over the image and gather every activated pixel in the window.
The pixels are then aggregated for each of the two lane lines and given to the next function describend in the next section

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In `calculate_radius_and_center(image, left_lane_inds, nonzerox, nonzeroy, out_img, right_lane_inds)' I use the detected points from the last function to lay a polynomial over the points and convert it from pixel space to meter space. This results in left and right curve radius.
next I calculate the displacement from the center which is taken at the bottom of the image.
When Plotted, the results look like this:

![alt text][image5]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the function `draw_lines_to_image(image, left_fitx, original, ploty, right_fitx)` I draw an area on the original image using the unwarp funcion. I also write the approximated curvature and the distance to the center line to the image, which look like this:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline will likely fail, when there are large differences in lighting on the road, for example when there are lots of shadows. This could be adressed by making the hls_gradient() function more robust to color changes. Also the algorithm does not take the results of the last frames computation into consideration, which could improve the time it takes to compute the video.