# Ingress
**ingress** konusuyla ilgili dosyalara buradan erişebilirsiniz.

uygulamalarimizin dis dunyadan erisilebilmesi icin nodeport ve loadbalancer objeleri 
bunlarin iki api layer da calistigi icin bazi sorunlarimizi cozemiyor (OSI layer 4)
 
$ minikube delete
$ minikube start --driver=hyperv
$ kubectl get nodes

ingress icin oncelikle ingress controler kurmamiz gerekiyor.
taninmislari nginx, traefic

https://kubernetes.github.io/ingress-nginx/deploy/

https://kubernetes.io/docs/concepts/services-networking/ingress/

$ minikube addons list
$ minikube addins enable ingress ==> ingress devreye alindi
$ kubectl get namespaces
$ kubectl get all -n ingress-nginx



***
Ingress objelerinin listelenmesi

```
$ kubectl get ingress
```
***
Ingress objelerinin silinmesi

```
$ kubectl delete ingress "statefulset_ingress"

Ör: kubectl delete ingress my-ingress
```
***


