# Advanced Lane Finding Project

The goals / steps for this project are:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images. 
* Apply a distortion correction to raw images. 
* Use colour transforms, gradients, etc., to create a threshold binary image. 
* Apply a perspective transform to rectify binary image (“birds-eye-view”).
* Detect lane pixels and fit to find the lane boundary. 
* Determine the curvature of the lane and vehicle position with respect to centre. 
* Warp the detected lane boundaries back into the original image. 
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position. 

[//]: # (Image References)
[image1]: ./readme_images/img1.png
[image2]: ./readme_images/img1.png
[image3]: ./readme_images/img1.png
[image4]: ./readme_images/img1.png
[image5]: ./readme_images/img1.png
[image6]: ./readme_images/img1.png
[image6]: ./readme_images/img1.png

## Camera calibration

1. Computation of camera matrix and distortion coefficients

I looked at the calibration images and noted the dimensions of the grid of the chessboard corners: 9x6. I took one of the images, changed its name to ‘test’ and held it out for subsequent testing. Then I set up an ‘object points’ array with the corresponding dimensions and values. I also created two empty lists to eventually hold the ‘image point’ and ‘object point’ arrays for each calibration image. I looped through the calibration images, performing a `cv2.findChessboardCorners` operation on each of them and storing the respective corner coordinates array in the ‘corner points’ list, along with the ‘object points’ array (which remains the same for each image). 

Having retrieved and put in the lists all the arrays, I ran the camera calibration function: `ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, img_size,None,None)`, which outputs the camera matrix (mtx) and distortion coefficients (dist). I put these data into a dictionary and stored it in a pickle file format for subsequent use and to avoid re-running the calibration in later work. 

2. Example of image un-distortion 

Below are the original and undistorted representations of the test image, after performing the `cv2.undistort(img, mtx, dist, None, mtx)` operation:

![alt text][image1]

The code containing all the steps related to camera calibration and image un-distortion is in the second and third cells of my Jupyter Notebook. 

## Pipeline of un-distortion and color and gradient thresholding (single images) 

1. Below is an example of one of the test images before and after un-distortion:

![alt text][image2]

The code and all the images are provided in my Jupyter NB section titled _Distortion correction applied to each of the 6 test images_. 

2. Combined gradient and color thresholding 

I defined and experimented with a number of functions for gradient and color thresholding. I tried basic thresholding after RGB-to-grayscale conversion; separate Sobel x and Sobel y gradient thresholding; thresholding on total gradient magnitude (square root of the squares of x and y derivatives); gradient direction thresholding; RGB-to-HSL conversion, experimenting with individual HSL color space channels. 

I finally used a combination of Sobel x and S-channel and R-channel thresholding, with the threshold values of sx_thresh=(20, 100), s_thresh=(170, 255), r\_thresh=(230,255), respectively. The example output, is below. It highlights the gradient-identified pixels in green, S-channel identified ones in blue and R-channel ones in pink. As we can see, the R-channel does a good job in identifying the yellow line even in the abruptly changing sunlight conditions: 

![alt text][image3]

The code for the above operations is in my Jupyter NB section titled _Combining un-distortion and color and gradient thresholding_.


3. Perspective transform

I used one of the “strainght\_line” test images for defining a procedure for perspective transform. I read in the image and opened it through matplotlib in an interactive mode. I manually selected the four points of a trapezoid along the lane lines in the image and recorded their coordinates. I also defined destination image coordinates (a simple rectangle) corresponding with the trapezoid coordinates of the source image:

   `src = np.float32([[609, 437], [669, 437], [351, 621], [954, 621]])`
   `dst = np.float32([[300, 0], [1000, 0], [300, 720], [1000, 720]])`
   
Then I extracted the perspective transform parameters: 

    `M = cv2.getPerspectiveTransform(src, dst)`
    
Read in the image, obtained its dimensions, undistorted it and warped it:

    `warped = cv2.warpPerspective(undistorted,M,image_dimensions)`
    
The original and resulting warped image is below. The code is in my Jupyter NB section titled _Perspective transform_.

![alt text][image4]

4. Identifying lane line pixels in the rectified image and fitting a polynomial 

I used a different test image (test3.jpg), with curved lines, for this exercise. The warped version of this image looks as follows:

![alt text][image5]

I used a histogram of pixels, followed by sliding windows search, to programmatically identify lane line pixels. Then I fit polynomial lines. The result is shown below. The respective code is in my Jupyter NB section titled _Identifying and fitting lane line pixels_.

![alt text][image6]

5. Determining the lane line curvature and the position of the vehicle with respect to the center in the lane

For the determination of lane line curvature, I used the formula which calculates the radius of the largest circle which “fits” at the point of interest in the curve. In the case of a second order polynomial, this formula simplifies to R = (1+(2Ay+B)2)3/2 / |2A|. At first, I calculated the curvature values in pixels and later, incorporated conversions to real world distances, assuming lane length of 30 m and lane width of 3.7 m each divided by the respective pixel distances in the image (720 and 700). For the final distance indication, I took the average value of the left and right lane line curvatures. 

For the determination of the position of the vehicle with respect to the lane center, I calculated the absolute value of the difference between the image center (the middle of the x dimension of the image) and the middle point between the lane lines. To convert the pixel distance to real world distance, I used the ratio of 3.7m per 700 pixels. 

The code can be found in my Jupyter NB section titled _Determining the lane line curvature and position of vehicle_.

6. Warping the detected lane boundaries back into the original image

I followed the drawing instructions in the course, reverse-perspective transformed the drawing and superimposed it on the undistorted version of the original image. The result is shown below and the code is in the relevant section of my Jupyter Notebook. The lane curvature and vehicle position are indicated at the top of the image. 

![alt text][image7]

7. Pipeline (video) 

My Jupyter NB’s last section, titled _Video processing pipeline_, contains all the code for processing the project video. The resulting video file is called ouput\_video.mp4 and is included in my submission. I processed the video using the above Sobel X, S-channel and R-channel, as well as L and H channel thresholding. I also used averaging of polynomial fit values over the previous 4 frames and also did a comparison of each subsequent frame lane fits with previous ones and if the difference was larger than a set threshold, I reverted to the previous values. 

8. Reflections 

This was a challenging project. I struggled with certain technicalities related to processing videos. Namely, how to display text containing lane curvature and vehicle position on the video stream. 
As far as the logic of the lane search is concerned, I grasped most of it quite solidly. The lane area in my video tended to flicker and deviate a lot initially, especially, in sunlit areas. Then I did trial and error with gradient and color thresholds and also stored lines fit in previous video frames and performed searches in subsequent frames in close vicinity of the previously identified lines. I then added one more color channel, R-channel, which helped with the detection of the yellow line in a sunlit area. And for further sensitivity, I added L and H channels. I also did averaging and comparison with past values, as described in the section above. I have to say that the addition of the R-channel accounted for biggest improvement in stability. 