---
title: Копирование данных из хранилища BLOB-объектов Azure в базу данных SQL Azure
description: В этом руководстве приведены пошаговые инструкции по копированию данных из хранилища BLOB-объектов Azure в Базу данных SQL Azure.
author: linda33wj
ms.service: data-factory
ms.topic: tutorial
ms.date: 02/18/2021
ms.author: jingwang
ms.openlocfilehash: 2a9e19f24c4889a194c85e33317860db1d3a68d2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101736565"
---
# <a name="copy-data-from-azure-blob-to-azure-sql-database-using-azure-data-factory"></a>Копирование данных из хранилища BLOB-объектов Azure в Базу данных SQL Azure с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этом руководстве вы создадите фабрику данных с конвейером, который перемещает данные из хранилища BLOB-объектов Azure в Базу данных SQL Azure. Шаблон конфигурации в этом руководстве применяется к копированию из файлового в реляционное хранилище данных. Список хранилищ данных, которые поддерживаются в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

При работе с этим руководством вы выполните следующие задачи:

> [!div class="checklist"]
> * Создали фабрику данных.
> * Создание связанных служб хранилища Azure и базы данных SQL Azure.
> * Создание наборов данных большого двоичного объекта Azure и базы данных SQL Azure.
> * Создание конвейера с действием копирования.
> * Запуск конвейера.
> * Мониторинг конвейера и выполнения действий.

В этом руководстве используется пакет SDK для .NET. Вы можете использовать другие механизмы для взаимодействия с Фабрикой данных Azure (см. примеры в разделе **Краткие руководства**).

Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/) Azure, прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* *Учетная запись хранения Azure.* В этом руководстве в качестве *источника* будет использоваться хранилище BLOB-объектов. Если у вас нет учетной записи хранения Azure, см. инструкции по [созданию учетной записи хранения общего назначения](../storage/common/storage-account-create.md).
* *База данных SQL Azure*. Используйте базу данных как хранилище данных-*приемник*. Если у вас нет базы данных в службе "База данных SQL Azure", воспользуйтесь кратким руководством [Создание отдельной базы данных в Базе данных SQL Azure](../azure-sql/database/single-database-create-quickstart.md).
* *Visual Studio*. В этом пошаговом руководстве используется Visual Studio 2019.
* *[Пакет Azure SDK для .NET](/dotnet/azure/dotnet-tools)* .
* *Приложение Azure Active Directory*. Если у вас нет приложения Azure Active Directory, см. раздел [о создании такого приложения](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) из [руководства по созданию приложения AAD с помощью портала](../active-directory/develop/howto-create-service-principal-portal.md). Скопируйте следующие значения для использования на следующих шагах: **идентификатор приложения (клиента)** , **ключ проверки подлинности** и **идентификатор каталога (арендатора)** . Назначьте приложению роль **Участник**, следуя указаниям в той же статье.

### <a name="create-a-blob-and-a-sql-table"></a>Создание большого двоичного объекта и таблицы SQL

Теперь подготовьте большой двоичный объект Azure и Базу данных SQL Azure для работы с этим руководством, создав BLOB-объект в качестве источника и таблицу SQL в качестве приемника.

#### <a name="create-a-source-blob"></a>Создание исходного большого двоичного объекта

Сначала нужно создать исходный BLOB-объект. Для этого создайте контейнер и отправьте в него входной текстовый файл.

1. Откройте Блокнот. Скопируйте следующий текст и сохраните его в локальный файл с именем *inputEmp.txt*.

    ```inputEmp.txt
    John|Doe
    Jane|Doe
    ```

2. С помощью [Обозревателя службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/) или аналогичного средства создайте контейнер *adfv2tutorial* и отправьте в него файл *inputEmp.txt*.

#### <a name="create-a-sink-sql-table"></a>Создание таблицы-приемника SQL

Теперь создайте таблицу SQL в качестве приемника.

1. Используйте следующий скрипт SQL, чтобы создать таблицу *dbo.emp* в базе данных SQL Azure.

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO

    CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
    ```

2. Предоставьте службам Azure доступ к базе данных SQL. Убедитесь, что доступ к службам Azure разрешен для вашего сервера, чтобы служба "Фабрика данных" могла записывать данные в базу данных SQL. Чтобы проверить и при необходимости включить этот параметр, сделайте следующее.

    1. Перейдите на [портал Azure](https://portal.azure.com) для управления сервером SQL Server. Выполните поиск по фразе **Серверы SQL** и выберите этот вариант.

    2. Выберите нужный сервер.

    3. В меню сервера SQL выберите в разделе **Безопасность** элемент **Брандмауэры и виртуальные сети**.

    4. На странице **Брандмауэры и виртуальные сети** для параметра **Разрешить доступ к серверу службам и ресурсам Azure** выберите значение **ВКЛ**.

## <a name="create-a-visual-studio-project"></a>Создание проекта Visual Studio

С помощью Visual Studio создайте консольное приложение C# .NET.

1. Запустите Visual Studio.
2. В окне **Начало работы** выберите **Создать проект**.
3. В окне **Создание проекта** выберите версию C# для элемента **Консольное приложение (.NET Framework)** в списке типов проекта. Выберите **Далее**.
4. В окне **Настроить новый проект** введите для параметра **Имя проекта** значение *ADFv2Tutorial*. Для параметра **Расположение** найдите или создайте каталог, в который будет сохранен проект. Щелкните **Создать**. Новый проект появится в интегрированной среде разработки Visual Studio.

## <a name="install-nuget-packages"></a>Установка пакетов Nuget

Теперь установите обязательные пакеты библиотек с помощью диспетчера пакетов NuGet.

1. В строке меню последовательно выберите **Средства** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.
2. На панели **Консоль диспетчера пакетов** выполните следующие команды, чтобы установить пакеты. Дополнительные сведения см. в документации по пакету NuGet [Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/) для Фабрики данных Azure.

    ```package manager console
    Install-Package Microsoft.Azure.Management.DataFactory
    Install-Package Microsoft.Azure.Management.ResourceManager -PreRelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-data-factory-client"></a>Создание клиента фабрики данных

Чтобы создать клиент фабрики данных, сделайте следующее.

1. Откройте файл *Program.cs* и замените все существующие инструкции `using` следующим кодом, чтобы добавить ссылки на пространства имен.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Rest;
    using Microsoft.Rest.Serialization;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.DataFactory;
    using Microsoft.Azure.Management.DataFactory.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

2. Добавьте приведенный ниже код в метод `Main`, в котором задаются переменные. Замените значения 14 заполнителей на собственные.

    Список регионов Azure, в которых сейчас доступна Фабрика данных см. на странице [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). В раскрывающемся списке **Продукты** выберите **Обзор** > **Аналитика** > **Фабрика данных**. Затем в раскрывающемся списке **Регионы** выберите все интересующие регионы. Отобразится сетка со сведениями о доступности продуктов Фабрики данных для выбранных регионов.

    > [!NOTE]
    > Используемые Фабрикой данных хранилища данных, например служба хранилища Azure и База данных SQL Azure, и вычислительные среды, например HDInsight, могут находиться не в том регионе, который выбран для самой Фабрики данных.

    ```csharp
    // Set variables
    string tenantID = "<your tenant ID>";
    string applicationId = "<your application ID>";
    string authenticationKey = "<your authentication key for the application>";
    string subscriptionId = "<your subscription ID to create the factory>";
    string resourceGroup = "<your resource group to create the factory>";

    string region = "<location to create the data factory in, such as East US>";
    string dataFactoryName = "<name of data factory to create (must be globally unique)>";

    // Specify the source Azure Blob information
    string storageAccount = "<your storage account name to copy data>";
    string storageKey = "<your storage account key>";
    string inputBlobPath = "adfv2tutorial/";
    string inputBlobName = "inputEmp.txt";

    // Specify the sink Azure SQL Database information
    string azureSqlConnString =
        "Server=tcp:<your server name>.database.windows.net,1433;" +
        "Database=<your database name>;" +
        "User ID=<your username>@<your server name>;" +
        "Password=<your password>;" +
        "Trusted_Connection=False;Encrypt=True;Connection Timeout=30";
    string azureSqlTableName = "dbo.emp";

    string storageLinkedServiceName = "AzureStorageLinkedService";
    string sqlDbLinkedServiceName = "AzureSqlDbLinkedService";
    string blobDatasetName = "BlobDataset";
    string sqlDatasetName = "SqlDataset";
    string pipelineName = "Adfv2TutorialBlobToSqlCopy";
    ```

3. Добавьте в метод `Main` следующий код, который создает экземпляр класса `DataFactoryManagementClient`. Этот объект используется не только для создания фабрики данных, связанной службы, наборов данных и конвейера, но и для отслеживания подробностей выполнения конвейера.

    ```csharp
    // Authenticate and create a data factory management client
    var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
    ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
    AuthenticationResult result = context.AcquireTokenAsync(
        "https://management.azure.com/", cc
    ).Result;
    ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
    var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };
    ```

## <a name="create-a-data-factory"></a>Создание фабрики данных

Добавьте в метод `Main` следующий код, который создает *фабрику данных*.

```csharp
// Create a data factory
Console.WriteLine("Creating a data factory " + dataFactoryName + "...");
Factory dataFactory = new Factory
{
    Location = region,
    Identity = new FactoryIdentity()
};

client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(dataFactory, client.SerializationSettings)
);

while (
    client.Factories.Get(
        resourceGroup, dataFactoryName
    ).ProvisioningState == "PendingCreation"
)
{
    System.Threading.Thread.Sleep(1000);
}
```

## <a name="create-linked-services"></a>Создание связанных служб

Работая с этим руководством, вы создадите две связанные службы для источника и приемника, соответственно.

### <a name="create-an-azure-storage-linked-service"></a>Создание связанной службы хранилища Azure

Добавьте в метод `Main` приведенный ниже код, который создает *связанную службу хранилища Azure*. Информацию о поддерживаемых свойствах и подробные сведения см. в статье [о свойствах связанной службы BLOB-объектов Azure](connector-azure-blob-storage.md#linked-service-properties).

```csharp
// Create an Azure Storage linked service
Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");

LinkedServiceResource storageLinkedService = new LinkedServiceResource(
    new AzureStorageLinkedService
    {
        ConnectionString = new SecureString(
            "DefaultEndpointsProtocol=https;AccountName=" + storageAccount +
            ";AccountKey=" + storageKey
        )
    }
);

client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, storageLinkedServiceName, storageLinkedService
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings)
);
```

### <a name="create-an-azure-sql-database-linked-service"></a>Создание связанной службы Базы данных SQL Azure

Добавьте в метод `Main` приведенный ниже код, который создает *связанную службу Базы данных SQL Azure*. Информацию о поддерживаемых свойствах и подробные сведения см. в статье [о свойствах связанной службы Базы данных SQL Azure](connector-azure-sql-database.md#linked-service-properties).

```csharp
// Create an Azure SQL Database linked service
Console.WriteLine("Creating linked service " + sqlDbLinkedServiceName + "...");

LinkedServiceResource sqlDbLinkedService = new LinkedServiceResource(
    new AzureSqlDatabaseLinkedService
    {
        ConnectionString = new SecureString(azureSqlConnString)
    }
);

client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, sqlDbLinkedServiceName, sqlDbLinkedService
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(sqlDbLinkedService, client.SerializationSettings)
);
```

## <a name="create-datasets"></a>Создание наборов данных

В этом разделе вы создадите два набора данных: по одному для источника и приемника.

### <a name="create-a-dataset-for-source-azure-blob"></a>Создание набора данных для исходного большого двоичного объекта Azure

Добавьте в метод `Main` приведенный ниже код, который создает *набор данных большого двоичного объекта Azure*. Информацию о поддерживаемых свойствах и подробные сведения см. в разделе [о свойствах набора данных BLOB-объектов Azure](connector-azure-blob-storage.md#dataset-properties).

Задайте набор данных, представляющий исходные данные в большом двоичном объекте Azure. Этот набор данных большого двоичного объекта относится к связанной службе хранилища Azure, созданной на предыдущем шаге, и описывает следующее:

- расположение большого двоичного объекта для копирования: `FolderPath` и `FileName`;
- формат большого двоичного объекта, указывающий, как анализировать содержимое, `TextFormat` и все параметры, как например разделитель столбцов;
- структура данных, включая имена столбцов и типы данных, которые в нашем примере сопоставляются с таблицей-приемником SQL.

```csharp
// Create an Azure Blob dataset
Console.WriteLine("Creating dataset " + blobDatasetName + "...");
DatasetResource blobDataset = new DatasetResource(
    new AzureBlobDataset
    {
        LinkedServiceName = new LinkedServiceReference {
            ReferenceName = storageLinkedServiceName
        },
        FolderPath = inputBlobPath,
        FileName = inputBlobName,
        Format = new TextFormat { ColumnDelimiter = "|" },
        Structure = new List<DatasetDataElement>
        {
            new DatasetDataElement { Name = "FirstName", Type = "String" },
            new DatasetDataElement { Name = "LastName", Type = "String" }
        }
    }
);

client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, blobDatasetName, blobDataset
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings)
);
```

### <a name="create-a-dataset-for-sink-azure-sql-database"></a>Создание набора данных для базы данных-приемника SQL Azure

Добавьте в метод `Main` приведенный ниже код, который создает *набор данных Базы данных SQL Azure*. Информацию о поддерживаемых свойствах и подробные сведения см. в разделе [о свойствах набора данных Базы данных SQL Azure](connector-azure-sql-database.md#dataset-properties).

Задайте набор данных, который представляет данные приемника в Базе данных SQL Azure. Этот набор данных относится к связанной службе Базы данных SQL Azure, которую вы создали на предыдущем шаге. Он также указывает таблицу SQL, которая содержит копируемые данных.

```csharp
// Create an Azure SQL Database dataset
Console.WriteLine("Creating dataset " + sqlDatasetName + "...");
DatasetResource sqlDataset = new DatasetResource(
    new AzureSqlTableDataset
    {
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = sqlDbLinkedServiceName
        },
        TableName = azureSqlTableName
    }
);

client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, sqlDatasetName, sqlDataset
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(sqlDataset, client.SerializationSettings)
);
```

## <a name="create-a-pipeline"></a>Создание конвейера

Добавьте в метод `Main` приведенный ниже код, который создает *конвейер с действием копирования*. В этом руководстве этот конвейер содержит одно действие `CopyActivity`, которое принимает в качестве источника набор данных большого двоичного объекта, а в качестве приемника — набор данных SQL. Информацию о поддерживаемых свойствах и подробные сведения см. в разделе [Действие копирования в Фабрике данных Azure](copy-activity-overview.md).

```csharp
// Create a pipeline with copy activity
Console.WriteLine("Creating pipeline " + pipelineName + "...");
PipelineResource pipeline = new PipelineResource
{
    Activities = new List<Activity>
    {
        new CopyActivity
        {
            Name = "CopyFromBlobToSQL",
            Inputs = new List<DatasetReference>
            {
                new DatasetReference() { ReferenceName = blobDatasetName }
            },
            Outputs = new List<DatasetReference>
            {
                new DatasetReference { ReferenceName = sqlDatasetName }
            },
            Source = new BlobSource { },
            Sink = new SqlSink { }
        }
    }
};

client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, pipeline);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(pipeline, client.SerializationSettings)
);
```

## <a name="create-a-pipeline-run"></a>Создание конвейера

Добавьте в метод `Main` следующий код, который *активирует выполнение конвейера*.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(
    resourceGroup, dataFactoryName, pipelineName
).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="monitor-a-pipeline-run"></a>Мониторинг выполнения конвейера

Теперь вставьте код, чтобы проверять состояния выполнения конвейера и получать сведения о выполнении действий копирования.

1. Добавьте в метод `Main` следующий код, который постоянно проверяет состояния выполнения конвейера, пока тот не завершит копирование данных.

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(
            resourceGroup, dataFactoryName, runResponse.RunId
        );
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

2. Добавьте в метод `Main` следующий код, который извлекает сведения о выполнении действия копирования, например размер записанных и прочитанных данных.

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    RunFilterParameters filterParams = new RunFilterParameters(
        DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)
    );

    ActivityRunsQueryResponse queryResponse = client.ActivityRuns.QueryByPipelineRun(
        resourceGroup, dataFactoryName, runResponse.RunId, filterParams
    );

    if (pipelineRun.Status == "Succeeded")
    {
        Console.WriteLine(queryResponse.Value.First().Output);
    }
    else
        Console.WriteLine(queryResponse.Value.First().Error);

    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>Выполнение кода

Создайте приложение, выбрав **Сборка** > **Собрать решение**. Создайте и запустите приложение, выбрав **Отладка** > **Начать отладку**, а затем проверьте выполнение конвейера.

Консоль выведет ход выполнения создания фабрики данных, связанной службы, наборов данных, конвейера и выполнения конвейера. Затем она проверяет состояние выполнения конвейера. Дождитесь появления сведений о действии копирования с информацией о размере записанных и прочитанных данных. Затем вы можете с помощью SSMS (SQL Server Management Studio), Visual Studio или аналогичных средств подключиться к Базе данных SQL Azure и проверить, скопированы ли данные в указанную таблицу.

### <a name="sample-output"></a>Пример выходных данных

```json
Creating a data factory AdfV2Tutorial...
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>"
      }
    }
  }
}
Creating linked service AzureSqlDbLinkedService...
{
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
      }
    }
  }
}
Creating dataset BlobDataset...
{
  "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": "adfv2tutorial/",
      "fileName": "inputEmp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "|"
      }
    },
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
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureStorageLinkedService"
    }
  }
}
Creating dataset SqlDataset...
{
  "properties": {
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": "dbo.emp"
    },
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureSqlDbLinkedService"
    }
  }
}
Creating pipeline Adfv2TutorialBlobToSqlCopy...
{
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "inputs": [
          {
            "type": "DatasetReference",
            "referenceName": "BlobDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SqlDataset"
          }
        ],
        "name": "CopyFromBlobToSQL"
      }
    ]
  }
}
Creating pipeline run...
Pipeline run ID: 1cd03653-88a0-4c90-aabc-ae12d843e252
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 18,
  "dataWritten": 28,
  "rowsCopied": 2,
  "copyDuration": 2,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)",
  "usedDataIntegrationUnits": 2,
  "billedDuration": 2
}

Press any key to exit...
```

## <a name="next-steps"></a>Дальнейшие действия

В этом примере конвейер копирует данные из одного расположения в другое в хранилище BLOB-объектов Azure. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создали фабрику данных.
> * Создание связанных служб хранилища Azure и базы данных SQL Azure.
> * Создание наборов данных большого двоичного объекта Azure и базы данных SQL Azure.
> * Создание конвейера с действием копирования.
> * Запуск конвейера.
> * Мониторинг конвейера и выполнения действий.

Перейдите к следующему руководству, чтобы узнать о копировании данных из локальной среды в облако:

> [!div class="nextstepaction"]
>[Копирование данных между локальной средой и облаком](tutorial-hybrid-copy-powershell.md)
