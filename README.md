# Automatic-Panoramic-Image-Stitching
Automatic Panoramic Image Stitching using SIFT detector and descriptor,  RANSAC algorithm for best-fit homograpy, linear blending

You can find more details in my <a href="https://medium.com/@yungyung7654321/python%E5%AF%A6%E4%BD%9C%E8%87%AA%E5%8B%95%E5%85%A8%E6%99%AF%E5%9C%96%E6%8B%BC%E6%8E%A5-automatic-panoramic-image-stitching-28629c912b5a">chinese medium blog</a>


## Introduction

Panorama is now prevalent in the camera software of smartphones. It is a method in which we first take a number of images, and then stitch them together to form a bigger one, with significantly larger horizon. We are going to implement this algorithm and focus on two image stitching at a time.  


## Pipeline

### 1. Cylindrical Warping (Optional)

 - The scene in the real world of a panorama image is of the shape of a cylinder, but it is reduced to a flat 2D space after being captured by cameras.
 - Thus we have to re-project the image back to the cylinder space in order to prevent the result from being distorted.

![](https://i.imgur.com/tK2eE3g.png)

![](https://i.imgur.com/djmyj8j.jpg)
![](https://i.imgur.com/19kWHr6.jpg)

 - The two images shown above shows the effect of applying cylindrical warping before stitching. 
 - It can be esaily seen that the road in the rightmost part of the upper image is disappeared in the lower one.
 
**Using Cylindrical Warping has better stitching result in some scenario.*

### 2. Feature detection & Feature description 
Detect and extract the feature points in images using OpenCV's built-in **[SIFT [1]](https://people.eecs.berkeley.edu/~malik/cs294/lowe-ijcv04.pdf)** descriptor.

![](https://i.imgur.com/vIkmAC1.jpg)



### 3. Match the key points with David Lowe's ratio test.
 - For each key point, we find the matching pair of not only the smallest distance but also the second smallest.
 - If the difference between the two distance is smaller than a threshold, than this pair is abandoned and not used in further calculation.
 - Like the image shown below, the distance between f1 and f2 and the one between f1 and f2' is very close, meaning they are not distinctive enough. 
    
 
![](https://i.imgur.com/zOGQJ7z.png)
    
### 4. RANSAC to find homography matrix H
#### Homography matrix
Homography is an projective transformation, we use Homography matrix to transform one image pixel to another image.
![](https://i.imgur.com/kI06Tli.jpg)

Homography matrix is **H** and pixel is **p** in following figure:

![](https://i.imgur.com/m3xYDMc.jpg)

*Refer to [[2]](https://cseweb.ucsd.edu/classes/wi07/cse252a/homography_estimation/homography_estimation.pdf) for how to solve a homography matrix.

And we need to solve the best Homography matrix H by RANSCAN algorithm.
#### RANSAC
RANdom SAmple Consensus (RANSAC) is an iterative method to **estimate parameters of a mathematical model from a set of observed data**. It is a non-deterministic algorithm in the sense that it produces a reasonable result only with a certain probability, with this probability increasing as more iterations are allowed. 
The step of RANSAC as follow:
1. **Sample**(Randomly) the number of points required to fit the model (Homography)
2. **Solve** for model parameters using samples
3. **Score** by the fraction of inlier within a preset threshold of the model

Repeat 1~3 step, randomly sample points N times and get the heighest score solution.

Take fitting straight line as simple RANSAC example:

![](https://i.imgur.com/RxnfMBN.jpg)

In this image, we run the RANSAC two times (And Sample two point each time) to fit two straight line, and obviously line *ab* is better than *cd*.



### 5. Blending

![](https://i.imgur.com/vEyZJnD.png)

 - We first create a large image, whose width is the width of the two images for sitching combined, then the left image is copied and pasted to the new one.
 - Then for those pixels still having no values, we find its correspoding values in the right image by using the homography matrix found in Step 4.
 - But the boundary between the two images is very clear if we directly stitch them, as shown below.

![](https://i.imgur.com/rPci8oc.png)

![](https://i.imgur.com/rrtiepG.png)

 - We can therefore use the technique of linear blending to eliminate such boundaries. 
 - The idea is to find the overlapping area in the two images, and when calculating for a pixel in the area, different weights according to how close they are to each image are given. The closer the bigger weight, and the farther the smaller weight. 

## Result 

#### Two images:
![](https://i.imgur.com/nq5ZAri.jpg)
![](https://i.imgur.com/gz9qmRI.jpg)

#### Stitching image:

![](https://i.imgur.com/hHSVJO4.png)

## References
[1] Distinctive Image Features from Scale-Invariant Keypoints (SIFT), https://people.eecs.berkeley.edu/~malik/cs294/lowe-ijcv04.pdf

[2] Homography Estimation, https://cseweb.ucsd.edu/classes/wi07/cse252a/homography_estimation/homography_estimation.pdf

[3] M. Brown, D. G. Lowe, Recognising Panoramas, ICCV 2003, http://matthewalunbrown.com/papers/iccv2003.pdf

[4] Stanford Slide, https://web.stanford.edu/class/cs231m/lectures/lecture-5-stitching-blending.pdf

[5] NTU slide, https://www.csie.ntu.edu.tw/~cyy/courses/vfx/12spring/lectures/handouts/lec04_stitching_4up.pdf

