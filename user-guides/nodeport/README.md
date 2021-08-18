# F5 CIS deployed in EKS Anywhere 

Amazon EKS Anywhere (EKS-A) is a Kubernetes installer based on and used by Amazon Elastic Kubernetes Service (EKS) to create reliable and secure Kubernetes clusters. This user-guide is created to document and validate F5 BIG-IP and F5 CIS integration with Amazon EKS Anywhere. More information on [EKS Anywhere](https://aws.amazon.com/eks/eks-anywhere/)

## The Easiest Way is Via a NodePort

NodePort is named quite literally like many other functional components within Kubernetes. It is an open port on every worker node in the cluster that has a pod for that service. When traffic is received on that open port, it directs it to a specific port on the ClusterIP for the service it is representing. In a single-node cluster this is very straight forward. In a multi-node cluster the internal routing can get more complicated. In that case its best using an F5 BIG-IP load balancer so you can spread traffic out across all the nodes and be able to handle failures a bit easier.

NodePort is great, but it has a few limitations. Ports available to NodePort are in the 30,000 to 32,767 range. [Source](https://platform9.com/blog/understanding-kubernetes-loadbalancer-vs-nodeport-vs-ingress/)

![diagram](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/servicetypelb/diagram/2021-04-27_10-11-10.png)

Demo on YouTube [video]()

## Prerequisites

* Recommend AS3 version 3.30.0 [repo](https://github.com/F5Networks/f5-appsvcs-extension/releases/tag/v3.30.0)
* CIS 2.5.1 [repo](https://github.com/F5Networks/k8s-bigip-ctlr/releases/tag/v2.5.1)

### Step 1 Create the CIS Deployment Configuration

Change the logging level to INFO once deployed 

```
args: 
  - "--bigip-username=$(BIGIP_USERNAME)"
  - "--bigip-password=$(BIGIP_PASSWORD)"
  - "--bigip-url=192.168.14.45"
  - "--namespace=default"
  - "--bigip-partition=eks"
  - "--pool-member-type=nodeport"
  - "--log-level=DEBUG"
  - "--insecure=true"
  - "--custom-resource-mode=true"
  - "--as3-validation=true"
  - "--log-as3-response=true"
```

Deploy CIS and CRD schema

```
kubectl create -f f5-cluster-deployment.yaml
kubectl create -f customresourcedefinitions.yaml
```

* cis-deployment [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/servicetypelb/cis-deployment/f5-cluster-deployment.yaml)
* crd-schema [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/blob/main/user_guides/servicetypelb/crd-schema/customresourcedefinitions.yaml)

### Step 2 Create the Service and Deployment

Create the pod deployments and services for the test and production application

```
kubectl create -f f5-demo-test-service.yaml
kubectl create -f f5-demo-production-service.yaml
```

pod-deployments [repo](https://github.com/mdditt2000/k8s-bigip-ctlr/tree/main/user_guides/servicetypelb/pod-deployment)