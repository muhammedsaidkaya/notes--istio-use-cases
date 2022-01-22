**RKE2 Cluster kuruldu** 

- Terraform 4 makina kaldırıldı.
- Jumpa Private Key kopyalandı.
- RKE2 Repo Github dan klonlandı.
- Python Pip3 ve Pip3 ile Ansible Install edildi
- Ansible için PATH update edildi.
- RKE2 playbook provision edildi.
- Worker da config.yaml içerisindeki cluster-cidr, service-cidr silinip rke2-agent systemd servisi restart edildi. (Playbook HATALI)
- Master da kubectl erişimi için PATH update edildi. (/var/lib/rancher/rke2/bin)
- Master da kubeconfig /etc/rancher/rke2/rke2.yaml dan home altındaki .kube a config olarak kopyalandı.
- Latest Istioctl install edildi.

**Istioctl ile Profile ı demo olan Istiod kuruldu**

**Default namespace e istio-injection=enabled diye LABEL verildi. Ardından BookInfo mikroservisi ile Istio-agentlar sidecar olarak inject edildi.**

**Gateway oluşturulup IstioIngressGateway e attach edildi.**

**Gateway den gelen requestleri eşleşen URI lara göre productpage svc sine yönlendiren VirtualService oluşturuldu.**

**Productpage den Reviews svc sine giden paketler, VirtualService ve DestinationRule subsetleri oluşturularak weight değerine göre routing yapıldı. (Sonucunda Canary Deployment ve A/B Testing gerçekleştirilebilir)**

**ProductPage e gelen http ve tcp requestlerine limit konarak - Circuit Breaker denendi.**

**Throttling için http paketinin header ındaki key value ye göre routing yapıldı.**

**Mikroservisler arası Authorization gerçekleştirildi. (AuthorizationPolicy). Böylelike svc lerin ClusterIP leri bile olsa request atma işlemi gerçekleştirilemiyor.**

**(?) mTLS namespace bazlı enable edilerek mikroservisler arası giden paketlerin encryption işlemi gerçekleştirildi.**

**Paketleri delay ederek ve timeout süreleri vererek Fault Injection işlemi gerçekleştirildi.**

**Prometheus, Jaeger, Kiali Addon ları kuruldu.**

**Kiali Metrik, Jeager Trace takibi yapıldı.**

**Bonus:** Instana agent kurularak Application Monitoring yapılmaya çalışıldı. Fakat çalışmadı :(