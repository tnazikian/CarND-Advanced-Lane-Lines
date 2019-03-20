## Writeup for Advanced Lane Finding Project


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

[image1]: ./output_images/chess.png "Undistorted"
[image2]: ./output_images/normvsundistort.png "Road Transformed"
[image3]: ./output_images/warp.png "Warp example"
[image4]: ./output_images/multiple.png "Multiple Binary"
[image5]: ./output_images/download.png "Lane Area"
[image6]: ./output_images/hist.png "Histogram"
[video1]: ./output_images/out1.mp4 "Output Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

This is my attempt to use various image processing techniques to draw the lanes on a road. All my code is present in 'Advanced-car.ipynb' and will be referenced from there.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

In the first four cells, I created helper functions 'get_points', camera_calibrate' and 'undistort' to calibrate the chessboard images. For finding objpoints and imgpoints I used Udacity's provided code. This was done under the assumption that the z-coordinates of the 3-D object were completely flat. 'camera_calibrate' wrapped the cv2.calibrateCamera function, while discarding the rvecs and tvecs. objpoints and imgpoints were passed in, and the function calculated the camera calibration and distortion coefficients, which were then passed into 'undistort'.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

All of my functions for applying individual thresholds are present in the twelvth code block of 'Advanced-car.ipynb'.
I experimented with individual color threshold channels and Canny-based algorithms and plotted the binary results side-by-side. This allowed me to evaluate the attributes of each one in different situations. I found that, while gradx and grady were good for identifying the shape of lanes under different lighting conditions, they would often only outline them rather than fill them in, which led to confusing histogram shapes. Converting the image into HLS space allowed far greater improvements for the final filter, as it allowed me to use the light channel to spot white lanes, and hue to search specifically for yellow lanes. Saturation was also very useful for identifying lanes, although it was susceptible to picking up shadows from trees. 

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I created a helper function 'warp' that took in an undistorted image and its perspective transform, and returned an image from the top-down perspective in the eighth code cell. 

```python
#For now I am going to use the example src and dst points provided by Udacity
img = plt.imread("../test_images/test1.jpg")
undistorted = undistort(img, mtx, dist)
src = np.float32([[585, 460], [203, 720], [1127, 720], [695, 460]])
dst = np.float32([[320, 0], [320,720], [960, 720], [960, 0]])
M, Minv = mminv(src, dst)
# img_size = (undistorted.shape[1], undistorted.shape[0])
# warped = cv2.warpPerspective(undistorted, M, img_size)
warped = warp(undistorted, M)

f = plot_compare(undistorted, warped, p1='undistorted', p2='warped')
f.savefig('warp.png')
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First, I tested some binary images by plotting them as a histogram with matplotlib:
![alt text][image6]

Once peaks for the left and right lanes could be identified, I used Udacity's provided code in the 22nd code block, where 'find_lane_pixels' and 'fit_polynomial' was copied to. I then modified 'fit_polynomial' to create 'fit_polynomial2', which skipped the visualization and just returned the characteristics of the two lanes, as well as the detected x and y points. I made similar modifications to 'search_around_poly' with 'search_around_poly2', which returns the line characteristics for the left and right lanes, as well as any points that belonged to them. I then defined two helper functions 'create_lines' and 'draw', which helped initialize, as well as draw, the lines onto the image. This was accomplished the same was as in code provided by Udacity, where a mask of the image is created, and cv2.fillPoly is used to draw the lines. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I made a helper function 'measure_curvature_real', which was identical to that provided by Udacity, except for changing the constants 'ym_per_pix' and 'xm_per_pix' based on the radius provided in the assignment prompt. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image4]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

![alt text][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had great difficulty deciding what was the best filter to use for identifying the yellow and white lines while reducing noise. While the six provided test images helped test when the color of the pavement changed, while processing the video the sudden change in color between frames would often result in the lines becoming heavily skewed. Adding better safeguards to make the program more robust to noise (shadows, pavement color changes, etc.) would be to add multiple filters that would factor in different lighting conditions, as well as tests that check whether a lane's curvature is changing between frames too rapidly. If a sudden change in the x-center of the lane or curvature is detected, then either the previous frame's line could be drawn again, or a different filter could be used on that specific frame. However, this would be very cpu intensive and not feasible in a real-time experiment. 
