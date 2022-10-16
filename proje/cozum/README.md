**1:** 5 node'lu "1 Master + 4 Worker Node" bir Kubernetes cluster kurun. 
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>
Minikube:

```
$ minikube start --nodes 5 -p multinode-demo

or 

$ minikube start --driver=hyperv --nodes 5 -p multinode-demo

$ minikube delete --all

$ kubectl get nodes
    <!-- NAME                 STATUS   ROLES           AGE     VERSION
    multinode-demo       Ready    control-plane   3m36s   v1.24.3
    multinode-demo-m02   Ready    <none>          2m53s   v1.24.3
    multinode-demo-m03   Ready    <none>          2m9s    v1.24.3
    multinode-demo-m04   Ready    <none>          74s     v1.24.3
    multinode-demo-m05   Ready    <none>          32s     v1.24.3 -->
```

AKS:

```
$ az login

$ az group create --name rg-k8sproje --location northeurope ==> resource group olusturmak icin

$ az aks create --name aks-k8sproje --resource-group rg-k8sproje --node-vm-size Standard_B2ms --node-count 4 --location northeurope ==> aks cluster create command

$ az aks get-credentials --name aks-k8sproje --resource-group rg-k8sproje ==> credential leri cekip config dosyasina merge etmesi icin
```
</details>

***
**2:** "test" ve "production" adlarında 2 namespace oluşturun.
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl create namespace test

$ kubectl create namespace production

$ kubectl get namespaces
    <!-- NAME              STATUS   AGE
    default           Active   4m40s
    kube-node-lease   Active   4m41s
    kube-public       Active   4m41s
    kube-system       Active   4m41s
    production        Active   6s
    test              Active   11s -->

    <!-- amacimiz test worknodelarimizi test namespacesinde production worknodelarimizi ise production namespacesinde olusturacagiz  -->
```
</details> 

***
**3:** "junior" isimli grubun "test" namespace'inde tüm kaynaklar üstünde "okuma, listeleme, yaratma..." gibi tüm haklara, "production" namespace'inde ise tüm kaynaklar üstünde sadece "okuma ve listeleme" haklarına sahip olacağı rol oluşturup bunları grupla (juniour) ilişkilendirin. Aynı şekilde "senior" isimli grubun "production" ve "test" namespace'lerindeki tüm kaynaklar üstünde "okuma, listeleme, yaratma..." gibi tüm haklara ve cluster üstündeki diğer kaynaklar üstünde ise sadece "okuma ve listeleme" haklarına sahip olacağı rol oluşturup bunları grupla ilişkilendirin. role / rolebinding -- cluster role / clusterrole binding

<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl apply -f ./yaml/jr-production-rb.yaml

$ kubectl apply -f ./yaml/jr-test-rb.yaml

$ kubectl apply -f ./yaml/sr-cluster-crb.yaml

$ kubectl apply -f ./yaml/sr-production-rb.yaml

$ kubectl apply -f ./yaml/sr-test-rb.yaml

$ kubectl get rolebindings.rbac.authorization.k8s.io -n test
    <!-- NAME         ROLE               AGE
         jr-test-rb   ClusterRole/edit   3m47s
         sr-test-rb   ClusterRole/edit   3m24s -->

$ kubectl get rolebinding.rbac.authorization.k8s.io -n production
    <!-- NAME               ROLE               AGE
         jr-production-rb   ClusterRole/view   5m20s
         sr-production-rb   ClusterRole/edit   5m4s -->

$ kubectl get clusterrolebinding.rbac.authorization.k8s.io | grep sr
    <!-- sr-cluster-crb                                         ClusterRole/view      -->
```
</details>

***
**4:** Sizin seçeceğiniz bir "ingress controller" kurun. (nginx, traefik, haproxy vb.) seceneklerinden istedigimizi kurabiliriz

Dis dunyadaki insanlarin cluster a ulasabilmeleri icin ingress controller kurmamiz gerekiyor

Layer 7 da application load balancing yapabilmemiz icin, self termination, path base routing yapabilmemiz icin 

<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

Nginx: https://kubernetes.github.io/ingress-nginx/deploy/

AKS:
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/cloud/deploy.yaml


```

Minikube:
```
$ kubectl get svc -A

<!-- ingress-ngnix dis dunyadan load balancer servicesi ile expose edildi -->

$ kubectl get all -n ingress-nginx

$ minikube addons list     

$ minikube addons enable ingress


<!-- bu komuttan itibaren minikube icerisinde ingress calisma sorunu cektigim icin hyper uzerinden yeni bir cluster ayaga kaldirdim -->

```
</details>

***
**5:** Cluster'da seçeceğiniz 3 worker node sadece "production" ortamında deploy edeceğiniz ve cluster tarafından oluşturulan podlar schedule edilebilsin. Bunun dışındaki podların bu worker node üstünde oluşturulmamasını sağlayın. 

olusturdugumuz 4 tane worker node un 3 unu production ortamina dedicate etmek istiyoruz. Test ortami icin olusturacagimiz podlar bu 3 worker node da calismasin, kalan bir tanesinde calistirilsin.

<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ node1=$(kubectl get no -o jsonpath="{.items[1].metadata.name}")

<!-- Farkli turde clusterlar ayaga kaldirmis olabiliriz. Bizim komutlari generic yazmamiz gerekiyor fakat farkli clusterlar olmasi sebebiyle de generic yazma imkanimiz yok
degiskene atma islemi yapmaktayiz -->

$ kubectl get no 
$ kubectl get no -o jsonpath="{.items[1].metadata.name}")
$ echo $node1

$ node2=$(kubectl get no -o jsonpath="{.items[2].metadata.name}")

$ node3=$(kubectl get no -o jsonpath="{.items[3].metadata.name}")

$ kubectl taint node $node1 tier=production:NoSchedule 

<!-- bu tainti tolere edemeyen hic bir worker node burada calistirlmayacak -->

$ kubectl taint node $node2 tier=production:NoSchedule

$ kubectl taint node $node3 tier=production:NoSchedule

$ kubectl label node $node1 tier=production

<!-- olusturulan nodelara labeller atamaktayiz -->

$ kubectl label node $node2 tier=production

$ kubectl label node $node3 tier=production
```

</details>

***
**6:** Wordpress uygulamasını hem "test" hem de "production" namespace'lerinde deploy edin. (wordpress:latest ve mysql:5.6 imajlarından oluşturulacak)

https://hub.docker.com/_/wordpress

https://hub.docker.com/_/mysql

- mysql "ClusterIp" tipi bir servisle cluster içinden erişilebilir olacak. 
- Her iki uygulamanın da uzun dönem verilerini "persistent volume"ler üzerinde saklanacak.

<!-- Mysql in tuttugu verileri dogal olarak uzun sure tutmak istiyoruz. Pod silindigi zaman veriler silinmesin.
docker run --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
YANI  ==> var/lib/mysql klasoru persistant volumenin baglanacagi konum
      ==> wordpress de ise /var/www/html -->

- Her iki uygulamanın da hiç bir hassas bilgisi "ör: şifre" uygulama ya da yaml dosyaları içerisinde tutulmayacak. 

<!-- Env variable olarak ya da secret olarak bu islemi yapmaliyiz. Bize tavsiye edilen yontem SECRET kullanimi -->

- Her iki uygulama da aynı worker node üstünde schedule edilecek.
- Her iki uygulama için de cpu ve memory kaynak kısıtları tanımlı olacak.  
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl create secret generic mysql-test-secret -n test --from-file=MYSQL_ROOT_PASSWORD=./yaml/mysql_root_password.txt --from-file=MYSQL_USER=./yaml/mysql_user.txt --from-file=MYSQL_PASSWORD=./yaml/mysql_password.txt --from-file=MYSQL_DATABASE=./yaml/mysql_database.txt

<!-- 
kubectl create secret generic ==> generic yani opac secret yaratalim
mysql-test-secret             ==> olusturulacak secret objesinin ismi
- n test                      ==> tesst namespace de olusturuyoruz
--from-file                   ==> klasor icerisinden secret bilgilerini okutuyoruz. command line da da gostermek istemiyoruz -->

$ kubectl create secret generic mysql-prod-secret -n production --from-file=MYSQL_ROOT_PASSWORD=./yaml/mysql_root_password.txt --from-file=MYSQL_USER=./yaml/mysql_user.txt --from-file=MYSQL_PASSWORD=./yaml/mysql_password.txt --from-file=MYSQL_DATABASE=./yaml/mysql_database.txt
 
$ kubectl apply -f ./yaml/wptest.yaml

<!-- Test ve production icin olusturulmus deployment filelari -->

$ kubectl get all -n test

$ kubectl get all -n test -w 

$ kubectl get deployments.app -n test

$ kubectl port-forward deployment/wp-deployment -n test 8080:80

<!-- Hernekadar ingress ile disdunyaya acacak olsakta test etmemiz icin portforward ile yayin yapmamiz gerekmektedir. 127.0.0.1:8080 ile ulasabiliriz -->

$ kubectl get pods -n test -o wide

<!-- Test nodelarinin tek bir worker node uzerinde calistigini gorebiliriz -->

$ kubectl apply -f ./yaml/wpprod.yaml

$ kubectl get all -n production

<!-- Test ve production icin olusturulmus deployment filelari / portforward i burada da uygulayabiliriz-->

```

</details>

***
**7:** "test" namespace'inde deploy edilen wordpress uygulaması "testblog.example.com", "production" namespace'inde deploy edilen wordpress uygulaması "companyblog.example.com" olarak ingress üstünden dış dünyaya expose edilecek.
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
<!-- 4. adimda ingress controler olarak ngnix kurmustuk. simdi de ingress objelerini olusturmamiz gerekiyor. -->

$ kubectl apply -f ./yaml/wpingress.yaml

$ kubectl get ingress -A -w 
<!-- adress kisminin bos oludugunu gorebiliriliz.Yukaridaki eslesmeyi yaparsak adres kisminin olustugunu gorecegiz. -->
 
```

</details>

***
**8:** "production" namespace'inde "ozgurozturknet/k8s:v1" imajından, 5 replikalı, update stratejisi olarak aynı anda 2 pod'un update edilebileceği bir deployment oluşturun. "/healthcheck" endpoint'ini sorgulayan "liveness probe" ve "/ready" endpoint'ini sorgulayan bir "readiness probe" tanımları da olsun. 
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl apply -f ./yaml/deployment.yaml

<!-- liveness probe ==> container icerisinde calisan uygulamaninkendisi ayakta olsa bile bazen crash edebiliyordu. Yapmasi gereken isi yapmama durumu olabiliyordu. Bunu tespit edebilecegimiz mekanizma livenessprobe

readiss probe  ==> container/pod ayaga kalkti ama servicenin arkaya alinmasina hazir mi degil mi. Bu sorgulamyi yapma imkani vermektedir. -->

$ kubectl get pods -w -n production ==> ready olmasi icin 20 saniye gecmesi lazim
```

</details>

***
**9:** Bir önceki görevde oluşturduğunuz deployment'i "loadbalancer" tipi bir servisle dış dünyadan erişilebilir hale getirin. 

Dis dunyaya expose edebilmemiz icin uc secenegimiz vardi
1- Nodeport
2- Cloud kullaniyorsak loadbalancer
3- Ingress

<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl expose deployment k8s-deployment --type=LoadBalancer -n production

$ kubectl get svc -n production -w 
```

</details>

***
**10:** Bu deployment'i önce 3 replikaya indirin. Ardından 10 replikaya çıkarın. Sonrasında bu deployment'i "ozgurozturknet/k8s:v2" imajıyla güncelleyin.
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl get pods -n production -w

$ kubectl scale deployment k8s-deployment --replicas=3 -n production

$ kubectl scale deployment k8s-deployment --replicas=10 -n production

$ kubectl set image deployment/k8s-deployment k8s=ozgurozturknet/k8s:v2 -n production

<!-- deployment file nin icerisinde bir den cok pod oldugu icin k8s= flagini sectik -->
```

</details>

***
**11:** "fluentd" uygulamasını bir "daemonset" olarak cluster'da deploy edin. 

Deamonset ==> bizim kac tane worker nodemuz varsa biz bunlarin uzerinde bir pod calistirmak istiyorsak ornekteki gibi log agentlarini storage provisioner ini calistirmamizi saglar

<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl apply -f ./yaml/daemonset.yaml

$ kubectl get daemonset.apps -w
```

</details>

***
**12:** 2 node'lu bir "mongodb" cluster'i "statefulset" olarak cluster'da deploy edin. "mongodb" cluster'ın çalıştığından emin olun. 

Statefulset ==> Kubernetes stateful uygulamalari icin cok uygun bir ortam degil. deployment, pod gibi objeler stateless icin daha uygun
Statefulset in deployment'tan temel farki sirayla olusturuluyor, sirayla siliniyor, hepsinin unic identitiy si oluyor.

<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl apply -f ./yaml/statefulset.yaml

$ kubectl get statefulsets.apps -w

<!-- ikiside ready hale geldikten sonra -->

$ kubectl exec -it mongostatefulset-0 -- bash

root@mongostatefulset-0:/# mongo

> rs.initiate({ _id: "MainRepSet", version: 1, 
members: [ 
 { _id: 0, host: "mongostatefulset-0.mongo-svc.default.svc.cluster.local:27017" }, 
 { _id: 1, host: "mongostatefulset-1.mongo-svc.default.svc.cluster.local:27017" } ]});

MainRepSet:PRIMARY> db.getSiblingDB("admin").createUser({
...       user : "mongoadmin",
...       pwd  : "P@ssw0rd!1",
...       roles: [ { role: "root", db: "admin" } ]
...  });

MainRepSet:PRIMARY> rs.status();

<!-- cluster icerisinden mongo ya baglanmak icin -->

$ kubectl get svc
```

</details>

***
**13:** Cluster'da tüm objeler üstünde "okuma ve listeleme" haklarına sahip bir "service account" oluşturun. Bu service account’u bağladığınız bir pod oluşturun ve bağlanarak "curl" ile cluster’daki tüm podları listeleyin. 
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl apply -f ./yaml/serviceaccount.yaml

$ kubectl get pod

$ kubectl exec -it pod-proje -- bash

$ hostname

bash-5.0# CERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

bash-5.0# TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

<!-- jwt.io den token in encode edilmis halini gorebiliriz -->

bash-5.0# curl --cacert $CERT https://kubernetes/api/v1/pods --header "Authorization:Bearer $TOKEN" | jq '.items[].metadata.name'

<!-- curl --cacert $CERT https://kubernetes/api/v1/pods ==> bazi uygulamalar icerisinde insecure option yok, 
bu kismi gozardi etmek icin --header optionunu gommek gerekiyor
curl --cacert $CERT https://kubernetes/api/v1/pods --header "Authorization:Bearer $TOKEN"
-->
```

</details>

***
**14:** Worker node'lardan bir tanesinin üzerindeki tüm podları tahliye edin ve ardından yeni pod schedule edilememesini sağlayın. 

Spesific bir pod uzerinde bir sey yapmak istiyorsak (guncelleme vs) tahliye islemi olmasi gerekiyor. Yani node uzerinde calisan podlar baska yerde calissin deriz.
Kubectl drain komutu ile yapabiliriz.
<details>
  <summary>Çözümü görmek için tıklayınız!</summary>

```
$ kubectl get pods -A -o wide   

$ nodedrain=$(kubectl get no -o jsonpath="{.items[3].metadata.name}")

<!-- 3 numarali worker node u nodedrain isimli bir degiskene atiyoruz -->

$ kubectl drain $nodedrain --ignore-daemonsets --delete-local-data

<!-- uzerindeki local verileri de sil (local veri anlaminda /emptydir) -->

$ kubectl cordon $nodedrain

bir makina uzerinde yeni pod schdule edilmesin ki ben bakim islemlerini yapayim dersen cordon komutu uygulanir (drain icerisinde cordon da barindirir)

tekrar devreye almak icin de;

$ kubectl uncordon $noderain

```

</details>

***
