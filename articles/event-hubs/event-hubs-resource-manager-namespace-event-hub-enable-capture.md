---
title: Создание концентратора событий с активированной функции "Сбор"в Центрах событий Azure | Документация Майкрософт
description: Создание пространства имен Центров событий Azure с одним концентратором событий и включение функции "Сбор" с помощью шаблона Azure Resource Manager.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 17157e05e4ad123ba2bbdffa199c111df9f8912e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100653029"
---
# <a name="create-a-namespace-with-event-hub-and-enable-capture-using-a-template"></a>Создание пространства имен с концентратором событий и включение записи с помощью шаблона

Из этой статьи вы узнаете, как с помощью шаблона Azure Resource Manager создать пространство имен [Центров событий](./event-hubs-about.md) с одним экземпляром концентратора событий и включить для него [функцию "Сбор"](event-hubs-capture-overview.md). Здесь вы узнаете, как определить развертываемые ресурсы и параметры, указываемые при развертывании. Этот шаблон можно использовать для собственных развертываний или настроить его в соответствии с вашими требованиями.

В этой статье также показано, как настроить запись событий в хранилище BLOB-объектов Azure или Azure Data Lake Store в зависимости от выбранного назначения.

Дополнительные сведения о создании шаблонов см. в статье [Создание шаблонов Azure Resource Manager][Authoring Azure Resource Manager templates]. Синтаксис и свойства JSON, используемые в шаблоне, см. в статье о [типах ресурсов Microsoft.EventHub](/azure/templates/microsoft.eventhub/allversions).

Дополнительные сведения о шаблонах и рекомендациях по соглашениям об именовании ресурсов Azure см. в статье [Naming conventions][Azure Resources naming conventions] (Соглашения об именовании).

Полные шаблоны доступны на сайте GitHub по следующим ссылкам:

- [шаблон для развертывания концентратора событий и включения записи в службу хранилища][Event Hub and enable Capture to Storage template]; 
- [шаблон для развертывания концентратора событий и включения записи в Azure Data Lake Store][Event Hub and enable Capture to Azure Data Lake Store template].

> [!NOTE]
> Чтобы узнать о новых шаблонах, посетите коллекцию [Шаблоны быстрого запуска Azure][Azure Quickstart Templates] и выполните поиск по запросу "Центры событий".
> 
> 

## <a name="what-will-you-deploy"></a>Что вы развернете?

С помощью этого шаблона вы развернете пространство имен Центров событий с концентратором событий и включите [функцию "Сбор" в Центрах событий](event-hubs-capture-overview.md). Функция "Сбор" в Центрах событий позволяет автоматически доставлять потоковые данные из Центров событий в хранилище BLOB-объектов Azure или Azure Data Lake Store с указанным интервалом времени или размера. Чтобы включить функцию "Сбор" в Центрах событий для передачи данных в службу хранилища Azure, нажмите эту кнопку:

[![Развертывание в Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture%2Fazuredeploy.json)

Чтобы включить функцию "Сбор" в Центрах событий для передачи данных в Azure Data Lake Store, нажмите эту кнопку:

[![Развертывание в Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture-for-adls%2Fazuredeploy.json)

## <a name="parameters"></a>Параметры

С помощью диспетчера ресурсов Azure можно определить параметры значений, которые должны указываться на этапе развертывания шаблона. В шаблоне есть раздел `Parameters` , содержащий все значения параметров. Для этих значений необходимо определить параметры, которые будут зависеть от развертываемого проекта либо от среды, в которой выполняется развертывание. Не определяйте параметры для значений, которые не меняются. Значение каждого параметра в шаблоне определяет развертываемые ресурсы.

Ниже описаны параметры, которые определяет шаблон.

### <a name="eventhubnamespacename"></a>eventHubNamespaceName

Имя создаваемого пространства имен Центров событий.

```json
"eventHubNamespaceName":{  
     "type":"string",
     "metadata":{  
         "description":"Name of the EventHub namespace"
      }
}
```

### <a name="eventhubname"></a>eventHubName

Имя концентратора событий, создаваемого в пространстве имен Центров событий.

```json
"eventHubName":{  
    "type":"string",
    "metadata":{  
        "description":"Name of the event hub"
    }
}
```

### <a name="messageretentionindays"></a>messageRetentionInDays

Число дней для хранения сообщений в концентраторе событий. 

```json
"messageRetentionInDays":{
    "type":"int",
    "defaultValue": 1,
    "minValue":"1",
    "maxValue":"7",
    "metadata":{
       "description":"How long to retain the data in event hub"
     }
 }
```

### <a name="partitioncount"></a>partitionCount

Число секций, создаваемых в концентраторе событий.

```json
"partitionCount":{
    "type":"int",
    "defaultValue":2,
    "minValue":2,
    "maxValue":32,
    "metadata":{
        "description":"Number of partitions chosen"
    }
 }
```

### <a name="captureenabled"></a>captureEnabled

Позволяет включить запись для концентратора событий.

```json
"captureEnabled":{
    "type":"string",
    "defaultValue":"true",
    "allowedValues": [
    "false",
    "true"],
    "metadata":{
        "description":"Enable or disable the Capture for your event hub"
    }
 }
```
### <a name="captureencodingformat"></a>captureEncodingFormat

Формат кодировки, указываемый для сериализации данных событий.

```json
"captureEncodingFormat":{
    "type":"string",
    "defaultValue":"Avro",
    "allowedValues":[
    "Avro"],
    "metadata":{
        "description":"The encoding format in which Capture serializes the EventData"
    }
}
```

### <a name="capturetime"></a>captureTime

Интервал времени, в пределах которого функция "Сбор" в Центрах событий начинает сбор данных.

```json
"captureTime":{
    "type":"int",
    "defaultValue":300,
    "minValue":60,
    "maxValue":900,
    "metadata":{
         "description":"The time window in seconds for the capture"
    }
}
```

### <a name="capturesize"></a>captureSize
Интервал размера, согласно которому функция записи начинает запись данных.

```json
"captureSize":{
    "type":"int",
    "defaultValue":314572800,
    "minValue":10485760,
    "maxValue":524288000,
    "metadata":{
        "description":"The size window in bytes for capture"
    }
}
```

### <a name="capturenameformat"></a>captureNameFormat

Формат имени, используемый функцией "Сбор" в Центрах событий, для записи файлов Avro. Обратите внимание, что в формате имени для функции записи должны быть поля `{Namespace}`, `{EventHub}`, `{PartitionId}`, `{Year}`, `{Month}`, `{Day}`, `{Hour}`, `{Minute}`, и `{Second}`. Они могут быть размещены в любом порядке, с разделителями или без них.
 
```json
"captureNameFormat": {
      "type": "string",
      "defaultValue": "{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}",
      "metadata": {
        "description": "A Capture Name Format must contain {Namespace}, {EventHub}, {PartitionId}, {Year}, {Month}, {Day}, {Hour}, {Minute} and {Second} fields. These can be arranged in any order with or without delimeters. E.g.  Prod_{EventHub}/{Namespace}\\{PartitionId}_{Year}_{Month}/{Day}/{Hour}/{Minute}/{Second}"
      }
    }
  
```

### <a name="apiversion"></a>версия_API

Версия API шаблона.

```json
 "apiVersion":{  
    "type":"string",
    "defaultValue":"2017-04-01",
    "metadata":{  
        "description":"ApiVersion used by the template"
    }
 }
```

Если в качестве места назначения вы выбрали службу хранилища Azure, используйте следующие параметры.

### <a name="destinationstorageaccountresourceid"></a>destinationStorageAccountResourceId

Функции записи требуется идентификатор ресурса учетной записи хранения Azure для включения записи в нужной учетной записи хранения.

```json
 "destinationStorageAccountResourceId":{
    "type":"string",
    "metadata":{
        "description":"Your existing Storage account resource ID where you want the blobs be captured"
    }
 }
```

### <a name="blobcontainername"></a>blobContainerName

Контейнер больших двоичных объектов, в который необходимо записывать данные события.

```json
 "blobContainerName":{
    "type":"string",
    "metadata":{
        "description":"Your existing storage container in which you want the blobs captured"
    }
}
```

Если в качестве назначения вы выбрали Azure Data Lake Store 1-го поколения, используйте указанные ниже параметры. Необходимо назначить разрешения для пути к хранилищу Data Lake Store, в которое необходимо записывать события. Чтобы задать разрешения, см. раздел [Сбор данных в Azure Data Lake Storage 1-го поколения](event-hubs-capture-enable-through-portal.md#capture-data-to-azure-data-lake-storage-gen-1).

### <a name="subscriptionid"></a>subscriptionId

Идентификатор подписки для пространства имен Центров событий и Azure Data Lake Store. Оба ресурса должны использовать один и тот же идентификатор подписки.

```json
"subscriptionId": {
    "type": "string",
    "metadata": {
        "description": "Subscription ID of both Azure Data Lake Store and Event Hubs namespace"
     }
 }
```

### <a name="datalakeaccountname"></a>dataLakeAccountName

Имя Azure Data Lake Store для записываемых событий.

```json
"dataLakeAccountName": {
    "type": "string",
    "metadata": {
        "description": "Azure Data Lake Store name"
    }
}
```

### <a name="datalakefolderpath"></a>dataLakeFolderPath

Путь к целевой папке для записываемых событий. Это папка в Data Lake Store, в которую будут отправляться события во время операции записи. Чтобы настроить разрешения для этой папки, ознакомьтесь со статьей [Сбор данных из Центров событий с помощью Azure Data Lake Store](../data-lake-store/data-lake-store-archive-eventhub-capture.md).

```json
"dataLakeFolderPath": {
    "type": "string",
    "metadata": {
        "description": "Destination capture folder path"
    }
}
```

## <a name="resources-to-deploy-for-azure-storage-as-destination-to-captured-events"></a>Развертываемые ресурсы для службы хранилища Azure как места назначения для записи событий

Здесь создается пространство имен типа **EventHub** с одним концентратором событий и включается запись в хранилище BLOB-объектов Azure.

```json
"resources":[  
      {  
         "apiVersion":"[variables('ehVersion')]",
         "name":"[parameters('eventHubNamespaceName')]",
         "type":"Microsoft.EventHub/Namespaces",
         "location":"[variables('location')]",
         "sku":{  
            "name":"Standard",
            "tier":"Standard"
         },
         "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('eventHubNamespaceName')]",
      "type": "Microsoft.EventHub/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "isAutoInflateEnabled": "true",
        "maximumThroughputUnits": "7"
      },
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('eventHubName')]",
          "type": "EventHubs",
          "dependsOn": [
            "[concat('Microsoft.EventHub/namespaces/', parameters('eventHubNamespaceName'))]"
          ],
          "properties": {
            "messageRetentionInDays": "[parameters('messageRetentionInDays')]",
            "partitionCount": "[parameters('partitionCount')]",
            "captureDescription": {
              "enabled": "true",
              "skipEmptyArchives": false,
              "encoding": "[parameters('captureEncodingFormat')]",
              "intervalInSeconds": "[parameters('captureTime')]",
              "sizeLimitInBytes": "[parameters('captureSize')]",
              "destination": {
                "name": "EventHubArchive.AzureBlockBlob",
                "properties": {
                  "storageAccountResourceId": "[parameters('destinationStorageAccountResourceId')]",
                  "blobContainer": "[parameters('blobContainerName')]",
                  "archiveNameFormat": "[parameters('captureNameFormat')]"
                }
              }
            }
          }

        }
      ]
    }
  ]
```

## <a name="resources-to-deploy-for-azure-data-lake-store-as-destination"></a>Развертываемые ресурсы для Azure Data Lake Store как места назначения

Здесь создается пространство имен типа **EventHub** с одним концентратором событий и включается запись в Azure Data Lake Store.

```json
 "resources": [
        {
            "apiVersion": "2017-04-01",
            "name": "[parameters('namespaceName')]",
            "type": "Microsoft.EventHub/Namespaces",
            "location": "[variables('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "resources": [
                {
                    "apiVersion": "2017-04-01",
                    "name": "[parameters('eventHubName')]",
                    "type": "EventHubs",
                    "dependsOn": [
                        "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
                    ],
                    "properties": {
                        "path": "[parameters('eventHubName')]",
                        "captureDescription": {
                            "enabled": "true",
                            "skipEmptyArchives": false,
                            "encoding": "[parameters('archiveEncodingFormat')]",
                            "intervalInSeconds": "[parameters('captureTime')]",
                            "sizeLimitInBytes": "[parameters('captureSize')]",
                            "destination": {
                                "name": "EventHubArchive.AzureDataLake",
                                "properties": {
                                    "DataLakeSubscriptionId": "[parameters('subscriptionId')]",
                                    "DataLakeAccountName": "[parameters('dataLakeAccountName')]",
                                    "DataLakeFolderPath": "[parameters('dataLakeFolderPath')]",
                                    "ArchiveNameFormat": "[parameters('captureNameFormat')]"
                                }
                            }
                        }
                    }
                }
            ]
        }
    ]
```

> [!NOTE]
> Вы можете включить или отключить создание пустых файлов за время окна сбора данных, когда события отсутствуют, с помощью свойства **skipEmptyArchives**. 

## <a name="commands-to-run-deployment"></a>Команды для выполнения развертывания

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Разверните шаблон, чтобы включить функцию "Сбор" в Центрах событий для передачи данных в службу хранилища Azure.
 
```powershell
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json
```

Разверните шаблон, чтобы включить функцию "Сбор" в Центрах событий для передачи данных в Azure Data Lake Store.

```powershell
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json
```

## <a name="azure-cli"></a>Azure CLI

Хранилище BLOB-объектов Azure в качестве места назначения:

```azurecli
az deployment group create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json][]
```

Azure Data Lake Store в качестве места назначения:

```azurecli
az deployment group create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json][]
```

## <a name="next-steps"></a>Дальнейшие действия

Функцию "Сбор" в Центрах событий можно включить с помощью [портала Azure](https://portal.azure.com). Дополнительные сведения см. в статье [Включение функции "Сбор" в Центрах событий с помощью портала Azure](event-hubs-capture-enable-through-portal.md).

Дополнительные сведения о Центрах событий см. в следующих источниках:

* [Общие сведения о Центрах событий](./event-hubs-about.md)
* [Создание концентратора событий](event-hubs-create.md)
* [Часто задаваемые вопросы о Центрах событий](event-hubs-faq.md)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/templates/template-syntax.md
[Azure Quickstart Templates]:  https://azure.microsoft.com/documentation/templates/?term=event+hubs
[Azure Resources naming conventions]: /azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging
[Event hub and enable Capture to Storage template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture
[Event hub and enable Capture to Azure Data Lake Store template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture-for-adls
