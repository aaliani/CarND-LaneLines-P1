# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on the work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of follwoing steps. 

* Convert the image to RGB for visualization purpose, since OpenCV Library reads in the image as BGR.
* Extract the patches of the image where the color is either yellow or white, corresponding to the color of lane lines. Although the recommended pipeline suggests to convert image to grayscale, that approach doesn't work well on the challenge.mp4 video simply because an RGB image has 3 channels vs 1 channel in the grayscale image, i.e. 3 times the information comppared to grayscale. A lot of information is lost when converting to grayscale. My pipeline leverages the yellow color information to give decent result on the challenge video.
* Apply gaussian blur to the image to smoothen the egdes out.
* Extract the edges using Canny Edge Detector algorithm.
* Remove the unnecessary edges by applying the region of interest mask.
* Apply Hough Transform to extract lines from the edge points.
* Draw the extracted lane lines on to the original image.
    

The Hough Transform outputs multiple edge lines for the lanes. In order to draw a single line on the left and right lanes, created another function, extrapolate_lines(), which takes in all the lines ouput by the hough transform, and gives just the two lines, the left and right lane lines as the output. The function divides the input line segments into sets of left and right lane line segments. It then created just one line in each of the sets that best all fits the line segments within the set using linear regression. It then extrapolates both lines to start at the bottom of the image and end close to the road vanishing point at 40% of the image height.

I modified the draw_lines() function to first call the extrapolate_lines() function of the output of Hough Transform adn then draw only two extrapolated lines on the original image.


### 2. Identify potential shortcomings with your current pipeline


This pipeline is created for and tested on relatively similar types of images. It could potenitaly break down when tested on other kinds of road images in a different envirnoment such as perhaps when it is snowing or raining.

The pipeline works well on the challenge video because I am using the color information. So the pipeline relies heavily on the white and yellow color for lanes. It would not work if lane lines are painted in any other color, or on the color values outside the defined threshholds for white and yellow.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to use the grayscale function after I have extracted the lane information using colors. This would make the algorithm more computationally efficient since in the subsequent steps of perfoeming the gaussian blur and then the canny egde detection, the algorithm would then be dealing with a single gray channel instead of 3 rgb channels, i.e. three times less processing required in each of the two steps.

Testing the algorithm on the challenge video gives an insight on its fragility. The algorithm runs smoothly on the two test videos. On the challenge video, however, although it gets the job done, one can clearly see the jitters from frame to frame, caused primarily by the changing lightening conditions. Such is a normal condition in real life and it clearly shows this algorithm is not robust for a real self driving car. In one frame for a split second, the right lane is completely off towards the left side of the car. In a real life situation, this could be catastrophic! In order to remedy that, it would probably be a good idea to develop a certain mechanism to propagate the knowledge of lane position and shape from frame to frame so that each new frame has some prior information to leverage that might prevent it from being completely off. Especially for lane detection, since one can be quite confident of the position of the lane for the next (few) frame(s), this could be built in as a safety mechanism to prevent false lane identifcation due to the bad data in one frame.

Another potential improvement could be to test the algorithm on different envirnments to identify other sources of shortcomings. 
