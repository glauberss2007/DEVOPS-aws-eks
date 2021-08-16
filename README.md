# eks-demo

## Create EKS cluster
- Create your [EKS Cluster in AWS](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)
- You can install eksctl using [this](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

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
