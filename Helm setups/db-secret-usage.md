# PostgreSQL Connection String ve Sealed Secret Kullanımı

Bu doküman, .NET Core uygulamanızda PostgreSQL bağlantı bilgisini Kubernetes üzerinde **Sealed Secret** ile güvenli şekilde nasıl yöneteceğinizi ve kullanacağınızı adım adım açıklar.

---

## 1. Bağlantı Bilgisi (Connection String)

Örnek connection string’iniz:

```
Host=postgresql.observability.svc.cluster.local;Port=5432;Database=appdb;Username=appuser;Password=appsecret
```

psql ile test etmek için:
```
psql -h postgresql.observability.svc.cluster.local -U appuser -d appdb
```

---

## 2. Sealed Secret Oluşturma

Önce bir Kubernetes Secret oluşturup, ardından bunu Sealed Secret’a dönüştürün:

```sh
kubectl create secret generic db-conn-secret \
  --namespace dev \
  --from-literal=ConnectionStrings__DefaultConnection='Host=postgresql.observability.svc.cluster.local;Port=5432;Database=appdb;Username=appuser;Password=appsecret' \
  --dry-run=client -o json | \
  kubeseal \
    --controller-name=sealed-secrets-controller \
    --controller-namespace=sealed-secrets \
    --format yaml > db-secret-dev-sealed.yaml
```

> **Not:** `ConnectionStrings__DefaultConnection` anahtarı, .NET Core uygulamaları için environment variable olarak otomatik tanınır.

---

## 3. Sealed Secret’ı Apply Et

```sh
kubectl apply -f db-secret-dev-sealed.yaml
```

Secret oluştuğunu kontrol edin:
```sh
kubectl get secret -n dev
```

---

## 4. Deployment’ta Secret’ı Kullanmak

Deployment manifestinizde ilgili container’a şu env tanımını ekleyin:

```yaml
env:
  - name: ConnectionStrings__DefaultConnection
    valueFrom:
      secretKeyRef:
        name: db-conn-secret
        key: ConnectionStrings__DefaultConnection
```

---

## 5. Uygulama Otomatik Olarak Connection String’i Kullanır

.NET Core uygulamanızda ekstra kod değişikliği yapmanıza gerek yoktur. Uygulama, environment variable olarak gelen connection string’i otomatik olarak kullanır.

---

## 6. Bağlantıyı Manuel Test Etmek

Pod içinden veya dışarıdan bağlantıyı test etmek için:

```sh
psql -h postgresql.observability.svc.cluster.local -U appuser -d appdb
```

Şifreyi sorarsa, `appsecret` girin.

---

## 7. Sorun Giderme
- Secret oluşmazsa, sealed secret manifestinde `namespace: dev` olduğundan emin olun.
- Uygulama pod loglarında bağlantı hatası varsa, connection string’in doğru geçtiğinden ve secret’ın doğru namespace’te olduğundan emin olun.
- Sealed Secrets controller pod’u çalışıyor olmalı (`kubectl get pods -n sealed-secrets`).

---

Daha fazla bilgi için: [Sealed Secrets GitHub](https://github.com/bitnami-labs/sealed-secrets) 