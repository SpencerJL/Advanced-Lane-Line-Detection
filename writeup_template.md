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

[image1]: ./test_output/chessboard_corners.png "Chessboard"
[image2]: ./test_output/undistort_chessboard.png "Undistort Chessboard"
[image3]: ./test_output/test_and_undistort_image.png "Comparision of original test image and undistort"
[image4]: ./test_output/undistort_and_unwarp_image.png "Comparision of undistort and unwarp image"
[image5]: ./test_output/various_channels_unwarp.png "Coloar Space"
[image6]: ./test_output/unwarp_and_sobel_thresh.png "Sobel threshold"
[image7]: ./test_output/binary_thresholding_pipeline.png "Binary thresholding pipeline"
[image8]: ./test_output/Polyfit_lane.png "Polyfit lans and windows"
[image9]: ./test_output/histogram.png "Histogram of lines"
[image10]: ./test_output/polyfit_with_previous_fit.png "Polyfit with previous fit"
[image11]: ./test_output/draw_lane.png "Lane drawing back onto the images"
[image12]: ./test_output/curve_radius.png "Curve radius"
[video1]: ./test_output/project_video_output.mp4 "Project Video"
[video2]: ./test_output/challenge_video_output.mp4 "Challenge Video"
[video3]: ./test_output/harder_challenge_video_output.mp4 "Harder challenge Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced Lane Line Detector.ipynb". 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

Arrays of object points, corresponding to the location of corners of the chessboard, image points and the pixel locations of chessboard corners determined by function cv2.findChessboardCorners, are delivered to function cv2.calibrateCamera and returns the camera calibration. Then the function undistort from opencv was used with the distortion coefficient obtained from calibration to undistort the image.

The corner drawn onto chessboards are presented below: 

![alt text][image1]

It should be noted that not all of the chessboard images can obtain corner draw as some of the images were unable to detect the desired number of internal corners and left blank on the images.

The image below presents the results of applying undistort, using the calibration and undistort function to one chessboard.

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]
![alt text][image4]
The effect of undistort function is subtle and hard to be detect by eyes.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image5]
![alt text][image6]

I have tried different color space for threshold such as R-G-B,H-S-V as shown in figure and also other color space like HLS and Lab. Ultimately I decided to use just L channel of HLS and B channel of the LAB to isolate the lines especially with yellow color. I slightly tuned the threshold and normalized the maximum values of channels to 255 as the values of lines can vary heavyly depending on lighting conditions Below are the examples of combination of L channel and B channel for different test images.

![alt text][image7]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform , which appears in Perspective Transform section in ipynb file. 
The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  
I chose the hardcode the source and destination points in the following manner:

src = [(575,464),(707,464),(258,682),(1049,682)]
dst = [(450,0),(weight - 450,0),(450,height),(weight - 450,height)]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The function lane_locating detect and polyfit the second order polynomial to left and right lanes which presented in 'Lane line locating and line polyfit section'. At the first, a histogram of the bottom of the image was ploted to obtain the maximum bottom x position at left and right lane lines. From the lecture, the histogram was draw from the local maximum value to the middle point of left and right, I change this from quarter to the midpoint of left and right lines which can help to remove the lanes at the sides of images. Then ten windows were performed to indentify lane pixels, each window centered on the midpoint of the pixels . This increase the processing speed by only searching activate points over an aera of image. Numpy polyfit() function was used to fit a second order polynomial to lanes which shown in below:

![alt text][image8]


The image below shows the histogram generated of the maximum bottom points for the left and right lanes which the two peaks close to the center. This can also proves that the quarter point is necessary to remove the peak at the right side. 

![alt text][image9]

The Polyfit using fit from previous frame section performs the same process with leveraging a previous fitting such as the last frame in video to search the lane pixels within a certain range. The image below shows that the green part is the previous fitting and the yellow part is the current image.

![alt text][image10]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curve was calculated and presented in the section of Measurements of radius of curvature and distance from lane center. In order to obtain the accurate curve radius, the position of car compare to center point was adjusted and calculated in the code. The left_fit and right_fit are the coefficient of second order polynomial.

```
left_fit_x_int = left_fit[0]*h**2 + left_fit[1]*h + left_fit[2]
right_fit_x_int = right_fit[0]*h**2 + right_fit[1]*h + right_fit[2]
lane_center_position = (right_fit_x_int + left_fit_x_int)/2
center_dist = (car_position - lane_center_position)*xm_per_pix
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the section of 'Draw polyfit lanes back onto the original image'. A polygon is generated based on plots of the left and rights fit, warped back to the perspective of the original image the inverse perspective matrix M_inv and overlaid onto the original image which shown below:

![alt text][image11]

This image presents the radius curve and center distance of car.

![alt text][image12]

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
For the project video, the channel B in LAB colorspace was highly effective especially on the part of isolating yellow lines. Even for the challenge video this function performed perfect at the part of light gray road which there was a clearly color difference between the two lane lines. However, as I used the normalized function, there was some noise when the light lane didn't present enough contrast with other parts of image such as the light condition of road such as the glass reflection effect on the road in sunny day or dark shadows on the lanes. 
I still got some problems with hard challenge video as the lane lines were extremly hard to detect in this video. This might because the lighting conditions and shadow or discoloration which occured in hard challenge video. I guess more dynamic thresholding combinations such as RGB or HSV might help or thresholding parameters for different horizontal slices of the image.

[Challenge video](./test_output/challenge_video_output.mp4)

[Harder challenge video](./test_output/harder_challenge_video_output.mp4)