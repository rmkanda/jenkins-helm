# Steps to setup Jenkins on GKE cluster - GCP

## Requirements

- GCP - GKE cluster
- Google Cloud SDK  
  `brew cask install google-cloud-sdk`
- Helm
  ```
  brew install kubernetes-helm
  helm init --client-only
  helm repo update
  ```

## Steps

- Create a two node GKE cluster

```
gcloud container clusters create test-cluster --num-nodes 2 --machine-type n1-standard-2 --scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"
```

- Create a Role

```
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
```

- Connect to the created cluster

```
gcloud container clusters get-credentials test-cluster

```

### Helm setup

- Create service account for tiller and initialize helm

```
kubectl create serviceaccount tiller --namespace kube-system

kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm init --service-account=tiller
```

- Verify helm installation

```
helm update
helm version
```

### Jenkins setup

- Install Jenkins using helm chart with custom values.yaml file

```
helm install -n ci-jenkins stable/jenkins -f values.yaml --wait
```

- Get the admin user's login passowrd for jenkins

```
printf $(kubectl get secret --namespace default ci-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

- Elevate the privileges of jenkins service account

```
kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:ci-jenkins

export POD_NAME=$(kubectl get pods -l "app.kubernetes.io/component=jenkins-master" -o jsonpath="{.items[0].metadata.name}")
```
