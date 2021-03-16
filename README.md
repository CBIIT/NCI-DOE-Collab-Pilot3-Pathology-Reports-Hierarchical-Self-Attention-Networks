## Multi Task-Convolutional Neural Networks (MT-CNN)

##### Author: Biomedical Sciences, Engineering, and Computing (BSEC) Group; Computer Sciences and Engineering Division; Oak Ridge National Laboratory

### Description
MT-CNN is a CNN for natural language processing (NLP) and information Extraction from free-form texts. BSEC group designed the model for information extraction from cancer pathology reports.

### User Community
Data scientists interested in classifying free form texts (such as pathology reports, clinical trials, abstracts, and so on). 

### Usability
The provided untrained model can be used by data scientists to be trained on their own data, or use the trained model to classify the provided test samples. The provided scripts use a pathology report that has been downloaded from the Genomics Data Commons, converted to text format, cleaned, and preprocessed. Here is an example [report](https://portal.gdc.cancer.gov/legacy-archive/files/a9a42650-4613-448d-895e-4f904285f508).

### Uniqueness
Classification of unstructured text is a classical problem in natural language processing. The community has developed state-of-the-art models like BERT, Bio-BERT, and Transformer. This model has the advantage of working on a relatively long report (that is, over 400 words) and shows scalability in terms of accuracy and speed with relatively small number of unstructured pathology reports. 
&#x1F534;_**(Question: Are you saying it should be scalable to a larger number of reports?)**_

### Components
* Original and processed training, validation, and test data.
* Untrained neural network model.
* Trained model weights and topology to be used in inference.

&#x1F534;_**Suggestion - let's move this technical information (below) to a Technical ReadME like our other repos.**_

### Completed Model Trans_Validate Template
&#x1F534;_**(Question: When I formatted this info as a table, I added generic column headings. Are these column headings ok?)**_
| Attribute  | Value |
| ------------- | ------------- |
| Model Developer / Point of Contact  | Hong-Jun Yoon |
| Model Name | MT-CNN |
| Inputs  | Indices of tokenized text  |
| Outputs  | softmax  |
| Training Data  | sample data available in the repo  |
| Uncertainty Quantification  | N/A  |
| Platform  | Keras/Tensorflow   |


### Software Setup
To set up the Python environment needed to train and run this model:
1. Install [conda](https://docs.conda.io/en/latest/) package manager.
2. Clone this repository. &#x1F534;**_(Question: Is this step referring to the repository that contains this readme file? If so, we could specifically name it here, in case someone takes this readme out of context.)_**
3. Create the environment as shown below.
```bash
   conda env create -f environment.yml -n mt-cnn
   conda activate mt-cnn
   ```
### Data Setup:

The data setup goes through multiple steps. Run all the commands below from the top level directory of the repository.

1- Download reports from modac.cancer.gov:

To download the  reports needed to train and test the model, and the trained model file, you should first create an account on the Model and Data Clearinghouse [MoDac](modac.cancer.gov).

To download the reports and their corresponding metadata, run the script  [./data_utils/download_data.py](./data_utils/download_data.py).

2- Generate training/validaton/test datasets:
The script  [./data_utils/trainTestSplitMetaData.py](./data_utils/trainTestSplitMetaData.py) splits the data into training/validation/test datasets and mapps the site and histology to integer categories. 


3- Process reports and generate features:
The script [./data_utils/data_handler.py](./data_utils/data_handler.py) does the following 
* Cleans up punctuation and unecessary tokens from the reports
* Generates a dictionary that maps words in the corpus with a least five appearances to a unique index. 
* Uses the word to index dictionary to encode every report into a numpy vector of size 1500, where 1500 is the maximum number of words in a pathology report. Every element in that array represent the index of the word in the corpus.
* Generates the corresponding numpy arrays for the training/validation/test datasets.


For more information about the original, cleaned, and generated data, refer to this [README](./data/README.md) file. Note all artifacts will be generated after you run the data setup commands above.

### Training

To train a MT-CNN model with the sample data, execute the script [mt_cnn_exp.py](./mt_cnn_exp.py). This script calls MT-CNN implementation in [keras_mt_shared_cnn.py](./keras_mt_shared_cnn.py). 


```
$ python mt_cnn_exp.py
Using TensorFlow backend.
....
Number of classes:  [25, 117]
Model file:  mt_cnn_model.h5
__________________________________________________________________________________________________
Layer (type)                    Output Shape         Param #     Connected to                     
==================================================================================================
Input (InputLayer)              (None, 1500)         0                                            
__________________________________________________________________________________________________
embedding (Embedding)           (None, 1500, 300)    6948900     Input[0][0]                      
__________________________________________________________________________________________________
0_thfilter (Conv1D)             (None, 1500, 100)    90100       embedding[0][0]                  
__________________________________________________________________________________________________
1_thfilter (Conv1D)             (None, 1500, 100)    120100      embedding[0][0]                  
__________________________________________________________________________________________________
2_thfilter (Conv1D)             (None, 1500, 100)    150100      embedding[0][0]                  
__________________________________________________________________________________________________
global_max_pooling1d_1 (GlobalM (None, 100)          0           0_thfilter[0][0]                 
__________________________________________________________________________________________________
global_max_pooling1d_2 (GlobalM (None, 100)          0           1_thfilter[0][0]                 
__________________________________________________________________________________________________
global_max_pooling1d_3 (GlobalM (None, 100)          0           2_thfilter[0][0]                 
__________________________________________________________________________________________________
concatenate_1 (Concatenate)     (None, 300)          0           global_max_pooling1d_1[0][0]     
                                                                 global_max_pooling1d_2[0][0]     
                                                                 global_max_pooling1d_3[0][0]     
__________________________________________________________________________________________________
dropout_1 (Dropout)             (None, 300)          0           concatenate_1[0][0]              
__________________________________________________________________________________________________
site (Dense)                    (None, 25)           7525        dropout_1[0][0]                  
__________________________________________________________________________________________________
histology (Dense)               (None, 117)          35217       dropout_1[0][0]                  
==================================================================================================
Total params: 7,351,942
Trainable params: 7,351,942
Non-trainable params: 0
__________________________________________________________________________________________________
None
Train on 4579 samples, validate on 509 samples
.....
.....
Epoch 00024: val_loss did not improve from 1.37346
Epoch 25/100
 - 19s - loss: 0.6999 - site_loss: 0.0430 - histology_loss: 0.2128 - site_acc: 0.9886 - histology_acc: 0.9393 - val_loss: 1.5683 - val_site_loss: 0.1621 - val_histology_loss: 0.9508 - val_site_acc: 0.9607 - val_histology_acc: 0.7937

Epoch 00025: val_loss did not improve from 1.37346
Prediction on test set 
task site test f-score: 0.9599,0.9389
task histology test f-score: 0.8184,0.4192
```

### Inference on test dataset:
To test the trained model in inference, first download the trained model by running this (script) [./data_utils/download_model.py]. 

Then run the script (mt_cnn_infer.py)[mt_cnn_infer.py] which performs the following:
* Performs inference on the test dataset
* Reports the micro, macro F1 scores of the model on the test dataset


```bash
   python mt_cnn_infer.py
   .....
   Prediction on test set 
   task site test f-score: 0.9662,0.9421
   task histology test f-score: 0.8168,0.4098
   ```

### Inference on a single report:
To run model in inference model for a single report, use the script  (predictions.py_[./predictions.py] which accepts as input a single txt report, run inference, and displays the true labels and the inferenced labels. There is a default report to be used by the script to prediciton.

```
   python predictions.py
   
   MTCNN Prediction
   prostate
   8140.0
   8141.
   Original Labels
         filename                                             site            histology
   3555  TCGA-2A-AAYO.3889AA76-F350-4DA4-987B-79E8D2349...    "prostate"      8140.0

```



### Disclaimer
UT-Battelle, LLC and the government make no representations and disclaim all warranties, both expressed and implied. There are no express or implied warranties:
* Of merchantability or fitness for a particular purpose, 
* Or that the use of the software will not infringe any patent, copyright, trademark, or other proprietary rights, 
* Or that the software will accomplish the intended results, 
* Or that the software or its use will not result in injury or damage. 

The user assumes responsibility for all liabilities, penalties, fines, claims, causes of action, and costs and expenses, caused by, resulting from or arising out of, in whole or in part the use, storage or disposal of the software.


### Acknowledgments
This work has been supported in part by the Joint Design of Advanced Computing Solutions for Cancer (JDACS4C) program established by the U.S. Department of Energy (DOE) and the National Cancer Institute (NCI) of the National Institutes of Health.
