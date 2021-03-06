[Home](https://mgcodesandstats.github.io/) |
[Portfolio](https://mgcodesandstats.github.io/portfolio/) |
[Terms and Conditions](https://mgcodesandstats.github.io/terms/) |
[E-mail me](mailto:contact@michaeljgrogan.com) |
[LinkedIn](https://www.linkedin.com/in/michaeljgrogan/)

# Deploying Python application using Docker and AWS

The use of Docker in conjunction with AWS can be highly effective when it comes to building a data pipeline.

Let me ask you if you have ever had this situation before. You are building a model in Python which you need to send over to a third-party, e.g. a client, colleague, etc. However, the person on the other end cannot run the code! Maybe they don't have the right libraries installed, or their system is not configured correctly.

Whatever the reason, Docker alleviates this situation by storing the necessary components in an image, which can then be used by a third-party to deploy an application effectively.

In this example, we will see how a simple Python script can be incorporated into a Docker image, and this image will then be pushed to ECR (Elastic Container Registry) in AWS.

## Python Script

Consider a simple Python script for calculating a cumulative binomial probability across 100 trials:

```
>>> import numpy as np
>>> 
>>> def prob(l):
...     p=np.arange(0,100,1)
...     h=1-l
...     q=1-(h**p)
...     print(q)
... 
>>> prob(0.02)
[0.         0.02       0.0396     0.058808   0.07763184 0.0960792
 0.11415762 0.13187447 0.14923698 0.16625224 0.18292719 0.19926865
 0.21528328 0.23097761 0.24635806 0.2614309  0.27620228 0.29067823
 0.30486467 0.31876738 0.33239203 0.34574419 0.3588293  0.37165272
 0.38421966 0.39653527 0.40860456 0.42043247 0.43202382 0.44338335
 0.45451568 0.46542537 0.47611686 0.48659452 0.49686263 0.50692538
 0.51678687 0.52645113 0.53592211 0.54520367 0.5542996  0.5632136
 0.57194933 0.58051035 0.58890014 0.59712214 0.60517969 0.6130761
 0.62081458 0.62839829 0.63583032 0.64311371 0.65025144 0.65724641
 0.66410148 0.67081945 0.67740306 0.683855   0.6901779  0.69637434
 0.70244686 0.70839792 0.71422996 0.71994536 0.72554646 0.73103553
 0.73641482 0.74168652 0.74685279 0.75191573 0.75687742 0.76173987
 0.76650507 0.77117497 0.77575147 0.78023644 0.78463171 0.78893908
 0.7931603  0.79729709 0.80135115 0.80532413 0.80921764 0.81303329
 0.81677263 0.82043717 0.82402843 0.82754786 0.8309969  0.83437697
 0.83768943 0.84093564 0.84411693 0.84723459 0.85028989 0.8532841
 0.85621842 0.85909405 0.86191217 0.86467392]
>>> prob(0.04)
[0.         0.04       0.0784     0.115264   0.15065344 0.1846273
 0.21724221 0.24855252 0.27861042 0.307466   0.33516736 0.36176067
 0.38729024 0.41179863 0.43532669 0.45791362 0.47959708 0.50041319
 0.52039666 0.5395808  0.55799757 0.57567766 0.59265056 0.60894453
 0.62458675 0.63960328 0.65401915 0.66785839 0.68114405 0.69389829
 0.70614236 0.71789666 0.7291808  0.74001356 0.75041302 0.7603965
 0.76998064 0.77918142 0.78801416 0.79649359 0.80463385 0.81244849
 0.81995055 0.82715253 0.83406643 0.84070377 0.84707562 0.8531926
 0.85906489 0.8647023  0.87011421 0.87530964 0.88029725 0.88508536
 0.88968195 0.89409467 0.89833088 0.90239765 0.90630174 0.91004967
 0.91364769 0.91710178 0.92041771 0.923601   0.92665696 0.92959068
 0.93240705 0.93511077 0.93770634 0.94019809 0.94259016 0.94488656
 0.94709109 0.94920745 0.95123915 0.95318959 0.955062   0.95685952
 0.95858514 0.96024174 0.96183207 0.96335878 0.96482443 0.96623146
 0.9675822  0.96887891 0.97012375 0.9713188  0.97246605 0.97356741
 0.97462471 0.97563972 0.97661413 0.97754957 0.97844759 0.97930968
 0.9801373  0.9809318  0.98169453 0.98242675]
>>> prob(0.06)
[0.         0.06       0.1164     0.169416   0.21925104 0.26609598
 0.31013022 0.35152241 0.39043106 0.4270052  0.46138489 0.49370179
 0.52407969 0.5526349  0.57947681 0.6047082  0.62842571 0.65072017
 0.67167696 0.69137634 0.70989376 0.72730013 0.74366213 0.7590424
 0.77349985 0.78708986 0.79986447 0.8118726  0.82316025 0.83377063
 0.84374439 0.85311973 0.86193255 0.87021659 0.8780036  0.88532338
 0.89220398 0.89867174 0.90475144 0.91046635 0.91583837 0.92088807
 0.92563478 0.9300967  0.93429089 0.93823344 0.94193943 0.94542307
 0.94869768 0.95177582 0.95466927 0.95738912 0.95994577 0.96234902
 0.96460808 0.9667316  0.9687277  0.97060404 0.9723678  0.97402573
 0.97558419 0.97704913 0.97842619 0.97972062 0.98093738 0.98208114
 0.98315627 0.98416689 0.98511688 0.98600987 0.98684927 0.98763832
 0.98838002 0.98907722 0.98973258 0.99034863 0.99092771 0.99147205
 0.99198373 0.9924647  0.99291682 0.99334181 0.9937413  0.99411682
 0.99446981 0.99480163 0.99511353 0.99540672 0.99568231 0.99594137
 0.99618489 0.9964138  0.99662897 0.99683123 0.99702136 0.99720008
 0.99736807 0.99752599 0.99767443 0.99781396]
 ```

The objective is to be able to incorporate this script into a Docker image (a Linux terminal is used for this example).

## Docker Components

A folder is created with the following components:

- Python code (PythonExample.py)
- Dockerfile

Please see the **cumulprobfunc** folder on the [Github repository](https://github.com/MGCodesandStats/docker-aws) for the full code.

The script **PythonExample.py** contains the full Python code that is being executed, while the **Dockerfile** contains the set of instructions used to build the Docker image.

### Dockerfile

```
FROM python
RUN pip install numpy
COPY . /src
CMD ["python", "/src/PythonExample.py"]
```

A terminal is opened, Docker is installed, and the image is generated:

```
sudo snap install docker --devmode
sudo docker build -t cumulprobfunc .
sudo docker images
sudo docker run cumulprobfunc
```

When checking ```sudo docker images```, we see that the image is confirmed to exist:

```
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
cumulprobfunc                                            latest              0123a45b678c        10 seconds ago      1.08GB
```

Moreover, running the application now generates the results:

```
>>> sudo docker run cumulprobfunc
[0.         0.02       0.0396     0.058808   0.07763184 0.0960792
 0.11415762 0.13187447 0.14923698 0.16625224 0.18292719 0.19926865
 0.21528328 0.23097761 0.24635806 0.2614309  0.27620228 0.29067823
 0.30486467 0.31876738 0.33239203 0.34574419 0.3588293  0.37165272
 0.38421966 0.39653527 0.40860456 0.42043247 0.43202382 0.44338335
 0.45451568 0.46542537 0.47611686 0.48659452 0.49686263 0.50692538
 0.51678687 0.52645113 0.53592211 0.54520367 0.5542996  0.5632136
 0.57194933 0.58051035 0.58890014 0.59712214 0.60517969 0.6130761
 0.62081458 0.62839829 0.63583032 0.64311371 0.65025144 0.65724641
 0.66410148 0.67081945 0.67740306 0.683855   0.6901779  0.69637434
 0.70244686 0.70839792 0.71422996 0.71994536 0.72554646 0.73103553
 0.73641482 0.74168652 0.74685279 0.75191573 0.75687742 0.76173987
 0.76650507 0.77117497 0.77575147 0.78023644 0.78463171 0.78893908
 0.7931603  0.79729709 0.80135115 0.80532413 0.80921764 0.81303329
 0.81677263 0.82043717 0.82402843 0.82754786 0.8309969  0.83437697
 0.83768943 0.84093564 0.84411693 0.84723459 0.85028989 0.8532841
 0.85621842 0.85909405 0.86191217 0.86467392]
[0.         0.04       0.0784     0.115264   0.15065344 0.1846273
 0.21724221 0.24855252 0.27861042 0.307466   0.33516736 0.36176067
 0.38729024 0.41179863 0.43532669 0.45791362 0.47959708 0.50041319
 0.52039666 0.5395808  0.55799757 0.57567766 0.59265056 0.60894453
 0.62458675 0.63960328 0.65401915 0.66785839 0.68114405 0.69389829
 0.70614236 0.71789666 0.7291808  0.74001356 0.75041302 0.7603965
 0.76998064 0.77918142 0.78801416 0.79649359 0.80463385 0.81244849
 0.81995055 0.82715253 0.83406643 0.84070377 0.84707562 0.8531926
 0.85906489 0.8647023  0.87011421 0.87530964 0.88029725 0.88508536
 0.88968195 0.89409467 0.89833088 0.90239765 0.90630174 0.91004967
 0.91364769 0.91710178 0.92041771 0.923601   0.92665696 0.92959068
 0.93240705 0.93511077 0.93770634 0.94019809 0.94259016 0.94488656
 0.94709109 0.94920745 0.95123915 0.95318959 0.955062   0.95685952
 0.95858514 0.96024174 0.96183207 0.96335878 0.96482443 0.96623146
 0.9675822  0.96887891 0.97012375 0.9713188  0.97246605 0.97356741
 0.97462471 0.97563972 0.97661413 0.97754957 0.97844759 0.97930968
 0.9801373  0.9809318  0.98169453 0.98242675]
[0.         0.06       0.1164     0.169416   0.21925104 0.26609598
 0.31013022 0.35152241 0.39043106 0.4270052  0.46138489 0.49370179
 0.52407969 0.5526349  0.57947681 0.6047082  0.62842571 0.65072017
 0.67167696 0.69137634 0.70989376 0.72730013 0.74366213 0.7590424
 0.77349985 0.78708986 0.79986447 0.8118726  0.82316025 0.83377063
 0.84374439 0.85311973 0.86193255 0.87021659 0.8780036  0.88532338
 0.89220398 0.89867174 0.90475144 0.91046635 0.91583837 0.92088807
 0.92563478 0.9300967  0.93429089 0.93823344 0.94193943 0.94542307
 0.94869768 0.95177582 0.95466927 0.95738912 0.95994577 0.96234902
 0.96460808 0.9667316  0.9687277  0.97060404 0.9723678  0.97402573
 0.97558419 0.97704913 0.97842619 0.97972062 0.98093738 0.98208114
 0.98315627 0.98416689 0.98511688 0.98600987 0.98684927 0.98763832
 0.98838002 0.98907722 0.98973258 0.99034863 0.99092771 0.99147205
 0.99198373 0.9924647  0.99291682 0.99334181 0.9937413  0.99411682
 0.99446981 0.99480163 0.99511353 0.99540672 0.99568231 0.99594137
 0.99618489 0.9964138  0.99662897 0.99683123 0.99702136 0.99720008
 0.99736807 0.99752599 0.99767443 0.99781396]
 ```

## Pushing to ECR - AWS

So, the application has been created in Docker. However, there are many instance where a Docker image will need to be pushed to a cloud environment. In this instance, here is how the image just created can be pushed to ECR.

Firstly, a repository is created in ECR - I choose to assign the name **cumulprob** to the repository in this instance:

![create-ecr-repository](create-ecr-repository.png)

To log into the AWS account through the CLI (command line interface), make sure you have configured your [access and secret access keys]. The following [guide from AWS](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) provides more information on this.

Once that is configured, the Docker image can now be pushed to the ECR.

In the terminal, log into the ECR instance:

```aws ecr get-login --no-include-email --region us-east-1```

In this example, the US-East region is used, but you should use the region that your repository is hosted in. Moreover, you should also make sure that this is the same region as associated with the one specified when configuring your login details - a mismatch can result in the connection failing.

A login link is generated, and this is then prefixed with **sudo** to login. You should see the prompt **"Login successful"**.

The relevant Docker image is tagged, and the repository directory is set:

```sudo docker tag cumulprobfunc:latest youraddress.dkr.ecr.us-east-1.amazonaws.com/cumulprob:latest```

Now, the Docker image is pushed to the repository:

```sudo docker push youraddress.dkr.ecr.us-east-1.amazonaws.com/cumulprob:latest```

Once all the instances display **Pushed**, then the Docker image should now appear in the repository.

Note that a Docker container or image can be removed by inputting the following into a Linux terminal:

**To remove a container:**

```sudo docker rm containerID```

**To remove an image:**

```sudo docker rmi imageID```

**containerID** and **imageID** are replaced with the sequence of letters and numbers that represents the container or image.

## Conclusion

In this example, we have seen how to generate a Docker image for a Python application, and then push the Docker image to an AWS ECR repository. Hope you found this useful!
