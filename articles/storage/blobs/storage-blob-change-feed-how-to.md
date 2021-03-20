---
title: Обработка веб-канала изменений в хранилище BLOB-объектов Azure | Документация Майкрософт
description: Узнайте, как обрабатывать журналы веб-каналов изменений в клиентском приложении .NET.
author: normesta
ms.author: normesta
ms.date: 09/08/2020
ms.topic: article
ms.service: storage
ms.subservice: blobs
ms.reviewer: sadodd
ms.custom: devx-track-csharp
ms.openlocfilehash: f0e89fdfba852fc056cf48efd1b92daabb272cf0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89568257"
---
# <a name="process-change-feed-in-azure-blob-storage"></a>Обработка веб-канала изменений в хранилище BLOB-объектов Azure

Веб-канал изменений предоставляет журналы транзакций всех изменений, происходящих в больших двоичных объектах и метаданных больших двоичных объектов в вашей учетной записи хранения. В этой статье показано, как считывать записи веб-канала изменений с помощью библиотеки обработчика канала изменений больших двоичных объектов.

Дополнительные сведения о веб-канале изменений см. в статье [веб-канал изменений в хранилище BLOB-объектов Azure](storage-blob-change-feed.md).

## <a name="get-the-blob-change-feed-processor-library"></a>Получение библиотеки обработчика канала изменений больших двоичных объектов

1. Откройте командное окно (например, Windows PowerShell).
2. В каталоге проекта установите [пакет NuGet **Azure. Storage. blobs. пр**](https://www.nuget.org/packages/Azure.Storage.Blobs.ChangeFeed/).

```console
dotnet add package Azure.Storage.Blobs --version 12.5.1
dotnet add package Azure.Storage.Blobs.ChangeFeed --version 12.0.0-preview.4
```
## <a name="read-records"></a>Чтение записей

> [!NOTE]
> Веб-канал изменений является неизменяемой сущностью и доступна только для чтения в вашей учетной записи хранения. Любое количество приложений может одновременно считывать и обрабатывать канал изменений, а также независимо от их удобства. Записи не удаляются из веб-канала изменений, когда приложение считывает их. Состояние чтения или итерации каждого потребляемого модуля чтения является независимым и поддерживается только приложением.

Этот пример выполняет перебор всех записей в веб-канале изменений, добавляет их в список, а затем возвращает этот список вызывающему объекту.
 
```csharp
public async Task<List<BlobChangeFeedEvent>> ChangeFeedAsync(string connectionString)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();

    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    // Get all the events in the change feed. 
    await foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedClient.GetChangesAsync())
    {
        changeFeedEvents.Add(changeFeedEvent);
    }

    return changeFeedEvents;
}
```

В этом примере на консоль выводятся несколько значений из каждой записи в списке. 

```csharp
public void showEventData(List<BlobChangeFeedEvent> changeFeedEvents)
{
    foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedEvents)
    {
        string subject = changeFeedEvent.Subject;
        string eventType = changeFeedEvent.EventType.ToString();
        string api = changeFeedEvent.EventData.Api;

        Console.WriteLine("Subject: " + subject + "\n" +
        "Event Type: " + eventType + "\n" +
        "Api: " + api);
    }
}
```

## <a name="resume-reading-records-from-a-saved-position"></a>Возобновление чтения записей из сохраненной должности

Вы можете сохранить свое расположение чтения в веб-канале изменений, а затем возобновить итерацию записей в будущем. Чтобы сохранить позицию чтения, можно получить курсор веб-канала изменений. Курсор является **строкой** , и приложение может сохранить эту строку любым способом, который имеет смысл в проектировании приложения (например, в файл или базу данных).

В этом примере выполняется перебор всех записей в веб-канале изменений, их добавление в список и сохранение курсора. Список и курсор возвращаются вызывающему. 

```csharp
public async Task<(string, List<BlobChangeFeedEvent>)> ChangeFeedResumeWithCursorAsync
    (string connectionString,  string cursor)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();
    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    IAsyncEnumerator<Page<BlobChangeFeedEvent>> enumerator = changeFeedClient
        .GetChangesAsync(continuation: cursor)
        .AsPages(pageSizeHint: 10)
        .GetAsyncEnumerator();

    await enumerator.MoveNextAsync();

    foreach (BlobChangeFeedEvent changeFeedEvent in enumerator.Current.Values)
    {
    
        changeFeedEvents.Add(changeFeedEvent);             
    }
    
    // Update the change feed cursor.  The cursor is not required to get each page of events,
    // it is intended to be saved and used to resume iterating at a later date.
    cursor = enumerator.Current.ContinuationToken;
    return (cursor, changeFeedEvents);
}
```

## <a name="stream-processing-of-records"></a>Потоковая обработка записей

Можно выбрать обработку записей веб-канала изменений по мере их фиксации в веб-канале изменений. См. [спецификации](storage-blob-change-feed.md#specifications). События изменения публикуются в веб-канале изменений в среднем в 60 секунд. Мы рекомендуем опрашивать новые изменения с учетом этого периода при указании интервала опроса.

В этом примере периодически опрашивается на наличие изменений.  Если записи изменений существуют, этот код обрабатывает эти записи и сохраняет курсор веб-канала изменений. Таким образом, если процесс был остановлен, а затем снова запущен, приложение может использовать курсор для возобновления обработки записей, в которых он остался последним. В этом примере курсор сохраняется в файле конфигурации локального приложения, но приложение может сохранить его в любой форме, которая наиболее подходит для вашего сценария. 

```csharp
public async Task ChangeFeedStreamAsync
    (string connectionString, int waitTimeMs, string cursor)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();

    while (true)
    {
        IAsyncEnumerator<Page<BlobChangeFeedEvent>> enumerator = changeFeedClient
        .GetChangesAsync(continuation: cursor).AsPages().GetAsyncEnumerator();

        while (true) 
        {
            var result = await enumerator.MoveNextAsync();

            if (result)
            {
                foreach (BlobChangeFeedEvent changeFeedEvent in enumerator.Current.Values)
                {
                    string subject = changeFeedEvent.Subject;
                    string eventType = changeFeedEvent.EventType.ToString();
                    string api = changeFeedEvent.EventData.Api;

                    Console.WriteLine("Subject: " + subject + "\n" +
                        "Event Type: " + eventType + "\n" +
                        "Api: " + api);
                }
            
                // helper method to save cursor. 
                SaveCursor(enumerator.Current.ContinuationToken);
            }
            else
            {
                break;
            }

        }
        await Task.Delay(waitTimeMs);
    }

}

public void SaveCursor(string cursor)
{
    System.Configuration.Configuration config = 
        ConfigurationManager.OpenExeConfiguration
        (ConfigurationUserLevel.None);

    config.AppSettings.Settings.Clear();
    config.AppSettings.Settings.Add("Cursor", cursor);
    config.Save(ConfigurationSaveMode.Modified);
}
```

## <a name="reading-records-within-a-time-range"></a>Чтение записей в диапазоне времени

Можно читать записи, попадающие в заданный диапазон времени. В этом примере выполняется итерация всех записей в веб-канале изменений, которые находятся в диапазоне от 3:00 марта 2 2020 г. и 2:00 AM 7 2020, добавляет их в список, а затем возвращает этот список вызывающему объекту.

### <a name="selecting-segments-for-a-time-range"></a>Выбор сегментов для диапазона времени

```csharp
public async Task<List<BlobChangeFeedEvent>> ChangeFeedBetweenDatesAsync(string connectionString)
{
    // Get a new blob service client.
    BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

    // Get a new change feed client.
    BlobChangeFeedClient changeFeedClient = blobServiceClient.GetChangeFeedClient();
    List<BlobChangeFeedEvent> changeFeedEvents = new List<BlobChangeFeedEvent>();

    // Create the start and end time.  The change feed client will round start time down to
    // the nearest hour, and round endTime up to the next hour if you provide DateTimeOffsets
    // with minutes and seconds.
    DateTimeOffset startTime = new DateTimeOffset(2020, 3, 2, 15, 0, 0, TimeSpan.Zero);
    DateTimeOffset endTime = new DateTimeOffset(2020, 8, 7, 2, 0, 0, TimeSpan.Zero);

    // You can also provide just a start or end time.
    await foreach (BlobChangeFeedEvent changeFeedEvent in changeFeedClient.GetChangesAsync(
        start: startTime,
        end: endTime))
    {
        changeFeedEvents.Add(changeFeedEvent);
    }

    return changeFeedEvents;
}
```

Предоставленное время начала округляется вниз до ближайшего часа, а время окончания округляется до ближайшего часа. Возможно, пользователи увидят события, произошедшие до времени начала и после времени окончания. Также возможно, что некоторые события, происходящие между временем начала и окончания, не будут отображаться. Это связано с тем, что события могут записываться в течение часа, предшествующего времени начала, или в течение часа после времени окончания.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о журналах веб-канала изменений. См. [веб-канал изменений в хранилище BLOB-объектов Azure](storage-blob-change-feed.md)
