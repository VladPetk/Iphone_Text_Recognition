# Collecting iPhone Usage Statistics From Screen Recordings

This a project for automated content analysis of screen recordings of iPhone usage settings. The script is used to analyze screen recordings submitted by participants of a study on phone usage and sleep quality. The script serves to extract app names and the corresponding duration of usage, the time indicators, notification count, etc.

The input is a .mp4 or .mov video file of an iPhone settings page (battery usage section). The output is a pandas dataframe for every video processed containing all relevant columns or a dictionary in case of a 'weekly overview' video. 

The code takes into account different versions of iOS (and thus different layout of the settings page), the possibility of 'night mode' being on, and the chance that a given video might be irrelavant. The code currently only works with screen recordings in English. 

##### The Process is essentially two-step:

First, an object-detection model was trained to recognize all the relevant information (with unique labels) in still images extracted from the recordings. The model was trained on a manually annotated set of images extracted from a sample of the available videos. It was then used to extract the recognized parts in the relevant frames of a given video (stored as numpy representations of the images).

Then, an optical character recognition model (Tesseract) was used to convert the text in the extracted images into strings. To improve the text recognition accuracy, similar images were processed to choose and the one with the highest estimated accuracy chosen. Since the same apps could appear at different times in the video, a sorting algorithm had to be build to assign the recognized apps, notifications, etc. to their corresponding timeframe.

##### The Files

The Model_training folder contains all the files that were modified from the orginal Tensorflow object detection repository.
The files from this folder should be put into the root folder of a repository below:
https://github.com/tensorflow/models/tree/079d67d9a0b3407e8d074a200780f3835413ef99/research/object_detection . 

The Jupyter Notebook file (Recognition_script.ipynb) contains the script used in the project. 

A list of required packages is to be added.
