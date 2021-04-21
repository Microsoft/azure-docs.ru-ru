---
title: Краткое руководство. Развертывание веб-приложения ASP.NET
description: Узнайте, как запускать веб-приложения в Службе приложений Azure, развернув свое первое приложение ASP.NET.
ms.assetid: b1e6bd58-48d1-4007-9d6c-53fd6db061e3
ms.topic: quickstart
ms.date: 03/30/2021
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, seodec18, contperf-fy21q1
zone_pivot_groups: app-service-ide
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-dotnetcore-uiex
ms.openlocfilehash: 482bf6d29fbc1e982ee4d17099d82915ff3a0241
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107762467"
---
<!-- NOTES:

I'm a .NET developer who wants to deploy my web app to App Service. I may develop apps with
Visual Studio, Visual Studio for Mac, Visual Studio Code, or the .NET SDK/CLI. This article
should be able to guide .NET devs, whether they're app is .NET Core, .NET, or .NET Framework.

As a .NET developer, when choosing an IDE and .NET TFM - you map to various OS requirements.
For example, if you choose Visual Studio - you're developing the app on Windows, but you can still
target cross-platform with .NET Core 3.1 or .NET 5.0.

| .NET / IDE         | Visual Studio | Visual Studio for Mac | Visual Studio Code | Command line   |
|--------------------|---------------|-----------------------|--------------------|----------------|
| .NET Core 3.1      | Windows       | macOS                 | Cross-platform     | Cross-platform |
| .NET 5.0           | Windows       | macOS                 | Cross-platform     | Cross-platform |
| .NET Framework 4.8 | Windows       | N/A                   | Windows            | Windows        |

-->

# <a name="quickstart-deploy-an-aspnet-web-app"></a>Краткое руководство. Развертывание веб-приложения ASP.NET

Из этого краткого руководства вы узнаете, как создать и развернуть свое первое веб-приложение ASP.NET в [Службе приложений Azure](overview.md). Служба приложений поддерживает разные версии приложений .NET и предоставляет службу веб-размещения с высоким уровнем масштабирования и автоматической установкой исправлений. Веб-приложения ASP.NET являются кросс-платформенными, т. е. могут размещаться в Linux или Windows. Когда вы закончите работу с ним, у вас будет создана группа ресурсов Azure с планом размещения Службы приложений и Службой приложений, где развернуто веб-приложение.

> [!TIP]
> .NET Core 3.1 является текущей версией .NET с долгосрочной поддержкой (LTS). Дополнительные сведения см. на странице с описанием [политики поддержки .NET](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

## <a name="prerequisites"></a>Предварительные требования

:::zone target="docs" pivot="development-environment-vs"

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet) бесплатно.
- <a href="https://www.visualstudio.com/downloads" target="_blank">Visual Studio 2019</a> с рабочей нагрузкой **ASP.NET и веб-разработка**.

    Если у вас уже установлена версия Visual Studio 2019:

    - Установите последние обновления для Visual Studio, выбрав **Справка** > **Проверить обновления**.
    - Добавьте рабочую нагрузку, выбрав **Инструменты** > **Получить средства и компоненты**.

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet) бесплатно.
- <a href="https://www.visualstudio.com/downloads" target="_blank">Visual Studio Code</a>.
- Расширение <a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack" target="_blank">Azure Tools</a>.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

<a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Установите последнюю версию пакета SDK для .NET Core 3.1.</a>

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

<a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">Установите последнюю версию пакета SDK для .NET 5.0.</a>

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

<a href="https://dotnet.microsoft.com/download/dotnet-framework/net48" target="_blank">Установите пакет разработчика .NET Framework 4.8.</a>

> [!NOTE]
> В отличие от Visual Studio Code .NET Framework не является кросс-платформенным решением. Если вы разрабатываете приложения для .NET Framework с помощью Visual Studio Code, мы рекомендуем использовать компьютер под управлением Windows, чтобы соблюсти требования к зависимостям для сборки.

---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet) бесплатно.
- <a href="/cli/azure/install-azure-cli" target="_blank">Интерфейс командной строки Azure</a>.
- Пакет SDK для .NET (включает среду выполнения и CLI).

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

<a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Установите последнюю версию пакета SDK для .NET Core 3.1.</a>

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

<a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">Установите последнюю версию пакета SDK для .NET 5.0.</a>

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

<a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">Установите последнюю версию пакета SDK для .NET 5.0</a> и <a href="https://dotnet.microsoft.com/download/dotnet-framework/net48" target="_blank"> пакет разработчика .NET Framework 4.8.</a>

> [!NOTE]
> В отличие от [.NET CLI](/dotnet/core/tools) .NET Framework не является кросс-платформенным решением. Если вы разрабатываете приложения для .NET Framework с помощью .NET CLI, мы рекомендуем использовать компьютер под управлением Windows, чтобы соблюсти требования к зависимостям для сборки.

---

:::zone-end

## <a name="create-an-aspnet-web-app"></a>Создание веб-приложения ASP.NET

:::zone target="docs" pivot="development-environment-vs"

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

1. Откройте Visual Studio и выберите **Создать проект**.
1. В разделе **Создание проекта** найдите и выберите **ASP.NET Web Core App** (Веб-приложение ASP.NET Core), а затем щелкните **Далее**.
1. В разделе **Настройка нового проекта** присвойте приложению имя _MyFirstAzureWebApp_ и щелкните **Далее**.

   :::image type="content" source="media/quickstart-dotnet/configure-webapp-net.png" alt-text="Настройка веб-приложения ASP.NET Core 3.1" border="true":::

1. Выберите **.NET Core 3.1 (долгосрочная поддержка)** .
1. Для параметра **Тип проверки подлинности** укажите значение **Нет**. Щелкните **Создать**.

   :::image type="content" source="media/quickstart-dotnet/vs-additional-info-netcoreapp31.png" alt-text="Visual Studio — выберите .NET Core 3.1, а для параметра &quot;Тип проверки подлинности&quot; выберите значение &quot;Нет&quot;." border="true":::

1. В меню Visual Studio выберите **Отладка** > **Запустить без отладки**, чтобы запустить приложение локально.

   :::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio — .NET Core 3.1, локальный запуск" lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

1. Откройте Visual Studio и выберите **Создать проект**.
1. В разделе **Создание проекта** найдите и выберите **ASP.NET Web Core App** (Веб-приложение ASP.NET Core), а затем щелкните **Далее**.
1. В разделе **Настройка нового проекта** присвойте приложению имя _MyFirstAzureWebApp_ и щелкните **Далее**.

   :::image type="content" source="media/quickstart-dotnet/configure-webapp-net.png" alt-text="Visual Studio — настройка веб-приложения ASP.NET 5.0." border="true":::

1. Выберите **.NET Core 5.0 (текущая)** .
1. Для параметра **Тип проверки подлинности** укажите значение **Нет**. Щелкните **Создать**.

   :::image type="content" source="media/quickstart-dotnet/vs-additional-info-net50.png" alt-text="Visual Studio — дополнительная информация при выборе .NET Core 5.0." border="true":::

1. В меню Visual Studio выберите **Отладка** > **Запустить без отладки**, чтобы запустить приложение локально.

   :::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio — локальное выполнение ASP.NET Core 5.0." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

1. Откройте Visual Studio и выберите **Создать проект**.
1. В окне **Создание проекта** найдите и выберите **Веб-приложение ASP.NET (.NET Framework)** , а затем щелкните **Далее**.
1. В окне **Настройка нового проекта** присвойте приложению имя _MyFirstAzureWebApp_ и щелкните **Создать**.

   :::image type="content" source="media/quickstart-dotnet/configure-webapp-netframework48.png" alt-text="Visual Studio — настройка веб-приложения ASP.NET Framework 4.8." border="true":::

1. Выберите шаблон **MVC** .
1. Убедитесь, что для параметра **Проверка подлинности** задано значение **Без проверки подлинности**. Щелкните **Создать**.

   :::image type="content" source="media/quickstart-dotnet/vs-mvc-no-auth-netframework48.png" alt-text="Visual Studio — выбор шаблона MVC." border="true":::

1. В меню Visual Studio выберите **Отладка** > **Запустить без отладки**, чтобы запустить приложение локально.

   :::image type="content" source="media/quickstart-dotnet/vs-local-webapp-netframework48.png" alt-text="Visual Studio — локальное выполнение ASP.NET Framework 4.8." lightbox="media/quickstart-dotnet/vs-local-webapp-netframework48.png" border="true":::

---

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

Создайте новую папку с именем _MyFirstAzureWebApp_ и откройте ее в Visual Studio Code. Откройте окно <a href="https://code.visualstudio.com/docs/editor/integrated-terminal" target="_blank">Терминал</a> и создайте новое веб-приложение .NET с помощью команды [`dotnet new webapp`](/dotnet/core/tools/dotnet-new#web-options).

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

```dotnetcli
dotnet new webapp -f netcoreapp3.1
```

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

```dotnetcli
dotnet new webapp -f net5.0
```

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

```dotnetcli
dotnet new webapp --target-framework-override net48
```

> [!IMPORTANT]
> Флаг `--target-framework-override` в произвольном тексте заменяет в проекте моникер целевой платформы (TFM), но он *не гарантирует* существование соответствующего шаблона или его соответствие требованиям. Компилировать и выполнять приложения .NET Framework можно только в среде Windows.

---

В окне **Терминал** Visual Studio Code запустите приложение в локальной среде с помощью команды [`dotnet run`](/dotnet/core/tools/dotnet-run).

```dotnetcli
dotnet run
```

Откройте веб-браузер и перейдите к приложению в `https://localhost:5001`.


### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

На странице браузера отобразится шаблон веб-приложения ASP.NET Core 3.1.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code — локальное выполнение .NET Core 3.1 в браузере." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

На странице браузера отобразится шаблон веб-приложения ASP.NET Core 5.0.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code — локальное выполнение .NET 5.0 в браузере." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

На странице браузера отобразится шаблон веб-приложения ASP.NET Framework 4.8.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net48.png" alt-text="Visual Studio Code — локальное выполнение .NET 4.8 в браузере." lightbox="media/quickstart-dotnet/local-webapp-net48.png" border="true":::

---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

Откройте окно терминала на компьютере и перейдите в рабочую папку. Создайте новое веб-приложение .NET с помощью команды [`dotnet new webapp`](/dotnet/core/tools/dotnet-new#web-options), а затем измените каталоги для только что созданного приложения.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

```dotnetcli
dotnet new webapp -n MyFirstAzureWebApp -f netcoreapp3.1 && cd MyFirstAzureWebApp
```

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

```dotnetcli
dotnet new webapp -n MyFirstAzureWebApp -f net5.0 && cd MyFirstAzureWebApp
```

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

```dotnetcli
dotnet new webapp -n MyFirstAzureWebApp --target-framework-override net48 && cd MyFirstAzureWebApp
```

> [!IMPORTANT]
> Флаг `--target-framework-override` в произвольном тексте заменяет в проекте моникер целевой платформы (TFM), но он *не гарантирует* существование соответствующего шаблона или его соответствие требованиям. Компилировать приложения платформы .NET Framework можно только в среде Windows.

---

В том же сеансе терминала запустите приложение локально с помощью команды [`dotnet run`](/dotnet/core/tools/dotnet-run).

```dotnetcli
dotnet run
```

Откройте веб-браузер и перейдите к приложению в `https://localhost:5001`.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

На странице браузера отобразится шаблон веб-приложения ASP.NET Core 3.1.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code — локальное выполнение ASP.NET Core 3.1 в браузере." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

На странице браузера отобразится шаблон веб-приложения ASP.NET Core 5.0.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net.png" alt-text="Visual Studio Code — локальное выполнение ASP.NET Core 5.0 в браузере." lightbox="media/quickstart-dotnet/local-webapp-net.png" border="true":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

На странице браузера отобразится шаблон веб-приложения ASP.NET Framework 4.8.

:::image type="content" source="media/quickstart-dotnet/local-webapp-net48.png" alt-text="Visual Studio Code — локальное выполнение ASP.NET Framework 4.8 в браузере." lightbox="media/quickstart-dotnet/local-webapp-net48.png" border="true":::

---

:::zone-end

## <a name="publish-your-web-app"></a>Публикация веб-приложения

Прежде чем опубликовать веб-приложение, следует создать и настроить новую Службу приложений, в которой вы сможете опубликовать это приложение.

В процессе настройки Службы приложений вы создадите следующее:

- Новая [группа ресурсов](../azure-resource-manager/management/overview.md#terminology) для всех ресурсов Azure, которые потребуются для этой службы.
- Новый [план размещения](overview-hosting-plans.md), который позволяет определить расположение, размер и функции фермы веб-серверов для размещения приложения.

Выполните следующие действия, чтобы создать Службу приложений и опубликовать веб-приложение:

:::zone target="docs" pivot="development-environment-vs"

1. Щелкните правой кнопкой мыши проект **MyFirstAzureWebApp** в **Обозревателе решений** и выберите **Опубликовать**.
1. В разделе **Публикация** выберите **Azure** и нажмите кнопку **Далее**.

    :::image type="content" source="media/quickstart-dotnet/vs-publish-target-Azure.png" alt-text="Visual Studio — публикация веб-приложения в Azure." border="true":::

1. Доступные параметры зависят от того, вошли ли вы в Azure и есть ли у вас учетная запись Visual Studio, связанная с учетной записью Azure. Выберите **Добавить учетную запись** или **Войти**, чтобы войти в подписку Azure. Если вы уже вошли, выберите нужную учетную запись.

    :::image type="content" source="media/quickstart-dotnetcore/sign-in-Azure-vs2019.png" border="true" alt-text="Visual Studio — диалоговое окно входа в Azure.":::

1. Выберите **Указанный целевой объект**: **Служба приложений Azure (Linux)** или **Служба приложений Azure (Windows)** .

    > [!IMPORTANT]
    > При нацеливании на ASP.NET Framework 4.8 вы будете использовать **Службу приложений Azure (Windows)** .

1. Справа от списка **Экземпляры Службы приложений** выберите **+** .

    :::image type="content" source="media/quickstart-dotnetcore/publish-new-app-service.png" border="true" alt-text="Visual Studio — диалоговое окно &quot;Новое приложение Службы приложений&quot;.":::

1. Для параметра **Подписка** подтвердите предложенный вариант или выберите другой из раскрывающегося списка.
1. В разделе **Группа ресурсов** выберите **Создать**. В разделе **Новое имя группы ресурсов** введите *myResourceGroup* и щелкните **ОК**.
1. В разделе **План размещения** щелкните **Создать**.
1. В диалоговом окне **План размещения. Создать новый** введите значения, указанные в следующей таблице.

    | Параметр          | Рекомендуемое значение          | Описание                                                           |
    |------------------|--------------------------|-----------------------------------------------------------------------|
    | **План размещения** | *MyFirstAzureWebAppPlan* | Имя плана службы приложений.                                         |
    | **Расположение**     | *Западная Европа*            | Центр обработки данных, где размещается веб-приложение.                           |
    | **Размер**         | *Бесплатный*                   | [Ценовая категория][app-service-pricing-tier] определяет возможности размещения. |

    :::image type="content" source="media/quickstart-dotnetcore/create-new-hosting-plan-vs2019.png" border="true" alt-text="Создание нового плана размещения":::

1. В поле **Имя** введите уникальное имя приложения, включающее только допустимые символы: `a-z`, `A-Z`, `0-9` и `-`. Вы можете использовать автоматически созданное уникальное имя. URL-адрес веб-приложения: `http://<app-name>.azurewebsites.net`, где `<app-name>` — имя приложения.
1. Выберите **Создать**, чтобы создать ресурсы Azure.

    :::image type="content" source="media/quickstart-dotnetcore/web-app-name-vs2019.png" border="true" alt-text="Visual Studio — диалоговое окно &quot;Создание ресурсов приложения&quot;.":::

   После завершения работы мастера ресурсы Azure будут созданы и готовы к публикации.

1. Выберите **Готово**, чтобы закрыть мастер.
1. На странице **Публикация** выберите **Опубликовать**. Visual Studio создает, упаковывает и публикует приложение в Azure, а затем запускает его в браузере по умолчанию.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    На странице браузера отобразится веб-приложение ASP.NET Core 3.1.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio — веб-приложение ASP.NET Core 3.1 в Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    На странице браузера отобразится веб-приложение ASP.NET Core 5.0.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio — веб-приложение ASP.NET Core 5.0 в Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    На странице браузера отобразится веб-приложение ASP.NET Framework 4.8.

    :::image type="content" source="media/quickstart-dotnet/vs-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/vs-Azure-webapp-net48.png" border="true" alt-text="Visual Studio — веб-приложение ASP.NET Framework 4.8 в Azure.":::

    ---

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

Чтобы развернуть веб-приложение с помощью расширения средств Azure для Visual Studio:

1. В Visual Studio Code откройте раздел [**Палитра команд**](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette) и нажмите клавиши <kbd>CTRL</kbd>+<kbd>SHIFT</kbd>+<kbd>P</kbd>.
1. Найдите и выберите "Служба приложений Azure: развернуть в веб-приложении".
1. Ответьте на запросы следующим образом:

    - Выберите для развертывания папку *MyFirstAzureWebApp*.
    - При появлении запроса выберите **Добавить конфигурацию**.
    - Если отобразится запрос, войдите в существующую учетную запись Azure.

    :::image type="content" source="media/quickstart-dotnet/vscode-sign-in-to-Azure.png" alt-text="Visual Studio Code — вход в Azure" border="true":::.

    - Выберите **Подписка**.
    - Щелкните **Создать веб-приложение... Дополнительно**.
    - В поле **Введите глобально уникальное имя** введите имя, которое еще не используется в Azure (*допускаются символы `a-z`, `0-9` и `-`* ). Рекомендуется использовать сочетание названия компании и идентификатора приложения.
    - Выберите **Создать группу ресурсов** и укажите имя, например `myResourceGroup`.
    - Когда появится запрос **Выберите стек времени выполнения**:
      - для *.NET Core 3.1* выберите **.NET Core 3.1 (LTS)** ;
      - для *.NET 5.0* выберите **.NET 5**;
      - для *.NET Framework 4.8* выберите **ASP.NET V4.8**.
    - Выберите операционную систему (Windows или Linux):
        - для *.NET Framework 4.8* будет автоматически выбрана платформа Windows.
    - Выберите **Создать новый план службы приложений**, укажите имя и выберите [ценовую категорию][app-service-pricing-tier] **Бесплатный F1**.
    - Выберите **Пропустить** для ресурса Application Insights.
    - Выберите ближайшее расположение.

1. Когда публикация завершится, щелкните **Обзор веб-сайта** в открывшемся уведомлении и нажмите кнопку **Открыть** при появлении запроса.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    На странице браузера отобразится веб-приложение ASP.NET Core 3.1.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio Code — веб-приложение ASP.NET Core 3.1 в Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    На странице браузера отобразится веб-приложение ASP.NET Core 5.0.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="Visual Studio Code — веб-приложение ASP.NET Core 5.0 в Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    На странице браузера отобразится веб-приложение ASP.NET Framework 4.8.

    :::image type="content" source="media/quickstart-dotnet/Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/vs-Azure-webapp-net48.png" border="true" alt-text="Visual Studio Code — веб-приложение ASP.NET Framework 4.8 в Azure.":::

    ---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

Разверните код в локальном каталоге *MyFirstAzureWebApp* с помощью команды [`az webapp up`](/cli/azure/webapp#az_webapp_up):

```azurecli
az webapp up --sku F1 --name <app-name> --os-type <os>
```

- Если команда `az` не распознается, проверьте, установили ли вы Azure CLI, как описано в разделе [Предварительные требования](#prerequisites).
- Замените `<app-name>` именем, уникальным для всех регионов Azure (*допустимыми символами являются `a-z`, `0-9`и `-`* ). Рекомендуется использовать сочетание названия компании и идентификатора приложения.
- Аргумент `--sku F1` создает веб-приложение с [ценовой категорией][app-service-pricing-tier] **Бесплатный**. Этот аргумент можно опустить, чтобы использовать более быструю ценовую категорию "Премиум" с почасовой оплатой.
- Измените `<os>` на `linux` или `windows`. При нацеливании на *ASP.NET Framework 4.8* необходимо использовать `windows`.
- При необходимости вы можете использовать аргумент `--location <location-name>`, где `<location-name>` является доступным регионом Azure. Список допустимых регионов для учетной записи Azure можно получить, выполнив команду [`az account list-locations`](/cli/azure/appservice#az_appservice_list_locations).

Выполнение этой команды может занять несколько минут. По мере выполнения будут отображаться сообщения о создании группы ресурсов, плане службы приложений и размещения приложения, настройке ведения журнала и последующем выполнении развертывания ZIP-файла. Затем отобразится сообщение с URL-адресом приложения:

```azurecli
You can launch the app at http://<app-name>.azurewebsites.net
```

Откройте браузер и перейдите по этому URL-адресу:

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

На странице браузера отобразится веб-приложение ASP.NET Core 3.1.

:::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="CLI — веб-приложение ASP.NET Core 3.1 в Azure.":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

На странице браузера отобразится веб-приложение ASP.NET Core 5.0.

:::image type="content" source="media/quickstart-dotnet/Azure-webapp-net.png" lightbox="media/quickstart-dotnet/Azure-webapp-net.png" border="true" alt-text="CLI — веб-приложение ASP.NET Core 5.0 в Azure.":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

На странице браузера отобразится веб-приложение ASP.NET Framework 4.8.

:::image type="content" source="media/quickstart-dotnet/Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/Azure-webapp-net48.png" border="true" alt-text="CLI — веб-приложение ASP.NET Framework 4.8 в Azure.":::

---

:::zone-end

## <a name="update-the-app-and-redeploy"></a>Обновление и повторное развертывание приложения

Чтобы обновить и повторно развернуть веб-приложение, сделайте следующее:

:::zone target="docs" pivot="development-environment-vs"

1. В **Обозревателе решений** откройте файл *Index.cshtml* вашего проекта.
1. Замените первый элемент `<div>` следующим кодом:

    ```razor
    <div class="jumbotron">
        <h1>.NET 💜 Azure</h1>
        <p class="lead">Example .NET app to Azure App Service.</p>
    </div>
    ```

   Сохраните изменения.

1. Чтобы выполнить повторное развертывание в Azure, щелкните правой кнопкой мыши проект **MyFirstAzureWebApp** в **Обозревателе решений**, а затем выберите **Опубликовать**.
1. На странице **Публикация** со сводными сведениями щелкните **Опубликовать**.

    По завершении публикации Visual Studio открывает в браузере страницу с URL-адресом веб-приложения.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    На странице браузера отобразится обновленное веб-приложение ASP.NET Core 3.1.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio — обновленное веб-приложение ASP.NET Core 3.1 в Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    На странице браузера отобразится обновленное веб-приложение ASP.NET Core 5.0.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio — обновленное веб-приложение ASP.NET Core 5.0 в Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    На странице браузера отобразится обновленное веб-приложение ASP.NET Framework 4.8.

    :::image type="content" source="media/quickstart-dotnet/vs-updated-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/vs-updated-Azure-webapp-net48.png" border="true" alt-text="Visual Studio — обновленное веб-приложение ASP.NET Framework 4.8 в Azure.":::

    ---

:::zone-end

:::zone target="docs" pivot="development-environment-vscode"

1. Откройте *Index.cshtml*.
1. Замените первый элемент `<div>` следующим кодом:

    ```razor
    <div class="jumbotron">
        <h1>.NET 💜 Azure</h1>
        <p class="lead">Example .NET app to Azure App Service.</p>
    </div>
    ```

   Сохраните изменения.

1. Откройте **боковую панель** в Visual Studio Code и щелкните значок **Azure**, чтобы развернуть доступные элементы.
1. В узле **Служба приложений** разверните нужную подписку и щелкните правой кнопкой мыши **MyFirstAzureWebApp**.
1. Выберите **Развернуть в веб-приложении**.
1. Щелкните **Развернуть** при появлении запроса.
1. Когда публикация завершится, щелкните **Обзор веб-сайта** в открывшемся уведомлении и нажмите кнопку **Открыть** при появлении запроса.

    ### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

    На странице браузера отобразится обновленное веб-приложение ASP.NET Core 3.1.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio Code — обновленное веб-приложение ASP.NET Core 3.1 в Azure.":::

    ### <a name="net-50"></a>[.NET 5.0](#tab/net50)

    На странице браузера отобразится обновленное веб-приложение ASP.NET Core 5.0.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Visual Studio Code — обновленное веб-приложение ASP.NET Core 5.0 в Azure.":::

    ### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

    На странице браузера отобразится обновленное веб-приложение ASP.NET Framework 4.8.

    :::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net48.png" border="true" alt-text="Visual Studio Code — обновленное веб-приложение ASP.NET Framework 4.8 в Azure.":::

    ---

:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->

В локальном каталоге выберите файл *Index.cshtml*. Замените первый элемент `<div>` следующим кодом:

```razor
<div class="jumbotron">
    <h1>.NET 💜 Azure</h1>
    <p class="lead">Example .NET app to Azure App Service.</p>
</div>
```

Сохраните изменения, а затем повторно разверните приложение с помощью команды `az webapp up`.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

ASP.NET Core 3.1 является кросс-платформенным решением, поэтому измените `<os>` на `linux` или `windows` в зависимости от того, как было развернуто приложение.

```azurecli
az webapp up --os-type <os>
```

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

ASP.NET Core 5.0 является кросс-платформенным решением, поэтому измените `<os>` на `linux` или `windows` в зависимости от того, как было развернуто приложение.

```azurecli
az webapp up --os-type <os>
```

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

ASP.NET Framework 4.8 имеет зависимости от платформы, поэтому всегда размещается в среде Windows.

```azurecli
az webapp up --os-type windows
```

> [!TIP]
> Если вы хотите размещать приложения .NET в Linux, рассмотрите возможность перехода с [ASP.NET Framework на ASP.NET Core](/aspnet/core/migration/proper-to-2x).

---

Эта команда использует значения, которые кэшируются локально в файле *.azure/config*, включая имя приложения, группу ресурсов и план службы приложений.

После завершения развертывания переключитесь в окно браузера, открытое на этапе **перехода в приложение**, и щелкните "Обновить".

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

На странице браузера отобразится обновленное веб-приложение ASP.NET Core 3.1.

:::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Интерфейс командной строки — обновленное веб-приложение ASP.NET Core 3.1 в Azure.":::

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

На странице браузера отобразится обновленное веб-приложение ASP.NET Core 5.0.

:::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net.png" border="true" alt-text="Интерфейс командной строки — обновленное веб-приложение ASP.NET Core 5.0 в Azure.":::

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

На странице браузера отобразится обновленное веб-приложение ASP.NET Framework 4.8.

:::image type="content" source="media/quickstart-dotnet/updated-Azure-webapp-net48.png" lightbox="media/quickstart-dotnet/updated-Azure-webapp-net48.png" border="true" alt-text="Интерфейс командной строки — обновленное веб-приложение ASP.NET Framework 4.8 в Azure.":::

---

:::zone-end

## <a name="manage-the-azure-app"></a>Управление приложением Azure

Чтобы управлять веб-приложением, перейдите на [портал Azure](https://portal.azure.com), найдите и выберите **Службы приложений**.

:::image type="content" source="media/quickstart-dotnetcore/app-services.png" alt-text="Портал Azure Portal — выбор варианта &quot;Службы приложений&quot;.":::

На странице **Службы приложений** выберите имя веб-приложения.

:::image type="content" source="./media/quickstart-dotnetcore/select-app-service.png" alt-text="Портал Azure — страница Служб приложений с выбранным примером веб-приложения.":::

На странице **Обзор** для веб-приложения вы можете выполнять базовые задачи управления: просмотр, завершение, запуск, перезагрузку и удаление. В меню слева есть дополнительные страницы для настройки приложения.

:::image type="content" source="media/quickstart-dotnetcore/web-app-overview-page.png" alt-text="Портал Azure — страница обзора для Службы приложений":::.

<!--
## Clean up resources - H2 added from the next three includes
-->
:::zone target="docs" pivot="development-environment-vs"
[!INCLUDE [Clean-up Portal web app resources](../../includes/clean-up-section-portal-web-app.md)]
:::zone-end

:::zone target="docs" pivot="development-environment-vscode"
[!INCLUDE [Clean-up Portal web app resources](../../includes/clean-up-section-portal-web-app.md)]
:::zone-end

<!-- markdownlint-disable MD044 -->
:::zone target="docs" pivot="development-environment-cli"
<!-- markdownlint-enable MD044 -->
[!INCLUDE [Clean-up CLI resources](../../includes/cli-samples-clean-up.md)]
:::zone-end

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать и развернуть веб-приложение ASP.NET в Службе приложений Azure.

### <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

Переходите к следующей статье, чтобы узнать, как создать приложение .NET Core и подключить его к Базе данных SQL.

> [!div class="nextstepaction"]
> [Руководство. Использование приложения ASP.NET Core с базой данных SQL](tutorial-dotnetcore-sqldb-app.md)

> [!div class="nextstepaction"]
> [Настройка приложений ASP.NET Core 3.1](configure-language-dotnetcore.md)

### <a name="net-50"></a>[.NET 5.0](#tab/net50)

Переходите к следующей статье, чтобы узнать, как создать приложение .NET Core и подключить его к Базе данных SQL.

> [!div class="nextstepaction"]
> [Руководство. Использование приложения ASP.NET Core с базой данных SQL](tutorial-dotnetcore-sqldb-app.md)

> [!div class="nextstepaction"]
> [Настройка приложения ASP.NET Core 5.0](configure-language-dotnetcore.md)

### <a name="net-framework-48"></a>[.NET Framework 4.8](#tab/netframework48)

Перейдите к следующей статье, чтобы узнать, как создать приложение .NET Framework и подключить его к Базе данных SQL.

> [!div class="nextstepaction"]
> [Руководство. Использование приложения ASP.NET с базой данных SQL](app-service-web-tutorial-dotnet-sqldatabase.md)

> [!div class="nextstepaction"]
> [Настройка приложения ASP.NET Framework](configure-language-dotnet-framework.md)

---

[app-service-pricing-tier]: https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
