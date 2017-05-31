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

I applied the same process to correct the images from the dashboard camera like the one shown above in Figure 3.  I wrote a wrapper function `camera_undistort()` in the [AdvancedLaneLines.ipynb Jupyter notebook](https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/AdvancedLaneLines.ipynb) that removed the intrinsic distortion we measured on the chessboards from the dashcam images (assuming the same camera was used in both cases.)  The result is shown in Figure4:

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/undistorted_straight_lines1.png?raw=true"  width=300>

<u><i>Figure 4 Undistorted image from dashboard camera with straight road line markers</u></i>

---

I then wrote a wrapper function `undistort_crop()` which used `cv2.warpPerspective()` to crop the image  for final display purposes so that blank areas were not included.  I did not apply this crop during the undistort stage, because I did not want to remove any information that might be useful in other transformations (e.g. birdseye transformation).

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/crop_undist_straight_lines1.png?raw=true"  width=300>

<u><i>Figure 5 Cropped undistorted image from dashboard camera with straight road line markers</u></i>

---

Using the image in figure 4 I found 4 co-ordinates  which represented a rectangular section of straight roadway.  These co-ordinates are shown with a polyline in figure 6.  I then wrote two wrapper functions `birdseye_transform()` and `birdseye_untransform()` to translate the image into a "birdseye" view from above and back again.  I decided to plot the birdseye image on a 720 x 1280 image to enable better visualization/discrimination of the roadway from above than in the 1280 x 720 format.  I carefully selected co-ordinates to transpose the identified image co-ordinates to in order that the straight road would be centered, would have a lane width of 200 pixels, and would only include minimal .  The source and destination co-ordinates are for the image transform are presented in Table 1 below and are outlined with a polyline in figures 6 (a) and (b)

---

| Source        | Destination   | 
|:-------------:|:-------------:| 
| [617, 450]      | [260, 640]        | 
| [711, 450]      | [460, 640]      |
| [988, 626]     | [460, 1260]      |
| [359, 626]      | [260, 1260]        |

<u><i>Table 1 Source and Destination co-ordinates for the Birdseye image transform</u></i>

---

|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/undistorted_andpoly_straight_lines1.png?raw=true"  width=300>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/birdseye_straight_lines1.png?raw=true"  width=300>|
|--|--|
| (a) | (b) |

<u><i>Figure 6 Birdseye Image Transformation - (a) Dashcam image (undistorted) with yellow polyline outlining source transform points, (b) Transformed birdseye view of road with new co-ordinates shown by yellow polyline</u></i>

---


#### 2. Image Thresholding

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.

---

|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/sobel_mag_straight_lines1.png?raw=true"  width=300>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/combo_thresh_straight_lines1.png?raw=true"  width=300>|
|--|--|
| (a) | (b) |

<u><i>Figure 7 Image thresholding applied to identify lane lines.  In (a) the image was processed using a custom sobel operation and thresholded, producing very good results.  in (b) the results of thresholding multiple color channels and sobel results were used to generate a single binary image with the lane lines extended almost the full length of the image</u></i>

---

sobel_mag_straight_lines1  




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
