## Writeup for Advanced Lane Finding Project

### Brian Chesney
### December 21, 2018
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

[image1]: ./camaera_cal/calibration18.jpg "One Checkerbaord Calibaration Image"
[image1a]: ./output_images/calibration18.jpg "Same Checkerboard Calibration Image Undistorted and Transformed"
[image2]: ./output_images/test6.jpg "Road Transformed"
[image3]: ./output_images/binary_warped.jpg "Binary Example"
[image4]: ./test_images/straight_lines1.jpg "UnWarped Input"
[image4a]: ./output_images/output_imagesstraight_lines1.jpg "Warp Output"
[image5]: ./output_images/poly_fit_overlay.jpg "Fit Visual"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

You're reading it!  All the code for this project is in the ./P2.ipynb IPython notebook.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image1a]

I saved the resulting 3x3 camera calibration matrix, mtx, and the distortion parameters, dist, in an object called lane_boundaries_pipeline.  Here are example values for mtx and dist computed from the checkerboard images provided with the project:

```python
mtx= [[  1.15662906e+03   0.00000000e+00   6.69041437e+02]
 [  0.00000000e+00   1.15169194e+03   3.88137239e+02]
 [  0.00000000e+00   0.00000000e+00   1.00000000e+00]]
dist= [[-0.2315715  -0.12000538 -0.00118338  0.00023305  0.15641572]]
```


### Pipeline (single images)

The lane_boundaries_pipeline object has most of the code for the project.  The constructor for this object calibrates the camera from the calibration images and also obtains the perspective transform from a straight_lines image in the examples directory.  The lane_boundaries_pipeline object also contains Line object for the left and right lane lines to compute and store the best fit polynomials representing the left and right lane lines.  The __call__ method of the lane_boundaries_pipeline object processes an input image abd returns the same image with three things annotated on it:

1. Green polygon indicating the lane
2. Radius of curvature of the lane (in m).
3. Offset from the center of the lane (in m).


#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to eventually create an image like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `lane_unwarp()`, which appears in section 1.2 of the notebook.  I choose to parameterize the src and dst points based on the image size.

```python
Y, X, Z = img.shape
src = np.float32([[(X//8, Y), (X//2-50, 5*Y//8), (X//2+50, 5*Y//8), (7*X//8, Y)]])
dst = np.float32([[(X//4, Y), (X//4,    0),      (3*X//4,  0),      (3*X//4, Y)]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 160, 720      | 320, 720      | 
| 590, 450      | 320, 0        |
| 690, 450      | 960, 0        |
| 1120, 720     | 960, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image4a]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the lane search from prior method for finding the lane lines as presented in the Advanced Computer Vision lesson.  I kept track of the best fit polynomial for the found pixels, as well as the best fit polynomial for the pixels mapped to meters, separately in the Line object.  For each fit, I averaged over the last 8 fits.

Below is an example fit (in pixels):

```python
best_fit = [  2.99396595e-04  -3.70259762e-01   1.10956875e+03]
```


Fitst there is a find_lane_pixels() function which is called when a fit hasn't been made yet.  Once a fit is made, the search_around_prior looks around a window of that polynomial and asks for a slightly different fit.

![alt text][image5]



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature in the measure_curvature() function in the Line object in section 2.0 of the notebook.
I took the fit in meters, used the max y in the stored y points and applied the formula from the lesson on radius of curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The annotate_image() function in section 4.1 does the actual annotation on the image.  As stated earlier, there are three things annotated on to the image:

1. Green polygon indicating the lane
2. Radius of curvature of the lane (in m).
3. Offset from the center of the lane (in m).

Here is an example of my result on a test image:

![alt text][image2]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Finding the src and dst points was one of the most frustrating points of the project.  As summarized in this post in the student hub, referencing the src/dst points from the writeup_template.md file:

take the src/dst points above and replace (585, 460) with (585, 400) and (695, 460) with (695, 400) and apply the transform and you'll get an image that isn't anywhere close to a top-down perspective. The order of the points isn't wrong, and the shape isn't wrong; but it wasn't immediately intuitive to me that all I had to do was move the top line of the trapezoid on the src image down by 60 pixels to go from completely unusable as a top-down perspective to spot on. I wasn't expecting that amount of sensitivity in the transform function - again, no change to the shape or order of the src points and no change to any of the dst points.

I would like to have implemented the convolution method for window search, but did not have time for it.  I would also have liked to spend time on the challenge videos, but had a difficult time taking frames from them to debug the pipeline.  Could someone give a code snippet that grabs one (or several) frames from a video to debug a pipeline - that would be really helpful and is apprently beyond me now!
