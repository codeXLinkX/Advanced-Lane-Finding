
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

[image1]: ./output_images/undist0.png "Undistorted"
[image2]: ./output_images/undist1.png "Undistorted"
[image3]: ./output_images/undistorted1.png "Binary Example"
[image4]: ./output_images/gradient_trans.png "Warp Example"
[image5]: ./output_images/r_and_b.png "Fit Visual"
[image6]: ./output_images/hls.png "Output"
[image7]: ./output_images/rgb_s.png "Output"
[image8]: ./output_images/luv.png "Output"
[image9]: ./output_images/all_combined.png "Output"
[image10]: ./output_images/points.png "Output"
[image11]: ./output_images/warped.png "Output"
[image12]: ./output_images/warped2.png "Output"
[window]: ./output_images/windowed.png "Output"
[image_last]: ./output_images/lanes.png "Output"
[histogram]: ./output_images/histogram.png "Output"
[fit]: ./output_images/fitted.png "Output"
[video1]: ./project_video.mp4 "Video"


---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function.

| ![image3][image1] |
|:--:| |:--:| |:--:|
| ![image3][image2] |

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

You can see the application of distortion correction to one of the test images like this one:

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The pipeline to create a final transformed image is in the function `gradient_threshold()`, which is later called by the main pipeline method `process_image()`.

I extensively tried many combinations of gradient and color transforms.

Here is the final gradient transform:

| ![image3][image4] |
|:--:| |:--:| |:--:|
| *Combination of SobelX, Magnitude and Directional Transforms* |

Next I have the combination of R and B channels of RGB image with custom thresholds:

| ![image3][image5] |
|:--:| |:--:| |:--:|
| *R and B channels* |

Then I worked on HLS images:

| ![image3][image6] |
|:--:| |:--:| |:--:|
| *H, L and S channels* |

S channel is the most helpful one, so combine it with RGB:

| ![image3][image7] |
|:--:| |:--:| |:--:|
| *Combination of RGB and S channels* |

Still this combination doesn't show our lines fully. So I played with the LUV channels as well:

| ![image3][image8] |
|:--:| |:--:| |:--:|
| *Combination of L, U and V channels* |

Final combination:

| ![image3][image9] |
|:--:| |:--:| |:--:|
| *Final combination* |

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`. The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

src = np.float32([[700, 450],
                  [1150, 720],
                  [200, 720],
                  [590, 450]])

dst = np.float32([[1150, 0],
                  [1150, 720],
                  [300, 720],
                  [300, 0]])

I didn't use the provided variables below:
```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image:

![alt text][image10]

![alt text][image11]

Final warped image with transformation looks like this:

![alt text][image12]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the function `findFit()`, which gets a warped image as input, I found the starting points of left and right lanes using a histogram. Here you can see the peak points in the histogram:

![alt text][histogram]

Then I used sliding windows (10 of them) to find the pixels that create the lines. When the windows provided the points, using `np.polyfit` I drew the lines:

![alt text][window]

Once the initial window search is done, for the next frames we skip this step to accelerate the process. You can see here a new fit based on the previous lane points:

![alt text][fit]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated in the `findCurv()` function. First I define conversions in x and y from pixels space to meters, then fit new polynomials to x,y in world space using `np.polyfit`. Finally I calculate the new radii of curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I tested my pipeline on the test images and here are the results:

![image3][image_last]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

To handle the case when I don't find any lane lines, I implemented a Lines() class where I am keeping the last 10 found lane lines and using their mean as my confident lanes.
