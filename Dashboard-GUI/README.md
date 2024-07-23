# Kubernetes Dashboard GUI Kurulumu
 
 ![Dashboard Kurulumu](Kubernetes/resim/resim.png)

Bu klasör, Kubernetes dashboard GUI kurulumu ile ilgili konfigürasyon dosyalarını içerir.

# Kubernetes Dashboard Kurulumu ve Yönetimi

Kubernetes Dashboard, Kubernetes kümenizi yönetmenize ve izlemenize olanak tanıyan web tabanlı bir kullanıcı arayüzüdür. Bu kılavuzda, Kubernetes Dashboard’un nasıl kurulacağını ve yapılandırılacağını adım adım ele alacağız.

## Kubernetes Dashboard’u Yükleme

Kubernetes Dashboard’un en güncel sürümünü yüklemek için, Kubernetes’in resmi GitHub deposundan `recommended.yaml` dosyasını indirmeniz gerekiyor. Bu dosya, Dashboard ve gerekli tüm kaynakları içeren bir yapılandırma dosyasıdır.

```bash
curl -LO https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml

