# Kubernetes HPA (Horizontal Pod Autoscaler) Kurulumu

 HPA, belirli bir ölçüte dayalı olarak Pod sayısını otomatik olarak ölçeklendiren bir mekanizmadır.

## Adım 1: Namespace Oluşturma

Kubernetes’te yeni bir `hpa` adında bir namespace oluşturalım.

```sh
kubectl create namespace hpa
```
## Deployment Oluşturma
HPA örneğini çalıştırmak için basit bir deployment örneği yazalım.

### hpa-deployment.yaml Dosyasını Oluşturma

Bir metin düzenleyici ile `hpa-deployment.yaml` adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-deployment
spec:
  selector:
    matchLabels:
      app: hpa-deployment
  replicas: 1
  template:
    metadata:
      labels:
        app: hpa-deployment
    spec:
      containers:
      - name: hpa-deployment
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "32Mi"
            cpu: "16m"
          requests:
            memory: "8Mi"
            cpu: "4m"
        command: ["/bin/sh"]
        args:
          - "-c"
          - |
            while true; do
              dd if=/dev/urandom of=/dev/null bs=8M count=3 &
              dd if=/dev/urandom of=/dev/null bs=8M count=3 &
              sleep 60
            done
```
## Deployment'ı Uygulama
Metin düzenleme aracından kaydedip çıktıktan sonra aşağıdaki komutla deployment'ı uygulayın
```
kubectl apply -f hpa-deployment.yaml
```
## Deployment ve Pod Durumunu Kontrol Etme
Deployment ve Pod durumunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```
kubectl get deployments -n hpa
```
```
kubectl get pods -n hpa
```
## Horizontal Pod Autoscaler (HPA) Oluşturma

Bu çalışmamızdaki amaç, CPU kullanımı %75 veya memory kullanımı %70 olduğunda hpa-deployment’ın pod sayısını otomatik olarak 5'e kadar artırarak yükü dağıtmaktır. Bu değerlerin altına düştüğünde ise, hpa-deployment’ın pod sayısını otomatik olarak 1'e düşürerek podları azaltması beklenir.

### hpa.yaml Dosyasını Oluşturma

Bir metin düzenleyici ile `hpa.yaml` adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  namespace: hpa
  name: hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 1
        periodSeconds: 30
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Pods
        value: 1
        periodSeconds: 15
```
## HPA'yi Uygulama
Metin düzenleme aracından kaydedip çıktıktan sonra aşağıdaki komutla HPA'yi uygulayın.
```
kubectl apply -f hpa.yaml
```
## HPA Durumunu Kontrol Etme
HPA'nın durumunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```
kubectl get hpa -n hpa
```
