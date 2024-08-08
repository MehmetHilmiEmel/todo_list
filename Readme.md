# To-Do List Uygulaması

Bu proje, Flask kullanarak geliştirdiğim bir To-Do List backend uygulamasını içermektedir. Bu backend uygulaması aşağıdaki endpoint'lere sahiptir:

- **GET /tasks**: Oluşturulan görevleri çağırmamıza yarar.
- **POST /tasks**: Yeni bir görev oluşturmamıza olanak sağlar.
- **PATCH /tasks/id**: Belirli bir görevi tamamlandığında `true` olarak güncellememizi sağlar.
- **DELETE /tasks/id**: Belirli bir görevi silmemize olanak sağlar.

## Docker ve Kubernetes Entegrasyonu

Sonraki aşamada, bu uygulamayı bir container haline getirmek için Docker ve Docker Compose dosyalarını oluşturduk. Docker Compose dosyasında iki tane container oluşturduk:
1. Backend uygulaması için.
2. Veri tabanı için.

Docker Compose kullanarak `docker-compose up --build` komutunu çalıştırarak image ve container'ı oluşturduk. Oluşturduğumuz image'ı Docker Hub'a pushladık. Bunun amacı, Kubernetes kullanırken oluşturduğumuz image'ı Docker Hub'dan çekebilmektir.

Container'ları orkestre edebilmek için Kubernetes kullandık. Bunun için `kind create cluster` komutu ile bir cluster oluşturduk. Ardından `deployment.yaml` ve `service.yaml` dosyalarını yazdık:
- `deployment.yaml`: Hem veri tabanı hem de backend için deployment tanımladık. Backend'in replica sayısı 3, veri tabanının replica sayısı ise 1 olarak ayarlandı.
- `service.yaml`: Backend ve veri tabanı için servis tanımladık. 

  - Veri tabanı için:
    ```yaml
    - protocol: TCP
      port: 5432
      targetPort: 5432
      nodePort: 30008  # Bu port numarasını istediğiniz şekilde belirleyebilirsiniz, 30000-32767 aralığında olmalıdır
    ```

  - Backend için:
    ```yaml
    - protocol: TCP
      port: 80
      targetPort: 5000
      nodePort: 30007  # Bu port numarasını istediğiniz şekilde belirleyebilirsiniz, 30000-32767 aralığında olmalıdır
    ```

Son olarak, `kubectl apply -f deployment.yaml` ve `kubectl apply -f service.yaml` komutlarını çalıştırdık. Burada dikkat etmeniz gereken nokta, `app.py`, `docker-compose.yaml`, `deployment.yaml` ve `service.yaml` dosyalarında veri tabanı ismini aynı tutmaktır. Ben bu ismi `todo-postgres` olarak belirledim; aksi takdirde `CrashLoopBackOff` hatası alıyorduk.

`kubectl get po,svc` komutu ile çalışan pod'ları ve servisleri görebildik. En son olarak, `kubectl port-forward svc/todo-list-backend 30007:80` komutunu çalıştırarak test.rest dosyasındaki istekleri gönderdik ve başarılı bir yanıt aldık.

## Process, Container ve Virtual Machine Karşılaştırması

### Process

**Tanım**: Bir bilgisayar programının çalışan bir örneğidir. İşletim sistemi tarafından bağımsız olarak çalıştırılan ve yönetilen bir birimdir.

**Özellikleri**:
- Her proses, kendi bellek alanını kullanır.
- İşletim sistemindeki her proses, benzersiz bir proses kimliği (PID) ile tanımlanır.
- Prosesler, kaynaklar (CPU, bellek vb.) arasında paylaşım yapabilir.

**Tarihi Gelişim**:
- İlk olarak Unix tabanlı işletim sistemlerinde ortaya çıktı.
- İşletim sistemlerinin çoklu görev yetenekleri ile gelişti.

**Avantajları**:
- Hafif ve hızlıdırlar.
- İşletim sistemi çekirdeği ile doğrudan etkileşim kurabilirler.

**Dezavantajları**:
- Diğer proseslerle izole edilmemiştir, bu yüzden güvenlik riskleri olabilir.
- Sistem çökmelerinde tüm prosesler etkilenebilir.

### Virtual Machine (VM)

**Tanım**: Fiziksel bir bilgisayarın yazılımsal bir kopyasıdır. Hypervisor yazılımı kullanılarak oluşturulur ve çalıştırılır.

**Özellikleri**:
- Her VM, kendi işletim sistemine sahiptir.
- Fiziksel donanımı taklit eden sanal donanım bileşenleri içerir.
- VM'ler, birbirinden tamamen izole edilmiş ortamlar sağlar.

**Tarihi Gelişim**:
- 1960'larda IBM tarafından geliştirilmiştir.
- VMware, Microsoft Hyper-V, Oracle VM gibi çözümlerle popülerlik kazanmıştır.

**Avantajları**:
- Yüksek izolasyon seviyesi sağlarlar.
- Birden fazla farklı işletim sistemini aynı fiziksel donanım üzerinde çalıştırabilirler.

**Dezavantajları**:
- Kaynak yoğun ve ağırdırlar.
- Her VM, tam bir işletim sistemi çalıştırdığı için fazla bellek ve depolama tüketir.

### Container

**Tanım**: Uygulamaları ve onların bağımlılıklarını izole bir ortamda çalıştıran hafif sanal makineler gibidir. Docker, en popüler konteyner platformlarından biridir.

**Özellikleri**:
- Tüm bağımlılıkları ve konfigürasyonları ile birlikte paketlenir.
- İşletim sisteminin çekirdeğini paylaşır ama bağımsız bir kullanıcı alanı sunar.
- Hızlı başlar ve hafiftir.

**Tarihi Gelişim**:
- 2000'lerin başında Linux konteyner teknolojilerinin (LXC) ortaya çıkması ile gelişti.
- 2013'te Docker'ın piyasaya sürülmesi ile yaygınlaştı.

**Avantajları**:
- Hafif ve hızlıdırlar.
- VM'lerden daha az kaynak tüketirler.
- Uygulama taşınabilirliğini artırır.
- Geliştirme, test ve üretim ortamları arasında tutarlılık sağlar.

**Dezavantajları**:
- VM'ler kadar yüksek izolasyon sağlamazlar.
- İşletim sistemi çekirdeği ile paylaşım yaptıkları için bazı güvenlik riskleri barındırabilirler.

## Tarihsel Gelişim ve Karşılaştırma

**Prosesler**: İlk ve en temel çalışan birimdir. İşletim sistemi ile birlikte evrimleşmişlerdir.

**VM'ler**: Fiziksel donanımı verimli kullanma ihtiyacından doğmuşlardır. Özellikle veri merkezleri ve bulut bilişim için önemlidir.

**Konteynerler**: VM'lerin getirdiği kaynak tüketim sorunlarını çözmek ve daha hafif, taşınabilir çözümler sunmak için geliştirilmiştir. DevOps ve mikroservis mimarileri ile popüler hale gelmişlerdir.

## Sonuç

Her bir teknoloji, farklı ihtiyaçlara ve kullanım senaryolarına göre avantaj ve dezavantajlar sunar. Docker konteynerleri, özellikle modern uygulama geliştirme, dağıtım ve yönetim süreçlerinde büyük avantajlar sağlamaktadır.
