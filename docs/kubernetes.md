# Setting-up Kubernetes

If you already have access to a Kubernetes cluster, then skip to [Configuring Ingress](#configuring-ingress) - you are almost ready-to-go. If not, then please read on.

## Deploying Locally

If you are new to Kubernetes or want to test deployments locally, then we recommend that you start by installing [Minikube](https://minikube.sigs.Kubernetes.io/docs/). Minikube will enable you with everything you need to use Bodywork, it is well documented and supported by a large community.

## Managed Kubernetes Services

When you are ready to deploy to the cloud, then the easiest path is via a managed Kubernetes service from one of the following cloud infrastructure providers:

- [AWS](https://aws.amazon.com/eks)
- [Azure](https://docs.microsoft.com/en-us/azure/aks/)
- [GCP](https://cloud.google.com/kubernetes-engine/)
- [Digital Ocean](https://www.digitalocean.com/products/kubernetes/)
- [Scaleway](https://www.scaleway.com/en/kubernetes-kapsule/)

## Required Kubernetes Version

Bodywork relies on the official [Kubernetes Python client](https://github.com/kubernetes-client/python), whose latest version (12.0.0) has full compatibility with Kubernetes 1.16. We recommend that you also use Kubernetes 1.16, but in-practice Bodywork will work with other versions - more information can be found [here](https://github.com/kubernetes-client/python#compatibility). Bodywork is tested against Kubernetes 1.16 running on [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/).

## Installing the kubectl Tool

Kubectl is the command-line tool that lets you control your Kubernetes cluster - see [here](https://kubernetes.io/docs/reference/kubectl/overview/) for an overview. Bodywork does **not** use kubectl (it talks directly to the Kubernetes API instead), and so it is **not** a requirement. Regardless, kubectl is an essential tool to have access to, so we strongly recommend that you install it - see [here](https://kubernetes.io/docs/tasks/tools/) for instructions.

## Configuring Ingress

If you want to expose Bodywork-deployed services to requests from outside your cluster, then you need to install the [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) within your cluster. This will act like an [API Gateway](https://www.nginx.com/learn/api-gateway/) for your cluster, that will route external HTTP requests to internal services.

The NGINX Ingress controller is an official Kubernetes project and can be installed with a single command - for local a Minikube cluster you would use,

```shell
$ minikube addons enable ingress
```

Or for EKS on AWS you would use,

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/aws/deploy.yaml
```

The precise details for every potential Kubernetes deployment option are listed [here](https://kubernetes.github.io/ingress-nginx/deploy/#minikube).

Managed Kubernetes services will also provision an external [load balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing)) to manage the flow of traffic to the ingress controller (and hence the cluster). Note, this will have an associated cost that is additional to that of the Kubernetes cluster.

### Connecting to the Cluster

Get the public-facing IP address for your ingress controller with the following command,

```shell
kubectl -n ingress-nginx get service ingress-nginx-controller
```

And make a note of the `EXTERNAL-IP` field, that will have to be used for all requests to Bodywork-deployed services originating from outside the cluster. Services within the cluster can communicate with one another using the cluster's internal network.

## Learning Kubernetes

Familiarity with basic [Kubernetes concepts](https://kubernetes.io/docs/concepts/) and some exposure to the [kubectl](#installing-the-kubectl-tool) command-line tool will make life easier, but are not essential. If you would like to learn a bit more about Kubernetes, then we recommend the first two introductory sections of Marko Lukša's excellent book [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action?query=kubernetes), or the introductory article we wrote on [Deploying Python ML Models with Flask, Docker and Kubernetes](https://alexioannides.com/2019/01/10/deploying-python-ml-models-with-flask-docker-and-kubernetes/).

## Getting Help

If you need help with Kubernetes, then please don't hesitate to [contact us](https://github.com/bodywork-ml/bodywork-core/discussions) and we'll do our best to get you up-and-running.
