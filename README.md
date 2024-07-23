# Kubernetes Proje Dokümantasyonu

Bu doküman, Kubernetes üzerinde çeşitli projelerin kurulumunu ve yapılandırmasını adım adım açıklar. Her bölüm, ilgili bileşenlerin nasıl kurulacağını ve yönetileceğini ayrıntılı olarak anlatır.

## Kubernetes Dashboard GUI Kurulumu

Bu proje, Kubernetes Dashboard'un kurulumunu ve yapılandırmasını açıklar. Kubernetes Dashboard, Kubernetes kümenizi web tabanlı bir arayüz üzerinden yönetmenizi ve izlemenizi sağlar. Kurulumdan sonra Dashboard'u yerel makinenizde çalıştırabilir ve Kubernetes kaynaklarınızı grafiksel bir arayüzden yönetebilirsiniz.

### Kurulum Adımları:

1. **Yükleme**: Kubernetes Dashboard'un en güncel sürümünü yükleyin.
2. **Yetkilendirme**: Dashboard'a erişim için gerekli ServiceAccount ve ClusterRoleBinding oluşturun.
3. **Token Oluşturma**: Dashboard'a giriş yapmak için bir token oluşturun.
4. **Erişim Sağlama**: Dashboard'u yerel makinenizden erişilebilir hale getirmek için `kubectl port-forward` komutunu kullanın.
5. **Tarayıcıda Erişim**: Dashboard'a web tarayıcınız üzerinden erişim sağlayın.

## Kubernetes HPA (Horizontal Pod Autoscaler) Kurulumu

Horizontal Pod Autoscaler (HPA), belirli metriklere göre Pod sayısını otomatik olarak ölçeklendiren bir Kubernetes bileşenidir. Bu proje, HPA'nın nasıl kurulacağını ve yapılandırılacağını gösterir, böylece Pod'larınızın performansını dinamik olarak optimize edebilirsiniz.

### Kurulum Adımları:

1. **Namespace Oluşturma**: HPA için bir namespace oluşturun.
2. **Deployment Oluşturma**: HPA'nın ölçeklendireceği bir deployment örneği oluşturun.
3. **HPA Oluşturma**: CPU ve bellek kullanımına göre Pod sayısını otomatik olarak ölçeklendiren bir HPA nesnesi oluşturun.
4. **Durum Kontrolü**: HPA'nın çalışma durumunu kontrol edin.

## Kubernetes’te Hata Veren Pod’ları Housekeeping CronJob ile Temizleme

Bu proje, Kubernetes ortamında hata veren Pod'ları düzenli olarak temizleyen bir CronJob'un nasıl oluşturulacağını açıklar. Bu işlem, kaynakların boşa harcanmasını önler ve sistemin sağlığını korur.

### Kurulum Adımları:

1. **Namespace Oluşturma**: Housekeeping işlemleri için bir namespace oluşturun.
2. **Hatalı Deployment Oluşturma**: Yanlış yapılandırılmış bir deployment örneği oluşturun.
3. **ServiceAccount, ClusterRole ve ClusterRoleBinding Oluşturma**: CronJob'un gerekli kaynaklara erişim izni sağlanır.
4. **Housekeeping CronJob Oluşturma**: Belirli durumlarda olan Pod'ları temizleyecek bir CronJob oluşturun.
5. **Log Kontrolü**: CronJob'un loglarını kontrol edin.

## Kubernetes Projesinde Nginx Gateway ve Backend Pod’ları ile Log Yönetimi ve Basit HTML Sayfası Oluşturma

Bu proje, Kubernetes üzerinde iki Nginx pod'u oluşturmayı içerir: biri gateway olarak görev yapar, diğeri ise backend olarak çalışır. Backend pod'u, gelen isteklere basit bir HTML sayfası sunar ve logları bir Persistent Volume üzerinde saklanır.

### Kurulum Adımları:

1. **Persistent Volume ve Persistent Volume Claim Oluşturma**: Verilerin saklanacağı PV ve PVC oluşturun.
2. **ConfigMap Oluşturma**: Backend pod'unun sunacağı HTML sayfasını bir ConfigMap içinde saklayın.
3. **Backend Deployment ve Service Oluşturma**: Backend pod'unu oluşturun ve ConfigMap'ten HTML dosyasını sunun.
4. **Gateway Deployment ve Service Oluşturma**: Gateway pod'unu oluşturun ve backend pod'una gelen istekleri yönlendirin.
5. **Log ve HTML Dosyası Kontrolleri**: Backend ve Gateway pod'larının log ve HTML dosyalarını kontrol edin.
6. **İnternet Sağlayıcı Üzerinden Erişim**: Backend pod'una internet üzerinden nasıl erişileceğini açıklayın.

## Kubernetes’te Pod Yeniden Başlatma CronJob’u Oluşturma

Bu proje, belirli zaman aralıklarında pod’ları yeniden başlatmayı ve durumlarını kontrol etmeyi amaçlayan bir CronJob oluşturma sürecini ayrıntılı olarak açıklar. Her gece saat 00:00'da pod’ları yeniden başlatmayı hedefler.

### Kurulum Adımları:

1. **Deployment Oluşturma**: Yeniden başlatmak istediğiniz pod'u içeren bir deployment oluşturun.
2. **ServiceAccount Oluşturma**: CronJob’un belirli kaynaklara erişim izni olması gerekir.
3. **Role ve RoleBinding Oluşturma**: CronJob'un gerekli izinlere sahip olmasını sağlar ve bu izinleri ServiceAccount ile bağlar.
4. **CronJob Oluşturma**: Belirli aralıklarla pod'ları yeniden başlatacak bir CronJob oluşturun.
5. **CronJob Zamanlaması**: CronJob’ların zamanlamasını ayarlayın.
6. **Log Kontrolü**: CronJob’un loglarını kontrol edin.
