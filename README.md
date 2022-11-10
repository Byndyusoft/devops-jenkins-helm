# Jenkins in cluster

## Dependencies
* Helm
* Helm chart repository https://github.com/jenkinsci/helm-charts/
* Kubernetes cluster deployed with deckhouse

## Steps to deploy

* Checkout repository
* Connect to kubernetes api
* Update values-custom.yaml with your parameters, which overwrites [default values.yaml]( https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/values.yaml)
* Replace all occurances of 'jenkins-ci' to <YOUR_RELEASE_NAME> in jenkins-custom.yml and values-custom.yaml
* Replace all occurances of 'k8s.local' to <YOUR_DOMAIN_NAME> in jenkins-custom.yml and values-custom.yaml
* Run commands in shell
```sh
# Add Namespace and cluster role
> kubectl apply -f jenkins-custom.yaml
# Pause
# Need to copy secrets for jenkins jcasc
> clientSecret=$(kubectl get secret dex-client-jenkins-ci -o jsonpath='{.data.clientSecret}' -n jenkins-ci | base64 -d)
> kubectl create secret generic dex-client-jenkins-copy --from-literal='clientsecret='$clientSecret -n jenkins-ci
```

* Create PVC for use persistance on host without cloud storage
```yaml
apiVersion: deckhouse.io/v1alpha1
kind: LocalPathProvisioner
metadata:
  name: localpath-jenkins-ci
spec:
  nodeGroups:
  - worker
  path: "/mnt/wdc/jenkins-ci" # Change this path
```
* Run shell command to deploy jenkins
```sh
# Add jenkins resources 
# Use '-f values-custom.yaml' for test purposes
  helm repo add jenkins https://charts.jenkins.io
  helm repo update
  helm upgrade -i --atomic --create-namespace -n jenkins-ci -f values-custom.yaml jenkins-ci jenkins/jenkins
```

* Add  user with user-authn module (Example: https://deckhouse.io/en/documentation/v1/modules/150-user-authn/usage.html#an-example-of-creating-a-static-user)
```sh
kubectl apply -f user.yml
```
> user.yml contents example
```yaml
## user.yml contents example
apiVersion: deckhouse.io/v1
kind: ClusterAuthorizationRule
metadata:
  name: username
spec:
  # Kubernetes RBAC accounts list
  subjects:
  - kind: User
    name: username@example.com
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
  name: username
spec:
  # user e-mail
  email: username@example.com
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
  email: username@example.com
  groups:
    - developers
```