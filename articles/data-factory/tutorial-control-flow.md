---
title: Ветвление в конвейере службы "Фабрика данных Azure"
description: Узнайте, как контролировать поток данных в фабрике данных Azure с помощью ветвления и создания цепочки действий.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 9/27/2019
ms.openlocfilehash: 83bc90ca251fddd498dd365f1a4b00689a471475
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104785671"
---
# <a name="branching-and-chaining-activities-in-a-data-factory-pipeline"></a>Ветвления и создание цепочки действий в конвейере фабрики данных

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве создается конвейер фабрики данных, который демонстрирует некоторые функции потока управления. Этот конвейер копирует данные из контейнера в хранилище BLOB-объектов Azure в другой контейнер в той же учетной записи хранения. Если копирование завершается успешно, конвейер отправляет сведения об успешной операции копирования в сообщении электронной почты. Эта информация может включать в себя объем записанных данных. Если действие копирования не удается, в сообщении электронной почты отправляются сведения о сбое копирования, например сообщение об ошибке. В этом руководстве вы научитесь передавать параметры.

Этот рисунок содержит общие сведения о сценарии:

![На схеме показано, что служба "Хранилище BLOB-объектов Azure" является целевым ресурсом копирования. В случае успешного выполнения копирования служба отправляет сообщение электронной почты с подробными сведениями, в случае ошибки — сообщение электронной почты с подробным описанием ошибки.](media/tutorial-control-flow/overview.png)

В этом учебнике показано, как выполнять следующие задачи.

> [!div class="checklist"]
> * Создание фабрики данных
> * Создание связанной службы хранилища Azure
> * Создание набора данных больших двоичных объектов Azure
> * Создание конвейера, содержащего действия копирования и веб-действие.
> * Отправка выходных данных действий для последующих действий.
> * Использование передачи параметров и системных переменных
> * Запуск конвейера.
> * Мониторинг конвейера и выполнения действий.

В этом руководстве используется пакет SDK для .NET. Для взаимодействия с фабрикой данных Azure можно использовать другие механизмы. Краткие руководства по Фабрике данных см. в [5-минутных руководствах по началу работы](./quickstart-create-data-factory-portal.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись хранения Azure. В этом руководстве в качестве источника будет использоваться хранилище BLOB-объектов. Если у вас нет учетной записи хранения Azure, ознакомьтесь с разделом [Создание учетной записи хранения](../storage/common/storage-account-create.md).
* Обозреватель хранилищ Azure Сведения об установке этого инструмента см. раздел [Обозреватель службы хранилища Azure](https://storageexplorer.com/).
* База данных SQL Azure. Вы используете базу данных как хранилище данных-приемник. Если у вас нет базы данных в службе "База данных SQL Azure", воспользуйтесь кратким руководством [Создание отдельной базы данных в Базе данных SQL Azure](../azure-sql/database/single-database-create-quickstart.md).
* приведенному. В этой статье используется Visual Studio 2019.
* Пакет Azure SDK для .NET. Загрузите и установите пакет [Azure SDK для .NET](https://azure.microsoft.com/downloads/).

Список регионов Azure, в которых в настоящее время доступна Фабрика Данных, см. в статье [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных и вычислительные мощности могут находиться в других регионах. Это могут быть как хранилища Azure, так базы данных SQL Azure. В качестве средства вычислений может быть HDInsight, используемый Фабрикой Данных.

Создайте приложение, как описано в разделе [Создание приложения Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal). Назначьте приложению роль **Участник**, следуя указаниям в той же статье. Вам потребуются несколько значений для последующих частей этого руководства, такие как **Application (client) ID** (Идентификатор приложения (клиента)) и **Directory (tenant) ID** (Идентификатор каталога (клиента)).

### <a name="create-a-blob-table"></a>Создание таблицы больших двоичных объектов

1. Откройте текстовый редактор. Скопируйте следующий текст и сохраните его локально в файл *input.txt*.

   ```
   Ethel|Berg
   Tamika|Walsh
   ```

1. Откройте обозреватель службы хранилища Azure. Разверните учетную запись хранения. Щелкните правой кнопкой мыши элемент **Blob Containers** (Контейнеры больших двоичных объектов) и выберите **Create Blob Container** (Создать контейнер BLOB-объектов).
1. Присвойте новому контейнеру имя *adfv2branch* и выберите **Передать**, чтобы добавить файл *input. txt* в контейнер.

## <a name="create-visual-studio-project"></a>Создание проекта Visual Studio<a name="create-visual-studio-project"></a>

Создайте консольное приложение .NET на C#.

1. Откройте Visual Studio и выберите **Создать проект**.
1. В окне **Создание проекта** выберите **Консольное приложение (.NET Framework)** для C# и нажмите кнопку **Далее**.
1. Назовите проект *ADFv2BranchTutorial*.
1. Выберите **.NET версии 4.5.2** или более поздней, а затем выберите команду **Создать**.

### <a name="install-nuget-packages"></a>Установка пакетов Nuget

1. Выберите **Инструменты** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.
1. В **консоли диспетчера пакетов** выполните следующие команды, чтобы установить пакеты. Дополнительные сведения см. в документации по пакету NuGet [Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/).

   ```powershell
   Install-Package Microsoft.Azure.Management.DataFactory
   Install-Package Microsoft.Azure.Management.ResourceManager -IncludePrerelease
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```

### <a name="create-a-data-factory-client"></a>Создание клиента фабрики данных

1. Откройте файл *Program.cs* и добавьте следующие инструкции:

   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Linq;
   using Microsoft.Rest;
   using Microsoft.Azure.Management.ResourceManager;
   using Microsoft.Azure.Management.DataFactory;
   using Microsoft.Azure.Management.DataFactory.Models;
   using Microsoft.IdentityModel.Clients.ActiveDirectory;
   ```

1. Добавьте эти статические переменные в класс `Program`. Замените заполнители своими значениями.

   ```csharp
   // Set variables
   static string tenantID = "<tenant ID>";
   static string applicationId = "<application ID>";
   static string authenticationKey = "<Authentication key for your application>";
   static string subscriptionId = "<Azure subscription ID>";
   static string resourceGroup = "<Azure resource group name>";

   static string region = "East US";
   static string dataFactoryName = "<Data factory name>";

   // Specify the source Azure Blob information
   static string storageAccount = "<Azure Storage account name>";
   static string storageKey = "<Azure Storage account key>";
   // confirm that you have the input.txt file placed in th input folder of the adfv2branch container.
   static string inputBlobPath = "adfv2branch/input";
   static string inputBlobName = "input.txt";
   static string outputBlobPath = "adfv2branch/output";
   static string emailReceiver = "<specify email address of the receiver>";

   static string storageLinkedServiceName = "AzureStorageLinkedService";
   static string blobSourceDatasetName = "SourceStorageDataset";
   static string blobSinkDatasetName = "SinkStorageDataset";
   static string pipelineName = "Adfv2TutorialBranchCopy";

   static string copyBlobActivity = "CopyBlobtoBlob";
   static string sendFailEmailActivity = "SendFailEmailActivity";
   static string sendSuccessEmailActivity = "SendSuccessEmailActivity";
   ```

1. Добавьте в метод `Main` следующий код. Этот код создает экземпляр класса `DataFactoryManagementClient`. Этот объект используется не только для создания фабрики данных, связанной службы, наборов данных и конвейера, но и для отслеживания подробностей выполнения конвейера.

   ```csharp
   // Authenticate and create a data factory management client
   var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
   ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
   AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
   ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
   var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };
   ```

### <a name="create-a-data-factory"></a>Создание фабрики данных

1. Добавьте метод `CreateOrUpdateDataFactory` в файл *Program.cs*:

   ```csharp
   static Factory CreateOrUpdateDataFactory(DataFactoryManagementClient client)
   {
       Console.WriteLine("Creating data factory " + dataFactoryName + "...");
       Factory resource = new Factory
       {
           Location = region
       };
       Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));

       Factory response;
       {
           response = client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, resource);
       }

       while (client.Factories.Get(resourceGroup, dataFactoryName).ProvisioningState == "PendingCreation")
       {
           System.Threading.Thread.Sleep(1000);
       }
       return response;
   }
   ```

1. Добавьте следующую строку, создающую фабрику данных, в метод `Main`.

   ```csharp
   Factory df = CreateOrUpdateDataFactory(client);
   ```

## <a name="create-an-azure-storage-linked-service"></a>Создание связанной службы хранилища Azure

1. Добавьте метод `StorageLinkedServiceDefinition` в файл *Program.cs*:

   ```csharp
   static LinkedServiceResource StorageLinkedServiceDefinition(DataFactoryManagementClient client)
   {
      Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");
      AzureStorageLinkedService storageLinkedService = new AzureStorageLinkedService
      {
          ConnectionString = new SecureString("DefaultEndpointsProtocol=https;AccountName=" + storageAccount + ";AccountKey=" + storageKey)
      };
      Console.WriteLine(SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings));
      LinkedServiceResource linkedService = new LinkedServiceResource(storageLinkedService, name:storageLinkedServiceName);
      return linkedService;
   }
   ```

1. Добавьте следующую строку, создающую связанную службу хранилища Azure, в метод `Main`.

   ```csharp
   client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
   ```

Дополнительные сведения о поддерживаемых свойствах и сведениях см. в разделе [Свойства связанной службы](connector-azure-blob-storage.md#linked-service-properties).

## <a name="create-datasets"></a>Создание наборов данных

В этом разделе создайте два набора данных: для источника и приемника.

### <a name="create-a-dataset-for-a-source-azure-blob"></a>Создание набора данных для исходного большого двоичного объекта Azure

Добавьте метод, создающий *набор данных BLOB-объекта Azure*. Дополнительные сведения о поддерживаемых свойствах и сведениях см. в разделе [Свойства набора данных BLOB-объектов Azure](connector-azure-blob-storage.md#dataset-properties).

Добавьте метод `SourceBlobDatasetDefinition` в файл *Program.cs*:

```csharp
static DatasetResource SourceBlobDatasetDefinition(DataFactoryManagementClient client)
{
    Console.WriteLine("Creating dataset " + blobSourceDatasetName + "...");
    AzureBlobDataset blobDataset = new AzureBlobDataset
    {
        FolderPath = new Expression { Value = "@pipeline().parameters.sourceBlobContainer" },
        FileName = inputBlobName,
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = storageLinkedServiceName
        }
    };
    Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
    DatasetResource dataset = new DatasetResource(blobDataset, name:blobSourceDatasetName);
    return dataset;
}
```

Задайте набор данных, представляющий исходные данные в большом двоичном объекте Azure. Этот набор данных большого двоичного объекта относится к связанной службе хранилища Azure, созданной на предыдущем шаге. Набор данных BLOB-объектов описывает расположение большого двоичного объекта, из которого производится копирование: *FolderPath* и *FileName*.

Обратите внимание на использование параметров *FolderPath*. `sourceBlobContainer` — это имя параметра. Выражение заменяется значениями, передаваемыми в конвейере. Синтаксис для определения параметров: `@pipeline().parameters.<parameterName>`

### <a name="create-a-dataset-for-a-sink-azure-blob"></a>Создание набора данных для большого двоичного объекта Azure приемника

1. Добавьте метод `SourceBlobDatasetDefinition` в файл *Program.cs*:

   ```csharp
   static DatasetResource SinkBlobDatasetDefinition(DataFactoryManagementClient client)
   {
       Console.WriteLine("Creating dataset " + blobSinkDatasetName + "...");
       AzureBlobDataset blobDataset = new AzureBlobDataset
       {
           FolderPath = new Expression { Value = "@pipeline().parameters.sinkBlobContainer" },
           LinkedServiceName = new LinkedServiceReference
           {
               ReferenceName = storageLinkedServiceName
           }
       };
       Console.WriteLine(SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings));
       DatasetResource dataset = new DatasetResource(blobDataset, name: blobSinkDatasetName);
       return dataset;
   }
   ```

1. Добавьте следующий код, создающий наборы данных-приемники и источники большого двоичного объекта Azure, в метод `Main`.

   ```csharp
   client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));

   client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));
   ```

## <a name="create-a-c-class-emailrequest"></a>Создание класса C# с именем EmailRequest

В проекте C# создайте класс с именем `EmailRequest`. Этот класс определяет свойства, которые конвейер отправляет в тексте запроса при отправке электронной почты. В этом руководстве конвейер отправляет четыре свойства из конвейера по адресу электронной почты:

* Message. Текст сообщения электронной почты. В случае успешного копирования это свойство содержит объем записанных данных. В случае сбоя копирования это свойство содержит сведения об ошибке.
* Имя фабрики данных. Имя фабрики данных
* Имя конвейера. Имя конвейера.
* Получатель. Переданный параметр. Это свойство указывает получателя сообщения электронной почты.

```csharp
    class EmailRequest
    {
        [Newtonsoft.Json.JsonProperty(PropertyName = "message")]
        public string message;

        [Newtonsoft.Json.JsonProperty(PropertyName = "dataFactoryName")]
        public string dataFactoryName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "pipelineName")]
        public string pipelineName;

        [Newtonsoft.Json.JsonProperty(PropertyName = "receiver")]
        public string receiver;

        public EmailRequest(string input, string df, string pipeline, string receiverName)
        {
            message = input;
            dataFactoryName = df;
            pipelineName = pipeline;
            receiver = receiverName;
        }
    }
```

## <a name="create-email-workflow-endpoints"></a>Создание конечных точек рабочего процесса электронной почты

Чтобы инициировать отправку сообщения электронной почты, используйте [Logic Apps](../logic-apps/logic-apps-overview.md) для определения рабочего процесса. Сведения о создании рабочего процесса приложения логики см. в статье [Создание первого рабочего процесса приложения логики для автоматизации процессов между облачными приложениями и облачными службами](../logic-apps/quickstart-create-first-logic-app-workflow.md).

### <a name="success-email-workflow"></a>Рабочий процесс успешной отправки сообщения электронной почты

На [портале Azure](https://portal.azure.com) создайте рабочий процесс Logic Apps с именем *CopySuccessEmail*. Определите триггер рабочего процесса как `When an HTTP request is received`. Для триггера запроса укажите в `Request Body JSON Schema` следующий фрагмент кода JSON:

```json
{
    "properties": {
        "dataFactoryName": {
            "type": "string"
        },
        "message": {
            "type": "string"
        },
        "pipelineName": {
            "type": "string"
        },
        "receiver": {
            "type": "string"
        }
    },
    "type": "object"
}
```

Рабочий процесс выглядит примерно так, как показано в примере ниже:

![Рабочий процесс успешной отправки сообщения электронной почты](media/tutorial-control-flow/success-email-workflow-trigger.png)

Это содержимое JSON соответствует классу `EmailRequest`, созданному в предыдущем разделе.

Добавляет действие `Office 365 Outlook – Send an email`. Для действия **Отправка электронного сообщения** настройте способ форматирования электронного сообщения, используя свойства, переданные в **схеме** запроса текста JSON. Ниже приведен пример:

![Конструктор приложения логики. Действие отправки электронной почты](media/tutorial-control-flow/customize-send-email-action.png)

После сохранения рабочего процесса скопируйте и сохраните значение **HTTP POST URL** из триггера.

## <a name="fail-email-workflow"></a>Рабочий процесс сбоя отправки сообщения электронной почты

Клонируйте **CopySuccessEmail** как другой рабочий процесс Logic Apps с именем *CopyFailEmail*. В триггере запроса действие `Request Body JSON schema` такое же. Измените формат сообщения электронной почты, например поле `Subject`, чтобы создать другое сообщение на случай сбоя. Например:

![Конструктор приложения логики. Рабочий процесс сбоя отправки сообщения электронной почты](media/tutorial-control-flow/fail-email-workflow.png)

После сохранения рабочего процесса скопируйте и сохраните значение **HTTP POST URL** из триггера.

Теперь у вас должны быть два URL-адреса рабочего процесса, как в следующих примерах:

```csharp
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000

//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

## <a name="create-a-pipeline"></a>Создание конвейера

Вернитесь к проекту в Visual Studio. Теперь мы добавим код, создающий конвейер с действием копирования и свойством `DependsOn`. В этом руководстве конвейер содержит одно действие: операцию копирования, которая принимает набор данных большого двоичного объекта в качестве источника и другой набор данных большого двоичного объекта в качестве приемника. Если действие копирования завершается успешно или неудачно, оно вызывает другие задачи электронной почты.

В этом конвейере используются следующие функции:

* Параметры
* Веб-действие
* Зависимость действий
* Использование выходных данных действия в качестве входных данных для другого действия

1. Добавьте следующий метод в проект. В следующих разделах приведены дополнительные сведения.

    ```csharp
    static PipelineResource PipelineDefinition(DataFactoryManagementClient client)
            {
                Console.WriteLine("Creating pipeline " + pipelineName + "...");
                PipelineResource resource = new PipelineResource
                {
                    Parameters = new Dictionary<string, ParameterSpecification>
                    {
                        { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                        { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
                        { "receiver", new ParameterSpecification { Type = ParameterType.String } }

                    },
                    Activities = new List<Activity>
                    {
                        new CopyActivity
                        {
                            Name = copyBlobActivity,
                            Inputs = new List<DatasetReference>
                            {
                                new DatasetReference
                                {
                                    ReferenceName = blobSourceDatasetName
                                }
                            },
                            Outputs = new List<DatasetReference>
                            {
                                new DatasetReference
                                {
                                    ReferenceName = blobSinkDatasetName
                                }
                            },
                            Source = new BlobSource { },
                            Sink = new BlobSink { }
                        },
                        new WebActivity
                        {
                            Name = sendSuccessEmailActivity,
                            Method = WebActivityMethod.POST,
                            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/00000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000000",
                            Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                            DependsOn = new List<ActivityDependency>
                            {
                                new ActivityDependency
                                {
                                    Activity = copyBlobActivity,
                                    DependencyConditions = new List<String> { "Succeeded" }
                                }
                            }
                        },
                        new WebActivity
                        {
                            Name = sendFailEmailActivity,
                            Method =WebActivityMethod.POST,
                            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/000000000000000000000000000000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=0000000000000000000000000000000000000000000",
                            Body = new EmailRequest("@{activity('CopyBlobtoBlob').error.message}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
                            DependsOn = new List<ActivityDependency>
                            {
                                new ActivityDependency
                                {
                                    Activity = copyBlobActivity,
                                    DependencyConditions = new List<String> { "Failed" }
                                }
                            }
                        }
                    }
                };
                Console.WriteLine(SafeJsonConvert.SerializeObject(resource, client.SerializationSettings));
                return resource;
            }
    ```

1. Добавьте следующую строку, создающую конвейер, в метод `Main`:

   ```csharp
   client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));
   ```

### <a name="parameters"></a>Параметры

В первой части кода конвейера определяются параметры.

* `sourceBlobContainer`. Исходный набор данных больших двоичных объектов использует этот параметр в конвейере.
* `sinkBlobContainer`. Исходный набор данных для целевого BLOB-объекта использует этот параметр в конвейере.
* `receiver`. Этот параметр используется в двух веб-действиях в конвейере, которые отправляют получателю сообщения об успешном или неуспешном выполнении.

```csharp
Parameters = new Dictionary<string, ParameterSpecification>
    {
        { "sourceBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "sinkBlobContainer", new ParameterSpecification { Type = ParameterType.String } },
        { "receiver", new ParameterSpecification { Type = ParameterType.String } }
    },
```

### <a name="web-activity"></a>Веб-действие

Веб-действие разрешает выполнять вызов любой конечной точки REST. Дополнительные сведения о этом действии см. в разделе [Веб-действие в фабрике данных Azure](control-flow-web-activity.md). Этот конвейер использует веб-действие для вызова рабочего процесса электронной почты Logic Apps. Вы создаете два веб-действия: одно, которое вызывает рабочий процесс `CopySuccessEmail`, и другое, которое вызывает `CopyFailWorkFlow`.

```csharp
        new WebActivity
        {
            Name = sendCopyEmailActivity,
            Method = WebActivityMethod.POST,
            Url = "https://prodxxx.eastus.logic.azure.com:443/workflows/12345",
            Body = new EmailRequest("@{activity('CopyBlobtoBlob').output.dataWritten}", "@{pipeline().DataFactory}", "@{pipeline().Pipeline}", "@pipeline().parameters.receiver"),
            DependsOn = new List<ActivityDependency>
            {
                new ActivityDependency
                {
                    Activity = copyBlobActivity,
                    DependencyConditions = new List<String> { "Succeeded" }
                }
            }
        }
```

В свойстве `Url` вставьте конечные точки **HTTP POST URL** из рабочих процессов Logic Apps. В свойстве `Body` передайте экземпляр класса `EmailRequest`. Запрос сообщения электронной почты содержит следующие свойства.

* Message. Передает значение `@{activity('CopyBlobtoBlob').output.dataWritten`. Обращается к свойству предыдущего действия копирования и передает значение `dataWritten`. В случае сбоя передает выходные данные ошибки вместо `@{activity('CopyBlobtoBlob').error.message`.
* Имя Фабрики данных. Передает значение `@{pipeline().DataFactory}`. Эта системная переменная позволяет получить доступ к соответствующему имени фабрики данных. Список поддерживаемых системных переменных см. в статье [Системные переменные, поддерживаемые фабрикой данных Azure](control-flow-system-variables.md).
* Имя конвейера. Передает значение `@{pipeline().Pipeline}`. Эта системная переменная позволяет получить доступ к соответствующему имени конвейера.
* Получатель. Передает значение `"@pipeline().parameters.receiver"`. Получает доступ к параметрам конвейера.

Этот код создает новую зависимость действия, которая зависит от предыдущего действия копирования.

## <a name="create-a-pipeline-run"></a>Создание конвейера

Добавьте в метод `Main` следующий код, активирующий выполнение конвейера.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};

CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="main-class"></a>Класс Main

Итоговый метод `Main` должен выглядеть следующим образом.

```csharp
// Authenticate and create a data factory management client
var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
AuthenticationResult result = context.AcquireTokenAsync("https://management.azure.com/", cc).Result;
ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };

Factory df = CreateOrUpdateDataFactory(client);

client.LinkedServices.CreateOrUpdate(resourceGroup, dataFactoryName, storageLinkedServiceName, StorageLinkedServiceDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSourceDatasetName, SourceBlobDatasetDefinition(client));
client.Datasets.CreateOrUpdate(resourceGroup, dataFactoryName, blobSinkDatasetName, SinkBlobDatasetDefinition(client));

client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, PipelineDefinition(client));

Console.WriteLine("Creating pipeline run...");
Dictionary<string, object> arguments = new Dictionary<string, object>
{
    { "sourceBlobContainer", inputBlobPath },
    { "sinkBlobContainer", outputBlobPath },
    { "receiver", emailReceiver }
};

CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(resourceGroup, dataFactoryName, pipelineName, arguments).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

Создайте и запустите программу для запуска конвейера.

## <a name="monitor-a-pipeline-run"></a>Мониторинг выполнения конвейера

1. Добавьте в метод `Main` следующий код:

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(resourceGroup, dataFactoryName, runResponse.RunId);
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

    Этот код постоянно проверяет состояние запуска до тех пор, пока копирование данных не завершится.

1. Добавьте в метод `Main` следующий код, извлекающий сведения о выполнении действия копирования, например размер записанных и прочитанных данных.

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    List<ActivityRun> activityRuns = client.ActivityRuns.ListByPipelineRun(
    resourceGroup, dataFactoryName, runResponse.RunId, DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)).ToList();

    if (pipelineRun.Status == "Succeeded")
    {
        Console.WriteLine(activityRuns.First().Output);
        //SaveToJson(SafeJsonConvert.SerializeObject(activityRuns.First().Output, client.SerializationSettings), "ActivityRunResult.json", folderForJsons);
    }
    else
        Console.WriteLine(activityRuns.First().Error);

    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>Выполнение кода

Создайте и запустите приложение, а затем проверьте выполнение конвейера.

Приложение выведет ход выполнения создания фабрики данных, связанной службы, наборов данных, конвейера и выполнения конвейера. Затем она проверяет состояние выполнения конвейера. Дождитесь появления сведений о действии копирования с размером записанных и прочитанных данных. Затем воспользуйтесь такими средствами, как обозреватель службы хранилища Azure, чтобы проверить, скопирован ли большой двоичный объект в *outputBlobPath* из *inputBlobPath*, как указано в переменных.

Результат должен выглядеть так:

```json
Creating data factory DFTutorialTest...
{
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "type": "AzureStorage",
  "typeProperties": {
    "connectionString": "DefaultEndpointsProtocol=https;AccountName=***;AccountKey=***"
  }
}
Creating dataset SourceStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sourceBlobContainer"
    },
    "fileName": "input.txt"
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating dataset SinkStorageDataset...
{
  "type": "AzureBlob",
  "typeProperties": {
    "folderPath": {
      "type": "Expression",
      "value": "@pipeline().parameters.sinkBlobContainer"
    }
  },
  "linkedServiceName": {
    "type": "LinkedServiceReference",
    "referenceName": "AzureStorageLinkedService"
  }
}
Creating pipeline Adfv2TutorialBranchCopy...
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
            "type": "DatasetReference",
            "referenceName": "SourceStorageDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SinkStorageDataset"
          }
        ],
        "name": "CopyBlobtoBlob"
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').output.dataWritten}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendSuccessEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Succeeded"
            ]
          }
        ]
      },
      {
        "type": "WebActivity",
        "typeProperties": {
          "method": "POST",
          "url": "https://xxx.eastus.logic.azure.com:443/workflows/... ",
          "body": {
            "message": "@{activity('CopyBlobtoBlob').error.message}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
          }
        },
        "name": "SendFailEmailActivity",
        "dependsOn": [
          {
            "activity": "CopyBlobtoBlob",
            "dependencyConditions": [
              "Failed"
            ]
          }
        ]
      }
    ],
    "parameters": {
      "sourceBlobContainer": {
        "type": "String"
      },
      "sinkBlobContainer": {
        "type": "String"
      },
      "receiver": {
        "type": "String"
      }
    }
  }
}
Creating pipeline run...
Pipeline run ID: 00000000-0000-0000-0000-0000000000000
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 20,
  "dataWritten": 20,
  "copyDuration": 4,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
}
{}

Press any key to exit...
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве описано, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание фабрики данных
> * Создание связанной службы хранилища Azure
> * Создание набора данных больших двоичных объектов Azure
> * Создание конвейера, содержащего действия копирования и веб-действие.
> * Отправка выходных данных действий для последующих действий.
> * Использование передачи параметров и системных переменных
> * Запуск конвейера.
> * Мониторинг конвейера и выполнения действий.

Теперь вы можете перейти к разделу ключевых концепций, чтобы получить дополнительные сведения о фабрике данных Azure.
> [!div class="nextstepaction"]
>[Конвейеры и действия](concepts-pipelines-activities.md)