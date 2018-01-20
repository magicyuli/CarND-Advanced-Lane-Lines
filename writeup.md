## Writeup Template

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

[checkboard_undistort]: ./output_images/checkboard_undistort.png "Chessboard Undistorted"
[undistorted]: ./output_images/undistorted.png "Undistorted"
[edge_detect]: ./output_images/edge_detect.png "Road Transformed"
[warped]: ./output_images/warped.png "Warp Example"
[warped_edge]: ./output_images/warped_edge.png "Warp Example"
[detected_lane_area]: ./output_images/detected_lane_area.png "Fit Visual"
[result]: ./output_images/result.png "Output"
[video1]: ./project_video_out.mp4 "Video1"
[video2]: ./challenge_video_out.mp4 "Video2"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd code cell of the IPython notebook located in "./pipeline.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `obj_point` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the chessboard images, and test images using the `cv2.undistort()` function and obtained the following results: 
![alt text][checkboard_undistort]
![alt text][undistorted]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][undistorted]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of white and yellow thresholds to generate a binary image (thresholding steps at cells 8th to 9th in `pipeline.ipynb`). Here's an example of my output for this step (from left to right: white output, yellow output, combine output).

![alt text][edge_detect]

I used threshold [200, 200, 200] to [255, 255, 255] on RGB images to isolate white lanes, and [200, 100, 0] to [255, 255, 150] on RGB images to isolate yellow lanes.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in cell 7th in the file `pipeline.ipynb`.  The `warp()` function takes as inputs an image (`img`), and computes the warped image with precomputed perspetive transform matrix `M`.  I chose the hardcode the source and destination points in the following manner:

```python
orig_vertices = np.array([(492, 480), (788, 480), (img_w, 690), (0, 690)], dtype=np.float32)
trans_vertices = np.array([(0, 0), (img_w, 0), (img_w, img_h), (0, img_h)], dtype=np.float32)
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 492, 480      | 0, 0        | 
| 788, 480      | 1280, 0      |
| 1280, 690     | 1280, 720      |
| 0, 690        | 0, 720        |

I verified that my perspective transform was working as expected by drawing the `orig_vertices` and `trans_vertices` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][warped]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Given an image, I process half of the image at a time (code cell 11) to reduce complexity and reuse code.

With each half of an image, first I use histogram to identify the window (size 100x72) where most non-zero pixels lie, and use the center of the window as the start centroid, then perform moving window search towards the top and bottom of the image from the start centroid, and collect all pixels that are within the windows. These logics are in code cell 11.

Next, I fit the pixels collected to a quadratic function with `np.polyfit`, and calcualte the curvature of the line (code cell 12). With two fitted lines, I draw the lane area with `cv2.fillPoly` (code cell 13).

For video, I discard a fitted line when it's too far from the fitted line from the last frame and just use the line from the last frame. I also average the line with previous frames by `0.2 * new_fit + 0.8 * previous_fit`, so every line is essentially a weighted rolling average of previous lines. These logics are in code cell 14.

The output is shown below
![alt text][detected_lane_area]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code cell 12 in my code in `pipeline.ipynb`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 16 in my code in `pipeline.ipynb` in the function `warpBackAndOverlay`.  Here is an example of my result on a test image:

![alt text][result]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's ![my project video result][video1], and ![my challenge video result][video2].
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

One problem is with a fixed perspective transform matrix, it doesn't work so well with sharp curves, due to the shortened road portion in front of the car. Another problem is with edge detection. Since the pixels are just picked by color thresholds, any similar colored noises around the lane area will influence the accuracy of the detection.

My pipeline is likely to fail on sharp curves.

I probably can dynamically detect the road portion, and use a dynamic perspective transform.
