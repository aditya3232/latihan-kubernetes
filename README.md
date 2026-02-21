## Pod
- pod merupakan sebuah group yang terdiri dari 1 atau beberapa container (umumnya 1 container)
- pod mempresentasikan sebuah unit aplikasi
- pod terdiri dari container (1 atau lebih), volume (penyimpanan), ip address (komunikasi jaringan)  
- pod merupakan unit terkecil dari kubernetes
- menjalankan berkas manifest
```bash
kubectl apply -f pod.yaml
```
- akses pod-ip
```bash
kubectl describe pod
```
- akses didalam container
```bash
kubectl exec mypod -- curl http://<pod-ip>:80
```

## Service
- agar dapat diakses lewat host, maka butuh service (ip pod berubah2)
- dengan service, pod (group dari beberapa container) dapat diakses internal (sesama pod) atau eksternal (host & internet)
- service juga sebagai load balancing (mendistribusikan traffic diantara pod)
- dan service discovery (mekanisme dimana pod dapat menemukan pod lain secara dinamis)
- tipe service:
- ClusterIP (default): membuat service hanya dapat dijangkau dari dalam cluster (internal). antara pod bisa akses lewat name service atau dns record
- NodePort: membuat service dapat diakses dari luar cluster menggunakan format <NodeIP>:<NodePort>
- LoadBalancer: membuat sebuah external load balancer di cloud provider (jika didukung) dan menetapkan public IP address yang tetap (fixed) ke Service
- ExternalName:  memetakan Service ke konten dari externalName field (contoh: foo.bar.example.com) dengan mengembalikan CNAME record beserta nilainya
```bash
kubectl apply -f service.yaml
kubectl get service
kubectl describe service webserver --> cek info Node Port
```
- cek nod IP
```bash
kubectl describe node | grep -i address -A 1
minikube ip
```

## Namespace
- merupakan kubernetes object untuk mengatur cluster menjadi sub cluster virtual
- biasanya untuk memisahkan sejumlah resource yang berbeda didalam cluster
- misal mengisolasi resource berdasarkan tim, proyek, env (dev,test,prod), aplikasi, dll
- object yang tidak bisa masuk namespace adalah node & peristance volume
- yang bisa pod, deployment, & service
```bash
kubectl get namespace
kubectl delete pod mypod && kubectl delete service webserver
kubectl apply -f pod.yaml -n webserver-ns && kubectl apply -f service.yaml -n webserver-ns --> buat pod & service pada namespace webserver-ns
```

## Deployment
- best practicenya membuat pod melalui deployment
- dengan deployment, kita dapat menentukan kondisi yg diinginkan dari pod
- misal mau 3 buah replica pod dengan Node.js image
- deployment ini diperuntukkan untuk aplikasi stateless (tidak meyimpan data secara lokal)
- deployment-ns.yaml --> untuk buat namespace
- data-tier.yaml --> service & deployment untuk redis database yang menyimpan nilai container
- app-tier.yaml --> berupa Node.js server yang menerima POST request untuk menambah nilai dari counter dan GET request untuk mendapatkan nilai terbaru dari counter
- support-tier.yaml --> berupa multi-container Pod (2 container di dalam satu Pod) yang berperan sebagai counter (kontinu membuat POST request ke server dengan nilai acak) dan poller (kontinu membuat GET request ke server dan mencetak nilai)
```bash
kubectl apply -f data-tier.yaml -f app-tier.yaml -f support-tier.yaml -n deployments
kubectl get deployment -n deployments
kubectl get pod -n deployments
kubectl logs support-tier-54564dbbc6-v5lv6 poller -f -n deployments
```

## HorizontalPodAutoscaler
- horizontal scaling --> penambahan atau pengurangan jumlah pod untuk aplikasi stateless
- manual scaling bukan solusi tepat production, kita harus pakai autoscaling
- HorizontalPodAutoscaler (HPA) merupakan object kubernetes untuk melakukan autoscaling pod
- biasanya HPA menyesuaikan berdasarkan cpu rata-rata, penggunaan memori rata-rata, atau metric khusus lainnya
- contohya menentukan jumlah min max replica yang diizinkan berdasarkan presentase cpu
- target cpu 60%; min replicas 2; max replicas 5
- proses HPA sangat bergantung pada metric yg dikumpulkan di cluster menggunakan object (metrics server)
```bash
kubectl apply -f metric-server.yaml --> deploy metric-server
kubectl get pods -n kube-system --> konfirmasi metric server sudah berjalan
kubectl top pods -n deployments --> melihat daftar penggunaan cpu dan memory untuk setiap pod pada suatu namespace
kubectl apply -f hpa.yaml -n deployments --> mendeklarasikan object HPA
kubectl get deployment -n deployments --watch --> memeriksa kondisi deployment secara realtime
kubectl describe hpa -n deployments --> periksa detail HPA; atau lebih ringkas kubectl get hpa -n deployments 
```

## Volume
- volume melakat pada pod
- karena melekat pada pod (bukan pada container), data akan tetap bertahan meski container restart
- digunakan untuk berbagi data diantara container dalam pod yg sama
- sifatnya volume ikut terhapus saat pod dihapus (karena scaling)
- umumnya volume digunakan untuk kebutuhan penyimpanan sementara

## Persistent Volume
- persistent volume tidak melakat pada pod
- dikelola secara terpisah oleh kubernetes
- meski pod dihapus, data akan tetap bertahan
- pod mesti membuat claim sebelum dapat menggunakan persistent volume (PVC)
- persistent volume claim (PVC), kita mendefinisikan jml penyimpanan yg dibutuhkan pod, jenis pv yg digunakan, & access mode yg dipakai
- ada 3 access mode: read-only many (volume dapat dipasang read-only oleh banyak node), 
- read-write once (volume dipasang sebagai read-write oleh satu node),
- read-write many (volume dapat dipasang sebagai read-write oleh banyak node)
```bash
kubectl apply -f stateful-ns.yaml --> namespace 
kubectl apply -f stateful-ns.yaml -->  deploy PVC
kubectl apply -f mysql-svc-deploy.yaml -n stateful-ns --> deploy service dan pod 
kubectl describe deployment mysql -n stateful-ns --> periksa deployment mysql
kubectl describe pvc mysql-pv-claim -n stateful-ns --> periksa pvc
kubectl describe pv mysql-pv-volume -n stateful-ns --> periksa mysql pv
kubectl run -it --rm --image=mysql:5.6 --restart=Never --namespace=stateful-ns mysql-client -- mysql -h mysql -ppassword --> jalankan mysql client untuk terhubung ke server
kubectl get pod -n stateful-ns --> lihat nama pod mysql
kubectl delete pod mysql-79c846684f-c5r8k -n stateful-ns --> delete pod mysql
```

## ConfigMap & Secret
- ConfigMap & Secret digunakan untuk memisahkan konfigurasi dari pod atau deployment manifest
- dengan begitu dapat memudahkan untuk mengelola dan mengubah konfigurasi (tidak perlu menyentuh deployment manifest lagi)
- ConfigMap & Secret menyimpan data sebagai key value
- untuk menggunakannya, pod harus mereferensikan ConfigMap & Secret terlebih dahulu

## ConfigMap
- ConfigMap digunakan untuk menyimpan informasi yg tidak sensitif
- seperti konfigurasi aplikasi & database host

## Secret
- dipakai untuk menyimpan informasi yg bersifat sensitif
- seperti password, token, api key
- dengan menggunakan secret, resiko pengeksposan data menjadi lebih minim
```bash
kubectl get all -n deployments --> cek object yg ada di namespace deployments [counter application di praktek Deployment]
kubectl apply -f data-tier-configmap.yaml -f data-tier.yaml -n deployments --> Nah, dengan begini, kita dapat mengonfigurasi Redis secara terpisah tanpa harus menyentuh Deployment manifest ini lagi ke depannya
kubectl exec -it -n deployments data-tier-d7747df69-f99tt -- /bin/bash --> masuk ke data-tier container
Setelah masuk, silakan konten dari berkas konfigurasi Redis. --> cat /etc/redis/redis.conf 

kubectl describe secret app-tier-secret -n deployments --> informasi detail dari secret
exec -n deployments app-tier-c77fb5fd-tbvtf -- env --> lihat semua env
```

## StatefulSet
- Deployment lebih cocok untuk stateless application
- karena pod yang dibuatnya selalu berubah-ubah
- mekanisme ini tidak cocok untuk pada stateful application
- StatefulSet ini adalah kubernetes object untuk mengelola stateful application
- Berbeda dengan Deployment, StatefulSet akan menyimpan identitas unik dari setiap Pod yang dikelolanya
- Ia menggunakan identitas yang sama setiap kali perlu meluncurkan ulang Pod
- StatefulSet menyediakan 2 identitas unik yang stabil untuk setiap Pod :
- Network Identity (identitas jaringan) yang memungkinkan kita untuk menetapkan DNS name yang sama ke suatu Pod, tak peduli berapa kali Pod tersebut di-restart
- Storage Identity (identitas penyimpanan) yang memungkinkan untuk mendapatkan Storage instance yang sama, tak peduli di Node mana Pod tersebut diluncurkan
- berbeda dengan Deployment, StatefulSet tidak membuat ReplicaSet. Controller StatefulSet langsung mengelola Pod satu per satu
- Deployment
   └── ReplicaSet
          └── Pod
- StatefulSet
    └── Pod (langsung dikelola controller StatefulSet)
- StatefulSet pakai konsep : Setiap replica akan dibuatkan PVC masing-masing
- mysql-0  → pakai PVC mysql-data-mysql-0
- mysql-1  → pakai PVC mysql-data-mysql-1
- jadi mysql kalau ada 2, biasanya menerapkan sinkronisasi data
- mysql-0 → primary
- mysql-1 → replica (replication)
- Sinkronisasi datanya dilakukan oleh MySQL replication
```bash
kubectl create namespace statefulset-ns
kubectl apply -f mysql-secret.yaml -n statefulset-ns --> manifest untuk menyimpan password mysql
kubectl apply -f mysql-pv.yaml -n statefulset-ns --> deploy pv
kubectl apply -f mysql-pvc.yaml -n statefulset-ns --> deploy pvc
kubectl apply -f mysql-service.yaml -n statefulset-ns --> headless service [biasa untuk persistent volume]
kubectl apply -f mysql-statefulset.yaml -n statefulset-ns --> deploy manifest StatefulSet
kubectl get statefulset,service,po,pv,pvc -n statefulset-ns --> periksa semua object Stateful App Mysql
kubectl get pod -o wide -n statefulset-ns --> cek pod yg dibuat oleh stateful set
```
