# Installing Bodywork

Bodywork is distributed as a Python package that exposes a Command Line Interface (CLI) for interacting with your Kubernetes cluster. The Bodywork CLI configures your cluster to run [pre-built Bodywork containers](https://hub.docker.com/repository/docker/bodyworkml/bodywork-core), that clone your project from its remote Git repository and then deploy it.

Bodywork can be downloaded and installed from PyPI with the following shell command,

```text
$ pip install bodywork
```

Or directly from the master branch of the [bodywork-core](https://github.com/bodywork-ml/bodywork-core) repository on GitHub using,

```text
$ pip install git+https://github.com/bodywork-ml/bodywork-core.git
```

Check that the installation has worked by running,

```text
$ bodywork
```

Which should display the following,

```text
Deploy machine learning projects developed in Python, to k8s.
--> see bodywork -h for help
```

## Required Python Version

Bodywork has been built and tested using Python 3.8. We recommend that Bodywork-compatible ML projects should also be developed and tested using Python 3.8, but in-practice your code is likely to work with other versions.

## Required Operating System

The Bodywork CLI has been developed for MacOS, Linux and Windows operating systems.
