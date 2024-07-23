# Kubernetes Dashboard GUI Kurulumu

Kubernetes Dashboard, Kubernetes kümenizi yönetmenize ve izlemenize olanak tanıyan web tabanlı bir kullanıcı arayüzüdür. Bu kılavuzda, Kubernetes Dashboard’un nasıl kurulacağını ve yapılandırılacağını adım adım ele alacağız.

## Kubernetes Dashboard’u Yükleme

Kubernetes Dashboard’un en güncel sürümünü yüklemek için, Kubernetes’in resmi GitHub deposundan `recommended.yaml` dosyasını indirmeniz gerekiyor. Bu dosya, Dashboard ve gerekli tüm kaynakları içeren bir yapılandırma dosyasıdır.

```sh
curl -LO https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml
```
Bu komut, `recommended.yaml` dosyasını yerel makinenize indirecektir.

### recommended.yaml Dosyasını Uygulama
İndirdiğiniz yapılandırma dosyasını uygulayarak Dashboard’un Kubernetes kümenize kurulmasını sağlayın.

```
kubectl apply -f recommended.yaml
```
## Hizmet Hesabı ve ClusterRoleBinding Oluşturma

Dashboard’a erişmek için yetkilendirme gereklidir. Bunun için bir `ServiceAccount` ve `ClusterRoleBinding` oluşturmanız gerekir.

### admin-rolebinding.yaml Dosyasını Oluşturma
Bir metin düzenleyici ile `admin-rolebinding.yaml` adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
Bu dosya, Dashboard’a admin yetkileri ile erişim sağlamak için gerekli izinleri içerir.

### admin-rolebinding.yaml Dosyasını Uygulama

```
kubectl apply -f admin-rolebinding.yaml
```
## Service Account ve Token Oluşturma

Dashboard’a erişim sağlamak için genellikle bir token kullanılır. `recommended.yaml` dosyasında bu yapılandırmalar varsa, bu adımı atlayabilirsiniz. Aksi takdirde, aşağıdaki komutla bir token oluşturabilirsiniz.

```
kubectl -n kubernetes-dashboard create token admin-user
```
Bu komut, `admin-user` isimli ServiceAccount için bir token oluşturur.

## Dashboard Servisini Erişilebilir Hale Getirme
Kubernetes Dashboard’u yerel makinenizden erişilebilir hale getirmek için servisi yerel makinenize yönlendirin

```
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8443:443
```
## Tarayıcıda Dashboard’a Erişim

Tarayıcınızda <https://localhost:8443> adresine gidin. Giriş ekranında, daha önce oluşturduğunuz token’i kullanarak giriş yapabilirsiniz.
