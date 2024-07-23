# Kubernetes’te Pod Yeniden Başlatma CronJob’u Oluşturma

Bu proje, belirli zaman aralıklarında pod’ları yeniden başlatmayı ve durumlarını kontrol etmeyi amaçlayan bir CronJob oluşturma sürecini ayrıntılı olarak açıklar. Her gece 00:00'da pod’ları yeniden başlatmayı hedefler.

## Deployment
İlk olarak, basit bir Deployment örneği oluşturalım.

```yaml
# cronjob-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: 'cronjob-deployment'
spec:
  selector:
    matchLabels:
      app: cronjob-deployment
  replicas: 3
  template:
    metadata:
      labels:
        app: cronjob-deployment
    spec:
      containers:
        - name: container
          image: nginx:latest
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
kubectl apply -f cronjob-deployment.yaml
```
## ServiceAccount Oluşturma
CronJob’un belirli kaynaklara erişim izni olması gerekir. İki farklı yöntemle ServiceAccount oluşturabilirsiniz.
```
kubectl create sa cronjob-deployment
```
## Role ve RoleBinding Oluşturma
Kubernetes kaynakları üzerinde hangi işlemlerin yapılabileceğini tanımlar.

### Role Oluşturma
Role, CronJob’un gerekli izinlere sahip olmasını sağlar.
```
kubectl create role cronjob-deployment-admin \
  --verb=get,list,watch,update,patch \
  --resource=deployments.apps
```
### RoleBinding Oluşturma
RoleBinding, Role ile ServiceAccount’u bağlar.
```
kubectl create rolebinding cronjob-deployment-admin-binding \
  --role=cronjob-deployment-admin \
  --serviceaccount=default:cronjob-deployment
```
## CronJob Oluşturma
```yaml
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: restart-cronjob
spec:
  schedule: "0 21 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: cronjob-deployment
          containers:
          - name: kubectl-container
            image: bitnami/kubectl:latest
            command:
            - /bin/sh
            - -c
            - |
              kubectl --namespace default rollout restart deployment/cronjob-deployment
              kubectl --namespace default rollout status deployment/cronjob-deployment
          restartPolicy: OnFailure
```
### CronJob Uygulama
```
kubectl apply -f cronjob.yaml
```
## CronJob Zamanlaması

CronJob’ların zamanlamasını `spec.schedule` altında ayarlayabilirsiniz.
```
schedule: '* * * * *'
```
Yıldızlar sırasıyla; Dakika, Saat, Ayın Günü, Ay ve Haftanın Günü’ne denk gelir. Daha fazla bilgi için <https://crontab.guru/> adresini ziyaret edebilirsiniz.

## Log Kontrolü
CronJob’un loglarını kontrol etmek için
```
kubectl logs <pod-adı>
```
