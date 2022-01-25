# Installing Bodywork

Bodywork is a command line tool, developed in Python. It is distributed as a Python package that exposes a Command Line Interface (CLI) for interacting with your Kubernetes cluster.

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
$ bodywork --help
```

Which should display the following,

```text
Usage: bodywork [OPTIONS] COMMAND [ARGS]...

Options:
  --install-completion [bash|zsh|fish|powershell|pwsh]
                                  Install completion for the specified shell.
  --show-completion [bash|zsh|fish|powershell|pwsh]
                                  Show completion for the specified shell, to
                                  copy it or customize the installation.
  --help                          Show this message and exit.

Commands:
  configure-cluster
  create
  delete
  get
  update
  validate
  version
```

## Required Python Version

Bodywork is developed and tested using Python 3.9. We recommend that your pipelines are also developed and tested using Python 3.9.

## Required operating System

The Bodywork CLI has been developed for MacOS, Linux and Windows operating systems.
