# Vehicle Detection From Video Imgae


The goals / steps of this project are the following:
* Demonstrate the benefits of Histogram of color as a feature in vehicle detection.
* Demonstrate the benefits of Spacial Bins as a feature in vehicle detection.
* Demonstrate the benefits of HOG-Histogram of Oriented Gradients as a feature in vehicle detection.
* Visualize the benifits of normalization
* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected. Soure(2)

[Link to Final Output Video](https://youtu.be/BbSBmZg715k)


## Intorduction

In order to be able to detect a vehicle near a car each image form a forward facing camera is scanned with a sliding window to identify windows with car or not-car images. Techniques are used to evaluate the window images. These techniques were HOG-Histogram of Oriented Gradients,  Spatial Bins and Histogram of color. Many color spaces were tested.

Note: A modified version of these functions were used to visualize the benefits each of these techniques. The modification enable visualizing any combination of the 3 channels of the color space. These modified functions are in the mod_functions.py folder.

The predictive classifier below used functions to process all 3 channels in the attempt to capture as much information as possible. These are the  functions listed in the 'Building the Classifier' section below.

## Spacial Binned Color Features
Image pixels were broken down to spacial bins. The image was resized and flattened. The visualization below shows the difference in spacial bins for the car and not-car samples.
Examples in jupyter notebook

## Color Histogram 
Histograms of the channels of an image. Each channel value is put in one of 32 bin, one set of bins for each channel of the color space. Below shows the 3 histograms. Below the data pattern of the 3 histograms for cars and the 3 histograms for non-car shows a difference between car and not-car. Again, these can be used as features in predicting if a sliding window across and image is car or not-car.
Examples in jupyter notebook


## HOG- Histogram of Oriented Gradients

HOG consists of:
- (optional) global image normalization
- computing the gradient image in x and y
- computing gradient histograms
- normalizing  across blocks
- flattening into a feature vector
source(1)

Examples in jupyter notebook



## Extract Features Function and Benefits of Normalization

The extract features function gets all the information from the HOG, Spacial Bin and Color Histogram method mentioned above. 

Examples in jupyter nootbook shows the benefit of normalizing the data.


## Classifier Functions and Methods

The classifier below uses HOG, spatial bins and color histogram method to get features to use to predict which of the sliding windows contained a car image and which did not. Multiple color spaces were evaluated and YCrCb was used because it showed good differenciate between car and not-car with the visualization methods above and had good preliminary results with my first version of the classifier algorithm. 


#### Histogram of Oriented Gradients (HOG)
Many HOG variables were tested with the following settings giving fewer false positives. 
set_orient = 9
set_pix_per_cell = 4                    
set_cell_per_block = 4
I thought fewer set_pix_per_cell would improve the resolution of the HOG image. This did reduce false positives but did increase run time.

Spatial binning dimensions were increased to (64, 64) and the number of histogram bins was also increased to 64.
I thought the increase would increase the differentiation between car an not-car. It did improve the results but also increased the run time.

These are in the cell 'Classifier and Settings' and starts with '*****HERE*****'.


#### Classifier 
The features were scaled and ran in a linearSVC classifier. I included a C=0.001. This decrease the number of false positives.
The classifier code is also in the cell 'Classifier and Settings' below.


#### Sliding Windows
A sliding windows, of different sizes((150, 100),(150, 150),(200, 150),(300, 150)) and shapes(squares and rectangles), were used to get a portion of the image. All sliding window images were evaluated by the classifier to determine if it had a car in the image. There were many false positives and many overlapping  true positives. A 50% overlap was used.

#### Class Object
I used a few methods to sanity check if a box was a false positive or a true positive. I used a class object call Windows to keep track of the boxes(windows) in the previous n images. 
Within the class was a dictionary containing: 

* previous_image_center_ys contains the center y value of the boxes
* previous_image_center_xs contains the center y value of the boxes
* previous_image_volumes contains volume of the boxes
* previous_image_windows contains the dimensions of the boxes

#### Sanity Check and Filtering
The dictionary contained a list of these values for each of the last 'n' image frames. This allow me to create the sanity check to remove some of the false positive flickers. A threshold 'm' was also set.
So the sanity check code asked:

* Does the current box have previous box or boxes within x pixels in the past n images?
* If it had 'm' boxes in the same area over the last 'n' images it is probably a true positive
* If it is a true positive I would get the box of the past n images with the greatest volume to use for this image.


Also, to reduce the overlapping boxes over the images of the cars in the video I wrote a function to merge some of the overlapping boxes. If two boxes were over lapping the dimensions were combined to make one bigger box. 


#### Discussion and Future Considerations
Here I used HOG, Spatial Bins and Color Histogram to create a features used to predict if a small section or window of a video image contained an image of a car. Many setting were attempted with each function. A linearSVC classifier was used. It predicted true positives but also produced many false negatives and false positives. I used a sanity check of if it had boxes in the same area in some of the past few images it was probably a true positive. Also I did not want a small box within a car image so if it satisfied the sanity check I added a box equal to the volume  of the box in the last few images with the biggest volume. Also, to reduce overlapping boxes on the car images I wrote code to merge two boxes if it  both satisfied the sanity check and overlapped.


Moving forward this classifier could be improved by simplifying the code. Is take over 2 second per video image. To decrease this time I could remove many of the 'for loops' and replacing with vector operation if possible. Also to save processing time building the classifier that uses only the best 2 channels of the color space to train and predict.  There are many overlapping boxes on the cars images. The code used to merge the boxes loops over all the boxes in an image and looks for overlapping, then merges. The problem is the new merged box does not get checked for overlap with other boxes. A possible solution would be to make this code into a function then loop over the function the will merge newly merged boxes. Better yet use the heat map to identify area of overlapping boxes to predict the locations of cars.

A heatmap was added in the latest version to better detect vehicles.






