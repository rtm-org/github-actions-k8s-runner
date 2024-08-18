# github-runner-manifests

## Requirements
- Have an existing Kubernetes cluster or set up a new one using a service like Minikube (local), GKE, EKS, or AKS (cloud)
- Have a repository that contains your application files

## Connect github actions to your kubernetes cluster
- I assume you already have a fork of this repo in your github account
- Create a secret in your forked repo called `KUBECONFIG` and add your kube configs

The actions workflow is setup to connect to your cluster using the kube config and then it applies the manifest files to the cluster. The workflow is triggered on every push to the main branch or can be triggered manually using the workflow_dispatch event. Once the workflow runs, verify that the manifests were successfully applied by checking the state of your Kubernetes cluster.

