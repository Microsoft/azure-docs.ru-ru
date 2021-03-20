---
title: Триггеры и привязки HTTP в службе "Функции Azure"
description: Узнайте, как использовать триггеры и привязки HTTP в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.openlocfilehash: 5236a3be9f8908d886274764dfc0c0da5b3565bc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92104484"
---
# <a name="azure-functions-http-triggers-and-bindings-overview"></a>Общие сведения о триггерах и привязках HTTP в функциях Azure

Функции Azure могут вызываться через HTTP-запросы для создания бессерверных интерфейсов API и реагирования на [веб-перехватчики](https://en.wikipedia.org/wiki/Webhook).

| Действие | Type |
|---------|---------|
| Выполнение функции из HTTP-запроса | [Триггер](./functions-bindings-http-webhook-trigger.md) |
| Выходные привязки HTTP для Функций Azure |[Выходная привязка](./functions-bindings-http-webhook-output.md) |

Код в этой статье по умолчанию использует синтаксис .NET Core, используемый в функциях версии 2. x и более поздних. Сведения о синтаксисе версии 1.x см. на странице с [соответствующими шаблонами функций](https://github.com/Azure/azure-functions-templates/tree/v1.x/Functions.Templates/Templates).

## <a name="add-to-your-functions-app"></a>Добавление в приложение функций

### <a name="functions-2x-and-higher"></a>Функции 2.x и более поздних версий

Для работы с триггером и привязками требуется ссылка на соответствующий пакет. Пакет NuGet используется для библиотек классов .NET, в то время как набор расширений используется для всех других типов приложений.

| Язык                                        | Добавить по...                                   | Remarks 
|-------------------------------------------------|---------------------------------------------|-------------|
| C#                                              | Установка [пакета NuGet], версия 3. x | |
| Скрипт C#, Java, JavaScript, Python, PowerShell | Регистрация [пакета расширений]          | [Расширение "инструменты Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack) " рекомендуется использовать с Visual Studio Code. |
| Скрипт C# (только в сети портал Azure)         | Добавление привязки                            | Чтобы обновить существующие расширения привязки без повторной публикации приложения функции, см. статью [Обновление расширений]. |

[core tools]: ./functions-run-local.md
[Пакет расширений]: ./functions-bindings-register.md#extension-bundles
[Пакет NuGet]: https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Http
[Обновление расширений]: ./functions-bindings-register.md
[Azure Tools extension]: https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack

### <a name="functions-1x"></a>Функции 1.x

Функции 1. x автоматически имеют ссылку на пакет NuGet [Microsoft. Azure. веб-задания](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) , версия 2. x.

## <a name="next-steps"></a>Дальнейшие действия

- [Выполнение функции из HTTP-запроса](./functions-bindings-http-webhook-trigger.md)
- [Выходные привязки HTTP для Функций Azure](./functions-bindings-http-webhook-output.md)