## Tutorial to run a basic pipeline using [tektoncd/pipeline](https://github.com/tektoncd/pipeline)

This tutorial will run a basic pipeline which builds an image from source code, push it to dockerhub and deploy to the cluster.

### Requirements

1. [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

2. [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

3. [DockerHub Account](https://hub.docker.com/)

4. [tektoncd-pipeline](https://github.com/tektoncd/pipeline/blob/master/docs/install.md)
    ```sh
    kubectl apply --filename https://storage.googleapis.com/tekton-releases/latest/release.yaml
    ```

#### Tutorial

Assuming your minikube cluster is up with tektoncd/pipeline installed

1. #### Clone the repository and move to directory

```sh
git clone https://github.com/piyush-garg/tekton-tutorial.git
```

```sh
cd tekton-tutorial
```

2. #### Update the docker secret file with your credentials

```sh
vi prep/dockersecret.yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-auth
  annotations:
    tekton.dev/docker-0: https://index.docker.io
type: kubernetes.io/basic-auth
data:
  username:
  password:
```

Set username and password
   * `echo -n dockerHubUsername | base64` e.g. `echo -n sthaha | base64`
   * `echo -n dockerHubPassword | base64 -w 0`

4. #### Apply Secret for Docker

    ```sh
    kubectl apply -f resource-descriptors/dockersecret.yaml
    ```

5. #### Create Service Account to use docker secret

    ```sh
    kubectl apply -f resource-descriptors/serviceaccount.yaml
    ```

6. #### Create Role and Role Bindings for service account to access pods, deployments and services api

    ```sh
    kubectl apply -f resource-descriptors/role.yaml
    ```

    ```sh
    kubectl apply -f resource-descriptors/rolebinding.yaml
    ```

3. #### Update the Pipeline Image Resources

    Edit the Pipeline Resource of Web Image and App image with your Docker Hub username.

    ```sh
    vi resources/pipelineResourceImageWeb.yaml
    ```

    Provide DockerHub username here

    ```yaml
          value: docker.io/DockerHubUsername/test-web
    ```

    Do the same for pipelineResourceImageApp.yaml


7. #### Create all the pipeline resources required for pipeline

    ```sh
    kubectl apply -f resource-descriptors/pipelineResourceImageWeb.yaml
    ```

    ```sh
    kubectl apply -f resource-descriptors/pipelineResourceImageApp.yaml
    ```

    ```sh
    kubectl apply -f resource-descriptors/pipelineResourceGit.yaml
    ```

8. #### Create task required for pipeline

    ```sh
    kubectl apply -f resource-descriptors/taskpush.yaml
    ```

    ```sh
    kubectl apply -f resource-descriptors/taskdeploy.yaml
    ```

9. #### Create pipeline and create first instance of pipeline using pipeline run

    ```sh
    kubectl apply -f resource-descriptors/pipeline.yaml
    ```

    ```sh
    kubectl apply -f resource-descriptors/pipelinerun.yaml
    ```

    This will build both the app and web dockerfile, push to github and apply the kubernetes yaml for both web and app as specified in github-repo https://github.com/piyush-garg/test-go

    It will create Deployment and Service for both web and app

10. #### Wait till both pods get in running state and access the service.

    ```sh
    kubectl get pods -w
    ```

    Wait till both web and app pod are in running state.

    Hit the service to access the application

    ```sh
    curl $(minikube service web --url)
    ```

### Thanks for trying out
