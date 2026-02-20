## Pod
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
