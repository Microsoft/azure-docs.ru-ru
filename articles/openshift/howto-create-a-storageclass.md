---
title: Создание StorageClass службы "Файлы Azure" в Azure Red Hat OpenShift 4
description: Узнайте, как создать файлы Azure Сторажекласс в Azure Red Hat OpenShift
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 10/16/2020
author: grantomation
ms.author: b-grodel
keywords: АТО, openshift, AZ АТО, Red Hat, CLI, файл Azure
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 039aa3cce6615e71960db810ae383d22d7bcd909
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102212983"
---
# <a name="create-an-azure-files-storageclass-on-azure-red-hat-openshift-4"></a>Создание StorageClass службы "Файлы Azure" в Azure Red Hat OpenShift 4

В этой статье вы создадите Сторажекласс для Azure Red Hat OpenShift 4, который динамически подготавливает хранилище Реадвритемани (RWX) с помощью службы файлов Azure. Вы узнаете, как выполнять такие задачи.

> [!div class="checklist"]
> * Настройка необходимых компонентов и установка необходимых средств
> * Создание Azure Red Hat OpenShift 4 Сторажекласс с помощью подготовки файлов Azure

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Перед началом

Развертывание кластера Azure Red Hat OpenShift 4 в подписке см. в статье [Создание кластера Azure Red Hat OpenShift 4](tutorial-create-cluster.md) .


### <a name="set-up-azure-storage-account"></a>Настройка учетной записи хранения Azure

На этом шаге будет создана группа ресурсов за пределами группы ресурсов кластера Azure Red Hat OpenShift (АТО). Эта группа ресурсов будет содержать общие папки службы файлов Azure, созданные динамическим средством подготовки Azure Red Hat OpenShift.

```bash
AZURE_FILES_RESOURCE_GROUP=aro_azure_files
LOCATION=eastus

az group create -l $LOCATION -n $AZURE_FILES_RESOURCE_GROUP

AZURE_STORAGE_ACCOUNT_NAME=aroazurefilessa

az storage account create \
    --name $AZURE_STORAGE_ACCOUNT_NAME \
    --resource-group $AZURE_FILES_RESOURCE_GROUP \
    --kind StorageV2 \
    --sku Standard_LRS
```

## <a name="set-permissions"></a>Настройка разрешений
### <a name="set-resource-group-permissions"></a>Задание разрешений для группы ресурсов

Субъекту-службе АТО требуется разрешение "listKeys" в новой группе ресурсов учетной записи хранения Azure. Чтобы добиться этого, назначьте роль "участник".

```bash
ARO_RESOURCE_GROUP=aro-rg
CLUSTER=cluster
ARO_SERVICE_PRINCIPAL_ID=$(az aro show -g $ARO_RESOURCE_GROUP -n $CLUSTER --query servicePrincipalProfile.clientId -o tsv)

az role assignment create --role Contributor --assignee $ARO_SERVICE_PRINCIPAL_ID -g $AZURE_FILES_RESOURCE_GROUP
```

### <a name="set-aro-cluster-permissions"></a>Задать разрешения для кластера АТО

Учетной записи службы привязки постоянного тома OpenShift потребуется возможность чтения секретов. Чтобы добиться этого, создайте и назначьте роль кластера OpenShift.
```bash
ARO_API_SERVER=$(az aro list --query "[?contains(name,'$CLUSTER')].[apiserverProfile.url]" -o tsv)

oc login -u kubeadmin -p $(az aro list-credentials -g $ARO_RESOURCE_GROUP -n $CLUSTER --query=kubeadminPassword -o tsv) $APISERVER

oc create clusterrole azure-secret-reader \
    --verb=create,get \
    --resource=secrets

oc adm policy add-cluster-role-to-user azure-secret-reader system:serviceaccount:kube-system:persistent-volume-binder
```

## <a name="create-storageclass-with-azure-files-provisioner"></a>Создание Сторажекласс с помощью службы "Подготовка файлов Azure"

На этом шаге мы создадим Сторажекласс с помощью подготовки файлов Azure. В манифесте Сторажекласс сведения о учетной записи хранения необходимы, чтобы кластер АТО знал о том, что учетная запись хранения находится за пределами текущей группы ресурсов.

Во время подготовки хранилища для подключения учетных данных создается секрет с именем secretName. В контексте с несколькими арендами настоятельно рекомендуется явно задать значение для Секретнамеспаце, в противном случае учетные данные учетной записи хранения могут быть прочитаны другими пользователями.

```bash
cat << EOF >> azure-storageclass-azure-file.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azure-file
provisioner: kubernetes.io/azure-file
parameters:
  location: $LOCATION
  secretNamespace: kube-system
  skuName: Standard_LRS
  storageAccount: $AZURE_STORAGE_ACCOUNT_NAME
  resourceGroup: $AZURE_FILES_RESOURCE_GROUP
reclaimPolicy: Delete
volumeBindingMode: Immediate
EOF

oc create -f azure-storageclass-azure-file.yaml
```

## <a name="change-the-default-storageclass-optional"></a>Изменение Сторажекласс по умолчанию (необязательно)

Сторажекласс по умолчанию для АТО называется Managed-Premium и использует службу подготовки диска Azure. Измените это, выполнив команды patch для манифестов Сторажекласс.

```bash
oc patch storageclass managed-premium -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

oc patch storageclass azure-file -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## <a name="verify-azure-file-storageclass-optional"></a>Проверка Сторажекласс файлов Azure (необязательно)

Создайте новое приложение и назначьте ему хранилище.

```bash
oc new-project azfiletest
oc new-app -template httpd-example

#Wait for the pod to become Ready
curl $(oc get route httpd-example -n azfiletest -o jsonpath={.spec.host})

oc set volume dc/httpd-example --add --name=v1 -t pvc --claim-size=1G -m /data

#Wait for the new deployment to rollout
export POD=$(oc get pods --field-selector=status.phase==Running -o jsonpath={.items[].metadata.name})
oc exec $POD -- bash -c "mkdir ./data"
oc exec $POD -- bash -c "echo 'azure file storage' >> /data/test.txt"

oc exec $POD -- bash -c "cat /data/test.txt"
azure file storage
```
Файл test.txt будет также виден через Обозреватель службы хранилища в портал Azure.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы создали динамическое постоянное хранилище с помощью файлов Microsoft Azure и Azure Red Hat OpenShift 4. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание учетной записи хранения
> * Настройка Сторажекласс в кластере Azure Red Hat OpenShift 4 с помощью средства подготовки файлов Azure

Перейдите к следующей статье, чтобы узнать о поддерживаемых ресурсах Azure Red Hat OpenShift 4.

* [Политика поддержки Azure Red Hat OpenShift](support-policies-v4.md)
