---
title: Индексирование файлов мультимедиа с помощью индексатора мультимедийных данных Azure
description: Индексатор мультимедийных данных Azure позволяет искать содержимое файлов мультимедиа и создавать полнотекстовую транскрипцию для скрытых субтитров и ключевых слов. В этом разделе показано, как использовать индексатор мультимедийных данных.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 827a56b2-58a5-4044-8d5c-3e5356488271
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.reviewer: johndeu
ms.custom: devx-track-csharp
ms.openlocfilehash: c575904d994232726cf8d0d9152b02130fd29cea
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103013260"
---
# <a name="indexing-media-files-with-azure-media-indexer"></a>Индексирование файлов мультимедиа с помощью индексатора мультимедийных данных Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!IMPORTANT]
> Пользователям рекомендуется выполнять миграцию из индексатора версии 1 и индексатора v2 в режим " [службы мультимедиа v3 Аудиоанализерпресет Basic](../latest/analyzing-video-audio-files-concept.md)". Процессор [Azure Media indexer](media-services-index-content.md) media и [Azure Media indexer 2 Preview](./legacy-components.md) обработчики мультимедиа выводятся из эксплуатации. Даты прекращения поддержки см. в разделе, посвященном [устаревшим компонентам](legacy-components.md).

Индексатор мультимедийных данных Azure позволяет искать содержимое файлов мультимедиа и создавать полнотекстовую транскрипцию для скрытых субтитров и ключевых слов. Вы можете обработать один файл мультимедиа или несколько файлов мультимедиа в пакете.  

При индексировании содержимого обязательно используйте файлы мультимедиа, содержащие отчетливую речь (без фоновой музыки, шума, эффектов или шипения микрофона). Примерами подходящего содержимого могут служить записи собраний, лекций или презентаций. Для индексирования, как правило, не подходит такое содержимое, как фильмы, телепередачи, любые материалы со смешанными аудио- и звуковыми эффектами, записи плохого качества с фоновым шумом (шипением).

Задание индексирования может создавать следующие выходные данные:

* Файлы скрытых субтитров в следующих форматах: **TTML** и **WebVTT**.
  
    Файлы скрытых субтитров содержат тег Recognizability, который используется для оценки задания индексирования в зависимости от степени разборчивости речи в исходном видео.  Значение Recognizability можно использовать для отбора пригодных для использования выходных файлов. Низкая оценка означает, что результаты индексирования не точны из-за качества звука.
* Файл с ключевыми словами (XML).

В этой статье показано, как создавать задания для индексирования **ресурса** и **нескольких файлов**.

## <a name="using-configuration-and-manifest-files-for-indexing-tasks"></a>Использование файлов конфигурации и манифеста для задач индексирования
С помощью конфигурации задачи можно задать дополнительные параметры задач индексирования. Например, можно указать, какие метаданные будут использоваться для файла мультимедиа. Эти метаданные используются для пополнения словаря обработчика языка и существенно повышают точность распознавания речи.  Вы также можете указать нужные выходные файлы.

С помощью файла манифеста можно также обработать несколько файлов мультимедиа одновременно.

Подробнее этот вопрос раскрыт в [Предустановка задачи для индексатора мультимедийных данных Azure](./legacy-components.md).

## <a name="index-an-asset"></a>Индексирование ресурса
Следующий метод передает файл мультимедиа как ресурс и создает задание для индексирования ресурса.

Если файл конфигурации не указан, файл мультимедиа индексируется со всеми параметрами по умолчанию.

```csharp
    static bool RunIndexingJob(string inputMediaFilePath, string outputFolder, string configurationFile = "")
    {
        // Create an asset and upload the input media file to storage.
        IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
            "My Indexing Input Asset",
            AssetCreationOptions.None);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration from file if specified.
        string configuration = string.IsNullOrEmpty(configurationFile) ? "" : File.ReadAllText(configurationFile);

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset", AssetCreationOptions.None);

        // Use the following event handler to check job progress.  
        job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

        // Launch the job.
        job.Submit();

        // Check job execution and wait for job to finish.
        Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
        progressJobTask.Wait();

        // If job state is Error, the event handling
        // method for job progress should log errors.  Here we check
        // for error state and exit if needed.
        if (job.State == JobState.Error)
        {
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
    }

    static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
    {
        IAsset asset = _context.Assets.Create(assetName, options);

        var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
        assetFile.Upload(filePath);

        return asset;
    }

    static void DownloadAsset(IAsset asset, string outputDirectory)
    {
        foreach (IAssetFile file in asset.AssetFiles)
        {
            file.Download(Path.Combine(outputDirectory, file.Name));
        }
    }

    static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
    {
        var processor = _context.MediaProcessors
        .Where(p => p.Name == mediaProcessorName)
        .ToList()
        .OrderBy(p => new Version(p.Version))
        .LastOrDefault();

        if (processor == null)
            throw new ArgumentException(string.Format("Unknown media processor",
                                                       mediaProcessorName));

        return processor;
    }  
```

<!-- __ -->
### <a name="output-files"></a><a id="output_files"></a>выходные файлы
По умолчанию задание индексирования создает следующие выходные файлы. Файлы хранятся в первом выходном ресурсе.

При наличии нескольких входных файлов мультимедиа индексатор создает файл манифеста для выходных данных задания с именем JobResult.txt. Для каждого входного файла мультимедиа результирующие файлы TTML, WebVTT и ключевых слов последовательно нумеруются и именуются с помощью псевдонима.

| Имя файла | Описание |
| --- | --- |
| **InputFileName.ttml**<br/>**InputFileName.vtt** |Файлы скрытых титров (CC) в форматах TTML и WebVTT.<br/><br/>С их помощью аудио- и видеофайлы можно сделать доступным для людей с нарушениями слуха.<br/><br/>Файлы скрытых субтитров содержат тег <b>Recognizability</b>, который используется для оценки задания индексирования в зависимости от степени разборчивости речи в исходном видео.  Значение <b>Recognizability</b> позволяет оценить пригодность выходных файлов к использованию. Низкая оценка означает, что результаты индексирования не точны из-за качества звука. |
| **InputFileName.kw.xml<br/>InputFileName.info** |Файлы ключевых слов и сведений. <br/><br/>Файл ключевых слов — это XML-файл, содержащий ключевые слова, извлеченные из речевого содержимого вместе с информацией о частоте и смещении. <br/><br/>Файл сведений — это обычный текстовый файл, который содержит подробные сведения о каждом распознанном термине. Первая строка является особой и содержит коэффициент узнаваемости. Каждая следующая строка представляет собой следующие данные с разделителями табуляцией: начальное время, конечное время, слово или фраза, достоверность. Значения времени указаны в секундах, а значение достоверности принадлежит диапазону от 0 до 1. <br/><br/>Пример строки: "1.20 1.45 word 0.67". <br/><br/>Этот файл можно использовать для различных целей, например для анализа речи, улучшения обнаружения файлов мультимедиа такими поисковыми системами, как Bing, Google или Microsoft SharePoint, или даже для повышения релевантности рекламы. |
| **JobResult.txt** |Выходной манифест, который создается только при индексировании нескольких файлов. Он содержит следующую информацию:<br/><br/><table border="1"><tr><th>InputFile</th><th>Псевдоним</th><th>MediaLength</th><th>Error</th></tr><tr><td>a.mp4</td><td>Media_1</td><td>300</td><td>0</td></tr><tr><td>b.mp4</td><td>Media_2</td><td>0</td><td>3000</td></tr><tr><td>c.mp4</td><td>Media_3</td><td>600</td><td>0</td></tr></table><br/> |

Если не все входные файлы мультимедиа проиндексированы успешно, задание индексирования завершается ошибкой с кодом 4000. Подробнее этот вопрос раскрыт в [Коды ошибок](#error_codes).

## <a name="index-multiple-files"></a>индексирования нескольких файлов
Следующий метод передает несколько файлов мультимедиа в виде ресурса и создает задание для индексации этих файлов в пакете.

При этом создается файл манифеста с расширением .lst, который затем отправляется в ресурс. Файл манифеста содержит список всех файлов ресурсов. Подробнее этот вопрос раскрыт в [Предустановка задачи для индексатора мультимедийных данных Azure](./legacy-components.md).

```csharp
    static bool RunBatchIndexingJob(string[] inputMediaFiles, string outputFolder)
    {
        // Create an asset and upload to storage.
        IAsset asset = CreateAssetAndUploadMultipleFiles(inputMediaFiles,
            "My Indexing Input Asset - Batch Mode",
            AssetCreationOptions.None);

        // Create a manifest file that contains all the asset file names and upload to storage.
        string manifestFile = "input.lst";            
        File.WriteAllLines(manifestFile, asset.AssetFiles.Select(f => f.Name).ToArray());
        var assetFile = asset.AssetFiles.Create(Path.GetFileName(manifestFile));
        assetFile.Upload(manifestFile);

        // Declare a new job.
        IJob job = _context.Jobs.Create("My Indexing Job - Batch Mode");

        // Get a reference to the Azure Media Indexer.
        string MediaProcessorName = "Azure Media Indexer";
        IMediaProcessor processor = GetLatestMediaProcessorByName(MediaProcessorName);

        // Read configuration.
        string configuration = File.ReadAllText("batch.config");

        // Create a task with the encoding details, using a string preset.
        ITask task = job.Tasks.AddNew("My Indexing Task - Batch Mode",
            processor,
            configuration,
            TaskOptions.None);

        // Specify the input asset to be indexed.
        task.InputAssets.Add(asset);

        // Add an output asset to contain the results of the job.
        task.OutputAssets.AddNew("My Indexing Output Asset - Batch Mode", AssetCreationOptions.None);

        // Use the following event handler to check job progress.  
        job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

        // Launch the job.
        job.Submit();

        // Check job execution and wait for job to finish.
        Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
        progressJobTask.Wait();

        // If job state is Error, the event handling
        // method for job progress should log errors.  Here we check
        // for error state and exit if needed.
        if (job.State == JobState.Error)
        {
            Console.WriteLine("Exiting method due to job error.");
            return false;
        }

        // Download the job outputs.
        DownloadAsset(task.OutputAssets.First(), outputFolder);

        return true;
    }

    private static IAsset CreateAssetAndUploadMultipleFiles(string[] filePaths, string assetName, AssetCreationOptions options)
    {
        IAsset asset = _context.Assets.Create(assetName, options);

        foreach (string filePath in filePaths)
        {
            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);
        }

        return asset;
    }
```

### <a name="partially-succeeded-job"></a>Частично успешно выполненное задание
Если не все входные файлы мультимедиа проиндексированы успешно, задание индексирования завершится ошибкой с кодом 4000. Подробнее этот вопрос раскрыт в [Коды ошибок](#error_codes).

При этом создаются те же выходные данные (как и в случае успешно выполненного задания). Чтобы узнать, для каких входных файлов произошел сбой, можно просмотреть выходной файл манифеста (значения в столбце Error). Для неудачных входных файлов результирующие файлы TTML, WebVTT и keyword не будут формироваться.

### <a name="task-preset-for-azure-media-indexer"></a><a id="preset"></a> Предустановка задачи для индексатора мультимедийных данных Azure
Работу индексатора мультимедийных данных Azure можно настроить под свои нужды, приложив к заданию необязательный файл предустановки задачи.  Далее описан формат этого конфигурационного XML-файла.

| Имя | Require | Описание |
| --- | --- | --- |
| **input** |false |Файлы ресурсов-контейнеров, которые необходимо проиндексировать.</p><p>Azure Media Indexer поддерживает следующие форматы файлов мультимедиа: MP4, WMV, MP3, M4A, WMA, AAC, WAV.</p><p>Имена файлов можно указать в атрибуте **name** или **list** элемента **input** (как показано ниже). Если не указать файл ресурса-контейнера, который нужно проиндексировать, будет выбран основной файл. Если основной файл ресурса-контейнера не задан, индексируется первый файл входного ресурса-контейнера.</p><p>Чтобы явно указать имя файла ресурса-контейнера, используйте запись следующего формата:<br/>`<input name="TestFile.wmv">`<br/><br/>Вы также можете настроить индексирование сразу нескольких файлов ресурсов-контейнеров (до 10). Для этого выполните следующие действия.<br/><br/><ol class="ordered"><li><p>Создайте текстовый файл (файл манифеста) с расширением LST. </p></li><li><p>Внесите в этот файл манифеста список всех файлов входного ресурса-контейнера. </p></li><li><p>Добавьте (передайте) файл манифеста в ресурс.  </p></li><li><p>Укажите имя файла манифеста в атрибуте list элемента input.<br/>`<input list="input.lst">`</li></ol><br/><br/>Примечание. Если в файле манифеста указано более 10 файлов, задание индексирования завершится ошибкой с кодом 2006. |
| **метаданных** |false |Метаданные для указанного файла ресурса, используемые для адаптации словаря.  Это удобный способ подготовить индексатор к распознаванию слов, не входящих в стандартные словари, например имен собственных.<br/>`<metadata key="..." value="..."/>` <br/><br/>Вы можете указать значения **value** для поддерживаемых ключей **key**. В настоящее время поддерживаются следующие ключи.<br/><br/>title и description — позволяют адаптировать словарь к особенностям вашего задания и повысить точность распознавания речи.  По значениям ключей выполняется поиск в Интернете текстовых документов, соответствующих контексту. На время выполнения задания индексирования эти документы дополняют внутренний словарь индексатора.<br/>`<metadata key="title" value="[Title of the media file]" />`<br/>`<metadata key="description" value="[Description of the media file] />"` |
| **особенностей** <br/><br/>  Добавлено в версии 1.2. В настоящее время поддерживается только одна функция: распознавание речи (ASR). |false |Функция распознавания речи имеет следующие ключи настройки.<table><tr><th><p>Ключ</p></th>        <th><p>Описание</p></th><th><p>Пример значения</p></th></tr><tr><td><p>Язык</p></td><td><p>Естественный язык, распознаваемый в файле мультимедиа,</p></td><td><p>English, Spanish</p></td></tr><tr><td><p>CaptionFormats</p></td><td><p>Список предпочтительных форматов вывода (если таковые имеются), разделенных точкой с запятой.</p></td><td><p>TTML; WebVTT</p></td></tr><tr><td><p></p></td><td><p> </p></td><td><p>True; False</p></td></tr><tr><td><p>GenerateKeywords</p></td><td><p>Логический флаг. Если он установлен, создается XML-файл ключевых слов.</p></td><td><p>True; False </p></td></tr><tr><td><p>ForceFullCaption</p></td><td><p>Логический флаг. Если он установлен, всегда создаются полные подписи (независимо от уровня достоверности).  </p><p>По умолчанию имеет значение false, в этом случае слова и фразы с уровнем достоверности меньше 50 % в создаваемых подписях заменяются на многоточие ("...").  Многоточие позволяет контролировать качество текста.</p></td><td><p>True; False </p></td></tr></table> |

### <a name="error-codes"></a><a id="error_codes"></a>Коды ошибок
В случае ошибки индексатор мультимедийных данных Azure возвращает один из следующих кодов ошибки.

| Код | Имя | Возможные причины |
| --- | --- | --- |
| 2000 |Недопустимая конфигурация |Недопустимая конфигурация |
| 2001 |Недопустимые входные ресурсы |Отсутствие входных ресурсов или пустой ресурс. |
| 2002 |Недопустимый манифест |Манифест пуст или содержит недопустимые элементы. |
| 2003 |Не удалось загрузить файл мультимедиа |Недопустимый URL-адрес в файле манифеста. |
| 2004 |Неподдерживаемый протокол |Протокол URL-адреса мультимедиа не поддерживается. |
| 2005 |Неподдерживаемый тип файла |Тип файла входного мультимедиа не поддерживается. |
| 2006 |Слишком много входных файлов |Во входном манифесте указано более 10 файлов. |
| 3000 |Не удалось расшифровать файл мультимедиа |Кодек мультимедиа не поддерживается  <br/>или<br/> файл мультимедиа поврежден <br/>или<br/> Входной файл мультимедиа не содержит аудиопотока. |
| 4000 |Индексирование пакета выполнено частично |Некоторые файлы входного мультимедиа не удалось проиндексировать. Дополнительные сведения см. в статье <a href="#output_files">Выходные файлы</a>. |
| др. |Внутренние ошибки |Обратитесь в службу поддержки. indexer@microsoft.com |

## <a name="supported-languages"></a><a id="supported_languages"></a>Поддерживаемые языки
В настоящее время поддерживаются английский и испанский языки.  

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Связанные ссылки
[Общие сведения об аналитике служб мультимедиа Azure](./legacy-components.md)

[Индексирование файлов мультимедиа с помощью индексатора мультимедийных данных Azure 2 (предварительная версия)](./legacy-components.md)