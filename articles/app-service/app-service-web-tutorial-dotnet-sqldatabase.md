---
title: Руководство по использованию приложения ASP.NET со службой "База данных Azure SQL"
description: Узнайте, как развернуть приложение C# ASP.NET в Azure и в службу "База данных Azure SQL"
ms.assetid: 03c584f1-a93c-4e3d-ac1b-c82b50c75d3e
ms.devlang: csharp
ms.topic: tutorial
ms.date: 03/18/2021
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, seodec18
ms.openlocfilehash: 533bd817b704db9976624b356a9950a9c48b8339
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104606025"
---
# <a name="tutorial-deploy-an-aspnet-app-to-azure-with-azure-sql-database"></a>Руководство по развертыванию приложения ASP.NET в Azure со службой "База данных SQL Azure"

[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости. В этом руководстве показано, как развернуть управляемое данными приложение ASP.NET в Службе приложений, а затем подключить его к [Базе данных SQL Azure](../azure-sql/database/sql-database-paas-overview.md). Выполнив описанные здесь действия, вы получите приложение ASP.NET, запущенное в Azure и подключенное к Базе данных SQL.

![Опубликованное приложение ASP.NET в Службе приложений Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Создание базы данных в службе "База данных SQL Azure"
> * Подключение приложения ASP.NET к базе данных SQL.
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов из Azure в окно терминала.
> * Управление приложением на портале Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

Установите <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2019</a> с рабочей нагрузкой **ASP.NET и веб-разработка**.

Если вы уже установили Visual Studio, добавьте в него рабочие нагрузки. Для этого последовательно выберите **Инструменты** > **Get Tools and Features (Получить инструменты и функции)** .

## <a name="download-the-sample"></a>Скачивание примера приложения

1. [Загрузите пример проекта](https://github.com/Azure-Samples/dotnet-sqldb-tutorial/archive/master.zip).

1. Извлеките (распакуйте) файл *dotnet-sqldb-tutorial-master.zip*.

Этот пример проекта содержит простое CRUD-приложение [ASP.NET MVC](https://www.asp.net/mvc), созданное на основе [Entity Framework Code First](/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application).

### <a name="run-the-app"></a>Запустите приложение

1. Откройте файл *dotnet-sqldb-tutorial-master/DotNetAppSqlDb.sln* в Visual Studio.

1. Введите `Ctrl+F5`, чтобы запустить приложение без отладки. Это приложение откроется в браузере по умолчанию.

1. Щелкните ссылку **Создать**, чтобы создать несколько элементов *списка дел*.

    ![Диалоговое окно "Новый проект ASP.NET"](media/app-service-web-tutorial-dotnet-sqldatabase/local-app-in-browser.png)

1. Проверьте ссылки **Изменить**, **Сведения** и **Удалить**.

Чтобы подключиться к базе данных, приложение использует контекст базы данных. В этом примере контекст базы данных использует строку подключения `MyDbConnection`. Строка подключения определена в файле *Web.config*. Ссылка на нее также имеется в файле *Models/MyDatabaseContext.cs*. Имя строки подключения потребуется далее в этом руководстве при подключении приложения Azure к базе данных SQL.

## <a name="publish-aspnet-application-to-azure"></a>Публикация приложения ASP.NET в Azure

1. В **обозревателе решений** щелкните правой кнопкой мыши проект **DotNetAppSqlDb** и выберите **Опубликовать**.

    ![Публикация в обозревателе решений](./media/app-service-web-tutorial-dotnet-sqldatabase/solution-explorer-publish.png)

1. Выберите **Azure** в качестве целевого объекта и нажмите кнопку **Далее**.

1. Выберите **Служба приложений Azure (Windows)** и нажмите кнопку **Далее**.

#### <a name="sign-in-and-add-an-app"></a>Вход и добавление приложения

1. В диалоговом окне **Публикация** в раскрывающемся списке диспетчера учетных записей щелкните команду **Добавить учетную запись**.

1. Войдите в подписку Azure. Если вы уже вошли в учетную запись Майкрософт, проверьте, содержит ли она подписку Azure. Если подписки нет, щелкните ее, чтобы добавить правильную учетную запись.

1. В области **App Service instances** (Экземпляры Службы приложений) щелкните **+** .

    ![Вход в Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/sign-in-azure.png)

#### <a name="configure-the-web-app-name"></a>Настройка имени веб-приложения

Вы можете использовать созданное имя веб-приложения или присвоить ему уникальное имя (допустимые символы: `a-z`, `0-9` и `-`). Это имя используется как часть URL-адреса по умолчанию для приложения (`<app_name>.azurewebsites.net`, где `<app_name>` — имя вашего веб-приложения). Оно должно быть глобально уникальным среди всех приложений Azure.

> [!NOTE]
> Пока не нажимайте кнопку **Создать**.

![Диалоговое окно "Создание службы приложений"](media/app-service-web-tutorial-dotnet-sqldatabase/wan.png)

#### <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [resource-group](../../includes/resource-group.md)]

1. Рядом с **группой ресурсов** щелкните **Создать**.

   ![Рядом с группой ресурсов щелкните "Создать".](media/app-service-web-tutorial-dotnet-sqldatabase/new_rg2.png)

1. Присвойте группе ресурсов имя **myResourceGroup**.

#### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

[!INCLUDE [app-service-plan](../../includes/app-service-plan.md)]

1. Рядом с полем **План размещения** щелкните команду **Создать**.

1. В диалоговом окне **Настроить план службы приложений** настройте новый план службы приложений, задав приведенные ниже параметры, и нажмите кнопку **ОК**.

   | Параметр  | Рекомендуемое значение | Дополнительные сведения |
   | ----------------- | ------------ | ----|
   |**План службы приложений**| myAppServicePlan | [Планы службы приложений](../app-service/overview-hosting-plans.md) |
   |**Расположение**| Западная Европа | [Регионы Azure](https://azure.microsoft.com/regions/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) |
   |**Размер**| Бесплатный | [Ценовые категории](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)|

   ![Создание плана службы приложений](./media/app-service-web-tutorial-dotnet-sqldatabase/configure-app-service-plan.png)

1. Щелкните команду **Создать** и дождитесь создания ресурсов Azure.

1. В диалоговом окне **Опубликовать** отобразятся настроенные ресурсы. Нажмите кнопку **Готово**.

   ![Созданные ресурсы](media/app-service-web-tutorial-dotnet-sqldatabase/app_svc_plan_done.png)


#### <a name="create-a-server-and-database"></a>Создание сервера и базы данных

Перед созданием базы данных необходимо создать [логический сервер SQL](../azure-sql/database/logical-servers.md). Логический сервер SQL — это логическая конструкция, которая содержит группу баз данных, которыми можно управлять как группой.

1. В диалоговом окне **Публикация** прокрутите вниз до раздела **Зависимости служб**. Рядом с элементом **База данных SQL Server** нажмите кнопку **Настроить**.

   ![Настройка зависимости от Базы данных SQL](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sqldb-dependency.png)

1. Выберите элемент **База данных SQL Azure** и нажмите кнопку **Далее**.

1. В диалоговом окне **Configure Azure SQL Database** (Настройка Базы данных SQL Azure) щелкните **+** .

1. Рядом с элементом **Сервер базы данных** нажмите кнопку **Создать**.

   Создастся имя сервера. Это имя используется как часть URL-адреса по умолчанию для сервера (`<server_name>.database.windows.net`). Оно должно быть уникальным на всех серверах в Azure SQL. Имя сервера можно изменить, но в рамках этого руководства используйте созданное значение.

1. Добавьте имя пользователя и пароль администратора. Требования к сложности пароля см. в статье [Политика паролей](/sql/relational-databases/security/password-policy).

   Запомните это имя пользователя и пароль. Они потребуются позже для управления сервером.

   ![Создание сервера](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database-server.png)

   > [!IMPORTANT]
   > Хотя пароль в строке подключения скрывается (в Visual Studio и в службе приложений), он все равно где-то сохраняется, что делает приложение уязвимым перед атаками. В таком случае служба приложений может использовать [управляемые удостоверения службы](overview-managed-identity.md), полностью устраняя необходимость хранить секреты в коде или конфигурации приложения. Дополнительные сведения см. в разделе [Дальнейшие действия](#next-steps).

1. Нажмите кнопку **ОК**.

1. В диалоговом окне **База данных SQL Azure** оставьте созданное по умолчанию **имя базы данных**. Щелкните команду **Создать** и дождитесь создания ресурсов базы данных.

    ![Настройка базы данных](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database.png)

#### <a name="configure-database-connection"></a>Настройка подключения к базе данных

1. Когда мастер завершит создание ресурсов базы данных, нажмите кнопку **Далее**.

1. В поле **Имя строки подключения базы данных** введите _MyDbConnection_. Это имя должно совпадать с именем строки подключения, указанном в _Models\MyDatabaseContext.cs_.

1. В полях **Database connection user name** (Имя пользователя для подключения базы данных) и **Database connection password** (Пароль для подключения базы данных) введите имя пользователя и пароль администратора, которые вы использовали при [создании сервера](#create-a-server-and-database).

1. Выберите элемент **Параметры приложений Azure** и нажмите кнопку **Готово**.

    ![Настройка строки подключения к базе данных](media/app-service-web-tutorial-dotnet-sqldatabase/configure-sql-database-connection.png)

1. Дождитесь завершения работы мастера настройки и нажмите кнопку **Закрыть**.

#### <a name="deploy-your-aspnet-app"></a>Развертывание приложения ASP.NET

1. На вкладке **Публикация** прокрутите вверх и щелкните команду **Опубликовать**. Когда приложение ASP.NET будет развернуто в Azure, откроется браузер по умолчанию с URL-адресом развернутого приложения.

1. Добавьте несколько элементов списка дел.

    ![Опубликованное приложение ASP.NET в приложении Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-app-in-browser.png)

    Поздравляем! Вы запустили управляемое данными приложение ASP.NET в службе приложений Azure.

## <a name="access-the-database-locally"></a>Локальный доступ к базе данных

В **обозревателе объектов SQL Server** Visual Studio вы сможете легко просматривать сведения о своей новой базе данных в Azure и управлять ею. Новая база данных уже открыла свой брандмауэр для созданного приложения Службы приложений, но для доступа к нему с локального компьютера (например, из Visual Studio) необходимо открыть брандмауэр для общедоступного IP-адреса локального компьютера. Если поставщик услуг Интернета изменит ваш общедоступный IP-адрес, необходимо перенастроить брандмауэр для возобновления доступа к базе данных Azure.

#### <a name="create-a-database-connection"></a>Создание подключения к базе данных

1. В меню **Представление** выберите **Обозреватель объектов SQL Server**.

1. В верхней части **обозревателя объектов SQL Server** нажмите кнопку **Добавить SQL Server**.

#### <a name="configure-the-database-connection"></a>Настройка подключения к базе данных

1. В диалоговом окне **Подключение** разверните узел **Azure**. Здесь перечислены все экземпляры базы данных SQL в Azure.

1. Выберите ранее созданную базу данных. В нижней части автоматически появится созданное ранее подключение.

1. Введите созданный ранее пароль администратора базы данных и нажмите кнопку **Подключиться**.

    ![Настройка подключения к базе данных из Visual Studio](./media/app-service-web-tutorial-dotnet-sqldatabase/connect-to-sql-database.png)

#### <a name="allow-client-connection-from-your-computer"></a>Разрешение клиентских подключений с вашего компьютера

Откроется диалоговое окно **Создать новое правило брандмауэра**. По умолчанию сервер разрешает подключаться к его базам данных только из служб Azure, таких как ваше приложение Azure. Чтобы подключаться к базе данных извне Azure, создайте правило брандмауэра на уровне сервера. Это правило разрешает подключения с общедоступного IP-адреса вашего локального компьютера.

В диалоговом окне уже указан общедоступный IP-адрес компьютера.

1. Убедитесь, что флажок **Добавить IP-адрес моего клиента** установлен, и щелкните **ОК**.

    ![Создание правила брандмауэра](./media/app-service-web-tutorial-dotnet-sqldatabase/sql-set-firewall.png)

    После окончания настройки брандмауэра для экземпляра базы данных SQL ваше подключение появится в **обозревателе объектов SQL Server**.

    В нем вы можете выполнять большинство распространенных операций с базой данных: выполнять запросы, создавать представления и хранимые процедуры и многое другое.

1. Разверните узел подключения, выберите **Базы данных** >  **&lt;ваша база данных >**  > **Таблицы**. Щелкните таблицу `Todoes` правой кнопкой мыши и выберите **Просмотреть данные**.

    ![Просмотр объектов базы данных SQL](./media/app-service-web-tutorial-dotnet-sqldatabase/explore-sql-database.png)

## <a name="update-app-with-code-first-migrations"></a>Изменение приложения с помощью Code First Migrations

Обновить базу данных и приложение в Azure можно с помощью знакомых инструментов Visual Studio. На этом шаге вы измените схему базы данных с помощью Code First Migrations в Entity Framework и опубликуете ее в Azure.

Дополнительные сведения об использовании Entity Framework Code First Migrations см. в статье [Getting Started with Entity Framework 6 Code First using MVC 5](/aspnet/mvc/overview/getting-started/getting-started-with-ef-using-mvc/creating-an-entity-framework-data-model-for-an-asp-net-mvc-application) (Начало работы с Entity Framework 6 Code First с помощью MVC 5).

#### <a name="update-your-data-model"></a>Обновление модели данных

Откройте файл _Models\Todo.cs_ в редакторе кода. Добавьте в класс `ToDo` следующее свойство:

```csharp
public bool Done { get; set; }
```
    
#### <a name="run-code-first-migrations-locally"></a>Локальный запуск Code First Migrations

Выполните несколько команд, чтобы обновить локальную базу данных.

1. В меню **Инструменты** выберите **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.

1. В окне консоли диспетчера пакетов включите Code First Migrations:

    ```powershell
    Enable-Migrations
    ```
    
1. Добавьте миграцию:

    ```powershell
    Add-Migration AddProperty
    ```
    
1. Обновите локальную базу данных:

    ```powershell
    Update-Database
    ```
    
1. Введите `Ctrl+F5`, чтобы запустить приложение. Проверьте ссылки "Изменить", "Сведения" и "Создать".

Если приложение загружается без ошибок, Code First Migrations успешно включен. Однако ваша страница не изменилась, так как новое свойство все еще не используется в логике приложения.

#### <a name="use-the-new-property"></a>Использование нового свойства

Внесите некоторые изменения в код, чтобы использовалось свойство `Done`. Для простоты мы изменим только представления `Index` и `Create`, чтобы просмотреть свойство в действии.

1. Откройте файл _Controllers\TodosController.cs_.

1. Найдите метод `Create()` в строке 52 и добавьте `Done` в список свойств атрибута `Bind`. Когда все будет готово, сигнатура метода `Create()` должна выглядеть следующим образом:

    ```csharp
    public ActionResult Create([Bind(Include = "Description,CreatedDate,Done")] Todo todo)
    ```
    
1. Откройте файл _Views\Todos\Create.cshtml_.

1. В коде Razor вы должны увидеть элемент `<div class="form-group">`, который использует `model.Description`, и еще один элемент `<div class="form-group">`, который использует `model.CreatedDate`. Сразу после этих двух элементов добавьте еще один элемент `<div class="form-group">`, который использует `model.Done`:

    ```csharp
    <div class="form-group">
        @Html.LabelFor(model => model.Done, htmlAttributes: new { @class = "control-label col-md-2" })
        <div class="col-md-10">
            <div class="checkbox">
                @Html.EditorFor(model => model.Done)
                @Html.ValidationMessageFor(model => model.Done, "", new { @class = "text-danger" })
            </div>
        </div>
    </div>
    ```
    
1. Откройте файл _Views\Todos\Index.cshtml_.

1. Найдите пустой элемент `<th></th>`. Добавьте следующий код Razor над этим элементом:

    ```csharp
    <th>
        @Html.DisplayNameFor(model => model.Done)
    </th>
    ```
    
1. Найдите элемент `<td>`, который содержит вспомогательные методы `Html.ActionLink()`. _Над_ элементом `<td>` добавьте еще один элемент `<td>` со следующим кодом Razor:

    ```csharp
    <td>
        @Html.DisplayFor(modelItem => item.Done)
    </td>
    ```
    
    Это все, что нужно сделать, чтобы увидеть изменения в представлениях `Index` и `Create`.

1. Введите `Ctrl+F5`, чтобы запустить приложение.

Теперь вы сможете добавить элемент списка дел и установить флажок **Готово**. После этого задание должно появиться на главной странице как выполненное. Помните, что в представлении `Edit` не отображается поле `Done`, так как вы не изменили представление `Edit`.

#### <a name="enable-code-first-migrations-in-azure"></a>Включение Code First Migrations в Azure

Теперь, когда изменения в коде, включая миграцию базы данных, выполнены успешно, можно опубликовать изменения в приложение Azure, а также обновить Базу данных SQL для использования Code First Migrations.

1. Как и прежде, щелкните правой кнопкой мыши свой проект и выберите **Опубликовать**.

1. Щелкните элементы **Дополнительные действия** > **Изменить**, чтобы открыть раздел параметров публикации.

    ![Открытие параметров публикации](./media/app-service-web-tutorial-dotnet-sqldatabase/publish-settings.png)

1. В раскрывающемся списке **MyDatabaseContext** выберите подключение к базе данных для Базы данных SQL Azure.

1. Установите флажок **Выполнять Code First Migrations (при запуске приложения)** и нажмите кнопку **Сохранить**.

    ![Включение Code First Migrations в приложении Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/enable-migrations.png)

#### <a name="publish-your-changes"></a>Публикация изменений

Включив Code First Migrations в приложении Azure, опубликуйте изменения кода.

1. На странице публикации щелкните **Опубликовать**.

1. Попробуйте добавить новые задачи, устанавливая флажок **Готово**. Эти задачи должны появляться на главной странице как выполненные.

    ![Приложение Azure после включения Code First Migration](./media/app-service-web-tutorial-dotnet-sqldatabase/this-one-is-done.png)

Все имеющиеся элементы списка дел по-прежнему отображаются. При повторной публикации приложения ASP.NET существующие данные в базе данных SQL не теряются. Кроме того, Code First Migrations изменяет только схему данных, оставляя существующие данные нетронутыми.

## <a name="stream-application-logs"></a>Потоковая передача журналов приложений

Сообщения трассировки можно передавать прямо из приложения Azure в Visual Studio.

Откройте файл _Controllers\TodosController.cs_.

Каждое действие начинается с метода `Trace.WriteLine()`. Этот код добавлен, чтобы показать, как добавить сообщения трассировки в приложение Azure.

#### <a name="enable-log-streaming"></a>Включение потоковой передачи журналов

1. В меню **Представление** выберите **Cloud Explorer**.

1. В **Cloud Explorer** разверните подписку Azure, в которой находится ваше приложение, и разверните узел **Служба приложений**.

1. Щелкните приложение правой кнопкой мыши и выберите **Просмотреть журналы потоковой передачи**.

    ![Включение потоковой передачи журналов](./media/app-service-web-tutorial-dotnet-sqldatabase/stream-logs.png)

    Теперь журналы передаются в окно **Выходные данные**.

    ![Потоковая передача журналов в окне "Выходные данные"](./media/app-service-web-tutorial-dotnet-sqldatabase/log-streaming-pane.png)

    Однако сообщения трассировки пока не отображаются. Это объясняется тем, что когда вы в первый раз выбираете пункт меню **Просмотреть журналы потоковой передачи**, приложение устанавливает уровень трассировки в `Error`, при котором в журнал записываются только события ошибок (с помощью метода `Trace.TraceError()`).

#### <a name="change-trace-levels"></a>Изменение уровней трассировки

1. Чтобы изменить уровень трассировки для вывода других сообщений трассировки, вернитесь в **Cloud Explorer**.

1. Снова щелкните приложение правой кнопкой мыши и выберите команду **Открыть на портале**.

1. На странице портала для управления приложением в меню слева выберите элемент **Журналы Службы приложений**.

1. В разделе **Ведение журнала приложений (файловая система)** выберите элемент **Подробно** в области **Уровень**. Выберите команду **Сохранить**.

    ![Измените уровень трассировки на "Подробно"](./media/app-service-web-tutorial-dotnet-sqldatabase/trace-level-verbose.png)

    > [!TIP]
    > Вы можете поэкспериментировать с различными уровнями трассировки, чтобы посмотреть, какие типы сообщений отображаются для каждого уровня. Например, уровень **Информация** включает все журналы, созданные `Trace.TraceInformation()`, `Trace.TraceWarning()`, и `Trace.TraceError()`, но не включает журналы, созданные `Trace.WriteLine()`.

1. В браузере снова перейдите к приложению по адресу *http://&lt;имя вашего приложения>.azurewebsites.net*, а затем щелкните правой кнопкой приложение списка дел в Azure. Теперь сообщения трассировки передаются в окно **Выходные данные** в Visual Studio.

    ```console
    Application: 2017-04-06T23:30:41  PID[8132] Verbose     GET /Todos/Index
    Application: 2017-04-06T23:30:43  PID[8132] Verbose     GET /Todos/Create
    Application: 2017-04-06T23:30:53  PID[8132] Verbose     POST /Todos/Create
    Application: 2017-04-06T23:30:54  PID[8132] Verbose     GET /Todos/Index
    ```
    
#### <a name="stop-log-streaming"></a>Выключение потоковой передачи журналов

Чтобы остановить службу потоковой передачи журналов, нажмите кнопку **Остановить наблюдение** в окне **Выходные данные**.

![Выключение потоковой передачи журналов](./media/app-service-web-tutorial-dotnet-sqldatabase/stop-streaming.png)

## <a name="manage-your-azure-app"></a>Управление приложением Azure

Перейдите на [портал Azure](https://portal.azure.com) для управления веб-приложением. Найдите в поиске и выберите **Службы приложений**.

![Поиск Служб приложений Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/azure-portal-navigate-app-services.png)

Выберите имя приложения Azure.

![Переход к приложению Azure на портале](./media/app-service-web-tutorial-dotnet-sqldatabase/access-portal.png)

Вы попадете на страницу приложения.

По умолчанию на портале отображается страница **Обзор**. Здесь вы можете наблюдать за работой приложения. Вы также можете выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление. На вкладках в левой части страницы отображаются различные страницы конфигурации, которые можно открыть.

![Страница службы приложений на портале Azure](./media/app-service-web-tutorial-dotnet-sqldatabase/web-app-blade.png)

[!INCLUDE [Clean up section](../../includes/clean-up-section-portal-web-app.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * Создание базы данных в службе "База данных SQL Azure"
> * Подключение приложения ASP.NET к базе данных SQL.
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов из Azure в окно терминала.
> * Управление приложением на портале Azure.

Переходите к следующему руководству, чтобы узнать, как без усилий обеспечить защиту подключения к базе данных SQL Azure.

> [!div class="nextstepaction"]
> [Безопасный доступ к Базе данных SQL с использованием управляемого удостоверения для ресурсов Azure](app-service-web-tutorial-connect-msi.md)

Дополнительные ресурсы

> [!div class="nextstepaction"]
> [Настройка приложения ASP.NET](configure-language-dotnet-framework.md)

Хотите оптимизировать и сократить ваши расходы на облако?

> [!div class="nextstepaction"]
> [Начните анализировать затраты с помощью службы "Управление затратами"](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)