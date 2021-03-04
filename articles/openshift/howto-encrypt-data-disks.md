---
title: Шифрование постоянного тома с помощью ключа, управляемого клиентом (CMK), в Azure Red Hat OpenShift (АТО)
description: Инструкции по развертыванию собственных ключей (BYOK) или ключа, управляемого клиентом (CMK) для Azure Red Hat OpenShift
ms.topic: article
ms.date: 02/24/2021
author: stuartatmicrosoft
ms.author: stkirk
ms.service: azure-redhat-openshift
keywords: Шифрование, byok, АТО, openshift, Red Hat
ms.openlocfilehash: 09e1f92a967c7b77d3bb8e27769f4cafd4fd4f53
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102055007"
---
# <a name="encrypt-persistent-volume-claims-with-a-customer-managed-key-cmk-on-azure-red-hat-openshift-aro-preview"></a>Зашифровать утверждения Постоянного тома с помощью ключа, управляемого клиентом (CMK) в Azure Red Hat OpenShift (АТО) (Предварительная версия)

Служба хранилища Azure шифрует все данные в неактивных учетных записях хранения. По умолчанию данные шифруются с помощью ключей, управляемых платформой Майкрософт, которые включают диски операционной системы и дисков данных. Для более быстрого управления ключами шифрования можно предоставить ключи, управляемые клиентом, для шифрования данных в кластерах Azure Red Hat OpenShift.

> [!NOTE]
> На этом этапе поддержка существует только для шифрования постоянных томов АТО с ключами, управляемыми клиентом. Эта функция в настоящее время недоступна для дисков операционной системы.

> [!IMPORTANT]
> Функции предварительной версии АТО доступны на уровне самообслуживания. Предварительные версии предоставляются «как есть» и «как есть», и они исключаются из соглашений об уровне обслуживания и ограниченной гарантии. Предварительные версии АТО частично охвачены службой поддержки клиентов. Таким образом, эти функции не предназначены для использования в рабочей среде.

## <a name="before-you-begin"></a>Перед началом
В этой статье предполагается, что:

* У вас есть существующий кластер АТО по адресу OpenShift версии 4,4 (или более поздней).

* У вас есть программа командной строки OpenShift "OC", Base64 (часть основных служебных программ) и "AZ" Azure CLI установлен.

* Вы вошли в кластер АТО, используя *OC* в качестве глобального кластера (кубеадмин).

* Вы вошли в Azure CLI с помощью команды *AZ* с учетной записью, которой предоставлен доступ для предоставления доступа участнику в той же подписке, что и кластер АТО.

## <a name="limitations"></a>Ограничения

* Доступность для шифрования ключей, управляемых клиентом, зависит от региона. Чтобы просмотреть состояние конкретного региона Azure, проверьте [регионы Azure][supported-regions].
* Если вы используете Ultra Disks, включите в подписку Ultra диски, прежде чем приступить к работе.

## <a name="create-an-azure-key-vault-instance"></a>Создание экземпляра Azure Key Vault
Для хранения ключей необходимо использовать экземпляр Azure Key Vault. Создайте новый Key Vault с включенной защитой очистки и обратимым удалением. Затем создайте новый ключ в хранилище для хранения собственного пользовательского ключа:

```azurecli-interactive
# Create an Azure Key Vault resource in a supported Azure region
az keyvault create -n $vaultName -g $buildRG --enable-purge-protection true -o table
# Create the actual key within the Azure Key Vault
az keyvault key create --vault-name $vaultName --name $vaultKeyName --protection software -o jsonc
```

## <a name="create-an-azure-disk-encryption-set"></a>Создание набора шифрования дисков Azure

Набор шифрования дисков Azure используется в качестве эталонной точки для дисков в АТО. Он подключен к Azure Key Vault, созданному на предыдущем шаге, и будет извлекать ключи, управляемые клиентом, из этого расположения.

```azurecli-interactive
# Retrieve the Key Vault Id and store it in a variable
keyVaultId="$(az keyvault show --name $vaultName --query [id] -o tsv)"

# Retrieve the Key Vault key URL and store it in a variable
keyVaultKeyUrl="$(az keyvault key show --vault-name $vaultName --name $vaultKeyName  --query [key.kid] -o tsv)"

# Create an Azure disk encryption set
az disk-encryption-set create -n $desName -g $myRG --source-vault $keyVaultId --key-url $keyVaultKeyUrl -o table
```

## <a name="grant-the-disk-encryption-set-access-to-key-vault"></a>Предоставьте набору шифрования дисков доступ к Key Vault
Используйте набор шифрования дисков, созданный на предыдущих шагах, и предоставьте набору шифрования дисков доступ к Azure Key Vault:

```azurecli-interactive
# First, find the disk encryption set's AppId value.
desIdentity="$(az disk-encryption-set show -n $desName -g $myRG --query [identity.principalId] -o tsv)"

# Next, update the Key Vault security policy settings to allow access to the disk encryption set.
az keyvault set-policy -n $vaultName -g $myRG --object-id $desIdentity --key-permissions wrapkey unwrapkey get -o table

# Now, ensure the disk encryption set can read the contents of the Azure Key Vault.
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId -o jsonc
```

### <a name="obtain-other-ids-required-for-role-assignments"></a>Получение других идентификаторов, необходимых для назначений ролей
Необходимо разрешить кластеру АТО использовать набор шифрования дисков для шифрования заявок на постоянные тома (PVC) в кластере АТО. Для этого мы создадим новый Управляемое удостоверение службы (MSI). Также будут заданы другие разрешения для MSI АТО и для набора шифрования диска.
```
# First, get the application ID of the service principal used in the ARO cluster.
aroSPAppId="$(oc get secret azure-credentials -n kube-system -o jsonpath='{.data.azure_client_id}' | base64 --decode)"

# Next, get the object ID of the service principal used in the ARO cluster.
aroSPObjId="$(az ad sp show --id $aroSPAppId -o tsv --query [objectId])"

# Set the name of the ARO Managed Service Identity. 
msiName="$aroCluster-msi"

# Create the Managed Service Identity (MSI) required for disk encryption.
az identity create -g $myRG -n $msiName -o jsonc

# Get the ARO Managed Service Identity application ID.
aroMSIAppId="$(az identity show -n $msiName -g $myRG -o tsv --query [clientId])"

# Get the resource ID for the disk encryption set and the Key Vault resource group.
myRGResourceId="$(az group show -n $myRG -o tsv --query [id])"
```

### <a name="implement-other-role-assignments-required-for-byokcmk-encryption"></a>Реализация других назначений ролей, необходимых для шифрования BYOK/CMK
Примените требуемые назначения ролей с помощью переменных, полученных на предыдущем шаге.

```azurecli-interactive
# Ensure the Azure Disk Encryption Set can read the contents of the Azure Key Vault.
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId -o jsonc

# Assign the MSI AppID 'Reader' permission over the disk encryption set & Key Vault resource group.
az role assignment create --assignee $aroMSIAppId --role Reader --scope $myRGResourceId -o jsonc

# Assign the ARO Service Principal 'Contributor' permission over the disk encryption set & Key Vault Resource Group.
az role assignment create --assignee $aroSPObjId --role Contributor --scope $myRGResourceId -o jsonc
```

## <a name="create-a-k8s-storage-class-for-encrypted-premium--ultra-disks-optional"></a>Создание класса хранения K8S для зашифрованных высокоскоростных дисков Premium & (необязательно)
Создайте классы хранения, которые будут использоваться для BYOK и CMK для дисков Premium_LRS и UltraSSD_LRS.
```
# Premium Disks
cat > managed-premium-encrypted-byok.yaml<< EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-encrypted-byok
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: Premium_LRS
  kind: Managed
  diskEncryptionSetID: "/subscriptions/subId/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/desName"
  resourceGroup: myRG
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
EOF

# Ultra Disks
cat > managed-ultra-encrypted-byok.yaml<< EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-ultra-encrypted-byok
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: UltraSSD_LRS
  kind: Managed
  diskEncryptionSetID: "/subscriptions/subId/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/desName"
  resourceGroup: myRG
  cachingmode: None
  diskIopsReadWrite: "2000"  # minimum value: 2 IOPS/GiB
  diskMbpsReadWrite: "320"   # minimum value: 0.032/GiB
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
EOF
```
### <a name="set-up-your-storage-class-configuration"></a>Настройка конфигурации класса хранения
Замените переменные, уникальные для кластера АТО, на два файла конфигурации класса хранения:
```
# Insert your current active subscription ID into the configuration
sed -i "s/subId/$subId/g" managed-premium-encrypted-byok.yaml
sed -i "s/subId/$subId/g" managed-ultra-encrypted-byok.yaml

# Replace the name of the Resource Group which contains the disk encryption set and Key Vault
sed -i "s/myRG/$myRG/g" managed-premium-encrypted-byok.yaml
sed -i "s/myRG/$myRG/g" managed-ultra-encrypted-byok.yaml

# Replace the name of the disk encryption set
sed -i "s/desName/$desName/g" managed-premium-encrypted-byok.yaml
sed -i "s/desName/$desName/g" managed-ultra-encrypted-byok.yaml
```
Затем запустите это развертывание в кластере АТО, чтобы применить конфигурацию класса хранения:
```
# Update cluster with the new storage classes
oc apply -f managed-premium-encrypted-byok.yaml
oc apply -f managed-ultra-encrypted-byok.yaml
```
## <a name="test-encryption-with-customer-managed-keys"></a>Тестирование шифрования с помощью управляемых клиентом ключей
Чтобы проверить, использует ли кластер ключ, управляемый клиентом, для шифрования PVC, мы создадим утверждение Постоянного тома с помощью соответствующего класса хранилища. Приведенный ниже фрагмент кода создает Pod и подключает постоянное требование к тому с помощью стандартных дисков.
```
# Create a pod which uses a persistent volume claim referencing the new storage class
cat > test-pvc.yaml<< EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypod-with-byok-encryption-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium-encrypted-byok
  resources:
    requests:
      storage: 1Gi
---
kind: Pod
apiVersion: v1
metadata:
  name: mypod-with-byok-encryption
spec:
  containers:
  - name: mypod-with-byok-encryption
    image: nginx:1.15.5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: mypod-with-byok-encryption-pvc
EOF
```
### <a name="apply-the-test-pod-configuration-file"></a>Применение файла конфигурации тестового модуля
Выполните приведенные ниже команды, чтобы применить конфигурацию модуля тестирования и вернуть идентификатор нового утверждения Постоянного тома. Идентификатор UID будет использоваться для проверки того, зашифрован ли диск с помощью BYOK/CMK.
```
# Apply the test pod configuration file and set the PVC UID as a variable to query in Azure later.
pvcUid="$(oc apply -f test-pvc.yaml -o json | jq -r '.items[0].metadata.uid')"

# Determine the full Azure Disk name.
pvName="$(oc get pv pvc-$pvcUid -o json |jq -r '.spec.azureDisk.diskName')"
```
### <a name="verify-pvc-disk-is-configured-with-encryptionatrestwithcustomerkey"></a>Проверка настройки диска PVC на "Енкриптионатрествискустомеркэй" 
Модуль должен создать постоянное утверждение тома, которое ссылается на класс хранилища BYOK/CMK. При выполнении следующей команды будет выполнена проверка развертывания PVC, как ожидалось:
```azurecli-interactive
# Describe the OpenShift cluster-wide persistent volume claims
oc describe pvc

# Verify with Azure that the disk is encrypted with a customer-managed key
az disk show -n $pvName -g $myRG -o json --query [encryption]
```

## <a name="next-steps"></a>Дальнейшие действия

<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: /azure/aks/operator-best-practices-cluster-security
[byok-azure-portal]: /azure/storage/common/storage-encryption-keys-portal
[customer-managed-keys]: /azure/virtual-machines/windows/disk-encryption#customer-managed-keys
[key-vault-generate]: /azure/key-vault/key-vault-manage-with-cli2
[supported-regions]: /azure/virtual-machines/windows/disk-encryption#supported-regions

