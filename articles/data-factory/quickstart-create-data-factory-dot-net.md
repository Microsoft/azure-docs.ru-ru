---
title: Создание фабрики данных Azure с помощью пакета SDK для .NET
description: Создайте фабрику данных Azure и конвейер с помощью пакета SDK для .NET, чтобы копировать данные между расположениями в Хранилище BLOB-объектов Azure.
author: linda33wj
ms.service: data-factory
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 03/27/2021
ms.author: jingwang
ms.openlocfilehash: 3c667fe20b392bfb52b8300ce4b8b59d15a13b9a
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/05/2021
ms.locfileid: "106385434"
---
# <a name="quickstart-create-a-data-factory-and-pipeline-using-net-sdk"></a>Краткое руководство. Создание фабрики данных и конвейера с помощью пакета SDK .NET

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Текущая версия](quickstart-create-data-factory-dot-net.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом кратком руководстве показано, как создать фабрику данных Azure с помощью пакета SDK для .NET. Конвейер, который вы создадите в этой фабрике данных, **копирует** данные из одной папки в другую в хранилище BLOB-объектов Azure. Инструкции по **преобразованию** данных с помощью Фабрики данных Azure см. в статье [Преобразование данных с помощью действия Spark в фабрике данных Azure](tutorial-transform-data-spark-portal.md).

> [!NOTE]
> Эта статья не содержит подробный обзор службы фабрики данных. Общие сведения о службе фабрики данных Azure см. в статье [Введение в фабрику данных Azure](introduction.md).

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)] 

### <a name="visual-studio"></a>Visual Studio

В этом пошаговом руководстве используется Visual Studio 2019. Процедуры в Visual Studio 2013, 2015 или 2017 незначительно отличаются.

## <a name="create-an-application-in-azure-active-directory"></a>Создание приложения в Azure Active Directory

В разделе *Практическое руководство. Создание приложения Azure Active Directory и субъекта-службы с доступом к ресурсам с помощью портала* следуйте инструкциям по выполнению этих задач.

1. В разделе [Создание приложения Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) создайте приложение, которое представляет приложение .NET, создаваемое в этом руководстве. В качестве URL-адреса входа можно указать фиктивный URL-адрес, как показано в статье (`https://contoso.org/exampleapp`).
2. В разделе [Получение значений для входа](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) получите **код приложения**, а также **идентификатор арендатора** и запишите эти значения. Они потребуются в дальнейшем при выполнении инструкций этого руководства. 
3. В разделе [Сертификаты и секреты](../active-directory/develop/howto-create-service-principal-portal.md#authentication-two-options) получите **ключ аутентификации** и запишите это значение. Оно потребуется в дальнейшем при выполнении инструкций этого руководства.
4. В разделе [Назначение приложению роли](../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) назначьте приложению роль **Участник** на уровне подписки, чтобы приложение могло создавать фабрики данных в подписке.

## <a name="create-a-visual-studio-project"></a>Создание проекта Visual Studio

Далее создайте в Visual Studio консольное приложение C# .NET.

1. Запустите **Visual Studio**.
2. В окне запуска выберите **Создать новый проект** > **Консольное приложение (платформа .NET Framework)** . Требуется .NET версии 4.5.2 или более поздней.
3. В окне **Имя проекта** введите **ADFv2QuickStart**.
4. Выберите **Создать**, чтобы создать проект.

## <a name="install-nuget-packages"></a>Установка пакетов Nuget

1. Выберите **Инструменты** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.
2. На панели **Консоль диспетчера пакетов** выполните следующие команды, чтобы установить пакеты. Дополнительные сведения см. в документации по пакету NuGet [Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/).

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactory
    Install-Package Microsoft.Azure.Management.ResourceManager -IncludePrerelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-data-factory-client"></a>Создание клиента фабрики данных

1. Откройте файл **Program.cs** и включите следующие инструкции, чтобы добавить ссылки на пространства имен.

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

2. Добавьте следующий код в метод **Main**, который задает следующие переменные. Замените значения заполнителей на собственные. Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (служба хранилища Azure, база данных SQL Azure и многие другие) и вычисления (HDInsight и другие), используемые фабрикой данных, могут располагаться в других регионах.

   ```csharp
   // Set variables
   string tenantID = "<your tenant ID>";
   string applicationId = "<your application ID>";
   string authenticationKey = "<your authentication key for the application>";
   string subscriptionId = "<your subscription ID where the data factory resides>";
   string resourceGroup = "<your resource group where the data factory resides>";
   string region = "<the location of your resource group>";
   string dataFactoryName = 
       "<specify the name of data factory to create. It must be globally unique.>";
   string storageAccount = "<your storage account name to copy data>";
   string storageKey = "<your storage account key>";
   // specify the container and input folder from which all files 
   // need to be copied to the output folder. 
   string inputBlobPath =
       "<path to existing blob(s) to copy data from, e.g. containername/inputdir>";
   //specify the contains and output folder where the files are copied
   string outputBlobPath =
       "<the blob path to copy data to, e.g. containername/outputdir>";

   // name of the Azure Storage linked service, blob dataset, and the pipeline
   string storageLinkedServiceName = "AzureStorageLinkedService";
   string blobDatasetName = "BlobDataset";
   string pipelineName = "Adfv2QuickStartPipeline";
   ```
> [!NOTE]
> Для национального облака необходимо использовать соответствующие конечные точки для ActiveDirectoryAuthority и ResourceManagerUrl (BaseUri). Например, в Azure US Gov используйте https://login.microsoftonline.us вместо https://login.microsoftonline.com и https://management.usgovcloudapi.net вместо https://management.azure.com/ , а затем создайте клиент управления фабрикой данных. С помощью PowerShell можно легко получить URL-адреса конечных точек для различных облаков, выполнив команду "Get-AzEnvironment | Format-List", которая возвращает список конечных точек для каждой облачной среды.

3. Добавьте в метод **Main** приведенный ниже код, создающий экземпляр класса **DataFactoryManagementClient**. Этот объект используется не только для создания фабрики данных, связанной службы, наборов данных и конвейера, но и для отслеживания подробностей выполнения конвейера.

   ```csharp
   // Authenticate and create a data factory management client
   var context = new AuthenticationContext("https://login.microsoftonline.com/" + tenantID);
   ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
   AuthenticationResult result = context.AcquireTokenAsync(
       "https://management.azure.com/", cc).Result;
   ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
   var client = new DataFactoryManagementClient(cred) {
       SubscriptionId = subscriptionId };
   ```


## <a name="create-a-data-factory"></a>Создание фабрики данных

Добавьте следующий код, создающий **фабрику данных**, в метод **Main**. 

```csharp
// Create a data factory
Console.WriteLine("Creating data factory " + dataFactoryName + "...");
Factory dataFactory = new Factory
{
    Location = region,
    Identity = new FactoryIdentity()
};
client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(dataFactory, client.SerializationSettings));

while (client.Factories.Get(resourceGroup, dataFactoryName).ProvisioningState ==
       "PendingCreation")
{
    System.Threading.Thread.Sleep(1000);
}
```

## <a name="create-a-linked-service"></a>Создание связанной службы

Добавьте следующий код, создающий **связанную службу хранилища Azure**, в метод **Main**.

Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В этом руководстве необходимо создать одну связанную службу хранилища Azure для копирования хранилища-источника и приемника. В примере она называется AzureStorageLinkedService.

```csharp
// Create an Azure Storage linked service
Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");

LinkedServiceResource storageLinkedService = new LinkedServiceResource(
    new AzureStorageLinkedService
    {
        ConnectionString = new SecureString(
            "DefaultEndpointsProtocol=https;AccountName=" + storageAccount +
            ";AccountKey=" + storageKey)
    }
);
client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, storageLinkedServiceName, storageLinkedService);
Console.WriteLine(SafeJsonConvert.SerializeObject(
    storageLinkedService, client.SerializationSettings));
```

## <a name="create-a-dataset"></a>Создание набора данных

Добавьте следующий код, который создает **набор данных большого двоичного объекта Azure**, в метод **Main**.

Вы можете определить набор данных, который будет представлять данные для копирования из источника в приемник. В этом примере набор данных большого двоичного объекта относится к связанным службам хранилища Azure, созданным на предыдущем шаге. Набор данных принимает параметр, значение которого задается в действии, использующем набор данных. Параметр используется для создания folderPath, указывающего, где находятся или хранятся данные.

```csharp
// Create an Azure Blob dataset
Console.WriteLine("Creating dataset " + blobDatasetName + "...");
DatasetResource blobDataset = new DatasetResource(
    new AzureBlobDataset
    {
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        },
        FolderPath = new Expression { Value = "@{dataset().path}" },
        Parameters = new Dictionary<string, ParameterSpecification>
        {
            { "path", new ParameterSpecification { Type = ParameterType.String } }
        }
    }
);
client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, blobDatasetName, blobDataset);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
```

## <a name="create-a-pipeline"></a>Создание конвейера

Добавьте в метод **Main** следующий код, создающий **конвейер с действием копирования**.

В этом примере этот конвейер содержит одно действие и принимает два параметра: путь входного и выходного большого двоичного объекта. Значения для этих параметров устанавливаются при активации или выполнении конвейера. Действие копирования ссылается на тот же набор данных большого двоичного объекта, который был создан на предыдущем шаге, в качестве входного и выходного. Если набор данных используется в качестве входного, указывается путь к входным данным. И если набор данных используется в качестве выходного, указывается путь к выходным данным. 

```csharp
// Create a pipeline with a copy activity
Console.WriteLine("Creating pipeline " + pipelineName + "...");
PipelineResource pipeline = new PipelineResource
{
    Parameters = new Dictionary<string, ParameterSpecification>
    {
        { "inputPath", new ParameterSpecification { Type = ParameterType.String } },
        { "outputPath", new ParameterSpecification { Type = ParameterType.String } }
    },
    Activities = new List<Activity>
    {
        new CopyActivity
        {
            Name = "CopyFromBlobToBlob",
            Inputs = new List<DatasetReference>
            {
                new DatasetReference()
                {
                    ReferenceName = blobDatasetName,
                    Parameters = new Dictionary<string, object>
                    {
                        { "path", "@pipeline().parameters.inputPath" }
                    }
                }
            },
            Outputs = new List<DatasetReference>
            {
                new DatasetReference
                {
                    ReferenceName = blobDatasetName,
                    Parameters = new Dictionary<string, object>
                    {
                        { "path", "@pipeline().parameters.outputPath" }
                    }
                }
            },
            Source = new BlobSource { },
            Sink = new BlobSink { }
        }
    }
};
client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, pipeline);
Console.WriteLine(SafeJsonConvert.SerializeObject(pipeline, client.SerializationSettings));
```

## <a name="create-a-pipeline-run"></a>Создание конвейера

Добавьте в метод **Main** следующий код, **активирующий выполнение конвейера**.

Этот код также указывает параметрам **inputPath** и **outputPath**, задаваемым в конвейере, фактические значения путей большого двоичного объекта источника и приемника.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> parameters = new Dictionary<string, object>
{
    { "inputPath", inputBlobPath },
    { "outputPath", outputBlobPath }
};
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(
    resourceGroup, dataFactoryName, pipelineName, parameters: parameters
).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="monitor-a-pipeline-run"></a>Мониторинг выполнения конвейера

1. Добавьте следующий код в метод **Main**, чтобы постоянно проверять состояние до завершения копирования данных.

   ```csharp
   // Monitor the pipeline run
   Console.WriteLine("Checking pipeline run status...");
   PipelineRun pipelineRun;
   while (true)
   {
       pipelineRun = client.PipelineRuns.Get(
           resourceGroup, dataFactoryName, runResponse.RunId);
       Console.WriteLine("Status: " + pipelineRun.Status);
       if (pipelineRun.Status == "InProgress" || pipelineRun.Status == "Queued")
           System.Threading.Thread.Sleep(15000);
       else
           break;
   }
   ```

2. Добавьте в метод **Main** следующий код, извлекающий сведения о выполнении действия копирования, например размер записанных и прочитанных данных.

   ```csharp
   // Check the copy activity run details
   Console.WriteLine("Checking copy activity run details...");

   RunFilterParameters filterParams = new RunFilterParameters(
       DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10));
   ActivityRunsQueryResponse queryResponse = client.ActivityRuns.QueryByPipelineRun(
       resourceGroup, dataFactoryName, runResponse.RunId, filterParams);
   if (pipelineRun.Status == "Succeeded")
       Console.WriteLine(queryResponse.Value.First().Output);
   else
       Console.WriteLine(queryResponse.Value.First().Error);
   Console.WriteLine("\nPress any key to exit...");
   Console.ReadKey();
   ```

## <a name="run-the-code"></a>Выполнение кода

Создайте и запустите приложение, а затем проверьте выполнение конвейера.

Консоль выведет ход выполнения создания фабрики данных, связанной службы, наборов данных, конвейера и выполнения конвейера. Затем она проверяет состояние выполнения конвейера. Дождитесь появления сведений о действии копирования с размером данных для чтения или записи. Затем воспользуйтесь такими средствами, как [Обозреватель службы хранилища Azure](https://azure.microsoft.com/features/storage-explorer/), чтобы проверить, скопированы ли большие двоичные объекты в outputBlobPath из inputBlobPath, как указано в переменных.

### <a name="sample-output"></a>Пример выходных данных

```json
Creating data factory SPv2Factory0907...
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
        "value": "DefaultEndpointsProtocol=https;AccountName=<storageAccountName>;AccountKey=<storageAccountKey>",
        "type": "SecureString"
      }
    }
  }
}
Creating dataset BlobDataset...
{
  "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@{dataset().path}",
        "type": "Expression"
      }
    },
    "linkedServiceName": {
      "referenceName": "AzureStorageLinkedService",
      "type": "LinkedServiceReference"
    },
    "parameters": {
      "path": {
        "type": "String"
      }
    }
  }
}
Creating pipeline Adfv2QuickStartPipeline...
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
            "type": "BlobSink"
          }
        },
        "inputs": [
          {
            "referenceName": "BlobDataset",
            "parameters": {
              "path": "@pipeline().parameters.inputPath"
            },
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "BlobDataset",
            "parameters": {
              "path": "@pipeline().parameters.outputPath"
            },
            "type": "DatasetReference"
          }
        ],
        "name": "CopyFromBlobToBlob"
      }
    ],
    "parameters": {
      "inputPath": {
        "type": "String"
      },
      "outputPath": {
        "type": "String"
      }
    }
  }
}
Creating pipeline run...
Pipeline run ID: 308d222d-3858-48b1-9e66-acd921feaa09
Checking pipeline run status...
Status: InProgress
Status: InProgress
Checking copy activity run details...
{
    "dataRead": 331452208,
    "dataWritten": 331452208,
    "copyDuration": 23,
    "throughput": 14073.209,
    "errors": [],
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (West US)",
    "usedDataIntegrationUnits": 2,
    "billedDuration": 23
}

Press any key to exit...
```

## <a name="verify-the-output"></a>Проверка выходных данных

Конвейер автоматически создает выходную папку в контейнере больших двоичных объектов **adftutorial**. Затем он копирует файл **emp.txt** из входной папки в выходную. 

1. На портале Azure на странице контейнера **adftutorial** в разделе [Добавление входной папки и файла для контейнера больших двоичных объектов](#add-an-input-folder-and-file-for-the-blob-container), описанном выше, выберите **Обновить**, чтобы просмотреть выходную папку. 
2. В списке папок щелкните **output** в списке папок.
3. Убедитесь, что файл **emp.txt** скопирован в папку output. 

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы программно удалить фабрики данных, добавьте следующие строки кода в программу: 

```csharp
Console.WriteLine("Deleting the data factory");
client.Factories.Delete(resourceGroup, dataFactoryName);
```

## <a name="next-steps"></a>Дальнейшие действия

В этом примере конвейер копирует данные из одного расположения в другое в хранилище BLOB-объектов Azure. Перейдите к [руководствам](tutorial-copy-data-dot-net.md), чтобы узнать об использовании фабрики данных в различных сценариях. 
