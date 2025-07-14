# Helm ile NFS Subdir External Provisioner Kurulumu

Bu dokümanda, Kubernetes ortamında dinamik NFS persistent volume provisioning için `nfs-subdir-external-provisioner` kurulumu adımları anlatılmaktadır.

## 1. Helm Repo Ekleme

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update
```

## 2. NFS Sunucu Bilgilerini Belirleyin

Kurulumdan önce aşağıdaki bilgilere ihtiyacınız olacak:
- NFS sunucu IP adresi veya DNS adı
- NFS üzerinde kullanılacak path (ör: `/srv/nfs/kubedata`)

## 3. Kurulum Komutu

Aşağıdaki komutu kendi NFS sunucu ve path bilginize göre düzenleyerek çalıştırın:

```bash
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=<NFS_SERVER_IP_ADRESI> --set nfs.path=<NFS_PATH>
```

Örnek:
```bash
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.1.100 \
  --set nfs.path=/srv/nfs/kubedata
```

## 4. StorageClass Kontrolü

Kurulumdan sonra aşağıdaki komut ile StorageClass oluştuğunu kontrol edin:

```bash
kubectl get storageclass
```

`nfs-client` adında bir StorageClass oluşmuş olmalı. Bunu varsayılan yapmak için:

```bash
kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## 5. Test

Bir PVC ve Pod ile NFS provisioner'ın çalıştığını test edebilirsiniz.

