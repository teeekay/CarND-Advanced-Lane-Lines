## **Advanced Lane Finding Project**

### Writeup by Tony Knight - 2017/05/15 
---

insert image here

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/examples/warpedcalibration3.png?raw=true"  width=300>



Figure 1
---


### Camera Calibration

#### 1. Chessboard Images

I calculated the camera matrix and distortion coefficients using the 20 chessboard images provided.  The code is located in the first code cell of the [jupyter notebook](https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/camera_calibration.ipynb).

The first code cell loads the images and attempts to locate the chessboard corners to determine `image points`.  A set of `object points` has been prepared for the chessboard in lines 9 and 10 which corresponds to the 9 X 6 interior corners of the chessboard.  The `cv2.findChessboardCorners()` function was able to detect the corners on 18 of the 20 images.  The code fails on the two images because they do not show the whole chessboard and one or more of the interior corners is outside the image.  Furthermore, when using grayscale, the code could not find the corners on two other images.  I tested equalizing the image, and also using individual channels from HSV, and RGB versions of the image. I found that all the corners could be found when using the blue channel only.  the image and object points were then used by the `cv2.calibrateCamera()` function to generate the camera calibration and distortion coefficients. 

In the second and third cells of the notebook I took the chessboard images, applied the distortion correction for the camera (calculated in cell 1) using `cv2.undistort()`, then attempted to locate the chessboard corners in order to apply a perspective correction obtained from  `cv2.getPerspectiveTransform()'.  

Figure 2 shows a chessboard image as taken, with intrinsic camera distortion removed, and with perspective correction applied.

---


|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/camera_cal/calibration3.jpg?raw=true"  width=300>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/undistortedcalibration3.png?raw=true"  width=300>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/warpedcalibration3.png?raw=true"  width=300>|
|--|--|--|
|(a) | (b) | (c) |

<u><i>Figure 2 (a) Chessboard image, (b) with camera distortion removed, (c) with perspective transform applied</u></i>

---

#### 1. Correcting Distortion on Images from Dashboard Cam

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/test_images/straight_lines1.jpg?raw=true"  width=300>

<u><i>Figure 3 Raw image from dashboard camera with straight road line markers</u></i>

I applied the same process to correct the images from the dashboard camera like the one shown above in Figure 3.  I wrote a wrapper function `camera_undistort()` in the [AdvancedLaneLines.ipynb Jupyter notebook](https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/AdvancedLaneLines.ipynb) that removed the intrinsic distortion we measured on the chessboards from the dashcam images (assuming the same camera was used in both cases.)  The result is shown in Figure4:  I also placed a polyline on the image which represented a section of a lane with straight lines.

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/undistorted_straight_lines1.png?raw=true"  width=300>

<u><i>Figure 4 Undistorted image from dashboard camera with straight road line markers</u></i>

---

I then wrote a wrapper function `undistort_crop()` which cropped the image for display purposes so that blank areas were not included in the final display image.  I did not apply this crop during the undistort stage, because I did not want to remove any information that could be used in other transformations (e.g. birdseye transformation).

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/crop_undist_straight_lines1.png?raw=true"  width=300>

<u><i>Figure 5 Cropped undistorted image from dashboard camera with straight road line markers</u></i>

---





#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
