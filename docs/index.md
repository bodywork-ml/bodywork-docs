# Overview

<div align="center">
<img src="images/bodywork_logo.png" alt="Bodywork logo">
</div>

Bodywork deploys machine learning projects developed in Python, to [Kubernetes](https://en.wikipedia.org/wiki/Kubernetes). It helps you:

* [x] serve models as microservices
* [x] execute batch jobs
* [x] run reproducible pipelines

On demand, or on a schedule. It automates repetitive DevOps tasks and frees machine learning engineers to focus on what they do best - solving data problems with machine learning.

## Where does Bodywork Fit?

Bodywork is aimed at teams who want to deploy machine learning projects in containers. It will deliver your project's Python modules directly from your Git repository into Docker containers and manage their deployment to a Kubernetes cluster.

### What is it Replacing?

The process of building container images and deploying them to an orchestration platform is a complex engineering task. The diagram below shows the steps required to deploy a model-scoring service, together with the tools you could use to achieve this.

![Docker Kubernetes DevOps pipeline](images/ml_devops_flow.png)

Developing and maintaining these deployment pipelines is time-consuming. If there are multiple projects, each requiring re-training and re-deployment, then without the type of automation that Bodywork provides, management of these pipelines will quickly become a large burden.

## Where do I Install Bodywork?

Bodywork is distributed as a Python package that exposes a command line interface for configuring Kubernetes to run Bodywork deployments. It takes just one command to schedule a pipeline to run every evening,

![Bodywork CLI](images/bodywork_cronjob_create.png)

## What does Bodywork Do?

When triggered, Bodywork clones your project's Git repository, analyses the configuration provided in a `bodywork.yaml` file, and then manages the deployment of the projects' stages - creating new [Bodywork containers](https://hub.docker.com/repository/docker/bodyworkml/bodywork-core) to run the Python modules that define each one. At no point is there any need to build Docker images, push them to a container registry or to configure Kubernetes directly.

This process is shown below for a `train-and-serve` pipeline with two stages: train model (as a batch job), then serve the trained model (as a microservice with a REST API).

![Bodywork deployment to Kubernetes](images/ml_pipeline.png)

## What will I need to Do?

Divide your project into discrete stages and create an executable Python module for each one. Bundle these files together with a `bodywork.yaml` configuration file, into a Git repository and you're ready to go.

<div align="center">
<img src="images/project_structure_map.png"/ alt="Git project structure">
</div>

You do **not** need to tie yourself to new APIs - just add `bodywork.yaml` to your existing codebase and watch as Bodywork pulls each stage into its own container and deploys to Kubernetes.

## CI/CD for Machine Learning

Because Bodywork can run deployments on a schedule, every time cloning the latest version of your codebase in the target branch, this system naturally forms an end-to-end CI/CD platform for your machine learning project, as illustrated below.

![CICD for machine learning](images/cicd_with_bodywork.png)

This is the [GitOps](https://www.gitops.tech) pattern for cloud native continuous delivery - see [CI/CD for ML](cicd_for_ml.md) for the details.

## Key Features

`Continuous Deployment`
: Batch jobs, model-scoring services as well as complex ML pipelines, using pre-built [Bodywork containers](https://hub.docker.com/repository/docker/bodyworkml/bodywork-core) to orchestrate end-to-end machine learning workflows.

`Resilience`
: Bodywork handles automatic retires for batch jobs and automatic roll-backs for service deployments, without any downtime.

`Horizontal Scaling`
: Bodywork can back your service endpoints with as many container replicas as you need to handle your API traffic volumes.

`No APIs to Learn`
: Bodywork does not require you to re-write your machine learning projects to conform to our view of how your codebase should be engineered. All you need to do is provide executable Python modules for starting service applications and running batch jobs.

`Multi-Cloud`
: Bodywork deploys to Kubernetes clusters, which are available as managed services from all major cloud providers. Kubernetes is indifferent to where it is running, so changing cloud provider is as easy as pointing to a different cluster.

`Written in Python`
: The native language of machine learning and data science, so your team can have full visibility of what Bodywork is doing and how.

`Open Source`
: Bodywork is built and maintained by machine learning engineers, for machine learning engineers, who are committed to keeping it 100% open-source.

Bodywork brings DevOps automation to your machine learning projects and will form the basis of your [Machine Learning Operations (MLOps)](https://en.wikipedia.org/wiki/MLOps) platform. It will ensure that your projects are always trained with the latest data, the most recent models are always deployed and your machine learning systems remain highly-available.

## We want your Feedback

If Bodywork sounds like a useful tool, then please give a **GitHub Star ★** to [bodywork-core](https://github.com/bodywork-ml/bodywork-core).

## Before you get Started

Before you start exploring what Bodywork can do for you, you will need:

`Access to a Kubernetes Cluster`
: Either locally using [minikube](https://minikube.sigs.k8s.io/docs/), or as a managed service from a cloud provider, such as [EKS on AWS](https://aws.amazon.com/eks) or [AKS on Azure](https://azure.microsoft.com/en-us/services/kubernetes-service/).

`An Account with a Git Repo Hosting Service such as GitHub`
: Currently, we support public and private repositories hosted on [GitHub](https://github.com), [GitLab](https://about.gitlab.com), [Azure DevOps](https://azure.microsoft.com/en-gb/services/devops/) or [BitBucket](https://bitbucket.org/product/).

If you are new to Kubernetes, then please take a look at our guide to [Getting Started with Kubernetes](kubernetes.md#getting-started-with-kubernetes). Familiarity with basic [Kubernetes concepts](https://kubernetes.io/docs/concepts/) and some exposure to the [Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) command-line tool makes life easier, but are not essential.

If you would like to learn a bit more about Kubernetes, then we recommend the first two introductory sections of Marko Lukša's excellent book [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action?query=kubernetes), or the introductory article we wrote on [Deploying Python ML Models with Flask, Docker and Kubernetes](https://alexioannides.com/2019/01/10/deploying-python-ml-models-with-flask-docker-and-kubernetes/).

If you need help with any of this, then please don't hesitate to [contact us](contact.md) and we'll do our best to get you up-and-running. Bodywork is brought to you by [Bodywork Machine Learning](https://www.bodyworkml.com).
