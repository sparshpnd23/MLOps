# MLOps - Integrating ML with DevOps

In this project, the aim is to automate the training of an ML model using DevOps. Whenever any code will be pushed to the mentioned github repo, it will be auto downloaded and analysed. Then, a highly customised docker container will be auto launched. This container would contain all the necessary python libraries required for that code.
Then, the code will be transferred into that container & the model will begin to train automatically. After the model has been trained, the accuracy will be retrieved if the accuracy is greater than the mentioned threshold (90 percent in this case), a success mail will be sent to the mentioned email.
in case the accuracy is below the specified threshold, the code will automatically make changes in the model & retrain it until accuracy thouches the benchmark. These changes will be made in the form of increasing layers, epochs, using augmentation in case of images, etc.

The detailed step by step description of the project is as follows :

**Step 1 :** A Jenkins task will work to auto download the code whenever it is pushed.


**Step 2:** I have a Dockerfile pre created in my Redhat Linux. This Dockerfile contains all the libraries necessary for training a model. However, you can add or remove according to your requirement.
           
           FROM centos:latest
           RUN yum install python3 -y
           RUN pip3 install --upgrade pip
           RUN pip3 install keras
           RUN pip3 install tensorflow
           RUN pip3 install numpy
           RUN pip3 install pandas
           RUN pip3 install sklearn
           RUN pip3 install scipy
           RUN pip3 install pillow
           RUN pip3 install opencv-python
           RUN pip3 install torch
           
           
           
Build an image from this Dokcerfile using :
           
           docker build -t NAME:TAG /location_of_Dockerfile
           

           
           
           
           
           

