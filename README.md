# MLOps - Integrating ML with DevOps

In this project, the aim is to automate the training of an ML model using DevOps. Whenever any code will be pushed to the mentioned github repo, it will be auto downloaded and analysed. Then, a highly customised docker container will be auto launched. This container would contain all the necessary python libraries required for that code.
Then, the code will be transferred into that container & the model will begin to train automatically. After the model has been trained, the accuracy will be retrieved if the accuracy is greater than the mentioned threshold (90 percent in this case), a success mail will be sent to the mentioned email.
in case the accuracy is below the specified threshold, the code will automatically make changes in the model & retrain it until accuracy thouches the benchmark. These changes will be made in the form of increasing layers, epochs, using augmentation in case of images, etc.

The detailed step by step description of the project is as follows :

**Step 1 :** A Jenkins task will work to auto download the code whenever it is pushed.

![](/images/cc1.png)

![](/images/cc2.png)

![](/images/cc3.png)


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
           
         
         
         
         
**Step : 3** A docker container will automatically run with a persistent volume attached to ensure no data loss. We had downloaded all the code in this same persistent volume in step 1. So, all the code is now present in the container. 

![](/images/dr4.png)


**Note : I have downloaded all the prominent libraries in my Dockerfile but if you want a customized image, you can just create a Dockerfile by installing python & upgrading pip. After that, you can analyze the code to auto download required libraries in the container, when it has been launched. This can be done by adding the following commands in the above Jenkins Task :**
                   
             if sudo cat ml/model.py | grep keras
             then
             sudo docker exec s1 bash -c "pip3 install keras"
             fi
             
             if sudo cat ml/model.py | grep sklearn
             then
             sudo docker exec s1 bash -c "pip3 install sklearn"
             fi
             
             if sudo cat ml/model.py | grep tensorflow
             then
             sudo docker exec s1 bash -c "pip3 install tensorflow"
             fi
             
             if sudo cat ml/model.py | grep opencv
             then
             sudo docker exec s1 bash -c "pip3 install opencv-python"
             fi
             
This way, you can mention all possible libraries as per your requirement. This will download only those libraries in the container, which are required in that specific program. Thus, a highly customized container will be created. 
I have not used this technique in my project because it will download the required libraries everytime a new ML model comes. I have mentioned commonly used libraries directly in my Dockerfile because in that way, I have to download the libraries just once. However, if you have a good internet connection & want a highly customized container, you can go for this technique.





**Step : 4** I have a small code present in the same persistent volume in which the ML model will be downloaded. This piece of code needs to be appended to the ML model. The task of this code is to check the accuracy & accordingly trigger other jenkins Jobs. If the accuracy is greater than 90 percent, a _**success mail**_ Jenkins job will be auto-trigerred to send a mail to the specified email.
If the accuracy is less than 90 percent, a _**model tweak**_ job will be auto triggered that will modify the model in several ways to increase it's accuracy & then the model will be retrained.

My ML model is as follows :

(I have tested it on various ML models. It works successfully. This is just a sample model if you wish to test the procedure. I have provided the _**wines.csv**_ dataset used in this model in the same repository.)
 


                import pandas as pd
                df = pd.read_csv('wines.csv')
                y = df['Class']
                y_cat = pd.get_dummies(y)
                X = df.drop('Class' , axis=1)
                from keras.models import Sequential
                model  =  Sequential()
                from keras.layers import Dense
                model.add(Dense(units=5 , input_shape=(13,), 
                                          activation='relu', 
                                          kernel_initializer='he_normal' ))
                model.add(Dense(units=8 , 
                                activation='relu', 
                                kernel_initializer='he_normal' ))
                model.add(Dense(units=2, 
                                activation='relu', 
                                kernel_initializer='he_normal' ))
                model.add(Dense(units=3, activation='softmax'))
                from keras.optimizers import RMSprop
                model.compile(optimizer=RMSprop(learning_rate=0.01),  
                              loss='categorical_crossentropy',
                              metrics=['accuracy']
                              )
                accuracy = model.fit(X,y_cat, epochs=100)

                model.save('modelsave.h5')

                acc=accuracy.history['accuracy'][-1:][0]


Note that I have saved my accuracy in _**acc**_ variable in this model. That's why I have used the same _**acc**_ variable in my code that'll be appended to this.

The piece of code to be appended is as follows :

               acc = acc * 100
               import os
               if int(acc) > 90 :
                 os.system('curl --user "admin:redhat" http://192.168.42.113:8080/job/success%20mail/build?token=sparsh')
               else :
                  os.system('curl --user "admin:redhat" http://192.168.42.113:8080/job/model%20tweak/build?token=sparsh')
                  
                  
It'll be automatically appended by the following Jenkins task :

![](/images/ap5.png)







**Step : 5** A Jenkins task will automatically start training the model as soon as code is appended. I have organised these tasks in a systematic way and linked them one after the another. When a task successfully completes, it triggers the next task.

![](/images/mt6.png)



**Step : 6** This is the _**success mail**_  task of Jenkins that will be trigerred only when the accuracy is greater than 90 percent.

![](/images/mail7.png)

If you mail fails to send, you need to go to Jenkins configuration & do the following setup :

![](/images/mail8.png)


After that you need to go to your Redhat VM & run --

![](/images/mail9.png)      


In that file, in the **JENKINS_JAVA_OPTIONS=" "**, make the following changes:

![](/images/mail10.png)


After that, restart your Jenkins using *systemctl restart jenkins*. 


If you still face error in sending email, & the popup says **authentication error**, you need to go to your Google Account & disable the two step authentication & turn on *less secure app access* option below it. 
After this, you can test the email service by sending a test mail. It'll work fine.




**Step : 7** This is the _**model tweak**_ task that'll be trigerred if the accuracy is less than 90 percent. This will make necessary changes in the model & retrain it.

   Ye task ho nhi paa rha. Koi help krr do pls. Model ko modify kaise kru......


**Step : 8** This task will  keep on monitoring the container in which model is being trained. If the container crashes due to any reason, this task will trigger **Step 3**, in which we had launched the docker container. Since the model is being saved after every train & there is a persistent volume attached, so no data loss will occur & new container would carry on from where last container had left.
This Jenkins task will keep on running each minute & I have deliberately failed it if an existing docker container with the name s1 is found. This task will only succeed if it finds no s1 running. Then, it will trigger Step 2 & all the next steps will be automatically followed.





