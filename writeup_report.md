## Project Writeup
---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use colour transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to centre.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/1.1_undistorted_checkerboard.png "Undistorted"
[image2]: ./output_images/1.2_undistorted_road.png "Road Transformed"
[image3]: ./output_images/2.1_binary_image_production.png "Binary Production"
[image4]: ./output_images/2.2_binary_image.png "Binary Example"
[image5]: ./output_images/3.1_perspective_transform_test.png "Warp Test"
[image6]: ./output_images/3.2_birds_eye_view.png "Birds-Eye View"
[image7]: ./output_images/4_detect_and_fit_lane_edges.png "Fit Lane Edges"
[image8]: ./output_images/5_test_image_with_annotated_lane.png "Overlay Lane Fit on Road"
[image9]: ./output_images/6_test_image_with_annotated_lane_and_text.png "Fully Annotated Output"
[image10]: ./output_images/7.1_annotated_lane_test_image_1.png "Pipeline applied to test 1"
[image11]: ./output_images/7.2_annotated_lane_test_image_2.png "Pipeline applied to test 2"
[image12]: ./output_images/7.3_annotated_lane_test_image_3.png "Pipeline applied to test 3"
[image13]: ./output_images/7.4_annotated_lane_test_image_4.png "Pipeline applied to test 4"
[image14]: ./output_images/7.5_annotated_lane_test_image_5.png "Pipeline applied to test 5"
[image15]: ./output_images/7.6_annotated_lane_test_image_6.png "Pipeline applied to test 6"
[image16]: ./output_images/7.7_annotated_lane_test_image_7.png "Pipeline applied to test 7"
[image17]: ./output_images/7.8_annotated_lane_test_image_8.png "Pipeline applied to test 8"
[video1]: ./project_video_annotated.avi "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

**NB**: *The code for this project was developed in the IPython notebook located at `./notebook.ipynb`. All the code referenced in this report can be found there. Please ignore `./notebook-problem-images`. It is a copy of notebook.ipynb and has been modified to enable the exploration of problem images captured from the project video.*

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd through 5th code cell of the IPython notebook located at `./notebook.ipynb`.  

*As stated in the template writeup...*

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
The code for this step can be found in the 6th code cell of the IPython Notebook.


As a part of the camera calibration and un-distortion process, I stored the results of the `mtx` and `dist` properties of the camera calibration process. Using these properties with `cv2.undistort()` applied to each test image allowed me to produce an undistorted version of all of the test images. An example of before and after the un-distortion process can be seen below. The difference is subtle, but noticeable.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step can be found in 9th code cell of the IPython Notebook. Code cells 7 and 8 were used for experimentation and not utilised to generate the binary images displayed below.

I used a combination of colour and gradient thresholds to generate a binary image.

The binary image generation process follows these steps:

- Convert the image to the HSV colour space and separate the channels.
- Increase the contrast of the saturation channel by multiplying its values by a factor of 2.
- Create a new channel that is the new high contrast saturation channel minus the inverse of the luminosity channel. This new channel helps highlight areas of high saturation that occur in the brighter areas of the image. This channel was created in attempt to exclude high saturation shadows in the image.
- Three binary channels were then created based on:
    + The x gradient of the luminosity channel (green in the colour composite)
    + The x gradient of the high contrast saturation channel (blue in the colour composite)
    + The value of the saturation minus inverse luminosity channel (red in the colour composite)

The final black and white binary image is black where none of the three layers have a value greater than 0 and white wherever any of the layers have a value greater than 0 (effectively an OR boolean operation).

![alt text][image3]

An isolated version of just the black and white binary image so it can be seen more clearly on narrow monitors.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step can be found in 10th through 12th code cells of the IPython Notebook.

The code for my perspective transform includes a function called `birds_eye_view()`, defined in the 10th code cell  of the IPython notebook.  The `birds_eye_view()` function takes as input an image (`img`). The `src` and `dst` properties for the perspective transformation are hard coded within `birds_eye_view()`. The hard coded values are as follows:

```
height = img.shape[0]
width = img.shape[1]

src_tl = [555, 450]
src_tr = [725, 450]
src_bl = [-200, height]
src_br = [width + 200, height]
src = np.float32([src_tl, src_tr, src_br, src_bl])

dst_tl = [0, 0]
dst_tr = [width, 0]
dst_bl = [0, height]
dst_br = [width, height]
dst = np.float32([dst_tl, dst_tr, dst_br, dst_bl])
```
This results in the following source and destination points:

| Source (mapped portion of the red triangle) | Destination (green rectangle) |
|:-------------------------------------------:|:-----------------------------:|
| 586, 450                                    | 229, 0                        |
| 694, 450                                    | 1051, 0                       |
| 1180, 720                                   | 1051, 720                     |
| 100, 720                                    | 229, 720                      |

I verified that my perspective transform works as expected by drawing a 1-point perspective triangle on the input image. As long as the perspective of the triangle matches the perspective in the image, we can be sure that the two sides of the triangle are in fact parallel in the 'space' of the image. I chose to position the base of the triangle exactly 100 pixels inside each edge of the image as it makes for easier calculations.

I then generated the birds-eye view of that image and drew a rectangle on the warped view that acted as a test for the transformation. By using the slope of the triangle and the dimensions of the perspective transformation, I was able to calculate the exact size and position of the rectangle in the warped image. Once this was done, it was just a matter of trial and error to find the right values of the `src` and `dst` polygons that define the transformation. This allowed me to be as certain as I could that the transformation is in fact a top down view of the road.

These are the images I used to test and confirm the perspective transformation.

![alt text][image5]

An example of the birds-eye view transformation applied to a binary image is below. The lane line pixels are very clear in the warped image. More importantly, the curvature of the lines is quite consistent along the height of the image and match quite well from left to right. This helps validate that the warped image is in fact displaying a top-down view of the road.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step can be found in 13th through 15th code cells of the IPython Notebook.

I chose to use the sliding windows using convolutions approach to matching the pixels to the lane lines. This approach was described in the Udacity course as follows:

> Another way to approach the sliding window method is to apply a convolution, which will maximise the number of "hot" pixels in each window. A convolution is the summation of the product of two separate signals, in our case the window template and the vertical slice of the pixel image.

> You slide your window template across the image from left to right and any overlapping values are summed together, creating the convolved signal. The peak of the convolved signal is where there was the highest overlap of pixels and the most likely position for the lane marker.

Through a large amount of trial and error using a variety of example binary images produced by my code, I found the following settings worked best

```
window_width = 50
window_height = 80
margin = 50
```

Such a small margin value works well for the example video and images in the project but would not generalise well for images and videos that display tight winding roads (such as the hard challenge video for this project).

Once we have the convolution pixels for the sliding window search for each of the left and right lane lines, we can fit a second order polynomial. to those pixels and define the 'fitted' lane lines.

This is an example plot of the convolution pixels mapped back onto the white areas of the binary image with the 'fitted' lane lines drawn on top in yellow.

![alt text][image7]

For the case of the video where we can make assumptions about the contiguous relationship between sequential frames, the average of the last n polynomial fits is used to help smooth the transition between sequential frames in the video. Through trial and error, a value of 5 for n was found to be a good compromise between smoothness and accuracy.

A further step was taken to help avoid dramatically inaccurate lane fits when frames of the video resulted in ambiguous or sparsely populated binary images.

I decided that it was reasonable to exclude the polynomial fits for a given frame if the left and right fits 'disagreed' with each other. I defined a disagreement as a squared difference between the first and second coefficients above a certain threshold. Adding this step made a massive difference in reducing the noice introduced by changes in pavement texture, shadows that are cast across the lane as well as the shadows cast by cars in the neighbouring lane that produce a higher contrast signal than the lines on the road.

These two thresholds were determined first by comparing successful fits for the provided test images and unsuccessful fits from problematic frames from the project video. The values were further fine tuned by producing several annotated videos and performing a qualitative comparison of the results. Inevitably, the choice of values came down to a compromise between smoothing out the lane detection for problematic areas in the video and maintain accuracy when corners straightened out.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to centre.

The code for this step can be found in 17th and 18th code cells of the IPython Notebook. The 16th code cell was used to experiment with the code from the course and not included in the pipeline.

The curvature of the fitted lane lines assumes that the curve follows the curve of a circle. By making this assumption, we can calculate the curvature in pixels and then map the pixel curvature to a set of conversion rations that allows us to calculate the curvature in meters. A single lane curvature is then calculated by taking the mean of the left and right curvature values.

A similar process was used to calculate the mid point between the 'fitted' lane lines on the x-axis and compare that position to the centre of the image (centre of the vehicle). The resulting value when converted to meters allows us to determine how far the car is from the centre of the lane with negative values implying the car is to the left of centre and positive values implying it is to the right.

For the case of the video where we can make assumptions about the contiguous relationship between sequential frames, the previous curvature and previous position values are averaged with the current ones to help smooth the transition between frames and help avoid individual jumps in these values.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step can be found in 19th and 22nd code cells of the IPython Notebook.

![alt text][image8]

![alt text][image9]

All of these steps were turned into a single function that called each step in succession. The `find_and_highlight_lane()` function was then applied to all 8 test images (shown below) and then applied to the video.  

![alt text][image10]
![alt text][image11]
![alt text][image12]
![alt text][image13]
![alt text][image14]
![alt text][image15]
![alt text][image16]
![alt text][image17]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

**NB**: *In the video, the green area represents the smoothed fitting of the lane lines, the blue and red areas show the un-smoothed window searches for that frame. I chose to leave the video like this to illustrate how the steps taken to smooth the lane detection overcome most of the problems with fitting individual frames.*

Here's a [link to my video result](./project_video_annotated.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail?  What could you do to make it more robust?

I took the approach of experimentation and then wrap the working code in a function. I new that the end result would need to be a single function that encapsulates the pipeline so I tried to keep that in mind. The first pass at each step of the project was simply 'get it working'. This was then followed by a long period of refinement where I improved the binary image generation and added steps to help smooth transitions in the annotated video and overcome problem areas. These are the problem areas I encountered.

- The two bridges: the window search highlighted areas of the edge of the bridge causing the lane to appear to curve right when it shouldn't have been.
- When the black car passes to the right of the view of the camera, the shadow of the car produces a stronger signal than the lines on the road.
- The shadow cast by the tree across the width of the lane causes all sorts of problems with the fit especially when the shadow is in the foreground of the image.

The pipeline is most likely to fail for very sharp corners, images / video where the lane lines are obscured or out of view of the camera, long stretches of road where the left and right lane line fits 'disagree' with each other.

I think wet road conditions would also prove to be very problematic as the thresholds used to generate the binary image would need to be entirely different.

I think the pipeline could be made more robust by using dynamic thresholds for the binary image generation as well as introducing more sophisticated fitting and smoothing techniques. This would be particularly necessary for videos where there are long stretches of 'ambiguous' lane lines.
