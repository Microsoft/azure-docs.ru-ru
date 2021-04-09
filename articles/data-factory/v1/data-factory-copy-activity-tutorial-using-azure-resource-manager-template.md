---
title: 'Руководство по Создание конвейера с помощью шаблона Resource Manager '
description: Работая с этим руководством, вы создадите конвейер фабрики данных Azure с помощью шаблона Azure Resource Manager. Этот конвейер копирует данные из Хранилища BLOB-объектов Azure в Базу данных SQL Azure.
author: linda33wj
ms.service: data-factory
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 18e8965a152deb6393d7e3b63ea22994484e100c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100377250"
---
# <a name="tutorial-use-azure-resource-manager-template-to-create-a-data-factory-pipeline-to-copy-data"></a>Руководство по Создание конвейера фабрики данных для копирования данных с использованием шаблона Azure Resource Manager 
> [!div class="op_single_selector"]
> * [Обзор и предварительные требования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Мастер копирования](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Шаблон Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API для .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, ознакомьтесь с руководством по [применению действия копирования](../quickstart-create-data-factory-dot-net.md). 

В этом руководстве показано, как создать фабрику данных Azure с использованием шаблона Azure Resource Manager. В этом руководстве конвейер данных копирует данные из исходного хранилища данных в целевое. Он не преобразовывает входные данные в выходные. Инструкции по преобразованию данных с помощью Фабрики данных Azure см. в [руководстве по созданию конвейера для преобразования данных с использованием кластера Hadoop](data-factory-build-your-first-pipeline.md).

В этом руководстве описано, как создать конвейер с одним действием — действием копирования. Действие копирования копирует данные из поддерживаемого хранилища данных в поддерживаемое хранилище данных-приемник. Список хранилищ данных, которые поддерживаются в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Это действие выполняется с помощью глобально доступной службы, обеспечивающей безопасное, надежное и масштабируемое копирование данных между разными хранилищами. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

Конвейер может содержать сразу несколько действий. Два действия можно объединить в цепочку (выполнить одно действие вслед за другим), настроив выходной набор данных одного действия как входной набор данных другого действия. Дополнительные сведения см. в разделе [Несколько действий в конвейере](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline). 

> [!NOTE] 
> В этом руководстве конвейер данных копирует данные из исходного хранилища данных в целевое. Инструкции по преобразованию данных с помощью Фабрики данных Azure см. в [руководстве по созданию конвейера для преобразования данных с использованием кластера Hadoop](data-factory-build-your-first-pipeline.md). 

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Прочтите [обзор руководства](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) и выполните **предварительные требования**.
* Чтобы установить последнюю версию Azure PowerShell на локальном компьютере, следуйте инструкциям в статье [Установка и настройка Azure PowerShell](/powershell/azure/) . В этом руководстве PowerShell используется для развертывания сущностей фабрики данных. 
* Сведения о шаблонах Azure Resource Manager см. в [этой статье](../../azure-resource-manager/templates/template-syntax.md) (необязательно).

## <a name="in-this-tutorial"></a>В этом учебнике рассматриваются следующие темы:
В этом руководстве вы создадите фабрику данных со следующими сущностями.

| Сущность | Описание |
| --- | --- |
| Связанная служба хранения Azure |Связывает учетную запись хранения Azure с фабрикой данных. Служба хранилища Azure — источник данных, а База данных SQL Azure — приемник данных для действия копирования, рассматриваемого в руководстве. Эта сущность указывает учетную запись хранения, содержащую входные данные для действия копирования. |
| Связанная служба "База данных SQL Azure" |Связывает нашу базу данных в службе "База данных SQL Azure" с фабрикой данных. Эта сущность указывает базу данных, содержащую выходные данные для действия копирования. |
| Входной набор данных BLOB-объекта Azure |Относится к связанной службе хранилища Azure. Связанная служба указывает на учетную запись хранения Azure, а набор данных большого двоичного объекта Azure указывает имя контейнера, папки и файла в хранилище, содержащем входные данные. |
| Выходной набор данных SQL Azure |Относится к связанной службе SQL Azure. Связанная служба SQL Azure указывает на логический сервер SQL Server, а набор данных SQL Azure указывает имя таблицы, содержащей выходные данные. |
| Конвейер данных |Конвейер содержит одно действие копирования, которое принимает в качестве входных данных набор данных большого двоичного объекта Azure, а в качестве выходных данных — набор данных SQL Azure. Действие копирования копирует данные из большого двоичного объекта Azure в таблицу в Базе данных SQL Azure. |

Фабрика данных может иметь один или несколько конвейеров. Конвейер может содержать одно или несколько действий. Есть два типа действий: [действия перемещения данных](data-factory-data-movement-activities.md) и [действия преобразования данных](data-factory-data-transformation-activities.md). В этом руководстве описывается создание конвейера с одним действием (действием копирования).

![Копирование большого двоичного объекта Azure в Базу данных SQL Azure](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/CopyBlob2SqlDiagram.png) 

В следующем разделе содержится полный шаблон Resource Manager для определения сущностей фабрики данных. Таким образом, вы сможете быстро изучить это руководство и протестировать шаблон. Дополнительные сведения о том, как определяется каждая сущность фабрики данных, см. в разделе [Сущности фабрики данных в шаблоне](#data-factory-entities-in-the-template).

## <a name="data-factory-json-template"></a>Шаблон JSON фабрики данных
Шаблон Resource Manager верхнего уровня для определения фабрики данных выглядит следующим образом: 

```json
{
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
                { ... },
                { ... },
                { ... },
                { ... }
            ]
        }
    ]
}
```
Создайте в папке **C:\ADFGetStarted** файл JSON с именем **ADFCopyTutorialARM.json** со следующим содержимым:

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
      "storageAccountName": { "type": "string", "metadata": { "description": "Name of the Azure storage account that contains the data to be copied." } },
      "storageAccountKey": { "type": "securestring", "metadata": { "description": "Key for the Azure storage account." } },
      "sourceBlobContainer": { "type": "string", "metadata": { "description": "Name of the blob container in the Azure Storage account." } },
      "sourceBlobName": { "type": "string", "metadata": { "description": "Name of the blob in the container that has the data to be copied to Azure SQL Database table" } },
      "sqlServerName": { "type": "string", "metadata": { "description": "Name of the logical SQL server that will hold the output/copied data." } },
      "databaseName": { "type": "string", "metadata": { "description": "Name of the Azure SQL Database in the logical SQL server." } },
      "sqlServerUserName": { "type": "string", "metadata": { "description": "Name of the user that has access to the logical SQL server." } },
      "sqlServerPassword": { "type": "securestring", "metadata": { "description": "Password for the user." } },
      "targetSQLTable": { "type": "string", "metadata": { "description": "Table in the Azure SQL Database that will hold the copied data." } 
      } 
    },
    "variables": {
      "dataFactoryName": "[concat('AzureBlobToAzureSQLDatabaseDF', uniqueString(resourceGroup().id))]",
      "azureSqlLinkedServiceName": "AzureSqlLinkedService",
      "azureStorageLinkedServiceName": "AzureStorageLinkedService",
      "blobInputDatasetName": "BlobInputDataset",
      "sqlOutputDatasetName": "SQLOutputDataset",
      "pipelineName": "Blob2SQLPipeline"
    },
    "resources": [
      {
        "name": "[variables('dataFactoryName')]",
        "apiVersion": "2015-10-01",
        "type": "Microsoft.DataFactory/datafactories",
        "location": "West US",
        "resources": [
          {
            "type": "linkedservices",
            "name": "[variables('azureStorageLinkedServiceName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureStorage",
              "description": "Azure Storage linked service",
              "typeProperties": {
                "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
              }
            }
          },
          {
            "type": "linkedservices",
            "name": "[variables('azureSqlLinkedServiceName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureSqlDatabase",
              "description": "Azure SQL linked service",
              "typeProperties": {
                "connectionString": "[concat('Server=tcp:',parameters('sqlServerName'),'.database.windows.net,1433;Database=', parameters('databaseName'), ';User ID=',parameters('sqlServerUserName'),';Password=',parameters('sqlServerPassword'),';Trusted_Connection=False;Encrypt=True;Connection Timeout=30')]"
              }
            }
          },
          {
            "type": "datasets",
            "name": "[variables('blobInputDatasetName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]",
              "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureBlob",
              "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
              "structure": [
                {
                  "name": "Column0",
                  "type": "String"
                },
                {
                  "name": "Column1",
                  "type": "String"
                }
              ],
              "typeProperties": {
                "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
                "fileName": "[parameters('sourceBlobName')]",
                "format": {
                  "type": "TextFormat",
                  "columnDelimiter": ","
                }
              },
              "availability": {
                "frequency": "Hour",
                "interval": 1
              },
              "external": true
            }
          },
          {
            "type": "datasets",
            "name": "[variables('sqlOutputDatasetName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]",
              "[variables('azureSqlLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureSqlTable",
              "linkedServiceName": "[variables('azureSqlLinkedServiceName')]",
              "structure": [
                {
                  "name": "FirstName",
                  "type": "String"
                },
                {
                  "name": "LastName",
                  "type": "String"
                }
              ],
              "typeProperties": {
                "tableName": "[parameters('targetSQLTable')]"
              },
              "availability": {
                "frequency": "Hour",
                "interval": 1
              }
            }
          },
          {
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
              "start": "2017-05-11T00:00:00Z",
              "end": "2017-05-12T00:00:00Z"
            }
          }
        ]
      }
    ]
  }
```

## <a name="parameters-json"></a>JSON-файл параметров
Создайте JSON-файл с именем **ADFCopyTutorialARM-Parameters.json**, содержащий параметры для шаблона Azure Resource Manager. 

> [!IMPORTANT]
> Укажите значения имени и ключа вашей учетной записи хранения Azure для параметров storageAccountName и storageAccountKey.  
> 
> Укажите логический сервер SQL Server, базу данных, пользователя и пароль для параметров sqlServerName, databaseName, sqlServerUserName и sqlServerPassword.  

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { 
        "storageAccountName": { "value": "<Name of the Azure storage account>"    },
        "storageAccountKey": {
            "value": "<Key for the Azure storage account>"
        },
        "sourceBlobContainer": { "value": "adftutorial" },
        "sourceBlobName": { "value": "emp.txt" },
        "sqlServerName": { "value": "<Name of the logical SQL server>" },
        "databaseName": { "value": "<Name of the database>" },
        "sqlServerUserName": { "value": "<Name of the user who has access to the database>" },
        "sqlServerPassword": { "value": "<password for the user>" },
        "targetSQLTable": { "value": "emp" }
    }
}
```

> [!IMPORTANT]
> Можно создать отдельные JSON-файлы параметров для сред разработки, тестирования и рабочей среды, которые можно использовать с одним и тем же шаблоном JSON фабрики данных. Скрипт PowerShell позволяет автоматизировать развертывание сущностей фабрики данных в этих средах.  
> 
> 

## <a name="create-data-factory"></a>Создание фабрики данных
1. Запустите **Azure PowerShell** и выполните следующие команды:
   * Выполните следующую команду и введите имя пользователя и пароль, которые используются для входа на портал Azure.
   
    ```PowerShell
    Connect-AzAccount       
    ```  
   * Выполните следующую команду, чтобы просмотреть все подписки для этой учетной записи.
   
    ```PowerShell
    Get-AzSubscription
    ```   
   * Выполните следующую команду, чтобы выбрать подписку, с которой вы собираетесь работать.
    
    ```PowerShell
    Get-AzSubscription -SubscriptionName <SUBSCRIPTION NAME> | Set-AzContext
    ```    
2. Чтобы развернуть сущности фабрики данных с помощью шаблона Resource Manager, созданного на шаге 1, выполните следующую команду.

    ```PowerShell   
    New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile C:\ADFGetStarted\ADFCopyTutorialARM.json -TemplateParameterFile C:\ADFGetStarted\ADFCopyTutorialARM-Parameters.json
    ```

## <a name="monitor-pipeline"></a>Отслеживание конвейера

1. Войдите на [портал Azure](https://portal.azure.com), используя свою учетную запись Azure.

1. Щелкните **Фабрики данных** в меню слева или выберите **Все службы** и щелкните **Фабрики данных** в категории **Аналитика**.
   
    ![Меню "Фабрики данных"](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/data-factories-menu.png)

1. На странице **Фабрики данных** найдите свою фабрику данных (AzureBlobToAzureSQLDatabaseDF). 
   
    ![Поиск фабрики данных](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/search-for-data-factory.png)  

1. Щелкните свою фабрику данных Azure. После этого откроется домашняя страница фабрики данных.
   
    ![Домашняя страница фабрики данных](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/data-factory-home-page.png)  

1. Следуйте инструкциям из раздела [Отслеживание конвейера](data-factory-monitor-manage-pipelines.md) для мониторинга конвейера и баз данных, созданных при работе с этим руководством. Сейчас Visual Studio не поддерживает мониторинг конвейеров фабрики данных.

1. Когда срез будет в состоянии **Готово**, проверьте, были ли скопированы данные в таблицу **emp** в Базе данных SQL Azure.

Дополнительные сведения о том, как использовать колонки на портале Azure для мониторинга конвейера и наборов данных, созданных с помощью этого руководства, см.в [этой статье](data-factory-monitor-manage-pipelines.md).

Дополнительные сведения о том, как использовать приложение по мониторингу и управлению для мониторинга конвейеров данных, см. в [этой статье](data-factory-monitor-manage-app.md).

## <a name="data-factory-entities-in-the-template"></a>Сущности фабрики данных в шаблоне

### <a name="define-data-factory"></a>Определение фабрики данных

Фабрику данных следует определить в шаблоне Resource Manager, как показано в следующем примере:

```json
{
  "resources": [
    {
      "name": "[variables('dataFactoryName')]",
      "apiVersion": "2015-10-01",
      "type": "Microsoft.DataFactory/datafactories",
      "location": "West US"
    }
  ]
}
```

Параметр dataFactoryName определяется следующим образом: 

```json
{
    "dataFactoryName": "[concat('AzureBlobToAzureSQLDatabaseDF', uniqueString(resourceGroup().id))]"
}
```

Это уникальная строка благодаря идентификатору группы ресурсов.

### <a name="defining-data-factory-entities"></a>Определение сущностей фабрики данных

В шаблоне JSON определены следующие сущности фабрики данных: 

1. [Связанная служба хранения Azure](#azure-storage-linked-service)
2. [Связанная служба SQL Azure](#azure-sql-database-linked-service)
3. [Набор данных большого двоичного объекта Azure](#azure-blob-dataset)
4. [Набор данных SQL Azure](#azure-sql-dataset)
5. [Конвейер данных с действием копирования](#data-pipeline)

#### <a name="azure-storage-linked-service"></a>Связанная служба хранения Azure

Связанная служба хранилища Azure связывает учетную запись хранения Azure с фабрикой данных. Вы создали контейнер и отправили данные в эту учетную запись хранения в ходе выполнения предварительных [требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). В этом разделе вы укажете имя и ключ вашей учетной записи хранения Azure. Дополнительные сведения о свойствах JSON для определения связанной службы хранилища Azure см. в разделе [Связанная служба хранилища Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service). 

```json
{
    "type": "linkedservices",
    "name": "[variables('azureStorageLinkedServiceName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureStorage",
        "description": "Azure Storage linked service",
        "typeProperties": {
            "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
        }
    }
}
```

Для connectionString используются параметры storageAccountName и storageAccountKey. Значения для этих параметров передаются с помощью файла конфигурации. В этом определении также используются переменные azureStorageLinkedService и dataFactoryName, заданные в шаблоне. 

#### <a name="azure-sql-database-linked-service"></a>Связанная служба "База данных SQL Azure"

Связанная служба SQL Azure (AzureSqlLinkedService) связывает базу данных в службе "База данных SQL Azure" с фабрикой данных. В этой базе данных хранятся данные, скопированные из хранилища BLOB-объектов. Вы создали пустую таблицу в этой базе данных в ходе выполнения [предварительных требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). В этом разделе вы укажете имя логического сервера SQL Server, имя базы данных, имя пользователя и пароль. Дополнительные сведения о свойствах JSON для определения связанной службы SQL Azure см. в разделе [Связанная служба SQL Azure](data-factory-azure-sql-connector.md#linked-service-properties).  

```json
{
    "type": "linkedservices",
    "name": "[variables('azureSqlLinkedServiceName')]",
    "dependsOn": [
      "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
      "type": "AzureSqlDatabase",
      "description": "Azure SQL linked service",
      "typeProperties": {
        "connectionString": "[concat('Server=tcp:',parameters('sqlServerName'),'.database.windows.net,1433;Database=', parameters('databaseName'), ';User ID=',parameters('sqlServerUserName'),';Password=',parameters('sqlServerPassword'),';Trusted_Connection=False;Encrypt=True;Connection Timeout=30')]"
      }
    }
}
```

Для connectionString используются параметры sqlServerName, databaseName, sqlServerUserName и sqlServerPassword, значения которых передаются с помощью файла конфигурации. В определении также используются переменные azureSqlLinkedServiceName dataFactoryName из шаблона.

#### <a name="azure-blob-dataset"></a>Набор данных большого двоичного объекта Azure
Связанная служба хранилища Azure указывает строку подключения, которую фабрика данных использует во время выполнения, чтобы подключиться к учетной записи хранения Azure. В определении набора данных большого двоичного объекта Azure укажите имена контейнера больших двоичных объектов, папки и файла, содержащего входные данные. Подробные сведения о свойствах JSON, которые используюся для определения набора данных большого двоичного объекта Azure, см. в разделе [Свойства типа "Набор данных большого двоичного объекта Azure"](data-factory-azure-blob-connector.md#dataset-properties). 

```json
{
    "type": "datasets",
    "name": "[variables('blobInputDatasetName')]",
    "dependsOn": [
      "[variables('dataFactoryName')]",
      "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureBlob",
          "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
        "structure": [
        {
              "name": "Column0",
              "type": "String"
        },
        {
              "name": "Column1",
              "type": "String"
        }
          ],
          "typeProperties": {
            "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
            "fileName": "[parameters('sourceBlobName')]",
            "format": {
                  "type": "TextFormat",
                  "columnDelimiter": ","
            }
          },
          "availability": {
            "frequency": "Hour",
            "interval": 1
          },
          "external": true
    }
}
```

#### <a name="azure-sql-dataset"></a>Набор данных SQL Azure
В этом разделе нужно указать имя таблицы в Базе данных SQL Azure, содержащей скопированные данные из Хранилища BLOB-объектов Azure. Подробные сведения о свойствах JSON, которые используются для определения набора данных SQL Azure, см. в разделе [Свойства типа "Набор данных SQL Azure"](data-factory-azure-sql-connector.md#dataset-properties). 

```json
{
    "type": "datasets",
    "name": "[variables('sqlOutputDatasetName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
          "[variables('azureSqlLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
          "type": "AzureSqlTable",
          "linkedServiceName": "[variables('azureSqlLinkedServiceName')]",
          "structure": [
        {
              "name": "FirstName",
              "type": "String"
        },
        {
              "name": "LastName",
              "type": "String"
        }
          ],
          "typeProperties": {
            "tableName": "[parameters('targetSQLTable')]"
          },
          "availability": {
            "frequency": "Hour",
            "interval": 1
          }
    }
}
```

#### <a name="data-pipeline"></a>Конвейер данных
Здесь вы определите конвейер, копирующий данные из набора данных большого двоичного объекта Azure в набор данных SQL Azure. В разделе [Конвейер JSON](data-factory-create-pipelines.md#pipeline-json) описаны элементы JSON, используемые для определения конвейера в этом примере. 

```json
{
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
          "start": "2017-05-11T00:00:00Z",
          "end": "2017-05-12T00:00:00Z"
    }
}
```

## <a name="reuse-the-template"></a>Повторное использование шаблона
В этом руководстве описывается создание шаблона для определения сущностей фабрики данных, а также шаблона, передающего значения для параметров. Конвейер копирует данные из учетной записи хранения Azure в Базу данных SQL Azure, указанную с помощью параметров. Чтобы использовать один шаблон для развертывания сущностей фабрики данных в разных средах, нужно создать файл параметров для каждой среды и использовать его при развертывании в определенной среде.     

Пример  

```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Dev.json
```
```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Test.json
```
```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Production.json
```

Обратите внимание, что первая команда использует файл параметров для среды разработки, вторая — для среды тестирования, а третья — для рабочей среды.  

Шаблон можно снова использовать для выполнения повторяющихся задач. Например, вам нужно создать несколько фабрик данных с одним или несколькими конвейерами, которые реализуют одинаковую логику, но при этом каждая фабрика данных использует разные учетные записи хранения и базы данных SQL. В этом сценарии для создания фабрик данных используется один шаблон в той же среде (разработки, тестирования или рабочей) с различными файлами параметров.   

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве показано, как использовать Хранилище BLOB-объектов Azure и Базу данных SQL Azure как исходное и целевое хранилища данных в операции копирования. В следующей таблице приведен список хранилищ данных, которые поддерживаются в качестве источников и целевых расположений для действия копирования. 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Чтобы получить дополнительные сведения о том, как скопировать данные в хранилище данных или из него, щелкните ссылку для хранилища данных в таблице.
