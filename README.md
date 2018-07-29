# CrashCatcher
<b>A Dashboard Camera Accident Detector</b>

<img src="https://github.com/rwk506/CrashCatcher/blob/master/imgs/frontpage1.png" alt="CrashCatcher">

<img src="https://github.com/rwk506/CrashCatcher/blob/master/imgs/frontpage2.png" alt="CrashCatcher">

Dashboard Camera video is becoming more and more common these days - whether it's ride-sharing companies, police departments, self-driving cars, or insurance companies, there are millions upon millions of hours of video being generated each month. But, accidents are rare, meaning most of this video is irrelevant to organizations and their goals. That's where Crash Catcher comes in - it can sift through this video and detect when a video clip contains an accident, saving time and money.

Visit the Crash Catcher website (website no longer running) to see the algorithm in action with example videos, or upload your own dashcam footage to see how it works.

The readable markdown of the code is [here](https://github.com/rwk506/CrashCatcher/blob/master/HRNN_training.md) though the full python code is also available for download as a .py file.

<br />

<h4>Table of Contents</h4>

[Summary](#Summary)<br /> 
[Example](#Example)<br />
[Data and Processing](#Data)<br />
[The Algorithm](#Model)<br />
[Documentation](#Docs)<br />
[Resources](#Resources)<br />
[Other Information](#Other)<br /><br />

<a name="Summary"/>
<h3>Summary</h3>

Advanced machine learning techniques are employed to compare video footage containing accidents to video free of crashes. Minute differences can be found using a hierarchical recurrent neural network from a labeled video set. These differences can then be used to find accidents in never-before-seen videos.

<br /><br />


<a name="Example"/>
<h3>Examples</h3>

<b>Insurance Companies</b>: With drivers recording millions of hours of video weekly, it becomes necessary to only save video as necessary. Automatically screening dashcam footage for anomalous events is an easy way to reduce storage costs and to save video relevant to their clients' insurance rates.

<b>Police Departments</b>: Dashcam footage from patrol generates more data than most police departments are equipped to handle, and is often deleted at regular intervals. If specific footage from the road is needed weeks or months later for a case, this can be a problem. Automatic screening for accidents can reduce the effort needed to retain relevant footage.

<b>Legal Proceedings</b>: Video evidence is often persuasive in investigating the reliability of testimony and in determining who is at-fault in an accident. Automatic Accident Detection will save time and energy when searching for footage relevant to the investigation at hand.

Of course, there are many other applications as well, from self-driving cars to ride-sharing companies to truck or auto delivery companies. Many research groups are working on 

<br /><br />



<a name="Data"/>
<h3>Data and Processing</h3>

<b>The Data</b>:

Over 250 videos (over 25,000 individual frames) are leveraged to build the hierarchical recurrent neural network model. Each clip is from dashboard camera footage taken from automobiles. The videos are 4 seconds long each and are derived from two data sources. One is the private ACCV dataset (VSLab Research), which largely populates the "negative" examples of driving through diverse road scenes without any accidents. This dataset also contains 4 second segments of additional footage that does contain car accidents. To supplement these videos, I scraped extra youtube dashcam footage into four second clips, focusing in particular on head-on collisions.


<b>Training Dataset</b>:

The ACCV dataset contains more than 1000 videos without crashes and more than 600 with crashes. However, many of the negative (non-crash) videos show the same driving scene -- this set was culled to have unique videos and to remove videos with logos, intro scenes, etc. This left 439 non-crash videos of serene driving.

Of the 600 videos with accidents, the types of accidents were exceptionally diverse, including any combination of pedestrians, bicyclists, mopeds, cars, trucks, etc., many of which were far in the distance of the dashcam footage. Even after weeding out non-auto and distance crashes, the wide variety of the scenes made training a model difficult. With 36 of these videos showing head-on collisions involving cars, I supplemented the data by turning to YouTube, where I found and extracted 4-second clips for 93 examples of head-on collisions.

For the final dataset, I had a total of 129 positive videos (those with crashes), and randomly sampled an equal number of negative videos (those without a crash) to ensure balanced classes.

<b>Processing</b>:

Each video is broken up into its individual frames to be analyzed separately. Each of these images is a two-dimensional array of pixels (1280x720) where each pixel has information about the red, green, and blue (RGB) color levels. To reduce the dimensionality at the individual image level, I convert the 3-D RGB color arrays to grayscale. Additionally, to make the computations more tractable on a CPU, I downsample each image by a factor of 5 - in effect, averaging every five pixels to reduce the size of each image to a 2-D array of 256x144 (as opposed to the initial 3-D array of 1280x720x3). (This sequence of events is illustrated in the image below).

Additional processing was explored for this project, such as median-subtracting out the background of the images and featurizing each frame in the image. The latter was done with the pre-trained inception and imagenet models (these are models that largely rely on convolutional neural networks to classify images based on a reduced set of extracted features from the images). These approaches did not produce an discernible improvement and were left out of the final product.<br/>

<img src="https://github.com/rwk506/CrashCatcher/blob/master/imgs/videotoimg.png" alt="Data Processing">

<br /> <br />



<a name="Model"/>
<h3>The Algorithm:</h3>

A hierarchical recurrent neural network algorithm is used to tackle the complex problem of classifying video footage.

<br />
<b>The Algorithm</b>:

Each video is a data point that either does or does not contain a car accident. However, each video is a set of individual images that are time-dependent sequences. The algorithm I've chosen - a hierarchical recurrent neural network - is able to treat each video as a time-dependent sequence, but still allow each video to be an independent data point.

The algorithm uses two layers of long short-term memory neural networks. The first neural network (NN) is a recurrent network that analyzes the time-dependent sequence of the images within each video. The second takes the encoding of the first NN and builds a second NN that reflects which videos contain accidents and which do not. The resulting model enables a prediction of whether new dashcam footage has an accident.

Through this method, the HRNN incorporates a time-dependent aspect of the frames within each video to predict how likely it is a new video contains a car accident.


<b>Training and Validation</b>:

The 258 video set was used with the HRNN for the classification of crash/not crash. 60% of the dataset was used to train the model and 20% to validate. I iterated with the training and validation sets repeatedly to test various setups of the algorithm and model parameters.

The hyperparameters - the number of layers in the neural network layers, the batch size, the number of epochs - were adjusted using a comparison to a control run (as a grid search was not feasible in the alloted timeframe). As an extra sanity check, I created fake data by populating images with random numbers and running the fake data through the model, achieving results equivalent to random chance. The final model uses a categorical crossentropy loss with a NAdam optimization and 128 layers in each neural network.

In the end, the ROC (receiver operating characteristic) curves were built with the final 20% of the data - the test set (as seen in the image above to the right). The area under the curve is 0.80 and the overall accuracy is ~81%, well above random chance (the dotted black line in the plot below).

Other approaches were explored initially, including logistic regression and basic NN to classify and predict based on hand-labeled frames (crash vs. not crash) from five longer (>30 seconds) YouTube videos merged together. The model performed excellent on a randomly selected hold-out set of frames from within these images, but was unable to generalize to any new test video, again showing how a lack of data heavily weighs on this problem. Additionally, unsupervised clustering on these same hand-labeled frames was tested on a single video. The goal was to determine if the frames containing an accident within these longer videos were distinct from the rest of the non-accident frames. While the accident frames did appear to occupy different clusters than most of the rest of the video, it was not a unique mapping; other events in the video (e.g.: going through a tunnel) prompted similar clustering behavior. <br />

<img src="https://github.com/rwk506/CrashCatcher/blob/master/imgs/roc_curve.png" alt="ROC Curve">


<br />
<b>Next Steps and Broader Applications</b>:


The ultimate goal is to build a model with the data I have to make predictions for a wider variety of situations. In general, neural networks rely on training on a LOT of data (thousands of data points - this case videos) to be able to accurately predict or classify in a new situation. Essentially more data = a more generalizable model.

There are a few ways to achieve this goal. One is to simply gather more data - however this is a heavily tedious, time-intensive task for humans. Another possible solution is to generate "new" data by slightly altering the data we have. This means applying rotation, horizontal flips, changing image quality, or other variations on the images within each video. Although most humans would recognize the altered videos as being essentially the same, it looks like different and new data to the algorithm. Repeating these alterations over and over to the same set of videos generates a larger dataset and will improve the generalization of the predictions to never-before-seen data.

Additionally, a true grid search would be beneficial - this would check various combinations of model parameters to find the best-performing set. Improvements in accuracy may be gained from a combination different optimization techniques, loss/cost functions, or activations of the neural net weights. Some of these were searched in this project (e.g.: Nadam, Adam, rmsprop optimizations; softmax, softsign, sigmoid acitvations; binary, hinge, logcosh losses; regularization), but the problem would benefit from a more comprehensive search.

This algorithm is best implemented at scale. The analysis already splits input longer videos into 4-second chunks as the video is read in. The short segments can be independently, simultaneously screened by the pre-trained crash-catcher algorithm by parallelizing across multiple GPUs/nodes. The segments that contain crashes can be saved and the rest removed from storage.
<br /><br />



<a name="Docs"/>
<h3>Documentation</h3>

There are three main files important to this project included here:
<ul>
<li><b>HRNN_pretrained_model.hdf5</b>: The model trained on the video dataset to predict crash/not crash. Can be loaded, compiled, and used to make predictions, as seen in HRNN_training.ipynb
<li><b>HRNN_training.py</b>: This file walks through the process of loading and prepping the dataset, setting up and training the HRNN model, and examining the results.
<li><b>YouTube_scraping.ipynb</b>: This short script contains a few functions to help download videos and scrape 4 second clips of dashcam video.
</ul>

<br /><br />


<a name="Resources"/>
<h3>Resources</h3>

<ul>
<li> <a href="https://github.com/fchollet/keras/blob/master/examples/mnist_hierarchical_rnn.py">HRNN for the MNIST Dataset for Handwritten Number Recognition</a>
<li> <a href="https://arxiv.org/abs/1506.01057">A Hierarchical Neural Autoencoder for Paragraphs and Documents</a>
<li> <a href="http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7298714">Hierarchical recurrent neural network for skeleton based action recognition</a>
<li> <a href="https://aliensunmin.github.io/project/dashcam/">AAVC Dataset from VSLab</a>
</ul>

<br /><br />


<a name="Other"/>
<h3>Other Information</h3>

I am a Ph.D. Astronomer turned Insight Data Science Fellow. Please feel free to reach out on [LinkedIn](www.linkedin.com/in/rawagnerkaiser/) or check out my other work on [GitHub](https://github.com/rwk506).

<br /><br />



