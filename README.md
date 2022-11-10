# Jenkins in cluster

## Dependencies
* Helm
* Helm chart repository https://github.com/jenkinsci/helm-charts/
* Kubernetes cluster deployed with deckhouse

## Steps to deploy

* Checkout repository
* Connect to kubernetes api
* Update values.yaml with your parameters
* Replace all occurances of 'jenkins-ci' to <YOUR_RELEASE_NAME> in jenkins-custom.yml and values.yaml
* Run command in shell
```sh
# Add Namespace and cluster role
  kubectl apply -f jenkins-custom.yaml
# Pause
# Need to copy secrets for jenkins jcasc
  clientSecret=$(kubectl get secret dex-client-jenkins-ci -o jsonpath='{.data.clientSecret}' -n jenkins-ci | base64 -d)
  kubectl create secret generic dex-client-jenkins-copy --from-literal='clientsecret='$clientSecret -n jenkins-ci

# Create PVC for use persistance on host without cloud storage
apiVersion: deckhouse.io/v1alpha1
kind: LocalPathProvisioner
metadata:
  name: localpath-jenkins-ci
spec:
  nodeGroups:
  - worker
  path: "/mnt/wdc/jenkins-ci" # Change this path

# Add jenkins resources 
# Use '-f values-custom.yaml' for test purposes
  helm repo add jenkins https://charts.jenkins.io
  helm repo update
  helm upgrade -i --atomic --create-namespace -n jenkins-ci -f values.yaml jenkins-ci  jenkins/jenkins
```
# Add  user
# file user.yml
apiVersion: deckhouse.io/v1
kind: ClusterAuthorizationRule
metadata:
  name: admin
spec:
  # Kubernetes RBAC accounts list
  subjects:
  - kind: User
    name: admin@example.com
  # pre-defined access template
  accessLevel: none
  # allow user to do kubectl port-forward
  portForwarding: true
---
# section containing the parameters of the static user
# version of the Deckhouse API
apiVersion: deckhouse.io/v1
kind: User
metadata:
  name: admin
spec:
  # user e-mail
  email: admin@example.com
  # this is a hash of the password ae3wicsxw5, generated  now
  # generate your own or use it at your own risk (for testing purposes)
  # echo "ae3wicsxw5" | htpasswd -BinC 10 "" | cut -d: -f2
  # you might consider changing this
  password: '$2a$10$I8s50icWL219xlVffLBhX.6hnTMDgP0I8I38m5cGlFUD1OlDktcT6'

# Adduser to alowed groups
Add to CRD users.deckhouse.io 
apiVersion: deckhouse.io/v1alpha1
kind: User
spec:
  email: admin@example.com
  groups:
    - developers