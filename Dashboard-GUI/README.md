# Kubernetes Dashboard GUI Kurulumu

Bu klasör, Kubernetes Dashboard GUI kurulumu ile ilgili konfigürasyon dosyalarını içerir. Aşağıdaki belgeler, Kubernetes Dashboard’un nasıl kurulacağı ve yapılandırılacağı hakkında adım adım bilgi sunmaktadır.

## İçerikler

- **admin-rolebinding.yaml**: Dashboard’a admin yetkileri ile erişim sağlamak için gerekli yapılandırma dosyası.

## Kubernetes Dashboard Kurulumu ve Yönetimi

Kubernetes Dashboard, Kubernetes kümenizi yönetmenize ve izlemenize olanak tanıyan web tabanlı bir kullanıcı arayüzüdür. Bu kılavuzda, Kubernetes Dashboard’un nasıl kurulacağını ve yapılandırılacağını adım adım ele alacağız.

### 1. Kubernetes Dashboard’u Yükleme

Kubernetes Dashboard’un en güncel sürümünü yüklemek için, Kubernetes’in resmi GitHub deposundan `recommended.yaml` dosyasını indirmeniz gerekiyor:

```bash
curl -LO https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml

