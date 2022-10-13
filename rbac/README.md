# RBAC
**rbac** konusuyla ilgili dosyalara buradan erişebilirsiniz.

$ kubectl config current-context

$ kubectl config use-context minikube

$ kubectl apply -f . 

    clusterrole.rbac.authorization.k8s.io/secret-reader created
    clusterrolebinding.rbac.authorization.k8s.io/read-secrets-global created
    role.rbac.authorization.k8s.io/pod-reader created
    rolebinding.rbac.authorization.k8s.io/read-pods created

