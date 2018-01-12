**Finding Lane Lines on the Road**

# Pipeline

The big picture of this project is to create a pipeline for a single image and apply the same for the entire stream of frames.
Lets walkthrough the list of steps applied on a single image frame. The pipeline consists of following steps

1. Isolate yellow and white from image
2. Convert image to grayscale for easier manipulation
3. Apply Gaussian Blur to smoothen edges
4. Apply Canny Edge Detection on smoothed gray image
5. Trace Region Of Interest and discard all other lines
6. Perform a Hough Transform to find lanes within our region of interest and trace them in red
7. Superimpose the lanes found on to the original image

Here is the input image 

[solidYellowCurve_input]: ./test_images/solidYellowCurve.jpg "Input Image"

![solidYellowCurve][solidYellowCurve_input]

### Step 1: Isolate yellow and white from image
mask = color_selection(image)

[solidYellowCurve_mask]: ./test_images_output/solidYellowCurve_mask.jpg "Color Selection"

![mask][solidYellowCurve_mask]

### Step 2: Convert image to grayscale for easier manipulation
gray = grayscale(mask)

[solidYellowCurve_gray]: ./test_images_output/solidYellowCurve_gray.jpg "Grayscale"

![gray][solidYellowCurve_gray]

### Step 3: Apply Gaussian Blur to smoothen edges
blur = gaussian_blur(gray, kernel_size)

[solidYellowCurve_blur]: ./test_images_output/solidYellowCurve_blur.jpg "Gaussian Blur"

![blur][solidYellowCurve_blur]

### Step 4: Apply Canny Edge Detection on smoothed gray image
edges = canny(blur, low_threshold, high_threshold)

[solidYellowCurve_edges]: ./test_images_output/solidYellowCurve_edges.jpg "Canny Edge Detection"

![edges][solidYellowCurve_edges]

### Step 5: Trace Region Of Interest and discard all other lines
roi = region_of_interest(edges, vertices)

[solidYellowCurve_roi]: ./test_images_output/solidYellowCurve_roi.jpg "Region Of Interest"

![roi][solidYellowCurve_roi]

### Step 6: Perform a Hough Transform to find lanes within our region of interest and trace them in red
lines = hough_lines(roi, rho, theta, threshold, min_line_length, max_line_gap)

[solidYellowCurve_lines]: ./test_images_output/solidYellowCurve_hough.jpg "Hough Transform and Extrapolated Lines"

![lines][solidYellowCurve_lines]

### Step 7: Superimpose the lanes found on to the original image
result = weighted_img(lines, image)

[solidYellowCurve_result]: ./test_images_output/solidYellowCurve_final.jpg "Merged Output"

![result][solidYellowCurve_result]

[solidYellowCurve_gif]: ./test_images_output/solidYellowCurve.gif "Video Output"

Here is the result when the pipeline is applied to each of the frame in the video.

![output][solidYellowCurve_gif]

---

# 1. Reflection

The draw_lines() function has been modified to draw two single lines one corresponding to the left lane of the road and the other for the right lane.

hough_lines() function returns a collection of lines. These lines needed to be grouped as belonging to left and right. The grouping is done based on the slope of the line, positive slope indicates they are slanting towards left, possible indicating a left line and negative slope indicate that the line is slanted towards right and possible left lane.

separate_lines() does this grouping using the slope on the input list of hough lines. The results are two collection of lanes, right and left.

Next task is to find a linear line which would pass all the lines. cv2.polyfit function does this calculation for us and returns the final slope and intercept of the line that would possible pass thought the given set of lane points.

average_lines() does this task for the 2 collections left and right lines found from separate_lines function call.

Finally extrapolate_lines does the task of calculating and drawing the two left and right lanes. The inputs to the function are slope and intercept along with the two co-ordinates that indicate the start and end for the lane. Start and End are the co-ordinates of our roi region which depicts the upper and lower boundary up to which we want to extrapolate the averaged lines.

[image1]: ./test_images/challenge_089.jpg "Grayscale"

If you'd like to include images to show how the pipeline works, here is how to include an image:

![alt text][image1]

# 2. Potential shortcomings with the pipeline


Current pipeline considers each frame to be independent and a slight jitter can be observed on the final video, as the lanes drawn on each frame don't overlap. If each frame can use input from the previous frame to extrapolate then the draw lines can be smother and increase the accuracy of the lane finding.

Current pipeline assumes that the car is in the centre of the lane, the current logic would fail if the car is switching lanes.

The pipeline assumes that the region of interest is same across all frames, this might not be true and a true pipeline might get additional sensor inputs as to compass or the orientation and elevation can help get a better region of interest for lane finding.
Example fo the car having to take a sharp turn or a elevation change on the road


## 3. Possible improvements to the pipeline

A possible improvement would be to use HSL and HSV and other color mapping to find and mask the color selection. Currently solution is bound to fail when the color of the white and yellow are at extreme shade boundaries. Also

Another potential improvement could be to separate left and right lanes based on their position in the image, along with the slope we we use the image centre to kind of determine if the line actually needs to be considered as left or right lane. This will help reduce the false positive lanes that are found which results in drawing incorrect lanes as shown below.
