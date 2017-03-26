# Advanced Lane Finding Project

Video output: https://www.youtube.com/watch?v=89Ck3d_JCMw

Udacity Self-Driving Car Nanodegree

The goals / steps of this project are the following:

1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Apply a distortion correction to raw images.
3. Use color transforms, gradients, etc., to create a thresholded binary image.
4. Apply a perspective transform to rectify binary image ("birds-eye view").
5. Detect lane pixels and fit to find the lane boundary.
6. Determine the curvature of the lane and vehicle position with respect to center.
7. Warp the detected lane boundaries back onto the original image.
8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Part 1: Camera Calibration

I used chessboard image calibration3.jpg to do the undistortion process. Here are the steps:
1. Prepare object points using variable objp
2. Create objpoints to store 3d points, create imgpoints to store image 2d points. 
3. Use cv2.findChessboardCorners and cv2.calibrateCamera functions to undistort images. 

Here is the original chessboard image:

![screen shot 2017-03-12 at 2 09 14 pm](https://cloud.githubusercontent.com/assets/11469505/23835959/96331496-072d-11e7-9cba-826e5953f4b5.png)

Here is the undistorted chessboard image: 
![screen shot 2017-03-12 at 2 09 22 pm](https://cloud.githubusercontent.com/assets/11469505/23835963/992d6a0c-072d-11e7-90ed-7d9e1e45721a.png)

Here is the original road image

![screen shot 2017-03-12 at 2 11 11 pm](https://cloud.githubusercontent.com/assets/11469505/23835988/f64f53bc-072d-11e7-9da9-2b6350c31626.png)


Here is the undistorted road image (you can see the white car on the right has been undistorted)

![screen shot 2017-03-12 at 2 11 20 pm](https://cloud.githubusercontent.com/assets/11469505/23835990/faed658a-072d-11e7-8154-a1f7d5867224.png)


Part 2: Color transforms, gradients and color thresholding to create binary image

Got inspirations from the following sources: 
- Udacity CarND Class
- More robust lane finding using advanced computer vision techniques
https://chatbotslife.com/robust-lane-finding-using-advanced-computer-vision-techniques-46875bb3c8aa#.q5ecs8rew
- Udacity SDCND : Advanced Lane Finding Using OpenCV
https://medium.com/@heratypaul/udacity-sdcnd-advanced-lane-finding-45012da5ca7d#.72sakpqy3

In order to accomplish the best result, I created a pipeline to get binary image. Here are the steps:

1. Convert img to HSV color space, sort out l channel
2. Take the derivative in x
3. Absolute x derivative to accentuate lines away from horizontal
4. Convert the absolute value image to 8-bit
5. Convert img to hsv, sort out v channel 
6. Create v_binary which is binary image only contains v value 
7. Get threshold x gradient sxbinary
8. Get threshold color channel l_binary 
9. Stack and combine all 3 binary thresholds

I used test1.jpg in the test_images folder to get the following result: 

![screen shot 2017-03-12 at 2 14 21 pm](https://cloud.githubusercontent.com/assets/11469505/23836003/4086935a-072e-11e7-8a7a-7153662926b9.png)


Binary Image 

![screen shot 2017-03-12 at 2 14 56 pm](https://cloud.githubusercontent.com/assets/11469505/23836006/4b191e96-072e-11e7-8bdd-807cf6340dba.png)


Part 3: Warp Images

After many tries, I used the following approach to warp the image: 

src = np.float32([ [540, 450], [720, 450], [1280, 720], [0, 720] ])
img_size = [1280, 720]
dst = np.float32([ [100,0], [img_size[0]-100, 0], 
    [img_size[0]-100,img_size[1]], [100,img_size[1]] ])
    
Another option is: 

src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) + 15), img_size[1]],
    [(img_size[0] * 5 / 6) + 90, img_size[1]],
    [(img_size[0] / 2 + 90), img_size[1] / 2 + 100]])

dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

I used cv2.getPerspectiveTransform to get perspective transformation matrix M, and use cv2.warpPerspective and pass in M to get warped image. 

Here is the warped image: 

![screen shot 2017-03-15 at 2 05 27 am](https://cloud.githubusercontent.com/assets/11469505/23940979/36667974-0924-11e7-8a8b-8e86daa0c10a.png)


Part 4: Identify lane-line pixels and fit their positions with a polynomial

Since this is a big section, I break down to the following steps: 
1. Create Histogram to observe the peaks
2. Create function get_pixels_for_array to return x,y coordinates where pixel value is 1
(use cv2.bitwise_and function and a mask)
3. Get pixels associated with lane lines from warped images 
  - Define number of slices as 5
  - Define the bottom and top y 
  - Finding left peak and right peak
  - Get left vertices and right vertices 
  - Get left pixels and right pixels
  - Extend left and right pixel values to corresponding arrays: x_left, y_left, x_right, y_right
  - Enhance polynomial curve 

Histogram 
![screen shot 2017-03-15 at 2 05 38 am](https://cloud.githubusercontent.com/assets/11469505/23940990/401ec58e-0924-11e7-883b-e231faa6375a.png)



Plot visualization
![screen shot 2017-03-15 at 2 05 46 am](https://cloud.githubusercontent.com/assets/11469505/23940994/4563cf62-0924-11e7-9a79-962b87594e38.png)



Part 5: Image and Video processing

To generate sample picture with clear lane detection, here are the steps: 

1. Use np.zeros create an image with zeros 
2. Transform x and y points, use cv2.fillPoly to fill in green color 
3. Use inverse perspective matrix, combine with original image

Lane area sample image
![screen shot 2017-03-12 at 7 18 41 pm](https://cloud.githubusercontent.com/assets/11469505/23839091/bfa4c970-0758-11e7-89e4-01582244957f.png)

Here are the steps I took to calculate the radius of curvature of the lane and the position of the vehicle with respect to center:

1. Convert x and y pixels to meters. 
2. Take initial position for the vehicle as the center of lane
3. Find left curvature difference, and right curvature difference 
4. Take the average value of the left and right radius of curvatures

Video project_video_output_mar15_2 is submitted along with Jupyter notebook.

Discussion: 

1. When I was taking Udacity class, it's pretty challenging to understand the steps to calculate the radius of curvature. I read many blog posts, viewed insights in Slack channel and finally got the solution. 

2. The pipeline will not work if the pixel detection is bad. I tested it with harder challenge video and it didn't perform too well (Therefore I didn't include it in the Jupyter notebook). Edge detection may have too much false positives (such as shadows). Compute the skelelon on edges masks will be a good start. 

