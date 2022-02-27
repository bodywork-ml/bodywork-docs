# CLI Reference

Bodywork CLI commands for managing ML pipelines on Kubernetes. Note that `bw` can be used as an alias for `bodywork` in any of the commands listed below.

---

## Get Version

```text
$ bodywork --version
```

Prints the Bodywork version.

---

## Validate Configuration File

The `bodywork.yaml` file can be checked for errors using,

```text
$ bodywork validate
```

Returns a list of configuration file errors.

**Options:**

`--check-files`
: Check that all paths specified as `executable_module_path` exist and can be reached by Bodywork, from the root directory where `bodywork.yaml` is located. This command assumes that `bodywork.yaml` is in the current working directory - if this is not the case, use the `--file` option to specify the path of `bodywork.yaml`.

---

## Configure Cluster

```text
$ bodywork configure-cluster
```

Prepare your Kubernetes cluster for use with Bodywork. Creates a dedicated Bodywork namespace, together with service accounts and roles for managing pipeline orchestration.

---

## Deploying Pipelines

Orchestrate stages on Kubernetes that run ML workloads and start ML services.

### Create Deployment

```text
$ bodywork create deployment GIT_REPO_URL
```

Deploy a pipeline to Kubernetes.

**Arguments:**

`GIT_REPO_URL`
: Location of the Git repository that contains the pipeline's codebase.

**Options:**

`--branch`
: Branch of Git repository to deploy, defaults to your repository's default branch (e.g., `master`).

`--retries`
: The number of times to retry the deployment, should any stage fail.

`--ssh-key-path`
: If the Git repo is private, then use the SSH key at this location to enable access. Automatically creates a secret for this purpose.

### Get Deployments

```text
$ bodywork get deployments PIPELINE_NAME SERVICE_NAME
```

Get information on pipelines and the service created by them. If no options are specified, then a list of all pipelines with active services will be returned.

**Arguments:**

`PIPELINE_NAME`
: (optional) List summary information on all active service stages associated with a pipeline. A pipeline's name is defined in `bodywork.yaml`.

`SERVICE_NAME`
: (optional) List detailed information for a single service deployed by a pipeline.

### Update Deployment

```text
$ bodywork update deployment GIT_REPO_URL
```

Redeploy a pipeline to Kubernetes.

**Arguments:**

`GIT_REPO_URL`
: Location of the Git repository that contains the pipeline's codebase.

**Options:**

`--branch`
: Branch of Git repository to deploy, defaults to your repository's default branch (e.g., `master`).

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

Pass secret data to the containers running pipeline stages - e.g., for authenticating with your cloud provider's API to access object storage. See [Managing Credentials and Other Secrets](user_guide.md#managing-credentials-and-other-secrets) and [Injecting Secrets into Stage Containers](user_guide.md#injecting-secrets-into-stage-containers).

### Create Secrets

```text
$ bodywork create secret SECRET_NAME \
    --group SECRETS_GROUP \
    --data SECRET_KEY_VALUE_PAIRS
```

**Arguments:**

`SECRET_NAME`
: The name to give this secret - e.g., `api-credentials`.

`SECRETS_GROUP`
: The group this secret belongs to - e.g., `dev-environment`

`SECRET_KEY_VALUE_PAIRS`
: Secret data items as keys and values - e.g., `USERNAME=api-username PASSWORD=api-password`.

Secret groups will be created automatically if they do not already exist.

### Get Secrets

```text
$ bodywork get secrets \
    --group SECRETS_GROUP \
    --name SECRET_NAME
```

If none of the options are specified, then `bodywork get secrets` will return a list of all secrets.

**Options:**

`--group`
: The secret group - e.g, `dev-environment` - to look in. If `--name` is not specified, then this will return a list of all secrets in the group.

`--name`
: Prints the full details for a single secret within a group.

### Update Secrets

```text
$ bodywork update secret SECRET_NAME \
    --group SECRETS_GROUP \
    --data SECRET_KEY_VALUE_PAIRS
```

**Arguments:**

`SECRET_NAME`
: The name of the secret to update - e.g., `api-credentials`.

`SECRETS_GROUP`
: The group this secret belongs to - e.g., `dev-environment`

`SECRET_KEY_VALUE_PAIRS`
: Updated secret data items - e.g., `USERNAME=new-api-username PASSWORD=-new-api-password REGION=api-region`.

### Delete Secrets

```text
$ bodywork delete secret \
    --group SECRET_GROUP \
    --name SECRET_NAME
```

**Arguments:**

`SECRETS_GROUP`
: The secret group - e.g., `dev-environment`. If no `--name` is specified, then all secrets within the group will be deleted.

**Options:**

`--name`
: If specified, will delete only the named secret in the group.

---

## Manage Cronjobs

Schedule pipeline runs using cronjobs.

### Create Cronjob

```text
$ bodywork create cronjob GIT_REPO_URL \
    --name NAME \
    --schedule CRON_SCHEDULE
```

**Arguments:**

`GIT_REPO_URL`
: Location of the Git repository that contains the pipeline's codebase.

`NAME`
: The cronjob's name - e.g., 'daily retraining'.

`CRON_SCHEDULE`
: Valid [cron expression](https://en.wikipedia.org/wiki/Cron#CRON_expression) - e.g. `0 * * * *` will run the pipeline every hour.

**Options:**

`--branch`
: Branch of Git repository to deploy, defaults to your repository's default branch (e.g., `master`).

`--retries`
: The number of times to retry executing the pipeline, should any stage fail (default is 3).

`--history-limit`
: The number of historical pipeline runs to retain logs for.

`--ssh-key-path`
: If the Git repo is private, then use the SSH key at this location to enable access. Automatically creates a secret for this purpose.

### Get Cronjobs

```text
$ bodywork get cronjob NAME
```

**Arguments:**

`NAME`
: (optional) The cronjob's name - e.g., 'daily retraining'. If omitted, then a list of all active cronjobs will be returned.

**Options:**

`--history`
: If specified together with `NAME`, then return a list of historical cronjob runs, for which logs have been retained. Use this to get the name assigned to a specific run, to use with the `--logs` option (see below).

`--logs`
: To return the logs of a historical pipeline run, specify this flag and use the specific name assigned to the historical run as the `NAME` argument. 

### Update Cronjob

```text
$ bodywork update cronjob GIT_REPO_URL \
    --name NAME \
    --schedule CRON_SCHEDULE
```

**Arguments:**

`GIT_REPO_URL`
: Location of the Git repository that contains the pipeline's codebase.

`NAME`
: The name of the cronjob to update - e.g., 'daily retraining'.

`CRON_SCHEDULE`
: Valid [cron expression](https://en.wikipedia.org/wiki/Cron#CRON_expression) - e.g. `0 * * * *` will run the pipeline every hour.

**Options:**

`--branch`
: Branch of Git repository to deploy, defaults to your repository's default branch (e.g., `master`).

`--retries`
: The number of times to retry executing the pipeline, should any stage fail.

`--history-limit`
: The number of historical pipeline runs to retain logs for.

### Delete Cronjob

```text
$ bodywork delete cronjob NAME
```

Will also delete all historical logs associated with this cronjob.

**Arguments:**

`NAME`
: The name of the cronjob to update - e.g., 'daily retraining'.

--- 

## Advanced Debugging

```text
$ bodywork debug SECONDS
```

Runs the Python `time.sleep` function for `SECONDS`. This is intended for use with the Bodywork image and kubectl - for deploying a container on which to open shell access for **advanced** debugging. For example, issuing the following command,

```text
$ kubectl create deployment DEBUG_DEPLOYMENT_NAME \
    -n YOUR_NAMESPACE \
    --image bodyworkml/bodywork-core:latest \
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
