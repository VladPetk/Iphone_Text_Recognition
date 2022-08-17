# Automatically Collecting iPhone Usage Statistics from iPhone Screen Recordings

This a repository containing code for automated content analysis of screen recordings of iPhone usage settings. The code was used in a paper titled *[A Novel iOS Data Donation Approach: Automatic Processing, Compliance, and Reactivity in a Longitudinal Study](https://doi.org/10.1177/08944393211071068)* (Baumgartner et al., 2022). The script is used to analyze screen recordings submitted by participants of a study on phone usage and sleep quality. The script serves to extract app names and the corresponding duration of usage, the time indicators, notification count, etc.

The input is a .mp4 or .mov video file of an iPhone settings page (battery usage section). The output is a CSV dataframe for every video processed containing all relevant columns. The code takes into account different versions of iOS (and thus different layout of the settings page), the possibility of 'night mode' being on, and the chance that a given video might be irrelavant. The code currently only works with screen recordings in English. 

## Paper Abstract

First, an object-detection model was trained to recognize all the relevant information (with unique labels) in still images extracted from the recordings. The model was trained on a manually annotated set of images extracted from a sample of the available videos. It was then used to extract the recognized parts in the relevant frames of a given video (stored as numpy representations of the images).

Then, an optical character recognition model (Tesseract) was used to convert the text in the extracted images into strings. To improve the text recognition accuracy, similar images were processed to choose and the one with the highest estimated accuracy chosen. Since the same apps could appear at different times in the video, a sorting algorithm had to be build to assign the recognized apps, notifications, etc. to their corresponding timeframe.

## The Code

In order to allow quick, easy and accurate processing of the video recordings, a Python script was developed. This script processes iPhone screen recordings of the Battery Section pages and provides a CSV file with the following information: which apps were used at each hour of the day, time in minutes that each app was open on screen per hour, and time in minutes an app was open in the background per hour. This is accomplished by (1) transforming the input video into a subset of (key) frames (as numeric arrays), (2) performing object detection to extract the relevant fields (app name, time indication, etc.) in each frame, (3) sorting and removing irrelevant fields, (4) running optical character recognition (OCR) to transform the detected fields into text data, and (5) processing the text data to re-combine and sort the relevant fields for the CSV output. The script currently processes videos based on the English or Dutch version of iOS.

### Video to Frames Conversion (1)
The videos are read using the OpenCV library and loaded into memory frame by frame as NumPy arrays. Since consecutive frames are likely to contain identical information, the script compares the information in each frame to the information in the subsequent frame and records the absolute difference between the two. This is accomplished by converting copies of the frames into grayscale, applying Gaussian blur (as to minimize the effect of artifacts), and subtracting one frame array from another. Then the baseline in the resulting array of differences is estimated and the values sufficiently distant from the baseline are indexed. These indices are used to select the relevant frames from the initially read NumPy arrays. Currently, the sufficient distance from the baseline is estimated automatically such that the number of relevant frames returned by the function is at least twice the length of the screen recording in seconds.

### Object Detection (2)
Once a selection of relevant frames is made, the frames are run through an object detection model to detect the following two fields: indications of which hour is being shown and apps with their logos, names, and durations of activity on screen and in the background. To detect these fields, transfer learning was utilized to finetune a pre-trained TensorFlow model on manually annotated screen captures (screenshots, N = 247) from a random sample of videos (N = 31). The training was initialized using a pre-trained model (Faster RCNN Inception v21) and trained for 170,000 steps on the new data (annotated captures). The script calls the trained model and processes all frames in one TensorFlow session outputting detection boxes (coordinates of the detected fields), types of fields, and detection confidence for each field.

### Sorting and De-duplication (3)
Then OCR is performed on the hour indication fields to: (1) detect frames that show app usage not by hour but for the given day overall and remove them, and (2) sort the detected app fields into their corresponding hours. The sorting is done by grouping together all the app fields that appear either in the same frame as the hour indication field or—in case the hour indication field is not in the frame—the app fields in all frames that precede a frame with an hour indication field showing a different hour. Thereafter, an OpenCV implementation of the scale-invariant feature transform (SIFT) algorithm is used to remove duplicated app fields within each hour to speed up the subsequent steps.

### Optical Character Recognition (4)
Another instance of the object detection mode is run on the previously detected app fields to extract the image coordinates of the app name and the time for which it was used in the given hour. As before, to do so, a TensorFlow model was finetuned on manually annotated app fields (N = 130) using a pre-trained model (Faster RCNN Inception v22). Separating the app name and usage time is done to improve the OCR accuracy. OCR is performed on each detected field (either the app name or usage time) using PyTesseract3—a Python implementation of the Tesseract OCR engine. Prior to supplying the images to Tesseract, they are pre-processed as to improve OCR accuracy: the images are enlarged, transformed into black and white, and blurred. The exact parameters for these pre-processing steps differ depending on the field being processed.

### Data Sorting (5)
Finally, the script combines all information into a data frame for the CSV output. The combined data frame has four columns: time of the day (by hour), app name, app time on screen, and app time in background. When the output of all the videos is combined into one dataset, fuzzy string matching is performed to account for minor OCR mistakes and standardize the app names in the dataset. The string matching is done by using simple ratio with the threshold of 90%; that is to say, all app name strings that share more than 90% of characters in the same order are considered to be identical and are standardized. The 90% threshold was chosen by trial and error with lower values resulting in the script erroneously considering different apps to be the same (e.g., ‘email’ and ‘mail’). The remaining steps of the script concern data cleaning: the indications of hours are transformed so that their format is consistent across all processed videos; the time indicators are stripped of all characters but digits (e.g., “1 min” becomes “1”); any duplicated entries are removed; finally, all faulty entries (e.g., NAs in the hour or name fields) are removed.

## The Files

The Model_training folder contains all the files that were modified from the orginal Tensorflow object detection repository.
The files from this folder should be put into the root folder of a cloned repository below:
[The link to the Tensorflow object detection repository](https://github.com/tensorflow/models/tree/079d67d9a0b3407e8d074a200780f3835413ef99/research/object_detection)

Frozen inference graphs are also inlcuded so that the model does not have to be trained again. 

The Jupyter Notebook file (Recognition_script.ipynb) contains the script used in the project. 

The images used to train the model were not upload due to their size. Please contact me if you would like to use them. 


