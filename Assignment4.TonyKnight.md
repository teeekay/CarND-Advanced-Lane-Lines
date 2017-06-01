## **Advanced Lane Finding Project**

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/video_snap1.png?raw=true"  width=800>

<i><u>Figure 1 Snapshot of Lanefinder Video</u></i>

---


### Writeup by Tony Knight - 2017/05/15 


### Camera Calibration

#### 1. Chessboard Images

I calculated the camera matrix and distortion coefficients using the 20 chessboard images provided.  The code is located in the first code cell of the [camera calibration notebook](https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/camera_calibration.ipynb).

The first code cell loads the images and attempts to locate the chessboard corners to determine `image points`.  A set of `object points` has been prepared for the chessboard in lines 9 and 10 which corresponds to the 9 X 6 interior corners of the chessboard.  The `cv2.findChessboardCorners()` function was able to detect the corners on 18 of the 20 images.  The code fails on the two images because they do not show the whole chessboard and one or more of the interior corners is outside the image.  Furthermore, when using grayscale, the code could not find the corners on two other images.  I tested equalizing the image, and also using individual channels from HSV, and RGB versions of the image. I found that all the corners could be found when using the blue channel only.  The image and object points were then used by the `cv2.calibrateCamera()` function to generate the camera calibration and distortion coefficients. 

In the second and third cells of the notebook I took the chessboard images, applied the distortion correction for the camera (calculated in cell 1) using `cv2.undistort()`, then attempted to locate the chessboard corners in order to apply a perspective correction obtained from  `cv2.getPerspectiveTransform()`.  

Figure 2 shows a chessboard image as taken, with intrinsic camera distortion removed, and with perspective correction applied.

---


|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/camera_cal/calibration3.jpg?raw=true"  width=300>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/undistortedcalibration3.png?raw=true"  width=300>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/warpedcalibration3.png?raw=true"  width=300>|
|--|--|--|
|(a) | (b) | (c) |

<b><i>Figure 2 (a) Chessboard image, (b) with camera distortion removed, (c) with perspective transform applied</i></b>

---

#### 1. Correcting Distortion on Images from Dashboard Cam

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/test_images/straight_lines1.jpg?raw=true"  width=300>

<u><i>Figure 3 Raw image from dashboard camera with straight road line markers</u></i>

I applied the same process to correct the images from the dashboard camera like the one shown above in Figure 3.  I wrote a wrapper function `camera_undistort()` (code cell 1 in [AdvancedLaneLines notebook](https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/AdvancedLaneLines.ipynb)) that removed the intrinsic distortion we measured on the chessboards from the dashcam images (assuming the same camera was used in both cases.)  The result is shown in Figure4:

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/undistorted_straight_lines1.png?raw=true"  width=300>

<u><i>Figure 4 Undistorted image from dashboard camera with straight road line markers</u></i>

---

I then wrote a wrapper function `undistort_crop()` (code cell 2) which used `cv2.warpPerspective()` to crop the image  for final display purposes so that blank areas were not included.  I did not apply this crop during the undistort stage, because I did not want to remove any information that might be useful in other transformations (e.g. birdseye transformation).

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/crop_undist_straight_lines1.png?raw=true"  width=300>

<u><i>Figure 5 Cropped undistorted image from dashboard camera with straight road line markers</u></i>

---

Using the image in figure 4 I found 4 co-ordinates  which represented a rectangular section of straight roadway.  These co-ordinates are shown with a polyline in figure 6(a).  I then wrote two wrapper functions `birdseye_transform()` and `birdseye_untransform()` (code cell 3) to translate the image into a "birdseye" view from above and back again.  I decided to plot the birdseye image on a 720 x 1280 image to enable better visualization/discrimination of the roadway from above than in the 1280 x 720 format.  I carefully selected co-ordinates to transpose the identified image co-ordinates to in order that the straight road would be centered, would have a lane width of 200 pixels, and would only include minimal .  The source and destination co-ordinates are for the image transform are presented in Table 1 below and are outlined with a polyline in figures 6 (a) and (b)

---

| Source        | Destination   | 
|:-------------:|:-------------:| 
| [617, 450]      | [260, 640]        | 
| [711, 450]      | [460, 640]      |
| [988, 626]     | [460, 1260]      |
| [359, 626]      | [260, 1260]        |

<b><i>Table 1 Source and Destination co-ordinates for the Birdseye image transform</i></b>

---

|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/undistorted_andpoly_straight_lines1.png?raw=true"  width=500>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/birdseye_straight_lines1.png?raw=true"  width=300>|
|--|--|
| (a) | (b) |

<b><i>Figure 6 Birdseye Image Transformation - (a) Dashcam image (undistorted) with yellow polyline outlining source transform points, (b) Transformed birdseye view of road with new co-ordinates shown by yellow polyline</i></b>

---


#### 2. Image Thresholding

I used a combination of color and gradient thresholds to generate binary images to locate the lane lines.  Examples of the outputs of this step are presented in Figure 7.

---

|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/sobel_mag_straight_lines1.png?raw=true"  width=200>|<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/combo_thresh_straight_lines1.png?raw=true"  width=200> | <img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/combo_thresh_conv_straight_lines1.png?raw=true"  width=200>|
|--|--|--|
| (a) | (b) | (c)

<b><i>Figure 7 Image thresholding applied to identify lane lines.  In (a) the image was processed using a custom sobel operation and thresholded, producing very good results.  in (b) the results of thresholding multiple color channels and sobel results were used to generate a single binary image with the lane lines extended almost the full length of the image (c) Thresholded Image filtered for noise and to match lane marker width of 10 pixels </i></b>

---

The thresholds were tested and developed in the [Image_Thresholding_Tests](./Image_thresholding_Tests.ipynb) notebook.

The thresholds were applied in the `line_pipeline()` function (Code cell 6)  in the [AdvancedLaneLines](https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/AdvancedLaneLines.ipynb) Notebook. I used combinations of thresholds using BGR, HSV, HLS, YCrCb colorspaces and the sobel responses from the V channel of the HSV colorspace.

I wrote a wrapper function `sobel_thresh()` around the 'cv2.Sobel()' function (code cell 4) to enable thresholding of results when using sobels in x direction, y direction, or when using magnitude of sobels or angle of sobel response.  After reading [Lane Estimation for Autonomous Vehicles
using Vision and LIDAR](http://rvsn.csail.mit.edu/Pubs/phd_ashuang_2010feb_laneestimation.pdf) I created a second specialized function `sobel_grad()` (code cell 4 lines 60-67) which used a length 5 1st derivative sobel in the x direction, and a length 3 second derivative filter in the y direction (which should be close to zero if parallel to lane lines).  I did not find a way to use the sobel angle, as it was generally too noisy.

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

I also added two convolutions after the thresholding.  The first is with a 3x3 filter to attempt to remove noise, and the second is a convolution with a custom filter (shown below) designed to respond to lane lines with a width of 10 pixels, and to remove any large patches erroneously detected by the thresholding.

```
	           [[0,-1,1,0,0,0,0,0,0,0,0,1,-1,0],
                [0,-1,1,0,0,0,0,0,0,0,0,1,-1,0],
                [0,-1,1,0,0,0,0,0,0,0,0,1,-1,0],
                [0,-1,1,0,0,0,0,0,0,0,0,1,-1,0],
                [0,-1,1,0,0,0,0,0,0,0,0,1,-1,0]]
```

The final binary threshold image generally identified the lane lines with a minimum of noise in the project video.  

#### 4. Mapping Lines to Binary image

The binary thresholded birdseye is sent to the `find_lines()` function (code cell 9) which attempts to fit 2nd order polynomials to the right and left lane markers using the numpy `np.polyfit()` functions.  

The birdseye image is initially flipped using `cv2.flip()` so that the y co-ordinates are zero at the front of the car and increase with distance along the road.  In this way the 3rd value 'C' of the polynomial equation we are trying to fit will represent the point where the lane lines intersect the front of the car.  For the project video  we know that the lines are generally located near x=260 and x=460 at the front of the car, so this will allow a quick check of the validity of the polynomial fit.

```
f(x) = A*x**2 + B*x + C`

where A = p[0], B=p[1], and c= p[2] in p = numpy.polyfit(x,y,2)
```

For the first iteration (when the location of the lines is not yet known) the non zero pixels are summed within the bottom 1/10th of the the binary image along the y-axis to provide a histogram of non zero pixels across the x axis (lines 39-49). An example of the histogram is shown on figure 8.  By picking the peak value in either half of the histogram, we can identify likely candidate location to start looking for pixels belonging to each lane line.

---

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/histogram.png?raw=true"  width=500>

<b><i>Figure 8 histogram of non-zero pixels across the x-axis used to locate the bottom of the lane lines</i></b>

---


A set of windows are used to progressively locate non-zero pixels likely associated with the lane lines moving up the image.  The x range of each set of windows can be moved if more than 45 pixels were identified in the previous window, in which case, the window is re-centered to the mean of the previous window (lines 63-91).

Based on tests with the video, I reduced the height that was scanned for lane lines to the bottom 2/3 of the birdseye image.  Beyond this point the binary image data was often too sparse and noisy and resulted in poor results.

The location of all the nonzero pixels are stored in a set of numpy arrays, and `np.polyfit()` is used to attempt to fit a 2 degree polynomial to the points (lines 113,114).

In the event that the lines were detected in a previous frame, the location of all non-zero pixels within 40 pixels of the previous polyfit line are used as input to np.polyfit() (lines 93-99)

At this point the algorithm attempts to check to see if the values of the third parameter of the polyfit match with where we expect the lane lines to be, and if the lines are approximately 200 pixels apart at this location. In the case that one of the lines looks incorrect, but the other seems valid, it will attempt to use the valid fit offset by 200 pixels for the incorrect line.  The algorithm averages the polyfit parameters over 4 measurements (within the `data_storage.save_fit()` method)

The x,y coordinates of the lane lines are then calculated using the polyfit parameters over the area of the search, and these co-ordinates are used to create images of the lane as a filled polygon with 'cv2.fillPoly()' (lines 147,148 and 189-191).  This image has to be flipped back to the correct orientation, and then retransformed to the original perspective using the `birdseye_untransform()` function.  It is then returned to the 'feed_the_beast()' function (code cell 8) where it is overlaid on the undistorted dashcam image and cropped.


#### Displacement from Center of the Lane

The offset of the car from the center of the lane was calculated within the `find_lines()` function (code cell 9 - lines 151 to 153).  The location of the center of the identified lines is calculate in pixel space on line 151 by averaging the left and right lane locations at y=20 in flipped co-ordinate space (y=1260 in birdseye image space) which is about the front of the hood of the car in the image.  The center of the car is assumed to be at x=360 (The center of the 720 pixel wide image).  The offset of the car is then easily calculated by multiplying the difference in these two locations by 3.7m (standard width of lanes) /200 pixels (measured pixel width of lanes in birdseye view).

####  Radius of Curvature

I calculated a radius of curvature for each image in the `find_radius()` function (Code cell 11).  

The identified lane_line pixels for left and right lines calculated from the polyfit parameters are passed into the function as arrays of the x co-ordinates( leftx and rightx )and the matching y co-ordinates (ploty).  These are passed into the np.polyfit() function while being scaled to ground co-ordinates in x and y directions.

The curvature of radius is calculated for each lane line at y=20 which represents the front of the car on lines 13 and 14.

The curvature displayed in the video is calculated by using an average of the measurements from 8 frames with the left and right lane curvatures averaged together.  See `data_storage.add_curvature()` (code cell 7 lines 42-46) and feed_the_beast() (Code cell 8 line 30). 

#### Speed

The speed of the car is estimated in the `estimate_speed()` function (code cell 10).  The function uses successive threshold binary birdseye images.  The most recent image is cropped between y=1000 and 1200 and x = 200 and 520 which is likely to contain only the lane lines.  The older frame is cropped between y=900 and 1200 and x=200 and 520.  The pixel count along the Y axis is calculated for each of these in lines 24 and 32 to create histogram-like profiles.  Least square differences are then calculated between the two profiles as they are slid past each other to find the location where they match the best (lines 40-44).

This displacement in pixels is added over several frames and  multiplied by the meters to pixels scaling factor (line 53).  This result is then multiplied by 3600/1000 and the (frame rate/number of frames) to generate a result in km/hr.

I had attempted to use cross correlation between successive images, but ran into problems that larger results could be encountered when correlating with a similar signal with a larger magnitude (which is often the case with similar sized lane lines with different intensities.  I then switched to the easier least square difference method which does not match well with similar shapes with different magnitudes.

#### Output

The results of the lanefinding algorithms are plotted onto the cropped and undistorted dashcam video as seen in figures 1 and 9.

<img src="https://github.com/teeekay/CarND-Advanced-Lane-Lines/blob/master/output_images/video_snap2.png?raw=true"  width=800>

<i><b>Figure 9 Snapshot of Lanefinder Video</b></i>

---

Here is a link to the [video](./project_video_output.mp4) generated from the notebook.

---

### Discussion

This program was able to deal with most of the situations encountered in the project video, using averaging and occasional corrections to compensate for some weaknesses.  However, it is not robust and requires a lot more work to be able to work on the challenge and harder challenge videos.

I think the primary issue is that the color thresholding I have used is not reliable in many situations, including when there is shadow, or low contrast between the road color and lane lines.

The line matching algorithm also needs to be optimized to bolster situations when there is little information in the existing frame.  In some situations this could potentially be improved by using image data from earlier frames to look back at the pixel distribution behind the front of the car, and/or overlaying multiple frames offset by the calculated displacement to generate a stronger signal.

It might also be interesting to look at using correlation of specific shapes to identify lane markings e.g. - 3 m long dashes, 10 cm x 10 cm raised pavement markers, etc.