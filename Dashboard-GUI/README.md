# Kubernetes Dashboard GUI Kurulumu 

Bu klasör, Kubernetes dashboard GUI kurulumu ile ilgili konfigürasyon dosyalarını içerir.

# Kubernetes Dashboard Kurulumu ve Yönetimi

Kubernetes Dashboard, Kubernetes kümenizi yönetmenize ve izlemenize olanak tanıyan web tabanlı bir kullanıcı arayüzüdür. Bu kılavuzda, Kubernetes Dashboard’un nasıl kurulacağını ve yapılandırılacağını adım adım ele alacağız.

## Kubernetes Dashboard’u Yükleme

Kubernetes Dashboard’un en güncel sürümünü yüklemek için, Kubernetes’in resmi GitHub deposundan `recommended.yaml` dosyasını indirmeniz gerekiyor. Bu dosya, Dashboard ve gerekli tüm kaynakları içeren bir yapılandırma dosyasıdır.

```bash
curl -LO https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml

Bu komut, recommended.yaml dosyasını yerel makinenize indirecektir.

recommended.yaml Dosyasını Uygulayın
İndirdiğiniz yapılandırma dosyasını uygulayarak Dashboard’un Kubernetes kümenize kurulmasını sağlayın
Bu komut, Kubernetes Dashboard’un gerekli kaynaklarını (örneğin, Deployment, Service, vs.) oluşturacaktır.
Hizmet Hesabı ve ClusterRoleBinding Oluşturma
Dashboard’a erişmek için yetkilendirme gereklidir. Bunun için bir ServiceAccount ve ClusterRoleBinding oluşturmanız gerekir.
admin-rolebinding.yaml Dosyasını Oluşturun
Bir metin düzenleyici ile admin-rolebinding.yaml adında bir dosya oluşturun ve aşağıdaki içeriği ekleyin

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

