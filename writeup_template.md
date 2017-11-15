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
[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./examples/undistort_road.png "Road Transformed"
[image3]: ./examples/binary_combo_example.png "Binary Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "AdvancedLaneFinding.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Below is an exmple of original test image and distortion-corrected image of the same by using the distortion coefficients computed in earlier step.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

1. First I started with creating functions for doing different image transforms that were described in the lecture videos in each code cell from 4-9
2. In the code cell [12] test_pipeline function, I experimented with combination of using different color spaces, thresholds and other techniques. I used the visualize method to see the results of different experiments.
3. Once I was satisified with my results for all the images in test_images, I used that combination in my pipeline function in code cell [11]

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 10th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`) and returns warped image.  
I tried different src and dst points and finally found the below points to give good results. I found them from the udacity forums.

This resulted in the following source and destination points:
src = np.float32([[200. , 720, ],[453. , 547.],[835. , 547. ],[1100. , 720. ]])

dst = np.float32([[300., 720],[300. , 590.4],[900., 590.4],[900. , 720]])



| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 300, 720        | 
| 453, 547      | 300, 590.4      |
| 835, 547     | 900, 590.4      |
| 1100, 720      | 900, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

1. I first take a histogram along all the columns in the lower half of the image
2. With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines.
3. From that point, I used sliding window approach explained in lecture videos to find the lines up to the top of the frame.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

1. I implemented getCurvature function in code cell [18]. I used the sample code provided in the lecture videos to compute the curvature.
2. I implemented getCarPosition function in code cell [19]. I used the following forum link to implement the function.

https://discussions.udacity.com/t/what-the-vehicle-position-with-respect-to-center-means/375281/4

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. I started with using different color spaces and thresholding techniques to get clear output with left and right lines.
2. It was lot of trial and error by viusalizing output of different combinations. Each combination has its own pros and cons. I tried various combinations and read through forums to see what other people are trying.
3. Finally I settled on the combination of using HSV color space S and V layers and LUV color space L layer which gave me best results. It worked pretty good in all scenarios for both left and right lanes.
4. Then I used the approach described in lectures to detect left and right lines, find curvatures.
5. I tried my code in challenge_video and I noticed that it isn't working well on that video because it had additional edge line in the middle of road which I wasn't taking care of.
6. I can imagine there can be many scenarios which were not considered in my implementation like road elevations, vehicle obstructions, construciton zones, etc.

Post Feedback:
I updated my pipeline to use the below guidelines and it worked even better.

def select_yellow(image):
  hsv = cv2.cvtColor(image, cv2.COLOR_RGB2HSV)
  lower = np.array([20,60,60])
  upper = np.array([38,174, 250])
  mask = cv2.inRange(hsv, lower, upper)
  return mask
  
and

def select_white(image):
  lower = np.array([202,202,202])
  upper = np.array([255,255,255])
  mask = cv2.inRange(image, lower, upper)
  return mask

and

def comb_thresh(image):
  yellow = select_yellow(image)
  white = select_white(image)
  combined_binary = np.zeros_like(yellow)
  combined_binary[(yellow >= 1) | (white >= 1)] = 1
  return combined_binary
