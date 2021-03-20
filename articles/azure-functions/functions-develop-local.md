---
title: Разработка и запуск функций Azure в локальной среде
description: Узнайте, как кодировать и тестировать функции Azure на локальном компьютере перед их запуском в службе "функции Azure".
ms.topic: conceptual
ms.date: 09/04/2018
ms.openlocfilehash: 9c37d51abcc8d612b777b845515cf07666369d4f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96168140"
---
# <a name="code-and-test-azure-functions-locally"></a>Как программировать и тестировать Функции Azure в локальной среде

Хотя вы можете разрабатывать и тестировать Функции Azure на [портале Azure], многие разработчики предпочитают локальную среду разработки. Решение "Функции" позволяет с легкостью использовать предпочитаемый редактор кода и средства разработки для создания и тестирования функций на локальном компьютере. Локальные функции можно подключать к действующим службам Azure, а их отладку можно выполнять на локальном компьютере с помощью полной среды выполнения решения "Функции".

## <a name="local-development-environments"></a>Локальные среды разработки

Способ разработки функций на локальном компьютере зависит от настроек [языка](supported-languages.md) и инструментария. В нижеприведенной таблице указаны среды, которые поддерживают локальную разработку:

|Среда                              |Языки         |Описание|
|-----------------------------------------|------------|---|
|[Visual Studio Code](functions-develop-vs-code.md)| [C# (библиотека классов)](functions-dotnet-class-library.md), [C# скрипт (.csx)](functions-reference-csharp.md), [JavaScript](functions-reference-node.md), [PowerShell](./create-first-function-vs-code-powershell.md), [Python](functions-reference-python.md) | [Расширение Функций Azure для VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) добавляет поддержку средства "Функции" в VS Code. Требуются основные инструменты. Поддерживает разработку на платформах Linux, MacOS и Windows при использовании версии 2.x основных инструментов. Для получения дополнительных сведений см. статью [Создание функции в Azure с помощью Visual Studio Code](./create-first-function-vs-code-csharp.md). |
| [Командная строка или терминал](functions-run-local.md) | [C# (библиотека классов)](functions-dotnet-class-library.md), [C# скрипт (.csx)](functions-reference-csharp.md), [JavaScript](functions-reference-node.md), [PowerShell](functions-reference-powershell.md), [Python](functions-reference-python.md) | [Azure functions Core Tools] предоставляет основную среду выполнения и шаблоны для создания функций, которые обеспечивают локальную разработку. Версия 2.x поддерживает разработку в Linux, MacOS и Windows. Во всех средах применяют основные инструменты для локальной среды выполнения средства "Функции". |
| [Visual Studio 2019](functions-develop-vs.md) | [C# (библиотека классов)](functions-dotnet-class-library.md) | Средства функций Azure включены в рабочую нагрузку **разработки Azure** [Visual Studio 2019](https://www.visualstudio.com/vs/) и более поздних версий. Позволяет компилировать функции в библиотеке классов и публиковать библиотеки DLL в Azure. Включает основные инструменты для локального тестирования. Дополнительные сведения см. в статье [Инструменты Функций Azure для Visual Studio](functions-develop-vs.md). |
| [Maven](./create-first-function-cli-java.md) (различные) | [Java](functions-reference-java.md) | Интегрируется с основными инструментами и обеспечивает возможность разработки функций Java. Версия 2.x поддерживает разработку в Linux, MacOS и Windows. Дополнительные сведения см. в статье [Создание первой функции с помощью Java и Maven](./create-first-function-cli-java.md). Также поддерживает разработку с помощью [Eclipse](functions-create-maven-eclipse.md) и [IntelliJ IDEA](functions-create-maven-intellij.md). |

[!INCLUDE [Don't mix development environments](../../includes/functions-mixed-dev-environments.md)]

Каждая из этих локальных сред разработки позволяет создавать проекты приложений-функций и использовать предопределенные шаблоны решения "Функции" для создания новых функций. Каждая из них использует основные инструменты, чтобы вы могли тестировать и отлаживать функции в реальной среде выполнения решения "Функции" на своем компьютере как и любое другое приложение. Вы также можете опубликовать проект приложения функции из любой из этих сред в Azure.

## <a name="next-steps"></a>Дальнейшие действия

+ Дополнительные сведения о локальной разработке скомпилированных функций C# с помощью Visual Studio 2019 см. в статье [разработка функций Azure с помощью Visual Studio](functions-develop-vs.md).
+ Дополнительные сведения о локальной разработке функций с помощью VS Code на компьютере Mac, Linux или Windows см. в статье [развертывание функций Azure из VS Code](/azure/developer/javascript/tutorial-vscode-serverless-node-01).
+ Дополнительные сведения о разработке функций из командной строки или терминала см. в статье [Как программировать и тестировать Функции Azure в локальной среде](functions-run-local.md).

<!-- LINKS -->

[Azure Functions Core Tools]: https://www.npmjs.com/package/azure-functions-core-tools
[Портал Azure]: https://portal.azure.com
[Node.js]: https://docs.npmjs.com/getting-started/installing-node#osx-or-windows