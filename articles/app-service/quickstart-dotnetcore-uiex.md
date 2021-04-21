---
title: Краткое руководство. Создание приложения ASP.NET Core на C#
description: Узнайте, как запустить веб-приложение в Службе приложений Azure, развернув первое приложение ASP.NET Core.
ms.assetid: b1e6bd58-48d1-4007-9d6c-53fd6db061e3
ms.topic: quickstart
ms.date: 11/23/2020
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, seodec18, contperf-fy21q1
zone_pivot_groups: app-service-platform-windows-linux
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: ff86bedf47395b50dc25e552b8b3ed4176e23b65
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769109"
---
# <a name="quickstart-create-an-aspnet-core-web-app-in-azure"></a>Краткое руководство. Создание веб-приложения ASP.NET Core в Azure

::: zone pivot="platform-windows"  

Из этого краткого руководства вы узнаете, как создать и развернуть ваше первое веб-приложение ASP.NET Core в <abbr title="Служба на основе HTTP для размещения веб-приложений, REST API и мобильных внутренних приложений.">Служба приложений Azure</abbr>. Служба приложений поддерживает приложения .NET 5.0.

По завершении вы получите <abbr title="Логический контейнер для связанных ресурсов Azure, которыми можно управлять как единым целым.">resource group</abbr>Azure, состоящую из <abbr title="План, который позволяет определить расположение, размер и функции фермы веб-серверов для размещения приложения.">План службы приложений</abbr> и <abbr title="Представление веб-приложения, которое содержит код приложения, имена узлов DNS, сертификаты и связанные ресурсы.">Приложение службы приложений</abbr> с развернутым примером приложения ASP.NET Core.

<hr/> 

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

- **Получение учетной записи Azure** с активной <abbr title="Базовая организационная структура, в которой можно управлять ресурсами в Azure, обычно связанными с частным лицом или отделом в пределах организации.">Подписка</abbr>. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet/) бесплатно.
- **Установите <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a>** с рабочей нагрузкой **ASP.NET и веб-разработки**.

<details>
<summary>Уже есть Visual Studio 2019?</summary>
Если у вас уже установлена версия Visual Studio 2019, сделайте следующее.

<ul>
<li><strong>Установите последние обновления</strong> для Visual Studio, выбрав <strong>Справка</strong> &gt; <strong>Проверить обновления</strong>. Последние обновления содержат пакет SDK для .NET 5.0.</li>
<li><strong>Добавьте рабочую нагрузку</strong>, выбрав <strong>Инструменты</strong> &gt; <strong>Получить инструменты и возможности</strong>.</li>
</ul>
</details>

<hr/> 

## <a name="2-create-an-aspnet-core-web-app"></a>2. Создание веб-приложения ASP.NET Core

1. Откройте Visual Studio и выберите **Создать проект**.

1. В разделе **Создание нового проекта** выберите **Веб-приложение ASP.NET Core** и убедитесь, что в списке для этого варианта указан язык **C#** , а затем щелкните **Далее**.

1. В окне **Настройка нового проекта** присвойте проекту веб-приложения имя *myFirstAzureWebApp* и щелкните **Создать**.

   ![Настройка проекта веб-приложения](./media/quickstart-dotnetcore/configure-web-app-project.png)

1. Для приложения .NET 5.0 в раскрывающемся списке выберите **ASP.NET Core 5.0**. В противном случае используйте значение по умолчанию.

1. Вы можете развернуть в Azure веб-приложение ASP.NET Core любого типа, но для этого краткого руководства нам нужен шаблон **Веб-приложение ASP.NET Core**. В разделе **Проверка подлинности** выберите вариант **Без проверки подлинности** и убедитесь, что остальные варианты не выбраны. Затем выберите **Создать**.

   ![Создание веб-приложения ASP.NET Core](./media/quickstart-dotnetcore/create-aspnet-core-web-app-5.png) 
   
1. В меню Visual Studio выберите **Отладка** > **Запустить без отладки**, чтобы запустить веб-приложение локально.

   ![Веб-приложение, выполняющееся локально](./media/quickstart-dotnetcore/web-app-running-locally.png)

<hr/> 

## <a name="3-publish-your-web-app"></a>3. Публикация веб-приложения

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

   | Параметр  | Рекомендуемое значение |
   | -------- | --------------- |
   | **План размещения**  | *myFirstAzureWebAppPlan* |
   | **Расположение**      | *Западная Европа* |
   | **Размер**          | *Бесплатный* |
   
   ![Создание нового плана размещения](./media/quickstart-dotnetcore/create-new-hosting-plan-vs2019.png)

1. В поле **Имя** введите уникальное имя приложения.

    <details>
        <summary>Какие символы можно использовать?</summary>
        Допустимые символы: a–z, A–Z, 0–9 и -. Вы можете использовать автоматически созданное уникальное имя. URL-адрес веб-приложения: http://<code>&lt;app-name&gt;.azurewebsites.net</code>, где <code>&lt;app-name&gt;</code> — имя приложения.
    </details>

1. Выберите **Создать**, чтобы создать ресурсы Azure. 

   ![Создание ресурсов приложения](./media/quickstart-dotnetcore/web-app-name-vs2019.png)

1. Подождите, пока мастер закончит создание ресурсов Azure. Выберите **Готово**, чтобы закрыть мастер.

1. На странице **Публикация** щелкните **Опубликовать**, чтобы развернуть проект. 

    <details>
        <summary>Что делает Visual Studio?</summary>
        Visual Studio создает, упаковывает и публикует приложение в Azure, а затем запускает его в браузере по умолчанию.
    </details>

   ![Опубликованное веб-приложение ASP.NET, работающее в Azure](./media/quickstart-dotnetcore/web-app-running-live.png)

<hr/> 

## <a name="4-update-the-app-and-redeploy"></a>4. Обновление и повторное развертывание приложения

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

<hr/> 

## <a name="5-manage-the-azure-app"></a>5. Управление приложением Azure

1. Перейдите на [портал Azure](https://portal.azure.com), после чего найдите и выберите **Службы приложений**.

    ![Выбор служб приложений](./media/quickstart-dotnetcore/app-services.png)
    
1. На странице **Службы приложений** выберите имя веб-приложения.

    :::image type="content" source="./media/quickstart-dotnetcore/select-app-service.png" alt-text="Снимок экрана: страница служб приложений с примером выбранного веб-приложения.":::

1. На странице **Обзор** для веб-приложения вы можете выполнять базовые задачи управления: просмотр, завершение, запуск, перезагрузку и удаление. В меню слева есть дополнительные страницы для настройки приложения.

    ![Служба приложений на портале Azure](./media/quickstart-dotnetcore/web-app-overview-page.png)
    
<hr/> 

## <a name="6-clean-up-resources"></a>6. Очистка ресурсов

1. В меню или на странице **Главная** портала Azure выберите **Группы ресурсов**. Затем на странице **Группы ресурсов** выберите **myResourceGroup**.

1. На странице **myResourceGroup** убедитесь, что перечислены те ресурсы, которые нужно удалить.

1. Выберите **Удалить группу ресурсов**, введите **myResourceGroup** в текстовое поле для подтверждения и щелкните **Удалить**.

<hr/> 

## <a name="next-steps"></a>Дальнейшие действия

Переходите к следующей статье, чтобы узнать, как создать приложение .NET Core и подключить его к Базе данных SQL.

- [Использование ASP.NET Core с базой данных SQL](tutorial-dotnetcore-sqldb-app.md)
- [Настройка приложения ASP.NET Core](configure-language-dotnetcore.md)

::: zone-end  

::: zone pivot="platform-linux"
В этом кратком руководстве показано, как создать приложение [.NET Core](/aspnet/core/) в <abbr title="Служба приложений на платформе Linux — это высокомасштабируемая служба размещения с самостоятельной установкой исправлений на основе операционной системы Linux.">Служба приложений в Linux</abbr>. Создайте приложение с помощью [Azure CLI](/cli/azure/get-started-with-azure-cli) и разверните код .NET Core в приложении с помощью Git.

<hr/> 

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

- **Получите учетную запись Azure** с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet/) бесплатно.
- **Установите** последний <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">пакет SDK для .NET Core 3.1</a> или <a href="https://dotnet.microsoft.com/download/dotnet/5.0" target="_blank">пакет SDK для .NET 5.0</a>.
- **<a href="/cli/azure/install-azure-cli" target="_blank">Установите последнюю версию Azure CLI</a>** .

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="2-create-the-app-locally"></a>2. Локальное создание приложения

1. Выполните `mkdir hellodotnetcore`, чтобы создать каталог.

    ```bash
    mkdir hellodotnetcore
    ```

1. Выполните `cd hellodotnetcore`, чтобы изменить каталог. 

    ```bash
    cd hellodotnetcore
    ```

1. Выполните `dotnet new web`, чтобы создать новое приложение .NET Core.

    ```bash
    dotnet new web
    ```

<hr/> 

## <a name="3-run-the-app-locally"></a>3. Локальный запуск приложения

1. Выполните `dotnet run`, чтобы увидеть, как выглядит приложение, развернутое в Azure.

    ```bash
    dotnet run
    ```
    
1. **Откройте веб-браузер** и перейдите к приложению в `http://localhost:5000`.

![Тестирование с помощью браузера](media/quickstart-dotnetcore/dotnet-browse-local.png)

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="4-sign-into-azure"></a>4. Вход в Azure

Выполните `az login`, чтобы войти в Azure.

```azurecli
az login
```

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="5-deploy-the-app"></a>5. Развертывание приложения

1. **Выполните** `az webapp up` в локальной папке. **Замените** <app-name> на глобально уникальное имя.

    ```azurecli
    az webapp up --sku F1 --name <app-name> --os-type linux
    ```
    
    <details>
    <summary>Устранение неполадок</summary>
    <ul>
    <li>Если команда <code>az</code> не распознана, проверьте, установили ли вы Azure CLI согласно сведениям, указанным в разделе <a href="#1-prepare-your-environment">Подготовка среды</a>.</li>
    <li>Замените <code>&lt;app-name&gt;</code> именем, уникальным для всех регионов Azure (<em>допустимыми символами являются <code>a-z</code>, <code>0-9</code>и <code>-</code></em>). Рекомендуется использовать сочетание названия компании и идентификатора приложения.</li>
    <li>Аргумент <code>--sku F1</code> создает веб-приложение в ценовой категории "Бесплатный". Этот аргумент можно опустить, чтобы использовать более быструю ценовую категорию "Премиум" с почасовой оплатой.</li>
    <li>При необходимости вы можете использовать аргумент <code>--location &lt;location-name&gt;</code>, где <code>&lt;location-name&gt;</code> является доступным регионом Azure. Список допустимых регионов для учетной записи Azure можно получить, выполнив команду <a href="/cli/azure/appservice#az_appservice_list_locations"><code>az account list-locations</code></a>.</li>
    </ul>
    </details>
    
1. Дождитесь завершения выполнения команды. Выполнение может занять несколько минут и заканчивается сообщением "Приложение можно запустить по ссылке http://&lt;app-name&gt;.azurewebsites.net".

    <details>
    <summary>Что делает <code>az webapp up</code>?</summary>
    <p>Команда <code>az webapp up</code> выполняет следующие действия:</p>
    <ul>
    <li>создание группы ресурсов по умолчанию;</li>
    <li>создание плана Службы приложений по умолчанию;</li>
    <li><a href="/cli/azure/webapp#az_webapp_create">создание приложения Службы приложений</a> с указанным именем;</li>
    <li><a href="/azure/app-service/deploy-zip">развертывание ZIP-файлов</a> для приложения из текущего рабочего каталога.</li>
    <li>При выполнении оно предоставляет сообщения о создании ресурсов, ведении журналов и развертывании из ZIP-файлов.</li>
    </ul>
    </details>
    
# <a name="net-core-31"></a>[.NET Core 3.1](#tab/netcore31)

![Пример выходных данных команды az webapp up](./media/quickstart-dotnetcore/az-webapp-up-output-3.1.png)

# <a name="net-50"></a>[.NET 5.0](#tab/net50)

![Пример выходных данных команды az webapp up](./media/quickstart-dotnetcore/az-webapp-up-output-5.0.png)

---

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="6-browse-to-the-app"></a>6. Переход в приложение

**Перейдите в развернутое приложение** с помощью веб-браузера.

```bash
http://<app_name>.azurewebsites.net
```

![Пример приложения, выполняющегося в Azure](media/quickstart-dotnetcore/dotnet-browse-azure.png)

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="7-update-and-redeploy-the-code&quot;></a>7. Обновление и повторное развертывание кода

1. **Откройте файл _Startup.cs_** в локальном каталоге. 

1. **Внесите небольшое изменение** в текст в вызове метода `context.Response.WriteAsync`.

    ```csharp
    await context.Response.WriteAsync(&quot;Hello Azure!");
    ```
    
1. **Сохраните изменения**.

1. **Выполните** `az webapp up`, чтобы повторить развертывание:

    ```azurecli
    az webapp up --os-type linux
    ```
    
    <details>
    <summary>Что делает <code>az webapp up</code> на этот раз?</summary>
    При первом выполнении команды она сохранила имя приложения, группу ресурсов и план службы приложений в файле <i>.azure/config</i> из корневой папки проекта. При повторном выполнении из корневой папки проекта команда использует значения, сохраненные в <i>.azure/config</i>, определяет, что ресурсы службы приложений уже существуют, и повторно выполняет развертывание из ZIP-файла.
    </details>
    
1. По завершении развертывания **нажмите кнопку "Обновить"** в ранее открытом окне браузера.

    ![Обновленный пример приложения, выполняющегося в Azure](media/quickstart-dotnetcore/dotnet-browse-azure-updated.png)
    
[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="8-manage-your-new-azure-app"></a>8. Управление новым приложением Azure

1. Перейдите на <a href="https://portal.azure.com" target="_blank">портал Microsoft Azure</a>.

1. В меню слева щелкните **Службы приложений**, а затем — имя своего приложения Azure.

    :::image type="content" source="./media/quickstart-dotnetcore/portal-app-service-list-up.png" alt-text="Снимок экрана: страница служб приложений, на которой выбран пример приложения Azure.":::

1. На странице обзора можно выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление. В меню слева доступно несколько страниц для настройки приложения. 

    ![Страница службы приложений на портале Azure](media/quickstart-dotnetcore/portal-app-overview-up.png)
    
<hr/> 

## <a name="9-clean-up-resources"></a>9. Очистка ресурсов

**Выполните** `az group delete --name myResourceGroup`, чтобы удалить группу ресурсов.

```azurecli-interactive
az group delete --name myResourceGroup
```

[Возникли проблемы? Сообщите нам!](https://aka.ms/DotNetAppServiceLinuxQuickStart)

<hr/> 

## <a name="next-steps"></a>Дальнейшие действия

- [Руководство. по приложению ASP.NET Core с Базой данных SQL](tutorial-dotnetcore-sqldb-app.md)
- [Настройка приложения ASP.NET Core](configure-language-dotnetcore.md)

::: zone-end
