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

[image1]: ./calibration/calibration18.jpg "One Checkerbaord Calibaration Image"
[image1a]: ./output_images/calibration18.jpg "Same Checkerboard Calibration Image Undistorted and Transformed"
[image2]: ./output_images/test7.jpg "Road Transformed"
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

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]


```python
best_fit = [  2.99396595e-04  -3.70259762e-01   1.10956875e+03]
```

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
