---
layout: post
title:  "Kubernetes on Windows 7/ Windows 10 Home"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Cloud Native, DevOps]
tags: [Kubernetes, Minikube, Docker]
---

## Overview

Configuring and deploying Kubernetes in Windows 7 / Windows 10 Home edition is bit challenging than the versions which support HyperV-1 enviroment. In this article we will look into the setup of Kubernetes environment in Windows 10 Home edition which is also applicable to Windows 7.

I have used Git bash terminal to run the commands which gives you the feel of using Linux bash shell.

## Part 1 - Docker and Minikube Setup

## Install Docker

Since the OS version does not support HyperV-1 , we have to use Legacy docker installation using DockerToolbox. Refer the [link](https://docs.docker.com/toolbox/toolbox_install_windows) to get the information on DockerToolbox instllation.

DockerToolbox contains the VirtualBox that creates the HyperV-2 environment which enables the installation of Kubernetes. Once the docker is installed, then start the docker machine by using the "Docker Quickstart Terminal" shortcut.

Important folders of docker in Windows
        /c/Users/{user}/.docker is the place where all docker machines are created.
        /c/Users/{user}/.docker/cache has the boot2docker imange 
        Note: boot2docker image is the light linux distribution running on VirtualBox.
        
        If you wants to add proxy configuration after docker machine creation then add it in the "Env" attribute of "HostOptions" in the config file of the docker machine (eg: /c/Users/{user}/.docker/machine/machines/default)

## Install Kubectl

Download the latest release v1.14.0 from [this link](https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/linux/amd64/kubectl).
    1. Add the binary location to your PATH environment variable.
    2. Test to ensure the version you installed is up-to-date, run the command
        kubectl version

## Install Minikube

Minikube is a tool which creates an environment to run Kubernetes locally. Minikube creates Single node cluster in the VirtualBox.

To install Minikube manually on Windows , download [minikube-windows-amd64](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64), rename it to minikube.exe and add it to the PATH variable.

Once the minikube is installed, you can start the cluster with the command:
        minikube start -p <cluster name>

This will download the Minikube ISO and then create a machine and it can be verified in /c/Users/{user}/.minikube/machines

## Setting up local Registry 

1. Create a Registry in Minikube

    The idea here is for the docker daemon on minikube to be able to pull from a registry called localhost:5000.

    This is achieved by actually running a registry on minikube and then setting up a proxy so that the minikube VM port 5000 maps to the registry’s 5000.

    Create a registry (a replication-controller and a service) and create a proxy to make sure the minikube VM’s 5000 is proxied to the registry service’s 5000.

    kubectl create -f kube-registry.yaml

    Get the kube-registry.yaml from [this link](https://gist.github.com/coco98/b750b3debc6d517308596c248daf3bb1)

Now, we need to make sure that the docker daemon on docker-machine thinks that localhost:5000 is legit.

So, we map 5000 of the docker-machine ⇢ 5000 of the host (via a reverse SSH tunnel) and 5000 of the host ⇢ 5000 of the registry running on minikube (via kubectl port-forward)

2. Map the host port 5000 to minikube registry pod

        kubectl port-forward --namespace kube-system \ 
        $(kubectl get po -n kube-system | grep kube-registry-v0 | \awk '{print $1;}') 5000:5000
3. Map docker-machine’s 5000 to the host’s 5000

Go to the bash command shell and then run the below command. If you have installed Git then you can see it bash.exe in Git installation folder

        ssh -i ~/.docker/machine/machines/default/id_rsa \
        -R 5000:localhost:5000 \docker@$(docker-machine ip)

## Part 2- Create Docker Image & Publish

Create a "Dockerfile" in your project directory so that docker command can read it to prepare the image. Given below the sample structure.

        FROM openjdk:8-jre-alpine

        ENV APP_FILE cloudmessage-customer-registration-0.0.1-SNAPSHOT.war

        ENV APP_HOME /usr/apps

        #ENV SPRING_CONFIG_LOCALTION="classpath:/application.properties"

        #ENV SPRING_PROFILES_ACTIVE="development"

        EXPOSE 8080

        COPY target/$APP_FILE $APP_HOME/

        WORKDIR $APP_HOME

        ENTRYPOINT ["sh", "-c"]

        CMD ["exec java -jar $APP_FILE"]

Run the below commands to create and publish the artifact to registry.

        docker build -t cloudmessage-customer-registration .
        docker tag cloudmessage-customer-registration localhost:5000/ cloudmessage-customer-registration
        docker push localhost:5000/ cloudmessage-customer-registration

## Pod Deployment

Run the below command to deploy the artifact and create the Pod in Kubernetes

        kubectl run cloudmessage-customer-registration --image=localhost:5000/ cloudmessage-customer-registration

Expose the service so that it can be accessed.

        kubectl expose deployment cloudmessage-customer-registration --type=NodePort --port=8080

To check the IP and port of Application run below command 

        kubectl describe services cloudmessage-customer-registration    

Result will look like below.

        Name:                     cloudmessage-customer-registration
        Namespace:                default
        Labels:                   run= cloudmessage-customer-registration
        Annotations:              <none>
        Selector:                 run= cloudmessage-customer-registration
        Type:                     NodePort
        IP:                       10.106.115.96
        Port:                     <unset>  8080/TCP
        TargetPort:               8080/TCP
        NodePort:                 <unset>  31108/TCP
        Endpoints:                172.17.0.6:8080
        Session Affinity:         None
        External Traffic Policy:  Cluster
        Events:                   <none>

You will be able to access the service by using cluster ip and NodePort. You can see the cluster IP in the output of  "minikube start" command.

## Helpfull commands to verify the deployments

1. Verify Your Deployments

The below commands help you the verify the deployments.

        kubectl get deployments → this will show all the deployments in the cluster
        kubectl get pods → this will show you the containers in the cluster
        kubectl get services → this will show you the service and port
        kubectl log <pod id>→ this will show the logs of a given pod
 

2. Stop Minikube

The below given commands can be used to delete deployments and stop minikube. Here the deployment name is " cloudmessage-customer-registration"

        kubectl delete services <deployment name>
        kubectl delete deployment <deployment name>
        minikube stop -p <cluster name>
 

3. Stop Docker Machine

The docker machine could be stopped by the command:

        docker-machine stop

## Conclusion

In this article, i tried to explain how to setup a kubernetes cluster with Legacy Docker and Minikube. In the first part it explains the setup of docker and kubernetes and in the second part deploying spring boot application.
I have taken the details of local registry setup and port forwaring from the link 
https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615/