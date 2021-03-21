---
title: Поиск по CSV-BLOB-объектам
titleSuffix: Azure Cognitive Search
description: Извлеките и импортируйте CSV-файл из хранилища BLOB-объектов Azure с помощью режима синтаксического анализа delimitedText.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/01/2021
ms.openlocfilehash: d9633031ca8358ab0498c2e806b22e6c4ddd3eab
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99430485"
---
# <a name="how-to-index-csv-blobs-using-delimitedtext-parsing-mode-and-blob-indexers-in-azure-cognitive-search"></a>Как индексировать большие двоичные объекты CSV с помощью режима синтаксического анализа delimitedText и индексаторов BLOB-объектов в Azure Когнитивный поиск

[Индексатор больших двоичных объектов](search-howto-indexing-azure-blob-storage.md) Azure когнитивный Поиск предоставляет `delimitedText` режим синтаксического анализа для CSV-файлов, который обрабатывает каждую строку в CSV как отдельный документ поиска. Например, при наличии приведенного ниже текста с разделителями-запятыми в `delimitedText` индексе поиска будут содержаться два документа: 

```text
id, datePublished, tags
1, 2016-01-12, "azure-search,azure,cloud"
2, 2016-07-07, "cloud,mobile"
```

Без `delimitedText` режима синтаксического анализа все содержимое CSV-файла будет рассматриваться как один документ поиска.

При создании нескольких документов поиска из одного большого двоичного объекта обязательно ознакомьтесь с [индексацией больших двоичных объектов, чтобы получить несколько документов для поиска](search-howto-index-one-to-many-blobs.md) , чтобы понять, как работают назначения ключей документов. Индексатор больших двоичных объектов способен находить или формировать значения, однозначно определяющие каждый новый документ. В частности, он может создать транзитный `AzureSearch_DocumentKey` объект, который создается при анализе большого двоичного объекта на более мелкие части, где значение затем используется в качестве ключа документа поиска в индексе.

## <a name="setting-up-csv-indexing"></a>Настройка индексирования CSV

Чтобы индексировать большие двоичные объекты CSV, создайте или обновите определение индексатора с помощью `delimitedText` режима анализа для запроса на [Создание индексатора](/rest/api/searchservice/create-indexer) :

```http
{
  "name" : "my-csv-indexer",
  ... other indexer properties
  "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "firstLineContainsHeaders" : true } }
}
```

`firstLineContainsHeaders` указывает, что первая (непустая) строка каждого BLOB-объекта содержит заголовки.
Если большие двоичные объекты не содержат строку заголовка, заголовки следует указать в конфигурации индексатора. 

```http
"parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } } 
```

Можно настроить символ разделителя с помощью параметра конфигурации `delimitedTextDelimiter`. Пример:

```http
"parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextDelimiter" : "|" } }
```

> [!NOTE]
> Сейчас поддерживается только кодирование UTF-8. Если требуется поддержка других кодировок, проголосуйте за них на сайте [UserVoice](https://feedback.azure.com/forums/263029-azure-search).

> [!IMPORTANT]
> При использовании режима синтаксического анализа текста с разделителями в Azure Когнитивный поиск предполагается, что все большие двоичные объекты в источнике данных будут иметь формат CSV. Если необходима поддержка как CSV-объектов, так и других больших двоичных объектов в одном источнике данных, проголосуйте на [нашем сайте UserVoice](https://feedback.azure.com/forums/263029-azure-search).
>

## <a name="request-examples"></a>Примеры запросов

Чтобы свести все описанное выше вместе, ознакомьтесь с приведенными далее примерами данных. 

Источник данных: 

```http
POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
    "name" : "my-blob-datasource",
    "type" : "azureblob",
    "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
    "container" : { "name" : "my-container", "query" : "<optional, my-folder>" }
}   
```

Индексатор:

```http
POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
Content-Type: application/json
api-key: [admin key]

{
  "name" : "my-csv-indexer",
  "dataSourceName" : "my-blob-datasource",
  "targetIndexName" : "my-target-index",
  "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } }
}
```


