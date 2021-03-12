---
author: craigshoemaker
ms.service: azure-functions
ms.topic: include
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: 05d136093bd509e8c23ce8622423216326b0f1f2
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102623579"
---
## <a name="add-to-your-functions-app"></a>Добавление в приложение функций

### <a name="functions-2x-and-higher"></a>Функции 2.x и более поздних версий

Для работы с триггером и привязками требуется ссылка на соответствующий пакет. Пакет NuGet используется для библиотек классов .NET, в то время как набор расширений используется для всех других типов приложений.

| Язык                                        | Добавить по...                                   | Remarks 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Установка [пакета NuGet], версия 3. x | |
| Скрипт C#, Java, JavaScript, Python, PowerShell | Регистрация [пакета расширений]          | [Расширение "инструменты Azure] " рекомендуется использовать с Visual Studio Code. |
| Скрипт C# (только в сети портал Azure)         | Добавление привязки                            | Чтобы обновить существующие расширения привязки без повторной публикации приложения функции, см. статью [Обновление расширений]. |

[Пакет NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs
[core tools]: ../articles/azure-functions/functions-run-local.md
[Пакет расширений]: ../articles/azure-functions/functions-bindings-register.md#extension-bundles
[Обновление расширений]: ../articles/azure-functions/functions-bindings-register.md
[Расширение "инструменты Azure"]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="event-hubs-extension-5x-and-higher"></a>Расширение концентраторов событий 5. x и более поздних версий

Новая версия расширения привязок концентраторов событий доступна в качестве [предварительной версии пакета NuGet](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs/5.0.0-beta.1). В этой предварительной версии вводится возможность [подключения с использованием удостоверения, а не секрета](../articles/azure-functions/functions-reference.md#configure-an-identity-based-connection). Для приложений .NET также изменяются типы, к которым можно выполнить привязку, заменяя типы `Microsoft.Azure.EventHubs` с более новыми типами из [Azure. Messaging. EventHubs](/dotnet/api/azure.messaging.eventhubs).

> [!NOTE]
> Предварительная версия пакета не включена в пакет расширений и должна быть установлена вручную. Для приложений .NET добавьте ссылку на пакет. Сведения о других типах приложений см. [в разделе Обновление расширений].

[core tools]: ./functions-run-local.md
[Пакет расширений]: ./functions-bindings-register.md#extension-bundles
[Пакет NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.EventHubs/
[Обновление расширений]: ./functions-bindings-register.md
[Расширение "инструменты Azure"]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Функции 1.x

Функции 1. x автоматически имеют ссылку на пакет NuGet [Microsoft. Azure. веб-задания](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) , версия 2. x.

## <a name="hostjson-settings"></a>Параметры файла host.json
<a name="host-json"></a>

В файле [host.json](../articles/azure-functions/functions-host-json.md#eventhub) содержатся параметры, управляющие реакцией триггера Центров событий на событие. Конфигурация будет разной в зависимости от версии Функций Azure.

[!INCLUDE [functions-host-json-event-hubs](../articles/azure-functions/../../includes/functions-host-json-event-hubs.md)]