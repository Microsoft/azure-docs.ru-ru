---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 07/05/2019
ms.author: glenga
ms.openlocfilehash: 5e1a2622df0038141dd5cb05237f93d5e33e0bfb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "78190927"
---
В проекте библиотеки классов C# привязки определяются как атрибуты привязки в методе функции. Затем на основе этих атрибутов автоматически создается файл *function.json*, обязательный для службы Функции Azure.

Откройте в проекте файл *HttpExample.cs* и добавьте следующий параметр в определение метода `Run`:

:::code language="csharp" source="~/functions-docs-csharp/functions-add-output-binding-storage-queue-cli/HttpExample.cs" range="17":::

Параметр `msg` относится к типу `ICollector<T>`, который представляет собой коллекцию сообщений, записываемых в выходную привязку после завершения функции. В этом случае выходные данные представляют собой очередь хранилища с именем `outqueue`. Строка подключения к учетной записи хранения задана атрибутом `StorageAccountAttribute`. Он указывает параметр, который содержит строку подключения к учетной записи хранения и может применяться на уровне класса, метода или параметра. В этом случае вы можете опустить `StorageAccountAttribute`, так как уже используете учетную запись хранения по умолчанию.

Определение метода Run должно выглядеть следующим образом:  

:::code language="csharp" source="~/functions-docs-csharp/functions-add-output-binding-storage-queue-cli/HttpExample.cs" range="14-18":::
