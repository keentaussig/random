```
@cloudshell:~ (fibonacci-k8s-273103)$ gcloud config set project fibonacci-k8s-273103
Updated property [core/project].
@cloudshell:~ (fibonacci-k8s-273103)$ gcloud config set compute/zone us-west1-a
Updated property [compute/zone].
@cloudshell:~ (fibonacci-k8s-273103)$ gcloud container clusters get-credentials cluster-fibonacci-k8s
Fetching cluster endpoint and auth data.

kubectl create secret generic pgpassword --from-literal PGPASSWORD=
```
