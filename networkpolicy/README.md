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
```

* Podları ve namespaceleri oluştur

```
$ kubectl apply -f deploy.yaml
```

* Network Policy deploy et
```
$ kubectl apply -f policy.yaml
```

* Poda'ya bağlan ve sadece 1.1.1.1'e 80 portuna gidebildiğini başka yere gidemediğini teyit et

```
$ kubectl exec -it -n ns-a poda -- bash
```

* PodB ve Frontend podlarından pod'a 80 portundan gidebildiğini teyit et. PodC'den ise gidemeyeceksin.
