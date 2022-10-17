# Network Policy

## Minikube

Cluster altinda her podun kendisine ait bir unic id si olmasi gerekiyor

Butun podlar birbirleri arasinda NAT olamadan haberlesebiliyorlar

Butun podlar birbirleri arasinda varsayilan olarak haberlesebiliyorlar

Podlar icerisinde bulunduklari worker nodun erisebildigi her yere de erisebilmektedirler

Yani bizim kubernetes altina koydugumuz butun podlar birbirleri ile anlasabiliyorlar. Biz bunlari namespace olarak ayirsak da haberlesebilirler. Fiziksel sunucu dahi olsalar
(worker1- worker2) haberlesemeye devam edebilirler. IP olarak podlar birbirleri ile LAYER 3-4 da konusabilirler

Ayrica eger dis dunya ile trafigi acarsak da ingress, nodeport, loadbalancer ile haberlesebilecekler

==>> Kisitlama olmadigi icin de bazen sorunlar yasanabilmektedir. Network olarak haberlesen podlari kisitlamak isteyebiliriz. (Frontend podu sadece Backend podu ile gorussun gibi)

Network policy bir kubernetes objesi ve bu sikintiyi kaldirma imkani vermektedir.

ORNEK ==> x podu sadece y namespacesinden erisilebilinsin / sadece 123 ip araligindan erisilebilinsin / xyz label erisebilsin gibi

 
* Minikube'u calico plug-in'iyle başlat

```
$ minikube start --cpus 4 --memory 6144 --cni=calico --container-runtime=docker --host-only-cidr=172.17.17.1/24

<!-- 
Sadece minikube start komutunu herhangi bir cni pluginini eklemeden uygularsak kubernetesin basit network altyapisi (kubelet dedigimiz) ile gelmektedir.   
Bunda network policy destegi yok. -->
```

* Podları ve namespaceleri oluştur

```
$ kubectl apply -f deploy.yaml
<!-- 
namespace/ns-a created
namespace/ns-b created
namespace/ns-c created
pod/poda created
pod/podb created
pod/podc created
pod/frontend created -->
```

```
$ kubectl get pods -A -o wide
<!-- 
NAMESPACE     NAME                                     READY   STATUS    RESTARTS      AGE    IP              NODE       NOMINATED NODE   READINESS GATES
kube-system   storage-provisioner                      1/1     Running   1 (93s ago)   2m5s   192.168.49.2    minikube   <none>           <none>
ns-a          frontend                                 1/1     Running   0             107s   10.244.120.69   minikube   <none>           <none>
ns-a          poda                                     1/1     Running   0             107s   10.244.120.68   minikube   <none>           <none>
ns-b          podb                                     1/1     Running   0             107s   10.244.120.67   minikube   <none>           <none>
ns-c          podc                                     1/1     Running   0             107s   10.244.120.70   minikube   <none>           <none> -->

```


* Network Policy deploy et
```
$ kubectl apply -f policy.yaml
```

* Poda'ya bağlan ve sadece 1.1.1.1'e 80 portuna gidebildiğini başka yere gidemediğini teyit et

```
$ kubectl exec -it -n ns-a poda -- bash
<!-- 
curl 1.1.1.1:80             ==> erisme saglanmaktadir
curl www.google.com:80      ==> erisme saglanamamaktadir -->

$ kubectl exec -it -n ns-b podb -- bash
<!-- 
Pod b uzerinden Pod a ya 80 portu uzerinden ulasilabilinmektedir. 
Pod c uzerinden Pod a ya 80 portu uzerinden trafik yoktur
-->





```

* PodB ve Frontend podlarından pod'a 80 portundan gidebildiğini teyit et. PodC'den ise gidemeyeceksin.
