# CLI Reference

Bodywork CLI commands for managing ML pipelines on Kubernetes.

---

## Get Version

```text
$ bodywork --version
```

Prints the Bodywork package version to stdout.

---

## Validate Configuration File

The `bodywork.yaml` file can be checked for errors using,

```text
$ bodywork validate
```

**Options:**

`--check-files`
: Check if all `executable_module_path` paths map to files that exist and can be reached by Bodywork, from the root directory where `bodywork.yaml` is located. This command assumes that `bodywork.yaml` is in the current working directory - if this is not the case, use the `--file` option to specify the path of `bodywork.yaml`. Validation errors are printed to stdout.

---

## Configure Cluster

```text
$ bodywork configure-cluster
```

Prepare your Kubernetes cluster for use with Bodywork. Creates a namespace, together with service accounts and roles for managing pipeline orchestration.

---

## Deploying Pipelines

Orchestrate stages on Kubernetes that run ML workloads and create ML services.

### Create Deployment

```text
$ bodywork create deployment REMOTE_GIT_REPO_URL REMOTE_GIT_REPO_BRANCH
```

Deploy a pipeline to Kubernetes.

**Arguments:**

`REMOTE_GIT_REPO_URL`
: Location of the Git repository that contains the pipeline's codebase.

`REMOTE_GIT_REPO_BRANCH`
: Branch of Git repository to deploy.

**Options:**

`--retries`
: The number of times to retry the deployment, should any stage fail.

### Get Deployments

```text
$ bodywork get deployments PIPELINE_NAME SERVICE_NAME
```

Get information on pipelines and the service stages created by them. If no options are specified, then a list of all pipelines with active service stages will be returned.

**Arguments:**

`PIPELINE_NAME`
: (optional) List summary information on all active service stages associated with a pipeline. A pipeline's name is defined in `bodywork.yaml`.

`SERVICE_NAME`
: (optional) List detailed information for a single service deployed by a pipeline. 

### Update Deployment

```text
$ bodywork update deployment REMOTE_GIT_REPO_URL REMOTE_GIT_REPO_BRANCH
```

Redeploy a pipeline to Kubernetes.

**Arguments:**

`REMOTE_GIT_REPO_URL`
: Location of the Git repository that contains the pipeline's codebase.

`REMOTE_GIT_REPO_BRANCH`
: Branch of Git repository to deploy.

**Options:**

`--retries`
: The number of times to retry the deployment, should any stage fail.

### Delete Deployment

```text
$ bodywork delete deployment PIPELINE_NAME
```

Delete all active services associated with a pipeline

**Arguments:**

`PIPELINE_NAME`
: The name of the pipelines that created the services. A pipeline's name is defined in `bodywork.yaml`.

---

## Manage Secrets

Secrets are used to pass credentials to the containers running pipeline stages - e.g., for authenticating with your cloud provider's API to access object storage. See [Managing Credentials and Other Secrets](user_guide.md#managing-credentials-and-other-secrets) and [Injecting Secrets into Stage Containers](user_guide.md#injecting-secrets-into-stage-containers).

### Create Secrets

```text
$ bodywork secret create \
    --namespace=YOUR_NAMESPACE \
    --name=SECRET_NAME \
    --data SECRET_KEY_1=secret-value-1 SECRET_KEY_2=secret-value-2
```

### Delete Secrets

```text
$ bodywork secret delete \
    --namespace=YOUR_NAMESPACE \
    --name=SECRET_NAME
```

### Get Secrets

```text
$ bodywork secret display \
    --namespace=YOUR_NAMESPACE
```

Will print all secrets in `YOUR_NAMESPACE` to stdout.

```text
$ bodywork secret display \
    --namespace=YOUR_NAMESPACE \
    --name=SECRET_NAME
```

Will only print `SECRET_NAME` to stdout.

---

## Manage Cronjobs

Workflows can be executed on a schedule using Bodywork cronjobs. Scheduled workflows will be managed by workflow-controller jobs that Bodywork starts automatically on your cluster.

### Get Cronjobs

```text
$ bodywork cronjob display \
    --namespace=YOUR_NAMESPACE
```

Will list all active cronjobs within `YOUR_NAMESPACE`.

### Create Cronjob

```text
$ bodywork cronjob create \
    --namespace=YOUR_NAMESPACE \
    --name=CRONJOB_NAME \
    --schedule=CRON_SCHEDULE \
    --git-repo-url=REMOTE_GIT_REPO_URL \
    --git-repo-branch=REMOTE_GIT_REPO_BRANCH \
    --retries=NUMBER_OF_TIMES_TO_RETRY_ON_FAILURE \
    --history-limit=MIN_NUMBER_OF_WORKFLOW_CONTROLLER_JOBS_TO_RETAIN
```

Will create a cronjob whose schedule must be a valid [cron expression](https://en.wikipedia.org/wiki/Cron#CRON_expression) - e.g. `0 * * * *` will run the workflow every hour. Use the `MIN_NUMBER_OF_WORKFLOW_CONTROLLER_JOBS_TO_RETAIN` argument to set the minimum number of historical workflow-controller jobs that are retained, at any given moment in time.

### Delete Cronjob

```text
$ bodywork cronjob delete \
    --namespace=YOUR_NAMESPACE \
    --name=CRONJOB_NAME
```

Will also delete all historic workflow-controller jobs associated with this cronjob.

### Get Cronjob History

```text
$ bodywork cronjob history \
    --namespace=YOUR_NAMESPACE \
    --name=CRONJOB_NAME
```

Display all workflow-controller jobs that were created by a cronjob.

### Get Cronjob Workflow Logs

```text
$ bodywork cronjob logs \
    --namespace=YOUR_NAMESPACE \
    --name=HISTORICAL_CRONJOB_WORKFLOW_EXECUTION_JOB_NAME
```

Stream the workflow logs from a historical workflow-controller job, to your terminal's standard output stream.

--- 

## Debug

```text
$ bodywork debug SECONDS
```

Runs the Python `time.sleep` function for `SECONDS`. This is intended for use with the Bodywork image and kubectl - for deploying a container on which to open shell access for advanced debugging. For example, issuing the following command,

```text
$ kubectl create deployment DEBUG_DEPLOYMENT_NAME \
    -n YOUR_NAMESPACE \
    --image=bodyworkml/bodywork-core:latest \
    -- bodywork debug SECONDS
```

Will deploy the Bodywork container and run the `bodywork debug SECONDS` command within it. While the container is sleeping, a shell on the container in this deployment can be started. To achieve this, first of all find the pod's name, using,

```text
$ kubectl get pods -n YOUR_NAMESPACE | grep DEBUG_DEPLOYMENT_NAME
```

And then [open a shell to the container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#getting-a-shell-to-a-container) within this pod using,

```text
$ kubectl exec DEBUG_DEPLOYMENT_POD_NAME -n YOUR_NAMESPACE -it -- /bin/bash
```

Once you're finished debugging, the deployment can be shut-down using,

```text
$ kubectl delete deployment DEBUG_DEPLOYMENT_NAME -n YOUR_NAMESPACE
```

---
