# Setting-up Kubernetes

If you already have access to a Kubernetes cluster, then skip to [Configuring Ingress](#configuring-ingress). If you are new to Kubernetes, then skip to [Getting Started with Kubernetes](#getting-started-with-kubernetes). Otherwise, please read on.

## Deploying Locally

If you want to test deployments locally, then you can run Kubernetes on your local machine with the help of one of the following tools:

- [Minikube](https://minikube.sigs.Kubernetes.io/docs/)
- [Kubernetes in Docker (KIND)](https://kind.sigs.k8s.io)
- [k3s]
- [Docker for Desktop](https://www.docker.com/products/docker-desktop)

We currently recommend Minikube, as it comes packaged with useful add-ons (e.g., ingress and the Kubernetes dashboard), that make life easy for those starting-out with Kubernetes. It is also well documented and supported by a large community.

## Managed Kubernetes Services

When you are ready to deploy to the cloud, then the easiest path is via a managed Kubernetes service from one of the following cloud infrastructure providers:

- [AWS](https://aws.amazon.com/eks)
- [Azure](https://docs.microsoft.com/en-us/azure/aks/)
- [GCP](https://cloud.google.com/kubernetes-engine/)
- [Digital Ocean](https://www.digitalocean.com/products/kubernetes/)
- [Scaleway](https://www.scaleway.com/en/kubernetes-kapsule/)

## Required Kubernetes Version

Bodywork relies on the official [Kubernetes Python client](https://github.com/kubernetes-client/python), whose latest version (12.0.1) has full compatibility with Kubernetes 1.16. We recommend that you also use Kubernetes 1.16, but in-practice Bodywork will work with other versions - more information can be found [here](https://github.com/kubernetes-client/python#compatibility). Bodywork is tested against Kubernetes 1.16 running on [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/).

## Installing the Kubectl Tool

Kubectl is the command-line tool that lets you control your Kubernetes cluster - see [here](https://kubernetes.io/docs/reference/kubectl/overview/) for an overview. Bodywork does **not** use Kubectl (it talks directly to the Kubernetes API instead), and so it is **not** a requirement. Regardless, Kubectl is an essential tool to have access to, so we strongly recommend that you install it - see [here](https://kubernetes.io/docs/tasks/tools/) for instructions.

## Configuring Ingress

If you want to expose Bodywork-deployed services to requests from outside your cluster, then you need to install the [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) within your cluster. This will act like an [API Gateway](https://www.nginx.com/learn/api-gateway/) for your cluster, that will route external HTTP requests to internal services.

The NGINX Ingress controller is an official Kubernetes project and can be installed with a single command - for local a Minikube cluster you would use,

```text
$ minikube addons enable ingress
```

Or for EKS on AWS you would use,

```text
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/aws/deploy.yaml
```

The precise details for every potential Kubernetes deployment option are listed [here](https://kubernetes.github.io/ingress-nginx/deploy/#minikube).

Managed Kubernetes services will also provision an external [load balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing)) to manage the flow of traffic to the ingress controller (and hence the cluster). Note, this will have an associated cost that is additional to that of the Kubernetes cluster.

### Connecting to the Cluster

Get the public-facing IP address for your ingress controller with the following command,

```text
kubectl -n ingress-nginx get service ingress-nginx-controller
```

And make a note of the `EXTERNAL-IP` field, that will have to be used for all requests to Bodywork-deployed services originating from outside the cluster. Services within the cluster can communicate with one another using the cluster's internal network.

## Getting Started with Kubernetes

An easy way to get started with Kubernetes, is with [Minikube](https://minikube.sigs.k8s.io/docs/). Minikube enables you to easily create and manage single-node Kubernetes clusters on your local machine, via the command line. It comes with everything you need to use Bodywork and prepare for deploying to remote clusters.

If you are running on MacOS with the [Homebrew](https://brew.sh) package manager available, then installing Minikube is as simple as running,

```text
$ brew install minikube
```

If you’re running on Windows or Linux, then follow the appropriate [installation instructions](https://minikube.sigs.k8s.io/docs/start/).

### Creating your first Cluster

Once you have Minikube installed, start a cluster using the latest version of Kubernetes that Bodywork supports,

```text
$ minikube start --kubernetes-version=v1.16.15
```

And then enable ingress, so we can route HTTP requests to services deployed using Bodywork.

```text
$ minikube addons enable ingress
```

You’ll also need the cluster’s IP address, which you can get using,

```text
$ minikube profile list

|----------|-----------|---------|--------------|------|----------|---------|-------|
| Profile  | VM Driver | Runtime |      IP      | Port | Version  | Status  | Nodes |
|----------|-----------|---------|--------------|------|----------|---------|-------|
| minikube | hyperkit  | docker  | 192.168.64.5 | 8443 | v1.16.15 | Running |     1 |
|----------|-----------|---------|--------------|------|----------|---------|-------|
```

When you’re done with this tutorial, the cluster can be powered-down using.

```text
$ minikube stop
```

### Basic Concepts

Here is a brief introduction to the most common types of Kubernetes resources, with a guide to how Bodywork uses them to deploy your projects:

`namespace`
: You can think of a namespace as a virtual cluster (within the cluster), where related resources can be grouped together. Bodywork creates and manages namespaces on your behalf.

`pod`
: A pod can be thought of as a collection of one or more containers, running on a single machine. Bodywork will create pods in which to run your batch jobs and services.

`deployment`
: A high-level resource for managing applications running in pods. It can ensure that a minimum number of pods are always operational (by restarting failed pods), manage rolling-updates and (where necessary) rollbacks. Bodywork uses deployments for managing your services.

`service`
: A service is a single constant IP address, through which clients can connect to services running in pods. Bodywork will create an internal cluster service for every service that you want to deploy. This enables any other client within the cluster to access it at this IP address, or via a domain name that uses the following convention,

    `http://SERVICE_NAME.NAMESPACE.svc.cluster.local`

`ingress`
: If you have enabled ingress for your cluster, then it will be running the [NGINX ingress-controller](#configuring-ingress). This will route requests from clients external to the cluster, to your services within the cluster, using the URL to locate the desired service. Bodywork can create and manages ingress rules for your services, so that they can be accessed by clients external to the cluster.

`secret`
: A mechanism for storing sensitive information in an encrypted format and securely distributing it to the pods that need it. Bodywork uses secrets to store any credentials that your projects may need access to - e.g., SHH keys for private Git repositories or API credentials.

### Accessing the Dashboard

The Kubernetes dashboard allows you to view all resources that have been deployed to your cluster. You can access it by issuing the following command,

```text
$ minikube dashboard
```

Which will open the dashboard in your default web browser. By default, it will only show you resources deployed to the `default` namespace. Use the namespace selector drop-down box at the top of the dashboard to switch to other namespaces - e.g., those created for your Bodywork deployments.

### The Kubectl Tool

Kubectl is the command-line tool that lets you control your Kubernetes cluster. Minikube comes packaged with a version of Kubectl that you can use via the Minikube CLI. For example, to get basic cluster information you would use,

```text
$ minikube kubectl cluster-info

Kubernetes master is running at https://192.168.64.5:8443
KubeDNS is running at https://192.168.64.5:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

If you get tired of prepending `minikube` to each Kubectl command, you can follow [our instructions](#installing-the-kubectl-tool) for installing the official version of Kubectl.

Some useful Kubectl commands to get started with:

#### Listing all Namespaces

```text
$ kubectl get ns
```

#### Deleting all resources within a Namespace

If your Bodywork deployment has come to an end,

```text
$ kubectl delete ns MY_NAMESPACE
```

#### Listing resources within a Namespace

To get a list of every resource within a namespace,

```text
$ kubectl -n MY_NAMESPACE get all
```

Particularly useful when used with the [watch tool](https://en.wikipedia.org/wiki/Watch_(command)) to monitor deployments from the command-line, as an alternative to using the Kubernetes dashboard.

To focus on a specific resource type, for example pods, you would instead use,

```text
$ kubectl -n MY_NAMESPACE get pods
```

#### Getting resource Information

```text
$ kubectl -n MY_NAMESPACE describe RESOURCE_TYPE RESOURCE_NAME
```

For example, to get high-level info for a pod named `foo-7dd8975899-57hj6`, you would use,

```text
$ kubectl -n MY_NAMESPACE describe pod foo-7dd8975899-57hj6
```

This will list all events associated with the pod, as well as a lot more information about how it has been configured by Bodywork.

#### Retrieving Pod Logs

To stream a pod's stdout and stderr to your local shell,

```text
$ kubectl -n MY_NAMESPACE logs MY_POD_NAME
```

Which can be useful for debugging.

#### Starting a shell within a running Container

For a running pod named `foo-7dd8975899-57hj6`, you can start a shell in this container using,

```text
kubectl exec -n MY_NAMESPACE foo-7dd8975899-57hj6 -it -- /bin/bash
```

This is also useful for debugging - for example, you can open a Python REPL or run `env` to list all environment variables (e.g. secrets) that have made it onto the container.

#### Starting a HTTP proxy server to the Kubernetes API

Issuing the following command,

```text
$ kubectl proxy --port 8001
```

Starts a proxy server that acts as a gateway to the Kubernetes API. Among other things, this allows you to access services on the cluster that are not exposed to the public internet. For example, with the proxy server operational, browsing to,

```http
http://localhost:8001/api/v1/namespaces/NAMESPACE/services/SERVICE_NAME/proxy/
```

Will take you to service `SERVICE_NAME`, in the namespace `NAMESPACE`.

### Working with remote Clusters

There are [many options](#managed-kubernetes-services) for creating managed Kubernetes clusters, in the cloud. Setting these up is beyond the scope of this introduction to Kubernetes. Once your remote cluster is operational, deploying to it is as easy as changing the cluster that Kubectl is targeting. To see what clusters Kubectl has been setup to use, run,

```text
$ kubectl config get-contexts

CURRENT   NAME                                         CLUSTER                            AUTHINFO                                     NAMESPACE
          aws_admin@my-cluster.eu-west-2.eksctl.io     my-cluster.eu-west-2.eksctl.io     aws_admin@my-cluster.eu-west-2.eksctl.io
*         minikube                                     minikube                           minikube                                     default
```

To switch from the `minikube` to the `aws_admin@my-cluster.eu-west-2.eksctl.io` I would run,

```text
$ kubectl config use-context aws_admin@my-cluster.eu-west-2.eksctl.io
```

And then Kubectl and Bodywork will automatically target my chosen cluster.

### Learning More

Familiarity with basic [Kubernetes concepts](https://kubernetes.io/docs/concepts/) and some exposure to the [Kubectl](#installing-the-kubectl-tool) command-line tool make life easier, but are not essential for using Bodywork. If you would like to learn a bit more about Kubernetes, then we recommend the first two introductory sections of Marko Lukša's excellent book [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action?query=kubernetes), or the introductory article we wrote on [Deploying Python ML Models with Flask, Docker and Kubernetes](https://alexioannides.com/2019/01/10/deploying-python-ml-models-with-flask-docker-and-kubernetes/).

### Getting Help

If you need help with Kubernetes, then please don't hesitate to post questions and ask for help on our [discussion board](https://github.com/bodywork-ml/bodywork-core/discussions). You are not alone and we'll do our best to get you up-and-running quickly.
