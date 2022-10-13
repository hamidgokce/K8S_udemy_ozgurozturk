# Authentication
**Authentication** konusuyla ilgili dosyalara buradan erişebilirsiniz.

Kubernetes normal kullanici hesaplari olusturabilme altyapisi sunmaz. Kimlik olusturma islemleri cluster disinda halledilir.
Her kubernetes clusterinde bir kok sertifika yetkilisi, root certificate authority altyapisi bulunur. x509 client sertifikalari da authenticate icin kullanilir
Obje olusturur gibi bir kullanici yaratamayiz

**Key ve CSR oluşturma**
Developer olarak bir private key ve certificate signing request dosyasi hazirlanip root yetkilisine gonderilirse root yetkilisi sertifika yaratabilir.
```
$ mkdir folder && cd folder
$ openssl genrsa -out ozgurozturk.key 2048 # private key olusturma

$ openssl req -new -key ozgurozturk.key -out ozgurozturk.csr -subj "//CN=ozgur@ozgurozturk.net//O=DevTeam"  # CSR bize sertifika saglayiciya belirli ozelliklerde 
digital sertifika sagla dememize imkan veren dosyalardir. Dosya olusturulup admine gonderilecek. 
Admin certificate authority ile bunu imzalayarak developer sertifikasini olusturup developera gonderecek
Developer da bu sertifika ile kubernetese baglanacak

openssl = kullandigimiz uygulama
reg     = csr olusturmak icin kullanilmakta
-new    = yeni oldugunu
-key    = bir onceki komutta olusan key
-out    = ozgurozturk.csr dosyasina yazmasini istiyoruz
-subj   = sertifika bilgilerini giriyoruz
CN      = kullanici adi olacak
O       = kullanicinin hangi gruplara uye olacagini

Bu asamadan sonra developer olarak gorev bitiyor. bu dosyalari admine gonderilecek
```

**CertificateSigningRequest oluşturma**
admin olarak yeni obje olusturulacak.
csr i kubernetese gonderecegiz. oda bize sertifika olusturacak

```
$ cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ozgurozturk
spec:
  groups:
  - system:authenticated
  request: $(cat ozgurozturk.csr | base64 | tr -d "\n")         # klasordeki csr i okutup base64 de code ediyoruz
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```
kubectl get csr
  condition kismi daha pending asamasinda / yani onaylanmamis


**CSR onaylama ve crt'yi alma**

```
$ kubectl get csr

$ kubectl certificate approve ozgurozturk

onaylama islemi sonrasi condition approved, issued olarak gorulecek

$ kubectl get csr ozgurozturk -o jsonpath='{.status.certificate}' 
$ kubectl get csr ozgurozturk -o jsonpath='{.status.certificate}' | base64 -d >> ozgurozturk.crt 

sertifika kismini almaktayiz ve bir dosyaya yaziyoruz

ls

```

artik certifika hazir ve developer a gonderilecek

**kubectl config ayarları**

developer olarak bu sertifikayi kubectl config dosyasina ekleyerek yeni bir context yaratilirsa kubernetes clustera bu context uzerinden baglanilabilir
```
config dosyasinda yeni kullanici olusturmak
$ kubectl config set-credentials ozgur@ozgurozturk.net --client-certificate=ozgurozturk.crt --client-key=ozgurozturk.key

olusturulan kullaniciyi cluster ile iliskilendirmeliyiz  
bunun icin de context olusturmaliyiz
$ kubectl config set-context ozgurozturk-context --cluster=minikube --user=ozgur@ozgurozturk.net

yeni olusturulan context e gecis.
ozgurozturk olarak cluster a baglanabiliriz.
$ kubectl config use-context ozgurozturk-context
```

su ana kadar bir kullanicinin kimlik dogrulamasi yapilarak kubernetese baglanmasi saglanildi.
her kullanici varsayilan olarak SIFIR yetki ile gelir ve listeleme dahil hicbir islem yapamaz

$ kubectl get pods
error: You must be logged in to the server (Unauthorized)

KULLANICIMIZ AUTHENTICATE OLDU FAKAT AUTHORIZATION EKSIK
