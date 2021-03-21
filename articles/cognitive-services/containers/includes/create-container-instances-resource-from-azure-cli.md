---
title: Поддержка контейнеров
titleSuffix: Azure Cognitive Services
description: Узнайте, как создать ресурс экземпляра контейнера Azure из Azure CLI.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 87007d3df3fe44ab04a330b09b8e495ec4b47e54
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97866026"
---
## <a name="create-an-azure-container-instance-resource-from-the-azure-cli"></a>Создание ресурса экземпляра контейнера Azure из Azure CLI

YAML ниже определяет ресурс экземпляра контейнера Azure. Скопируйте и вставьте содержимое в новый файл с именем `my-aci.yaml` и замените значения комментария собственными. См. [Формат шаблона][template-format] для допустимого YAML. Сведения о доступных именах образов и соответствующих репозиториях см. в разделе [репозитории и образы контейнеров][repositories-and-images] . Дополнительные сведения о справочнике по YAML для экземпляров контейнеров см. в разделе [Справочник по YAML: экземпляры контейнеров Azure][aci-yaml-ref].

```YAML
apiVersion: 2018-10-01
location: # < Valid location >
name: # < Container Group name >
properties:
  imageRegistryCredentials: # This is only required if you are pulling a non-public image that requires authentication to access. For example Text Analytics for health.
  - server: containerpreview.azurecr.io
    username: # < The username for the preview container registry >
    password: # < The password for the preview container registry >
  containers:
  - name: # < Container name >
    properties:
      image: # < Repository/Image name >
      environmentVariables: # These env vars are required
        - name: eula
          value: accept
        - name: billing
          value: # < Service specific Endpoint URL >
        - name: apikey
          value: # < Service specific API key >
      resources:
        requests:
          cpu: 4 # Always refer to recommended minimal resources
          memoryInGb: 8 # Always refer to recommended minimal resources
      ports:
        - port: 5000
  osType: Linux
  volumes: # This node, is only required for container instances that pull their model in at runtime, such as LUIS.
  - name: aci-file-share
    azureFile:
      shareName: # < File share name >
      storageAccountName: # < Storage account name>
      storageAccountKey: # < Storage account key >
  restartPolicy: OnFailure
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 5000
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

> [!NOTE]
> Не все расположения имеют одинаковую доступность ЦП и памяти. Список доступных ресурсов для контейнеров в расположении и ОС см. в таблице " [расположение и ресурсы][location-to-resource] ".

Мы будем полагаться на файл YAML, созданный для [`az container create`][azure-container-create] команды. В Azure CLI выполните `az container create` команду, заменив `<resource-group>` собственными. Кроме того, для защиты значений в развертывании YAML ссылаются на [защищенные значения][secure-values].

```azurecli
az container create -g <resource-group> -f my-aci.yaml
```

Выходные данные команды являются `Running...` допустимыми, после того как выходные данные изменяются на строку JSON, представляющую созданный ресурс ACI. Образ контейнера, скорее всего, будет недоступен в течение определенного времени, но теперь ресурс развернут.

> [!TIP]
> Обратите особое внимание на местоположения общедоступной предварительной версии предложения службы "Поиск в Azure", так как YAML потребуется соответствующим образом скорректировать их в соответствии с расположением.

[azure-container-create]: /cli/azure/container#az-container-create
[template-format]: /azure/templates/Microsoft.ContainerInstance/2018-10-01/containerGroups#template-format
[aci-yaml-ref]: ../../../container-instances/container-instances-reference-yaml.md
[repositories-and-images]: ../container-image-tags.md
[location-to-resource]: ../../../container-instances/container-instances-region-availability.md
[secure-values]: ../../../container-instances/container-instances-environment-variables.md#secure-values
