
[//]: # (Image References)

[image1]: ./examples/gray.jpg "Grayscale"
[image2]: ./examples/uneven_pavement.png "Bad pavement"
[image3]: ./examples/shadows.png "Shadows"
[image4]: ./examples/edges.jpg "Canny edges"
[image5]: ./examples/mask.jpg "Region of interest mask"
[image6]: ./examples/failure.png "Without angle filter"
[image7]: ./examples/masked_edges.jpg "ROI applied"
[image8]: ./examples/HoughLines.jpg "Hough Lines"
[image9]: ./examples/final_lines.jpg "Final lines"

### Reflection

### 1. Pipeline description. Draw_lines() modifications.

My pipeline consisted of 5 steps:

1. First, I converted the images to grayscale.
![gray][image1]

2. Then I extracted Canny edges, without prior explicit blurring, since cv2.Canny() already has Gaussian blurring with the 5x5 kernel inside.
![Canny][image4]

3. After I applied region of interest mask of this shape:
![ROI mask][image5]
And left only with these Canny edges:
![ROI mask applied][image7]

4. Extracted Hough lines. Decided to draw them with blue colored lines, just for debugging and clear presentation of the pipeline:
![Hough lines][image8]

5. And finally, extrapolated these Hough lines to the whole ROI area:
![Extrapolated lines][image9]

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by collecting all coordinates of negative and positive sloped lines in separated lists and then making linear regression on these points, using np.polyfit() function.

With good dataset, like the first two videos, this algorithm of line extrapolation worked perfectly. But on optional dataset there is a lot of wrong edges, caused by poor asphalt and car's hood.
![failure][image6]

Filtering slope of lane lines to be near the average from previous two videos, helped to solve the problem of complete line extraction failure.
![shadows][image3]

But still, in some frames there was not enough edges, detected on lane lines.
And I've somehow fixed it by applying simplified version of Kalman filter with fixed Kalman gain. I store lines parameters from the previous frame in global variables and since our car can't move instantly sideways, lane lines more probably will stay near the previous position.
![uneven pavement][image2]

As we see on the picture, although it works better, but there is a lot of room to improvements.

### 2. Potential shortcomings with current pipeline

One potential shortcoming would be when lane lines are curved, or have different slope than my algorithm expects, or not visible enough, or there is some sign on the asphalt. My algorithm will give some wrong estimation of the lane lines.

Another shortcoming could be if the car engage lane switch, one of the lines will be out of our region of interest. My algorithm will most probably draw two wrong lines because it assumes that there is exactly two lane lines in the image. And there is a possibility to have a perfectly vertical edge, which will cause division by zero error. I've marked it, but I won't fix it for now, because it works and I'm a bit exhausted (not lazy xD).

Finally, this pipeline is quite intolerant to road defects, shadows, camera placement, image resolution and unexpected conditions. Need to improve it to use in real life :)


### 3. Possible improvements to my pipeline

A possible improvement would be to work in HSV space instead of RGB to better filter yellow and white colours.

Another potential improvement could be to use homography properties of lane lines (they need to be parallel and on almost constant distance from each other).
