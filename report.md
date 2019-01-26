
**Finding Lane Lines on the Road**

[//]: # (Image References)

[gray]: ./test_images_output/gray.jpg "Grayscale"
[blur_gray]: ./test_images_output/blur_gray.jpg "Grayscale Gaussian"
[edges]: ./test_images_output/edges.jpg "edges"
[mask]: ./test_images_output/mask.jpg "mask"
[mask_edges]: ./test_images_output/mask_edges.jpg "masked_edges"
[hough_lines]: ./test_images_output/hough_lines.jpg "hough_lines"
[hough_lines_img]: ./test_images_output/hough_lines_img.jpg "hough_lines_img"
[solid_lines]: ./test_images_output/solidYellowLeft.jpg "solid_lines"

---

### Pipeline

#### 1. apply grayscale with `cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)`
![alt text][gray]

#### 2. apply Gaussian smoothing to reduce noise with `cv2.GaussianBlur`
![alt text][blur_gray]

#### 3. find edges in image with `cv2.Canny` 
![alt text][edges]

#### 4. assuming the camera is mounted at a fixed position on the vehicle based on test images we can use a polygon to focus on a region of interest mask
![alt text][mask]

#### 5. mask edges image to focus on region of interest
![alt text][masked_edges]

#### 6. detect straight lines in region of interest with `cv2.HoughLinesP`
![alt text][hough_lines]

#### 7. hough lines on original image
![alt text][hough_lines_img]

#### 8. finally extrapolate lines to the top and bottom of the lane
![alt text][solid_lines]

To extrapolate lines to the top and bottom of the lane;
first i needed to separate the right lines from the left lines
since all lines found by hough algorithim was in a single array `lines`
to do this i calculated the slope of each line segment in the array using

    slope = ((y2-y1)/(x2-x1))

negative and positive slope value belong to the left lane and right lanes respectively
Then using the "point-slope" form of the equation of a straight line: `y − y1 = m(x − x1)`
solving for `x = ( (y-y1)/ m ) + x1`
given 

    start_y = 320 # top of line
    end_y = img.shape[0] # bottom of img

solved for x by picking any point on the respective line (x1,y1)
example given point(x1,y1) as `[748.0, 465.75]` the corresponding
end_x and end_y is `493 877`

I observed that picking a point at random in each lane resulted in occationally
incorrect extrapolated lines. 

So solve this i had to average the points to smooth things out
in other words avg all `x1` and `y1`

### Shortcomings


one shortcoming i observed was when calculating the average of `x1` and `y1`
the average gets completely thrown off in the case of lines detected outside of the lane line.

The region of interest shaped as a triangle tends to include a lot of irrelevant regions
in cases of sharp turns in the road


### Possible improvements

A possible improvement would be in addition to the avg `x1`, `x2` the pipeline
needs a way to detect and exclude lines from objects that are close to the lane

The pipeline also needs to be able to employ some form of dynamic update on the
region of interest using the slope or as the road curves in order to focus on the lane lines.
