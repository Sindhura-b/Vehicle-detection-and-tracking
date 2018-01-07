## Vehicle Detection Project

---

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, Apply a color transform and append binned color features, as well as histograms of color, to HOG feature vector. 
* Normalize the features and randomize a selection for training and testing.
* Implement a sliding-window technique and use trained classifier to search for vehicles in images.
* Run the pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_non-car.png
[image2]: ./output_images/heatmap_hogsubsamp.png
[image3]: ./output_images/heatmap_sliding.png
[image4]: ./output_images/sliding_window.png
[image5]: ./output_images/hog_features.png
[image6]: ./output_images/hog_subsampling.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The main code for this step is contained in the second code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example of HOG visualization for a car and non-car image using the `RGB` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image5]

#### 2. Explain how you settled on your final choice of HOG parameters.

The parameters that are tuned for training and testing the classifier are color space, oreintation, pixel per cell, cells per block, spatial size, histbin, hog features, spatial features and histogram features. I tried various combinations of parameters and found that the testing accuracy (0.97 to 0.99)  and time taken to classify (40 to 90 sec) accordingly. After alot of experiementation and based on the discussions in Udacity's forum, I understood that color space is the dominant parameter in detecting vehicles and choosing 'YCrCb' or 'YUV' yielded best results on frames in a test video. Following table shows my final parameter set values used for training ad testing.

| Parameter        		|     Value	        					| 
|:---------------------:|:---------------------------------------------:| 
| Color space       | YCrCb   |
| Orientations      | 11   |
| Pixels per cell      | 16   |
| Cells per block      | 2   |
| Spatial size      | (32,32)  |
| Hist bins      | 32  |
| Hog features | True |
|Histogram features | True |
|Spatial features | True |

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

Training basically involves extracting features from the training set and supplying these features to the training algorithm along with corresponding labels. The labeled data set for this project is provided by Udacity and contains vehicle (8972) and non-vehicle (8968) images. I chose to combine all the features from HOG and color classification instead of relying on a single method of feature extraction. Also, the classifier chosen in this project is linear kernal Support Vector Machine (SVM), which has been shown to perform well with HOG features. However, I haven't applied GridSearchCV for choosing best paramter combination and C is left for this default value. The data is nomalized using `StandardScaler` method, to avoid individual features or sets of features dominating the response of classifier. After this, the data is shuffled and split using `train_test_split` function, with 4:1 ratio respectively, and supplied to the classifier. Using the above parameter values as input and above described pre-processing steps, I trained my linear SVM classifier which resulted in a testing accuracy of 0.987. 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

Sliding window search involves stepping across an image in grid pattern and extracting the same features the classifier is trained on at each window. Then, the classfier is run at each window step to predict the presence of Car in that window. The whole process of sliding window search is implemented using functions `sliding_window`, `single_img_features` and `search_windows`. The parameters effecting the sliding window search implementation are window size and window overlap. A window size of (96,96) and overlap of (0.75,0.75) produced the best results for detecting cars in a given image. Windows with vehicles detected in test images using sliding window search are shown below.

![alt text][image4]

I also explored HOG subsampling, in which HOG and color features are extracted only once, and is a more efficient method for doing the sliding window approach. The code for performing HOG subsampling is written in `find_cars` function using Udacity lecture notes. This function requires tuning only one parameter called scale factor to determine the window size and over lap. I ran this function multiple times for different scale values ranging from 1.0 to 2.5 in steps of 0.2 to generate multiple-scaled search windows. Windows with vehicles detected in test image-4 using HOG subsampling approach are shown below.

![alt text][image6]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I searched only for a single window size and overlap using sliding window search approach. However, I searched on multiple scales in HOG subsampling approach using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector. Both methods yielded satisfactory vehicle detection results.

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result from sliding window search](./project_video_slidingwin_output.mp4)
Here's a [link to my video result from sliding window search](./project_video_hogsubsamp.mp4)

#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions. The value of heat threshold chosen in this project is 2. I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmaps of test images using sliding window search approach, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the test images:

![alt text][image3]

Here's an example result showing the heatmaps of test images using HOG subsampling, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the test images:

![alt text][image2]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

