## menjalankan berkas manifest
```bash
kubectl apply -f pod.yaml
```

## akses pod-ip
```bash
kubectl describe pod
```

## akses didalam container
```bash
kubectl exec mypod -- curl http://<pod-ip>:80
```

## agar dapat diakses lewat host, maka butuh service (ip pod berubah2)
- dengan service, pod (group dari beberapa container) dapat diakses internal (sesama pod) atau eksternal (host & internet)
- service juga sebagai load balancing (mendistribusikan traffic diantara pod)
- dan service discovery (mekanisme dimana pod dapat menemukan pod lain secara dinamis)
- tipe service:
- ClusterIP (default): membuat service hanya dapat dijangkau dari dalam cluster (internal)
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
