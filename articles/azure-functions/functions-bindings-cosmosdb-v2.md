---
title: Привязки Azure Cosmos DB для функций 2. XD и более поздних версий
description: Узнайте, как использовать триггеры и привязки Azure Cosmos DB в службе "Функции Azure".
author: craigshoemaker
ms.topic: reference
ms.date: 02/24/2017
ms.author: cshoe
ms.openlocfilehash: dde3b95c7997a7f742a9d48a964f4275169f6e9c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92106932"
---
# <a name="azure-cosmos-db-trigger-and-bindings-for-azure-functions-2x-and-higher-overview"></a>Azure Cosmos DB триггеры и привязки для функций Azure 2. x и более поздних обзоров

> [!div class="op_single_selector" title1="Выберите версию используемой среды выполнения функций Azure: "]
> * [Версия 1](functions-bindings-cosmosdb.md)
> * [Версии 2 и выше](functions-bindings-cosmosdb-v2.md)

В этом наборе статей объясняется, как работать с привязками [Azure Cosmos DB](../cosmos-db/serverless-computing-database.md) в функциях Azure 2. x и более поздних версий. Служба "Функции Azure" поддерживает привязки триггера, а также входные и выходные привязки для Azure Cosmos DB.

| Действие | Type |
|---------|---------|
| Выполнение функции при создании или изменении документа Azure Cosmos DB | [Триггер](./functions-bindings-cosmosdb-v2-trigger.md) |
| Чтение Azure Cosmos DB документа | [Входная привязка](./functions-bindings-cosmosdb-v2-input.md) |
| Сохранение изменений в документе Azure Cosmos DB  |[Выходная привязка](./functions-bindings-cosmosdb-v2-output.md) |

> [!NOTE]
> Эта ссылка предназначена для [функций Azure версии 2. x и более поздних](functions-versions.md).  Для получения дополнительных сведений об использовании Функций версии 1.x см. раздел [Функции 1.х](functions-bindings-cosmosdb.md).
>
> Эта привязка называлась DocumentDB. В функциях версии 2. x и более поздних, триггер, привязки и пакет имеют имя Cosmos DB.

## <a name="supported-apis"></a>Поддерживаемые API

[!INCLUDE [SQL API support only](../../includes/functions-cosmosdb-sqlapi-note.md)]

## <a name="add-to-your-functions-app"></a>Добавление в приложение функций

### <a name="functions-2x-and-higher"></a>Функции 2.x и более поздних версий

Для работы с триггером и привязками требуется ссылка на соответствующий пакет. Пакет NuGet используется для библиотек классов .NET, в то время как набор расширений используется для всех других типов приложений.

| Язык                                        | Добавить по...                                   | Комментарии 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Установка [пакета NuGet], версия 3. x | |
| Скрипт C#, Java, JavaScript, Python, PowerShell | Регистрация [пакета расширений]          | [Расширение "инструменты Azure] " рекомендуется использовать с Visual Studio Code. |
| Скрипт C# (только в сети портал Azure)         | Добавление привязки                            | Чтобы обновить существующие расширения привязки без повторной публикации приложения функции, см. статью [Обновление расширений]. |

[Пакет NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.CosmosDB
[core tools]: ./functions-run-local.md
[Пакет расширений]: ./functions-bindings-register.md#extension-bundles
[Обновление расширений]: ./functions-bindings-register.md
[Расширение "инструменты Azure"]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Функции 1.x

Функции 1. x автоматически имеют ссылку на пакет NuGet [Microsoft. Azure. веб-задания](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) , версия 2. x.

## <a name="next-steps"></a>Дальнейшие действия

- [Выполнение функции при создании или изменении документа Azure Cosmos DB (триггер)](./functions-bindings-cosmosdb-v2-trigger.md)
- [Чтение Azure Cosmos DB документа (входная привязка)](./functions-bindings-cosmosdb-v2-input.md)
- [Сохранение изменений в документе Azure Cosmos DB (Выходная привязка)](./functions-bindings-cosmosdb-v2-output.md)