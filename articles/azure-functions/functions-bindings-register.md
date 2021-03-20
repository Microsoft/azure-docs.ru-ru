---
title: Регистрация расширений привязки функций Azure
description: Узнайте, как зарегистрировать расширение привязки функций Azure на основе вашей среды.
author: craigshoemaker
ms.topic: reference
ms.date: 08/16/2020
ms.author: cshoe
ms.openlocfilehash: 942ca3229808b57894598c3477e9dc97e40e8c80
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88689575"
---
# <a name="register-azure-functions-binding-extensions"></a>Регистрация расширений привязки функций Azure

Начиная с функций Azure версии 2. x, среда выполнения функций по умолчанию включает в себя только триггеры HTTP и Timer. Другие [триггеры и привязки](./functions-triggers-bindings.md) доступны в виде отдельных пакетов.

Приложения функций библиотеки классов .NET используют привязки, установленные в проекте как пакеты NuGet. Пакеты расширений позволяют приложениям non-.NET функций использовать одни и те же привязки без необходимости работать с инфраструктурой .NET.

В следующей таблице показано, когда и как регистрируются привязки.

| Среда разработки |Регистрация<br/> в службе "Функции" версии 1.x  |Регистрация<br/> в функциях 3. x/2. x  |
|-------------------------|------------------------------------|------------------------------------|
|Портал Azure|Автоматически|Автоматически<sup>*</sup>|
|Non-.NET языки|Автоматически|Использовать [пакеты расширений](#extension-bundles) (рекомендуется) или [явно устанавливать расширения](#explicitly-install-extensions)|
|Библиотека классов C# с использованием Visual Studio|[С помощью средств NuGet](#vs)|[С помощью средств NuGet](#vs)|
|Библиотека классов C# с использованием Visual Studio Code|Н/Д|[С помощью .NET Core CLI](#vs-code)|

<sup>*</sup> Портал использует пакеты расширений.

## <a name="access-extensions-in-non-net-languages"></a>Доступ к расширениям на языках non-.NET

Для приложений Java, JavaScript, PowerShell, Python и пользовательских обработчиков событий мы рекомендуем использовать наборы расширений для доступа к привязкам. В случаях, когда не удается использовать пакеты расширений, можно явно установить расширения привязки.

### <a name="extension-bundles"></a><a name="extension-bundles"></a>Пакеты расширений

Пакеты расширений — это способ добавления совместимого набора расширений привязки в приложение-функцию. Пакеты расширений можно включить в *host.js* файла приложения.

Вы можете использовать пакеты расширений с версией 2. x и более поздними версиями среды выполнения функций.

Пакеты расширений имеют версию. Каждая версия содержит конкретный набор расширений привязки, которые проверяются для совместной работы. Выберите версию пакета на основе расширений, необходимых в приложении.

Чтобы добавить пакет расширений в приложение функции, добавьте `extensionBundle` раздел в *host.js*. Во многих случаях Visual Studio Code и Azure Functions Core Tools будут автоматически добавлять их.

[!INCLUDE [functions-extension-bundles-json](../../includes/functions-extension-bundles-json.md)]

В следующей таблице перечислены доступные в настоящее время версии пакета *Microsoft. Azure. functions. екстенсионбундле* по умолчанию и ссылки на расширения, которые они включают.

| Версия пакета | Версия в host.js | Добавленные расширения |
| --- | --- | --- |
| 1.x | `[1.*, 2.0.0)` | См. [extensions.jsоб](https://github.com/Azure/azure-functions-extension-bundles/blob/v1.x/src/Microsoft.Azure.Functions.ExtensionBundle/extensions.json) использовании для создания пакета |
| 2.x | `[2.*, 3.0.0)` | См. [extensions.jsоб](https://github.com/Azure/azure-functions-extension-bundles/blob/v2.x/src/Microsoft.Azure.Functions.ExtensionBundle/extensions.json) использовании для создания пакета |

> [!NOTE]
> Хотя можно указать диапазон настраиваемых версий в host.js, рекомендуется использовать значение версии из этой таблицы.

### <a name="explicitly-install-extensions"></a><a name="explicitly-install-extensions"></a>Явно установить расширения

[!INCLUDE [functions-extension-register-core-tools](../../includes/functions-extension-register-core-tools.md)]

## <a name="install-extensions-from-nuget-in-net-languages"></a><a name="local-csharp"></a>Установка расширений из NuGet на языках .NET

Для проекта функций на основе библиотеки классов C# следует устанавливать расширения напрямую. Пакеты расширений предназначены специально для проектов, которые не основаны на библиотеке классов C#.

### <a name="c-class-library-with-visual-studio"></a><a name="vs"></a>\#Библиотека классов C в Visual Studio

В **Visual Studio** пакеты можно установить из консоли диспетчера пакетов с помощью команды [Install-Package](/nuget/tools/ps-ref-install-package) , как показано в следующем примере:

```powershell
Install-Package Microsoft.Azure.WebJobs.Extensions.ServiceBus -Version <TARGET_VERSION>
```

Имя пакета, используемого для данной привязки, приведено в справочной статье для этой привязки. Например, вы можете ознакомиться с [разделом о пакетах в справочной статье о привязках Служебной шины](functions-bindings-service-bus.md#functions-1x).

Замените `<TARGET_VERSION>` в этом примере определенной версией пакета, например `3.0.0-beta5`. Допустимые версии перечислены на отдельных страницах пакета по адресу [NuGet.org](https://nuget.org). Основные версии, соответствующие функциям среды выполнения 1. x или 2. x, указаны в справочной статье для привязки.

При использовании `Install-Package` для ссылки на привязку не нужно использовать [пакеты расширений](#extension-bundles). Этот подход характерен для библиотек классов, созданных в Visual Studio.

### <a name="c-class-library-with-visual-studio-code"></a><a name="vs-code"></a> Библиотека классов C# с Visual Studio Code

В **Visual Studio Code** установите пакеты для проекта библиотеки классов C# из командной строки с помощью команды [DotNet add package](/dotnet/core/tools/dotnet-add-package) в .NET Core CLI. В следующем примере показано, как добавить привязку:

```terminal
dotnet add package Microsoft.Azure.WebJobs.Extensions.<BINDING_TYPE_NAME> --version <TARGET_VERSION>
```

Интерфейс .NET Core CLI можно использовать только при разработке в среде "Функции Azure" версии 2.х.

Замените на `<BINDING_TYPE_NAME>` имя пакета, содержащего необходимую привязку. Нужную ссылочную статью привязки можно найти в [списке поддерживаемых привязок](./functions-triggers-bindings.md#supported-bindings).

Замените `<TARGET_VERSION>` в этом примере определенной версией пакета, например `3.0.0-beta5`. Допустимые версии перечислены на отдельных страницах пакета по адресу [NuGet.org](https://nuget.org). Основные версии, соответствующие функциям среды выполнения 1. x или 2. x, указаны в справочной статье для привязки.

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Пример триггера и привязки функции Azure](./functions-bindings-example.md)
