# **Finding Lane Lines on the Road** 


[//]: # (Image References)

[image1]: ./result_images_original_drawlines/solidYellowCurve.jpg "Solid Yellow Curve"
[image2]: ./result_images_original_drawlines/solidWhiteCurve.jpg "Solid White Curve"
[image3]: ./result_images_original_drawlines/challenge2.jpg "Challenge2"
[image4]: ./result_images_original_drawlines/challenge3.jpg "Challenge3"
[image5]: ./result_images_modified_drawlines_without_filter/solidWhiteCurve.jpg "Solid White Curve"
[image6]: ./result_images_modified_drawlines_without_filter/challenge3.jpg "Challenge3 No Filter"
[image7]: ./result_images_modified_drawlines_without_filter/challenge4.jpg "Challenge4 No Filter"
[image8]: ./result_images/challenge3.jpg "Challenge3"
[image10]: http://coolvoyage.ru/wp-content/uploads/2014/05/monaco_raceway_hairpin.jpg "Monaco"

---

### Reflection

### 1. A Description of My Pipeline.

My pipeline consists of 5 steps. First, I convert the images to grayscale. Second, I do a Gaussian blur with a kernel size of 5 to remove the high frequency edges in the image. Third, I use the Canny algorithm, which uses gradients (among other things) to determine the edges in the image. I use a low threshold of 50 and a high threshold of 150 with the Canny algorithm. Fourth, I mask the portion of the image that will likely contain the lane lines by defining a quadrilateral and then creating a mask image with the quadrilateral and performing a bitwise and with the mask and the output of the Canny algorithm. Fifth, I do a Hough transform to find the line segments within the edges. For the Hough transform, I use a rho of 2, theta of Pi/180, a threshold of 15, and minimum line length and maximum line gap of 30 and 20 respectively. I then draw these lines on the original image using the original version of the draw_lines function. An example image is shown directly below. Note that I have shown the outline of the quadrilateral mask on this image in green.

![alt text][image1]

On this image you can see that the lane lines on the right are not solid. Similar behavior is seen on the 
image below which has white dotted lines on the other side of the road.

![alt text][image2]

### 1. A Description of My Modified Draw_Lines Function.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function to calculate the slope of each of the lane lines. From examining printouts of the slopes of the line segments that the Hough transform was returning, I could see that the slope of the lines on the right were positive and the ones on the left were negative. It was easy then to calculate the average of both the positive and negative slopes separately. I next calculated the average x and y coordinates for the left and right lines. With the slope and a point on the line, it was simple math using y=mx+b to determine the point for each line at the top and base of the masking quadrilateral. Using this algorithm produced the intended result as shown in the "solid white curve"image below. Note that the masking quadrilateral is now removed. 

![alt text][image5]

Upon looking at the Challenge video, however, it was clear that this algorithm was not sufficient. I took some stills from that video and the result using the algorithm described above is shown below.

![alt text][image6]

In order to understand what was going on, I reverted to using the unmodified, Hough line segments and using that on the same image shown above, produced the output shown below. The problem is easily visualized. Even with our masking quadrilateral there are still a lot of spurious edges detected in the video.

![alt text][image4]

The solution was to use a bandpass filter of sorts. Based on looking at the slopes of the line segments, it was clear that there are only certain slopes that could possibly equate to a lane line. This slope would vary depending on the camera used and how it was mounted, and it would vary as the road curved, but a slope near zero, for example, would be impossible. So, I added logic to filter out slopes except those within a reasonable range. I came up with a separate range for both the right and left lane lines. The values I used were: 0.4 to 0.7 for the right lane and -0.9 to -0.6 for the left lane. Tuning these for the challenge video were, the hardest because when the road curves, the slope of the lanes will vary and that variance will be greater, the tighter that the curve is. Upon implementing this algorithm, the same still image shown above looks as shown below and this is my final result. All of the other images also worked well.

![alt text][image8]



### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming of this approach is extremely tight curves. Take a look at the following curve which is the famous hairpin on the circuit for the Monaco Grand Prix. This is actually a public road as you can see here. I suspect that my algorithm would not work well on this particular road or at least the range of slopes for possible lane lines would have to be widened so much that it would reintroduce the spurious lines. If Udacity would like to pay for me to travel to Monaco to test if my algorithm would work and to gather video data that could be used in future classes, I am available for such work!

![alt text][image10]


### 3. Suggest possible improvements to your pipeline

One possible improvement would be to to make the masking region dynamic. In particular, on a tightly curved road, the lines closest to the car would have a slope within a well defined range, but the range should be wider the farther we got away from the car. So for detected lines closer to the car we may want to have a narrower range for the slope filter than we would apply for detected lines farther from the car. Also, in a real world application we could potentially use mapping data and GPS to determine when we were approaching a curve and to approximate the radius of the curve. Unfortunately, this would not work well in areas where the GPS signal was impeded by tall buildlings or other interference. Another solution would be to take advantage of the fact that the curvature of roads tends to change in a gradual way. In other words, if calculating the average slopes of the lines differs by too much from the previous video frame, we could apply a "smoothing" algorithm. 

Another potential improvement could be to ...
