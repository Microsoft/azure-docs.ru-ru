---
title: Создание резервной копии приложения кластера Azure Red Hat OpenShift 4 с помощью Велеро
description: Узнайте, как создать резервную копию приложений кластера Azure Red Hat OpenShift с помощью Велеро
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 06/22/2020
author: troy0820
ms.author: b-trconn
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc
ms.openlocfilehash: bbfe280ed0b1b562e0f50b23a09ea159750c4a79
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102217097"
---
# <a name="create-an-azure-red-hat-openshift-4-cluster-application-backup"></a>Создание резервной копии приложения кластера Azure Red Hat OpenShift 4

В этой статье вы подготовите среду для создания резервной копии приложения кластера Azure Red Hat OpenShift 4. Вы узнаете, как:

> [!div class="checklist"]
> * Настройка необходимых компонентов и установка необходимых средств
> * Создание резервной копии приложений Azure Red Hat OpenShift 4

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Перед началом

### <a name="install-velero"></a>Установка Велеро

Чтобы [установить](https://velero.io/docs/main/basic-install/) велеро в системе, следуйте рекомендациям по работе с операционной системой.

### <a name="set-up-azure-storage-account-and-blob-container"></a>Настройка учетной записи хранения Azure и контейнера больших двоичных объектов

На этом шаге будет создана группа ресурсов за пределами группы ресурсов кластера АТО.  Эта группа ресурсов позволит сохранять резервные копии, а также восстанавливать приложения в новых кластерах.

```bash
AZURE_BACKUP_RESOURCE_GROUP=Velero_Backups
az group create -n $AZURE_BACKUP_RESOURCE_GROUP --location eastus

AZURE_STORAGE_ACCOUNT_ID="velero$(uuidgen | cut -d '-' -f5 | tr '[A-Z]' '[a-z]')"
az storage account create \
    --name $AZURE_STORAGE_ACCOUNT_ID \
    --resource-group $AZURE_BACKUP_RESOURCE_GROUP \
    --sku Standard_GRS \
    --encryption-services blob \
    --https-only true \
    --kind BlobStorage \
    --access-tier Hot

BLOB_CONTAINER=velero
az storage container create -n $BLOB_CONTAINER --public-access off --account-name $AZURE_STORAGE_ACCOUNT_ID
```

## <a name="set-permissions-for-velero"></a>Задание разрешений для Велеро

### <a name="create-service-principal"></a>Создание субъекта-службы

Велеро требуются разрешения на архивацию и восстановление. При создании субъекта-службы вы предоставляете Велеро разрешение на доступ к группе ресурсов, определенной на предыдущем шаге. На этом шаге будет получена группа ресурсов кластера:

```bash
export AZURE_RESOURCE_GROUP=$(az aro show --name <name of cluster> --resource-group <name of resource group> | jq -r .clusterProfile.resourceGroupId | cut -d '/' -f 5,5)
```


```bash
AZURE_SUBSCRIPTION_ID=$(az account list --query '[?isDefault].id' -o tsv)

AZURE_TENANT_ID=$(az account list --query '[?isDefault].tenantId' -o tsv)
```

```bash
AZURE_CLIENT_SECRET=$(az ad sp create-for-rbac --name "velero" --role "Contributor" --query 'password' -o tsv \
--scopes  /subscriptions/$AZURE_SUBSCRIPTION_ID)
AZURE_CLIENT_ID=$(az ad sp list --display-name "velero" --query '[0].appId' -o tsv)

```

```bash
cat << EOF  > ./credentials-velero.yaml
AZURE_SUBSCRIPTION_ID=${AZURE_SUBSCRIPTION_ID}
AZURE_TENANT_ID=${AZURE_TENANT_ID}
AZURE_CLIENT_ID=${AZURE_CLIENT_ID}
AZURE_CLIENT_SECRET=${AZURE_CLIENT_SECRET}
AZURE_RESOURCE_GROUP=${AZURE_RESOURCE_GROUP}
AZURE_CLOUD_NAME=AzurePublicCloud
EOF
```

## <a name="install-velero-on-azure-red-hat-openshift-4-cluster"></a>Установка Велеро в кластере Azure Red Hat OpenShift 4

Этот шаг устанавливает Велеро в собственный проект и [пользовательские определения ресурсов](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) , необходимые для резервного копирования и восстановления с помощью велеро. Убедитесь, что вы успешно вошли в кластер Azure Red Hat OpenShift v4.


```bash
velero install \
--provider azure \
--plugins velero/velero-plugin-for-microsoft-azure:v1.1.0 \
--bucket $BLOB_CONTAINER \
--secret-file ~/path/to/credentials-velero.yaml \
--backup-location-config resourceGroup=$AZURE_BACKUP_RESOURCE_GROUP,storageAccount=$AZURE_STORAGE_ACCOUNT_ID \
--snapshot-location-config apiTimeout=15m \
--velero-pod-cpu-limit="0" --velero-pod-mem-limit="0" \
--velero-pod-mem-request="0" --velero-pod-cpu-request="0"
```

## <a name="create-a-backup-with-velero"></a>Создание резервной копии с помощью Велеро

Чтобы создать резервную копию приложения с помощью Велеро, необходимо включить пространство имен, в котором находится это приложение.  Если у вас есть `nginx-example` пространство имен и вы хотите включить в резервную копию все ресурсы в этом пространстве имен, выполните в терминале следующую команду:

```bash
velero create backup <name of backup> --include-namespaces=nginx-example
```
Состояние резервного копирования можно проверить, выполнив команду.

```bash
oc get backups -n velero <name of backup> -o yaml
```

Будет выводиться успешная резервная копия `phase:Completed` , и объекты будут находиться в контейнере в учетной записи хранения.

## <a name="create-a-backup-with-velero-to-include-snapshots"></a>Создание резервной копии с Велеро для включения моментальных снимков

Чтобы создать резервную копию приложения с Велеро для включения постоянных томов приложения, необходимо включить пространство имен, в которое входит приложение, а также включить `snapshot-volumes=true` флаг при создании резервной копии.

```bash
velero backup create <name of backup> --include-namespaces=nginx-example --snapshot-volumes=true --include-cluster-resources=true
```

Состояние резервного копирования можно проверить, выполнив команду.

```bash
oc get backups -n velero <name of backup> -o yaml
```

Успешная архивация с выходными данными `phase:Completed` и объекты будут находиться в контейнере в учетной записи хранения.

Дополнительные сведения о создании резервных копий и восстановлении с помощью Велеро см. [в статье резервное копирование ресурсов OpenShift в собственном виде](https://www.openshift.com/blog/backup-openshift-resources-the-native-way) .

## <a name="next-steps"></a>Дальнейшие действия

В этой статье была создана резервная копия приложения кластера Azure Red Hat OpenShift 4. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание резервной копии приложения кластера OpenShift v4 с помощью Велеро
> * Создание резервной копии приложения кластера OpenShift v4 с моментальными снимками с помощью Велеро


Перейдите к следующей статье, чтобы узнать, как создать восстановление приложения кластера Azure Red Hat OpenShift 4.

* [Создание восстановления приложения кластера Azure Red Hat OpenShift 4](howto-create-a-restore.md)
* [Создание восстановления приложения кластера Azure Red Hat OpenShift 4, включая моментальные снимки](howto-create-a-restore.md)
