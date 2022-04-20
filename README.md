# Jenkins in cluster

## Dependencies
* Helm
* Helm chart repository https://github.com/jenkinsci/helm-charts/
* Kubernetes cluster deployed with deckhouse

## Steps to deploy

* Checkout repository
* Connect to kubernetes api
* Update values.yaml with your parameters
* Replace all occurances of 'jenkins-test' to <YOUR_RELEASE_NAME> in jenkins-custom.yml and values.yaml
* Run command in shell
``` sh
  kubectl apply -f jenkins-custom.yml
# Need to copy secrets for jenkins jcasc
  clientSecret=$(kubectl get secret dex-client-jenkins-test -o jsonpath='{.data.clientSecret}' -n jenkins-test | base64 -d)
  kubectl create secret generic dex-client-jenkins-copy --from-literal='clientsecret='$clientSecret -n jenkins-test

  helm repo add jenkins https://charts.jenkins.io
  helm repo update
  helm upgrade -i --atomic --create-namespace -n <NAMESPACE_NAME> -f values.yaml <RELEASE_NAME>  jenkins/jenkins
```

