---
title: Модули бицеп
description: Описывает определение и использование модуля, а затем способы использования областей модуля.
ms.topic: conceptual
ms.date: 03/17/2021
ms.openlocfilehash: 2edeb5c96f771867f964963b2d27768291ae2d4a
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889711"
---
# <a name="use-bicep-modules"></a>Использование модулей Бицеп

Бицеп позволяет разбить сложное решение на модули. Модуль Бицеп — это набор из одного или нескольких ресурсов, которые должны быть развернуты вместе. Модули не являются сложными сведениями об объявлении необработанного ресурса, что может повысить удобочитаемость. Вы можете повторно использовать эти модули и поделиться ими с другими людьми. В сочетании с [спецификациями шаблонов](./template-specs.md)создается способ повторного использования модульных и программных кодов. Руководство см. в разделе [учебник. Добавление модулей бицеп](./bicep-tutorial-add-modules.md).

## <a name="define-modules"></a>Определение модулей

Каждый файл Бицеп можно использовать как модуль. Модуль предоставляет доступ к параметрам и выходам только в качестве контракта к другим Бицеп файлам. Параметры и выходные данные являются необязательными.

Следующий файл Бицеп можно развернуть напрямую, чтобы создать учетную запись хранения или использовать в качестве модуля.  В следующем разделе показано, как использовать модули:

```bicep
@minLength(3)
@maxLength(11)
param storagePrefix string

@allowed([
  'Standard_LRS'
  'Standard_GRS'
  'Standard_RAGRS'
  'Standard_ZRS'
  'Premium_LRS'
  'Premium_ZRS'
  'Standard_GZRS'
  'Standard_RAGZRS'
])
param storageSKU string = 'Standard_LRS'
param location string

var uniqueStorageName = '${storagePrefix}${uniqueString(resourceGroup().id)}'

resource stg 'Microsoft.Storage/storageAccounts@2019-04-01' = {
  name: uniqueStorageName
  location: location
  sku: {
    name: storageSKU
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
  }
}

output storageEndpoint object = stg.properties.primaryEndpoints
```

OUTPUT используется для передачи значений в родительские файлы Бицеп.

## <a name="consume-modules"></a>Использование модулей

Используйте ключевое слово _module_ для использования модуля. Следующий файл Бицеп развертывает ресурс, определенный в файле модуля, на который указывает ссылка:

```bicep
@minLength(3)
@maxLength(11)
param namePrefix string
param location string = resourceGroup().location

module stgModule './storageAccount.bicep' = {
  name: 'storageDeploy'
  params: {
    storagePrefix: namePrefix
    location: location
  }
}

output storageEndpoint object = stgModule.outputs.storageEndpoint
```

- **module**. Ключевое слово.
- **символьное имя** (стгмодуле): идентификатор модуля.
- **файл модуля**: путь к модулю в этом примере указывается с помощью относительного пути (./сторажеаккаунт.бицеп). Все пути в Bicep должны быть указаны с помощью разделителя каталогов косой черты (/), чтобы обеспечить последовательную компиляцию на разных платформах. Символ обратной косой черты Windows ( \\ ) не поддерживается.
- Свойство **_Name_** (сторажедеплой) является обязательным при использовании модуля. Когда Бицеп создает шаблон IL, это поле используется как имя вложенного ресурса развертывания, созданного для модуля:

    ```json
    ...
    ...
    "resources": [
      {
        "type": "Microsoft.Resources/deployments",
        "apiVersion": "2020-10-01",
        "name": "storageDeploy",
        "properties": {
          ...
        }
      }
    ]
    ...
    ```

Чтобы получить выходное значение из модуля, извлеките значение свойства с помощью синтаксиса, например, `stgModule.outputs.storageEndpoint` где `stgModule` — это идентификатор модуля.

## <a name="configure-module-scopes"></a>Настройка областей модуля

При объявлении модуля можно указать свойство _Scope_ , чтобы задать область, в которой будет развернут модуль:

```bicep
module stgModule './storageAccount.bicep' = {
  name: 'storageDeploy'
  scope: resourceGroup('someOtherRg') // pass in a scope to a different resourceGroup
  params: {
    storagePrefix: namePrefix
    location: location
  }
}
```

Свойство _Scope_ можно опустить, если целевая область модуля и Целевая область родителя совпадают. Если свойство Scope не указано, модуль развертывается в целевой области родителя.

В следующем Бицеп-файле показано, как создать группу ресурсов и развернуть модуль в группе ресурсов.

```bicep
// set the target scope for this file
targetScope = 'subscription'

@minLength(3)
@maxLength(11)
param namePrefix string

param location string = deployment().location

var resourceGroupName = '${namePrefix}rg'
resource myResourceGroup 'Microsoft.Resources/resourceGroups@2020-01-01' = {
  name: resourceGroupName
  location: location
  scope: subscription()
}

module stgModule './storageAccount.bicep' = {
  name: 'storageDeploy'
  scope: myResourceGroup
  params: {
    storagePrefix: namePrefix
    location: location
  }
}

output storageEndpoint object = stgModule.outputs.storageEndpoint
```

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы пройти обучение, см. раздел [учебник. Добавление модулей бицеп](./bicep-tutorial-add-modules.md).
