# github-runner-manifests
## Requirements
- Have an existing Kubernetes cluster or set up a new one using a service like Minikube (local), GKE, EKS, or AKS (cloud)
- Have a repository that contains your application files

## Connect github actions to your kubernetes cluster
- I assume you already have a fork of this repo in your github account
- Create a secret in your forked repo called `KUBECONFIG` and add your kube configs
