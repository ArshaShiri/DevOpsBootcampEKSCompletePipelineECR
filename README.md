# Complete CI/CD Pipeline Using ECR and EKS


## Create ECR Repository

We create the repository with the default settings. In general, a repository can be created for each application.

![image](https://user-images.githubusercontent.com/18715119/235351159-cfc844f8-7026-4c50-a2e8-925d0f6e563d.png)

## Create Credentials in Jenkins

We can use the `view push commands` section of the ECR to help us:

![image](https://user-images.githubusercontent.com/18715119/235351250-cdd5a63e-8fc6-4579-8ac3-a9236187af4b.png)
![image](https://user-images.githubusercontent.com/18715119/235351265-266eab0e-c8e2-456b-90f9-da2167132be9.png)

    # On our local machine we can run the following command to get the password of the ECR repo:
    aws ecr get-login-password --region eu-central-1
    
We then create a credential on Jenkins. The username for this credential is AWS as can be seen in the command from the `view push commands` section. The password then can be copied from the output of the command above.

![image](https://user-images.githubusercontent.com/18715119/235351490-5bf4e26e-d6fc-4de5-b830-6f846f0baece.png)

## Create Secret for AWS ECR

    kubectl create secret docker-registry aws-registry-key \
    --docker-server=849690659475.dkr.ecr.eu-central-1.amazonaws.com \
    --docker-username=AWS \
    --docker-password={password output from the command above}


We can then run the pipeline.
