---
title: Выходные данные из Azure Stream Analytics
description: В этой статье описываются параметры вывода данных, доступные для Azure Stream Analytics.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: conceptual
ms.custom: contperf-fy21q1
ms.date: 12/9/2020
ms.openlocfilehash: 3ce4f673657561e196520466b569d0cf83d75a8a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019352"
---
# <a name="outputs-from-azure-stream-analytics"></a>Выходные данные из Azure Stream Analytics

Задание Azure Stream Analytics состоит из входных данных, запроса и выходных данных. Существует несколько типов выходных данных, для которых можно отправить преобразованные данные. В этой статье перечислены поддерживаемые выходные данные Stream Analytics. При составлении запроса Stream Analytics укажите имя выходных данных с помощью [предложения INTO](/stream-analytics-query/into-azure-stream-analytics). Можно использовать один выход для каждого задания или несколько выходных данных для задания потоковой передачи (если они необходимы), добавив несколько предложений INTO в запрос.

Для создания, изменения и проверки выходных данных задания Stream Analytics можно использовать [портал Azure](stream-analytics-quick-create-portal.md#configure-job-output), [Azure PowerShell](stream-analytics-quick-create-powershell.md#configure-output-to-the-job), [API .NET](/dotnet/api/microsoft.azure.management.streamanalytics.ioutputsoperations), [REST API](/rest/api/streamanalytics/) и [Visual Studio](stream-analytics-quick-create-vs.md).

Некоторые типы выходных данных поддерживают [секционирование](#partitioning), и [размер выходного пакета](#output-batch-size) изменяется для оптимизации пропускной способности. В следующей таблице приведены функции, поддерживаемые для каждого типа выходных данных.

| Тип выходных данных | Секционирование | Безопасность | 
|-------------|--------------|----------|
|[Azure Data Lake Storage 1-го поколения](azure-data-lake-storage-gen1-output.md)|Да|Пользователь Azure Active Directory </br> , Управляемое удостоверение|
|[База данных SQL Azure](sql-database-output.md)|Да, необязательно.|Проверка подлинности пользователя SQL </br> Управляемое удостоверение (предварительная версия)|
|[Azure Synapse Analytics](azure-synapse-analytics-output.md)|Да|Проверка подлинности пользователя SQL </br> Управляемое удостоверение (предварительная версия)|
|[Хранилище BLOB-объектов и Azure Data Lake Gen 2](blob-storage-azure-data-lake-gen2-output.md)|Да|Ключ доступа, </br> Управляемое удостоверение (предварительная версия)|
|[Центры событий Azure](event-hubs-output.md)|Да, необходимо задать ключевой столбец секции в выходной конфигурации.|Ключ доступа, </br> Управляемое удостоверение (предварительная версия)|
|[Power BI](power-bi-output.md)|Нет|Azure Active Directory пользователь, </br> Управляемое удостоверение|
|[Хранилище таблиц Azure](table-storage-output.md)|Да|Ключ учетной записи|
|[Очереди служебной шины Azure](service-bus-queues-output.md)|Да|Ключ доступа|
|[Разделы по служебной шине Azure](service-bus-topics-output.md)|Да|Ключ доступа|
|[Azure Cosmos DB](azure-cosmos-db-output.md)|Да|Ключ доступа|
|[Функции Azure](azure-functions-output.md)|Да|Ключ доступа|

## <a name="partitioning"></a>Секционирование

Stream Analytics поддерживает секции для всех выходных данных, за исключением Power BI. Дополнительные сведения о ключах разделов и количестве модулей записи вывода см. в статье, посвященной определенному типу выхода. Все выходные статьи связаны в предыдущем разделе.  

Кроме того, для более сложной настройки секций количество модулей записи вывода можно контролировать с помощью `INTO <partition count>` предложения (см.) [](/stream-analytics-query/into-azure-stream-analytics#into-shard-count)в запросе, что может быть полезным при достижении требуемой топологии задания. Если выходной адаптер не секционирован, недостаток данных в одной входной секции приведет к задержке до наступления времени задержки. В таком случае выходные данные объединяются в общий модуль записи, что может привести к возникновению узких мест в конвейере. Дополнительные сведения о политике допустимого интервала поступления с задержкой см. в статье [Рассмотрение порядка событий Azure Stream Analytics](./stream-analytics-time-handling.md).

## <a name="output-batch-size"></a>Размер выходного пакета

Все выходные данные поддерживают пакетную обработку, но только некоторые размеры пакета поддерживаются явным образом. Azure Stream Analytics использует пакеты переменного размера для обработки событий и записи выходных данных. Обычно модуль Stream Analytics не записывает сообщение по одному, и использует пакеты для повышения эффективности. Если интенсивность входящих и исходящих событий высока, Stream Analytics использует большие пакеты. При низкой интенсивности исходящего трафика используются пакеты меньшего размера, чтобы обеспечить низкую задержку.

## <a name="parquet-output-batching-window-properties"></a>Свойства окна пакетной обработки выходных данных Parquet

При использовании развертывания шаблона Azure Resource Manager или REST API в двух свойствах окна пакетной обработки:

1. *timeWindow*

   Максимальное время ожидания на пакет. Значение должно быть строкой TimeSpan. Например, "00:02:00" в течение двух минут. По истечении этого времени пакет записывается в выходные данные, даже если не будет выполнено требование минимальных строк. Значение по умолчанию — 1 минута, а максимально допустимое — 2 часа. Если у выходных данных большого двоичного объекта есть частота шаблона пути, время ожидания не может превышать диапазон времени раздела.

2. *сизевиндов*

   Количество минимальных строк на пакет. Для Parquet каждый пакет создает новый файл. Текущее значение по умолчанию — 2000 строк, а допустимое максимальное — 10 000 строк.

Эти свойства окна пакетной обработки поддерживаются только API версии **2017-04-01-Preview**. Ниже приведен пример полезных данных JSON для вызова REST API.

```json
"type": "stream",
      "serialization": {
        "type": "Parquet",
        "properties": {}
      },
      "timeWindow": "00:02:00",
      "sizeWindow": "2000",
      "datasource": {
        "type": "Microsoft.Storage/Blob",
        "properties": {
          "storageAccounts" : [
          {
            "accountName": "{accountName}",
            "accountKey": "{accountKey}",
          }
          ],
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>
> [Краткое руководство. по созданию задания Stream Analytics с помощью портала Azure](stream-analytics-quick-create-portal.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/
