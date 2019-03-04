# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images/pipeline_output/grayscale.jpg "Grayscale"
[image2]: ./test_images/pipeline_output/gaussian_blur.jpg "Gaussian Blur"
[image3]: ./test_images/pipeline_output/canny_edges.jpg "Canny Edges"
[image4]: ./test_images/pipeline_output/region_interest.jpg "Region of Interest"
[image5]: ./test_images/pipeline_output/hough_lines.jpg "Hough Lines"
---

### Reflection

### 1. The pipeline for line segment detection in images

My pipeline consist of 5 steps described below: 


Step 1: I converted the images to grayscale :

![Gray Scale Image][image1]

Step 2: I applied gaussian smoothing to get a blur gray image :

![Blue Gray Image][image2]

Step 3: I applied the canny edge detector algorithm on the gray image to detect all the edges in the image while converting
other non-edge pixels to black:

![Canny Edge][image3]

Step 4: I used a four sided polygon to mask the edges image, consequently defining my region of interest on the edge image

![Region Of Interest][image4]

Step 5: Using the hough space algorithm, I was able to detect straight lines on the image, the left and right lines in particular. 

![Hough Lines][image5]


### Understanding hough transform

Hough Transform is a computer vision technique for detecting shapes in an image. In our case, we want to detect lines. A straight line is represented by the line equation y = mx + b where m is the slope (i.e y2 -y1/x2-x1) and b is the intercept. This line can be represented in hough space. A line in x and y space is represented by a dot in hough space. A set of dots in hough space that a line can be drawn through indicate that the lines in x and y space are part of the same line.

The point of line intercept in hough space with m amd b values correspond to the sets of lines in x and y space whose points connect to the same points on the x and y space.

In drawing lines to connect several points, you could have several lines trying to fit several points together. How to we determine the line that best fit? in this case we vote, by making a grid over hough space and counting the number of intercepts. The intercepts with highest number of line is voted as our best fit. The slope and intecept values of this intercept in hough space is used in our x and y space to determine a line.

So far, can we correctly identify lines ? Not yet because our approach will only work for lines with slopes but not vertical and horizontal lines whose slope is infinity. We cannot represent this slope in hough space, therefore we need a more robust solution. Instead of representing our parameter in cartesian coordinates, we represent them as polar coordinates. Instead of our straight line representation of y=mx+b we use polar coordinate.

The following parameters define hough lines when passed to the opencv function cv2.HoughLinesP()

rho - It is the grid over hough space, which is a 2 dimensional array which contains the bins of rows and columns we use in voting the best line fit 
theta - theta specifies the size of each bin in rho. It is the angle of the accumulator in radians. The smaller the bins the lesser the decision of which lines to be detected. The smaller theta is, the smaller the number of intersections, the consequently the higher the precision in detecting our lines. If theta is too small however, the more inconsitent results we get and the longer it takes to run
threshold - The minimum number of votes needed to accept a candidate line
minimum line length - The length of line in pixels which we will accept as a line
maximum line gap - The maximum distance in pixels between segmented lines which we will allow to be connected to a single line
instead of being broken up



### How to average/extrapole line segments

I modified the draw_lines() function by writing an average_slope_intercept() function that took as input hough lines and returning the average line; one for the left line and the other for the right line.

In particular, the average_slope_intercept() function works in the following manner :

1. Declare two empty arrays left_fit and right_fit
2. Loop through the hough lines, unpack each line into coordinate points (x1,y1,x2,y2) 
3. Fit a polynomial of degree one y = mx + b into a coordinate point; the result is the coefficient of y = mx + b [m,b]
   where m is the slope and b is the intercept
4. Store all values of m that are positive into the earlier defined left_fit array with its corresponding intercept and 
   vice_versa for negative m values into right_fit array
5. Average both left_fit, and right_fit array using numpy function np.average
6. Use the average value of the slope and intercept to compute the coordinate point for both the left line and the right line
   i)   y1 is the starting point of the line from the bottom of the image which corresponds to the image height  
        y1=image.shape[0]
   ii)  y2 is how far from y1 the line should go, I use y1*(3/5) to calculate an offset 
   iii) x1 can be computed from the equation y = mx1 + b => x1 = (y1-b)/m  where y1, m, and b is known
   iv)  x2 can be computed from the equation y = mx2 + b => x2 = (y2-b)/m  where y2, m, and b is known
7. We now have the coordinate points for the left and right line, these are unpacked and passed to the
   opencv function cv2.line(), with the color value and thickness to draw the line on the image
7. We now have a smooth and continoous line stretching from the bottom of the image into the horizon.
   
   

### 2. Possible Challenges

One potential shortcoming would be what would happen when the computations lead to overeflowerror on some test images.
Another shortcoming could be detecting line segments in roads with sharp bends and been able to get good result nonetheless.


### 3. Possible Pipeline improvement

Handle overflowerror to make the pipeline more robust for any kind of video image

Another potential improvement could be to make the line drawn on broken line segments not to flicker but remain steady and continous
