# Udagram - Photo Sharing App develop as Continuous Deployment with Travis CI and Kubernetes on AWS using Terraform and Kubeone

## Table of Contents

* [OverView](#OverView)
* [Setup Environment](#Setting-Environment)
* [Installation Docker](#Installation-Docker)
* [Installation Kubernetes](#Installation-Kubernetes)
* [Project Reference Sources Or Links](#references)

## OverView

In this project, we will apply the skills we have acquired in this cloud engineering course to design, deploy and operate a cloud native photo sharing application. We will reuse our existing application and convert and extend into a microservice architecture. The microservice architecture makes it more easier to scale it to large systems. They are great enablers for continuous integration and delivery too. This includes independent scaling, independent releases and deployments  and independent development so that ACH service has its own codebase. After the application is divided into smaller services, we will containerize it and deploy it to a Kubernetes cluster. This includes deployment pipeline, scalability, observability, Services, Networking and deployment strategies to service the system. 

For this project we will uses Oracle Vitualbox Virtual Machine for simulating the development Environment. For simplicity we will setup 2 development virtual machine, one is for Docker, and the other is for Kubernetes. 

## Setting-Environment

 * Virtaul Machine for Docker
    1. To install please follow the this link: https://docs.docker.com/toolbox/toolbox_install_windows/. After Installed you will be able to use most of the docker functions.

 * Virtual Machine for Kubernetes
    1. Download the Ubuntu from the Ubuntu website: https://ubuntu.com/download, we will download the Ubuntu 18.0.4 desktop version.
    2. Download the Terraform (For generating the AWS infrasturture and kubernetes cluster) link: https://www.terraform.io/downloads.html , please choose Linux and download the latest version, follow the steps for installation.
    3. Download the KubeOne (a management tool for managing High Availability Cluster) from this Site: https://github.com/kubermatic/kubeone.  
    4. Dowload the Kubectl (for running commands against the Kubernetes Cluster). Follow this link: https://kubernetes.io/docs/tasks/tools/install-kubectl/ .
    5. Install AWS Cli. Follow this link: https://linuxhint.com/install_aws_cli_ubuntu/ .    

 * Setting the Envormnet Variables:
    For Windows, Open the `File Explorer` and right click the `The PC`, Select the properties. From pop-up window choose `Advance System Settings`, then choose `Environment Variables`.

    Please add the following Environment Variables (fill in your values, for JWT_SECRET fill in 'jwt').
    - AWS_MEDIA_BUCKET
    - AWS_PROFILE
    - AWS_PROFILE
    - JWT_SECRET
    - POSTGRESS_DB
    - POSTGRESS_HOST
    - POSTGRESS_PASSWORD
    - POSTGRESS_USERNAME
    - URL
    
## Installation-Docker

* The only options to install Udagram is by cloning from the git hub please follow the instructions below .

    1. Install the Docker Runtime Engine:
        * From this link `https://docs.docker.com/install/linux/docker-ce/ubuntu/`
    2. Install Docker Compose:
        * From this link `https://phoenixnap.com/kb/install-docker-compose-ubuntu`
    2. From the Ubuntu/Linux Terminal, type in the command:
        * git clone `https://github.com/programmingwebsite/aws-microservices-kubernetes.git`. 
    3. After cloning, browse to the exercises folder. From the exercise folder,  you need to install the packages and dependencies for     `udacity-c3-frontend`, `udacity-c3-restapi-feed` and `udacity-c3-user`. Type in the following command for each folder.
        * npm ci
    4. Build and Create the docker Images.
        * Browser to the folder `udacity-c3-deployment/docker`. Type the following command: 
        `docker-compose -f docker-compose-build.yaml build --parallel`. This will build all the images for the Udagram Project.
    5. Push all the images to the Docker Hub Repository. You need to have a Docker Hub account.
        * From the Docker Terminal. Type in `docker login`, you will be prompt in the username and password.
        * To push all the images to Docker Hub , type in `docker-compose -f docker-compose-build.yaml push`.

## Installation-Kubernetes 

* The whole setup kubernetes describe below is referenced from my classmate 'Dan B'.

    * From the Docker machine copy the whole folder `k8s`. (inside the `udacity-c3-deployment` folder) to the kubernetes virtual machine.

        1. export both your AWS_ACCESS_KEY_ID and export AWS_SECRET_ACCESS_KEY in your terminal. If you quit your bash session you’ll have to do this again. Execute echo $AWS_ACCESS_KEY_ID to make sure you’ve done this correctly.

        2. Download Terraform and kubeone per the resources above.

        3. Follow the instructions

            * POTENTIAL ERRORS: “Handshake”

                This has to do with your public/private rsa id

                once you’ve executed terraform apply take a look at this: https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

                Specifically the section relating to “Adding your SSH key to the ssh-agent”. I found after my cluster was up but before I installed kubeone, I ran the agent and executing ssh-add I was able to execute kubeone install config.yaml -t tf.json and it worked.

                Make sure you then execute: export KUBECONFIG=$PWD/${name of your cluster}-kubeconfig

            * Deploying Everything: Things to solve ahead of time:

                - aws-secret.yaml
                    * Find your credentials in ~/.aws/credentials.
                    You need to save your key in base64 —> type in 'base64 credentials'  —> this output is what you save under “credentials” in your aws-secret.yaml file.
                - env-secret.yaml
                    * Repeat the process above for your POSTGRESS_USERNAME / POSTGRESS_PASSWORD
                    Both of those variable should be in your bash_profile. If they are then just execute echo -n $POSGRESS_USERNAME or POSTGRESS_PASSWORD| base64 for the correct values.
                - env-configmap.yaml
                    * Make sure all these variables are correct. This is pretty self explanatory. You should know where to get these values if the are not   already saved in your bash profile.
            * CONSISTENCY IS KEY!!!

                - NOTE: I changed my AWS_MEDIA_BUCKET to AWS_BUCKET, but I did a find and replace. So all instances that pointed to AWS_MEDIA_BUCKET now     pointed to AWS_BUCKET. This is because Ruttner and Scheele call it two different things.

                NOTE: I changed all my POSTGRESS_DB to POSTGRESS_DATABASE. It doesn’t matter. Just be consistent. Save your self the head ache and figure this out now rather than debugging it in the most agonizing, foolish way possible —> pods break, restart, break, restart, etc…

                OK, so everything is saved properly, your clusters are up.
                Kubectl apply -f [all your files] This should be about 11 in total

                If your reserve proxy is not running: Execute: kubectl logs ${your pod name here}

                If you see anything relating to a “taint” then you don’t have enough resources. You’ll notice on the videos scheele has two worker nodes and 3 masters. When I spun this up I had 1 worker node and 3 masters.
                Execute this to enable a master as a worker node —> this is how I fixed this problem: kubectl taint node ${your node here}- role.kubernetes.io/master:NoSchedule-

                To find your node value execute kubectl get nodes -o wide. Copy the NAME for all nodes that are MASTERS into the bash command above. This should fix your problem.

                If you have any “CrashLoopBackOff” Errors then you have a variable issue. Make sure your env-configmap.yaml has the proper values. If need be: Kubectl delete secret env-config update your env-configmap.yaml Kubectl apply -f env-configmap.yaml

            * Helpful commands:

                Kubectl describe pod/svc/rs/deployment ${name of pod/service/replicaset/deployment} Kubectl log ${name of pod/service/replicatset/deployemnt}

                Its helpful to look at the logs if something is wrong. It’ll show you the code output in your containers should you need to debug.        
## References

* Code References
    * [Kubeone](https://github.com/kubermatic/kubeone)
    * [Udacity](https://www.udacity.com/)
    * [GitHub](https://github.com/)
    * [Generating ssh key and add it to the agent](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
    * [Linux Too Many Open Files Solution](https://blog.csdn.net/fdipzone/article/details/34588803)
    * [kubectl (exec error: unable to upgrade connection: Forbidden ) Solution](https://github.com/opsnull/follow-me-install-kubernetes-cluster/issues/255)
    * [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
    * [Installing Kubeadmin , Kubelet and Kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
    * [Running HA Kubernetes Cluster Using Kubeone in AWS](https://medium.com/@alexander_15213/running-ha-kubernetes-clusters-on-aws-using-kubeone-535b93af57ab)
    * [How to install Kubernetes on AWS Cluster using Kubeone](https://github.com/kubermatic/kubeone/blob/master/docs/quickstart-aws.md)
    * [Learn Kubernetes Basic](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
    * [Install and setup Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
    * [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)    
    * [kubectl untainted Node](https://forum.linuxfoundation.org/discussion/846483/lab2-1-kubectl-untainted-not-working)    
