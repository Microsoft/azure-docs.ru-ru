---
title: Настройка приложений ASP.NET
description: Узнайте, как настроить приложение ASP.NET в службе приложений Azure. В этой статье показаны наиболее распространенные задачи настройки.
ms.devlang: dotnet
ms.custom: devx-track-csharp, devx-track-azurecli
ms.topic: article
ms.date: 06/02/2020
ms.openlocfilehash: 8ed6835583cc4881b19eee14ed392b193324535e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92744155"
---
# <a name="configure-an-aspnet-app-for-azure-app-service"></a>Настройка приложения ASP.NET для службы приложений Azure

> [!NOTE]
> Дополнительные ASP.NET Core см. [в статье Настройка приложения ASP.NET Core для службы приложений Azure](configure-language-dotnetcore.md) .

Приложения ASP.NET должны быть развернуты в службе приложений Azure как скомпилированные двоичные файлы. Средство публикации Visual Studio создает решение, а затем развертывает скомпилированные двоичные файлы напрямую, в то время как модуль развертывания службы приложений сначала развертывает репозиторий кода, а затем компилирует двоичные файлы.

Это краткое описание содержит основные понятия и инструкции для разработчиков ASP.NET. Если вы никогда не использовали службу приложений Azure, сначала выполните инструкции из руководства по [ASP.NET](quickstart-dotnet-framework.md) и [ASP.NET с базой данных SQL](app-service-web-tutorial-dotnet-sqldatabase.md) .

## <a name="show-supported-net-framework-runtime-versions"></a>Отображение поддерживаемых версий среды выполнения платформа .NET Framework

В службе приложений на экземплярах Windows уже установлены все поддерживаемые версии платформа .NET Framework. Чтобы отобразить доступные версии среды выполнения платформа .NET Framework и пакета SDK, перейдите к `https://<app-name>.scm.azurewebsites.net/DebugConsole` и выполните соответствующую команду в консоли на основе браузера:

Для версий среды выполнения CLR 4 (платформа .NET Framework 4 и выше):

```CMD
ls "D:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\"
```

Последняя версия платформа .NET Framework может быть недоступна немедленно.

Для версий среды выполнения CLR 2 (платформа .NET Framework 3,5 и ниже):

```CMD
ls "D:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\"
```

## <a name="show-current-net-framework-runtime-version"></a>Показывать текущую версию среды выполнения платформа .NET Framework

Выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query netFrameworkVersion
```

Значение `v4.0` означает, что используется последняя версия CLR 4 (платформа .NET Framework 4. x). Значение `v2.0` означает, что используется версия CLR 2 (платформа .NET Framework 3,5).

## <a name="set-net-framework-runtime-version"></a>Задать версию среды выполнения платформа .NET Framework

По умолчанию служба приложений использует последнюю поддерживаемую версию платформа .NET Framework для запуска приложения ASP.NET. Чтобы запустить приложение с помощью платформа .NET Framework 3,5, выполните следующую команду в [Cloud Shell](https://shell.azure.com) (v 2.0 означает CLR 2):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --net-framework-version v2.0
```

## <a name="access-environment-variables"></a>Доступ к переменным среды

В службе приложений можно [задать параметры приложения](configure-common.md#configure-app-settings) и строки подключения вне кода приложения. Затем вы можете обращаться к ним в любом классе, используя стандартный шаблон ASP.NET:

```csharp
using System.Configuration;
...
// Get an app setting
ConfigurationManager.AppSettings["MySetting"];
// Get a connection string
ConfigurationManager.ConnectionStrings["MyConnection"];
}
```

Если вы настроили параметр приложения с тем же именем в службе приложений и в *web.config*, значение службы приложений имеет приоритет над значением *web.config* . Локальное значение *web.config* позволяет выполнять отладку приложения локально, но значение службы приложений позволяет запускать приложение в продукте с параметрами рабочей среды. Строки подключения работают таким же образом. Таким образом вы сможете защитить секреты приложения за пределами репозитория кода и получить доступ к соответствующим значениям без изменения кода.

## <a name="deploy-multi-project-solutions"></a>Развертывание решений с несколькими проектами

Если решение Visual Studio включает несколько проектов, процесс публикации Visual Studio уже включает в себя выбор развертываемого проекта. При развертывании в подсистеме развертывания службы приложений, например в Git или с ZIP-развертыванием с включенной автоматизацией сборки, подсистема развертывания службы приложений выбирает первый веб-сайт или проект веб-приложения, который он находит в качестве приложения службы приложений. Вы можете указать, какую службу приложений проекта следует использовать, указав `PROJECT` параметр приложения. Например, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings PROJECT="<project-name>/<project-name>.csproj"
```

## <a name="get-detailed-exceptions-page"></a>Страница «Получение подробных сведений об исключениях»

Когда приложение ASP.NET создает исключение в отладчике Visual Studio, браузер отображает подробную страницу исключения, но в службе приложений эта страница заменяется универсальным сообщением об ошибке. Чтобы отобразить страницу подробного исключения в службе приложений, откройте файл *Web.config* и добавьте `<customErrors mode="Off"/>` элемент в `<system.web>` элемент. Пример:

```xml
<system.web>
    <customErrors mode="Off"/>
</system.web>
```

Повторно разверните приложение с помощью обновленной *Web.config*. Теперь должна отобразиться одна и та же страница подробного исключения.

## <a name="access-diagnostic-logs"></a>Доступ к журналам диагностики

Диагностические сообщения можно добавить в код приложения с помощью [System. Diagnostics. Trace](/dotnet/api/system.diagnostics.trace). Пример: 

```csharp
Trace.TraceError("Record not found!"); // Error trace
Trace.TraceWarning("Possible data loss"); // Warning trace
Trace.TraceInformation("GET /Home/Index"); // Information trace
```

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Создание приложения ASP.NET в Azure с подключением к базе данных SQL](app-service-web-tutorial-dotnet-sqldatabase.md)
