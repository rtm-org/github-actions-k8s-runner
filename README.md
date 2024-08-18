# github-runner-manifests

## Requirements
- Have an existing Kubernetes cluster or set up a new one using a service like Minikube (local), GKE, EKS, or AKS (cloud)
- Have a repository that contains your application files

## Connect github actions to your kubernetes cluster
- I assume you already have a fork of this repo in your github account
- Create a secret in your forked repo called "KUBECONFIG" and add your kube configs

The actions workflow is setup to connect to your cluster using the kube config and then it applies the manifest files to the cluster. The workflow is triggered on every push to the main branch or can be triggered manually using the workflow_dispatch event. Once the workflow runs, verify that the manifests were successfully applied by checking the state of your Kubernetes cluster.

## Authenticating to the GitHub API
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

