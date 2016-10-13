Team name: minions

Team members: Chirag Jain, Sharmin Pathan

Project 3 - Image Classification

Approach: Develop a CNN using Deep Learning to classify CIFAR-10 imageset

Technologies Used:
- Python 2.7
- Caffe
- Apache Spark 2.0
- CUDA
- CuDNN


Preprocessing:
- The images were accessed from LMDB. LMDB is a database when using large datasets. It provides easy access.
- LMDB requires text files that contain image paths and labels for the training and validation sets.
- Xtrain.txt and Xval.txt contain the image lists extracted from S3 for the training and validation images. 70% of the X_train dataset was used for training and the rest 30% for validation.
- LMDBs were created using the following commands:

  path-to-caffe/build/tools/convert-imageset --shuffle path-to-images Xtrain.txt trainLmdb
  
  path-to-caffe/build/tools/convert-imageset --shuffle path-to-images Xval.txt valLmdb
  
- The above commands create trainLmdb and valLmdb for training and validation respectively.
- The next step is to compute the image mean for the training set which is used for both training and prediction. The image mean is computed using the following command:

  path-to-caffe/build/tools/compute_image_mean trainLmdb mean.binaryproto
  
  
Model:
- The model contains three files namely, train.prototxt, deploy.prototxt, and solver.prototxt
- Details about the layers in our model are as follows
  conv1 - kernel size: 5 - stride: 1 (convolution layer)
  pool1 - kernel size: 3 - stride: 2 (pooling)
  relu1 (activation layer on pool1)
  conv2 - kernel size: 5 - stride: 1 (convolution layer)
  relu2 (activation layer on pool2)
  pool2 - kernel size: 3 - stride: 2 (pooling)
  conv3 - kernel size: 5 - stride: 1 (convolution layer)
  relu2 (activation layer on conv3)
  pool3 - kernel size: 3 - stride: 2 (pooling)
  
- train.prototxt: It defines layers that correspond to the transformations to be carried on the training and validation images. The first layer two layers are the data layers that contain paths to the LMDBs and mean.binaryproto. Further layers are transformations on the images.
- deploy.prototxt: Its a replica of the train.prototxt but without the data layers since the data layers are no longer valid as we won't be providing labelled data.
- solver.prototxt: It defines how the network is to be trained. Details about solver parameters https://github.com/BVLC/caffe/wiki/Solver-Prototxt
- Snapshots are taken after every 59 iterations. 


Training:
- With all the files in place, the following command starts the training
  path-to-caffe/build/tools/caffe train -solver solver.prototxt
- After the successful training in the first pass, weights of the snapshot which achieved highest accuracy were used to perform the training once again by increasing the no. of epochs and dropping the learning rate.
  path-to-caffe/build/tools/caffe train -solver solver.prototxt -weights snapshot_iter_no.caffemodel
  
  
Prediction:
- The first step is to select the model that we will use for the training ie. the model which gave us the highest accuracy.
- predict.py handles the code to make the predictions. It takes the deploy.prototxt, model, mean.binaryproto, images from the test dataset. It defines the net solver and calculates predictions on the images and stores the predictions in the finalOutput.txt


