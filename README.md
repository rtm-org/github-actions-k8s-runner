# Kubernetes-Based CI/CD Pipeline with Self-Hosted GitHub Actions Runners
This project uses a workflow to deploy manifests to a Kubernetes cluster using Helm. The actions workflow is setup to connect to your cluster using the kube config and then it applies the manifest files to the cluster. The workflow is triggered on every push to the main branch or can be triggered manually using the workflow_dispatch event. Once the workflow runs, verify that the manifests were successfully applied by checking the state of your Kubernetes cluster.

### Requirements
- Have an existing Kubernetes cluster or set up a new one using a service like Minikube (local), GKE, EKS, or AKS (cloud)
- Have a repository that contains your application files

## Connect github actions to your kubernetes cluster
- I assume you already have a fork of this repo in your github account
- Create a secret in your forked repo called "KUBECONFIG" and add your kube configs

### Authenticating to the GitHub API
You can authenticate Actions Runner Controller (ARC) to the GitHub API by using a GitHub App or by using a personal access token (PAT). The primary benefit of using a GitHub App is increased API quota. For this project, we will use a GitHub App. Setting up the app will provide you with an "App ID" and a "Private Key, installing it on the repo will also provide you with an "App Installation ID". All these are to be added to this repo as secrets named "APP_ID", "APP_KEY", and "APP_INSTALLATION_ID" respectively.

You can find the app installation ID on the app installation page, which has the following URL format:
`https://github.com/organizations/ORGANIZATION/settings/installations/INSTALLATION_ID`

To set up a GitHub App, you can either create an app in your personal account or install it at the organization level if you intend to use the self-hosted runner across your organization repos.

### Steps to Create a GitHub App
1. Go to your GitHub settings and navigate to the "Developer settings" section.
2. Click on "GitHub Apps" and then click on "New GitHub App".
3. Give the app a name (the name must be globally unique)
4. Set the homepage URL to the URL of your self-hosted runner repository
5. Set the description to a brief description of your app
6. Under "Webhook" in permissions, click on the "active" checkmark to disable it. This step is necessary as our runner won't have direct access to the internet, hence, we would not be able to use webhooks.
7. Under "Permissions," click Repository permissions. Then use the dropdown menus to select the following access permissions.
    - Administration: Read and write
    - Metadata: Read-only
8. Under "Permissions," click Organization permissions. Then use the dropdown menus to select the following access permissions.
    - Self-hosted runners: Read and write
9. Click on "Create GitHub App" button
10. On the creating page, locate the "Private Key" section and create a new key. This will download a .pem file to your local machine
11. Next, on tha app menu, select the "Install App" page and install the app on the repository(ies) you want.

### Register GitHub App Info as Secret
Next, create a namespace in your kubernetes cluster. This namespace is where the "Action Runners" will be hosted.
```
kubectl create namespace ${NAMESPACE}
```
Then, create a secret in the namespace to hold the GitHub App info.
```
kubectl create secret generic runner-controller-manager \
--namespace=${NAMESPACE} \
--from-literal=github_app_id=${APP_ID} \
--from-literal=github_app_installation_id=${APP_INSTALLATION_ID} \
--from-literal=github_app_private_key="${APP_KEY}"
```

### Create Secrets and Variables
Next, create secret and variables in this repository setting that will be used when the github actions workflow in this repo is triggered.

The secret to create is:
- KUBECONFIG: This is the path to your kube config file. This is used to connect

The workflow also uses several variables:

- CONTROL_NAMESPACE: The namespace for the Actions Runner Controller.
- CONTROLLER_INSTALLATION_NAME: The name of the Actions Runner Controller installation.
- NAMESPACE: The namespace for the actions runners.
- RUNNERS_INSTALLATION_NAME: The name of the runners installation.

### Triggers

The workflow is triggered on:

- Push events to the main branch.
- Pull request events to the main branch.
- Manual workflow dispatch (allowing users to trigger the workflow manually).

### Job
The workflow has a single job named deploy that runs on an ubuntu-latest environment.

### Steps

The job consists of 6 steps:

- `Checkout`: Uses the actions/checkout@v4 action to check out the repository code.
- `Install Kubectl CLI`: Uses the azure/setup-kubectl@v4 action to install the latest version of the Kubernetes CLI (kubectl).
- `Set Kubeconfig Context`: Uses the azure/k8s-set-context@v4 action to set the Kubernetes context using a kubeconfig file stored as a secret in the repository (${secrets.KUBECONFIG}).
- `Install Helm`: Uses the azure/setup-helm@v4.2.0 action to install Helm version 3.13.3.
- `Install Actions Runner Controller`: Installs or upgrades the Actions Runner Controller using Helm. The script checks if the controller is already installed, and if so, upgrades it. Otherwise, it installs it. The installation uses a values file (controller/values.yaml) and a Helm chart from the ghcr.io/actions/actions-runner-controller-charts repository.
- `Install runners`: Installs or upgrades the runners using Helm, similar to the previous step. The script checks if the runners are already installed, and if so, upgrades them. Otherwise, it installs them. The installation uses a values file (runners/values.yaml) and a Helm chart from the ghcr.io/actions/actions-runner-controller-charts repository.