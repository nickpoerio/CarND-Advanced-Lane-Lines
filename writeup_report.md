## Writeup Report

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

[image1]: ./output_images/calib_img2_undistort.png "Undistorted calibration image"
[image2]: ./test_images/test1.jpg "Test 1 image"
[image2bis]: ./output_images/test1_undistort.jpg "Undistorted test image"
[image3]: ./output_images/test1_binary.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./output_images/straight1_fit.png "Fit straight1"
[image5bis]: ./output_images/test1_fit.png "Fit test1"
[image5tris]: ./output_images/test2_fit.png "Fit test2"
[image6]: ./output_images/straight1_output.png "Output straight1"
[image6bis]: ./output_images/test1_output.png "Output test1"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./project.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I show how I apply the distortion correction to one of the test images (test1):
![alt text][image2]

And here's the undistorted version:
![alt text][image2bis]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color thresholds (RGB,HLS,GRAY) to generate a binary image (cell #4).  Here's an example of my output for this step, again from 'test1' example.

![alt text][image3]

See cell #5 or output_images folder for further examples.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in cell #7 of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as the mapping matrix.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 63, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 63), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 578, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 703, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I implemented the window search function (cell #11) and fit my lane lines with a 2nd order polynomial. Here's a few examples:

![alt text][image5]   ![alt text][image5bis]   ![alt text][image5tris]

I also implemented an "informed" version of the window search (cell #12), which will mostly be used in the video processing pipeline (see next).

In both implementations, the code is quite robust for missing lines.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In cell #13 of my IPython notebook I implemented the curve radius calculation and the position of the vehicle.
The left and right radii are calculated analitically on the polynomial fits at the bottom of the image. The radius is then considered the average of these two values. The vehicle position is considered to be in the center of the image. The lane center is considered to be the average between left and right lines, again at the bottom of the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In cell #15 I implemented the final processing, that is the lane drawing on the undistorted image. See a couple of examples here:

![alt text][image6]   ![alt text][image6bis]

Curve radius and lateral position are noted at the top left corner of the image. Observe that if the radius is greater than 10000 m, the road is considered to be straight.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

The smoothness of the output has been assured applying a linear moving average on line fits (see cell #17)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most critical part and also the part where more improvements can be introduced is the binarization. Further analysis should be carried on different test images, e.g. taken from the most challenging videos.

For the last video, the interpolation should be limited at a certain distance, otherwise a quadratic polynomial fitting would not be  sufficient.

I am quite satisfied with the linear moving average filtering approach for the basic challenge, but it is possible that the exponentially weighted average would work better for more complex scenarios (e.g. the harder one).
