# **Finding Lane Lines on the Road**

### This is my term1 project 1 of the self-driving nanodegree.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps.

* First, I converted the images to grayscale,
* then I apply a Gaussian Noise kernel to denoise the image
* after that, I use canny edge detector to detect the edge in the image
* then I use a mask to get the ROI.
* get the HoughLines and draw the lines in the image

To make the line segments be a line, I modified the draw_lines() function, first I  divide lines into 2 groups by the sign of the slope, and calculate the slope and offset parameters of each line:

```python
for line in lines:
    for x1, y1, x2, y2 in line:
        if x2-x1 < 0.01:
            continue
        slope = (float(y2-y1)/float(x2-x1))
        if slope > 0:
            right.append([x1, y1, x2, y2])
            slope_right = slope_right + slope
            b = y1 - slope * x1
            right_b_term.append(b)
        else:
            left.append([x1, y1, x2, y2])
            slope_left = slope_left + slope
            b = y1 - slope * x1
            left_b_term.append(b)
        y_top = min(y_top, y1, y2)
```

I use the y of topest point as the y_top, then I calculate the average of the slope and b:

```python
if len(left) != 0:
    slope_left = slope_left / len(left)
    left_avg_b = sum(left_b_term)/len(left_b_term)
    x1, y1, x2, y2 = extrapolate(left, slope_left, y_top, img.shape[0], left_avg_b)
    cv2.line(img, (x1, y1), (x2, y2), color, thickness)

if len(right) != 0:
    slope_right = slope_right / len(right)
    right_avg_b = sum(right_b_term)/len(right_b_term)
    x1, y1, x2, y2 = extrapolate(right, slope_right, y_top, img.shape[0], right_avg_b)
    cv2.line(img, (x1, y1), (x2, y2), color, thickness)
```

finally I calculate and draw the line by using the slope, b, y_bottom, y_top of the left line and right line.


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be the complex lane line detection. When the lane line is a curve line, this pipeline will not work since it can only draw straight line.

Another potential shortcoming is that the pipeline is susceptible to other vehicles in the same lane. Because the pipeline was implemented by the edges detection, the edge of other car can influence the performance.


### 3. Suggest possible improvements to your pipeline
maybe we can use some kind of higher order polynomial fit to handle curvy lanes.

maybe, some artificial intelligence methods can help to improve the performance of the pipeline.
