---
title: Управление активами и связанными сущностями с помощью пакета SDK служб мультимедиа для .NET
description: Описание способов управления активами и связанными сущностями с помощью пакета SDK служб мультимедиа для .NET.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 1bd8fd42-7306-463d-bfe5-f642802f1906
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: b859ff49edb495be15cdf76719c3aaaf1ac72a6e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103013777"
---
# <a name="managing-assets-and-related-entities-with-media-services-net-sdk"></a>Управление активами и связанными сущностями с помощью пакета SDK служб мультимедиа для .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-manage-entities.md)
> * [REST](media-services-rest-manage-entities.md)
> 
> 

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

В этой статье показано, как управлять сущностями служб мультимедиа с помощью .NET.

Начиная с 1 апреля 2017 г. все записи задания в вашей учетной записи и связанные с ней записи задач старше 90 дней будут автоматически удалены, даже если общее число записей не превышает значение максимальной квоты. Например, 1 апреля 2017 г. будет автоматически удалена любая запись задания в вашей учетной записи, созданная ранее 31 декабря 2016 г. Если необходимо архивировать данные задания или задачи, то можно использовать код, описанный в этой статье.

## <a name="prerequisites"></a>Предварительные требования

Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 

## <a name="get-an-asset-reference"></a>Получение ссылки на актив
Распространенной задачей является получение ссылки на существующий актив в службах мультимедиа. В следующем примере кода показано, как можно получить ссылку на ресурс из коллекции Assets в объекте контекста сервера на основе идентификатора ресурса. В следующем примере кода используется запрос LINQ для получения ссылки на существующий объект IAsset.

```csharp
    static IAsset GetAsset(string assetId)
    {
        // Use a LINQ Select query to get an asset.
        var assetInstance =
            from a in _context.Assets
            where a.Id == assetId
            select a;
        // Reference the asset as an IAsset.
        IAsset asset = assetInstance.FirstOrDefault();

        return asset;
    }
```

## <a name="list-all-assets"></a>Перечисление всех активов
По мере роста числа активов в хранилище полезно получать список всех активов. В следующем примере кода показано, как обойти всю коллекцию активов (Assets) в объекте контекста сервера. С каждым активом пример также записывает некоторые значения его свойств в консоль. Например, каждый актив может содержать множество файлов мультимедиа. Пример кода записывает все файлы, связанные с каждым активом.

```csharp
    static void ListAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IAsset asset in _context.Assets)
        {
            // Display the collection of assets.
            builder.AppendLine("");
            builder.AppendLine("******ASSET******");
            builder.AppendLine("Asset ID: " + asset.Id);
            builder.AppendLine("Name: " + asset.Name);
            builder.AppendLine("==============");
            builder.AppendLine("******ASSET FILES******");

            // Display the files associated with each asset. 
            foreach (IAssetFile fileItem in asset.AssetFiles)
            {
                builder.AppendLine("Name: " + fileItem.Name);
                builder.AppendLine("Size: " + fileItem.ContentFileSize);
                builder.AppendLine("==============");
            }
        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="get-a-job-reference"></a>Получение ссылки на задание

При работе с задачами обработки в коде служб мультимедиа часто требуется получить ссылку на существующее задание на основе идентификатора. В следующем примере кода показано, как получить ссылку на объект IJob из коллекции заданий.

Получение ссылки на задание может потребоваться при запуске длительного задания кодирования, когда необходимо проверять состояние задания в программном потоке. В таких случаях при возвращении метода из потока требуется получить обновленную ссылку на задание.

```csharp
    static IJob GetJob(string jobId)
    {
        // Use a Linq select query to get an updated 
        // reference by Id. 
        var jobInstance =
            from j in _context.Jobs
            where j.Id == jobId
            select j;
        // Return the job reference as an Ijob. 
        IJob job = jobInstance.FirstOrDefault();

        return job;
    }
```

## <a name="list-jobs-and-assets"></a>Перечисление заданий и активов
Важная связанная задача — перечисление активов с заданиями, с которыми они связаны, в службах мультимедиа. В следующем примере кода показано, как перечислить каждый объект IJob, а затем для каждого задания отображаются свойства задания, все связанные задачи, все входные активы и все выходные активы. Данный пример кода может быть полезен для множества других задач. Например, если требуется перечислить выходные активы из одного или нескольких заданий кодирования, которые выполнялись ранее, данный код показывает, как получить доступ к этим выходным активам. При наличии ссылки на выходной актив можно затем доставлять содержимое другим пользователям или приложениям путем загрузки или передачи URL-адресов. 

Дополнительные сведения о вариантах доставки ресурсов-контейнеров см. в статье [Доставка ресурсов-контейнеров с помощью пакета SDK служб мультимедиа для .NET](media-services-deliver-streaming-content.md).

```csharp
    // List all jobs on the server, and for each job, also list 
    // all tasks, all input assets, all output assets.

    static void ListJobsAndAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IJob job in _context.Jobs)
        {
            // Display the collection of jobs on the server.
            builder.AppendLine("");
            builder.AppendLine("******JOB*******");
            builder.AppendLine("Job ID: " + job.Id);
            builder.AppendLine("Name: " + job.Name);
            builder.AppendLine("State: " + job.State);
            builder.AppendLine("Order: " + job.Priority);
            builder.AppendLine("==============");


            // For each job, display the associated tasks (a job  
            // has one or more tasks). 
            builder.AppendLine("******TASKS*******");
            foreach (ITask task in job.Tasks)
            {
                builder.AppendLine("Task Id: " + task.Id);
                builder.AppendLine("Name: " + task.Name);
                builder.AppendLine("Progress: " + task.Progress);
                builder.AppendLine("Configuration: " + task.Configuration);
                if (task.ErrorDetails != null)
                {
                    builder.AppendLine("Error: " + task.ErrorDetails);
                }
                builder.AppendLine("==============");
            }

            // For each job, display the list of input media assets.
            builder.AppendLine("******JOB INPUT MEDIA ASSETS*******");
            foreach (IAsset inputAsset in job.InputMediaAssets)
            {

                if (inputAsset != null)
                {
                    builder.AppendLine("Input Asset Id: " + inputAsset.Id);
                    builder.AppendLine("Name: " + inputAsset.Name);
                    builder.AppendLine("==============");
                }
            }

            // For each job, display the list of output media assets.
            builder.AppendLine("******JOB OUTPUT MEDIA ASSETS*******");
            foreach (IAsset theAsset in job.OutputMediaAssets)
            {
                if (theAsset != null)
                {
                    builder.AppendLine("Output Asset Id: " + theAsset.Id);
                    builder.AppendLine("Name: " + theAsset.Name);
                    builder.AppendLine("==============");
                }
            }

        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="list-all-access-policies"></a>Перечисление всех политик доступа
В службах мультимедиа можно определить политику доступа для актива или его файлов. Политика доступа определяет разрешения для файла или актива (тип доступа и продолжительность). В коде служб мультимедиа политика доступа обычно определяется путем создания объекта IAccessPolicy и последующей его привязки к существующему активу. Затем создается объект ILocator, который позволяет предоставить прямой доступ к ресурсам в службах мультимедиа. Проект Visual Studio, прилагаемый к этой документации, содержит несколько примеров кода, которые показывают, как создавать и назначать политики доступа и указатели на активы.

В следующем примере кода показано, как перечислить все политики доступа на сервере, а также показаны связанные с каждой из них типы разрешений. Другой полезный способ просмотра политик доступа — получить список всех объектов ILocator на сервере, а затем для каждого указателя можно перечислить связанную политику доступа с помощью его свойства AccessPolicy.

```csharp
    static void ListAllPolicies()
    {
        foreach (IAccessPolicy policy in _context.AccessPolicies)
        {
            Console.WriteLine("");
            Console.WriteLine("Name:  " + policy.Name);
            Console.WriteLine("ID:  " + policy.Id);
            Console.WriteLine("Permissions: " + policy.Permissions);
            Console.WriteLine("==============");

        }
    }
```
    
## <a name="limit-access-policies"></a>Ограничение политик доступа 

>[!NOTE]
> Действует ограничение в 1 000 000 записей для разных политик AMS (например, для политики Locator или ContentKeyAuthorizationPolicy). Следует указывать один и тот же идентификатор политики, если вы используете те же дни, разрешения доступа и т. д. Например, политики для указателей, которые должны оставаться на месте в течение длительного времени (не политики передачи). 

Например, вы можете создать универсальный набор политик, используя следующий код, который будет выполняться в приложении только один раз. Идентификаторы можно записать в файл журнала для последующего использования:

```csharp
    double year = 365.25;
    double week = 7;
    IAccessPolicy policyYear = _context.AccessPolicies.Create("One Year", TimeSpan.FromDays(year), AccessPermissions.Read);
    IAccessPolicy policy100Year = _context.AccessPolicies.Create("Hundred Years", TimeSpan.FromDays(year * 100), AccessPermissions.Read);
    IAccessPolicy policyWeek = _context.AccessPolicies.Create("One Week", TimeSpan.FromDays(week), AccessPermissions.Read);

    Console.WriteLine("One year policy ID is: " + policyYear.Id);
    Console.WriteLine("100 year policy ID is: " + policy100Year.Id);
    Console.WriteLine("One week policy ID is: " + policyWeek.Id);
```

Затем вы можете использовать существующие идентификаторы в своем коде, как показано здесь:

```csharp
    const string policy1YearId = "nb:pid:UUID:2a4f0104-51a9-4078-ae26-c730f88d35cf";


    // Get the standard policy for 1 year read only
    var tempPolicyId = from b in _context.AccessPolicies
                       where b.Id == policy1YearId
                       select b;
    IAccessPolicy policy1Year = tempPolicyId.FirstOrDefault();

    // Get the existing asset
    var tempAsset = from a in _context.Assets
                where a.Id == assetID
                select a;
    IAsset asset = tempAsset.SingleOrDefault();

    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
        policy1Year,
        DateTime.UtcNow.AddMinutes(-5));
    Console.WriteLine("The locator base path is " + originLocator.BaseUri.ToString());
```

## <a name="list-all-locators"></a>Перечисление всех указателей
Указатель — это URL-адрес, предоставляющий прямой путь для доступа к активу, а также разрешения для этого актива, определенные связанной с указателем политикой доступа. Каждый актив может содержать коллекцию объектов ILocator, связанную с ним через его свойство Locators. Контекст сервера также содержит коллекцию Locators, содержащую все указатели.

В следующем примере кода перечисляются все указатели на сервере. Для каждого указателя отображается идентификатор (Id) связанного актива и политики доступа. Также отображаются тип разрешений, дата окончания срока действия и полный путь к активу.

Обратите внимание, что путь указателя к активу представляет собой только базовый URL-адрес актива. Чтобы создать прямой путь к отдельным файлам, которые может просматривать пользователь или приложения, в коде необходимо добавить к пути указателя конкретный файловый путь. Сведения о том, как это сделать, см. в статье [Доставка ресурсов-контейнеров с помощью пакета SDK служб мультимедиа для .NET](media-services-deliver-streaming-content.md).

```csharp
    static void ListAllLocators()
    {
        foreach (ILocator locator in _context.Locators)
        {
            Console.WriteLine("***********");
            Console.WriteLine("Locator Id: " + locator.Id);
            Console.WriteLine("Locator asset Id: " + locator.AssetId);
            Console.WriteLine("Locator access policy Id: " + locator.AccessPolicyId);
            Console.WriteLine("Access policy permissions: " + locator.AccessPolicy.Permissions);
            Console.WriteLine("Locator expiration: " + locator.ExpirationDateTime);
            // The locator path is the base or parent path (with included permissions) to access  
            // the media content of an asset. To create a full URL to a specific media file, take 
            // the locator path and then append a file name and info as needed.  
            Console.WriteLine("Locator base path: " + locator.Path);
            Console.WriteLine("");
        }
    }
```

## <a name="enumerating-through-large-collections-of-entities"></a>Перечисление больших коллекций сущностей
При запросе сущностей существует ограничение в 1000 сущностей, возвращаемых за один раз, так как в открытой версии 2 REST количество результатов запросов ограничено 1000. При перечислении больших коллекций сущностей необходимо использовать предложения "Пропустить" и "Принять". 

Следующая функция перебирает все задания для предоставленной учетной записи служб мультимедиа. Службы мультимедиа возвращают 1000 заданий в коллекции заданий. Эта функция использует предложения "Пропустить" и "Принять", чтобы убедиться в том, что все задания перечислены (при наличии более 1000 заданий в учетной записи).

```csharp
    static void ProcessJobs()
    {
        try
        {

            int skipSize = 0;
            int batchSize = 1000;
            int currentBatch = 0;

            while (true)
            {
                // Loop through all Jobs (1000 at a time) in the Media Services account
                IQueryable _jobsCollectionQuery = _context.Jobs.Skip(skipSize).Take(batchSize);
                foreach (IJob job in _jobsCollectionQuery)
                {
                    currentBatch++;
                    Console.WriteLine("Processing Job Id:" + job.Id);
                }

                if (currentBatch == batchSize)
                {
                    skipSize += batchSize;
                    currentBatch = 0;
                }
                else
                {
                    break;
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
    }
```

## <a name="delete-an-asset"></a>Удаление актива
В следующем примере удаляется актив.

```csharp
    static void DeleteAsset( IAsset asset)
    {
        // delete the asset
        asset.Delete();

        // Verify asset deletion
        if (GetAsset(asset.Id) == null)
            Console.WriteLine("Deleted the Asset");

    }
```

## <a name="delete-a-job"></a>Удаление задания
Чтобы удалить задание, необходимо проверить состояние задания, как указано в свойстве State. Можно удалить задания, которые завершены или отменены, в то время как задания, которые находятся в некоторых других состояниях (помещенные в очередь, запланированные или обрабатываемые), необходимо сначала отменить, а затем их можно будет удалить.

В следующем примере кода показан метод удаления задания, проверяющий состояния заданий и удаляющий их при завершенном или отмененном состоянии. Этот код зависит от содержащегося выше раздела данной статьи "Получение ссылки на задание".

```csharp
    static void DeleteJob(string jobId)
    {
        bool jobDeleted = false;

        while (!jobDeleted)
        {
            // Get an updated job reference.  
            IJob job = GetJob(jobId);

            // Check and handle various possible job states. You can 
            // only delete a job whose state is Finished, Error, or Canceled.   
            // You can cancel jobs that are Queued, Scheduled, or Processing,  
            // and then delete after they are canceled.
            switch (job.State)
            {
                case JobState.Finished:
                case JobState.Canceled:
                case JobState.Error:
                    // Job errors should already be logged by polling or event 
                    // handling methods such as CheckJobProgress or StateChanged.
                    // You can also call job.DeleteAsync to do async deletes.
                    job.Delete();
                    Console.WriteLine("Job has been deleted.");
                    jobDeleted = true;
                    break;
                case JobState.Canceling:
                    Console.WriteLine("Job is cancelling and will be deleted "
                        + "when finished.");
                    Console.WriteLine("Wait while job finishes canceling...");
                    Thread.Sleep(5000);
                    break;
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    job.Cancel();
                    Console.WriteLine("Job is scheduled or processing and will "
                        + "be deleted.");
                    break;
                default:
                    break;
            }

        }
    }
```


## <a name="delete-an-access-policy"></a>Удаление политики доступа
В следующем примере кода показано, как получить ссылку на политику доступа на основе идентификатора (Id) политики, а затем удалить эту политику.

```csharp
    static void DeleteAccessPolicy(string existingPolicyId)
    {
        // To delete a specific access policy, get a reference to the policy.  
        // based on the policy Id passed to the method.
        var policyInstance =
                from p in _context.AccessPolicies
                where p.Id == existingPolicyId
                select p;
        IAccessPolicy policy = policyInstance.FirstOrDefault();

        policy.Delete();

    }
```


## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
