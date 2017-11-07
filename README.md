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

To make the line segments be a line, I modified the draw_lines() function, first I use a filter to select the line segment by controlling the slope. the code is:

```python
for line in lines:
    for x1,y1,x2,y2 in line:
        if abs(x2 - x1) < 0.001:
            continue
        slope = (float(y2-y1)/float(x2-x1))
        if not np.isnan(slope) or np.isinf(slope) or (slope == 0):
            if (slope > -1.5) and (slope < -0.3) :
                linesFiltered.append(line)
            if (slope > 0.3) and (slope < 1.5) :
                linesFiltered.append(line)
```
After the filter, I divide these lines into 2 groups by the sign of the slope, and calculate the slope and b with weighted average, the weights I used here is the length of these line segments:

```python
for line in linesFiltered:
    for x1, y1, x2, y2 in line:
        slope = (float(y2-y1)/float(x2-x1))
        if not np.isnan(slope) or np.isinf(slope) or (slope == 0):
            if slope > 0:
                right.append([x1, y1, x2, y2])
                length = math.sqrt((x2-x1)**2 + (y2-y1)**2)
                right_length.append(length)
                slope_right.append(slope)
                b = y1 - slope * x1
                right_b_term.append(b)
                right_y_top = min(right_y_top, y1, y2)
            else:
                left.append([x1, y1, x2, y2])
                length = math.sqrt((x2-x1)**2 + (y2-y1)**2)
                left_length.append(length)
                slope_left.append(slope)
                b = y1 - slope * x1
                left_b_term.append(b)
                left_y_top = min(left_y_top, y1, y2)

if len(left) != 0:
    s_l, b_l = weight_mean(left_length, slope_left, left_b_term)
    # fix the y_top at 320
    x1, y1, x2, y2 = extrapolate(s_l, left_y_top, img.shape[0], b_l)
    cv2.line(img, (x1, y1), (x2, y2), color, thickness)

if len(right) != 0:
    s_r, b_r = weight_mean(right_length, slope_right, right_b_term)
    x1, y1, x2, y2 = extrapolate(s_r, right_y_top, img.shape[0], b_r)
    cv2.line(img, (x1, y1), (x2, y2), color, thickness)
```

I use the y of topest point as the y_top in one side, the weight_mean() function is :

```python
def weight_mean(length, slope, b):
    sum_length = sum(length)
    weights = []
    for l in length:
        weights.append(l/sum_length)
    for i in range(len(slope)):
        slope[i] = slope[i] * weights[i]
        b[i] = b[i] * weights[i]
    s_l = sum(slope)
    b_l = sum(b)
    return s_l, b_l
```


finally I calculate and draw the line by using the slope, b, y_bottom, y_top of the left line and right line.


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be the complex lane line detection. When the lane line is a curve line, this pipeline will not work since it can only draw straight line.

Another potential shortcoming is that the pipeline is susceptible to other vehicles in the same lane. Because the pipeline was implemented by the edges detection, the edge of other car can influence the performance.


### 3. Suggest possible improvements to your pipeline
maybe we can use some kind of higher order polynomial fit to handle curvy lanes.

maybe, some artificial intelligence methods can help to improve the performance of the pipeline.
