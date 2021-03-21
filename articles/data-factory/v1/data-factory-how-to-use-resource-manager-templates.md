---
title: Использование шаблонов диспетчер ресурсов в фабрике данных
description: Узнайте, как создать шаблоны Azure Resource Manager и использовать их для создания сущностей фабрики данных.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: c17ef939b8d27035e5b28f4a651b9be2db8f2279
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100382945"
---
# <a name="use-templates-to-create-azure-data-factory-entities"></a>Создание сущностей фабрики данных Azure с помощью шаблонов
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. 

## <a name="overview"></a>Обзор
Пользуясь фабрикой данных Azure для интеграции данных, возможно, вы захотите повторно использовать один и тот же шаблон в разных средах или несколько раз реализовать одну задачу в том же самом решении. Шаблоны помогают реализовать эти сценарии и с легкостью управлять ими. Шаблоны в фабрике данных Azure идеально подходят для сценариев, включающих возможность многократного использования и повторения.

Рассмотрим ситуацию, когда организация имеет 10 производственных фабрик по всему миру. Журналы каждого завода хранятся в отдельной SQL Server базе данных. Компании требуется создать единое хранилище данных в облаке для автоматизированной аналитики. Требуется также использовать одну логику и разные конфигурации для сред разработки, тестирования и рабочих сред.

В этом случае в пределах одной среды задача выполняется несколько раз. Однако при этом для каждой производственной фабрики следует использовать разные значения в 10 фабриках данных. По факту **повторение** реализовано. Использование шаблонов (то есть одинаковых действий в конвейерах в каждой фабрике данных) позволяет абстрагировать этот общий процесс. Однако при этом для каждой производственной фабрики используется отдельный файл параметров.

Кроме того, так как организации нужно развернуть эти 10 фабрик данных несколько раз в различных средах, шаблоны могут получить эту **возможность многократного использования** за счет использования отдельных файлов параметров для сред разработки, тестирования и рабочих сред.

## <a name="templating-with-azure-resource-manager"></a>Использование шаблонов с помощью Azure Resource Manager
[Шаблоны Azure Resource Manager](../../azure-resource-manager/templates/overview.md) — отличный способ реализации возможности использования шаблонов в фабрике данных Azure. Они определяют инфраструктуру и конфигурацию решения Azure в JSON-файле. Так как шаблоны Azure Resource Manager совместимы почти со всеми службами Azure, с их помощью можно упростить управление всеми ресурсами Azure. Общие сведения о шаблонах Azure Resource Manager см. в статье [Создание шаблонов диспетчера ресурсов Azure](../../azure-resource-manager/templates/template-syntax.md).

## <a name="tutorials"></a>Учебники
Пошаговые инструкции по созданию сущностей фабрики данных с помощью шаблонов Resource Manager см. в следующих руководствах:

* [Tutorial: Create a pipeline to copy data by using Azure Resource Manager template](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md) (Руководство. Создание конвейера для копирования данных с помощью шаблона Azure Resource Manager)
* [Tutorial: Create a pipeline to process data by using Azure Resource Manager template](data-factory-build-your-first-pipeline.md) (Руководство. Создание конвейера для обработки данных с помощью шаблона Azure Resource Manager)

## <a name="data-factory-templates-on-github"></a>Шаблоны фабрики данных на сайте GitHub
Ознакомьтесь со следующими шаблонами быстрого запуска Azure на сайте GitHub:

* [Create a Data factory to copy data from Azure Blob Storage to Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-blob-to-sql-copy) (Создание фабрики данных для копирования данных из хранилища BLOB-объектов Azure в Базу данных SQL Azure)
* [Create a Data factory with Hive activity on Azure HDInsight cluster](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-hive-transformation) (Создание фабрики данных с действием Hive в кластере Azure HDInsight)
* [Create a Data factory to copy data from Salesforce to Azure Blobs](https://github.com/Azure/azure-quickstart-templates/tree/master/101-data-factory-salesforce-to-blob-copy) (Создание фабрики данных для копирования данных из Salesforce в большие двоичные объекты Azure)
* [Создание фабрики данных, которая связывает действия в цепочку: копирует данные из FTP-сервера в большие двоичные объекты Azure, вызывает скрипт Hive в кластере HDInsight по запросу для преобразования данных и копирует результаты в базу данных SQL Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/201-data-factory-ftp-hive-blob)

Вы можете поделиться шаблонами фабрики данных Azure в [кратком руководстве Azure](https://azure.microsoft.com/documentation/templates/). Если вы разрабатываете шаблоны, к которым можно предоставить общий доступ в этом репозитории, см. [руководство по добавлению](https://github.com/Azure/azure-quickstart-templates/tree/master/1-CONTRIBUTION-GUIDE).

В следующих разделах представлены сведения об определении ресурсов фабрики данных в шаблоне Resource Manager.

## <a name="defining-data-factory-resources-in-templates"></a>Определение ресурсов фабрики данных в шаблонах
Шаблон верхнего уровня для определения фабрики данных выглядит следующим образом:

```JSON
"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
"contentVersion": "1.0.0.0",
"parameters": { ...
},
"variables": { ...
},
"resources": [
{
    "name": "[parameters('dataFactoryName')]",
    "apiVersion": "[variables('apiVersion')]",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "westus",
    "resources": [
    { "type": "linkedservices",
        ...
    },
    {"type": "datasets",
        ...
    },
    {"type": "dataPipelines",
        ...
    }
}
```

### <a name="define-data-factory"></a>Определение фабрики данных
Фабрику данных следует определить в шаблоне Resource Manager, как показано в следующем примере:

```JSON
"resources": [
{
    "name": "[variables('<mydataFactoryName>')]",
    "apiVersion": "2015-10-01",
    "type": "Microsoft.DataFactory/datafactories",
    "location": "East US"
}
```
Параметр dataFactoryName определяется в разделе переменных следующим образом:

```JSON
"dataFactoryName": "[concat('<myDataFactoryName>', uniqueString(resourceGroup().id))]",
```

### <a name="define-linked-services"></a>Определение связанных служб

```JSON
"type": "linkedservices",
"name": "[variables('<LinkedServiceName>')]",
"apiVersion": "2015-10-01",
"dependsOn": [ "[variables('<dataFactoryName>')]" ],
"properties": {
    ...
}
```

Дополнительные сведения о свойствах JSON для конкретной связанной службы, которую нужно развернуть, см. в разделе [Связанная служба хранилища](data-factory-azure-blob-connector.md#azure-storage-linked-service) или [Связанные службы вычислений](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Параметр dependsOn указывает имя соответствующей фабрики данных. Пример определения связанной службы для службы хранилища Azure показан в следующем определении JSON:

### <a name="define-datasets"></a>Определение наборов данных

```JSON
"type": "datasets",
"name": "[variables('<myDatasetName>')]",
"dependsOn": [
    "[variables('<dataFactoryName>')]",
    "[variables('<myDatasetLinkedServiceName>')]"
],
"apiVersion": "2015-10-01",
"properties": {
    ...
}
```
Дополнительные сведения о свойствах JSON для конкретного типа набора данных, который нужно развернуть, см. в разделе [Поддерживаемые хранилища данных](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Обратите внимание, что параметр dependsOn указывает имя соответствующей фабрики данных и связанной службы хранилища. Пример определения типа набора данных хранилища BLOB-объектов Azure показан в следующем определении JSON:

```JSON
"type": "datasets",
"name": "[variables('storageDataset')]",
"dependsOn": [
    "[variables('dataFactoryName')]",
    "[variables('storageLinkedServiceName')]"
],
"apiVersion": "2015-10-01",
"properties": {
"type": "AzureBlob",
"linkedServiceName": "[variables('storageLinkedServiceName')]",
"typeProperties": {
    "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
    "fileName": "[parameters('sourceBlobName')]",
    "format": {
        "type": "TextFormat"
    }
},
"availability": {
    "frequency": "Hour",
    "interval": 1
}
```

### <a name="define-pipelines"></a>Определение конвейеров

```JSON
"type": "dataPipelines",
"name": "[variables('<mypipelineName>')]",
"dependsOn": [
    "[variables('<dataFactoryName>')]",
    "[variables('<inputDatasetLinkedServiceName>')]",
    "[variables('<outputDatasetLinkedServiceName>')]",
    "[variables('<inputDataset>')]",
    "[variables('<outputDataset>')]"
],
"apiVersion": "2015-10-01",
"properties": {
    activities: {
        ...
    }
}
```

Дополнительные сведения о свойствах JSON для определения конкретного конвейера и действий, которые нужно развернуть, см. в разделе [Определение конвейеров](data-factory-create-pipelines.md#pipeline-json). Обратите внимание, что параметр dependsOn указывает имя фабрики данных, а также все соответствующие связанные службы или наборы данных. В следующем фрагменте кода JSON показан пример конвейера, копирующего данные из хранилища BLOB-объектов Azure в Базу данных SQL Azure:

```JSON
"type": "datapipelines",
"name": "[variables('pipelineName')]",
"dependsOn": [
    "[variables('dataFactoryName')]",
    "[variables('azureStorageLinkedServiceName')]",
    "[variables('azureSqlLinkedServiceName')]",
    "[variables('blobInputDatasetName')]",
    "[variables('sqlOutputDatasetName')]"
],
"apiVersion": "2015-10-01",
"properties": {
    "activities": [
    {
        "name": "CopyFromAzureBlobToAzureSQL",
        "description": "Copy data frm Azure blob to Azure SQL",
        "type": "Copy",
        "inputs": [
            {
                "name": "[variables('blobInputDatasetName')]"
            }
        ],
        "outputs": [
            {
                "name": "[variables('sqlOutputDatasetName')]"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "BlobSource"
            },
            "sink": {
                "type": "SqlSink",
                "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM {0}', 'emp')"
            },
            "translator": {
                "type": "TabularTranslator",
                "columnMappings": "Column0:FirstName,Column1:LastName"
            }
        },
        "Policy": {
            "concurrency": 1,
            "executionPriorityOrder": "NewestFirst",
            "retry": 3,
            "timeout": "01:00:00"
        }
    }
    ],
    "start": "2016-10-03T00:00:00Z",
    "end": "2016-10-04T00:00:00Z"
}
```
## <a name="parameterizing-data-factory-template"></a>Параметризация шаблона фабрики данных
Рекомендации по параметризации см. в статье [Описание структуры и синтаксиса шаблонов Azure Resource Manager](../../azure-resource-manager/templates/template-best-practices.md). Как правило, следует использовать как можно меньше параметров, особенно если вместо них можно использовать переменные. Предоставляйте параметры только в следующих сценариях:

* параметры различаются в зависимости от среды (например, среда разработки, тестирования и рабочая среда);
* секретов (например, паролей);

Если при развертывании сущностей фабрики данных Azure с помощью шаблонов секреты необходимо извлечь из [хранилища ключей Azure](../../key-vault/general/overview.md), укажите **хранилище ключей** и **секретное имя**, как показано в следующем примере:

```JSON
"parameters": {
    "storageAccountKey": {
        "reference": {
            "keyVault": {
                "id":"/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.KeyVault/vaults/<keyVaultName>",
             },
            "secretName": "<secretName>"
           },
       },
       ...
}
```

> [!NOTE]
> Сейчас экспорт шаблонов для существующих фабрик данных не поддерживается. Однако планируется реализовать такую возможность.
>
>