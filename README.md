# SFND 2D Feature Tracking

<img src="images/keypoints.png" width="820" height="248" />

The idea of the camera course is to build a collision detection system - that's the overall goal for the Final Project. As a preparation for this, you will now build the feature tracking part and test various detector / descriptor combinations to see which ones perform best. This mid-term project consists of four parts:

* First, you will focus on loading images, setting up data structures and putting everything into a ring buffer to optimize memory load. 
* Then, you will integrate several keypoint detectors such as HARRIS, FAST, BRISK and SIFT and compare them with regard to number of keypoints and speed. 
* In the next part, you will then focus on descriptor extraction and matching using brute force and also the FLANN approach we discussed in the previous lesson. 
* In the last part, once the code framework is complete, you will test the various algorithms in different combinations and compare them with regard to some performance measures. 

See the classroom instruction and code comments for more details on each of these parts. Once you are finished with this project, the keypoint matching part will be set up and you can proceed to the next lesson, where the focus is on integrating Lidar points and on object detection using deep-learning. 

## Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./2D_feature_tracking`.

## WriteUp about the rubric points in this project

* MP.1 Implement a vector for dataBuffer objects.
  * Here I use the predefined DataFrame to load the image into data buffer, and then check if the data buffer size is larger than the predefined size. If it is, that means the buffer is full and the first image in the buffer need to be erased. In this method the data buffer will be kept in a certain size, which means there will be a certain number of images in memory. That ensures the whole program not being slow down.

* MP.2 Implement detectors HARRIS, FAST, BRISK, ORB, AKAZE, and SIFT and make them selectable.
  * Here I use `.compare()` to compare the string of the detector name, in order to select the corresponding detector. In which, FAST, BRISK, ORB, AKAZE, SIFT will be further chosen in function `detKeypointsModern(...)` in `matching2D_Student.cpp`.

* MP.3 Implement a rectangle Region of Interest, remove all the keypoints outside of this rectangle. 
  * In order to focus on the preceding vehicle, we define the region of interest as a rectangle shape in the center of images. Here I use `cv::Rect` to define the rectangle and then implement a for-loop to traverse all the keypoints and check if the keypoint is in the rectangle or not. If so, `push_back()` the keypoint; if not, ignore it. 

* MP.4 Implement descriptors BRIEF, ORB, FREAK, AKAZE and SIFT and make them selectable.
  * This task is similar like the task in MP.2. First use `.compare()` to compare the string of the descriptor name and then use `cv::AKAZE::create()`, `cv::xfeatures2d::FREAK::create()`... to create a corresponding descriptor.

* MP.5 Implement FLANN matching and K-Nearest-Neighbor selection.
  * If we choose the matcher FLANN, then it should be created in `matching2D_Student.cpp` as following
  * `cv::DescriptorMatcher::create(cv::DescriptorMatcher::FLANNBASED);`
  * But here may throw an error `Aborted` because of unmatched data type. So we need to add an IF-statement to check if the type is CV_32F, if not, use `**.convertTo(**, CV_32F)` to convert it to CV_32F before create the matcher.
  * For KNN matcher, we use the predefined function in OpenCV `matcher->knnMatch` to choose k best matches for each descriptor from a query set.

* MP.6 Use the K-Nearest-Neighbor matching to implement the descriptor distance ratio test.
  * In order to accept the nearest neighbor as a good match, here we use a descriptor distance ratio to test. Because we expect a good match to be much closer to the feature than the second best match. If both features are similarly close to the query, we cannot decide which one is really the best one. So here we use a threshold 0.8 in `matching2D_Student.cpp` to test these matches.
  * In `matching2D_Student.cpp`, we traverse all the matches and use IF-statement as following:
  * `if ((*it)[0].distance < (*it)[1].distance * minDescDistRatio)`
  * if so, `push_back` the better match `(*it)[0]` into `matches`.


* MP.7, MP.8, MP.9
  * For the convinience of logging all the data we need in MP.7.8.9, we use 2 For-Loops to traverse all the detector types and descriptor types, then contain the previous main loop over all images. 
  * In order to achieve that, we need to define a string vector for detector types and a string vector for descriptor types before the 2 For-Loops. Then create csv files with output file stream as following:
  * `ofstream logKeyPoints("../MP7_KeyPoints.csv");`
  * `ofstream logMatchedPoints("../MP8_MatchedPoints.csv");`
  * `ofstream logDetDescTime("../MP9_DetDescTime.csv");`
  
* MP.7 Count the number of keypoints on the preceding vehicle for all 10 images.
  * After processing with each combination of detector and descriptor, log the number of keypoints as following:
  * `logKeyPoints << ", " << keypoints.size();`
  
* MP.8 Count the number of matched keypoints for all 10 images using all possible combinations of detectors and descriptors.
  * After matching keypoints with the given matcher type, selector type and descriptor, log the number of matched keypoints as following:
  * `logMatchedPoints << ", " << matches.size();`

* MP.9 Log the time it takes for keypoint detection and descriptor extraction.
  * use `(double)cv::getTickCount()` to get the time before and after processing and then divided by `cv::getTickFrequency()` to calculate the processing time of keypoint detection and descriptor extraction and then log it as following:
  * `t = ((double)cv::getTickCount() - t) / cv::getTickFrequency();`
  * `logDetDescTime << ", " << 1000 * t ;`
