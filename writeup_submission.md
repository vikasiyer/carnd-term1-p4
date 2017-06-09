## Writeup Submission

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

[image1]: ./documentationImages/undistortedImage.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./documentationImages/binaryBeforeWarping.png "Binary Example"
[image4]: ./documentationImages/warpedImage.png "Warp Example"
[image5]: ./documentationImages/peaks.png "Fit Visual"
[image7]: ./documentationImages/slidingWindow.png "Output"
[image6]: ./documentationImages/output280.jpg "Output"
[image8]: ./documentationImages/outputShadow.jpg "Shadow Area"
[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third and fourth code cell of the IPython notebook located in "./P4.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I apply distortion correction to one of the test images using cv2.undistort(). Here is an example:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. The code for this is in code cell 9, in method pipeline(). I used HLS color representation and applied thresholding on saturation. I also applied magnitude gradient thresholding.   Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in code cell 7 of the iPython notebook.  The `warp()` function takes as inputs an image (`img`).  After playing with the interactive qt tool, I could get some coordinates for source and destination points. Based on trial and error, I picked some coordinates and hardcoded the source and destination points in the following manner:

```python
src = np.array([
                [570, 460],
                [110, 720],
                [1010, 720],
                [680, 460]]
              ).astype(np.float32)

        dst = np.array([
                [250, 0],
                [250, 720],
                [970, 720],
                [970, 0]]
              ).astype(np.float32)
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 570, 460      | 250, 0        |
| 110, 720      | 250, 720      |
| 1010, 720     | 970, 720      |
| 680, 460      | 970, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this can be found in the code section 9 in the iPython notebook, where I have defined the methods for lane line detection(primarily - findHistogramPeaks, slidingWindow). I found histogram peaks to find lane lines, and then used sliding windows to draw lines.

In certain frames, if the line was not detected, I calculate an average of previous 10 frames and extrapolate a line. The code for this can be found in the slidingWindow() method in code cell 9

![alt text][image5]

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the getRadius() and getVehiclePosition() method in code cell 9 of the iPython notebook. For getRadius, I leveraged the code taught in the classes. For getVehiclePosition(), I got the idea from the class forums to assume the image midpoint as the car axis, and then calculate the mean of the left_fitx and right_fitx to determine the position of the car. The difference between the position of the car and the car axis gives the position of the vehicle with respect to the center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the method plotLinesAndSaveToDisk() method in code cell 12, which in turn calls the plotLinesNew() method in code cell 10 of the iPython notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had trouble getting the right mix of thresholding, but followed the lessons repetitively to arrive at suitable numbers.

I also lost plenty of time understanding how np.dstack works. The lack of understanding caused some delays as the dstacks would have an image dimension of 9 instead of 3 which would not make the code from the lessons work as-is. Reading through the Udacity forums helped resolve some of the issues.

I also have some difficulty in fitting lines in a couple of spots in the video frame, especially the one where there are shadows. I hope to improve this in future by experimenting with some more thresholding values using different color schemes.

*Edit: Submission 2*

Based on the reviewer's recommendation, I made some changes to the thresholding. I used the code suggested by the reviewer to add debug frames to my video to see how lane detection looks like in areas with wobbling. For the first wobble, thresholding addition of H and L values helped solving the problem.

For the shadow patch area, I plotted the sliding window and found that the lane detections found plenty of points in a box and no detections otherwise. I tried making several changes to thresholding in the HLS space. While the wobbling reduced, the green lane detection would still squeeze a bit. Thresholding couldn't make any further improvements. Based on the reviewer's suggestions and the debug plots (see image below), I noticed that the lane width goes too small in the shadow patch. So if I detect a shadow (i.e  "if len(good_right_inds) > 3500" in the slidingWindow method), I look for a wider lane width to record lane detections and ignore every other lane detections. This helped reduce false positives, and helped smoothen the lane detection. 

![alt text][image8]
