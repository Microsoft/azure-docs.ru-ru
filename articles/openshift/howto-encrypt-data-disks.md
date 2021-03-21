---
title: Шифрование постоянного тома с помощью ключа, управляемого клиентом (CMK), в Azure Red Hat OpenShift (АТО)
description: Инструкции по развертыванию собственных ключей (BYOK) или ключа, управляемого клиентом (CMK) для Azure Red Hat OpenShift
ms.topic: article
ms.date: 02/24/2021
author: stuartatmicrosoft
ms.author: stkirk
ms.service: azure-redhat-openshift
keywords: Шифрование, byok, АТО, CMK, openshift, Red Hat
ms.openlocfilehash: fa84096dcc44e668a6cf7ebd0369c6d3631c28d2
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102555624"
---
# <a name="encrypt-persistent-volume-claims-with-a-customer-managed-key-cmk-on-azure-red-hat-openshift-aro-preview"></a>Зашифровать утверждения Постоянного тома с помощью ключа, управляемого клиентом (CMK) в Azure Red Hat OpenShift (АТО) (Предварительная версия)

Служба хранилища Azure использует шифрование на стороне сервера (SSE) для автоматического [шифрования](../storage/common/storage-service-encryption.md) данных при их сохранении в облаке. По умолчанию данные шифруются с помощью ключей, управляемых платформой Майкрософт. Для дополнительного контроля над ключами шифрования можно предоставить собственные ключи, управляемые клиентом, для шифрования данных в кластерах Azure Red Hat OpenShift.

> [!NOTE]
> На этом этапе поддержка существует только для шифрования постоянных томов АТО с ключами, управляемыми клиентом. Эта функция сейчас недоступна для дисков операционной системы главного или рабочего узла.

> [!IMPORTANT]
> Функции предварительной версии АТО доступны на уровне самообслуживания. Предварительные версии предоставляются «как есть» и «как есть» и исключаются из соглашений об уровне обслуживания и ограниченной гарантии. Предварительные версии АТО частично охвачены службой поддержки клиентов. Таким образом, эти функции не предназначены для использования в рабочей среде.

## <a name="before-you-begin"></a>Перед началом
В этой статье предполагается, что:

* У вас есть существующий кластер АТО по адресу OpenShift версии 4,4 (или более поздней).

* У вас есть программа командной строки **OC** OpenShift, Base64 (часть кореутилс) и **AZ** Azure CLI installed.

* Вы вошли в кластер АТО с помощью **OC** в качестве глобального кластера (кубеадмин).

* Вы вошли в Azure CLI с помощью команды **AZ** с учетной записью, которой разрешено предоставлять доступ участника в той же подписке, что и кластер АТО.

## <a name="limitations"></a>Ограничения

* Доступность для шифрования ключей, управляемых клиентом, зависит от региона. Чтобы просмотреть состояние конкретного региона Azure, проверьте [регионы Azure][supported-regions].
* Если вы хотите использовать Ultra Disks, прежде чем приступить к работе, необходимо включить их в подписке.

## <a name="declare-cluster--encryption-variables"></a>Объявление переменных шифрования & кластера
Следует настроить приведенные ниже переменные так, чтобы вы могли подойти к кластеру АТО, в котором вы хотите включить управляемые клиентом ключи шифрования:
```
aroCluster="mycluster"             # The name of the ARO cluster that you wish to enable CMK on. This may be obtained from **az aro list -o table**
buildRG="mycluster-rg"             # The name of the resource group used when you initially built the ARO cluster. This may be obtained from **az aro list -o table**
desName="aro-des"                  # Your Azure Disk Encryption Set name. This must be unique in your subscription.
vaultName="aro-keyvault-1"         # Your Azure Key Vault name. This must be unique in your subscription.
vaultKeyName="myCustomAROKey"      # The name of the key to be used within your Azure Key Vault. This is the name of the key, not the actual value of the key that you will rotate.
```

## <a name="obtain-your-subscription-id"></a>Получение идентификатора подписки
Идентификатор подписки Azure используется несколько раз в конфигурации CMK. Получите его и сохраните в виде переменной:
```azurecli-interactive
# Obtain your Azure Subscription ID and store it in a variable
subId="$(az account list -o tsv | grep True | awk '{print $3}')"
```

## <a name="create-an-azure-key-vault-instance"></a>Создание экземпляра Azure Key Vault
Для хранения ключей необходимо использовать экземпляр Azure Key Vault. Создайте новый Key Vault с включенной защитой очистки. Затем создайте новый ключ в хранилище для хранения собственного пользовательского ключа:

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
az disk-encryption-set create -n $desName -g $buildRG --source-vault $keyVaultId --key-url $keyVaultKeyUrl -o table
```

## <a name="grant-the-disk-encryption-set-access-to-key-vault"></a>Предоставьте набору шифрования дисков доступ к Key Vault
Используйте набор шифрования дисков, созданный на предыдущих шагах, и предоставьте набору шифрования дисков доступ к Azure Key Vault:

```azurecli-interactive
# First, find the disk encryption set's Azure Application ID value.
desIdentity="$(az disk-encryption-set show -n $desName -g $buildRG --query [identity.principalId] -o tsv)"

# Next, update the Key Vault security policy settings to allow access to the disk encryption set.
az keyvault set-policy -n $vaultName -g $buildRG --object-id $desIdentity --key-permissions wrapkey unwrapkey get -o table

# Now, ensure the Disk Encryption Set can read the contents of the Azure Key Vault.
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId -o jsonc
```

### <a name="obtain-other-ids-required-for-role-assignments"></a>Получение других идентификаторов, необходимых для назначений ролей
Необходимо разрешить кластеру АТО использовать набор шифрования дисков для шифрования заявок на постоянные тома (PVC) в кластере АТО. Для этого будет создан новый Управляемое удостоверение службы (MSI). Также будут заданы другие разрешения для MSI АТО и для набора шифрования диска.

```azurecli-interactive
# First, get the Azure Application ID of the service principal used in the ARO cluster.
aroSPAppId="$(oc get secret azure-credentials -n kube-system -o jsonpath='{.data.azure_client_id}' | base64 --decode)"

# Next, get the object ID of the service principal used in the ARO cluster.
aroSPObjId="$(az ad sp show --id $aroSPAppId -o tsv --query [objectId])"

# Set the name of the ARO Managed Service Identity. 
msiName="$aroCluster-msi"

# Create the Managed Service Identity (MSI) required for disk encryption.
az identity create -g $buildRG -n $msiName -o jsonc

# Get the ARO Managed Service Identity Azure Application ID.
aroMSIAppId="$(az identity show -n $msiName -g $buildRG -o tsv --query [clientId])"

# Get the resource ID for the disk encryption set and the Key Vault resource group.
buildRGResourceId="$(az group show -n $buildRG -o tsv --query [id])"
```

### <a name="implement-other-role-assignments-required-for-cmk-encryption"></a>Реализация других назначений ролей, необходимых для шифрования CMK
Примените требуемые назначения ролей с помощью переменных, полученных на предыдущем шаге.

```azurecli-interactive
# Ensure the Azure Disk Encryption Set can read the contents of the Azure Key Vault.
az role assignment create --assignee $desIdentity --role Reader --scope $keyVaultId -o jsonc

# Assign the MSI AppID 'Reader' permission over the disk encryption set & Key Vault resource group.
az role assignment create --assignee $aroMSIAppId --role Reader --scope $buildRGResourceId -o jsonc

# Assign the ARO Service Principal 'Contributor' permission over the disk encryption set & Key Vault Resource Group.
az role assignment create --assignee $aroSPObjId --role Contributor --scope $buildRGResourceId -o jsonc
```

## <a name="create-a-k8s-storage-class-for-encrypted-premium--ultra-disks"></a>Создание класса хранения K8S для зашифрованных высокоскоростных дисков & Premium
Создайте классы хранения, которые будут использоваться для CMK для дисков Premium_LRS и UltraSSD_LRS.

```azurecli-interactive
# Premium Disks
cat > managed-premium-encrypted-cmk.yaml<< EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-encrypted-cmk
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: Premium_LRS
  kind: Managed
  diskEncryptionSetID: "/subscriptions/$subId/resourceGroups/$buildRG/providers/Microsoft.Compute/diskEncryptionSets/$desName"
  resourceGroup: $buildRG
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
EOF

# Ultra Disks
cat > managed-ultra-encrypted-cmk.yaml<< EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-ultra-encrypted-cmk
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: UltraSSD_LRS
  kind: Managed
  diskEncryptionSetID: "/subscriptions/$subId/resourceGroups/$buildRG/providers/Microsoft.Compute/diskEncryptionSets/$desName"
  resourceGroup: $buildRG
  cachingmode: None
  diskIopsReadWrite: "2000"  # minimum value: 2 IOPS/GiB
  diskMbpsReadWrite: "320"   # minimum value: 0.032/GiB
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
EOF
```

Затем запустите это развертывание в кластере АТО, чтобы применить конфигурацию класса хранения:

```azurecli-interactive
# Update cluster with the new storage classes
oc apply -f managed-premium-encrypted-cmk.yaml
oc apply -f managed-ultra-encrypted-cmk.yaml
```

## <a name="test-encryption-with-customer-managed-keys-optional"></a>Проверка шифрования с помощью управляемых клиентом ключей (необязательно)
Чтобы проверить, использует ли кластер ключ, управляемый клиентом, для шифрования PVC, мы создадим утверждение Постоянного тома с помощью нового класса хранилища. Приведенный ниже фрагмент кода создает Pod и подключает устойчивое утверждение тома с помощью дисков уровня "Премиум".
```
# Create a pod which uses a persistent volume claim referencing the new storage class
cat > test-pvc.yaml<< EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypod-with-cmk-encryption-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium-encrypted-cmk
  resources:
    requests:
      storage: 1Gi
---
kind: Pod
apiVersion: v1
metadata:
  name: mypod-with-cmk-encryption
spec:
  containers:
  - name: mypod-with-cmk-encryption
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
        claimName: mypod-with-cmk-encryption-pvc
EOF
```
### <a name="apply-the-test-pod-configuration-file-optional"></a>Применение файла конфигурации тестового модуля (необязательно)
Выполните приведенные ниже команды, чтобы применить конфигурацию модуля тестирования и вернуть идентификатор нового утверждения Постоянного тома. Идентификатор UID будет использоваться для проверки того, зашифрован ли диск с помощью CMK.
```azurecli-interactive
# Apply the test pod configuration file and set the PVC UID as a variable to query in Azure later.
pvcUid="$(oc apply -f test-pvc.yaml -o jsonpath='{.items[0].metadata.uid}')"

# Determine the full Azure Disk name.
pvName="$(oc get pv pvc-$pvcUid -o jsonpath='{.spec.azureDisk.diskName}')"
```
> [!NOTE]
> В некоторых случаях при применении назначений ролей в Azure Active Directory может возникнуть небольшая задержка. В зависимости от скорости выполнения этих инструкций команда "определить полное имя диска Azure" может не выполняться. В этом случае ознакомьтесь с выходными данными о **PVC мипод-with-CMK-Encryption-PVC** , чтобы убедиться, что диск успешно подготовлен. Если распространение назначения ролей не завершено, возможно, потребуется *Удалить* и повторно *применить* модуль Pod & PVC YAML.

### <a name="verify-pvc-disk-is-configured-with-encryptionatrestwithcustomerkey-optional"></a>Убедитесь, что для диска PVC настроено "Енкриптионатрествискустомеркэй" (необязательно)
Модуль должен создать постоянное утверждение тома, которое ссылается на класс хранилища CMK. При выполнении следующей команды будет выполнена проверка развертывания PVC, как ожидалось:
```azurecli-interactive
# Describe the OpenShift cluster-wide persistent volume claims
oc describe pvc

# Verify with Azure that the disk is encrypted with a customer-managed key
az disk show -n $pvName -g $buildRG -o json --query [encryption]
```

<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: /azure/aks/operator-best-practices-cluster-security
[byok-azure-portal]: /azure/storage/common/storage-encryption-keys-portal
[customer-managed-keys]: /azure/virtual-machines/windows/disk-encryption#customer-managed-keys
[key-vault-generate]: /azure/key-vault/key-vault-manage-with-cli2
[supported-regions]: /azure/virtual-machines/windows/disk-encryption#supported-regions

