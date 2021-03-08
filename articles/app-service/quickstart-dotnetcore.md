---
title: Краткое руководство. Создание приложения ASP.NET Core на C#
description: Узнайте, как запустить веб-приложение в Службе приложений Azure, развернув первое приложение ASP.NET Core.
ms.assetid: b1e6bd58-48d1-4007-9d6c-53fd6db061e3
ms.topic: quickstart
ms.date: 11/23/2020
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, seodec18, contperf-fy21q1
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-dotnetcore-uiex
ms.openlocfilehash: 77e0768a617ef79ab8510f88bfdcd41d5647f9bf
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101701643"
---
# <a name="quickstart-create-an-aspnet-core-web-app-in-azure"></a>Краткое руководство. Создание веб-приложения ASP.NET Core в Azure

::: zone pivot="platform-windows"  

Из этого краткого руководства вы узнаете, как создать и развернуть первое веб-приложение ASP.NET Core в [Службе приложений Azure](overview.md). Служба приложений поддерживает приложения .NET 5.0.

Когда вы закончите работу с ним, у вас будет создана группа ресурсов Azure с планом размещения Службы приложений и Службой приложений, где развернуто веб-приложение.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet/) бесплатно.
- Установите <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a> с рабочей нагрузкой **ASP.NET и веб-разработка**.

  Если у вас уже установлена версия Visual Studio 2019, сделайте следующее.

  - Установите последние обновления для Visual Studio, выбрав **Справка** > **Проверить обновления**. Последние обновления содержат пакет SDK для .NET 5.0.
  - Добавьте рабочую нагрузку, выбрав **Инструменты** > **Получить средства и компоненты**.


## <a name="create-an-aspnet-core-web-app"></a>Создание веб-приложения ASP.NET Core

Создайте веб-приложение ASP.NET Core в Visual Studio, выполнив следующие действия:

# <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

1. Откройте Visual Studio и выберите **Создать проект**.

1. В разделе **Создание нового проекта** выберите **Веб-приложение ASP.NET Core** и убедитесь, что в списке для этого варианта указан язык **C#** , а затем щелкните **Далее**.

1. В окне **Настройка нового проекта** присвойте проекту веб-приложения имя *myFirstAzureWebApp* и щелкните **Создать**.

   ![Настройка проекта веб-приложения](./media/quickstart-dotnetcore/configure-web-app-project.png)

1. Вы можете развернуть в Azure веб-приложение ASP.NET Core любого типа, но для этого краткого руководства нам нужен шаблон **Веб-приложение**. В разделе **Проверка подлинности** выберите вариант **Без проверки подлинности** и убедитесь, что остальные варианты не выбраны. Затем выберите **Создать**.

   ![Создание веб-приложения ASP.NET Core](./media/quickstart-dotnetcore/create-aspnet-core-web-app.png) 
   
1. В меню Visual Studio выберите **Отладка** > **Запустить без отладки**, чтобы запустить веб-приложение локально.

   ![Веб-приложение, выполняющееся локально](./media/quickstart-dotnetcore/web-app-running-locally.png)

# <a name="net-50"></a>[.NET 5.0](#tab/net50)

1. Откройте Visual Studio и выберите **Создать проект**.

1. В разделе **Создание нового проекта** выберите **Веб-приложение ASP.NET Core** и убедитесь, что в списке для этого варианта указан язык **C#** , а затем щелкните **Далее**.

1. В окне **Настройка нового проекта** присвойте проекту веб-приложения имя *myFirstAzureWebApp* и щелкните **Создать**.

   ![Настройка проекта веб-приложения](./media/quickstart-dotnetcore/configure-web-app-project.png)

1. Для приложения .NET 5.0 в раскрывающемся списке выберите **ASP.NET Core 5.0**.

1. Вы можете развернуть в Azure веб-приложение ASP.NET Core любого типа, но для этого краткого руководства нам нужен шаблон **Веб-приложение ASP.NET Core**. В разделе **Проверка подлинности** выберите вариант **Без проверки подлинности** и убедитесь, что остальные варианты не выбраны. Затем выберите **Создать**.

   ![Создание веб-приложения ASP.NET Core](./media/quickstart-dotnetcore/create-aspnet-core-web-app-5.png) 
   
1. В меню Visual Studio выберите **Отладка** > **Запустить без отладки**, чтобы запустить веб-приложение локально.

   ![Веб-приложение, выполняющееся локально](./media/quickstart-dotnetcore/web-app-running-locally.png)

---

## <a name="publish-your-web-app"></a>Публикация веб-приложения

Прежде чем опубликовать веб-приложение, следует создать и настроить новую Службу приложений, в которой вы сможете опубликовать это приложение. 

В процессе настройки Службы приложений вы создадите следующее:

- Новая [группа ресурсов](../azure-resource-manager/management/overview.md#terminology) для всех ресурсов Azure, которые потребуются для этой службы.
- Новый [план размещения](./overview-hosting-plans.md), который позволяет определить расположение, размер и функции фермы веб-серверов для размещения приложения.

Выполните следующие действия, чтобы создать Службу приложений и опубликовать веб-приложение:

1. Щелкните правой кнопкой мыши проект **myFirstAzureWebApp** в **обозревателе решений** и выберите **Опубликовать**. 

1. В разделе **Публикация** выберите **Azure** и нажмите кнопку **Далее**.

1. Доступные параметры зависят от того, вошли ли вы в Azure и есть ли у вас учетная запись Visual Studio, связанная с учетной записью Azure. Выберите **Добавить учетную запись** или **Войти**, чтобы войти в подписку Azure. Если вы уже вошли, выберите нужную учетную запись.

   ![Вход в Azure](./media/quickstart-dotnetcore/sign-in-azure-vs2019.png)

1. Справа от **экземпляров Службы приложений** щелкните **+** .

   ![Новое приложение Службы приложений](./media/quickstart-dotnetcore/publish-new-app-service.png)

1. Для параметра **Подписка** подтвердите предложенный вариант или выберите другой из раскрывающегося списка.

1. В разделе **Группа ресурсов** выберите **Создать**. В разделе **Новое имя группы ресурсов** введите *myResourceGroup* и щелкните **ОК**. 

1. В разделе **План размещения** щелкните **Создать**. 

1. В диалоговом окне **План размещения. Создать новый** введите значения, указанные в следующей таблице.

   | Параметр  | Рекомендуемое значение | Описание |
   | -------- | --------------- | ----------- |
   | **План размещения**  | *myFirstAzureWebAppPlan* | Имя плана службы приложений. |
   | **Расположение**      | *Западная Европа* | Центр обработки данных, где размещается веб-приложение. |
   | **Размер**          | *Бесплатный* | [Ценовая категория](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) определяет возможности размещения. |
   
   ![Создание нового плана размещения](./media/quickstart-dotnetcore/create-new-hosting-plan-vs2019.png)

1. В поле **Имя** введите уникальное имя приложения, включающее только допустимые символы: `a-z`, `A-Z`, `0-9` и `-`. Вы можете использовать автоматически созданное уникальное имя. URL-адрес веб-приложения: `http://<app-name>.azurewebsites.net`, где `<app-name>` — имя приложения.

2. Выберите **Создать**, чтобы создать ресурсы Azure.

   ![Создание ресурсов приложения](./media/quickstart-dotnetcore/web-app-name-vs2019.png)

   После завершения работы мастера ресурсы Azure будут созданы и готовы к публикации.

3. Выберите **Готово**, чтобы закрыть мастер.

1. На странице **Публикация** щелкните **Опубликовать**. Visual Studio создает, упаковывает и публикует приложение в Azure, а затем запускает его в браузере по умолчанию.

   ![Опубликованное веб-приложение ASP.NET, работающее в Azure](./media/quickstart-dotnetcore/web-app-running-live.png)

**Поздравляем!** Ваше веб-приложение ASP.NET Core работает в Службе приложений Azure в реальном времени.

## <a name="update-the-app-and-redeploy"></a>Обновление и повторное развертывание приложения

Чтобы обновить и повторно развернуть веб-приложение, сделайте следующее:

1. В **обозревателе решений** в проекте откройте **Страницы** > **Index.cshtml**.

1. Замените весь тег `<div>` следующим кодом:

   ```html
   <div class="jumbotron">
       <h1>ASP.NET in Azure!</h1>
       <p class="lead">This is a simple app that we've built that demonstrates how to deploy a .NET app to Azure App Service.</p>
   </div>
   ```

1. Чтобы выполнить повторное развертывание в Azure, щелкните правой кнопкой мыши проект **myFirstAzureWebApp** в **обозревателе решений**, а затем выберите **Опубликовать**.

1. На странице **Публикация** со сводными сведениями щелкните **Опубликовать**.

   <!-- ![Publish update to web app](./media/quickstart-dotnetcore/publish-update-to-web-app-vs2019.png) -->

    По завершении публикации Visual Studio открывает в браузере страницу с URL-адресом веб-приложения.

    ![Обновленное веб-приложение ASP.NET, работающее в Azure](./media/quickstart-dotnetcore/updated-web-app-running-live.png)

## <a name="manage-the-azure-app"></a>Управление приложением Azure

Чтобы управлять веб-приложением, перейдите на [портал Azure](https://portal.azure.com), найдите и выберите **Службы приложений**.

![Выбор служб приложений](./media/quickstart-dotnetcore/app-services.png)

На странице **Службы приложений** выберите имя веб-приложения.

:::image type="content" source="./media/quickstart-dotnetcore/select-app-service.png" alt-text="Снимок экрана: страница служб приложений с примером выбранного веб-приложения.":::

На странице **Обзор** для веб-приложения вы можете выполнять базовые задачи управления: просмотр, завершение, запуск, перезагрузку и удаление. В меню слева есть дополнительные страницы для настройки приложения.

![Служба приложений на портале Azure](./media/quickstart-dotnetcore/web-app-overview-page.png)

[!INCLUDE [Clean-up section](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы применили Visual Studio, чтобы создать и развернуть веб-приложение ASP.NET Core в Службе приложений Azure.

Переходите к следующей статье, чтобы узнать, как создать приложение .NET Core и подключить его к Базе данных SQL.

> [!div class="nextstepaction"]
> [Использование ASP.NET Core с базой данных SQL](tutorial-dotnetcore-sqldb-app.md)

> [!div class="nextstepaction"]
> [Настройка приложения ASP.NET Core](configure-language-dotnetcore.md)

::: zone-end  

::: zone pivot="platform-linux"
[Служба приложений на платформе Linux](overview.md#app-service-on-linux) — это высокомасштабируемая служба размещения с самостоятельной установкой исправлений на основе операционной системы Linux. В этом кратком руководстве показано, как создать приложение [.NET Core](/aspnet/core/) и развернуть его в размещенной Службе приложений Linux с помощью [Azure CLI](/cli/azure/get-started-with-azure-cli).

![Пример приложения, выполняющегося в Azure](media/quickstart-dotnetcore/dotnet-browse-azure.png)

Выполните инструкции, приведенные в этом руководстве, с помощью компьютера Mac, Windows или Linux.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-initial-environment"></a>Настройка начальной среды

# <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

Для работы с этим кратким руководством сделайте следующее:

* <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Установите последний пакет SDK для .NET Core 3.1.</a>
* <a href="/cli/azure/install-azure-cli" target="_blank">Установите последнюю версию Azure CLI</a>.

# <a name="net-50"></a>[.NET 5.0](#tab/net50)

Для работы с этим кратким руководством сделайте следующее:

* <a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">Установите последний пакет SDK для .NET Core 5.0.</a>
* <a href="/cli/azure/install-azure-cli" target="_blank">Установите последнюю версию Azure CLI</a>.

---

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

## <a name="create-the-app-locally"></a>Локальное создание приложения

В окне терминала на компьютере создайте каталог `hellodotnetcore` и перейдите в него.

```bash
mkdir hellodotnetcore
cd hellodotnetcore
```

Создание нового приложения .NET Core

```bash
dotnet new web
```

## <a name="run-the-app-locally"></a>Локальный запуск приложения

Запустите приложение локально, чтобы увидеть, как оно будет выглядеть после развертывания в Azure. 

```bash
dotnet run
```

Откройте веб-браузер и перейдите к приложению в `http://localhost:5000`.

На странице отобразится сообщение **Hello World** из примера приложения.

![Тестирование с помощью браузера](media/quickstart-dotnetcore/dotnet-browse-local.png)

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

## <a name="sign-into-azure"></a>Вход в Azure
В окне терминала войдите в Azure с помощью следующей команды:

```azurecli
az login
```

## <a name="deploy-the-app"></a>Развертывание приложения

Разверните код в локальной папке (*hellodotnetcore*) с помощью команды `az webapp up`.

```azurecli
az webapp up --sku F1 --name <app-name>
```

- Если команда `az` не распознана, проверьте, установили ли вы Azure CLI согласно сведениям, указанным в разделе [Настройка начальной среды](#set-up-your-initial-environment).
- Замените `<app-name>` именем, уникальным для всех регионов Azure (*допустимыми символами являются `a-z`, `0-9`и `-`* ). Рекомендуется использовать сочетание названия компании и идентификатора приложения.
- Аргумент `--sku F1` создает веб-приложение в ценовой категории "Бесплатный". Этот аргумент можно опустить, чтобы использовать более быструю ценовую категорию "Премиум" с почасовой оплатой.
- При необходимости вы можете использовать аргумент `--location <location-name>`, где `<location-name>` является доступным регионом Azure. Список допустимых регионов для учетной записи Azure можно получить, выполнив команду [`az account list-locations`](/cli/azure/appservice#az-appservice-list-locations).

Выполнение этой команды может занять несколько минут. Во время выполнения она предоставляет сообщения о создании группы ресурсов, плане службы приложений и приложении размещения, настройке ведения журнала и последующем выполнении развертывания ZIP-файла. Затем оно выдает сообщение You can launch the app at http://&lt;app-name&gt;.azurewebsites.net (Вы можете запустить приложение по адресу http://<app-name>.azurewebsites.net). Это URL-адрес приложения в Azure.

# <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

![Пример выходных данных команды az webapp up](./media/quickstart-dotnetcore/az-webapp-up-output-3.1.png)

# <a name="net-50"></a>[.NET 5.0](#tab/net50)

<!-- Deploy the code in your local folder (*hellodotnetcore*) using the `az webapp up` command:

```azurecli
az webapp up --sku B1 --name <app-name> --os-type linux
```

- If the `az` command isn't recognized, be sure you have the Azure CLI installed as described in [Set up your initial environment](#set-up-your-initial-environment).
- Replace `<app-name>` with a name that's unique across all of Azure (*valid characters are `a-z`, `0-9`, and `-`*). A good pattern is to use a combination of your company name and an app identifier.
- The `--sku B1` argument creates the web app in the Basic pricing tier, which incurs an hourly cost. Omit this argument to use a faster premium tier, which costs more.
- You can optionally include the argument `--location <location-name>` where `<location-name>` is an available Azure region. You can retrieve a list of allowable regions for your Azure account by running the [`az account list-locations`](/cli/azure/appservice#az-appservice-list-locations) command.

The command may take a few minutes to complete. While running, it provides messages about creating the resource group, the App Service plan and hosting app, configuring logging, then performing ZIP deployment. It then gives the message, "You can launch the app at http://&lt;app-name&gt;.azurewebsites.net", which is the app's URL on Azure. -->

![Пример выходных данных команды az webapp up](./media/quickstart-dotnetcore/az-webapp-up-output-5.0.png)

---

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

[!include [az webapp up command note](../../includes/app-service-web-az-webapp-up-note.md)]

## <a name="browse-to-the-app"></a>Переход в приложение

Перейдите в развертываемое приложение с помощью веб-браузера.

```bash
http://<app_name>.azurewebsites.net
```

Пример кода .NET Core выполняется в службе приложений в Linux со встроенным образом.

![Пример приложения, выполняющегося в Azure](media/quickstart-dotnetcore/dotnet-browse-azure.png)

**Поздравляем!** Вы развернули свое первое приложение .NET Core в службе приложений в Linux.

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

## <a name="update-and-redeploy-the-code"></a>Обновление и повторное развертывание кода

В локальном каталоге откройте файл _Startup.cs_. Внесите некоторые изменения в текст в вызове метода `context.Response.WriteAsync`:

```csharp
await context.Response.WriteAsync("Hello Azure!");
```

Сохраните изменения, а затем повторно разверните приложение с помощью команды `az webapp up`.

```azurecli
az webapp up --os-type linux
```

Эта команда использует значения, которые кэшируются локально в файле *.azure/config*, включая имя приложения, группу ресурсов и план службы приложений.

После завершения развертывания переключитесь в окно браузера, открытое на этапе **перехода в приложение**, и щелкните "Обновить".

![Обновленный пример приложения, выполняющегося в Azure](media/quickstart-dotnetcore/dotnet-browse-azure-updated.png)

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

## <a name="manage-your-new-azure-app"></a>Управление новым приложением Azure

Перейдите на <a href="https://portal.azure.com" target="_blank">портал Azure</a>, чтобы управлять созданным приложением.

В меню слева щелкните **Службы приложений**, а затем — имя своего приложения Azure.

:::image type="content" source="./media/quickstart-dotnetcore/portal-app-service-list-up.png" alt-text="Снимок экрана: страница служб приложений, на которой выбран пример приложения Azure.":::

Отобразится страница обзора вашего приложения. Вы можете выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление. 

![Страница службы приложений на портале Azure](media/quickstart-dotnetcore/portal-app-overview-up.png)

В меню слева доступно несколько страниц для настройки приложения. 

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. по приложению ASP.NET Core с Базой данных SQL](tutorial-dotnetcore-sqldb-app.md)

> [!div class="nextstepaction"]
> [Настройка приложения ASP.NET Core](configure-language-dotnetcore.md)

::: zone-end
