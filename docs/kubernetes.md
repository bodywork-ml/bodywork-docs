# Setup Kubernetes

Bodywork uses Kubernetes as a back-end for running ML pipelines. Although this means that you'll need access to a Kubernetes cluster to use Bodywork, it doesn't require you to be familiar with Kubernetes in any way.

This page contains everything you need to get up-and-running with a local test cluster, or to configure a cluster for production use.

## Quickstart

An easy way to get started with Kubernetes is with [Minikube](https://minikube.sigs.k8s.io/docs/). This will enable you to easily create and manage a local single-node Kubernetes cluster via the command line, and it comes bundled with everything you need to test Bodywork and run the [Quickstart Tutorials](quickstart_ml_pipeline.md).

If you are running on MacOS and using [Homebrew](https://brew.sh) for package management, then installing Minikube is as simple as running,

```text
$ brew install minikube
```

If you’re running on Windows or Linux, then follow the [recommended installation instructions](https://minikube.sigs.k8s.io/docs/start/).

### Creating your first Cluster

Once you have Minikube installed, start a cluster using the latest version of Kubernetes that Bodywork supports,

```text
$ minikube start --kubernetes-version=v1.22.6 --addons=ingress
```

High-level cluster information can be found using,

```text
$ minikube profile list

|----------|-----------|---------|--------------|------|----------|---------|-------|
| Profile  | VM Driver | Runtime |      IP      | Port | Version  | Status  | Nodes |
|----------|-----------|---------|--------------|------|----------|---------|-------|
| minikube | docker    | docker  | 192.168.64.5 | 8443 | v1.22.6  | Running |     1 |
|----------|-----------|---------|--------------|------|----------|---------|-------|
```

When you’re done, the cluster can be powered-down using.

```text
$ minikube stop
```

### Accessing Services

The most robust method for getting a URL to use as a gateway for accessing services deployed by Bodywork, is by opening a new terminal and running,

```text
$ minikube service ingress-nginx-controller --namespace ingress-nginx --url
```

An alternative option is to open a new terminal and instead run,

```text
$ minikube tunnel
```

This will also enable you to access services using `127.0.0.1` (localhost) as a gateway URL to your cluster, but you will be prompted to enter your user password after these services have been deployed, so you will need to keep an eye on the terminal output for this.

### Monitoring Resources

You can monitor Kubernetes resources and read logs via the [Kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/). You can get get a URL to the dashboard by running,

```text
$ minikube dashboard --url=true
```

## Supported Kubernetes Versions

Bodywork relies on the official [Kubernetes Python client](https://github.com/kubernetes-client/python), whose latest version (22.6.0) supports Kubernetes v1.22 and below. We recommend that you use a version of Kubernetes that is between v1.19 and v1.22. More information can be found [here](https://github.com/kubernetes-client/python#compatibility).

## Managed Kubernetes Services

When you are ready to deploy to a production system, then the easiest path is via a managed Kubernetes service from one of the following cloud infrastructure providers:

- [AWS](https://aws.amazon.com/eks)
- [Azure](https://docs.microsoft.com/en-us/azure/aks/)
- [GCP](https://cloud.google.com/kubernetes-engine/)
- [Digital Ocean](https://www.digitalocean.com/products/kubernetes/)
- [Scaleway](https://www.scaleway.com/en/kubernetes-kapsule/)

## Deploying Locally

If you want to test deployments locally, then you can run Kubernetes on your local machine with the help of one of the following tools:

- [Minikube](https://minikube.sigs.Kubernetes.io/docs/)
- [Kubernetes in Docker (KIND)](https://kind.sigs.k8s.io)
- [k3s](https://k3s.io)
- [Docker for Desktop](https://www.docker.com/products/docker-desktop)

We currently recommend Minikube, as it comes packaged with useful add-ons (e.g. ingress and the Kubernetes dashboard), that makes life easy for those starting with Kubernetes.

## Preparing a cluster for Bodywork

Bodywork is almost entirely reliant on Kubernetes resource primitives (e.g. Jobs, Deployments, Secrets, etc.), and won't install any 3rd party components onto your cluster. If you want to expose services (e.g. prediction APIs), to HTTP requests originating from outside the cluster, then you will need to install an ingress controller. Bodywork supports the [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/), which is an open-source project and maintained by the Kubernetes team.

### Installing NGINX

This will act like an [API Gateway](https://www.nginx.com/learn/api-gateway/) for your cluster, that will route external HTTP requests to internal services that have been created by your ML pipelines.

Installing the ingress controller can be achieved with a single command - e.g. for local a Minikube cluster you would use,

```text
$ minikube addons enable ingress
```

Or for EKS on AWS you would use,

```text
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/aws/deploy.yaml
```

The precise details for every potential Kubernetes deployment option are listed [here](https://kubernetes.github.io/ingress-nginx/deploy/).

Managed Kubernetes services will also provision an external [load balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing)) to manage the flow of traffic to the ingress controller (and hence the cluster). Note, this will have an associated cost that is additional to that of the Kubernetes cluster.

### Connecting to the Cluster

You can retrieve the public-facing IP address for your ingress controller with the following command,

```text
kubectl -n ingress-nginx get service ingress-nginx-controller
```

Make a note of the `EXTERNAL-IP` field for reaching services from outside the cluster. Services within the cluster can communicate using the cluster's internal network and DNS.

## Kubernetes 101

Here is a brief introduction to the most common types of Kubernetes resources, with a guide to how Bodywork uses them to deploy your projects:

`namespace`
: You can think of a namespace as a virtual cluster (within the cluster), where related resources can be grouped together. Bodywork creates and manages namespaces on your behalf, using one for each pipeline deployed.

`pod`
: A pod can be thought of as a collection of one or more containers, running on a single machine. Bodywork will create pods in which to run your batch jobs and services.

`deployment`
: A high-level resource for managing applications running in pods. It can ensure that a minimum number of pods are always operational (by restarting failed pods), manage rolling-updates and (where necessary) rollbacks. Bodywork uses deployments for managing services created by your ML pipelines.

`service`
: A service is a single constant IP address, through which clients can connect to services running in pods. Bodywork will create an internal cluster service for every service that you want to deploy. This enables any other client within the cluster to access it at this IP address, or via a domain name following the convention,

    `http://SERVICE_NAME.NAMESPACE.svc.cluster.local`

`ingress`
: If you have enabled ingress for your cluster, then it will be running the [NGINX ingress-controller](#preparing-a-cluster-for-bodywork). This will route requests from clients external to the cluster, to your services within the cluster, using the URL to locate the desired service. Bodywork can create and manage ingress rules for your services, so that they're accessible by clients external to the cluster.

`secret`
: A mechanism for storing sensitive information in an encrypted format and securely distributing it to the pods that need it. Bodywork uses secrets to store any credentials that your projects may need access to - e.g., SHH keys for private Git repositories or API credentials.

### Accessing the Dashboard

The [Kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) allows you to view all resources that have been deployed to your cluster and also provides some basic resource management functionality. Minikube users can access it by issuing the following command,

```text
$ minikube dashboard
```

Which will open the dashboard in your default web browser. By default, it will only show you resources deployed to the `default` namespace. Use the namespace selector drop-down box at the top of the dashboard to switch to other namespaces - e.g., those created for your Bodywork deployments.

### The Kubectl Tool

Kubectl is the command-line tool that lets you control your Kubernetes cluster. Minikube comes packaged with a version of Kubectl that you can use via the Minikube CLI. For example, to get basic cluster information you would use,

```text
$ minikube kubectl -- cluster-info

Kubernetes master is running at https://192.168.64.5:8443
KubeDNS is running at https://192.168.64.5:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

i.e., you add the Kubectl command you want to run, after `minikube kubectl --`. If you get tired of prepending this to each Kubectl command, you can [install the official version](https://kubernetes.io/docs/tasks/tools/).

Some useful Kubectl commands to get started with:

#### Listing all Namespaces

```text
$ kubectl get ns
```

#### Deleting all resources within a Namespace

If your Bodywork deployment has come to an end and you want tear it down manually,

```text
$ kubectl delete ns MY_NAMESPACE
```

#### Listing resources within a Namespace

To get a list of every resource within a namespace,

```text
$ kubectl -n MY_NAMESPACE get all
```

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

Starts a local proxy server that acts as a gateway to the Kubernetes API. Among other things, this allows you to access services on the cluster that are not exposed to the public internet. For example, with the proxy server operational, browsing to,

```http
http://localhost:8001/api/v1/namespaces/NAMESPACE/services/SERVICE_NAME/proxy/
```

Will take you to service `SERVICE_NAME`, in the namespace `NAMESPACE`.

### Monitoring Deployments

An effective way of monitoring a Bodywork deployment, is via the Kubernetes dashboard. Before you trigger a new deployment, open the dashboard and browse to `Workloads`, selecting the namespace created for your pipeline (this is the name of the project set in `bodywork.yaml`). Leave your browser open while you trigger the deployment using the Bodywork CLI. The dashboard will update automatically, showing you the resources that have been created as they are deployed.

An alternative to the Kubernetes dashboard, is to use the [watch](https://en.wikipedia.org/wiki/Watch_(command)) command from within a shell, to monitor the results of a Kubectl command. For example,

```text
$ watch --interval 1 kubectl -n default get all
```

Will display a list of all resources in the default namespace, updating with an interval of 1 second.

### Working with remote Clusters

There are [many options](#managed-kubernetes-services) for creating managed Kubernetes clusters in the cloud. Setting these up is beyond the scope of this introduction to Kubernetes. Once your remote cluster is operational, deploying to it is as easy as changing the cluster that Kubectl is targeting. To see what clusters Kubectl has been setup to use, run,

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

If you would like to learn a bit more about Kubernetes, then we recommend the first two introductory sections of Marko Lukša's excellent book [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action?query=kubernetes), or the introductory article we wrote on [Deploying Python ML Models with Flask, Docker and Kubernetes](https://alexioannides.com/2019/01/10/deploying-python-ml-models-with-flask-docker-and-kubernetes/).

### Getting Help

If you need help with Kubernetes, then please don't hesitate to post questions and ask for help on our [discussion board](https://github.com/bodywork-ml/bodywork-core/discussions). You are not alone and we'll do our best to get you up-and-running quickly.
