# Transformation-Recovery-and-Image-Registration


## Task 1: Recovering Transformation Parameters: 
Given two images I1 and its transformed image I2 your task is to recover the Affine Transformation T that
when applied on I1 will give us I2.

1. Get correspondences between two images. One way of gathering correspondences is to create a
graphical interface asking user to click a point in one image and then asking to click for the same
point in second image (in Python you can use function matplotlib.pyplot.ginput).
2. From the given correspondences solve for the Affine Transformation by using pseudo inverse or
least square algorithm. If using pseudo inverse method each correspondence is going to give two
equations, which will form two rows of matrix. (Since there are 6 unknowns, how much
correspondences do we need? how many we need if we have to recover perspective
transformation?). To get good result don’t take all the points in small neighborhood, spread them
across image. You can use numpy or other libraries to find pseudo inverse.
3. Use recovered T to warp I2 into I1. Let I'2 be the warped image. You can use OpenCV to apply the
recovered transformation matrix on I2. See cv2.warpAffine()

How will we test if our recovery is good? We can calculate mean square error (MSE) of the pixel intensity
between warped image I'2 and image I

![equation](https://latex.codecogs.com/svg.image?MSE&space;=&space;\sum_{x}\sum_{y}(I(x,y)-I'(x,y))^{2})
or we can calculate mean square error between the corresponding points. For that you should have more
than required number of correspondences, use only subset to compute the T and test on others by calculating
distance between transformed points and corresponding points. Report both of these errors
Write a function like function prototype given below: 
```
  def recoverTransformation(Image1, Image2):
       #Image1 and Image2 are original and transformed input image respectively
       #MSEPix is mean squared error of pixels of T_image and Image2
       #MSECorPts is mean squared error of correspondence points and transformed points
       #T is recovered transformation
       #T_Image is transformed Image
       #Your code goes here
  return [MSEPix, MSECorPts, T, T_Image]
```
  
## Task 2: Now and Then

This task is an application of registration problem. Given pair of the images you have to transform one
image such that it overlaps its corresponding part in the other image. Use the previous method to recover
where should this image go (the code of part 1 will be used here to recover the transformation). You will
notice that you will need more than required points.

Also, here you will have to do masking, to allow pixels from one image in certain area to replace original
pixels but not in other areas. In case of affine transformation, it’s easy to apply transformation onto corners
of rectangle. If, however you are not assuming affine (perspective transformation is optional and can be
done by using cv2.warpPerspective()) then one easy way to do masking is take matrix of ones and transform
it similarly as you will transform image. Wherever this new transformed image is greater than zero place a
1 there. Now this mask could be used to choose pixels from one image in one part and from other image in
another part.

Write a function registerImages that takes input 2 arguments new image and old image and outputs a
registered image. 
```
def registerImages(ImageNew, ImageOld)
 #ImageNew and ImageOld are input images
   #registerdImage is the output image
   #Your code goes here (some steps are given below for your help)
     #1- get correspondence
     #2- recover transformation (use code of part 1)
     #3- Transform ImageOld to a new matrix of ImageNew size – use
   cv2.warpAffine() for affine and cv2.warpPerspective() for
perspective transformation.
       #4- Generate a mask of 1's and 0's (1's will represent where pixels of ImageOld will land and 0's will represent where pixels of ImageNew will land)
   return
registeredImage

```

![image](https://user-images.githubusercontent.com/105145104/168337224-2664b9db-9ea9-40f1-bbdb-c7914ecf63b2.png)
![image](https://user-images.githubusercontent.com/105145104/168337295-83b8c552-2fa8-47ac-bbf3-1c6c72dfe3eb.png)

## Task 3: Automated Image Stitching using SIFT Feature Matching

The purpose of this task is to stitch overlapping images acquired by a panning camera into a mosaic.
Two images from which we want to create a panorama or mosaic.

![image](https://user-images.githubusercontent.com/105145104/168337427-39db0a29-f3d0-4171-8888-b1829a8af8ad.png)

### SIFT (Scale Invariant Feature Transform)
SIFT finds and matches features between two images. The Scale-Invariant Feature Transform (SIFT)
bundles a feature detector and a feature descriptor. 

![image](https://user-images.githubusercontent.com/105145104/168337524-5d44da95-7e2d-4617-b98e-193ea80ac71f.png)

### Algorithm Outline

1. Input images I1 and I2.
2. Estimate the transformation between I1 and I2:
   i. Find local features using SIFT between in both I1 and I2.
   ii. Extract feature descriptor using SIFT for each feature point.
   iii. Find the correspondences between both images (Undergraduate students: use Euclidean
distance, Graduate students: use NNDR (Szeliski Book: section 4.1.3)).
   iv. Remove outliers with RANSAC.
3. Warp I1 to I2 and composite warped images into a single image

#### Each step is explained below:
### Extracting local feature and descriptors using SIFT: 
Both the keypoints and descriptor are accessible by OpenCV’s SIFT. We compute the SIFT frames (key points) and descriptors by using: 
```
noOfmaxKeypts = 800 # extracts (at max) 800 keypoints
sift = cv2.xfeatures2d.SIFT_create(noOfmaxKeypts)
[f,d] = sift.detectAndCompute(img, None)
```
f is a list of keypoints. Each keypoint has certain attributes which you can access as follows: 
![image](https://user-images.githubusercontent.com/105145104/168337949-2013bc17-b180-42fd-884a-8de6ab6b696b.png)

A frame is a disk of center f[0].pt, scale f[0].size and orientation f[0].angle. Repeat this procedure for all
the images. ‘d’ is the descriptor for each feature. For your ease, make an array whose shape is (4,N) where
N is total number of descriptors in an image. Each column will represent attributes of descriptor. Store the
attributes in order of x,y,radius and angle. So essentially you will be looping through all the keypoints and
extracting the attributes and then storing in the array. You will notice benefits of this array in coming tasks.
Next step is to find the matches between I1 and I2 images by Euclidean distance or NNDR. Results for two
images are given. (see Figure 2). You can show this using cv2.drawKeypoints(). 

#### Matching feature descriptors between two images using Euclidean Distance (Undergraduate Students) 
In this section you have to use Euclidean distance to find the matching feature descriptors between two
images. Apply distance formula on feature descriptor, one from I1 and one from I2. You can use built-in
functions to calculate Euclidean distance.

#### Matching feature descriptors between two images using NNDR (Graduate Students) 
In this section you have to use nearest neighbor distance ratio (NNDR) to find the matching feature
descriptors between two images. First read the topic in Szeliski Book section 4.1.3 (Feature matching).
Apply distance formula on feature descriptor, one from reference X and one from any other image, and
compute the nearest neighbor distance ratio. You can define this nearest neighbor distance ratio
(Mikolajczyk and Schmid 2005) as:

![equation](https://latex.codecogs.com/svg.image?NNDR=\frac{D1}{D2}=\frac{|D_{A}-D_{B}|}{|D_{A}-D_{C}|})

Where d1 and d2 are the nearest and second nearest neighbor distances, is the targets descriptor, and and
are its closest two neighbors. For more understanding, read the referred section from book. After this you’ll
get the best matches between given frames. For above two images, matching feature points are computed
using NNDR with a threshold of 0.8. These matches are given in .mat file.

#### Visualization:
You can also visualize the matches by drawing the lines between two images. One way to do this is keep
images side by side in a new array. The columns of the image on right will be offset by the number of
columns of the image on left. Then draw lines between corresponding matching pairs using opencv or
other libraries built-in functions. (See Figure 4) 

![image](https://user-images.githubusercontent.com/105145104/168339410-2cb53219-f0cb-4a0a-b982-7b2b0320c237.png)

#### Applying RANSAC to remove outliers
3-point RANSAC is used to remove outliers from the correspondence points. [4]
   1. Select three points randomly.
   2. Compute Affine Transformation T based on those points (use code you wrote in task 1).
   3. Transform the remaining points using T.
   4. Compare their transformed coordinates with the matching point coordinates in the second image.
   5. Remove outliers by using RANSAC and repeat the process by selecting the points again.
   6. Keep track of the largest set of inliers.
   7. Use them as the final set of correspondence points.
   8. Re-compute transformation using the inliers. 

![image](https://user-images.githubusercontent.com/105145104/168339734-1c0e5332-b870-45c2-bf9c-f8ea2a8d7e26.png)

**Warping and Blending images together for Panorama**
**Warping and Blending:** You are well familiar with the image warping. You can blend the overlapping image regions by using the maximum intensity values or average intensity value in order to obtain a flatten image. 
![image](https://user-images.githubusercontent.com/105145104/168339797-d8b9b058-24ed-4588-be12-41a951dde08d.png)




