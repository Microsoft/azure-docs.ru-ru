---
title: Подключение тома файлов Azure к группе контейнеров
description: Узнайте, как подключить том файлов Azure для сохранения состояния с помощью Экземпляров контейнеров Azure
ms.topic: article
ms.date: 07/02/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d52ad8ad02735c98b29a83d8ca69cdea8c6af7d8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97954980"
---
# <a name="mount-an-azure-file-share-in-azure-container-instances"></a>Подключение общего файлового ресурса Azure с помощью службы "Экземпляры контейнеров Azure"

Состояние в экземплярах контейнеров Azure не отслеживается по умолчанию. Если контейнер перезапускается, завершается со сбоем или останавливается, все его состояния теряются. Чтобы сохранить состояние после истечения времени существования контейнера, необходимо подключить том из внешнего хранилища. Как показано в этой статье, служба "экземпляры контейнеров Azure" может подключать файловый ресурс Azure, созданный с помощью службы [файлов Azure](../storage/files/storage-files-introduction.md). Служба файлов Azure предоставляет полностью управляемые общие файловые ресурсы, размещенные в службе хранилища Azure, которые доступны через стандартный промышленный протокол SMB. Использование файлового ресурса Azure с Экземплярами контейнеров Azure предоставляет функции обмена файлами, похожие на использование файлового ресурса Azure совместно с виртуальными машинами Azure.

> [!NOTE]
> Подключение общего ресурса службы файлов Azure сейчас поддерживается только для контейнеров Linux. Найдите текущие различия в платформе в [обзоре](container-instances-overview.md#linux-and-windows-containers).
>
> Подключение файлового ресурса Azure к экземпляру контейнера аналогично [подключению DOCKER BIND](https://docs.docker.com/storage/bind-mounts/). Имейте в виду, что при подключении общей папки к каталогу контейнера, в котором находятся файлы или каталоги, эти файлы или каталоги закрываются подключением и недоступны во время выполнения контейнера.
>

> [!IMPORTANT]
> При развертывании групп контейнеров в виртуальной сети Azure необходимо добавить [конечную точку службы](../virtual-network/virtual-network-service-endpoints-overview.md) в учетную запись хранения Azure.

## <a name="create-an-azure-file-share"></a>Создание файлового ресурса Azure

Перед использованием файлового ресурса Azure с Экземплярами контейнеров Azure необходимо создать его. Выполните приведенный ниже скрипт, чтобы создать учетную запись хранения для размещения файлового ресурса и общего ресурса. Имя учетной записи хранения должно быть глобально уникальным, поэтому скрипт добавляет случайное значение к базовой строке.

```azurecli-interactive
# Change these four parameters as needed
ACI_PERS_RESOURCE_GROUP=myResourceGroup
ACI_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
ACI_PERS_LOCATION=eastus
ACI_PERS_SHARE_NAME=acishare

# Create the storage account with the parameters
az storage account create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --location $ACI_PERS_LOCATION \
    --sku Standard_LRS

# Create the file share
az storage share create \
  --name $ACI_PERS_SHARE_NAME \
  --account-name $ACI_PERS_STORAGE_ACCOUNT_NAME
```

## <a name="get-storage-credentials"></a>Получение учетных данных хранилища

Для подключения файлового ресурса Azure в качестве тома в Экземплярах контейнеров Azure требуется три значения: имя учетной записи хранения, имя общего ресурса и ключ доступа к хранилищу.

* **Имя учетной записи хранения** . Если вы использовали предыдущий скрипт, имя учетной записи хранения было сохранено в `$ACI_PERS_STORAGE_ACCOUNT_NAME` переменной. Для отображения имени учетной записи введите:

  ```console
  echo $ACI_PERS_STORAGE_ACCOUNT_NAME
  ```

* **Имя общего ресурса** — это значение уже известно (определено как `acishare` в предыдущем сценарии).

* **Ключ учетной записи хранения** . это значение можно найти с помощью следующей команды:

  ```azurecli-interactive
  STORAGE_KEY=$(az storage account keys list --resource-group $ACI_PERS_RESOURCE_GROUP --account-name $ACI_PERS_STORAGE_ACCOUNT_NAME --query "[0].value" --output tsv)
  echo $STORAGE_KEY
  ```

## <a name="deploy-container-and-mount-volume---cli"></a>Развертывание контейнера и подключение тома — CLI

Чтобы подключить общую папку Azure как том в контейнере с помощью Azure CLI, укажите общую папку и точку подключения тома при создании контейнера с помощью команды [AZ Container Create][az-container-create]. Если вы проработали предыдущие шаги, можно подключить созданную ранее общую папку, выполнив следующую команду, чтобы создать контейнер:

```azurecli-interactive
az container create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label aci-demo \
    --ports 80 \
    --azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --azure-file-volume-account-key $STORAGE_KEY \
    --azure-file-volume-share-name $ACI_PERS_SHARE_NAME \
    --azure-file-volume-mount-path /aci/logs/
```

`--dns-name-label`Значение должно быть уникальным в пределах региона Azure, в котором создается экземпляр контейнера. Если при выполнении команды появится сообщение об ошибке **Метка DNS-имени**, обновите значение в предыдущей команде.

## <a name="manage-files-in-mounted-volume"></a>Управление файлами в подключенном томе

После запуска контейнера можно использовать простое веб-приложение, развернутое с помощью образа Microsoft [ACI-hellofiles][aci-hellofiles] , чтобы создать небольшие текстовые файлы в общей папке Azure по указанному пути подключения. Получите полное доменное имя (FQDN) веб-приложения с помощью команды [az container show][az-container-show].

```azurecli-interactive
az container show --resource-group $ACI_PERS_RESOURCE_GROUP \
  --name hellofiles --query ipAddress.fqdn --output tsv
```

После сохранения текста с помощью приложения можно использовать [портал Azure][portal] или средство, такое как [Обозреватель службы хранилища Microsoft Azure][storage-explorer] , для получения и проверки файла или файлов, записанных в общую папку.

## <a name="deploy-container-and-mount-volume---yaml"></a>Развертывание контейнера и подключение тома — YAML

Вы также можете развернуть группу контейнеров и подключить том в контейнере с помощью Azure CLI и [шаблона YAML](container-instances-multi-container-yaml.md). Развертывание с помощью шаблона YAML является предпочтительным методом при развертывании групп контейнеров, состоящих из нескольких контейнеров.

Следующий шаблон YAML определяет группу контейнеров с одним контейнером, созданным с помощью `aci-hellofiles` образа. Контейнер подключает файловый ресурс Azure *acishare* , созданный ранее в качестве тома. Где указано, введите имя и ключ хранилища для учетной записи хранения, в которой размещена общая папка. 

Как и в примере CLI, `dnsNameLabel` значение должно быть уникальным в пределах региона Azure, в котором создается экземпляр контейнера. При необходимости обновите значение в файле YAML.

```yaml
apiVersion: '2019-12-01'
location: eastus
name: file-share-demo
properties:
  containers:
  - name: hellofiles
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-hellofiles
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /aci/logs/
        name: filesharevolume
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
      - port: 80
    dnsNameLabel: aci-demo
  volumes:
  - name: filesharevolume
    azureFile:
      sharename: acishare
      storageAccountName: <Storage account name>
      storageAccountKey: <Storage account key>
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

Для развертывания с помощью шаблона YAML сохраните предыдущий YAML в файл с именем `deploy-aci.yaml`, а затем выполните команду [az container create][az-container-create] с параметром `--file`:

```azurecli
# Deploy with YAML template
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```
## <a name="deploy-container-and-mount-volume---resource-manager"></a>Развертывание контейнера и подключение тома — диспетчер ресурсов

В дополнение к развертыванию CLI и YAML можно развернуть группу контейнеров и подключить том в контейнере с помощью [шаблона Azure диспетчер ресурсов](/azure/templates/microsoft.containerinstance/containergroups).

Сначала заполните массив `volumes` в разделе `properties` группы контейнеров шаблона. 

Затем для каждого контейнера, в котором вы хотите подключить том, заполните `volumeMounts` массив в `properties` разделе определения контейнера.

Следующий шаблон диспетчер ресурсов определяет группу контейнеров с одним контейнером, созданным с помощью `aci-hellofiles` образа. Контейнер подключает файловый ресурс Azure *acishare* , созданный ранее в качестве тома. Где указано, введите имя и ключ хранилища для учетной записи хранения, в которой размещена общая папка. 

Как и в предыдущих примерах, `dnsNameLabel` значение должно быть уникальным в пределах региона Azure, в котором создается экземпляр контейнера. При необходимости обновите значение в шаблоне.

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "variables": {
    "container1name": "hellofiles",
    "container1image": "mcr.microsoft.com/azuredocs/aci-hellofiles"
  },
  "resources": [
    {
      "name": "file-share-demo",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "containers": [
          {
            "name": "[variables('container1name')]",
            "properties": {
              "image": "[variables('container1image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "/aci/logs"
                }
              ]
            }
          }
        ],
        "osType": "Linux",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "tcp",
              "port": "80"
            }
          ],
          "dnsNameLabel": "aci-demo"
        },
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
                "shareName": "acishare",
                "storageAccountName": "<Storage account name>",
                "storageAccountKey": "<Storage account key>"
            }
          }
        ]
      }
    }
  ]
}
```

Чтобы выполнить развертывание с помощью шаблона диспетчер ресурсов, сохраните предыдущий JSON в файл с именем `deploy-aci.json` , а затем выполните команду [AZ Deploy Group Create][az-deployment-group-create] с `--template-file` параметром:

```azurecli
# Deploy with Resource Manager template
az deployment group create --resource-group myResourceGroup --template-file deploy-aci.json
```


## <a name="mount-multiple-volumes"></a>Подключение нескольких томов

Для подключения нескольких томов в экземпляре контейнера необходимо выполнить развертывание с помощью [шаблона Azure Resource Manager](/azure/templates/microsoft.containerinstance/containergroups), файла YAML или другого программного метода. Чтобы использовать шаблон или файл YAML, укажите сведения об общем ресурсе и определите тома, заполнив `volumes` массив в `properties` разделе файла. 

Например, если вы создали две общие папки службы файлов Azure с именами *share1* и *share2* в учетной записи хранения *myStorageAccount*, `volumes` массив в шаблоне диспетчер ресурсов будет выглядеть следующим образом:

```JSON
"volumes": [{
  "name": "myvolume1",
  "azureFile": {
    "shareName": "share1",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
},
{
  "name": "myvolume2",
  "azureFile": {
    "shareName": "share2",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
}]
```

Затем для каждого контейнера в группе контейнеров, в которой нужно подключить тома, заполните массив `volumeMounts` в разделе `properties` определения контейнера. В результате этого, подключатся, например, тома *myvolume1* и *myvolume2*, определенные ранее.

```JSON
"volumeMounts": [{
  "name": "myvolume1",
  "mountPath": "/mnt/share1/"
},
{
  "name": "myvolume2",
  "mountPath": "/mnt/share2/"
}]
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о подключении других типов томов в службе "Экземпляры контейнеров Azure" см. в следующих статьях:

* [Подключение тома emptyDir в службе "Экземпляры контейнеров Azure"](container-instances-volume-emptydir.md)
* [Подключение тома gitRepo в службе "Экземпляры контейнеров Azure"](container-instances-volume-gitrepo.md)
* [Подключение тома secret в службе "Экземпляры контейнеров Azure"](container-instances-volume-secret.md)

<!-- LINKS - External -->
[aci-hellofiles]: https://hub.docker.com/_/microsoft-azuredocs-aci-hellofiles 
[portal]: https://portal.azure.com
[storage-explorer]: https://storageexplorer.com

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
