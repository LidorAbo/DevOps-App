# CI/CD for App

<img src="https://miro.medium.com/max/700/1*TNJ7Rpr5G1OJHtKH-IBEFw.png" width="500">

## Description

This project build [CI/CD] process for App in order to deploy App locally on debian linux distribution. The App required several services to work properly:

* Streamer - read video stream frames

* Analyzer - analyze the frames

* Publisher - publish out the annotated frames

* Notifier - notify users about events

## Table of contents

* [Installation](#Installation)
  * [python3](#python3)
    * [pip3](#pip3)
    * [flask](#flask)
  * [docker](#docker)
  * [minikube](#minikube)
  * [kubectl](#kubectl)
  * [Jenkins](#Jenkins)
* [Deploy](#deploy)
  * [Local registry](#local-registry)
  * [minikube k8s cluster](#minikube-k8s-cluster)
* [CI/CD](#ci-cd)
  * [CI](#ci)
  * [CD](#cd)
* [Access App](#access-app)
  * [Analyzer](#analyzer)
  * [Notifier](#notifier)
  * [Publisher](#publisher)
  * [Streamer](#streamer)
* [K8S minikube components](#K8s-minikube-components)
  * [kube-apiserver](#kube-apiserver)
  * [etcd](#etcd)
  * [kube-scheduler](#kube-scheduler)
  * [kube-controller-manager](#kube-controller-manager)
  * [kubelet](#kubelet)
  * [kube-proxy](#kube-proxy)
  * [Container runtime](#container-run-time)

## Installation

### python3

#### pip3

Install pip3 package manager(for install flask module) with following bash commands:

```bash
sudo apt update 
sudo apt install -y python3-pip
```

Verify installition by running bash commands:

```bash
pip3 --version
```

The version number may vary, but it will look something like this:

```bash
pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.6)
```

#### flask

Install flask module(in using for each App service python file) with pip3 package manager by running following bash commands:

```bash
pip3 install flask
```

Verify installition by running following bash commands:

```bash
pip3 freeze | grep Flask
```

The version number may vary, but it will look something like this:

```bash
Flask==1.1.2
```

### docker

Install docker by using convention script by running following bash commands:

```bash
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
```

> **_NOTE:_** This script update remote repository to docker, update local repository and install  latest docker in your linux distribution.<br/>
> **_Optional:_** You can add current host user to docker group and run docker as current host user(non-root user).  

Verify that Docker Engine is installed correctly by running the hello-world image by root user:

```bash
sudo docker run hello-world 
```

Or by using by host user(if added to docker group):

```bash
docker run hello-world
```

In this step, [docker] is configured and  **Ready** for deploy local registry and [minikube] cluster

### kubectl

Install latest binary(used for accessing to [minikube] [K8S] cluster) with running following bash commands:

```bash
# Download binary
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kubectl
# Add execute permissions
chmod +x ./kubectl
# Adding binary to PATH - for access from any folder in the terminal
sudo mv ./kubectl /usr/local/bin/kubectl
```

Test to ensure the version you installed is up-to-date by running following bash commands:

```bash
kubectl version --client
```

You should see something like this:

```bash
kubectl version --client
Client Version: version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.1", GitCommit:"c4d752765b3bbac2237bf87cf0b1c2e307844666", GitTreeState:"clean", BuildDate:"2020-12-18T12:09:25Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```

> **_NOTE:_** The version is latest based time that commands was running.<br/>
> **_Optional:_** kubectl provides autocompletion support for bash which can save you a lot of typing. for more info please visit  [autocomplete-kubectl]

### minikube

Install binary with running following bash commands:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

Confirm installition with running following bash commands:

```bash
minikube version
```

You should see something like this:

```bash
minikube version: v1.16.0
commit: 9f1e482427589ff8451c4723b6ba53bb9742fbb1
```

In this step, [minikube]  is installed and  **Ready** for deploy [minikube] cluster

### Jenkins

Firstly install java that required in order to jenkins work properly with following bash commands:

```bash
sudo apt update
sudo apt install -y openjdk-11-jdk
```

Confirm installition with running following bash command:

```bash
java -version
```

The result must be something like:

```bash
openjdk version "11.0.9.1" 2020-11-04
OpenJDK Runtime Environment (build 11.0.9.1+1-post-Debian-1deb10u2)
OpenJDK 64-Bit Server VM (build 11.0.9.1+1-post-Debian-1deb10u2, mixed mode, sharing)
```

Run following bash commands to install jenkins:

```bash
# Add remote jenkins repository
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
# Update local repository
sudo apt-get update
# install jenkins
sudo apt-get install -y jenkins
```

Confirm installition by running following bash commands:

```bash
systemctl status jenkins
```

You should see a bright green entry that says ```active (exited)``` - This means the service is running. Exit the status screen by pressing **Ctrl+Z** key.

Chnage default user for jenkins by change ```JENKINS_USER``` to current host user in ```/etc/default/jenkins```(required to connect minikube-api cluster) by running following bash command:

```bash
# -E - supporting regex expressions
sudo sed -i -E   "s/(JENKINS_USER)=jenkins/\1=`whoami`/" /etc/default/jenkins  
```

Change ownership of jenkins folders to current host user(required to create folders and files by changed default user of jenkins - ```whoami```) by running following bash command:

```bash
# Change user and group owner to current host user
sudo chown -R `whoami`:`whoami` /var/lib/jenkins/
sudo chown -R `whoami`:`whoami` /var/log/jenkins/
sudo chown -R `whoami`:`whoami` /var/cache/jenkins/
sudo chown -R `whoami`:`whoami` /var/run/jenkins/
```

Restart jenkins service to apply chnages with following bash command:

```bash
systemctl restart jenkins
```

Confirm installition again by running following bash command:

```bash
systemctl status jenkins
```

You should see a bright green entry that says ```active (exited)``` - This means the service is running. Exit the status screen by pressing **Ctrl+Z** key.

Access jenkins on url combined with YOUR_SERVER_IP and default PORT - 8080: ```http://YOUR_SERVER_IP:PORT```

You should see the unlock Jenkins screen, which displays the location of the initial password:
&nbsp;
&nbsp;

![alt text](https://assets.digitalocean.com/articles/jenkins-install-ubuntu-1604/unlock-jenkins.png)

In terminal window, run following bash command to display the password:

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the 32-character alphanumeric password from the terminal and paste it into the **Administrator** password field, then click **Continue**.

The next screen presents the option of installing suggested plugins or selecting specific plugins - You need click **Install suggested plugins** to install most useful plugins in [CI/CD] process:

![alt text](https://assets.digitalocean.com/articles/jenkins-install-ubuntu-1804/customize_jenkins_screen_two.png)

When the installation is complete, you will be prompted to set up the first administrative user. Create administrative user based of your choice:

![alt text](https://assets.digitalocean.com/articles/jenkins-install-ubuntu-1804/jenkins_user_info.png)

Fill all the fields in this screen: Username and Password based on your choice, Full name and E-mail address. After that click **Save and Continue**.

After that, Instance configuration page will show up as shown below:

![alt text](https://miro.medium.com/max/700/1*1GY9pShE9k1xYzNDUrnHBg.png)

You can change the default Jenkins URL if you want to. I’ll be using the default one. Click on save and finish to finish the setup. Click on **Start Using Jenkins** and Jenkins Dashboard will be displayed as shown below:

![alt text](https://miro.medium.com/max/700/1*f_duadFyV9MHpFw0pxarQg.png)

In this step, [Jenkins] is configured and  **Ready** for build [CI/CD] process.

## Deploy

### Local registry

Run following bash commands to deploy private registry in your local host:

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

Verify that local registry work properly by following bash commands:

```bash
#Pusing image to local registry that runs as container in your local host
docker run -it hello-world
docker tag hello-world localhost:5000/hello-world
docker push localhost:5000/hello-world
#Remove local images - to ensure that you can check pull images from local registry 
docker rmi -f hello-world localhost:5000/hello-world
# Test pulling image from local registry
docker run localhost:5000/hello-world
```

You should see something like this:

```bash
Status: Downloaded newer image for localhost:5000/hello-world:latest
```

In this step, local registry is configured and  **Ready** for using in CI process from [Jenkins] job.

### minikube k8s cluster

Starting [minikube] cluster by running following bash commands:

```bash
minikube start --insecure-registry="192.168.1.0/24" --mount --mount-string="/mnt/data/:/mnt/data"
```

> **_NOTE:_** The above command starts cluster based docker driver(run in docker container) and allow accessing from minikube to insecure docker (not via port 443) that runs in the host locally in docker container. Also create bind mount between host path and minikube(docker container) that required to mount volume in postgresql db path.

You should see something like this:

```bash
😄  minikube v1.16.0 on Ubuntu 20.04 (vbox/amd64)
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.20.0 preload ...
    > preloaded-images-k8s-v8-v1....: 491.00 MiB / 491.00 MiB  100.00% 5.42 MiB
🔥  Creating docker container (CPUs=2, Memory=2900MB) ...
🐳  Preparing Kubernetes v1.20.0 on Docker 20.10.0 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

Confirm healthy one-node [K8s] cluster - Check healthy of main [K8s] components(pods in kube-system namespace) by running following bash commands:

```bash
minikube get pods -A
```

You should see something like this:

```bash
NAME                               READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-dkn7g            1/1     Running   0          25h
etcd-minikube                      1/1     Running   0          25h
kube-apiserver-minikube            1/1     Running   0          25h
kube-controller-manager-minikube   1/1     Running   0          25h
kube-proxy-x9mbj                   1/1     Running   0          25h
kube-scheduler-minikube            1/1     Running   0          25h
storage-provisioner                1/1     Running   0          25h
```

Add metrics-server addon to minikube [K8s] cluster by running following bash commands:

```bash
minikube addons enable metrics-server
```

You should see something like this:

```bash
🌟  The 'metrics-server' addon is enabled
```

Verify metrics-server pod healthy by running following bash command:

```bash
kubectl get pods -A | grep metrics

```

You should see something like this:

```bash
kube-system   metrics-server-d9b576748-c54ds     1/1     Running   0          4m36s
```

In this step, [minikube] is configured and ready to deploy App(Deploy [K8s] objects such as pods, services,pv and pvc that deployed to cluster in CD process later).

## CI-CD

This process builds and deploys app services in jobs while each job based Jenkinsfile(includes pipeline) for each sub process(first CI and CD that triggered in the end of CI process)
> **_NOTE:_** This process requires to configure to seperate jobs(for running CI and CD pipelines) in jenkins dashboard based each branch in [CI-CD repo]. see <https://www.jenkins.io/doc/book/pipeline/getting-started/#defining-a-pipeline>
> **_NOTE:_** If any of stage is failed(for one of pipelines) the appropriate pipeline is failed based stage in specific pipeline that failed(achieved by using [exceptions], [when] and  [groovy variables]).

### CI

This pipeline includes few stages that required to performing test and build App services:

#### Checkout

Checkouts to main branch in [App repo] to App folder - Includes Dockerfile(using for build image and deploy to local registry in later stage) and python file (using for sanity test and for entrypoint for each docker image) for each service.

#### Sanity check

Checks if each service app response to GET request after running flask server locally for each service(run in port 5000). while all app services returns 200 HTTP code the stage completly successfully.

> **_NOTE:_** This step running in dir(checked out in previous step in appropriate branch in [App repo]) that includes all the appropriate files(python and Dockerfile files for each service)
**_NOTE:_** Achieved by using bash commands that runs with [sh] and externl local [bash] script.

#### Deploy local registry

Test access to local registry such as described in [Local registry](#local-registry)

#### Build App services containers and push to local registry

Builds each service Dockerfile (based appropriate python file and flask docker image) and pushes each docker images to local registry(using in CD job to deploy App services to the cluster)

#### Trigger CD job

Triggers CD job with appropriate groovy variables - service port and App services(that used in pod and service in [minkube] cluster)

### CD

This pipeline includes few stages that required to deploy App services:

#### Checkout

Checkouts to main branch in [Devops repo] to [K8s] folder - Including all [K8s] yaml files for objects in [minikube] cluster for each App service(service, pod and horozinal pod autoscaler) and yaml file for pv(persistent volume) and pvc(persistent volume claim) for applying shared path to all pods in the cluster.

#### Deploy [K8s] services

In [K8s] folder first applies pv and pvc objects to [minikube]   cluster by applies appropriate yaml file by running following bash commands:

```bash
kubectl apply -f db-persistent-volume.yaml
```

 second applies appropriate [K8s] objects(pod, service and hpa) to [minikube] cluster for each App service by applies appropriate yaml files for each service by running following bash command for each service:

```bash
kubectl apply -f SERVICE_FOLDER/
```

> **_NOTE:_** each pod and service run in port 5000 except streamer that runs service with port 5001 while pod not has incosistnent IP and service has consistent IP(Allowing to recreate deployment(as needed) and to get access to each service via permenant IP). Also each pod for each service mount shared path(mounted with pv and pvc as /mnt/data path in the host path) in postgresql path(/var/lib/postgresql/data).
> **_NOTE:_** hpa create pods from specific service when cpu utilization is 50 percentages and above while min pods is 1 and max pods is 10(required to cpu request for each service pod that defined in pod object for each service).
> **_NOTE:_** each K8s service for each App service can find the appropriate pod by using   selector(shared label)

#### Expose ports locally

Exposes App services locally to host by using [parallel], [kubectl port-forward] and [minikube tunnel] by running following bash commands:

```bash
kubectl apply -f svc/app-analyzer-svc 5002:5000
```

Forwards port 5002 in localhost to analyzer service in [minikube] [K8s] cluster.

```bash
kubectl apply -f svc/app-notifier-svc 5003:5000
```

Forwards port 5003 in localhost to notifier service in [minikube] [K8s] cluster

```bash
kubectl apply -f svc/app-publisher-svc 5004:
```

Forwards port 5004 in localhost to publisher service in [minikube] [K8s] cluster

```bash
minikube tunnel
```

Expose streamer service as its ip(EXTERNAL-IP) in the [minikube] [K8s] cluster that you can access to service outside from cluster with this ip.

When this stage finish completly [CI/CD] process has completed and the App tested and deployed completly.

## Access App

### Analyzer

```bash
curl localhost:5002
```

You should see something like this:

```bash
Analyzing the frames...
```

### Notifier

```bash
curl localhost:5003
```

You should see something like this:

```bash
Notifying users about events...
```

### Publisher

```bash
curl localhost:5004
```

You should see something like this:

```bash
Publishing out the annotated frames...
```

### Streamer

```bash
curl CLUSTER_SERVICE_IP:5004
```

You should see something like this:

```bash
Reading the video stream frames...
```

## K8S minikube components

### Nodes

[minikube] [K8s] cluster running in worker-master(control plane) node(docker container) that running all the containerized applications in pods based own docker daemon.

### kube-apiserver

Main component of [minikube] [K8s] cluster that aim to apply all requests to the cluster(apply objects such as pods, services, hpa, rules, etc)

> **_NOTE:_** Required config file(Includes appropriate certificates and URL of minikube cluster) in .kube dir in user's home directory in order to access and make changes in the [minikube] [K8s] cluster.

### etcd

Main component of [minikube] [K8s] cluster that functions as db(highly-available key value store) to store status for each object in the minikube cluster and based key values entries planning operations(such as apply deployments based replicas for example).

### kube-scheduler

Control plane component that aim to watches of request of creating new pods and assign them to specific node(in case of minikube only minikube node).

> **_NOTE:_** optional to assign pods based factors such as: hardware/software, deadlines, etc.

### kube-controller-manager

Main process in [minikube] cluster that have few responsibilities:

* Node controller: Responsible for noticing and responding when [minikube] node(docker container) goes down
* Replication controller: responsible for update pods to desired state based user request that sent to [minikube] [K8s] cluster via kube-apiserver
* Endpoints controller: responsible for creating objects in [minikube] node(docker container)

### kubelet

 worker node component that responsible for ensure that containers that defined in PodSpecs running and healthy.

> **_NOTE:_** kubelet doesn't manage containers which were not created by [minikube] [K8s] cluster.

### kube-proxy

kube-proxy maintains network rules on [minikube] node(docker container). This network rules allowing to access pods inside and outside from [minikube] [K8s] cluster. Implementing part of Service concept in [K8s](When you access to service IP in specific PORT you will be redirected to appropriate pod IP in specific PORT)

> **_NOTE:_** PORT will be different in service and pod.

### Container runtime

The container runtime is the software that is responsible for running containers - In [minikube] node is [docker].

[CI/CD]: <https://en.wikipedia.org/wiki/CI/CD>
[Jenkins]: <https://www.jenkins.io/>
[K8s]: <https://kubernetes.io/>
[minikube]: <https://minikube.sigs.k8s.io/docs/start/>
[docker]: <https://www.docker.com/>
[autocomplete-kubectl]: <https://kubernetes.io/docs/tasks/tools/install-kubectl/#optional-kubectl-configurations>
[CI-CD repo]: <https://github.com/LidorAbo/ci-cd.git>
[DevOps repo]: <https://github.com/LidorAbo/DevOps-App.git>
[App repo]: <https://github.com/LidorAbo/App.git>
[exceptions]: <https://www.jenkins.io/doc/book/pipeline/syntax/#flow-control>
[when]: <https://www.jenkins.io/doc/book/pipeline/syntax/#when>
[groovy variables]: <http://groovy-lang.org/semantics.html>
[sh]: <https://www.jenkins.io/doc/pipeline/tour/running-multiple-steps/#linux-bsd-and-mac-os>
[bash]: <https://en.wikipedia.org/wiki/Bash_(Unix_shell)>
[parallel]: <https://www.jenkins.io/blog/2017/09/25/declarative-1/#parallel-stages-with-declarative-pipeline-1-2>
[kubectl port-forward]: <https://github.com/LidorAbo/DevOps-App#kubectl>
[minikube tunnel]: <https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/#forward-a-local-port-to-a-port-on-the-pod>
[minikube tunnel]: <https://minikube.sigs.k8s.io/docs/commands/tunnel/#minikube-tunnel>

