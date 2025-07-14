# Observability Bileşenleri Kurulum ve PV Tanımları

Bu dokümanda, Helm ile kuracağınız tüm observability bileşenlerinin (Grafana, Loki, Tempo, OpenTelemetry Collector, Jenkins, Prometheus, PostgreSQL) kurulum komutları ve gerekli PersistentVolume (PV) tanımları yer almaktadır.

---

## 1. Namespace Oluşturma
```bash
kubectl create namespace observability
```

---

## 2. Grafana Kurulumu
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana -n observability -f grafana-values.yaml
```

---

## 3. Loki Kurulumu
```bash
helm install loki grafana/loki -n observability -f loki-values.yaml
```

---

## 4. Tempo Kurulumu
```bash
helm install tempo grafana/tempo -n observability -f tempo-values.yaml
```

---

## 5. OpenTelemetry Collector Kurulumu
```bash
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
helm install otel-collector open-telemetry/opentelemetry-collector -n observability -f otel-collector-values.yaml
```

> OpenTelemetry Collector için genellikle PV gerekmez, ancak gerekiyorsa benzer şekilde ekleyebilirsiniz.

---

## 6. Jenkins Kurulumu
```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install jenkins jenkins/jenkins -n observability -f jenkins-values.yaml
```

---

## 7. Prometheus ve Prometheus-Operator (kube-prometheus-stack) Kurulumu
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n observability --create-namespace -f prometheus-values.yaml
```

> Not: Prometheus ve Alertmanager için de dinamik NFS storage kullanmak isterseniz, prometheus-values.yaml dosyanızda persistence.storageClass: nfs-client olarak ayarlayın.

---

## 8. PostgreSQL Kurulumu
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install postgresql bitnami/postgresql -n observability -f postgresql-values.yaml
```

> Not: PostgreSQL için de dinamik NFS storage kullanmak isterseniz, postgresql-values.yaml dosyanızda persistence.storageClass: nfs-client olarak ayarlayın.

---

> Not: Tüm bileşenlerin values.yaml dosyalarında persistence.storageClass: nfs-client olarak ayarlanmalıdır.
> NFS sunucu ve path bilgilerini kendi ortamınıza göre güncelleyiniz.
> nfs-client gibi bir storageClass ve dinamik NFS provisioner varsa, PV'leri elle oluşturmanıza gerek yoktur. PVC otomatik olarak oluşur ve provisioner yeni PV yaratır.
