# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_images/solidYellowCurveColorFilter.jpg "ColorFilter"
[image2]: ./writeup_images/solidYellowCurveCanny.jpg "Canny"
[image3]: ./writeup_images/challenge_4s.jpg "Challenge 4s"
[image4]: ./test_images_output/solidWhiteCurve.jpg "SolidYellowCurve"

---

### Reflection
### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.
The steps of the pipeline are as follows: 

    1. Filter whites and yellows from image.
        This is done by converting the image colorspace to HSV and masking the original image with a combined white and yellow filter. This step attempts to minimize "noisy" gradients that result in false positives in the Hough line detection.

    2. Convert to grayscale.
        Convert image to grayscale to improve Canny edge detection algorithm's ability to detect gradients between neighboring pixels.
        The following image is the SolidYellowCurve.jpg with the yellow and white filter applied converted to grayscale.
![alt text][image1]

    3. Perform Canny edge detection.
        Identify edges in image by comparing gradients between neighboring pixels.
![alt text][image2]

    4. Smooth Canny edges using Gaussian blur in preparation for Hough line detection.
        Gaussian blur is used to smooth the image and remove noisy edges that would result in incorrectly characterized lines by the Hough line detection.

    5. Apply mask such that a predefined region is analyzed during Hough line detection.

    6. Perform Hough line transformation and combine detected lines.
       Attempt to characterize pixels in Canny edges as lines.
       Here the `draw_lines()` method was modified to separate the Hough lines into candidates for left and right lane lines by filtering based on slope and line end position ie group based on x2 after ordering y coordinates from low to high.
           If the x2 component of a detected line falls in the left/right half of the image, it becomes a candidate for the left/right lane line.
           A slope filter is applied to determine if the slope roughly corresponds to the expected slope of the lane line.
           After initial binning of the candidate lines, perform a weighted averaging of the left and right candidate lines based on total left/right candidate line length.
           Extrapolate weighted average to the y1, y2 extents of the masked region.

    7 Overlay image containing predicted lane lines on top of original image.
       The following is the SolidYellowCurve image with the predicted lines overlaid.
![alt text][image4]


### 2. Identify potential shortcomings with your current pipeline
  The predicted lane lines jitter about the actual lane lines in the overlaid videos.  In most cases the predicted lane line is close to the actual lane line but variations from video frame to frame make the overlaid output erratic.  This is particularly pronounced in the challenge video.

  The current approach assumes the lane lines fall within a somewhat tightly bound masked region.  The algorithm would fail to predict lane lines if the actual lines fell outside the masked region.

  The implementation also struggles to filter lane lines in highly saturated images.  The following image is the detected lane line at 4 seconds in the challenge video. The detection for the right lane line is quite weak.

![alt text][image3]

  Lastly, the algorithm expects a candidate line is generated for both left and right lane lines.  The implementation will raise an exception if this is not the case.  Proper error handling needs to be added to handle cases where the algorithm fails to detect lane lines.  In particular, at 4s in the challenge video, the highly saturated image results in very few predictions for the right lane.


### 3. Suggest possible improvements to your pipeline
The jitter in the output videos could be mitigated by performing a moving average of predicted lines over a period of multiple video frames.

Fine tuning the parameters governing the white/yellow filters, Canny edge detection, and Hough transforms could be improved to result in a stronger set of line predictions.

Proper error handling for cases in which the algorithm does not confidently identify lane lines could be implemented.  The weighted averaging of candidate line segments could be replaced with a more robust algorithm to produce a confidence metric.

