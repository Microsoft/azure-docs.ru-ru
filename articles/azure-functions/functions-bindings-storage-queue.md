---
title: Обзор триггеров и привязок хранилища очередей Azure для функций Azure
description: Узнайте, как использовать триггер службы хранилища очередей Azure и выходную привязку в службе "Функции Azure".
author: craigshoemaker
ms.topic: reference
ms.date: 02/18/2020
ms.author: cshoe
ms.custom: cc996988-fb4f-47
ms.openlocfilehash: a1b9d03da29b7c89055303fa97fc38c2ef734b23
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100381483"
---
# <a name="azure-queue-storage-trigger-and-bindings-for-azure-functions-overview"></a>Обзор триггеров и привязок хранилища очередей Azure для функций Azure

Функции Azure могут выполняться по мере создания новых сообщений хранилища очередей Azure и могут записывать сообщения очереди в функцию.

| Действие | Type |
|---------|---------|
| Выполнение функции как изменений данных хранилища очередей | [Триггер](./functions-bindings-storage-queue-trigger.md) |
| Сообщения хранилища очереди записи |[Выходная привязка](./functions-bindings-storage-queue-output.md) |

## <a name="add-to-your-functions-app"></a>Добавление в приложение функций

### <a name="functions-2x-and-higher"></a>Функции 2.x и более поздних версий

Для работы с триггером и привязками требуется ссылка на соответствующий пакет. Пакет NuGet используется для библиотек классов .NET, в то время как набор расширений используется для всех других типов приложений.

| Язык                                        | Добавить по...                                   | Remarks 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Установка [пакета NuGet], версия 3. x | |
| Скрипт C#, Java, JavaScript, Python, PowerShell | Регистрация [пакета расширений]          | [Расширение "инструменты Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) " рекомендуется использовать с Visual Studio Code. |
| Скрипт C# (только в сети портал Azure)         | Добавление привязки                            | Чтобы обновить существующие расширения привязки без повторной публикации приложения функции, см. статью [Обновление расширений]. |

#### <a name="storage-extension-5x-and-higher"></a>Расширение хранилища 5. x и более поздние версии

Новая версия расширения привязок хранилища доступна в качестве [предварительной версии пакета NuGet](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.2). В этой предварительной версии вводится возможность [подключения с использованием удостоверения, а не секрета](./functions-reference.md#configure-an-identity-based-connection). Для приложений .NET также изменяются типы, к которым можно выполнить привязку, заменяя типы `WindowsAzure.Storage` и `Microsoft.Azure.Storage` с более новыми типами из [Azure. Storage. Queues](/dotnet/api/azure.storage.queues).

> [!NOTE]
> Предварительная версия пакета не включена в пакет расширений и должна быть установлена вручную. Для приложений .NET добавьте ссылку на пакет. Сведения о других типах приложений см. [в разделе Обновление расширений].

[core tools]: ./functions-run-local.md
[Пакет расширений]: ./functions-bindings-register.md#extension-bundles
[Пакет NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage
[Обновление расширений]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Функции 1.x

Функции 1. x автоматически имеют ссылку на пакет NuGet [Microsoft. Azure. веб-задания](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) , версия 2. x.

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

<a name="host-json"></a>  

## <a name="hostjson-settings"></a>Параметры файла host.json

В этом разделе описаны глобальные параметры конфигурации, доступные для этой привязки в версиях 2. x и более поздних. Пример *host.jsв* файле ниже содержит только параметры версии 2. x + для этой привязки. Дополнительные сведения о глобальных параметрах конфигурации в версиях 2. x и более поздних версий см. в разделе [host.jsв справочнике по функциям Azure](functions-host-json.md).

> [!NOTE]
> Чтобы получить дополнительные сведения о файле host.json в Функции 1.x, см. статью [host.json reference for Azure Functions 1.x](functions-host-json-v1.md)(Справочник по файлу host.json для службы "Функции Azure" версии 1.x.).

```json
{
    "version": "2.0",
    "extensions": {
        "queues": {
            "maxPollingInterval": "00:00:02",
            "visibilityTimeout" : "00:00:30",
            "batchSize": 16,
            "maxDequeueCount": 5,
            "newBatchThreshold": 8,
            "messageEncoding": "base64"
        }
    }
}
```

|Свойство  |По умолчанию | Описание |
|---------|---------|---------|
|maxPollingInterval|00:00:01|Максимальный интервал между опросами очереди. Минимальное значение — 00:00:00.100 (100 мс) и увеличивается до 00:01:00 (1 мин.).  В функциях 2. x и выше тип данных — `TimeSpan` , а в версии 1. x — в миллисекундах.|
|visibilityTimeout|00:00:00|Интервал времени между повторными попытками, когда при обработке сообщения возникает сбой. |
|batchSize|16|Количество сообщений очереди, которые среда выполнения функций одновременно получает и обрабатывает в параллельном режиме. Когда число обрабатываемых сообщений достигает `newBatchThreshold`, среда выполнения получает следующий пакет и начинает обработку содержащихся в нем сообщений. Поэтому максимальное количество сообщений, одновременно обрабатываемых каждой функцией, равно `batchSize` плюс `newBatchThreshold`. Это ограничение применяется отдельно к каждой функции, активируемой с помощью очереди. <br><br>Если вы не хотите, чтобы сообщения из одной очереди обрабатывались параллельно, можно установить для `batchSize` значение 1. Однако этот параметр исключает параллелизм, пока приложение-функция выполняется только на одной виртуальной машине. Если приложение-функция развернуто на нескольких виртуальных машинах, каждая машина может запускать один экземпляр каждой функции, активируемой с помощью очереди.<br><br>Максимальное значение `batchSize` — 32. |
|maxDequeueCount|5|Число повторных попыток обработки сообщения, прежде чем поместить его в очередь подозрительных сообщений.|
|newBatchThreshold|batchSize/2|Каждый раз, когда количество сообщений, обрабатываемых параллельно, достигает этого числа, среда выполнения получает другой пакет.|
|messageEncoding|base64| Этот параметр доступен только в [расширениях версии 5.0.0 и выше](#storage-extension-5x-and-higher). Он представляет формат кодирования сообщений. Допустимые значения: `base64` и `none`.|

## <a name="next-steps"></a>Дальнейшие действия

- [Выполнение функции как изменения данных хранилища очередей (триггер)](./functions-bindings-storage-queue-trigger.md)
- [Сообщения хранилища очереди записи (Выходная привязка)](./functions-bindings-storage-queue-output.md)
