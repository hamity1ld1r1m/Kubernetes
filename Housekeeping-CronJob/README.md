# Kubernetes’te Hata Veren Pod’ları Housekeeping CronJob ile Temizleme

Kubernetes ortamında düzenli olarak hatalı podların temizlenmesini gerçekleştiren bir CronJob’dur. Bu görev, sistemin sağlığını ve performansını korumak için yapılır ve hatalı veya gereksiz podların otomatik olarak silinmesi işlemini içerir.

## Housekeeping CronJob Örnek Kullanım Durumu

- **Hatalı Podların Temizlenmesi**: Başarısız olan veya tamamlanan podları otomatik olarak silerek kaynakların boşa harcanmasını önler.

### 1. Namespace Oluşturma

Kubernetes kümesinde housekeeping adlı bir namespace oluştururuz:

```bash
kubectl create namespace housekeeping
```
Varsayılan namespace’i housekeeping olarak ayarlamak için kullanılır
```
kubectl config set-context --current --namespace=housekeeping
```
## Hatalı Deployment Örneği
Housekeeping CronJob örneğini çalıştırmak için basit bir hatalı deployment örneği yazalım.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: housekeeping
  name: 'cronjob-deployment-delete'
spec:
  selector:
    matchLabels:
      app: cronjob-deployment-delete
  replicas: 1
  template:
    metadata:
      labels:
        app: cronjob-deployment-delete
    spec:
      containers:
        - name: container
          image: nginx:latesttttt  # Yanlış image adı
          ports:
            - containerPort: 8080
              protocol: TCP
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```
### Deployment'ı Uygulama
```
kubectl apply -f cronjob-deployment-delete.yaml
```
## ServiceAccount Oluşturma
CronJob’un belirli kaynaklara erişim izni olması gerektiği için bu ServiceAccount tanımlanır.
```
kubectl create sa housekeeping-sa -n housekeeping
```
## ClusterRole ve ClusterRoleBinding Oluşturma
Kubernetes kaynakları üzerinde hangi işlemlerin yapılabileceğini tanımlar.
### ClusterRole Oluşturma
```
kubectl create clusterrole housekeeping-clusterrole \
  --verb=list \
  --resource=namespaces \
  --verb=list,delete \
  --resource=pods
```
### ClusterRoleBinding Oluşturma
```
kubectl create clusterrolebinding housekeeping-clusterrolebinding \
  --clusterrole=housekeeping-clusterrole \
  --serviceaccount=housekeeping:housekeeping-sa
```
## Housekeeping CronJob Oluşturma
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: housekeeping-cronjob-delete
  namespace: housekeeping
spec:
  schedule: '0 21 * * *'
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: housekeeping-sa
          containers:
          - name: housekeeping
            image: bitnami/kubectl:latest
            command:
            - /bin/sh
            - -c
            - |
              set -e
              echo "ImagePullBackOff, ErrImagePull, Pending, Error veya CrashLoopBackOff durumunda olan podları kontrol ediliyor"
              # Podları durumlarıyla listele ve ImagePullBackOff, ErrImagePull, Pending, Error veya CrashLoopBackOff durumunda olanları filtrele
              failed_pods=$(kubectl get pods -n housekeeping --no-headers | grep -E "ImagePullBackOff|ErrImagePull|Pending|CrashLoopBackOff|Error" | awk '{print $1}')
              if [ ! -z "$failed_pods" ]; then
                echo "Listelenen Podlar Siliniyor: $failed_pods"
                kubectl delete pod -n housekeeping $failed_pods
              else
                echo "ImagePullBackOff, ErrImagePull, Pending, CrashLoopBackOff veya Error durumunda olan pod bulunamadı"
              fi
          restartPolicy: OnFailure
```
### Housekeeping CronJob Uygulama
```
kubectl apply -f housekeeping-cronjob-delete.yaml
```
## CronJob Zamanlaması

CronJob’un zamanlamasını schedule altında ayarlıyoruz. Cron tabanlı zamanlama kullanılır.
```
schedule: '0 21 * * *'
```
Yıldızlar sırasıyla; Dakika, Saat, Ayın Günü, Ay ve Haftanın Günü’ne denk gelir. Daha fazla bilgi için <https://crontab.guru/> adresini ziyaret edebilirsiniz.

## Log Kontrolü
Housekeeping CronJob’un loglarını kontrol etmek için
```
kubectl logs <pod-adı>
```


