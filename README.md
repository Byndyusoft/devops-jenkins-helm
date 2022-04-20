# Jenkins in cluster

## Dependencies
* Helm
* Helm chart repository https://github.com/jenkinsci/helm-charts/
* Kubernetes cluster deployed with (deckhouse](https://deckhouse.io/en/)

## Steps to deploy

* Checkout repository
* Connect to kubernetes api
* Update values.yaml with your parameters
* Replace all occurances of 'jenkins-test' to <YOUR_RELEASE_NAME> in jenkins-custom.yaml and values.yaml
* Run command in shell
``` sh
  kubectl apply -f jenkins-custom.yaml
# Need to copy secrets for jenkins jcasc
  clientSecret=$(kubectl get secret dex-client-jenkins-test -o jsonpath='{.data.clientSecret}' -n jenkins-test | base64 -d)
  kubectl create secret generic dex-client-jenkins-copy --from-literal='clientsecret='$clientSecret -n jenkins-test

  helm repo add jenkins https://charts.jenkins.io
  helm repo update
  helm upgrade -i --atomic -n <NAMESPACE_NAME> -f values.yaml <RELEASE_NAME>  jenkins/jenkins
```
* Create users with deckhouse, which add acces to jenkins <br>
  [Docs for uses CRD](https://deckhouse.io/ru/documentation/v1/modules/150-user-authn/usage.html#%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80-%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D1%8F-%D1%81%D1%82%D0%B0%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B3%D0%BE-%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D1%8F) <br>

Example:
```yml
apiVersion: deckhouse.io/v1alpha1
kind: User
metadata:
  name: developer
spec:
  email: developer@byndyusoft.com
  groups:
    - developers
  # generating password example
  # echo "gh123uri8" | htpasswd -BinC 10 "" | cut -d: -f2
  password: $2y$10$Ro/Tz8x/X9v686nNeZMageaNNSLlXOCJUWTNC1foj0pYCcXaO8ee
```

## Jcasc configuration plugin
In repository https://github.com/jenkinsci/configuration-as-code-plugin
