---
title: Руководство по C#. Использование ИИ с большими двоичными объектами Azure
titleSuffix: Azure Cognitive Search
description: Узнайте, как выполнять извлечение текста и обработку естественного языка по содержимому хранилища BLOB-объектов, используя C# и пакет SDK .NET для Когнитивного поиска Azure.
manager: nitinme
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/23/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 6cbfa01788f1897264b4197798aa5879cf724db3
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104770960"
---
# <a name="tutorial-use-net-and-ai-to-generate-searchable-content-from-azure-blobs"></a>Руководство. Использование .NET и искусственного интеллекта для создания доступного для поиска содержимого на основе BLOB-объектов Azure

Если у вас есть неструктурированный текст или изображения в хранилище BLOB-объектов Azure, [конвейер обогащения с помощью искусственного интеллекта](cognitive-search-concept-intro.md) поможет извлекать информацию и создавать содержимое для сценариев полнотекстового поиска и интеллектуального анализа. 

В этом учебнике рассматривается следующее.

> [!div class="checklist"]
> * Настройка среды разработки
> * Определите конвейер с использованием возможностей OCR, определения языка и распознавания сущностей и ключевых фраз.
> * Выполнение конвейера, чтобы начать преобразования, а также создать и загрузить индекс поиска.
> * Изучите результаты с помощью полнотекстового поиска и расширенного синтаксиса запросов.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="overview"></a>Обзор

В этом руководстве используется C# и клиентская библиотека [**Azure.Search.Documents**](/dotnet/api/overview/azure/search.documents-readme) для создания источника данных, индекса, индексатора и набора навыков.

Индексатор подключается к контейнеру больших двоичных объектов, указанному в объекте источника данных, и отправляет все индексированное содержимое в существующий индекс поиска.

Набор навыков прикрепляется к индексатору. Он использует встроенные навыки корпорации Майкрософт для поиска и извлечения информации. К этапам конвейера относятся оптическое распознавание символов (OCR) на изображениях, определение языка в тексте, извлечение ключевых фраз и распознавание сущностей (организаций). Новые сведения, создаваемые конвейером, хранятся в новых полях индекса. После заполнения индекса вы можете использовать поля в запросах, аспектах и фильтрах.

## <a name="prerequisites"></a>Предварительные требования

* [Visual Studio](https://visualstudio.microsoft.com/downloads/)
* [Пакет NuGet Azure.Search.Documents 11.x](https://www.nuget.org/packages/Azure.Search.Documents) 
* [Хранилище Azure](https://azure.microsoft.com/services/storage/)
* [Когнитивный поиск Azure](https://azure.microsoft.com/services/search/)

> [!Note]
> Для выполнения инструкций из этого руководства вы можете использовать бесплатную версию службы поиска. В бесплатной версии вы можете использовать не более трех индексов, трех индексаторов и трех источников данных. В этом руководстве создается по одному объекту из каждой категории. Перед началом работы убедитесь, что у службы есть достаточно места, чтобы принять новые ресурсы.

## <a name="download-sample-data"></a>Загрузка примера данных

Пример данных состоит из 14 файлов содержимого смешанного типа, которые будут отправлены в хранилище BLOB-объектов Azure на более позднем этапе.

1. Откройте эту [папку OneDrive](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4) и щелкните **Скачать** вверху слева, чтобы скопировать файлы на локальный компьютер. 

1. Щелкните ZIP-файл правой кнопкой мыши и выберите **Извлечь все**. Доступно 14 файлов разных типов. В этом примере вам потребуются семь из них.

Вы также можете скачать исходный код для этого руководства. Исходный код находится в папке **tutorial-ai-enrichment/v11** в репозитории [azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples).

## <a name="1---create-services"></a>1\. Создание служб

В этом учебнике используются Когнитивный поиск Azure для индексирования и создания запросов, сервер Cognitive Services — для обогащения с помощью искусственного интеллекта и хранилище BLOB-объектов Azure — для предоставления данных. В этом учебнике остается свободное выделение 20 транзакций на индексатор в день в Cognitive Services, поэтому необходимо создать только службы поиска и хранения.

Желательно создать все службы в одном регионе и одной группе ресурсов, чтобы упростить взаимодействие и управление. В реальной системе учетная запись хранения Azure может находиться в любом регионе.

### <a name="start-with-azure-storage"></a>Служба хранилища Azure

1. [Войдите на портал Azure](https://portal.azure.com/) и щелкните **+ Создать ресурс**.

1. Выполните поиск по строке *учетная запись хранения* и выберите "Учетная запись хранения Microsoft".

   ![Создание учетной записи хранения](media/cognitive-search-tutorial-blob/storage-account.png "Создание учетной записи хранения")

1. На вкладке "Основные сведения" нужно ввести следующие сведения. Сохраните значения по умолчанию для остальных параметров.

   * **Группа ресурсов**. Вы можете выбрать существующую группу ресурсов или создать новую, но для всех служб это должна быть одна и та же группа для совместного управления.

   * **Имя учетной записи хранения.** Если вы ожидаете, что у вас будет несколько ресурсов одного типа, используйте имена, которые будут четко определять тип и регион, например *blobstoragewestus*. 

   * **Расположение.** Желательно выбрать то же расположение, которое использовалось для Когнитивного поиска Azure и Cognitive Services. При использовании одного расположения плата за трафик не взимается.

   * **Тип учетной записи.** Выберите значение по умолчанию *StorageV2 (общего назначения версии 2)* .

1. Щелкните **Просмотреть и создать**, чтобы создать службу.

1. По завершении процесса щелкните **Перейти к ресурсу** чтобы открыть страницу со сводкой.

1. Щелкните службу **Большие двоичные объекты**.

1. Щелкните **+ Контейнер**, чтобы создать контейнер, и присвойте ему имя *cog-search-demo*.

1. Выберите *cog-search-demo* и щелкните **Отправить**, чтобы открыть папку с сохраненными файлами для скачивания. Выберите все четырнадцать файлов и нажмите кнопку **ОК** для отправки.

   ![Отправка образцов файлов](media/cognitive-search-quickstart-blob/sample-data.png "Отправка образцов файлов")

1. Прежде чем выйти из службы хранилища Azure, получите строку подключения для создания подключения в Когнитивном поиске Azure. 

   1. Вернитесь на страницу обзора учетной записи хранения (в качестве примера мы использовали *blobstoragewestus*). 

   1. В области навигации слева выберите **Ключи доступа** и скопируйте одну из строк подключения. 

   Строка подключения — это URL-адрес в следующем формате:

      ```http
      DefaultEndpointsProtocol=https;AccountName=blobstoragewestus;AccountKey=<your account key>;EndpointSuffix=core.windows.net
      ```

1. Сохраните эту строку подключения в Блокноте. Она понадобится позже при настройке подключения к источнику данных.

### <a name="cognitive-services"></a>Cognitive Services

Обогащение ИИ основано на платформе Cognitive Services, которая включает службы "Анализ текста" для обработки естественного языка и "Компьютерное зрение" для обработки изображений. Если бы вы создавали реальный прототип или проект, на этом этапе нужно было бы создать Cognitive Services (в том же регионе, что и Когнитивный поиск Azure) для связывания с операциями индексирования.

Но для нашего примера подготовку ресурсов можно пропустить, так как Когнитивный поиск Azure может подключаться к Cognitive Services в фоновом режиме и предоставляет 20 бесплатных транзакций для каждого выполнения индексатора. Так как в этом руководстве используется 14 транзакций, будет достаточно выделения на уровне "Бесплатный". Для крупных проектов вам, скорее всего, нужно будет подготовить Cognitive Services на уровне S0 с оплатой по мере использования. См. сведения о [подключении Cognitive Services](cognitive-search-attach-cognitive-services.md).

### <a name="azure-cognitive-search"></a>Когнитивный поиск Azure

Третьим компонентом является Когнитивный поиск Azure, который можно [создать на портале](search-create-service-portal.md), либо можно [найти в существующую службу поиска](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) в подписке.

Для выполнения действий в этом пошаговом руководстве можно использовать уровень "Бесплатный". 

### <a name="copy-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Копирование ключа API и URL-адреса конечной точки для администрирования Когнитивного поиска Azure

Для взаимодействия со службой "Когнитивный поиск Azure" вам нужны URL-адрес службы и ключ доступа. Служба поиска создана с обоими элементами, поэтому если вы добавили службу "Когнитивный поиск Azure" в подписку, выполните следующие действия для получения необходимых сведений:

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

1. В разделе **Параметры** > **Ключи** скопируйте ключ администратора, чтобы получить полные права на службу. Существуют два взаимозаменяемых ключа администратора, предназначенных для обеспечения непрерывности бизнес-процессов на случай, если вам потребуется сменить один из них. Вы можете использовать первичный или вторичный ключ для выполнения запросов на добавление, изменение и удаление объектов.

   Получите также ключ запроса. Мы рекомендуем создавать запросы с доступом только для чтения.

   ![Получение имени службы, ключей запросов и администратора](media/search-get-started-javascript/service-name-and-keys.png)

Если есть действительный ключ, для каждого запроса устанавливаются отношения доверия между приложением, которое отправляет запрос, и службой, которая его обрабатывает.

## <a name="2---set-up-your-environment"></a>2\. Настройка среды

Сначала откройте Visual Studio и создайте новый проект консольного приложения, которое будет выполняться на базе .NET Core.

### <a name="install-azuresearchdocuments"></a>Установка Azure.Search.Documents

[Пакет SDK службы "Когнитивный поиск Azure" для .NET](/dotnet/api/overview/azure/search) содержит клиентскую библиотеку, которая позволяет управлять индексами, источниками данных, индексаторами и наборами навыков, а также отправлять документы, управлять ими и выполнять запросы, не вникая в детали HTTP и JSON. Эта клиентская библиотека распространяется как пакет NuGet.

Для этого проекта установите `Azure.Search.Documents` версии 11 или более поздней и `Microsoft.Extensions.Configuration` самой поздней версии.

1. В Visual Studio выберите **Сервис** > **Диспетчер пакетов NuGet** > **Управление пакетами NuGet для решения...**

1. Найдите [Azure.Search.Document](https://www.nuget.org/packages/Azure.Search.Documents).

1. Выберите самую последнюю версию и щелкните **Установить**.

1. Повторите предыдущие шаги для установки [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) и [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json).

### <a name="add-service-connection-information"></a>Добавление сведений о подключении службы

1. Щелкните правой кнопкой мыши проект в обозревателе решений и последовательно выберите **Добавить** > **Новый элемент...** . 

1. Присвойте файлу имя `appsettings.json` и нажмите **Добавить**. 

1. Добавьте файл в каталог выходных данных.
    1. Щелкните правой кнопкой мыши `appsettings.json` и выберите пункт **Свойства**. 
    1. Измените значение параметра **Копировать в выходной каталог** на **Копировать более новые**.

1. Скопируйте указанный ниже файл JSON в новый файл JSON.

    ```json
    {
      "SearchServiceUri": "Put your search service URI here",
      "SearchServiceAdminApiKey": "Put your primary or secondary API key here",
      "SearchServiceQueryApiKey": "Put your query API key here",
      "AzureBlobConnectionString": "Put your Azure Blob connection string here",
    }
    ```

Укажите службу поиска и добавьте сведения об учетной записи хранения BLOB-объектов. Помните, что эти сведения можно получить во время выполнения действий по подготовке службы из предыдущего раздела.

В поле **SearchServiceUri** введите полный URL-адрес.

### <a name="add-namespaces"></a>Добавление пространств имен

В файле `Program.cs` добавьте следующие пространства имен.

```csharp
using Azure;
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;
using Microsoft.Extensions.Configuration;
using System;
using System.Collections.Generic;
using System.Linq;

namespace EnrichwithAI
```

### <a name="create-a-client"></a>Создание клиента

Создайте экземпляр класса `SearchIndexClient` и `SearchIndexerClient` в разделе `Main`.

```csharp
public static void Main(string[] args)
{
    // Create service client
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    string searchServiceUri = configuration["SearchServiceUri"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];
    string cognitiveServicesKey = configuration["CognitiveServicesKey"];

    SearchIndexClient indexClient = new SearchIndexClient(new Uri(searchServiceUri), new AzureKeyCredential(adminApiKey));
    SearchIndexerClient indexerClient = new SearchIndexerClient(new Uri(searchServiceUri), new AzureKeyCredential(adminApiKey));
}
```

> [!NOTE]
> Клиенты подключатся к службе поиска. Во избежание открытия слишком большого числа подключений в приложении рекомендуется по возможности совместно использовать один экземпляр. Методы поддерживают такое использование, так как являются потокобезопасными.
> 

### <a name="add-function-to-exit-the-program-during-failure"></a>Добавление функции для выхода из программы во время сбоя

Этот учебник поможет вам разобраться в каждом этапе работы конвейера индексирования. Если произошла критическая ошибка, которая не позволяет программе создать источник данных, набор навыков, индекс или индексатор, программа выдаст сообщение об ошибке и выполнит выход, чтобы можно было разобраться в ее причине и устранить ее.

Добавьте `ExitProgram` в `Main`, чтобы обработать сценарии, требующие завершения работы программы.

```csharp
private static void ExitProgram(string message)
{
    Console.WriteLine("{0}", message);
    Console.WriteLine("Press any key to exit the program...");
    Console.ReadKey();
    Environment.Exit(0);
}
```

## <a name="3---create-the-pipeline"></a>3\. Создание конвейера

В Когнитивном поиске Azure обработка ИИ выполняется во время индексирования (или приема данных). В этой части пошагового руководства описано, как создать четыре объекта: источник данных, определение индекса, набор навыков и индексатор. 

### <a name="step-1-create-a-data-source"></a>Шаг 1. Создание источника данных

`SearchIndexerClient` имеет свойство [`DataSourceName`](/dotnet/api/azure.search.documents.indexes.models.searchindexer.datasourcename), которое можно задать для объекта `SearchIndexerDataSourceConnection`. Этот объект предоставляет все методы, которые требуются для создания, перечисления, обновления или удаления источников данных Когнитивного поиска Azure.

Вызовите метод `indexerClient.CreateOrUpdateDataSourceConnection(dataSource)`, чтобы создать экземпляр `SearchIndexerDataSourceConnection`. Следующий код создает источник данных типа `AzureBlob`.

```csharp
private static SearchIndexerDataSourceConnection CreateOrUpdateDataSource(SearchIndexerClient indexerClient, IConfigurationRoot configuration)
{
    SearchIndexerDataSourceConnection dataSource = new SearchIndexerDataSourceConnection(
        name: "demodata",
        type: SearchIndexerDataSourceType.AzureBlob,
        connectionString: configuration["AzureBlobConnectionString"],
        container: new SearchIndexerDataContainer("cog-search-demo"))
    {
        Description = "Demo files to demonstrate cognitive search capabilities."
    };

    // The data source does not need to be deleted if it was already created
    // since we are using the CreateOrUpdate method
    try
    {
        indexerClient.CreateOrUpdateDataSourceConnection(dataSource);
    }
    catch (Exception ex)
    {
        Console.WriteLine("Failed to create or update the data source\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without a data source");
    }

    return dataSource;
}
```

При успешном выполнении запроса метод вернет сведения о созданном источнике данных. Если при выполнении запроса возникнет ошибка, например из-за недопустимого параметра, метод вернет исключение.

Теперь добавьте строку в раздел `Main`, чтобы вызвать только что добавленную функцию `CreateOrUpdateDataSource`.

```csharp
// Create or Update the data source
Console.WriteLine("Creating or updating the data source...");
SearchIndexerDataSourceConnection dataSource = CreateOrUpdateDataSource(indexerClient, configuration);
```

Выполните сборку и запуск решения. Так как это ваш первый запрос, на портале Azure убедитесь, что источник данных был создан в службе "Когнитивный поиск Azure". На странице обзора службы поиска проверьте наличие нового элемента в списке "Источники данных". Возможно, вам придется подождать несколько минут, пока обновится страница портала.

  ![Плитка источников данных на портале](./media/cognitive-search-tutorial-blob/data-source-tile.png "Плитка источников данных на портале")

### <a name="step-2-create-a-skillset"></a>Шаг 2. Создание набора навыков

В рамках этого раздела вы определите набор требуемых шагов обогащения данных. Каждый шаг обогащения называется *навыком*, а набор шагов обогащения — *набором навыков*. В этом руководстве для набора навыков используются [встроенные когнитивные навыки](cognitive-search-predefined-skills.md):

* [Оптическое распознавание символов](cognitive-search-skill-ocr.md) для распознавания печатного и рукописного текста в файлах изображений.

* [Объединение текста](cognitive-search-skill-textmerger.md) для объединения текста из коллекции полей в одном поле.

* [Распознавание языка](cognitive-search-skill-language-detection.md) для определения языка содержимого.

* [Разделение текста](cognitive-search-skill-textsplit.md) для разбиения объемного содержимого на более мелкие фрагменты перед вызовом навыка извлечения ключевых фраз и навыка распознавания сущностей. Функции извлечения ключевых фраз и распознавания сущностей принимают входные данные объемом не более 50 000 символов. Некоторые примеры файлов следует разделить, чтобы удовлетворить это ограничение.

* [Распознавание сущностей](cognitive-search-skill-entity-recognition.md) для извлечения названий организаций из содержимого в контейнере больших двоичных объектов.

* [Извлечение ключевых фраз](cognitive-search-skill-keyphrases.md) для получения основных ключевых фраз.

В ходе первоначальной обработки Когнитивный поиск Azure открывает каждый документ для получения содержимого из разных форматов файлов. Текст из исходного файла помещается в созданное поле `content`, по одному для каждого документа. Поэтому задайте для входных данных `"/document/content"`, чтобы использовать этот текст. Содержимое изображения помещается в созданное поле `normalized_images`, указанное в наборе навыков как `/document/normalized_images/*`.

Выходные данные могут быть сопоставлены с индексом, используемым в качестве входных данных для нисходящего навыка, или к обоим, как в случае с кодом языка. В индексе код языка полезен для фильтрации. В качестве входных данных код языка используется навыками анализа текста, чтобы указать лингвистические правила для разбивки слов.

Общие сведения о наборах навыков см. в статье [How to create a skillset in an enrichment pipeline](cognitive-search-defining-skillset.md) (Способ создания набора навыков в конвейере обогащения).

### <a name="ocr-skill"></a>Навык OCR

Навык **оптического распознавания текста** позволяет извлечь текст из изображений. При использовании этого навыка предполагается наличие поля normalized_images. Чтобы создать это поле, далее в руководстве мы зададим для конфигурации ```"imageAction"``` в определении индексатора значение ```"generateNormalizedImages"```.

```csharp
private static OcrSkill CreateOcrSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("image")
    {
        Source = "/document/normalized_images/*"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("text")
    {
        TargetName = "text"
    });

    OcrSkill ocrSkill = new OcrSkill(inputMappings, outputMappings)
    {
        Description = "Extract text (plain and structured) from image",
        Context = "/document/normalized_images/*",
        DefaultLanguageCode = OcrSkillLanguage.En,
        ShouldDetectOrientation = true
    };

    return ocrSkill;
}
```

### <a name="merge-skill"></a>Навык объединения

В рамках этого раздела вы создадите навык **объединения**, который позволяет объединить поле содержимого документа с текстом, полученным с помощью навыка оптического распознавания текста.

```csharp
private static MergeSkill CreateMergeSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/content"
    });
    inputMappings.Add(new InputFieldMappingEntry("itemsToInsert")
    {
        Source = "/document/normalized_images/*/text"
    });
    inputMappings.Add(new InputFieldMappingEntry("offsets")
    {
        Source = "/document/normalized_images/*/contentOffset"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("mergedText")
    {
        TargetName = "merged_text"
    });

    MergeSkill mergeSkill = new MergeSkill(inputMappings, outputMappings)
    {
        Description = "Create merged_text which includes all the textual representation of each image inserted at the right location in the content field.",
        Context = "/document",
        InsertPreTag = " ",
        InsertPostTag = " "
    };

    return mergeSkill;
}
```

### <a name="language-detection-skill"></a>Навык распознавания языка

Навык **распознавания языка** определяет язык введенного текста и сообщает один код языка для каждого документа, отправленного по запросу. Мы будем использовать выходные данные навыка **распознавания языка** в качестве части входных данных для навыка **разделения текста**.

```csharp
private static LanguageDetectionSkill CreateLanguageDetectionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/merged_text"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("languageCode")
    {
        TargetName = "languageCode"
    });

    LanguageDetectionSkill languageDetectionSkill = new LanguageDetectionSkill(inputMappings, outputMappings)
    {
        Description = "Detect the language used in the document",
        Context = "/document"
    };

    return languageDetectionSkill;
}
```

### <a name="text-split-skill"></a>Навык разделения текста

Приведенный ниже код навыка **разделения текста** позволяет разделить текст на страницы и устанавливает для них ограничение в 4000 символов, измеренных методом `String.Length`. С помощью алгоритма текст будет при возможности разделен фрагменты, размер которых не превышает значение `maximumPageLength`. В этом случае будет сделано все возможное, чтобы разделить предложение на его границе. Поэтому размер фрагмента будет немного меньше, чем значение `maximumPageLength`.

```csharp
private static SplitSkill CreateSplitSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/merged_text"
    });
    inputMappings.Add(new InputFieldMappingEntry("languageCode")
    {
        Source = "/document/languageCode"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("textItems")
    {
        TargetName = "pages",
    });

    SplitSkill splitSkill = new SplitSkill(inputMappings, outputMappings)
    {
        Description = "Split content into pages",
        Context = "/document",
        TextSplitMode = TextSplitMode.Pages,
        MaximumPageLength = 4000,
        DefaultLanguageCode = SplitSkillLanguage.En
    };

    return splitSkill;
}
```

### <a name="entity-recognition-skill"></a>Навык распознавания сущностей

Этот экземпляр `EntityRecognitionSkill` настроен для распознавания типа категории `organization`. Навык **распознавания сущностей** также может распознать типы категорий `person` и `location`.

Обратите внимание, что в поле context задается значение ```"/document/pages/*"``` со звездочкой. Это означает, что этап обогащения вызывается для каждой страницы в ```"/document/pages"```.

```csharp
private static EntityRecognitionSkill CreateEntityRecognitionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/pages/*"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("organizations")
    {
        TargetName = "organizations"
    });

    EntityRecognitionSkill entityRecognitionSkill = new EntityRecognitionSkill(inputMappings, outputMappings)
    {
        Description = "Recognize organizations",
        Context = "/document/pages/*",
        DefaultLanguageCode = EntityRecognitionSkillLanguage.En
    };
    entityRecognitionSkill.Categories.Add(EntityCategory.Organization);

    return entityRecognitionSkill;
}
```

### <a name="key-phrase-extraction-skill"></a>Навык извлечения ключевых фраз

Как и созданный только что экземпляр `EntityRecognitionSkill`, навык **извлечения ключевых фраз** вызывается для каждой страницы документа.

```csharp
private static KeyPhraseExtractionSkill CreateKeyPhraseExtractionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry("text")
    {
        Source = "/document/pages/*"
    });
    inputMappings.Add(new InputFieldMappingEntry("languageCode")
    {
        Source = "/document/languageCode"
    });

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry("keyPhrases")
    {
        TargetName = "keyPhrases"
    });

    KeyPhraseExtractionSkill keyPhraseExtractionSkill = new KeyPhraseExtractionSkill(inputMappings, outputMappings)
    {
        Description = "Extract the key phrases",
        Context = "/document/pages/*",
        DefaultLanguageCode = KeyPhraseExtractionSkillLanguage.En
    };

    return keyPhraseExtractionSkill;
}
```

### <a name="build-and-create-the-skillset"></a>Сборка и создание наборов навыков

Выполните сборку `Skillset` с использованием созданных вами навыков.

```csharp
private static SearchIndexerSkillset CreateOrUpdateDemoSkillSet(SearchIndexerClient indexerClient, IList<SearchIndexerSkill> skills,string cognitiveServicesKey)
{
    SearchIndexerSkillset skillset = new SearchIndexerSkillset("demoskillset", skills)
    {
        Description = "Demo skillset",
        CognitiveServicesAccount = new CognitiveServicesAccountKey(cognitiveServicesKey)
    };

    // Create the skillset in your search service.
    // The skillset does not need to be deleted if it was already created
    // since we are using the CreateOrUpdate method
    try
    {
        indexerClient.CreateOrUpdateSkillset(skillset);
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to create the skillset\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without a skillset");
    }

    return skillset;
}
```

Добавьте в раздел `Main` следующие строки.

```csharp
// Create the skills
Console.WriteLine("Creating the skills...");
OcrSkill ocrSkill = CreateOcrSkill();
MergeSkill mergeSkill = CreateMergeSkill();
EntityRecognitionSkill entityRecognitionSkill = CreateEntityRecognitionSkill();
LanguageDetectionSkill languageDetectionSkill = CreateLanguageDetectionSkill();
SplitSkill splitSkill = CreateSplitSkill();
KeyPhraseExtractionSkill keyPhraseExtractionSkill = CreateKeyPhraseExtractionSkill();

// Create the skillset
Console.WriteLine("Creating or updating the skillset...");
List<SearchIndexerSkill> skills = new List<SearchIndexerSkill>();
skills.Add(ocrSkill);
skills.Add(mergeSkill);
skills.Add(languageDetectionSkill);
skills.Add(splitSkill);
skills.Add(entityRecognitionSkill);
skills.Add(keyPhraseExtractionSkill);

SearchIndexerSkillset skillset = CreateOrUpdateDemoSkillSet(indexerClient, skills, cognitiveServicesKey);
```

### <a name="step-3-create-an-index"></a>Шаг 3. Создание индекса

В этом разделе вы определите схему индекса, указав поля для включения в индекс поиска, а также атрибуты поиска для каждого поля. Поля имеют тип и могут принимать атрибуты, определяющие их использование (поиск, сортировка и т. д.). Имена полей в индексе не обязательно должны точно соответствовать именам полей в источнике. На более позднем этапе вы добавите сопоставления полей в индексаторе для подключения полей "источник — назначение". Для этого шага определите индекс, используя соглашения об именовании полей, относящиеся к вашему поисковому приложению.

В этом упражнении используются следующие поля и типы полей:

| Имена полей | Типы полей |
| --- | --- |
| `id` | Edm.String |
| `content` | Edm.String |
| `languageCode` | Edm.String |
| `keyPhrases` | List<Edm.String> |
| `organizations` | List<Edm.String> |

#### <a name="create-demoindex-class"></a>Создание класса DemoIndex

Поля для этого индекса определяются с помощью класса модели. Каждое свойство класса модели имеет атрибуты, которые определяют связанные с поиском характеристики соответствующего поля индекса. 

Мы добавим класс модели в новый файл C#. Щелкните проект правой кнопкой мыши и последовательно выберите **Добавить** > **Новый элемент...** . Выберите "Класс", присвойте файлу имя `DemoIndex.cs`, а затем щелкните **Добавить**.

Обязательно укажите, что нужно использовать типы из пространств имен `Azure.Search.Documents.Indexes` и `System.Text.Json.Serialization`.

Добавьте указанное ниже определение класса модели в файл `DemoIndex.cs` и включите его в то же пространство имен, где будете создавать индекс.

```csharp
using Azure.Search.Documents.Indexes;
using System.Text.Json.Serialization;

namespace EnrichwithAI
{
    // The SerializePropertyNamesAsCamelCase is currently unsupported as of this writing. 
    // Replace it with JsonPropertyName
    public class DemoIndex
    {
        [SearchableField(IsSortable = true, IsKey = true)]
        [JsonPropertyName("id")]
        public string Id { get; set; }

        [SearchableField]
        [JsonPropertyName("content")]
        public string Content { get; set; }

        [SearchableField]
        [JsonPropertyName("languageCode")]
        public string LanguageCode { get; set; }

        [SearchableField]
        [JsonPropertyName("keyPhrases")]
        public string[] KeyPhrases { get; set; }

        [SearchableField]
        [JsonPropertyName("organizations")]
        public string[] Organizations { get; set; }
    }
}
```

Теперь, когда вы определили класс модели, можно легко создать определение индекса в файле `Program.cs`. У этого индекса будет имя `demoindex`. Если индекс с таким именем уже существует, он будет удален.

```csharp
private static SearchIndex CreateDemoIndex(SearchIndexClient indexClient)
{
    FieldBuilder builder = new FieldBuilder();
    var index = new SearchIndex("demoindex")
    {
        Fields = builder.Build(typeof(DemoIndex))
    };

    try
    {
        indexClient.GetIndex(index.Name);
        indexClient.DeleteIndex(index.Name);
    }
    catch (RequestFailedException ex) when (ex.Status == 404)
    {
        //if the specified index not exist, 404 will be thrown.
    }

    try
    {
        indexClient.CreateIndex(index);
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to create the index\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without an index");
    }

    return index;
}
```

При тестировании вы можете обнаружить, что пытаетесь создать уже существующий индекс. Поэтому предварительно проверяйте, не создан ли он.

Добавьте в раздел `Main` следующие строки.

```csharp
// Create the index
Console.WriteLine("Creating the index...");
SearchIndex demoIndex = CreateDemoIndex(indexClient);
```

Добавьте следующую инструкцию using, чтобы устранить неоднозначность ссылки.

```csharp
using Index = Azure.Search.Documents.Indexes.Models;
```

Дополнительные сведения об основных понятиях индекса см. в статье [Создание индекса (REST API)](/rest/api/searchservice/create-index).

### <a name="step-4-create-and-run-an-indexer"></a>Шаг 4. Создание и запуск индексатора

Пока вы создали источник данных, набор навыков и индекс. Эти три компонента становятся частью [индексатора](search-indexer-overview.md), который объединяет каждую часть в единую многофазную операцию. Чтобы связать их вместе в индексаторе, необходимо определить сопоставления полей.

* Обработка FieldMappings выполняется перед набором навыков. Исходные поля из источника данных сопоставляются с целевыми полями в индексе. Если имена и типы полей одинаковы в исходном и целевом расположениях, сопоставление не требуется.

* Обработка OutputFieldMappings выполняется после набора навыков. Добавляются ссылки на элементы sourceFieldNames, которые не существуют, пока они не будут созданы при анализе или обогащении документов. TargetFieldName — это поле в индексе.

Помимо привязки входных данных к выходным данным, вы можете использовать сопоставления полей для преобразования структур данных в плоские структуры. Для получения дополнительных сведений см. раздел [Сопоставление обогащенных полей с индексом, поддерживающим поиск](cognitive-search-output-field-mapping.md).

```csharp
private static SearchIndexer CreateDemoIndexer(SearchIndexerClient indexerClient, SearchIndexerDataSourceConnection dataSource, SearchIndexerSkillset skillSet, SearchIndex index)
{
    IndexingParameters indexingParameters = new IndexingParameters()
    {
        MaxFailedItems = -1,
        MaxFailedItemsPerBatch = -1,
    };
    indexingParameters.Configuration.Add("dataToExtract", "contentAndMetadata");
    indexingParameters.Configuration.Add("imageAction", "generateNormalizedImages");

    SearchIndexer indexer = new SearchIndexer("demoindexer", dataSource.Name, index.Name)
    {
        Description = "Demo Indexer",
        SkillsetName = skillSet.Name,
        Parameters = indexingParameters
    };

    FieldMappingFunction mappingFunction = new FieldMappingFunction("base64Encode");
    mappingFunction.Parameters.Add("useHttpServerUtilityUrlTokenEncode", true);

    indexer.FieldMappings.Add(new FieldMapping("metadata_storage_path")
    {
        TargetFieldName = "id",
        MappingFunction = mappingFunction

    });
    indexer.FieldMappings.Add(new FieldMapping("content")
    {
        TargetFieldName = "content"
    });

    indexer.OutputFieldMappings.Add(new FieldMapping("/document/pages/*/organizations/*")
    {
        TargetFieldName = "organizations"
    });
    indexer.OutputFieldMappings.Add(new FieldMapping("/document/pages/*/keyPhrases/*")
    {
        TargetFieldName = "keyPhrases"
    });
    indexer.OutputFieldMappings.Add(new FieldMapping("/document/languageCode")
    {
        TargetFieldName = "languageCode"
    });

    try
    {
        indexerClient.GetIndexer(indexer.Name);
        indexerClient.DeleteIndexer(indexer.Name);
    }
    catch (RequestFailedException ex) when (ex.Status == 404)
    {
        //if the specified indexer not exist, 404 will be thrown.
    }

    try
    {
        indexerClient.CreateIndexer(indexer);
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to create the indexer\n Exception message: {0}\n", ex.Message);
        ExitProgram("Cannot continue without creating an indexer");
    }

    return indexer;
}
```

Добавьте в раздел `Main` следующие строки.

```csharp
// Create the indexer, map fields, and execute transformations
Console.WriteLine("Creating the indexer and executing the pipeline...");
SearchIndexer demoIndexer = CreateDemoIndexer(indexerClient, dataSource, skillset, demoIndex);
```

Обработка индексатора займет некоторое время. Несмотря на то что набор данных невелик, аналитические навыки выполняют интенсивные вычисления. Некоторые навыки, такие как анализ изображений, долго выполняются.

> [!TIP]
> Создание индексатора вызывает конвейер. Если есть проблемы с получением данных, сопоставлением входных и выходных данных или порядком операций, они появятся на этом этапе.

### <a name="explore-creating-the-indexer"></a>Подробнее о создании индексатора

В коде для `"maxFailedItems"` устанавливается значение -1, которое инструктирует механизм индексирования игнорировать ошибки во время импорта данных. Это полезно, потому что в демонстрационном источнике данных мало документов. Для большего источника данных необходимо установить значение больше 0.

Также обратите внимание, что для `"dataToExtract"` задано значение `"contentAndMetadata"`. Этот оператор указывает индексатору автоматически извлекать содержимое из разных форматов файлов, а также метаданные, относящиеся к каждому файлу.

Когда содержимое будет извлечено, вы можете установить `imageAction` для извлечения текста из изображений, найденных в источнике данных. `"imageAction"` с заданной конфигурацией `"generateNormalizedImages"` вместе с навыком распознавания текста и навыком объединения текста инструктирует индексатор извлекать текст из изображений (например слово "стоп" из знака остановки движения) и вставлять его как часть поля содержимого. Это относится как к изображениям, встроенным в документы (например, изображение внутри PDF-файлов), так и к изображениям, найденным в источнике данных, например к файлу JPG.

<a name="check-indexer-status"></a>

## <a name="4---monitor-indexing"></a>4\. Мониторинг индексирования

После того как индексатор будет определен, он будет выполняться автоматически при отправке запроса. В зависимости от того, какие когнитивные навыки вы определили, индексирование может занять больше времени, чем вы ожидаете. Определить, запущен ли еще индексатор, можно с помощью метода `GetStatus`.

```csharp
private static void CheckIndexerOverallStatus(SearchIndexerClient indexerClient, SearchIndexer indexer)
{
    try
    {
        var demoIndexerExecutionInfo = indexerClient.GetIndexerStatus(indexer.Name);

        switch (demoIndexerExecutionInfo.Value.Status)
        {
            case IndexerStatus.Error:
                ExitProgram("Indexer has error status. Check the Azure Portal to further understand the error.");
                break;
            case IndexerStatus.Running:
                Console.WriteLine("Indexer is running");
                break;
            case IndexerStatus.Unknown:
                Console.WriteLine("Indexer status is unknown");
                break;
            default:
                Console.WriteLine("No indexer information");
                break;
        }
    }
    catch (RequestFailedException ex)
    {
        Console.WriteLine("Failed to get indexer overall status\n Exception message: {0}\n", ex.Message);
    }
}
```

`demoIndexerExecutionInfo` представляет текущее состояние и журнал выполнения индексатора.

Предупреждения часто возникают с некоторыми исходными файлами и комбинациями навыков и не всегда указывают на проблему. В этом руководстве предупреждения являются неопасными (например, нет текстовых входных данных из файлов JPEG).

Добавьте в раздел `Main` следующие строки.

```csharp
// Check indexer overall status
Console.WriteLine("Check the indexer overall status...");
CheckIndexerOverallStatus(indexerClient, demoIndexer);
```

## <a name="5---search"></a>5\. Поиск

В консольных приложениях в руководствах по Когнитивному поиску Azure мы обычно добавляем двухсекундную задержку перед выполнением запросов, возвращающих результаты, но, поскольку обогащение занимает несколько минут, мы закроем консольное приложение и будем использовать другой подход.

Самый простой вариант — [обозреватель поиска](search-explorer.md) на портале. Сначала можно запустить пустой запрос, возвращающий все документы, или более целевой поиск, который возвращает содержимое нового поля, созданного конвейером. 

1. На портале Azure Microsoft на странице "Обзор" выберите **Индексы**.

1. Найдите в списке **`demoindex`** . Он должен содержать 14 документов. Если число документов равно нулю, то индексатор либо еще выполняется, либо страница пока не обновлялась. 

1. Выберите **`demoindex`**. Обозреватель поиска находится на первой вкладке.

1. Поиск в содержимом можно начать сразу после загрузки первого документа. Чтобы проверить наличие содержимого, выполните неопределенный запрос, нажав кнопку **Поиск**. Этот запрос возвращает все текущие проиндексированные документы, и вы получаете представление о том, что содержит индекс.

1. Затем вставьте следующую строку для получения более управляемых результатов: `search=*&$select=id, languageCode, organizations`

<a name="reset"></a>

## <a name="reset-and-rerun"></a>Сброс и повторный запуск

На ранних экспериментальных этапах разработки самый практичный подход к итерации схемы — удалить все объекты из службы "Когнитивный поиск Azure" и восстановить их с помощью кода. Имена ресурсов являются уникальными. Удаление объекта позволяет воссоздать его с использованием того же имени.

Пример кода для этого учебника проверяет имеющиеся объекты и удаляет их, чтобы вы могли повторно выполнить код. Для удаления индексов, индексаторов, источников данных и наборов навыков также можно использовать портал.

## <a name="takeaways"></a>Общие выводы

В этом руководстве описано, как создать расширенный конвейер индексирования путем создания таких компонентов, как источник данных, набор навыков, индекс и индексатор.

Вы получили сведения о [встроенных навыках](cognitive-search-predefined-skills.md), а также об определении набора навыков и механизме построения цепочек навыков путем сопоставления входных и выходных данных. Вы также узнали, что `outputFieldMappings` в определении индексатора требуется для маршрутизации обогащенных значений из конвейера в индекс для поиска в службе "Когнитивный поиск Azure".

Наконец, вы узнали, как тестировать результаты и выполнять сброс системы для дальнейших итераций. Вы узнали, что отправка запросов к индексу возвращает результат, созданный обогащенным конвейером индексирования. Вы также узнали, как проверить состояние индексатора, и какие объекты нужно удалить перед повторным запуском конвейера.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, после завершения проекта целесообразно удалить созданные ресурсы, которые вам больше не потребуются. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок "Все ресурсы" или "Группы ресурсов" в области навигации слева.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знакомы со всеми объектами в конвейере обогащения с помощью ИИ, давайте более подробно рассмотрим определения набора навыков и отдельные навыки.

> [!div class="nextstepaction"]
> [Создание набора навыков](cognitive-search-defining-skillset.md)