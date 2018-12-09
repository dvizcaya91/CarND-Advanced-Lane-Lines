## README

---

**Lane lines detection project**

[//]: # (Image References)

[image1]: ./report_images/image1.png "Undistorted"
[image2]: ./report_images/image1.png "Road Transformed"
[image3]: ./report_images/image1.png "Binary Example"
[image4]: ./report_images/image1.png "Warp Example"
[image5]: ./report_images/image1.png "Fit Visual"
[image6]: ./report_images/image1.png "Output"
[video1]: ./output_videos/project_video.mp4 "Video"


---

### README

This project objective was to create a pipeline capable of automatically detecting the lane lines of the road, as well as its curvature. 

### 1. Camera calibration 

The code for this section can be observed in the first cells of the notebook project_2.ipynb.
First, all calibration images were loaded. Each image was converted to grayscale and then the
corners of the chessboard were calculated using the findChessboardCorners cv2 functions. The
points of the corners of all images were saved in the imgpoints variable. It is worth mentioning that
the function did not found the corners for 3 images. Nevertheless, this should not affect the
performance, given the number of calibration images.
After that, the variables created were used to calibrate the camera using the cv2 function
calibrateCamera.
A corners_unwarp function was created to evaluate the performance of the distortion over
the calibration images.

### 2. Pipeline 

The pipeline that process all images and video frames was developed under the function
image_pipeline. Inside this pipeline, other functions were called as will be explained below.

* *Distortion corrected image*
The first step of the pipeline is to correct the distortion of the image, using the
parameters found when calibrating the camera. Following can be found some examples of
images before and after the distortion correction

![alt text][image1]
![alt text][image2]


* *Color and gradients thresholds*
The operations for color and gradients thresholds can be found on the function called
transform_image. For the color transformation, the image was converted to HLS, the s channel
was selected, and the points were keep if the values were over 170, up to 255. The Sobel in x
was also evaluated. The thresholds in this case were 20 and 100.

![alt text][image3]

* *Perspective transformation*
The perspective transformation of an image was performed in the image_pipeline
function. The source points were defined as `[[(1280/2)*.9, (720/2)*1.3], [(1280/2)*1.1,
(720/2)*1.3], [1100,720], [200,720]]`. In the following example images, the red dots represent
the source and the blue points the destination.

![alt text][image4]

* *Lane-line pixels and polynomials*
In order to identify the lane-lines polynomials, the first thing was to calculate the
histogram of the half bottom section of the image. This was done on the image_pipeline
function. After that, the polynomials were calculated on the get_polys function.
The get_polys function receives as input the image, the histogram and the polynomials
of a previous frame (optional). An option for debugging the images is also available. The process
is different depending on whether there are polynomials of previous frames, or not.
In the case there are no previous polynomials, the first thing was to find the points that
were located in a window defined by the half bottom of the image, and the greater point on the
histogram +/- the margin (for both left and right). After that, a first polynomial was fitted. Then
the process was repeated, increasing the vertical scope by the window height, and the horizontal
scope was defined by the polynomial curve +/- the margin.
When there was a previous frame, there was no loop but instead, the whole image was
checked at once, where the pixel search was limited to the previous polynomial +/- the margin.
In order to avoid drastic changes, the x value of the polynomial at y=0 was compared with the
one from the frame before. In case the change of the x position was over the margin, a correction
was made by fitting the polynomial with both the current and the old pixels. This rule was very
helpful to avoid problems in some frames were the color and gradient thresholds included pixels
that were not the actual line.

![alt text][image5]

* *Curvature and car offset*
The curvature of the polynomial was calculated using the measure_curvature_real
function. This function received the pixels of the images for both lines. Those pixels were
converted to meters considering 30 meters per 720 pixels in the y dimension and 3.7 m per 700
pixels in the x dimension. After the pixels were converted, a new polynomial was fitted, and the
curvature was calculated with the same equation presented on the lessons.
The car offset was calculated on the get_car_offset function. This function calculates the
x value for y = max(y) for both lines (based on polynomials). After that, the distance between
the center of the image and each of the lines lower point was measured and converted to
meters.

![alt text][image6]

### 3. Pipeline (Video)

The pipeline used for the video was the same one described above. The only difference was
that for this case, a class called ProcessVideo was created. The purpose of this was to be able to save
the polynomials information between frames.
The final video can be found under the output_videos/project_video.mp4.[link to the video](./output_videos/project_video.mp4)

![alt text][image1]

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

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
