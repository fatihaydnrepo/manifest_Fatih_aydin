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

---

## 9. ArgoCD Kurulumu

ArgoCD, Kubernetes üzerinde GitOps tabanlı uygulama yönetimi sağlar. Kurulumu için aşağıdaki adımları izleyin:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n observability -f argocd-values.yaml
```

> Not: ArgoCD'nin varsayılan admin şifresini almak için:
> ```bash
> kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
> ```

ArgoCD UI'ya erişmek için port-forward:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:80
```
veya bir ingress tanımı ile dışarıya açabilirsiniz.

---

## Sealed Secrets Kurulumu (Ayrı Namespace ile)

### 1. Namespace Oluşturma
Öncelikle Sealed Secrets için ayrı bir namespace oluşturalım:

```sh
kubectl create namespace sealed-secrets
```

### 2. Helm ile Sealed Secrets Kurulumu

```sh
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo update
helm install sealed-secrets-controller sealed-secrets/sealed-secrets \
  --namespace sealed-secrets \
  --set fullnameOverride=sealed-secrets-controller
```

### 3. Kurulum Sonrası
- Sealed Secrets controller pod'u `sealed-secrets` namespace'inde çalışacak.
- Artık sealed secret manifestlerini bu controller ile kullanabilirsin.

Daha fazla bilgi için: [Sealed Secrets GitHub](https://github.com/bitnami-labs/sealed-secrets)

---

## NGINX Ingress Controller Kurulumu

Kubernetes cluster'ında Ingress kaynaklarını yönetmek için NGINX Ingress Controller kurabilirsiniz. Aşağıdaki adımları izleyin:

1. Gerekli Helm repo'yu ekleyin ve güncelleyin:

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

2. NGINX Ingress Controller'ı deploy edin:

```sh
helm install nginx-ingress ingress-nginx/ingress-nginx \
  -n observability --create-namespace \
  -f nginx-ingress-values.yaml
```

> Not: Değerleri kendi ortamınıza göre düzenlemek için `nginx-ingress-values.yaml` dosyasını kullanabilirsiniz.

Kurulumdan sonra, LoadBalancer tipi bir servis oluşacak ve dışarıdan erişim için kullanılabilecektir.
