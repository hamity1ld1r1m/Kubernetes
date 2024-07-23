# Kubernetes Projesinde Nginx Gateway ve Backend Pod’ları ile Log Yönetimi ve Basit HTML Sayfası Oluşturma
Bu projede, Kubernetes üzerinde iki Nginx pod’u oluşturulacak: biri gateway olarak görev yapacak, diğeri ise backend olarak çalışacak. Backend pod’u, gelen her isteğe işletim sistemi bilgilerini döndüren basit bir HTML sayfası sunacak. Ayrıca, her iki pod’un logları bir Persistent Volume (PV) üzerinde saklanacak. Aşağıda bu projenin bileşenlerine ve nasıl yapılandırılacağına dair detaylı bilgiler verilmiştir.

## Persistent Volume (PV) ve Persistent Volume Claim (PVC) Oluşturma

Persistent Volume (PV), Kubernetes kümesine önceden sağlanan depolama birimidir. Bu depolama birimi, çeşitli altyapı sağlayıcılarında mevcut olabilir. PV, belirli bir depolama kapasitesine ve erişim modlarına sahip olabilir.

Persistent Volume Claim (PVC) ise, kullanıcıların belirli bir depolama alanı talep etmelerine olanak tanır. PVC, PV’nin depolama kaynaklarını talep eden bir istek nesnesidir. PVC, ihtiyaç duyulan depolama kapasitesini ve erişim modunu belirtir ve bu taleplerle eşleşen bir PV bulur.

### PV ve PVC Tanımları

`pv.yaml` dosyasını oluşturun

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-logs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/nginx-logs
```
Bu dosyayı uygulamak için
```
kubectl apply -f pv.yaml
```
`pvc.yaml` dosyasını oluşturun
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-logs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
Bu dosyayı uygulamak için
```
kubectl apply -f pvc.yaml
```
## ConfigMap Oluşturma
ConfigMap, pod’lara yapılandırma verileri sağlamak için kullanılan bir Kubernetes nesnesidir. Bu projede, backend pod’unun sunacağı basit HTML sayfası bir ConfigMap içerisinde saklanacak.

`configmap.yaml` dosyasını oluşturun
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-index-html
data:
  index.html: |
    <!DOCTYPE html>
    <html lang="tr">
    <head>
      <meta charset="UTF-8">
      <title>Backend</title>
      <style>
        body {
          font-family: Arial, sans-serif;
          background-color: #e9ecef;
          color: #333;
          margin: 0;
          padding: 0;
          display: flex;
          justify-content: center;
          align-items: center;
          height: 100vh;
        }
        .container {
          background: #ffffff;
          border-radius: 10px;
          padding: 25px;
          box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
          max-width: 700px;
          width: 90%;
          text-align: center;
        }
        h1 {
          color: #007bff;
          margin-bottom: 20px;
          font-size: 24px;
          font-weight: 600;
        }
        .info {
          font-size: 18px;
        }
        .info p {
          margin: 12px 0;
          padding: 10px;
          border: 1px solid #dee2e6;
          border-radius: 8px;
          background-color: #f8f9fa;
        }
        .info span {
          font-weight: bold;
          color: #495057;
        }
        .datetime {
          display: flex;
          justify-content: center;
          gap: 15px;
          margin-bottom: 20px;
        }
        @media (max-width: 600px) {
          .datetime {
            flex-direction: column;
          }
        }
      </style>
    </head>
    <body>
      <div class="container">
        <h1>Backend</h1>
        <div class="info">
          <div class="datetime">
            <p>Tarih: <span id="date"></span></p>
            <p>Saat: <span id="time"></span></p>
          </div>
          <p>İşletim Sistemi: <span id="os"></span></p>
          <p>İşlemci Çekirdek Sayısı: <span id="cpu"></span></p>
          <p>Toplam Bellek: <span id="memory"></span></p>
          <p>Ekran Çözünürlüğü: <span id="resolution"></span></p>
          <p>Tarayıcı Dili: <span id="language"></span></p>
          <p>Ekran Boyutu: <span id="screen-size"></span></p>
          <p>Tarayıcı: <span id="browser"></span></p>
        </div>
      </div>
      <script>
        // Tarih ve saat
        const now = new Date();
        document.getElementById('date').innerText = now.toLocaleDateString('tr-TR');
        document.getElementById('time').innerText = now.toLocaleTimeString('tr-TR');

        // İşletim sistemi bilgisi
        document.getElementById('os').innerText = navigator.platform || 'Bilgi yok';

        // İşlemci çekirdek sayısı
        document.getElementById('cpu').innerText = (navigator.hardwareConcurrency || 'Bilgi yok') + ' çekirdek';

        // Toplam bellek bilgisi
        document.getElementById('memory').innerText = (navigator.deviceMemory ? navigator.deviceMemory + ' GB' : 'Bilgi mevcut değil');

        // Ekran çözünürlüğü
        document.getElementById('resolution').innerText = `${window.screen.width} x ${window.screen.height}`;

        // Tarayıcı dili
        document.getElementById('language').innerText = navigator.language || 'Bilgi yok';

        // Ekran boyutu
        document.getElementById('screen-size').innerText = `${window.innerWidth} x ${window.innerHeight}`;

        // Tarayıcı adı ve versiyonu
        document.getElementById('browser').innerText = navigator.userAgent || 'Bilgi yok';
      </script>
    </body>
    </html>
```
Bu dosyayı uygulamak için
```
kubectl apply -f configmap.yaml
```
## Backend Deployment ve Service Oluşturma
Backend pod’u, Nginx image’ını kullanarak oluşturulacak ve yukarıda oluşturduğumuz ConfigMap içindeki HTML sayfasını sunacak. Loglar, PVC ile bağlanmış PV’ye yazılacak.

` backend.yaml` dosyasını oluşturun
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-backend
  template:
    metadata:
      labels:
        app: nginx-backend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: logs
        persistentVolumeClaim:
          claimName: nginx-logs-pvc
      - name: html
        configMap:
          name: nginx-index-html
          items:
          - key: index.html
            path: index.html
```
Bu dosyayı uygulamak için
```
kubectl apply -f backend.yaml
```
`svc-backend.yaml` dosyasını oluşturun
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-backend-service
spec:
  selector:
    app: nginx-backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30002
  type: NodePort
```
Bu dosyayı uygulamak için
```
kubectl apply -f svc-backend.yaml
```

## Gateway Deployment ve Service Oluşturma

Gateway pod’u, Nginx image’ını kullanarak oluşturulacak ve backend pod’una gelen istekleri yönlendirecek. Gateway pod’unun logları da PVC ile bağlanmış PV’ye yazılacak.

`gateway.yaml` dosyasını oluşturun
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-gateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-gateway
  template:
    metadata:
      labels:
        app: nginx-gateway
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
        - name: config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: logs
        persistentVolumeClaim:
          claimName: nginx-logs-pvc
      - name: config
        configMap:
          name: nginx-config
```
Bu dosyayı uygulamak için
```
kubectl apply -f gateway.yaml
```
`svc-gateway.yaml` dosyasını oluşturun
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-gateway-service
spec:
  selector:
    app: nginx-gateway
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30001
  type: NodePort
```
Bu dosyayı uygulamak için
```
kubectl apply -f svc-gateway.yaml
```
## Backend Pod İçindeki HTML Dosyasını Kontrol Etme

Backend pod’unun HTML dosyasını kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```
kubectl exec -it <backend-pod-name> -- cat /usr/share/nginx/html/index.html
```
Burada `<backend-pod-name>`, backend pod'unuzun adıdır. Bu komut, backend pod'unda bulunan HTML dosyasının içeriğini görüntülemenizi sağlar.


## Backend Pod İçindeki Log Dosyalarını Kontrol Etme
Backend pod’unda log dosyalarını kontrol etmek için aşağıdaki komutları kullanabilirsiniz.
### Log dosyalarını listeleme
```
kubectl exec -it <backend-pod-name> -- ls /var/log/nginx
```
### access.log dosyasını görüntüleme
```
kubectl exec -it <backend-pod-name> -- cat /var/log/nginx/access.log
```
### error.log dosyasını görüntüleme
```
kubectl exec -it <backend-pod-name> -- cat /var/log/nginx/error.log
```
Bu komutlar, backend pod'unuzdaki log dosyalarının içeriğini kontrol etmenizi sağlar.

## Gateway Pod’unda HTML Dosyasını Kontrol Etme
Gateway pod’unun HTML dosyasını kontrol etmek için aşağıdaki komutu kullanabilirsiniz
```
kubectl exec -it <gateway-pod-name> -- cat /usr/share/nginx/html/index.html
```
Burada `<gateway-pod-name>`, gateway pod'unuzun adıdır. Bu komut, gateway pod'unda bulunan HTML dosyasının içeriğini görüntülemenizi sağlar.

## Gateway Pod’unda Log Dosyalarını Kontrol Etme

### Log dosyalarını listeleme
```
kubectl exec -it <gateway-pod-name> -- ls /var/log/nginx
```
### access.log dosyasını görüntüleme
```
kubectl exec -it <gateway-pod-name> -- cat /var/log/nginx/access.log
```
### error.log dosyasını görüntüleme
```
kubectl exec -it <gateway-pod-name> -- cat /var/log/nginx/error.log
```
Bu komutlar, gateway pod'unuzdaki log dosyalarının içeriğini kontrol etmenizi sağlar.

## İnternet Sağlayıcı Üzerinden Erişim
İnternet sağlayıcıdan erişimi kontrol etmeden önce master node’un IP adresini doğrulamanız gerekir. Bunun için aşağıdaki adımları izleyebilirsiniz
```
kubectl get nodes -o wide
```
Bu komut, Kubernetes node'larınızın IP adreslerini gösterecektir.

## Backend Pod’una Erişim
Web tarayıcınıza gidin ve aşağıdaki URL’yi girin
```
http://<Kubernetes_Node_IP>:30002
```
Burada `<Kubernetes_Node_IP>`, Kubernetes node'unuzun IP adresidir. Bu, backend pod'undaki HTML sayfasını görüntülemenizi sağlar.

## Gateway Pod’una Erişim
Web tarayıcınıza gidin ve aşağıdaki URL’yi girin.
```
http://<Kubernetes_Node_IP>:30001
```
Burada `<Kubernetes_Node_IP>`, Kubernetes node'unuzun IP adresidir. Bu, gateway pod'una erişim sağlar.
NodePort servisi kullanarak, belirttiğiniz node portları üzerinden pod’lara dışarıdan erişim sağlamış oluyorsunuz.
## Sonuç
Bu adımları tamamladığınızda, Kubernetes cluster’ınızda iki Nginx pod’u çalışır durumda olacaktır: biri gateway, diğeri backend olarak görev yapacak. Backend pod’u, işletim sistemi bilgilerini döndüren bir HTML sayfası sunacak, ve her iki pod’un logları belirttiğiniz PV üzerinde saklanacaktır.
