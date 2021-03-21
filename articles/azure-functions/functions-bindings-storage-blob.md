---
title: Триггер и привязки хранилища BLOB-объектов Azure для функций Azure
description: Узнайте, как использовать триггер и привязки хранилища BLOB-объектов Azure в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.openlocfilehash: 4ec21086ee94610be1d9cf5da7b64c837b5311a9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100381534"
---
# <a name="azure-blob-storage-bindings-for-azure-functions-overview"></a>Общие сведения о привязках хранилища BLOB-объектов Azure для функций Azure

Функции Azure интегрируются с [хранилищем Azure](../storage/index.yml) с помощью [триггеров и привязок](./functions-triggers-bindings.md). Интеграция с хранилищем BLOB-объектов позволяет создавать функции, реагирующие на изменения в данных больших двоичных объектов, а также значения чтения и записи.

| Действие | Type |
|---------|---------|
| Выполнение функции в качестве изменений данных хранилища BLOB-объектов | [Триггер](./functions-bindings-storage-blob-trigger.md) |
| Чтение данных хранилища BLOB-объектов в функции | [Входная привязка](./functions-bindings-storage-blob-input.md) |
| Разрешить функции записывать данные хранилища BLOB-объектов |[Выходная привязка](./functions-bindings-storage-blob-output.md) |

## <a name="add-to-your-functions-app"></a>Добавление в приложение функций

### <a name="functions-2x-and-higher"></a>Функции 2.x и более поздних версий

Для работы с триггером и привязками требуется ссылка на соответствующий пакет. Пакет NuGet используется для библиотек классов .NET, в то время как набор расширений используется для всех других типов приложений.

| Язык                                        | Добавить по...                                   | Remarks 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Установка [пакета NuGet], версия 3. x | |
| Скрипт C#, Java, JavaScript, Python, PowerShell | Регистрация [пакета расширений]          | [Расширение "инструменты Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) " рекомендуется использовать с Visual Studio Code. |
| Скрипт C# (только в сети портал Azure)         | Добавление привязки                            | Чтобы обновить существующие расширения привязки без повторной публикации приложения функции, см. статью [Обновление расширений]. |

#### <a name="storage-extension-5x-and-higher"></a>Расширение хранилища 5. x и более поздние версии

Новая версия расширения привязок хранилища доступна в качестве [предварительной версии пакета NuGet](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage/5.0.0-beta.2). В этой предварительной версии вводится возможность [подключения с использованием удостоверения, а не секрета](./functions-reference.md#configure-an-identity-based-connection). Для приложений .NET также изменяются типы, к которым можно выполнить привязку, заменяя типы `WindowsAzure.Storage` и `Microsoft.Azure.Storage` с более новыми типами из [Azure. Storage. blobs](/dotnet/api/azure.storage.blobs).

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

## <a name="hostjson-settings"></a>Параметры файла host.json

> [!NOTE]
> Этот раздел не применяется при использовании версий расширений до 5.0.0. Для этих версий нет глобальных параметров конфигурации для больших двоичных объектов.

В этом разделе описаны глобальные параметры конфигурации, доступные для этой привязки при использовании [расширения версии 5.0.0 и более поздней](#storage-extension-5x-and-higher). Пример *host.jsв* файле ниже содержит только параметры версии 2. x + для этой привязки. Дополнительные сведения о глобальных параметрах конфигурации в функциях версии 2. x и более поздних см. в разделе [host.jsв справочнике по функциям Azure](functions-host-json.md).

```json
{
    "version": "2.0",
    "extensions": {
        "blobs": {
            "maxDegreeOfParallelism": "4"
        }
    }
}
```

|Свойство  |По умолчанию | Описание |
|---------|---------|---------|
|maxDegreeOfParallelism|8 * (число доступных ядер)|Целое число одновременных вызовов, разрешенное для каждой функции, активируемой с помощью большого двоичного объекта. Минимальное допустимое значение — 1.|

## <a name="next-steps"></a>Дальнейшие действия

- [Выполнение функции при изменении данных хранилища BLOB-объектов](./functions-bindings-storage-blob-trigger.md)
- [Чтение данных хранилища BLOB-объектов при выполнении функции](./functions-bindings-storage-blob-input.md)
- [Запись данных хранилища BLOB-объектов из функции](./functions-bindings-storage-blob-output.md)
