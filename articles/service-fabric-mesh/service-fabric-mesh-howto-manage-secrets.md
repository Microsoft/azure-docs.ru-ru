---
title: Управление секретами приложения сети Service Fabric Azure
description: Управляйте секретами приложений, чтобы безопасно создать и развернуть приложение "Сетка Service Fabric".
ms.date: 4/2/2019
ms.topic: conceptual
ms.custom: devx-track-azurecli
ms.openlocfilehash: b3be0c2b21c3405f4f42b2ff4d02ca95c78956de
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99626966"
---
# <a name="manage-service-fabric-mesh-application-secrets"></a>Управление секретами приложения "Сетка Service Fabric"

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Служба "Сетка Service Fabric" поддерживает секреты в качестве ресурсов Azure. Секретом в службе "Сетка Service Fabric" может быть любая конфиденциальная текстовая информация, например строки подключения к хранилищу, пароли или другие значения, которые должны хранится и передаваться защищенными. В этой статье показано, как использовать службу Secure Store Service Fabric для развертывания и поддержки секретов.

Секрет приложения в Сетке состоит из следующих компонентов:
* Ресурс **Секреты** — это контейнер, в котором хранятся секреты в текстовом формате. Секреты, находящиеся в пределах ресурса **Секреты**, хранятся и передаются в защищенном виде.
* Один или несколько ресурсов **Секреты** (или Значения), которые хранятся в контейнере ресурса **Секреты**. Каждый ресурс **Секреты** (или Значения) отличается номером версии. Изменить версию ресурса **Секреты/Значения** невозможно, можно только добавить новую версию.

Управление секретами заключается в следующих действиях:
1. Объявите ресурс " **секреты** сети" в YAML или JSON-файле модели ресурсов Azure с помощью определений ContentType инлинедвалуе Kind и секретссторереф.
2. Объявите ресурсы для **секретов или значений** сетки в файле модели ресурсов Azure YAML или JSON, которые будут храниться в **секретном** ресурсе (из шага 1).
3. изменение приложения сетки для ссылки на значения секретов сетки;
4. развертывание или последовательное обновление приложения сетки для использования значений секретов;
5. использование команд Azure CLI "az" для управления жизненным циклом службы Secure Store.

## <a name="declare-a-mesh-secrets-resource"></a>Объявление ресурса секретов сетки
Ресурс секреты сетки объявляется в формате JSON или YAML в модели ресурсов Azure с помощью определения типа Инлинедвалуе. Ресурс секретов сетки поддерживает секреты, поставляемые службой Secure Store. 
>
Ниже представлен пример объявления ресурсов сетки "Секреты" в файле JSON.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "WestUS",
      "metadata": {
        "description": "Location of the resources (e.g. westus, eastus, westeurope)."
      }
    }
  },
  "sfbpHttpsCertificate": {
      "type": "string",
      "metadata": {
        "description": "Plain Text Secret Value that your container ingest"
      }
  },
  "resources": [
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx",
      "type": "Microsoft.ServiceFabricMesh/secrets",
      "location": "[parameters('location')]", 
      "dependsOn": [],
      "properties": {
        "kind": "inlinedValue",
        "description": "SFBP Application Secret",
        "contentType": "text/plain",
      }
    }
  ]
}
```
Ниже представлен пример объявления ресурсов сетки "Секреты" в файле YAML.
```yaml
    services:
      - name: helloWorldService
        properties:
          description: Hello world service.
          osType: linux
          codePackages:
            - name: helloworld
              image: myapp:1.0-alpine
              resources:
                requests:
                  cpu: 2
                  memoryInGB: 2
              endpoints:
                - name: helloWorldEndpoint
                  port: 8080
          secrets:
            - name: MySecret.txt
            description: My Mesh Application Secret
            secret_type: inlinedValue
            content_type: SecretStoreRef
            value: mysecret
    replicaCount: 3
    networkRefs:
      - name: mynetwork
```

## <a name="declare-mesh-secretsvalues-resources"></a>Объявление ресурсов сетки "Секреты/Значения"
Ресурсы сетки "Секреты/Значения" зависят от ресурса сетки "Секреты", определенного на предыдущем шаге.

Относительно связи между полями "value:" и "name:" раздела "resources": во второй части строки "name:", отделенной двоеточием, указан номер версии, используемый для секрета, а имя перед двоеточием должно совпадать со значением секрета сетки, от которого она зависит. Например, для элемента ```name: mysecret:1.0``` номер версии — 1.0, а имя ```mysecret``` должно совпадать с ранее определенным ```"value": "mysecret"```.

>
Ниже представлен пример объявления ресурсов сетки "Секреты/Значения" в файле JSON.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "WestUS",
      "metadata": {
        "description": "Location of the resources (e.g. westus, eastus, westeurope)."
      }
    }
  },
  "sfbpHttpsCertificate": {
      "type": "string",
      "metadata": {
        "description": "Plain Text Secret Value that your container ingest"
      }
  },
  "resources": [
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx",
      "type": "Microsoft.ServiceFabricMesh/secrets",
      "location": "[parameters('location')]", 
      "dependsOn": [],
      "properties": {
        "kind": "inlinedValue",
        "description": "SFBP Application Secret",
        "contentType": "text/plain",
      }
    },
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx/2019.02.28",
      "type": "Microsoft.ServiceFabricMesh/secrets/values",
      "location": "[parameters('location')]",
      "dependsOn": [
        "Microsoft.ServiceFabricMesh/secrets/sfbpHttpsCertificate.pfx"
      ],
      "properties": {
        "value": "[parameters('sfbpHttpsCertificate')]"
      }
    }
  ],
}
```
Ниже представлен пример объявления ресурсов сетки "Секреты/Значения" в файле YAML.
```yaml
    services:
      - name: helloWorldService
        properties:
          description: Hello world service.
          osType: linux
          codePackages:
            - name: helloworld
              image: myapp:1.0-alpine
              resources:
                requests:
                  cpu: 2
                  memoryInGB: 2
              endpoints:
                - name: helloWorldEndpoint
                  port: 8080
          Secrets:
            - name: MySecret.txt
            description: My Mesh Application Secret
            secret_type: inlinedValue
            content_type: SecretStoreRef
            value: mysecret
            - name: mysecret:1.0
            description: My Mesh Application Secret Value
            secret_type: value
            content_type: text/plain
            value: "P@ssw0rd#1234"
    replicaCount: 3
    networkRefs:
      - name: mynetwork
```

## <a name="modify-mesh-application-to-reference-mesh-secret-values"></a>Изменение приложения сетки для ссылки на значения секретов сетки
Приложения сетки Service Fabric должны учитывать две приведенные ниже строки, чтобы использовать значения секретов службы Secure Store.
1. Microsoft.ServiceFabricMesh/Secrets.name содержит имя файла и будет содержать значение секретов в виде открытого текста.
2. Переменная среды "Fabric_SettingPath" в Windows или Linux содержит путь к каталогу, в котором файлы, содержащие значения секретов службы Secure Store, будут доступны. Для приложений сетки, размещенных в Windows — это "C:\Settings", а в Linux — "/var/settings".

## <a name="deploy-or-use-a-rolling-upgrade-for-mesh-application-to-consume-secret-values"></a>Развертывание или последовательное обновление приложения сетки для использования значений секретов
Создание ресурсов "Секреты" и/или версионных ресурсов "Секреты/Значения" ограничено объявленными развертываниями модели ресурсов. Единственный способ создать эти ресурсы — передать файл JSON или YAML модели ресурсов с помощью команды **az mesh deployment**, как показано ниже.

```azurecli-interactive
az mesh deployment create –-<template-file> or --<template-uri>
```

## <a name="azure-cli-commands-for-secure-store-service-lifecycle-management"></a>Команды Azure CLI для управления жизненным циклом службы Secure Store

### <a name="create-a-new-secrets-resource"></a>Создание нового ресурса "Секреты"
```azurecli-interactive
az mesh deployment create –-<template-file> or --<template-uri>
```
Передайте или **template-file**, или **template-uri** (но не оба).

Пример:
- az mesh deployment create — c:\MyMeshTemplates\SecretTemplate1.txt
- AZ сеток развертывание Create--HTTPS: \/ /www.fabrikam.com/MyMeshTemplates/SecretTemplate1.txt

### <a name="show-a-secret"></a>Отображение секрета
Возвращает описание секрета (но не его значение).
```azurecli-interactive
az mesh secret show --Resource-group <myResourceGroup> --secret-name <mySecret>
```

### <a name="delete-a-secret"></a>Удаление секрета

- Удалить секрет невозможно, пока на него ссылается приложение сетки.
- При удалении ресурса "Секреты" удаляются все версии секретов и ресурсов.
  ```azurecli-interactive
  az mesh secret delete --Resource-group <myResourceGroup> --secret-name <mySecret>
  ```

### <a name="list-secrets-in-subscription"></a>Перечисление секретов в подписке
```azurecli-interactive
az mesh secret list
```
### <a name="list-secrets-in-resource-group"></a>Перечисление секретов в группе ресурсов
```azurecli-interactive
az mesh secret list -g <myResourceGroup>
```
### <a name="list-all-versions-of-a-secret"></a>Перечисление всех версий секрета
```azurecli-interactive
az mesh secretvalue list --Resource-group <myResourceGroup> --secret-name <mySecret>
```

### <a name="show-secret-version-value"></a>Отображение значения версии секрета
```azurecli-interactive
az mesh secretvalue show --Resource-group <myResourceGroup> --secret-name <mySecret> --version <N>
```

### <a name="delete-secret-version-value"></a>Удаление значения версии секрета
```azurecli-interactive
az mesh secretvalue delete --Resource-group <myResourceGroup> --secret-name <mySecret> --version <N>
```

## <a name="next-steps"></a>Дальнейшие действия 
Чтобы узнать больше о службе "Сетка Service Fabric", прочитайте этот обзор:
- [Обзор службы "Сетка Service Fabric"](service-fabric-mesh-overview.md)
