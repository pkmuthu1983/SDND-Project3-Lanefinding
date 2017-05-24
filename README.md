## **Advanced lane Finding**
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

[image1]: ./output_images/chessboard_image_orig.png "Chessboard - Oriinal"
[image2]: ./output_images/chessboard_image_undistorted.png "Chessboard - Undistorted"
[image3]: ./output_images/road_view_orig.png "Road view - original"
[image4]: ./output_images/road_view_undistorted.png "Road view - undistorted"
[image5]: ./output_images/road_view_thresholded.png "Thresholded binary image"
[image6]: ./output_images/road_view_topview.png "Top view of road"
[image7]: ./output_images/road_view_thresholded_topview.png "Top view of thresholded image"
[image8]: ./output_images/lanefit_sliding_window.png "Sliding window fit"
[image9]: ./output_images/lanefit_extrapolate.png "Extrapolation fit"
[image10]: ./output_images/sliding_window_unwarped.png "Lanes marked on a sample project video image"
[image11]: ./output_images/extrapolate_unwarped.png "Lanes marked on a sample project video image"
[image12]: ./output_images/challenge_video_unwarped.png "Lanes marked on a sample challenge video image"
[video1]: ./project_video_withlanes.mp4 "Project Video"
[video2]: ./challenge_video_withlanes.mp4 "Challenge Video"


### Camera Calibration:
The very first task is to calibrate the camera so we can obtain an undistored view of the road later. Calibration is done using sample chessboard images. Please refer code in cell 2 of ipython notebook 'Pipeline-Final.ipynb'. To calibrate the camera, first read-in the chess board images and find the corners for each image using cv2.findChessboardcorners(). Then, define 'objpoints' as points on a grid (2-D plane). Finally, apply cv2.calibratecamera to transform the distorted chess board corners to undistorted grid points (objpoints) on a plane. This process gives the calibration matrix to get an undistorted view of the road.

The images below shows distorted and undistored view of chessboard.

![Actual chessboard image][image1]
![Undistorted chessboard image][image2]


### Pipeline

#### 1. Distortion correction:
Use the calibration matrix obtained in the above step to correct the distortion. The images shows distorted and undistorted view of road.

![Actual image of road][image3]
![Undistorted image of road][image4]

#### 2. Thresholding:
One of the main tasks of the pipeline is to obtain a binary image that we can use to identify lanes. In order to do this, I combined the gradients of grayscale image along with the gradient of saturation component of the image, to detect edges in the image. The function gradient_threshold() (Pipeline-Final.ipynb, cell 5) performs this task. Appropriate thresholds are also applied to the gradients to retain sharp edges. The figure below shows the thresholded binary image corresponding to the image show above.

![alt text][image5]

#### 3. Perspective transform:
The pipeline uses two matrices for perspective transform (TransM1, TransM2). The matrices are obtained using cv2.getPerspectiveTransform() (refer cell 4). TransM1 is used most of the time by the pipeline. TransM2 is used during initial frames to get an estimate of lane width. TransM2 matrix will provide a top-view of the portion of the road that is very close to the camera, so, we can cleary identify the lanes. TransM1, on the other hand, takes a longer view of the road ahead and gives us the top-view. These two matrices are obtained by manually mapping points on the lane to "estimated" top-view points, using the test images provided. The source and destination corners are shown in the notebook (refer src_corners, and dst_corners).

The images below shows the perspective transform (using TransM1) for the road images shown above (original image as well the thresholded image).

![alt text][image6]
![alt text][image7]


#### 4. Lane idenfitication:
Now, we have the thresholded (and transformed) binary image, which clearly shows lane lines as white pixles. The next task is to find lanes lines from this image. The Pipeline uses two functions -findLanes_slidingwindow() and findLanes_extrapolate() - to fit lines to transformed thresholded binary image. 

- findLanes_slidingwindow() uses sliding window technique starting from the bottom of the image to find non-zero pixels and fits a quadratic function to left and right white pixels. 
- findLanes_extrapolate() extrapolates polynomials fits that are found for previous frames and tries to get a new fit for the current image.

These two functions are defined in cell 9 Pipeline-Final.ipynb. Images below shows the quadratic fit obtained by sliding window and extrapolation.

![alt text][image8]
![alt text][image9]

The Pipelines uses three measures to measure goodness of fit:
0. confidence value - a measure of number of points used to fit. For ex, for the image shown above in the left, the confidence value for left and right lanes are 1 and 0.5 respectively. This is because all boxes in left side are filled with pixels. But, only 6 out 12 boxes are filled.
1. r2 value - this measures mean square error of fit. Sometimes even if we have fewer points to fit, we can get a very good quadratic fit. These fits will have low error.
2. Parallelism - This meaures the degree of parallelism between left and right fits.

** Algorithm **(refer lines 264- 397):
0. Initially, the pipeline uses sliding window technique (findLanes_slidingwindow) to find lanes and stores them in the line objects. Furthermore, the top-view transformation for intial frames consider very close view of camera (TransM2), so we can detect both lanes accurately. During this process, the lane width is also calculated and stored.
1. Later on, the pipeline uses extrapolation (findLanes_extrapolate) to extend previously fitted lines. If these lines are not good, then it tries sliding window again.
2. Once we have polynomial fits for left and right lanes for a given image, the pipeline selects one or both of them using the quality measures discussed earlier. If both fits are good, it uses both. In some cases, only one of the two lanes may be accurately fitted. In this case, the pipeline uses this good fit as reference to calculate the fit for the other lane. The estimated lane width is used to get this fit. In some other cases, we may not have a good fit for both lanes; in this case, it uses average fit for both lanes.

The functions findLanes_slidingwindow/findLanes_extrapolate also calcuate the radius of curvature around lines 100-114 and 220 - 230 of cell 9.

#### Sample Output:

The final ouput from the pipeline looks like images shown below, where the lane area is identified and shaded with green. The two images below are sample frames from project_video.mp4. We can see that the pipeline accurately detects both the lanes during these frames.

![alt text][image10]
![alt text][image11]

The next image is a frame output for the challenge video. For this frame, the pipeline can only detect the right lane accurately. So, it uses the right lane as reference and then estimates the left lane boundary by adding the lane width.

![alt text][image12]

---

### Video output

#### 1. Project video:
The link to video is below. The Pipeline finds good fit most of the time. The estimated radius of curvature is also shown on the video. The left and right radius of curvature is close to 1000m during curves.

[link to video](./project_video_withlanes.mp4)

#### 2. Challenge video:
The link to video is below. The Pipeline finds struggles initially to fit lanes, but does reasonably well later.

[link to video](./challenge_video_withlanes.mp4)

---

### Discussion

There were couple of challenges:
0. First challenge was to get a good thresholded binary image that can accurately show lanes in project video as well as challenge video. The gradient_threshold function did a good job on project video, but not so good on the challenge video. I tried several things here, such as: 1) playing with thresholds, 2) improving image contrast (via histogram equalization), 3) detect yellow/white pixels and convert other pixels in image to black, so that lines are more visible. But none of them gave good performance. So, I used the quality measures to use drop bad fits, while using the good fits to estimate the bad ones.
1. Second challenge was to set suitable thresholds for the quality measures defined in Section 4. I tried various values before choosing them. But, even in this case, the pipeline did not perform very well on challenge video (especially in the beginning). The main issue in challenge video, seems to be the "cracks" near the lanes, which makes it hard to find lanes. I believe, a lot more image processing (such as thresholding, remove shadows) is neeeded.



