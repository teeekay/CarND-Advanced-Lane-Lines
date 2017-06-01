## **Advanced Lane Finding Project**

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/video_snap.png?raw=true"  width=800>

<i><u>Figure 1 Snapshot of Lanefinder Video</u></i>

---


### Writeup by Tony Knight - 2017/05/15 


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

I used a combination of color and gradient thresholds to generate binary images to locate the lane lines.  Here's an example of outputs for this step.

---

|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/sobel_mag_straight_lines1.png?raw=true"  width=300>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/combo_thresh_straight_lines1.png?raw=true"  width=300>|
|--|--|
| (a) | (b) |

<u><i>Figure 7 Image thresholding applied to identify lane lines.  In (a) the image was processed using a custom sobel operation and thresholded, producing very good results.  in (b) the results of thresholding multiple color channels and sobel results were used to generate a single binary image with the lane lines extended almost the full length of the image</u></i>

---

The thresholds were applied in the `line_pipeline()` function in the `AdvancedLaneLines.ipynb` Notebook. I used combinations of thresholds using BGR, HSV, HLS, YCrCb colorspaces and the sobel responses from the V channel of the HSV colorspace.

I wrote a wrapper function `sobel_thresh()` around the 'cv2.Sobel()' function to enable thresholding of results when using sobels in x direction, y direction, or when using magnitude of sobels or angle of sobel response.  After reading http://rvsn.csail.mit.edu/Pubs/phd_ashuang_2010feb_laneestimation.pdf I created a second specialized function which used a length 5 1st derivative sobel in the x direction, and a length 3 second derivative filter in the y direction (which should be close to zero if parallel to lane lines).  I did not find a way to used the sobel angle, as it was generally too noisy.

The color channel thresholds generally worked by translating the image to a specific colorspace using the `cv2.cvtColor()` function, and then separating and thresholding each image.  I worked through the test images trying to determine the appropriate thresholds for each channel (where found to be useful).  I selected a combination of the following thresholds shown in Table 2 (using 8 bit integer space for color channels) .

|parameter | criteria|
|:----|:----:|
| h from HSV |h>= 19 & h <=24 |
| H, S, L from HSL | (17 < H < 45) and (L > 140) and (S > 80 while H <180) |
| L from HSL | L > 220 |
| Y from YCrCb |Y > 200 |
| CR from YCrCb |142 > Cr < 170 |
| Cb from YCrCb |30 > CB < 110 |
| R, G, B from BGR | R > 225 and G > 180 and B < 170 |
| sobel_mag | sobel_mag > 1750 |

<b><i> Table 2: Criteria for thresholding Images </i></b>




#### 4.

After the birdseye road image has been thresholded, the binary image is sent to the `find_lines()` function which attempts to fit 2nd order polynomials to the right and left lane markers.  

The birdseye image is flipped using `cv2.flip()` so that the y co-ordinates are zero at the front of the car and increase with distance forwards.  In this way the 3rd value 'C' of the polynomial equation we are trying to fit will represent the point the lines intersect the front of the car.  For the project video  we know that the lines are generally located near x=260 and x=460 at the front of the car, so this will allow a quick check of the validity of the polynomial fit.
Y = Ax^2 + Bx + C

If we are starting fresh, the algorithm sums the non zero pixels along the bottom 1/10th of the the binary image on the y-axis to provide a histogram of non zero pixels across the x axis. An example is shown on figure 8.  This allows us to position search windows to look for pixel

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/histogram.png?raw=true"  width=300>

<u><i>Figure 8 histogram of non-zero pixels across the x-axis used to locate the bottom of the lane lines</u></i>

---



![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. 
Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
