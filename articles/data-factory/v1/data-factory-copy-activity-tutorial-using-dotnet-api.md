---
title: 'Руководство по Создание конвейера с действием копирования с помощью API .NET '
description: В этом руководстве описано, как с помощью API .NET создать конвейер фабрики данных Azure с действием копирования.
author: linda33wj
ms.service: data-factory
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 8c2baf4da4e7893b650e1f885fb66be835c4b736
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100377080"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-net-api"></a>Руководство по Создание конвейера с действием копирования с помощью API .NET
> [!div class="op_single_selector"]
> * [Обзор и предварительные требования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Мастер копирования](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Шаблон Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API для .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, ознакомьтесь с руководством по [применению действия копирования](../quickstart-create-data-factory-dot-net.md). 

В этой статье показано, как создать фабрику данных c конвейером, который копирует данные из Хранилища BLOB-объектов Azure в Базу данных SQL Azure, с помощью [.NET API](https://portal.azure.com). Если вы еще не работали с фабрикой данных Azure, перед выполнением действий, описанных в этом руководстве, ознакомьтесь со статьей [Введение в фабрику данных Azure](data-factory-introduction.md).   

В этом руководстве описано, как создать конвейер с одним действием — действием копирования. Действие копирования копирует данные из поддерживаемого хранилища данных в поддерживаемое хранилище данных-приемник. Список хранилищ данных, которые поддерживаются в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Это действие выполняется с помощью глобально доступной службы, обеспечивающей безопасное, надежное и масштабируемое копирование данных между разными хранилищами. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

Конвейер может содержать сразу несколько действий. Два действия можно объединить в цепочку (выполнить одно действие вслед за другим), настроив выходной набор данных одного действия как входной набор данных другого действия. Дополнительные сведения см. в разделе [Несколько действий в конвейере](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline). 

> [!NOTE] 
> Полную документацию по .NET API для Data Factory см. в [Справочник по .NET API фабрики данных](/dotnet/api/overview/azure/data-factory).
> 
> В этом руководстве конвейер данных копирует данные из исходного хранилища данных в целевое. Инструкции по преобразованию данных с помощью Фабрики данных Azure см. в [руководстве по созданию конвейера для преобразования данных с использованием кластера Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Ознакомьтесь с [обзором руководства](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) и выполните **предварительные требования** .
* Visual Studio 2012, 2013 или 2015
* Скачайте и установите пакет [Azure .NET SDK](https://azure.microsoft.com/downloads/)
* Установите Azure PowerShell. Далее, чтобы установить Azure PowerShell на локальном компьютере, следуйте указаниям в разделе [Установка и настройка Azure PowerShell](/powershell/azure/install-Az-ps) . С помощью Azure PowerShell вы создадите приложение Azure Active Directory.

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
    $azureAdApplication = New-AzADApplication -DisplayName "ADFCopyTutotiralApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfcopytutorialapp.org/example" -Password "Pass@word1"
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
4. Добавьте следующий раздел **appSetttings** в файл **App.config**. Эти параметры используются вспомогательным методом **GetAuthorizationHeader**.

    Замените значения **&lt;Application ID&gt;** , **&lt;Password&gt;** , **&lt;Subscription ID&gt;** и **&lt;tenant ID&gt;** собственными значениями.

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

5. Добавьте следующие операторы **using** в файл исходного кода (Program.cs) в проекте.

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
    string resourceGroupName = "ADFTutorialResourceGroup";
    string dataFactoryName = "APITutorialFactory";

    TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader().Result);

    Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
    ```

   > [!IMPORTANT]
   > Замените значение **resourceGroupName** именем своей группы ресурсов Azure.
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

    Фабрика данных может иметь один или несколько конвейеров. Конвейер может содержать одно или несколько действий. Это может быть, например, действие копирования данных из исходного хранилища в целевое или действие HDInsight Hive для выполнения скрипта Hive, преобразующего входные данные в выходные данные продукта. Начнем с создания фабрики данных.
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

    Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В этом руководстве не используются службы вычислений, например Azure HDInsight или Azure Data Lake Analytics. Вы используете два хранилища данных — служба хранилища Azure (источник) и база данных SQL Azure (конечное хранилище). 

    Поэтому нужно создать две связанные службы с именами AzureStorageLinkedService и AzureSqlLinkedService и такими типами: AzureStorage и AzureSqlDatabase.  

    Связанная служба хранилища Azure связывает учетную запись хранения Azure с фабрикой данных. В этой учетной записи хранения вы создали контейнер и отправили в нее данные в ходе выполнения предварительных [требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
9. Добавьте следующий код, который создает **связанную службу SQL Azure**, в метод **Main**.

   > [!IMPORTANT]
   > Укажите вместо **servername**, **databasename**, **username** и **password** имя своего сервера, имя базы данных, имя пользователя и пароль.

    ```csharp
    // create a linked service for output data store: Azure SQL Database
    Console.WriteLine("Creating Azure SQL Database linked service");
    client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new LinkedServiceCreateOrUpdateParameters()
        {
            LinkedService = new LinkedService()
            {
                Name = "AzureSqlLinkedService",
                Properties = new LinkedServiceProperties
                (
                    new AzureSqlDatabaseLinkedService("Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30")
                )
            }
        }
    );
    ```

    Связанная служба SQL Azure (AzureSqlLinkedService) связывает Базу данных SQL Azure с фабрикой данных. В этой базе данных хранятся данные, скопированные из хранилища BLOB-объектов. Вы создали пустую таблицу в этой базе данных в ходе выполнения [предварительных требований](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).
10. Добавьте следующий код, создающий **входные и выходные наборы данных**, в метод **Main**.

    ```csharp
    // create input and output datasets
    Console.WriteLine("Creating input and output datasets");
    string Dataset_Source = "InputDataset";
    string Dataset_Destination = "OutputDataset";

    Console.WriteLine("Creating input dataset of type: Azure Blob");
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,

    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Source,
            Properties = new DatasetProperties()
            {
                Structure = new List<DataElement>()
                {
                    new DataElement() { Name = "FirstName", Type = "String" },
                    new DataElement() { Name = "LastName", Type = "String" }
                },
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

    Console.WriteLine("Creating output dataset of type: Azure SQL");
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new DatasetCreateOrUpdateParameters()
        {
            Dataset = new Dataset()
            {
                Name = Dataset_Destination,
                Properties = new DatasetProperties()
                {
                    Structure = new List<DataElement>()
                    {
                        new DataElement() { Name = "FirstName", Type = "String" },
                        new DataElement() { Name = "LastName", Type = "String" }
                    },
                    LinkedServiceName = "AzureSqlLinkedService",
                    TypeProperties = new AzureSqlTableDataset()
                    {
                        TableName = "emp"
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
    
    На предыдущем шаге вы создали связанные службы, связывающие учетную запись хранения Azure и Базу данных SQL Azure с фабрикой данных. На этом этапе вы определите два набора данных (InputDataset и OutputDataset), представляющие входные и выходные данные в хранилищах данных, на которые ссылаются службы AzureStorageLinkedService и AzureSqlLinkedService соответственно.

    Связанная служба хранилища Azure указывает строку подключения, которую фабрика данных использует во время выполнения, чтобы подключиться к учетной записи хранения Azure. А входной набор данных больших двоичных объектов (InputDataset) определяет контейнер и папку с входными данными.  

    Аналогичным образом связанная служба Базы данных SQL Azure указывает строку подключения, которую служба Фабрики данных использует во время выполнения, чтобы подключиться к базе данных. А выходной набор данных таблицы SQL (OututDataset) определяет таблицу в базе данных, в которую копируются данные из хранилища BLOB-объектов.

    На этом этапе вы создадите набор данных с именем InputDataset. Он указывает на файл большого двоичного объекта (emp.txt) в корневой папке контейнера больших двоичных объектов в службе хранилища Azure, которая представлена связанной службой AzureStorageLinkedService. Если не указать значение fileName (или пропустить его), данные из всех больших двоичных объектов в папке входных данных копируются в целевое расположение. В этом руководстве вы укажете значение параметра fileName.    

    На этом этапе мы создадим выходной набор данных с именем **OutputDataset**. Этот набор данных указывает на таблицу SQL в базе данных (представляется **AzureSqlLinkedService**).
11. Добавьте следующий код, который **создает и активирует конвейер**, в метод **Main**. На этом этапе вы создадите конвейер с **действием копирования**, которое использует **InputDataset** в качестве входных данных и **OutputDataset** в качестве выходных.

    ```csharp
    // create a pipeline
    Console.WriteLine("Creating a pipeline");
    DateTime PipelineActivePeriodStartTime = new DateTime(2017, 5, 11, 0, 0, 0, 0, DateTimeKind.Utc);
    DateTime PipelineActivePeriodEndTime = new DateTime(2017, 5, 12, 0, 0, 0, 0, DateTimeKind.Utc);
    string PipelineName = "ADFTutorialPipeline";

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
                            Name = "BlobToAzureSql",
                            Inputs = new List<ActivityInput>()
                            {
                                new ActivityInput() {
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
                    }
                }
            }
        });
    ```

    Обратите внимание на следующие моменты.
   
    - В разделе действий доступно только одно действие, параметр **type** которого имеет значение **Copy**. Дополнительные сведения о действии копирования см. в статье [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md). В решениях фабрики данных можно также использовать [действия преобразования данных](data-factory-data-transformation-activities.md).
    - Для этого действия параметру input присвоено значение **InputDataset**, а параметру output — значение **OutputDataset**. 
    - В разделе **typeProperties** в качестве типа источника указано **BlobSource**, а в качестве типа приемника — **SqlSink**. Список хранилищ данных, поддерживаемых действием копирования в качестве источников и приемников, см. в разделе [Поддерживаемые хранилища данных и форматы](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Чтобы узнать, как использовать конкретное хранилище данных в качестве источника или приемника, щелкните ссылку в таблице.  
   
    Сейчас на основе этого набора настраивается расписание. В этом руководстве выходной набор данных создает срез раз в час. Для конвейера настроено время начала и время окончания с разницей в сутки. Таким образом, конвейер создает 24 среза для выходного набора данных.
12. Добавьте следующий код в метод **Main** , чтобы получить состояние среза данных выходного набора данных. Существует только срез, ожидаемый в этом образце.

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

13. Добавьте в метод **Main** следующий код, чтобы получить сведения о выполнении для среза данных.

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
            }
        );

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

14. Добавьте следующий вспомогательный метод, используемый методом **Main**, в класс **Program**.

    > [!NOTE] 
    > При копировании и вставке следующего кода убедитесь, что скопированный код находится на том же уровне, что и метод Main.

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

15. В обозревателе решений разверните проект (DataFactoryAPITestApp), щелкните правой кнопкой мыши **Ссылки**, а затем выберите **Добавить ссылку**. Установите флажок для сборки **System.Configuration**. Затем нажмите кнопку **ОК**.
16. Постройте консольное приложение. В меню щелкните **Собрать** и выберите **Собрать решение**.
17. Убедитесь, что в контейнере **adftutorial** в хранилище BLOB-объектов Azure есть как минимум один файл. В противном случае создайте в блокноте файл **Emp.txt** со следующим содержимым, а затем отправьте его в контейнер adftutorial.

    ```
    John, Doe
    Jane, Doe
    ```
18. Запустите пример, щелкнув **Отладка** -> **Начать отладку** в меню. При появлении сообщения **Getting run details of a data slice** (Получение сведений о выполнении для среза данных) подождите несколько минут и нажмите клавишу **ВВОД**.
19. Перейдите на портал Azure и убедитесь, что фабрика данных **APITutorialFactory** создана с использованием следующих артефактов:
    * Связанная служба: **LinkedService_AzureStorage**.
    * Набор данных: **InputDataset** и **OutputDataset**.
    * Конвейер: **PipelineBlobSample**.
20. Убедитесь, что в таблице **emp** в указанной базе данных созданы две записи сотрудников.

## <a name="next-steps"></a>Дальнейшие действия
Полную документацию по .NET API для Data Factory см. в [Справочник по .NET API фабрики данных](/dotnet/api/overview/azure/data-factory).

В этом руководстве показано, как использовать Хранилище BLOB-объектов Azure и Базу данных SQL Azure как исходное и целевое хранилища данных в операции копирования. В следующей таблице приведен список хранилищ данных, которые поддерживаются в качестве источников и целевых расположений для действия копирования. 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Чтобы получить дополнительные сведения о том, как скопировать данные в хранилище данных или из него, щелкните ссылку для хранилища данных в таблице.

