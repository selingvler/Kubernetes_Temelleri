# Kubernetes Temelleri
## İçindekiler
- [Kubernetes Mimarisi](#kubernetes-mimarisi)
- [Kubernetes 101](#kubernetes-101)
  - [Kubectl](#kubectl)
  - [Pod](#pod)
  - [Label, Selector, Annotation](#label,-selector,-annotation)
  - [Namespace, Deployment, ReplicaSet](#namespace,-deployment,-replicaset)

## Kubernetes Mimarisi
##### Master Node
- kube-apiserver: tüm komponent ve node bileşenlerinin direkt iletişim kurduğu tek komponenttır. authentication ve authorization
- etcd: cluster mevcut durumu ile ilgili bilgi, key-value
- kube-scheduler: pod'un çalışacağı en uygun worker node hangisidir diye bakar
- kube-controller-manager: mevcut durumla istenilen durum arasında fark olup olmadığına bakar, içerisinde bir çok controller barındırır (Node Controller, Job Controller, Service Account & Token Controller)
Endpoints Controller.
##### Worker Node
- container runtime: Docker -> Containerd
- kubelet: API Server aracılığıyla etcd’yi kontrol eder ve sched tarafından bulunduğu node üzerinde çalışması gereken podları yaratır. Containerd’ye haber gönderir ve belirlenen özelliklerde bir container çalışmasını sağlar.
- kube-proxy: Nodelar üstünde ağ kurallarını ve trafik akışını yönetir. Pod’larla olan iletişime izin verir, izler. DNS, GUI, TCP, UDP

_ Semantic versioning izler (x.y.z. -> x: major, y: minor, z: patch) ve her 4 ayda bir minor version çıkartır. Her ay patch version yayınlar. Bir kubernetes platformu en fazla 1 yıl kullanılır, 1 yıldan sonra güncellemek gerekmektedir.

## Kubernetes 101
### Kubectl
#### Kubectl config
- config dosyasını vscode'ta açmak için
  ```
  cd /Users/selinguler/.kube
  code .
  ```
- context -> Cluster ile user bilgilerini birleştirerek context bilgisini oluşturur. “Bu cluster’a bu user’la bağlanacağım.” anlamına geliyor.
- ```kubectl config```
- ```kubectl config get-contexts```Kubectl baktığı config dosyasındaki mevcut contextleri listeler.
- hangisinin yanında * varsa o current context'tir. ya da ```kubectl config current-context```
- ```kubectl config use-context docker-desktop```
#### Kubectl kullanımı
 ```
 kubectl <fiil> <object> ​
 
 # <fiil> = get, delete, edit, apply 
 # <object> = pod
 ```
- ```kubectl cluster-info``` Geçerli context'de tanımlı Kubernetes cluster hakkında bilgi edinme
- ```kubectl get "obje_tipi" Ör: kubectl get pods``` Varsayılan namespace'de bulunan objeleri listeleme
- ```kubectl get "obje_tipi" -n "namespace_adi" Ör: kubectl get pods -n kube-system```Varsayılan dışında başka bir namespace'de bulunan objeleri listeleme (-n "namespace_adi" ekiyle komutların belirtilen namespace üstünde çalıştırılması sağlanmaktadır)
- ```kubectl get "obje_ismi" --all-namespaces Ör: kubectl get pods --all-namespaces``` Tüm namespace'lerde bulunan objeleri listeleme
- ```-o opsiyonu``` output türünü belirlemek için
- ```kubectl get pods -A -o json | jq -r ".items[].spec.containers[].name"``` json formatındaki output'u formatladık
- ```kubectl explain "obje_tipi" Ör: kubectl explain pods``` Kubernetes obje tipleriyle ilgili detaylı bilgi edinmek
### Pod
- Pod’lar bir veya birden fazla container barındırabilir. Best Practice için her bir pod bir tek container barındırır.
- Her pod’un unique bir ID’si (uid) vardır ve unique bir IP’si vardır. Api-server, bu uid ve IP’yi etcd’ye kaydeder. Scheduler ise herhangi bir podun node ile ilişkisi kurulmadığını görürse, o podu çalıştırması için uygun bir worker node seçer ve bu bilgiyi pod tanımına ekler. Pod içerisinde çalışan kubelet servisi bu pod tanımını görür ve ilgili container’ı çalıştırır.
- Aynı pod içerisindeki containerlar aynı node üzerinde çalıştırılır ve bu containerlar localhost üzerinden haberleşir.
#### Pod oluşturma
- ```kubectl run "pod_ismi" --image="image_ismi" --restart=Never Ör: kubectl run firstpod --image=nginx --restart=Never``` imperative yöntem
- ```kubectl describe "obje_tipi" "obje_ismi" Ör: kubectl describe pods firstpod``` bir objenin ayrıntılı özellikleri (start time, container id, **events**-troubleshoot'ta işe yarayacak)
- ```kubectl logs firstpod | kubectl logs -f firstpod``` bir pod objesinin loglarını görüntüleme. (-f opsiyonu çıktıya yapışmanızı ve anlık olarak üretilen logları görmenizi sağlar)
- ```kubectl exec firstpod -- printenv | kubectl exec firstpod -c container1 --printenv``` Pod'da komut çalıştırma. (Eğer pod içerisinde birden fazla container varsa -c "container_ismi" opsiyonu ile komutun çalıştırılması istenilen container belirtilebilir)
- ```kubectl exec -it firstpod -- /bin/sh```Pod'a shell bağlantısı oluşturma.
- ```kubectl delete pods firstpod```Bir Kubernetes objesini silme
#### pod.yaml
- kind –> Hangi object türünü oluşturmak istiyorsak buraya yazarız. ÖR: pod
- apiVersion –> Oluşturmak istediğimiz object’in hangi API üzerinde ya da endpoint üzerinde sunulduğunu gösterir.
- metadata –> Object ile ilgili unique bilgileri tanımladığımız yerdir. ÖR: namespace, annotation vb.
- spec –> Oluşturmak istediğimiz object’in özelliklerini belirttiğimiz yerdir. Her object için gireceğimiz bilgiler farklıdır. Burada yazacağımız tanımları, dokümantasyondan bakabiliriz. Pod oluştururken bu pod'u oluşturan containerları mesela
- ```kubectl explain pods``` apiVersion'ı bulmak için (bu komutun çalışması için minikube'un başlatılmış olması gerekir)
- ```kubectl apply -f "dosya_yolu/dosya_ismi" Ör: kubectl apply -f ./pod1.yaml``` Json ya da Yaml formatında hazırlanmış bir konfigurasyon dosyası aracılığıyla yeni obje oluşturma
- ```kubectl delete -f "dosya_yolu/dosya_ismi" Ör: kubectl delete -f ./pod1.yaml```
- ```kubectl run "pod_ismi" --image="image_ismi" --port="port_numarası" --labels"anahtar:değer_eşlenikleri" --restart=Never Ör: kubectl run secondpod --image=nginx --port=80 --labels="app=front-end,team=developer" --restart=Never``` Label ve port konfigurasyonlarını da ekleyerek bir pod oluşturma (Burada daha sonra bir label eklemek istediğimde kubectl run ile hata verecek secondpod zaten çalıştığı için)
- ```kubectl edit "obje_tipi" "obje_ismi" Ör: kubectl edit pods firstpod ``` Cluster'da bulunan bir Kubernetes objesini varsayılan text editörü ile açarak özelliklerini güncelleme.
#### Pod yaşam döngüsü
- Pending –> Pod oluşturmak için bir YAML dosyası yazdığımızda, YAML dosyasında yazan configlerle varsayılanlar harmanlanır ve etcd’ye kaydolur.
- Creating –> kube-sched, etcd’yi sürekli izler ve herhangi bir node’a atanmamış pod görülürse devreye girer ve en uygun node’u seçer ve node bilgisini ekler. Eğer bu aşamada takılı kalıyorsa, uygun bir node bulunamadığı anlamına gelir.
  - etcd’yi sürekli izler ve bulunduğu node’a atanmış podları tespit eder. Buna göre containerları oluşturmak için image’leri download eder. Eğer image bulunamazsa veya repodan çekilemezse ImagePullBackOff durumuna geçer.
  - Eğer image doğru bir şekilde çekilir ve containerlar oluşmaya başlarsa Pod Running durumuna geçer.
- Restart Policy: always, never, on-failure
- Pod'un statüsü: succeeded, failed, completed
- Restart= always ise sürekli yeniden başlatılacağı için o pod sadece succeeded olur. **CrashLookBackOff** pod sürekli yeniden başlatılıyor, burada bir sıkıntı var
#### Pod yaşam döngüsü uygulama
- yaml dosyasında image belirtilirken bir typo yapılmışsa Pending -> Creating -> ImagePullBackOff -> ErrImagePull ```kubectl get pods -w``` status'u bu komut ile izledik
- container içerisinde çalışması istenen uygulamayı command ile biz belirliyoruz. status: Pending -> ContainerCreating -> Running -> Completed 
  ```
  metadata:
    name: succeededpod
  spec:
  restartPolicy: Never
  containers:
  - name: succeedcontainer
    image: ubuntu:latest
    command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 20']
  ```
- olmayan bir uygulamayı çalıştırmaya çalıştık. Status: Pending -> ContainerCreating -> Error
  ```
  metadata:
    name: failedpod
  spec:
    restartPolicy: Never
    containers:
    - name: failcontainer
      image: ubuntu:latest
      command: ['sh', '-c', 'abc']
  ```
- Status: Pending -> ContainerCreating -> Running -> Completed -> Running -> Completed -> CrashLookBackOff -> Running -> Completed -> CrashLookBackOff
  ```
  metadata:
    name: crashloopbackpod
  spec:
    restartPolicy: Always
    containers:
    - name: crashloopbackcontainer
      image: ubuntu:latest
      command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 20']
  ```
#### Çoklu container pod
- Neden bir container'a iki uygulama koyamıyoruz? izolasyon ve yatay scaling yapamamaktan dolayı
- Bağımlı çalışan uygulamalar için çoklu container pod önemli. İkinci bir wordpress pod'una ihtiyacım olduğunda yine ikinci bir fluentd pod'u deploy etmem gerekecek, sildiğimde de benzer şekilde. Ayrıca aynı lokal volume'a bağlanabilmek için aynı worker node üzerinde çalışmaları gerekir.
- Aynı pod içinde yaratılan containerlar aynı worker node üzerinde oluşturulur.
- Pod oluşturulunca iki container birden oluşturulur, silinirse de iki container birden silinir.
- Bu containerlar arasında network izolasyonu bulunmaz ve birbirlerine localhost üzerinden ulaşabilirler.
- Tek bir volume yaratılarak her iki container'a mount edilebilir.
#### Çoklu container pod uygulama
- yaml dosyası inceleme
  ```
  metadata:
    name: multicontainer
  spec:
    containers:
    - name: webcontainer        # ana container
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
      - name: sharedvolume
        mountPath: /usr/share/nginx/html        #/var/log klasörü ve bu klasör aynı yere bakacak
    - name: sidecarcontainer            
      image: busybox
      command: ["/bin/sh"]
      args: ["-c", "while true; do wget -O /var/log/index.html https://raw.githubusercontent.com/ozgurozturknet/hello-world/master/index.html; sleep 15; done"]
      volumeMounts:
      - name: sharedvolume
        mountPath: /var/log
    volumes:
    - name: sharedvolume
      emptyDir: {}
    ```
- ```kubectl exec -it <podName> -c <containerName> -- /bin/sh | kubectl exec -it multicontainer -c webcontainer -- /bin/sh```  Eğer multi-container’a sahip pod varsa ve bu containerlardan birine bağlanmak istersek
- ```kubectl exec -it multicontainer -c webcontainer -- /bin/sh | ifconfig``` inetaddr'lerin aynı olduğunu gördüm
- ```cd /usr/share/nginx/html ``` bu klasörde dosya oluşturduğumda sidecontainer'ın shellinden /var/logs2a girdiğim zaman oluşturduğum dosyayı burada göreceğim
- ```lubectl logs -f multicontainer -c sidecontainer```
- ```kubectl port-forward "obje_tipi"/"obje_ismi" "local_port":"hedef_port" | Ör: kubectl port-forward pod/multicontainer 8080:80``` kubectl'in çalıştırıldığı bilgisayar üzerinden cluster'da bulunan bir objeye tünel açılarak bağlantı kontrolü yapılması
#### Init container
1. Uygulama container’ı başlatılmadan önce Init Container ilk olarak çalışır.
2. Init Container yapması gerekenleri yapar ve kapanır.
3. Uygulama container’ı, Init Container kapandıktan sonra çalışmaya başlar. Init Container kapanmadan uygulama container’ı başlamaz.
- ```kubectl apply -f podinitcontainer.yaml```
- ```watch -n 2 kubectl describe pod initcontainerpod```
- ```kubectl logs -f initcontainerpod -c initcontainer```
- ```kubectl apply -f service1.yaml```
- initcontainer'ın işi aşağıdaki gibi olduğu için service1.yaml kurulduktan sonra status Init:0/1 -> Running oldu
  ```
  initContainers:
  - name: initcontainer
    image: busybox
    command: ['sh', '-c', "until nslookup myservice; do echo waiting for myservice; sleep 2; done"]
  ```
### Label, Selector, Annotation
#### Label ve selector
- label: key-value
- equality-based syntax
- ```kubectl get pods -l "app" --show-labels``` app anahtarına sahip pod'ları listele
- ```kubectl get pods -l "app=firstapp" --show-labels```
- ```kubectl get pods -l "app=firstapp, tier=front-end" --show-labels | kubectl get pods -l "app=firstapp, tier!=front-end" --show-labels```
- ```kubectl get pods -l "app, tier=front-end" --show-labels``` app anahtarı olsun ve tier = front-end olsun
- set based
- ```kubectl get pods -l "app in (firstapp)" --show-labels``` app'i firstapp olan
- ```kubectl get pods -l "app in (firstapp, secondapp)" --show-labels``` app'i firstapp ya da secondapp olan, equality-based'de ve vardı buna benzer yazınca bir app hem firstapp hem secondapp olamayacağı için boş döner
- ```kubectl get pods -l "app in (firstapp)" --show-labels``` app'i firstapp olmayanlar
- ```kubectl get pods -l "app, app notin (firstapp)" --show-labels``` app anahtarına sahip olsun ve bu değer firstapp olmasın (yukarıdakinden farklı sonuç döndürür çünkü pod11'in label'ı yok)(virgülü bu şekilde koyunca ve anlamına geliyor)
- ```kubectl get pods -l "!app" --show-labels``` app anahtarına sahip olmayan
- pod9 isimli pod'a bir label daha ekledim```kubectl label "obje_tipi" "obje_ismi" "anahtar=değer" Ör: kubectl label pods pod9 app=thirdapp```
- silme benzer şekilde ```kubectl label "obje_tipi" "obje_ismi" "anahtar-" Ör: kubectl label pods pod9 app-```
- atanmış etiketi güncelleme ```kubectl label --overwrite "obje_tipi" "obje_ismi" "anahtar=değer" Ör: kubectl label --overwrite pods pod9 team=team3```
- Bir namespace’deki tüm objelere toplu halde label eklemek```kubectl label "obje_tipi" --all "anahtar=değer" Ör: kubectl label pods --all foo=bar```
- bu pod'un hangi node'ta oluşturulacağının seçimini label'lar aracılığıyla yapabiliriz. hddtype: ssd label’ına sahip node’u seçmesini sağlayabiliriz. Böylece, pod ile node arasında label’lar aracılığıyla bir ilişki kurmuş oluruz.
  ```
  metadata:
    name: pod11
  spec:
    containers:
    - name: nginx
      image: nginx:latest
      ports:
      - containerPort: 80
    nodeSelector:
      hddtype: ssd
  ```
  pod11 ssd label’ına sahip node bulunmadığı için Pending'te takılı kaldı
  ```
  kubectl get nodes --show-labels
  kubectl label nodes minikube hddtype=ssd  #minikube adındaki node'a hddtype:ssd etiketi koyduk
  kubectl delete -f podlabel.yaml
  ```
#### Annotation
- metadata altına yazılır
- ```kubectl annotate pods annotationpod foo=bar``` ekleme
- ```kubectl annotate pods annotationpod foo- ``` silme
### Namespace, Deployment, ReplicaSet
#### Namespaces
- kube ile başlayan namespace'ler işleyişle ilgili
  ```
  kubectl get pods --namespace kube-system
  # Tüm namespace'lerdeki podları listelemek için:
  kubectl get pods --all-namespaces
  ```
- ```kubectl create namespace app1```
- yaml dosyasında metadata altında namespace: şeklinde tanımlanır
- terminal 
  ```
  kubectl apply -f podnamespace.yaml
  kubectl get pods -n development
  kubectl exec -it namespacepod -n development -- /bin/sh
  ```
- ```kubectl config set-context --current --namespace=development``` varsayılan namescape'i değiştirme
- ```kubectl delete namespaces <namespaceName>```
#### Deployment
- Deployment, bir veya birden fazla pod’u için bizim belirlediğimiz desired state’i sürekli current state‘e getirmeye çalışan bir object tipidir. Deployment’lar içerisindeki deployment-controller ile current state’i desired state’e getirmek için gerekli aksiyonları alır.
- ```kubectl create deployment "deployment_ismi" --image="image_ismi" --replicas="replika_adeti" Ör: kubectl create deployment firstdeployment --image=nginx:latest --replicas=2```
- ```kubectl delete deployment "deployment_ismi" Ör: kubectl delete deployment firstdeployment``` podlardan bir tanesini manuel olarak sildim. sistemde 1 pod kaldı. current state ve desired state eşit olmadığı için yeni bir pod yaratıldı
- ```kubectl set image deployment/"deployment_ismi" "container_ismi"="yeni_imaj" Ör: kubectl set image deployment/firstdeployment nginx=httpd:alpine``` önce yeni yarattı çalışınca eskisini sildi sonra yeni bir tane yarattı ve yine çalışınca eskisini sildi
- ```kubectl scale deployment "deployment_ismi" --replicas="istenilen_replika_adeti" Ör: kubectl scale deployment firstdeployment --replicas=5``` hemen 3 tane daha replika yarattı
- ```kubectl delete deployment "deployment_ismi" Ör: kubectl delete deployment firstdeployment```
- Deployment oluşturacak yaml dosyasında template kısmının altına yapıştır. (Indent’lere dikkat!) pod template içerisinden name alanını sil.
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: firstdeployment
    labels:
      team: development
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: frontend # **template içerisindeki pod'la eşleşmesi için kullanılacak label.** senin yöneteceiğin pod'lar frontend app'ine sahip olmalı, selector zorunlu
    template:	    # Oluşturulacak podların özelliklerini belirttiğimiz alan.
      metadata:
        labels:
          app: frontend # deployment ile eşleşen pod'un label'i.
      spec:
        containers:
        - name: nginx
          image: nginx:latest
          ports:
          - containerPort: 80 # dışarı açılacak port.
  ```
- ```kubectl apply -f deploymenttemplate.yaml```
#### ReplicaSet
- deployment ReplicaSet oluşturur oluşturulan ReplicaSet pod'u oluşturur. Güncelleme yaptığımda yine ReplicaSet oluşur fakat eskisi silinmez.
- ```kubectl rollout undo deployment "deployment_ismi" Ör: kubectl rollout undo deployment firstdeployment``` Deployment yapılan son değişikliğin geri alınması
#### Rollout ve Rollback
- recreate: bu deployment’ta bir değişiklik yaparsam, öncelikle tüm podları sil, sonrasında yenilerini oluştur
- RollingUpdate: değişiklik yaptığım zaman, hepsini silip; yenilerini oluşturma.” Bu strateji’de önemli 2 parametre vardır:
maxUnavailable –> En fazla burada yazılan sayı kadar pod’u sil. Bir güncellemeye başlandığı anda en fazla x kadar pod silinecek sayısı. (%20 de yazabiliriz.)
maxSurge –> Güncelleme geçiş sırasında sistemde toplamda kaç max aktif pod’un olması gerektiği sayıdır.
```
kubectl set image deployment rolldeployment nginx=httpd-alphine --record=true

# tüm değişiklik listesi getirilir.
kubectl rollout history deployment rolldeployment 

# nelerin değiştiğini spesifik olarak görmek için:
kubectl rollout history deployment rolldeployment --revision=2

# Bir önceki duruma geri dönmek için:
kubectl rollout undo deployment rolldeployment

# Spesifik bir revision'a geri dönmek için:
kubectl rollout undo deployment rolldeployment --to-revision=1

kubectl rollout status deployment rolldeployment -w

kubectl rollout pause deployment rolldeployment
kubectl rollout resume deployment rolldeployment
```
#### k8s ağ yapısı ve service
- Bir container farklı bir node içerisindeki başka bir container’la haberleşmesindeki sıkıntıyı çözmek için overlay network ya da eth0'lara ip adresi tanımlayabiliriz. Çözüm: **CNI network plugin**
