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

[//]: # (Image References)

[image1]: ./output_images/Calibration.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image2.5]: ./output_images/test1_undistorted.png "Road Undistorted"
[image3]: ./output_images/test1_binary.png "Binary Example"
[image4]: ./output_images/test1_warped.png "Warp Example"
[image4.5]: ./output_images/test1_output.png "Centroids"
[image5]: ./output_images/test1_poly.png "Fit Visual"
[image6]: ./output_images/test1_final.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in `./Advanced-Lane-Finding.ipynb`  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
![alt text][image2.5]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I tried a combination of color and gradient thresholds to generate a binary image. Ultimately I found that a trinary combination of H, S and L created an excellent line contrast (thresholding steps in cell 5 in the notebook).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 60, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 60), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 580, 460      | 320, 0        |
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 700, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the 12th cell of the notebook I begin detecting the lane lines. I decided to use the sliding window technique to establish a collection of candidate lane points. This resulted in the following centroid fits:

![alt][image4.5]

Then in the 14th cell I pass those points to a polynomial fit. The resulting polynomial regressions for image `test1.jpg` are shown below:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 15th cell. First I put in an estimate for the relative x and y pixel dimensions. Then I refit the polygon. Unlike the example, I didn't have a full linspace, but rather my several example points, so I needed to adjust y_eval to be replaced with the max right and left heights. They could as easily be hard coded 720, the image max. Based on the tangential curvature function, we have a curve that is on the oder of 2-3km wide. Considering the poor quality of the right lane line in the test1.jpg image, this should be sufficiently on-target.

The off-center calculation was as simple as finding r-width minus l-width, so `(r-c) - (c-l)` or `r + l - 2c`. Then I convert that pixel offset with the x pixel conversion factor established above.
```txt
>    2194.94847946 m 3605.74113663 m
     drift right:  0.153285714286 m
```
#### 6.Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 16th cell, getting the transform from dst->src as Minv and applying it a polygon of the lane space to generate the following image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

Looking at the output video, my lines are very consistent with the video, save for a few frames which drop on the right side. This is probably due to the broken line on the right: the line is lost for a frame and defaults to the centerline, before recovering in the next. My performance on the harder video was nothing to write home about: I mostly detected the left half alone. The harder challenge, however, presented very interesting (buggy) behavior, and it whips back and forth between some very narrow curvatures, possibly due to the elevation change skewing the perspective of the road.

California Vehicle Code stipulates that in the event of a single left line, drivers are to stay as close to the left as possible. Most of our techniques are dependent on a full 2-line border, and we could put in failsafes to keep to the center line if the shoulder line is gone.

Considering robustness, I could have packaged my code into more helper functions, instead of recopying a large part of my notebook into a giant `processFrame()` function at the end. So much of this project was taken directly from the lessons that I have worked in this way to minimize the extra overhead in nameing functions and passing variables back and forth. I intend to take more authority over the projects I originate.

And in terms of performance, I could use strong detections from prior frames to inform the search region for subsequent frames. I could also add smoothing between frames to cut down on jitter.
