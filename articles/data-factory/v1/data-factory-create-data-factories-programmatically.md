---
title: Создание конвейеров данных с помощью пакета SDK для Azure .NET
description: Узнайте, как программным способом создавать, отслеживать и контролировать фабрики данных Azure с помощью пакета SDK фабрики данных.
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: e67f924806909d55d17151c36f28f81312223b23
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104782781"
---
# <a name="create-monitor-and-manage-azure-data-factories-using-azure-data-factory-net-sdk"></a>Создание, отслеживание фабрик данных Azure и управление ими с помощью пакета SDK фабрики данных Azure для .NET
> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, ознакомьтесь с руководством по [применению действия копирования](../quickstart-create-data-factory-dot-net.md). 

## <a name="overview"></a>Обзор
Создание, отслеживание фабрик данных и управление ими программным способом с помощью пакета .NET SDK фабрики данных. Эта статья содержит пошаговое руководство по созданию образца консольного приложения .NET, которое будет создавать и отслеживать фабрику данных. 

> [!NOTE]
> В этой статье рассматриваются не все API-интерфейсы .NET фабрики данных. Полную документацию по API .NET для фабрики данных см. в [справочнике по API .NET фабрики данных](/dotnet/api/overview/azure/data-factory). 

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Visual Studio 2012, 2013 или 2015
* Скачанный и установленный [пакет SDK для Azure .NET](https://azure.microsoft.com/downloads/).
* Установите Azure PowerShell. Далее, чтобы установить Azure PowerShell на локальном компьютере, следуйте указаниям в разделе [Установка и настройка Azure PowerShell](/powershell/azure/) . С помощью Azure PowerShell вы создадите приложение Azure Active Directory.

### <a name="create-an-application-in-azure-active-directory"></a>Создание приложения в Azure Active Directory
Вы создадите приложение Azure Active Directory и субъект-службу для приложения, а затем назначите роль **Участник Data Factory** .

1. Запустите **PowerShell**.
2. Выполните следующую команду и введите имя пользователя и пароль, которые используются для входа на портал Azure.

    ```powershell
    Connect-AzAccount
    ```
3. Выполните следующую команду, чтобы просмотреть все подписки для этой учетной записи.

    ```powershell
    Get-AzSubscription
    ```
4. Выполните следующую команду, чтобы выбрать подписку, с которой вы собираетесь работать. Замените **&lt;NameOfAzureSubscription**&gt; именем своей подписки Azure.

    ```powershell
    Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
    ```

   > [!IMPORTANT]
   > Запишите значения **SubscriptionId** и **TenantId**, указанные в выходных данных этой команды.

5. Создайте группу ресурсов Azure с именем **ADFTutorialResourceGroup** , выполнив следующую команду в PowerShell.

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

    Если группа ресурсов уже есть, укажите, требуется или не требуется ее обновить (Y или N соответственно).

    Если вы используете другую группу ресурсов, укажите ее имя вместо ADFTutorialResourceGroup.
6. Создайте приложение Azure Active Directory.

    ```powershell
    $azureAdApplication = New-AzADApplication -DisplayName "ADFDotNetWalkthroughApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfdotnetwalkthroughapp.org/example" -Password "Pass@word1"
    ```

    Если возникнет следующая ошибка, укажите другой URL-адрес и запустите команду еще раз.
    
    ```powershell
    Another object with the same value for property identifierUris already exists.
    ```
7. Создайте субъект-службу AD.

    ```powershell
    New-AzADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId
    ```
8. Назначьте субъекту-службе роль **Участник Data Factory** .

    ```powershell
    New-AzRoleAssignment -RoleDefinitionName "Data Factory Contributor" -ServicePrincipalName $azureAdApplication.ApplicationId.Guid
    ```
9. Получите идентификатор приложения.

    ```powershell
    $azureAdApplication    
    ```
    Запишите идентификатор приложения (applicationID) из выходных данных.

Вы должны получить следующие четыре значения:

* Tenant ID
* Идентификатор подписки
* Идентификатор приложения
* пароль (указан в первой команде).

## <a name="walkthrough"></a>Пошаговое руководство
В этом пошаговом руководстве вы создадите фабрику данных с конвейером, который содержит действие копирования. Действие копирования копирует данные из папки в хранилище BLOB-объектов Azure в другую папку в том же хранилище. 

Действие копирования перемещает данные в фабрике данных Azure. Это действие выполняется с помощью глобально доступной службы, обеспечивающей безопасное, надежное и масштабируемое копирование данных между разными хранилищами. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

1. С помощью Visual Studio 2012, 2013 или 2015 создайте консольное приложение C# .NET.
   1. Запустите **Visual Studio** 2012, 2013 или 2015.
   2. В меню **Файл** выберите **Создать**, а затем **Проект**.
   3. Разверните раздел **Шаблоны** и выберите **Visual C#** . В этом пошаговом руководстве используется C#, но можно использовать любой язык .NET.
   4. Выберите **Консольное приложение** в списке типов проектов справа.
   5. В качестве имени введите **DataFactoryAPITestApp** .
   6. В качестве расположения укажите **C:\ADFGetStarted**.
   7. Чтобы создать проект, нажмите кнопку **ОК** .
2. Щелкните **Инструменты**, наведите указатель мыши на **Диспетчер пакетов NuGet** и щелкните **Консоль диспетчера пакетов**.
3. В **консоли диспетчера пакетов** выполните следующие действия.
   1. Выполните следующую команду, чтобы установить пакет фабрики данных: `Install-Package Microsoft.Azure.Management.DataFactories`
   2. Выполните следующую команду, чтобы установить пакет Azure Active Directory (используйте Active Directory API в коде): `Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.19.208020213`
4. Замените содержимое файла **App.config** в проекте следующим содержимым: 
    
    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <appSettings>
            <add key="ActiveDirectoryEndpoint" value="https://login.microsoftonline.com/" />
            <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
            <add key="WindowsManagementUri" value="https://management.core.windows.net/" />

            <add key="ApplicationId" value="your application ID" />
            <add key="Password" value="Password you used while creating the AAD application" />
            <add key="SubscriptionId" value= "Subscription ID" />
            <add key="ActiveDirectoryTenantId" value="Tenant ID" />
        </appSettings>
    </configuration>
    ```
5. В файле App.Config обновите значения **&lt; идентификатора &gt; приложения**, **&lt; пароля &gt;**, **&lt; идентификатора &gt; подписки** и **&lt; &gt; идентификатора клиента** собственными значениями.
6. Добавьте следующие инструкции **using** в файл **Program.cs** в проекте.

    ```csharp
    using System.Configuration;
    using System.Collections.ObjectModel;
    using System.Threading;
    using System.Threading.Tasks;

    using Microsoft.Azure;
    using Microsoft.Azure.Management.DataFactories;
    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Common.Models;

    using Microsoft.IdentityModel.Clients.ActiveDirectory;

    ```
6. Добавьте в метод **Main** приведенный ниже код, который создает экземпляр класса **DataPipelineManagementClient**. Этот объект используется не только для создания фабрики данных, связанной службы, входных и выходных наборов данных, а также конвейера, но и для отслеживания срезов набора данных при выполнении.

    ```csharp
    // create data factory management client

    //IMPORTANT: specify the name of Azure resource group here
    string resourceGroupName = "ADFTutorialResourceGroup";

    //IMPORTANT: the name of the data factory must be globally unique.
    // Therefore, update this value. For example:APITutorialFactory05122017
    string dataFactoryName = "APITutorialFactory";

    TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader().Result);

    Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
    ```

   > [!IMPORTANT]
   > Замените значение **resourceGroupName** именем своей группы ресурсов Azure. Для создания группы ресурсов можно использовать командлет [New-AzureResourceGroup](/powershell/module/az.resources/new-azresourcegroup) .
   >
   > Измените имя фабрики данных (dataFactoryName) на уникальное. чтобы оно было глобально уникальным. Ознакомьтесь с разделом [Фабрика данных — правила именования](data-factory-naming-rules.md) , чтобы узнать о правилах именования артефактов фабрики данных.
7. Добавьте следующий код, создающий **фабрику данных**, в метод **Main**.

    ```csharp
    // create a data factory
    Console.WriteLine("Creating a data factory");
    client.DataFactories.CreateOrUpdate(resourceGroupName,
        new DataFactoryCreateOrUpdateParameters()
        {
            DataFactory = new DataFactory()
            {
                Name = dataFactoryName,
                Location = "westus",
                Properties = new DataFactoryProperties()
            }
        }
    );
    ```
8. Добавьте следующий код, который создает **связанную службу хранилища Azure**, в метод **Main**.

   > [!IMPORTANT]
   > Замените значения **storageaccountname** и **accountkey** именем и ключом вашей учетной записи хранения Azure.

    ```csharp
    // create a linked service for input data store: Azure Storage
    Console.WriteLine("Creating Azure Storage linked service");
    client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new LinkedServiceCreateOrUpdateParameters()
        {
            LinkedService = new LinkedService()
            {
                Name = "AzureStorageLinkedService",
                Properties = new LinkedServiceProperties
                (
                    new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<accountkey>")
                )
            }
        }
    );
    ```
9. Добавьте следующий код, создающий **входные и выходные наборы данных**, в метод **Main**.

    **FolderPath** для входного большого двоичного объекта задан как **adftutorial/**, где **adftutorial** — это имя контейнера в хранилище BLOB-объектов. Если этот контейнер не существует в хранилище больших двоичных объектов Azure, создайте контейнер с именем **adftutorial** и отправьте в него текстовый файл.

    FolderPath для выходного большого двоичного объекта задан как **adftutorial/apifactoryoutput/{срез}**, где **срез** динамически рассчитывается на основе значения **SliceStart** (начальные дата и время каждого среза).

    ```csharp
    // create input and output datasets
    Console.WriteLine("Creating input and output datasets");
    string Dataset_Source = "DatasetBlobSource";
    string Dataset_Destination = "DatasetBlobDestination";
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Source,
            Properties = new DatasetProperties()
            {
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/",
                    FileName = "emp.txt"
                },
                External = true,
                Availability = new Availability()
                {
                    Frequency = SchedulePeriod.Hour,
                    Interval = 1,
                },
    
                Policy = new Policy()
                {
                    Validation = new ValidationPolicy()
                    {
                        MinimumRows = 1
                    }
                }
            }
        }
    });
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Destination,
            Properties = new DatasetProperties()
            {
    
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/apifactoryoutput/{Slice}",
                    PartitionedBy = new Collection<Partition>()
                    {
                        new Partition()
                        {
                            Name = "Slice",
                            Value = new DateTimePartitionValue()
                            {
                                Date = "SliceStart",
                                Format = "yyyyMMdd-HH"
                            }
                        }
                    }
                },
    
                Availability = new Availability()
                {
                    Frequency = SchedulePeriod.Hour,
                    Interval = 1,
                },
            }
        }
    });
    ```
10. Добавьте следующий код, который **создает и активирует конвейер**, в метод **Main**. В этом конвейере есть действие **CopyActivity**, принимающие **BlobSource** как источник и **BlobSink** как приемник.

    Действие копирования перемещает данные в фабрике данных Azure. Это действие выполняется с помощью глобально доступной службы, обеспечивающей безопасное, надежное и масштабируемое копирование данных между разными хранилищами. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

    ```csharp
    // create a pipeline
    Console.WriteLine("Creating a pipeline");
    DateTime PipelineActivePeriodStartTime = new DateTime(2014, 8, 9, 0, 0, 0, 0, DateTimeKind.Utc);
    DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
    string PipelineName = "PipelineBlobSample";
    
    client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new PipelineCreateOrUpdateParameters()
    {
        Pipeline = new Pipeline()
        {
            Name = PipelineName,
            Properties = new PipelineProperties()
            {
                Description = "Demo Pipeline for data transfer between blobs",
    
                // Initial value for pipeline's active period. With this, you won't need to set slice status
                Start = PipelineActivePeriodStartTime,
                End = PipelineActivePeriodEndTime,
    
                Activities = new List<Activity>()
                {
                    new Activity()
                    {
                        Name = "BlobToBlob",
                        Inputs = new List<ActivityInput>()
                        {
                            new ActivityInput()
                {
                                Name = Dataset_Source
                            }
                        },
                        Outputs = new List<ActivityOutput>()
                        {
                            new ActivityOutput()
                            {
                                Name = Dataset_Destination
                            }
                        },
                        TypeProperties = new CopyActivity()
                        {
                            Source = new BlobSource(),
                            Sink = new BlobSink()
                            {
                                WriteBatchSize = 10000,
                                WriteBatchTimeout = TimeSpan.FromMinutes(10)
                            }
                        }
                    }
    
                },
            }
        }
    });
    ```
12. Добавьте следующий код в метод **Main** , чтобы получить состояние среза данных выходного набора данных. В этом примере ожидается только один срез.

    ```csharp
    // Pulling status within a timeout threshold
    DateTime start = DateTime.Now;
    bool done = false;
    
    while (DateTime.Now - start < TimeSpan.FromMinutes(5) && !done)
    {
        Console.WriteLine("Pulling the slice status");
        // wait before the next status check
        Thread.Sleep(1000 * 12);
    
        var datalistResponse = client.DataSlices.List(resourceGroupName, dataFactoryName, Dataset_Destination,
            new DataSliceListParameters()
            {
                DataSliceRangeStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString(),
                DataSliceRangeEndTime = PipelineActivePeriodEndTime.ConvertToISO8601DateTimeString()
            });
    
        foreach (DataSlice slice in datalistResponse.DataSlices)
        {
            if (slice.State == DataSliceState.Failed || slice.State == DataSliceState.Ready)
            {
                Console.WriteLine("Slice execution is done with status: {0}", slice.State);
                done = true;
                break;
            }
            else
            {
                Console.WriteLine("Slice status is: {0}", slice.State);
            }
        }
    }
    ```
13. **(Необязательно.)** Добавьте в метод **Main** следующий код для получения сведений о выполнении для среза данных.

    ```csharp
    Console.WriteLine("Getting run details of a data slice");
    
    // give it a few minutes for the output slice to be ready
    Console.WriteLine("\nGive it a few minutes for the output slice to be ready and press any key.");
    Console.ReadKey();
    
    var datasliceRunListResponse = client.DataSliceRuns.List(
        resourceGroupName,
        dataFactoryName,
        Dataset_Destination,
        new DataSliceRunListParameters()
        {
            DataSliceStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString()
        });
    
    foreach (DataSliceRun run in datasliceRunListResponse.DataSliceRuns)
    {
        Console.WriteLine("Status: \t\t{0}", run.Status);
        Console.WriteLine("DataSliceStart: \t{0}", run.DataSliceStart);
        Console.WriteLine("DataSliceEnd: \t\t{0}", run.DataSliceEnd);
        Console.WriteLine("ActivityId: \t\t{0}", run.ActivityName);
        Console.WriteLine("ProcessingStartTime: \t{0}", run.ProcessingStartTime);
        Console.WriteLine("ProcessingEndTime: \t{0}", run.ProcessingEndTime);
        Console.WriteLine("ErrorMessage: \t{0}", run.ErrorMessage);
    }
    
    Console.WriteLine("\nPress any key to exit.");
    Console.ReadKey();
    ```
14. Добавьте следующий вспомогательный метод, используемый методом **Main**, в класс **Program**. Этот метод выводит диалоговое окно, которое позволяет ввести **имя пользователя** и **пароль**, используемые для входа на портал Azure.

    ```csharp
    public static async Task<string> GetAuthorizationHeader()
    {
        AuthenticationContext context = new AuthenticationContext(ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] + ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
        ClientCredential credential = new ClientCredential(
            ConfigurationManager.AppSettings["ApplicationId"],
            ConfigurationManager.AppSettings["Password"]);
        AuthenticationResult result = await context.AcquireTokenAsync(
            resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
            clientCredential: credential);

        if (result != null)
            return result.AccessToken;

        throw new InvalidOperationException("Failed to acquire token");
    }
    ```

15. В обозреватель решений разверните проект: **DataFactoryAPITestApp**, щелкните правой кнопкой мыши **ссылки** и выберите команду **Добавить ссылку**. Установите флажок для сборки `System.Configuration` и нажмите кнопку **ОК**.
15. Постройте консольное приложение. В меню щелкните **Собрать** и выберите **Собрать решение**.
16. Убедитесь, что как минимум один файл существует в контейнере adftutorial в хранилище больших двоичных объектов Azure. В противном случае создайте файл Emp.txt в блокноте со следующим содержимым и передайте его в контейнер adftutorial.

    ```
    John, Doe
    Jane, Doe
    ```
17. Запустите пример, щелкнув **Отладка** -> **Начать отладку** в меню. При появлении сообщения **Getting run details of a data slice** (Получение сведений о выполнении для среза данных) подождите несколько минут и нажмите клавишу **ВВОД**.
18. Перейдите на портал Azure и убедитесь, что фабрика данных **APITutorialFactory** создана с использованием следующих артефактов:
    * Связанная служба: **AzureStorageLinkedService**.
    * Набор данных: **DatasetBlobSource** и **DatasetBlobDestination**.
    * Конвейер: **PipelineBlobSample**.
19. Убедитесь, что выходной файл создан в папке **apifactoryoutput** в контейнере **adftutorial**.

## <a name="get-a-list-of-failed-data-slices"></a>Получение списка невыполненных срезов данных 

```csharp
// Parse the resource path
var ResourceGroupName = "ADFTutorialResourceGroup";
var DataFactoryName = "DataFactoryAPITestApp";

var parameters = new ActivityWindowsByDataFactoryListParameters(ResourceGroupName, DataFactoryName);
parameters.WindowState = "Failed";
var response = dataFactoryManagementClient.ActivityWindows.List(parameters);
do
{
    foreach (var activityWindow in response.ActivityWindowListResponseValue.ActivityWindows)
    {
        var row = string.Join(
            "\t",
            activityWindow.WindowStart.ToString(),
            activityWindow.WindowEnd.ToString(),
            activityWindow.RunStart.ToString(),
            activityWindow.RunEnd.ToString(),
            activityWindow.DataFactoryName,
            activityWindow.PipelineName,
            activityWindow.ActivityName,
            string.Join(",", activityWindow.OutputDatasets));
        Console.WriteLine(row);
    }

    if (response.NextLink != null)
    {
        response = dataFactoryManagementClient.ActivityWindows.ListNext(response.NextLink, parameters);
    }
    else
    {
        response = null;
    }
}
while (response != null);
```

## <a name="next-steps"></a>Следующие шаги
См. Следующий пример создания конвейера с помощью пакета SDK для .NET, который копирует данные из хранилища BLOB-объектов Azure в базу данных SQL Azure. 

- [Руководство. Создание конвейера с действием копирования с помощью API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
