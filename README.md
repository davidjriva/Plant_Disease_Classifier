# Plant_Disease_Classifier
## Introduction

The core focus of our term project is addressing the pressing issue of plant diseases that result in an annual economic loss of $220 billion globally(Gula). The challenges lie in early detection and prevention, a crucial hurdle for agriculturalists with vast crop spans and the swift, unnoticed multiplication of diseases. Our project’s primary goal is to develop a robust software-based solution that accurately identifies and flags diseased plants, offering early warnings to agriculturalists for proactive disease control in diverse environments. 
This effort is significant in the realm of Big Data due to the complexity of classifying a multitude of plant diseases swiftly, accurately, and with precision. Leveraging a dataset encompassing approximately 30 plant classes, including healthy and diseased specimens, totaling around 1.5GB, we aim to harness Big Data technology to create a classifier that doesn’t mislead agriculturalists and is robust against natural factors. This project’s importance lies in providing farmers worldwide with a tool to accurately determine if their crops are diseased in order to take action against the spread within their crops.
Our project was sparked by a collective interest in deep learning, data augmentation, and biological datasets. Plant diseases became a focal point due to the availability of a relevant dataset and the problems it presented. Past research at Massey University, New Zealand, in 2019, explored plant disease classification using deep learning models such as AlexNet, GoogLeNet, and ResNet (Saleem). This study highlighted gaps in existing research, emphasizing the need for more realistic datasets encompassing a variety of environmental conditions.
	In summary, our project aims to fill these gaps by leveraging Big Data and deep learning tools like Spark and Keras to create a robust classifier capable of accurately identifying plant diseases, offering a vital solution for agriculturalists globally.

## Methodology

  Our modeling goal is to discern intricate details within plant imagery and make informed decisions for multiclass image classification. To tackle the challenge of multi-class image classification, distinguishing between healthy and diseased plants across a variety of species, our approach commenced with comprehensive background research.
Our exploratory analysis of the dataset revealed peculiarities along three dimensions: the idealized quality of the images, the types and distribution of the augmentations previously applied to the images by their source on Kaggle, and the existing dataset partitioning for evaluation. We characterize the samples as “idealized” due to the images' centered and isolated specimen-like presentation of the leaves. This staging is contrary to how one might expect to capture images of the leaves in the field, where one would likely capture various kinds of occlusions and background noise. The augmentations previously applied to the images by their source appear to address some of the variance expected at inference time but are limited and unevenly applied across the classes. Finally, we discovered that the existing dataset partitioning appears to be a random split between a training and validation set, without consideration for whether an outcome of augmentation and its source are colocated.
	Each of the three conclusions that we drew from our data analysis influenced our data preparation and experiment design in its own way. First, since the images are so carefully staged, we feared that any test set drawn from the existing data might not generalize to an agricultural field environment. To test this hypothesis, we composed a new test set by introducing two varieties of augmentation via simulated occlusions, namely rain and mud spatter. We describe the results of this line of experimentation in the "Occlusion Augmentation'' experiments section. Second, to preserve even class distribution and the proportions of source images in the original dataset, we layer our new augmentations onto any existing ones rather than working with the source images alone. Thereby, despite certain classes being wholly of unaltered images and other classes being predominantly altered, we maintain these underlying distributions in our new dataset. Third, we suspected that the random validation split in the original dataset constituted "group leakage" (Ayotte, 2021), where each group consists of a source image and all images that were augmented from it. We manually reviewed these image groups and it was clear that they were indeed visually correlated. To test our hypothesis that a split without consideration for image groups might leak information into the target, we compared stratified partitioning to a group-based partitioning scheme. This experiment is further discussed in the "Group Partitioning'' experiments section.
After evaluating several models, we settled on using the VGG16 model from Keras, a Convolutional Neural Network well known for its exceptional performance in computer vision. Its architecture comprises 16 convolutional layers and 138 million parameters, which suits the complexity of our classification problem. VGG16’s capability to handle such complexity while training lightweight enough for compute resource-restricted environments makes it an ideal choice. Despite VGG16 typically expecting input images in a 224x224x3 format, we’ve noted that our images are in a 256x256x3 format, which can be adjusted to match the model’s requirements. 
To initiate our model training, we began with the VGG16 (Simonyan) model and loaded the pre-trained ImageNet weights. The top layers were removed as they are specific to the 1000 classes ImageNet represents (Deng). We froze the remaining layers to preserve the model's previously learned ability to extract general features. We replaced the removed top layers with new trainable layers. Firstly, a flattening layer converts the multi-dimensional images into the 1-dimensional arrays expected by fully-connected layers. Following the original VGG16 architecture, we added a fully connected layer with 4,096 nodes utilizing ReLu activation. Finally, a dense layer employing softmax activation allows for multi-class classification.
	For optimization, we chose Adam due to its well-performing default parameters across diverse problem sets and its adaptive learning rate adjustments during training. We chose categorical cross entropy as our loss function, chosen for its popularity in state-of-the-art models. We also gave our model an early stopping callback to avoid overfitting, allowing training to finish prematurely if accuracy on the validation set remains unchanged by more than 0.1 for three consecutive epochs. Although the models had the potential for more, all were trained for only four epochs. 
Our overall technology stack was split neatly between Spark for data processing and Keras for modeling. We leveraged Spark for exploratory data analysis, including generating healthy/diseased sample distributions and image class distributions. Since our experimental data augmentations tripled the size of our dataset to over 260 thousand images, we distributed our image manipulations via Spark for the sake of speed. However, we opted against using Spark for deep learning due to its lack of support for popular deep learning libraries compared to Keras. Overall, our methodology aims to deliver a robust and accurate solution for multi-class image classification, ensuring precise identification of plant diseases across diverse species.
	
 
 ## Dataset

  The dataset comprises 1.5GB of images featuring both healthy and diseased plant leaves, with a total of 88,000 images categorized into 38 distinct classes for classification of 14 different plants. Each of these categories contains approximately 2,500 images. The dataset provides train, validation and test splits of the data. However, these splits were determined to be unusable, as the test set didn’t represent all the categories.
The original dataset is accessible on Kaggle via the following hyperlink: Kaggle Plant Diseases Dataset. It originates from the PlantVillage-Dataset repository on Github found at Plant Village Dataset, which contains approximately 54,000 images. The Kaggle dataset incorporates offline data augmentations to mitigate bias and enhance the model’s robustness, however the augmentations are not well documented. Below are some valuable graphs for understanding the distributions of our data:


Figure 1: Healthy vs. Diseased Distribution Across Original Dataset























Figure 2: Distribution of Images Per Class



According to Figure 1, there are roughly twice as many diseased classes as healthy classes because for a single plant, there is only one possible classification of healthy, while many diseases could plague the plant. This was beneficial to our modeling process as it ensured that we were able to discern between several diseases occuring in the same plant, which may look very similar. Figure 2 confirms that there is roughly an equal distribution of examples per class with the lowest amount being 1,750 examples and the highest being 2,000 examples. The only missing images to consider are 5 plant species (Blueberry, Orange, Raspberry, Soybean, Squash) that each have either a healthy or diseased set, but not both. We’ve chosen to exclude these species entirely from our experiment because evaluating our model on them doesn’t offer a clear way to achieve our primary objective: identifying diseased plants using healthy and diseased training examples.
The augmentations previously applied to the dataset by its source on Kaggle are not documented. However, through careful analysis of patterns in the dataset file names, thirteen variants of augmentations can be discerned:
Three types of rotation: 90°, 180°, 270°.
Six combinations of a flip along an axis combined with a rotation, including both axes and 30°, 90°, and 200° rotations. 
Two variants of channel dropout, replacing the blue channel with either the green or the red.
A 25% increase in exposure.
A masking of the background behind the leaf (using black pixels).
We present the overall distribution of the percentage of augmented samples per class in Figure 3. Evidently, these augmentations are unevenly applied, with some classes consisting of 100% original unaugmented images and others ranging as low as 8% original unaugmented images. Given that there is an even overall distribution of classes among samples in our dataset, we assume that the primary goal of the source's augmentation was to create an even class distribution. 


Figure 3: Distribution of raw (unaugmented) images per class in the original dataset

After a manual review of each augmentation type, we concluded that they effectively address diverse leaf orientation and lighting conditions but not the common environmental occlusions that one might expect in the field, such as by water or dirt. As a result, we generated two additional copies of the dataset, one with rain and one with mud spatter augmentation. We performed this data preprocessing using PySpark to load the images and distribute them across the cluster to be manipulated by the Albumentations library. We present examples of our augmentations in Figure 4. After transformation, the images and a manifest dataframe were collected and written to disk on the driver node. The purpose of this manifest dataframe is to serve as an in-memory forward index mapping file paths to metadata, such as the class, source image, and augmentations performed on the image. This manifest afforded us speed and flexibility in specifying varied data partitioning schemes for our modeling experiments described in the following section.

Figure 4: Example of mud and rain spatter augmentation

##  Discussion and analysis


### Group Partitioning Experiment
"Group leakage" is a form of data leak where samples belonging to some correlated group are present in both the training and evaluation datasets, but no such group will be present at inference time (Ayotte, 2021). In our case, visually correlated groups augmented from the same source image would appear in both the training and evaluation sets if we were to follow the original dataset's random partitioning scheme. Consequently, the model may learn spurious correlations between augmentations and source images instead of features representing disease. In order to evaluate whether this is a relevant problem for our dataset, we generated two partitioning schemes: one stratified by class and another with image groups collocated per partition. If group leakage had an adverse effect on our modeling, we would expect that the first scheme should be overly optimistic, resulting in a higher test accuracy.
For the stratified scheme, we divided our data into train/validation/test sets using an 80/10/10 split and trained two models: one on the original data and another on the original data combined with our new augmentations. For the second scheme, we organized the images based on what we determined to be their original image before performing the splitting process. This involved creating distinct lists of source image names, which were then divided into train/validation/test sets. All images sharing the same source were appropriately filtered into their respective data splits. 
When comparing the models trained with grouped or not grouped splits, there weren’t large differences in the models’ metrics (precision, recall, F1-score and accuracy), nearly all being only 0.01 apart, as can be seen by comparing Figures 5 and 6. From this we concluded that the augmentations, both from the original dataset and our augmentations, exhibit significant differences, preventing the model from memorizing specific features.
However, the model trained using the grouped original data exhibits better accuracy and recall in testing with the original grouped test set, outperforming its ungrouped counterpart by nearly 5%. This difference, bolded in Figure 5, is contrary to our expectations because grouping the images by source was anticipated to potentially increase the difficulty of the problem, not improve performance. One reason that might explain this surprise improvement is the distribution of classes that resulted from the group partitioning. Since we split on source image groups without stratification, the expected distribution of classes would be similar to the distribution visualized in Figure 3. It’s plausible that this new class distribution simplified the problem for the model enough to impact results at a macro averaged level. Note that any such influence is not tangible when considering the addition of our augmentations, possibly getting overshadowed by the tripling in dataset size. Consequently, we opted to stick with stratified partitioning for our augmentations experiment.

Figure 5: Classification metrics for VGG16 on unaugmented vs augmented datasets with same-source images grouped. Differences (grouped minus not grouped) from Figure 6 shown in parenthesis.

### Occlusion Augmentation Experiment

  To evaluate the impact of our environmental occlusion augmentations on model robustness, we compared two models: one trained solely on the original dataset including pre-existing augmentations and another trained on the entirety of our data—original images and augmented ones featuring mud spatter or rain spatter. For evaluation, we created four distinct testing sets: one comprising only normal images, another combining normal and augmented images, a third containing solely mud spatter images, and a fourth containing only rain spatter images. The separation of mud spatter and rain spatter images aimed to gauge if the original model struggled with both or just one of these augmentations. Results are provided in Figure 6.

Figure 6: Classification metrics for VGG16 on unaugmented vs augmented datasets


  The model solely trained on original images performed well on the original image test set, scoring an accuracy of 91.12%. However, its performance dropped to 65.05% on the normal and augmented images. Furthermore, it achieved 49.68% accuracy on the rain spatter images and 54.52% on the mud spatter images. This demonstrated that our initial model lacked the adaptability to generalize effectively in agricultural settings where the model would likely encounter dirt or water on the leaves, indicating the need for improved robustness. Additionally, it struggled similarly on both the mud spatter and rain spatter images, indicating consistent difficulty with these alterations.
	Conversely, the model trained on original and augmented images(mud and rain spatter) performed better across the board. It performed well on the original test set getting around 93% accuracy, better than the model trained only on the original images by almost two percentage points. Additionally, it saw around 92% accuracy on the other test sets, supporting our hypothesis that the addition of augmented images would lead to a more robust model, capable of identifying plant diseases in the field. 

Figure 7: Original Model Confusion Matrix with Augmented + Original Test Set

Figure 8:  Augmented Model Confusion Matrix with Augmented + Original Test Set
	
To further understand our model’s performance, we calculated the per class accuracy, precision, recall, and F1-scores. While assessing the model trained on the original dataset with augmented images, we noticed consistent mispredictions across various classes, specifically strawberry leaf scorch and tomato early blight (Figure 7). This discrepancy likely stems from the specks present in these specific plant diseases, resembling the mud spatter augmentations introduced during the image augmentations. However, the model trained on augmented data is far more resilient to these misclassifications and overall does a better job of correctly classifying the plant diseases. Despite this, the model struggles with tomato early blight, often misidentifying it as tomato late blight (Figure 8). Additionally, it encounters challenges with two other tomato diseases, leaf mold and septoria leaf spot, where the model trained on the original data performed notably better (Figure 9). This further underscores the intricate nature of our dataset and the complexities involved in accurately classifying various plant diseases.

Figure 9: Commonly Misclassified Plant Diseases 


Tomato target spot with mud spatter augmentation
Strawberry leaf scorch
Tomato early blight

### Conclusion
  Our project has overall faced several challenges, such as the lack of deep learning models available in PySpark, group leakage within our initial dataset, and challenges with Spark’s image capabilities. We addressed the first issue by splitting our workflow into two domains: data preparation and modeling. Spark was applied to the first and Keras to the second. We addressed the group leakage problem by evaluating two dataset partitioning schemes — one stratified and one grouped—to empirically evaluate the impact of a leak, finding it negligible. Finally, we worked around Spark’s limitations regarding processing and saving images by keeping only the augmentation distributed while collecting all results on a driver node for saving to disk. We found creative ways to overcome these obstacles and learned a great deal about how the fields of machine learning and Big Data overlap in powerful and promising ways. 
	Overall, our experiment indicates that we have created a novel and useful approach to the classification and identification of plant diseases in crops. We have demonstrated that a model trained only on "idealized" leaf images does not generalize well to images augmented with occlusions featuring virtual rain and mud. Concurrently, our results indicate that the architecture of VGG16 can handle the complexity introduced by said augmentations without degrading performance on the pre-existing dataset. We believe, but cannot assert, that our occlusions mimic some of the diversity of agricultural settings exposed to the environment. As a consequence, we submit that in order to further our stated goal of developing a robust solution for proactive disease control in diverse agricultural environments, it is warranted to invest in collecting additional ground truth data that is less "ideally" staged in order to establish the correlation between realistic field data and augmentations of idealized images. Future works in this line of research may consider the applications of these models onto satellite or aerial imagery to allow the application of our model onto larger plots of lands. Additionally, we should consider applying this model to real examples of plant diseases from farms across the United States with a variety of conditions. Other augmentations, like different lighting, may be required to further improve our modeling process. This problem remains a significant issue faced by agriculturalists, but we have brought the scientific community one step closer to a solution by our research. 

