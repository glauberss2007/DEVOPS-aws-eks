# eks-demo

## Create EKS cluster
- [INstalling AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- [Create an user with delegated permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)
PS: Remember to copy the access key

After user creation configure the CLI:
````
aws configure
````
- [Installing kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
``
kubectl version --short --client
``
- You can install eksctl using [this](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
````
eksctl version
````
### Spin up a very fast cluster
Comands examples (Change <> by the name removing the <> symbols):
````
eksctl create cluster
eksctl create cluster --name <> --version 1.21 --node-type t3.micro --nodes 2
eksctl create cluster --name <> --version 1.21 --nodegroup-name <> --node-type t3.micro --nodes 2 --managed
eksctl create cluster --name <> --fargate
````
PS:[EKS Cluster in AWS](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)

Execute the comand to create the EKS cluster and then check the resource in AWS GUI console:
````
eksctl create cluster --name test --version 1.21 --nodegroup-name test-group --node-type t3.micro --nodes 2 --managed
````

Now you can use kubectl comands:
``
kubectl get all
``

Lets create the nodegroup using yaml file ([what is a nodegroup?](https://www.pulumi.com/blog/day-2-kubernetes-migrating-eks-nodegroups-with-zero-downtime)):
Clone this repo or copy the "eksctl-create-ng.yaml" inside CLI host and then:
````
eksctl create nodegroup --config-file=eksctl-create-ng.yaml
eksctl get nodegroup --cluster=test
````

Lets create a EKS cluster using a manifest parameter file:
````
eksctl create cluster --config-file=eksctl-create-cluster.yaml
````

Now lets delete our example cluster to avoi unecessary cost:
``
eksctl get cluster
eksctl delete cluster --name=test
eksctl get cluster
``

## Replicaset & Deployment
````
kubectl apply -f nginx-deployment-withrolling.yaml
````

Delete a pod:
````
kubectl delete pod <podname>
````

Delete replicaset:
````
kubectl delete rs <rsname>
kubectl describe deployment
````

Update nginx version changing the yml and them reapply it
````
kubectl apply -f nginx-deployment-withrolling.yaml
````

Use the watch parameter to see the canarian deployment:
````
kubectl get rs --watch
````

### Demo - load balance service

Lets deploy our nodeport service using the manifest in this repository:
````
kubectl apply -f loadbalancer-service.yaml
````

### Demo - NodePort service

Lets deploy our nodeport service using the manifest in this repository:
````
kubectl apply -f nodeport-service.yaml
kubectl get all
kubectl get pods
kubectl get pods -o wide
````

If necessary create the nodeportsecurity group in the VPC that K8S is runing an then an inbound rule, from anywhere, for the port used.

PS: Declarative is a way to document the infrastructure too!

## EC2 instance type and POD limit

- Max number of allowed pod depends on EC2 instance type
- Bigger the instance type, more pods

In this repositore you can see a max pod calculator and a list of EC2 type and recomended limit of pods for each one:

- link: https://github.com/awslabs/amazon-eks-ami/blob/master/files/eni-max-pods.txt

In K8S there are some PODs responsable for the system, that you can see using:
````
kubectl get ns
kubectl get pods -n kube-system
````
Lets deploy a simple deployment to check:
``
kubectl apply -f nginx-deployment.yaml
kubectl get all
``

Increase the replica number in the yaml used and apply it again:
``
kubectl apply -f nginx-deployment.yaml
kubectl get pods
``

Some of them anr pending status, lets check it using describe command:
``
kubectl describe pod <podName>
``
PS: Warning: Insiffitient pods

## EKS managed nodegroups (AWS feature)

Imaging everything is working well and some task appear:
- K8S needs update
- Deploy Security patches
- EC2 AMI need to be updated

To resolve it in an self managed K8S we need to orchestrate an High Availabile (HA) env to reach a zero downtime, otherwise the APP will goes down when we create the new EC2 AMI node.

EKS managed nodegroups resolve it by providing:

![image](https://user-images.githubusercontent.com/22028539/129732351-2bae65e3-d192-4030-b8c2-d201833ef792.png)

![image](https://user-images.githubusercontent.com/22028539/129732743-118d8286-1cfb-42e0-bd65-e1b312a9ed3f.png)

## Manage nodegroup demo

Lest create our eks cluster:
````
eksctl create cluster --name demoeks --version 1.21 --nodegroup-name demoeks-nodes --node-type t3.micro --nodes 4 --managed
````

Then deploy some services using manifests:
````
kubectl apply -f nginx-deployment.yaml
kubectl get all
````

So in order to update the k8s you just need to access the AWS GUI console and go to "Services > Resource Group > Amazon Container Services > Cluster" and click on UPDATE NOW buttom!

To update a nodegroup using CLI you can use:
````
eksctl upgrade nodegroup --name-managed-ng-1 --cluster=managed-cluster
````

You can use kubectl get nodes to see the rolling up proccess!

PS: Nodegroup is free of charge!

## Helm and charts

What does it solves?

Example to deploying wordpress:

It is difficult to manage multiple manifests because every time something cahnge you need to update and re apply multiple ymls...

![image](https://user-images.githubusercontent.com/22028539/129738630-8e873032-3ffe-4385-9620-9166779de950.png)

Helm and charts can help us on it!

![image](https://user-images.githubusercontent.com/22028539/129740573-d6f12bd0-2fd5-4484-9ade-0863e79b20b5.png)

An example of chart structure is:

![image](https://user-images.githubusercontent.com/22028539/129740837-be307992-6886-4597-b22a-60b5aa55378c.png)

## Helm demo

Installing helm [installation](https://helm.sh/docs/intro/install/):
``
choco install kubernetes-helm
helm version
helm search repo
helm search hub
``

Add repo and search for nginx one:
``
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo nginx
``

Dowloading a helm repo:
``
helm pull bitnami/nginx --untar=true
``
PS: in this repo you can see the yml and files.

Let install it on our K8S:
``
kubectl get all
helm install helm-nginx bitnami/nginx
kubectl get all
kubectl get service
kubectl describe svc
``
## Design an APP on EKS

![image](https://user-images.githubusercontent.com/22028539/129767462-05f5ee0d-6671-4140-a42f-80456e9950ed.png)

In this simple project we dont have a backend, so the structure is like this one:

![image](https://user-images.githubusercontent.com/22028539/129768133-9339acbc-186a-499e-a704-66c48bd79a43.png)

The guestbook-go project is in https://github.com/glauberss2007/examples forked from https://github.com/kubernetes/examples

Clone the repo above to your machine and them use VS Code to open the guestbook-go and run the json manifest acording to steps showed on README

App structure inside EKS:

![image](https://user-images.githubusercontent.com/22028539/129779865-c1b302e3-36d7-4076-a5d0-e897828b22d3.png)

## Wordpress using helm

The tutorial is in https://bitnami.com/stack/wordpress/helm



