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



