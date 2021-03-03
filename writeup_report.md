## Writeup Report



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

[image1]: ./output_images/Testing_Module_2/Distortion_Correction_Result.jpg "Undistorted"
[image2]: ./output_images/Testing_Module_5/test2_output_undist.jpg "Road Undistorted"
[image3]: ./output_images/Testing_Module_5/test2_output_binary.jpg "Color and Gradient Thresholding Result"
[image4]: ./output_images/Testing_Module_6/Perspective_Transform_Result2.jpg "Road Transform"
[image7]: ./output_images/Testing_Module_6/Perspective_Transform_Result.jpg "Road Transform binary"
[image5]: ./output_images/Testing_Module_8/Polynomimal_Result.jpg "Fit Polynomimal"
[image6]: ./output_images/Testing_Module_10/test2_lane_area.jpg "Land area final output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  


### Module 1 and Module 2 - Camera Calibration and Distortion Correction


- The code for this step is contained in **(Testing Module 1)** and **(Testing Module 2)** of the IPython notebook located in `"./video_pipeline.ipynb" `

- I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

- I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction **(Testing Module 2)** to the test image using the `cv2.undistort()` function and obtained this result from one of the testing images: 

    ![Undistorted Chessboard Image][image1]

- The output images of all the test images could be find under `/output_images/Testing_Module_2`

### Module 5 - Apply the color threshold on HLS and the Gradient threshold 

- The code for this step is contained in **(Testing Module 5)** of the IPython notebook located in `"./video_pipeline.ipynb" ` 

- Frist, I stored the camera calibration result in `mtx` and `dist` in Module 2 previously, then I perform the distortion correction in this module via using `mtx`, `dist` and `cv2.undistort()` function. the distortion correction to one of the test images is like this one:
    ![Undistorted Road Image][image2]

- Second, in `color_grd_threshold_check` function, I applied the color and x gradient threshold as follows: 
    - Convert to HLS color space
    - Use `L channel` and apply `x gradient` threshold (30, 100)
    - Use color threshold on `S channel` (150, 255)
    - combine the reulst from both and come up with the reuslt like the image below:

    ![Color and Gradient Thresholding Result][image3]

    - The output of all the testing images could be find under `/output_images/Testing_Module_5`


#### Module 6 - Perspective Transform

- The code for this step is contained in **(Testing Module 6)** of the IPython notebook located in `"./video_pipeline.ipynb" `

- The code for my perspective transform includes a function called `perp_transform()`. The `perp_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

- This resulted in the following source and destination points:

    | Source        | Destination   | 
    |:-------------:|:-------------:| 
    | 585, 460      | 320, 0        | 
    | 203, 720      | 320, 720      |
    | 1127, 720     | 960, 720      |
    | 695, 460      | 960, 0        |

- I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

    ![alt text][image4]

- I also converted the above inage into binary format as follows:
    ![alt text][image7]

#### Module 8 - Sliding Window and Fit a Polynominal

- The code for this step is contained in **(Testing Module 8)** of the IPython notebook located in `"./video_pipeline.ipynb" `

- Based on the binarytop-down image produced by `Module 6`, send it to the function called `fit_polynomial()`. The `fit_polynomial()` function will find the lane line pixels via `find_lane_pixels()` function.

- In `find_lane_pixels()` function, there are multiple steps to return the result, they are describes as follows: 
    - Take a histogram of the bottom half of the image
    - Find the peak of the left and right halves of the histogram, These will be the starting point for the left and right lines
    - Define the hyper parameters, such as `nwindows`, `margin` and `minpix`
    - Set height of windows - based on nwindows above and image shape
    - Identify the x and y positions of all nonzero pixels in the image (`nonzeroy`, `nonzerox`)
    - record the current position in `leftx_current` and `rightx_current`
    - For each window, find the nonzero pixels in x and y and use them to recenter the next window
    - Finally, return all the nonzero pixels in `leftx`, `lefty`, `rightx`, `righty`

- Once the result from `find_lane_pixels()` is retrieved, use them via `np.polyfit` to get the coefficients and stored in `detected_line.left_fitx`, `detected_line.right_fitx` and `detected_line.ploty`
- I also get the x position of both of the left and right lanes at the most bottom part of the image, calculate the average and stored in the `detected_line.car_position` 

- The result of 2nd order polynominal cocfficient and the lane lines could be decsribed as follows:
    ![Fit Polynomimal][image5]

#### Module 11 - Calculate the Curvature and the destance from the center

- The code for this step is contained in **(Testing Module 11)** of the IPython notebook located in `"./video_pipeline.ipynb" `

- In function `convert_pixel_to_real()`, I used the data stored in `detected_line.ploty`, `detected_line.left_fitx`, `detected_line.right_fitx` in `Module 8` and `np.polyfit` to come up with the new `left_fit_cr` and `left_fit_cr`, which are the 2nd order polynominal cocfficient.

- Then I calculated the real curvature in ihe function `measure_curvature_real()` by using the result of `convert_pixel_to_real()`, here are the steps in that function: 
    - Define conversions in x and y from pixels space to meters
        - 30/720 meters per pixel in y dimension
        - 3.7/700 meters per pixel in x dimension
    - Define `y_eval` (max y-value in `detected_line.ploty`, it's also the bottom of the image) as the radius of curvature
    - use `left_fit_cr` and `left_fit_cr` and `y_eval` to gecalculate the R_curve (radius of curvature) and stored in `left_curverad` and `right_curverad`
    - By using the `detected_line.car_position`, calculate the distance from the lane center 


#### Module 10 - Lane Area Drawing

- The code for this step is contained in **(Testing Module 10)** of the IPython notebook located in `"./video_pipeline.ipynb" `

- I implemented this step in the function `result_prep()`.  Here is an example of my result on a test image:
    ![Land area final output][image6]

---

### Production Module - Building Pipeline (video)

#### After running the tetsing Module 1 ~ 11 one by one, all the necessary elements are collected. 

#### The I run the 3 cells in this module, and the final output video is displayed as the following link:

Here's a [link to my output video result](./test_vide_output/project_video.mp4)

---

### Discussion

- When there is a large of shadow, my implementation seems to be not auucrate.
- It's very difficult for me to use the same approach on challenge_video.mp4 and harder_challenge_video.mp4
- The technique that I used in this project might not be capable of scaling up to challenging images.
- I will need to stury Deep Learning approach further to compare the result with this project and maybe improve the final output.
