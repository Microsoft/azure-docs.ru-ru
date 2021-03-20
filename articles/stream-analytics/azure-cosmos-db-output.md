---
title: Azure Cosmos DB выходные данные Azure Stream Analytics
description: В этой статье описывается, как выводить данные из Azure Stream Analytics в Azure Cosmos DB.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 9382ac8dc71d1ccb3a85dc0a7a027c8e99296cc6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98016583"
---
# <a name="azure-cosmos-db-output-from-azure-stream-analytics"></a>Azure Cosmos DB выходные данные Azure Stream Analytics

[Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/) — это глобально распределенная служба базы данных, предоставляющая гибкое масштабирование без ограничений по всему миру, расширенные возможности запросов и автоматическое индексирование по моделям данных, не зависящим от схемы. Дополнительные сведения о параметрах контейнера Azure Cosmos DB для Stream Analytics см. в статье [Stream Analytics с Azure Cosmos DB в качестве выходных данных](stream-analytics-documentdb-output.md).

Выходные данные Azure Cosmos DB из Stream Analytics в настоящее время недоступны в регионах Azure для Китая (21Vianet) и Azure для Германии (T-Systems International).

> [!Note]
> Azure Stream Analytics поддерживает подключение только к Azure Cosmos DB с помощью API SQL.
> Другие API Azure Cosmos DB в данный момент не поддерживаются. Если указать модулю Azure Stream Analytics учетные записи Azure Cosmos DB, созданные при помощи других API, это может привести к неправильному сохранению данных.

В следующей таблице описаны свойства для создания выходных данных Azure Cosmos DB.

| Имя свойства | Описание |
| --- | --- |
| Псевдоним выходных данных | Псевдоним для ссылки на эти выходные данные в запросе Stream Analytics. |
| Приемник | Azure Cosmos DB. |
| Вариант импорта | Выберите один из вариантов: **Select Cosmos DB from your subscription** (Выбрать Cosmos DB из своей подписки) или **Provide Cosmos DB settings manually** (Указать параметры Cosmos DB вручную).
| Идентификатор учетной записи | Имя или универсальный код ресурса (URI) конечной точки учетной записи Azure Cosmos DB. |
| Ключ учетной записи | Общедоступный ключ доступа к учетной записи Azure Cosmos DB. |
| База данных | Имя базы данных Azure Cosmos DB. |
| Имя контейнера | Имя используемого контейнера, который должен существовать в Cosmos DB. Пример  <br /><ul><li> _MyContainer_: Контейнер с именем "MyContainer" должен существовать.</li>|
| Идентификатор документа |Необязательный параметр. Имя поля в выходных событиях, используемое для указания первичного ключа, на котором основываются операции вставки или обновления.

## <a name="partitioning"></a>Секционирование

Ключ секции основан на предложении PARTITION BY в запросе. Количество модулей записи вывода соответствует входному секционированию для [полностью параллельных запросов](stream-analytics-scale-jobs.md). Stream Analytics преобразует ключ секции Cosmos DB вывода в строку. Например, если имеется ключ секции со значением 1 типа bigint, он преобразуется в «1» типа String. Это преобразование всегда происходит независимо от того, записывается ли свойство секции в Cosmos DB.

## <a name="output-batch-size"></a>Размер выходного пакета

Максимальный размер сообщения см. в разделе [ограничения Azure Cosmos DB](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-cosmos-db-limits). Размер пакета и частота записи корректируются динамически с учетом ответов Azure Cosmos DB. Предопределенные ограничения для Stream Analytics отсутствуют.

## <a name="next-steps"></a>Дальнейшие действия

* [Краткое руководство. по созданию задания Stream Analytics с помощью портала Azure](stream-analytics-quick-create-portal.md)
* [Краткое руководство. Создание задания Azure Stream Analytics с помощью Azure CLI](quick-create-azure-cli.md)
* [Краткое руководство. Создание задания Azure Stream Analytics с помощью шаблона ARM](quick-create-azure-resource-manager.md)
* [Краткое руководство. Создание задания Stream Analytics с помощью Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Краткое руководство. Создание задания Azure Stream Analytics с помощью Visual Studio](stream-analytics-quick-create-vs.md)
* [Краткое руководство. Создание задания Azure Stream Analytics в Visual Studio Code](quick-create-visual-studio-code.md)
