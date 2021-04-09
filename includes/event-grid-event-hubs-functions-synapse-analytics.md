---
title: включить файл
description: включить файл
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: include
ms.date: 12/07/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: d6b8b35b36830568d6ff8c46a381fec0cfc57e84
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96854266"
---
:::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/overview.png" alt-text="Обзор приложений":::

Эта схема демонстрирует рабочий процесс решения, создаваемого в рамках этого руководства. 

1. Данные, отправляемые в концентратор событий Azure, записываются в хранилище BLOB-объектов Azure.
2. После сбора данных создается событие и отправляется в Сетку событий Azure. 
3. Сетка событий переадресовывает данные этого события в приложение-функцию Azure.
4. Приложение-функция использует URL-адрес большого двоичного объекта из данных события, чтобы извлечь большой двоичный объект из хранилища. 
5. Приложение-функция переносит данные больших двоичных объектов в Azure Synapse Analytics. 

Ниже перечислены действия, которые вы выполните в этой статье.

> [!div class="checklist"]
> - Развертывание необходимой инфраструктуры для работы с руководством.
> - Публикация кода в приложение-функцию.
> - Создание подписки Сетки событий 
> - Потоковая передача примера данных в Центры событий.
> - Проверка собранных данных в Azure Synapse Analytics

## <a name="prerequisites"></a>Предварительные требования
Для работы с этим руководством требуется:

- Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
- [Visual Studio 2019](https://www.visualstudio.com/vs/) с рабочими нагрузками для разработки классических приложений .NET, Azure, ASP.NET, Node.js, Python и веб-разработки.
- Скачать на свой компьютер [пример проекта EventHubsCaptureEventGridDemo](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo).
    - WindTurbineDataGenerator — простой издатель, который отправляет пример данных ветровой турбины в концентратор событий с поддержкой сбора данных.
    - FunctionDWDumper — функция Azure, которая получает уведомление Сетки событий, когда файл AVRO записывается в большой двоичный объект в службе хранилища Azure. Она получает путь URI для большого двоичного объекта, считывает его содержимое и помещает эти данные в Azure Synapse Analytics (выделенный пул SQL).

## <a name="deploy-the-infrastructure"></a>Развертывание инфраструктуры
На этом шаге вы развернете необходимую инфраструктуру с помощью [шаблона Resource Manager](https://github.com/Azure/azure-docs-json-samples/blob/master/event-grid/EventHubsDataMigration.json). При развертывании шаблона создаются следующие ресурсы.

* Концентратор событий со включенной функцией "Сбор".
* Учетная запись хранения для собранных файлов. 
* План службы приложений для размещения приложения-функции.
* Приложение-функция для обработки события.
* SQL Server для размещения хранилища данных.
* Azure Synapse Analytics (выделенный пул SQL) для хранения перенесенных данных.

### <a name="use-azure-cli-to-deploy-the-infrastructure"></a>Использование Azure CLI для развертывания инфраструктуры

1. Войдите на [портал Azure](https://portal.azure.com). 
2. Нажмите кнопку **Cloud Shell**, которая находится вверху.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/azure-portal.png" alt-text="Портал Azure":::
3. В нижней части окна веб-браузера вы увидите открытый Cloud Shell.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/launch-cloud-shell.png" alt-text="Cloud Shell":::
4. Если в Cloud Shell у вас будет выбор между **Bash** и **PowerShell**, выберите **Bash**. 
5. Если вы используете Cloud Shell в первый раз, создайте учетную запись хранилища, выбрав **Create storage** (Создать хранилище). Azure Cloud Shell требуется учетная запись хранилища Azure для хранения некоторых файлов. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/create-storage-cloud-shell.png" alt-text="Создание хранилища для Cloud Shell":::
6. Подождите, пока не инициализируется Cloud Shell. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/cloud-shell-initialized.png" alt-text="Cloud Shell после инициализации":::
1. Создайте группу ресурсов Azure, выполнив следующую команду в окне CLI: 
    1. Скопируйте следующую команду и вставьте ее в окно Cloud Shell. При желании измените имя группы ресурсов и расположение.

        ```azurecli
        az group create -l eastus -n rgDataMigration
        ```
    2. Нажмите клавишу **ВВОД**. 

        Например:
    
        ```azurecli
        user@Azure:~$ az group create -l eastus -n rgDataMigration
        {
          "id": "/subscriptions/00000000-0000-0000-0000-0000000000000/resourceGroups/rgDataMigration",
          "location": "eastus",
          "managedBy": null,
          "name": "rgDataMigration",
          "properties": {
            "provisioningState": "Succeeded"
          },
          "tags": null
        }
        ```
2. Разверните все ресурсы, упомянутые в предыдущем разделе (концентратор событий, учетная запись хранения, приложение-функция, Azure Synapse Analytics), выполнив следующую команду интерфейса командной строки: 
    1. Скопируйте эту команду и вставьте ее в окно Cloud Shell. Или же вы можете скопировать и вставить команду в свой любимый редактор, задать значения, а затем скопировать ее в Cloud Shell. 

        > [!IMPORTANT]
        > Перед выполнением этой команды укажите значения для следующих сущностей: 
        > - Имя группы ресурсов, созданной ранее.
        > - Название пространства имен концентратора событий. 
        > - Имя концентратора событий. Вы можете оставить имеющееся значение (hubdatamigration).
        > - Имя для SQL Server.
        > - Имя и пароль пользователя SQL. 
        > - Имя базы данных.
        > - Имя учетной записи хранения. 
        > - Имя приложения-функции. 


        ```azurecli
        az deployment group create \
            --resource-group rgDataMigration \
            --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json \
            --parameters eventHubNamespaceName=<event-hub-namespace> eventHubName=hubdatamigration sqlServerName=<sql-server-name> sqlServerUserName=<user-name> sqlServerPassword=<password> sqlServerDatabaseName=<database-name> storageName=<unique-storage-name> functionAppName=<app-name>
        ```
    3.  Нажмите клавишу **ВВОД** в окне Cloud Shell, чтобы запустить команду. Этот процесс может занять некоторое время, так как вы создаете не один ресурс. После выполнения команды убедитесь, что сбоев не было. 
1. Закройте Cloud Shell, нажав кнопку **Cloud Shell** на портале или кнопку **X** в правом верхнем углу окна Cloud Shell. 

### <a name="verify-that-the-resources-are-created"></a>Проверка создания ресурсов

1. На портале Azure в меню слева выберите **Группы ресурсов**. 
2. Отфильтруйте список групп ресурсов, введя имя нужной группы ресурсов в поле поиска. 
3. Выберите в списке свою группу ресурсов.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/select-resource-group.png" alt-text="Выберите группу ресурсов":::
4. Убедитесь, что в группе ресурсов находятся следующие ресурсы:

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/resources-in-resource-group.png" alt-text="Ресурсы в группе ресурсов" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/resources-in-resource-group.png":::

### <a name="create-a-table-in-azure-synapse-analytics"></a>Создание таблицы в Azure Synapse Analytics
Создайте таблицу в хранилище данных, выполнив сценарий [CreateDataWarehouseTable.sql](https://github.com/Azure/azure-event-hubs/blob/master/samples/e2e/EventHubsCaptureEventGridDemo/scripts/CreateDataWarehouseTable.sql). Чтобы запустить скрипт, воспользуйтесь Visual Studio или редактором запросов на портале. В следующих шагах продемонстрировано, как использовать редактор запросов. 

1. В списке ресурсов в группе ресурсов выберите свой **выделенный пул SQL**. 
2. На странице **Выделенный пул SQL** в разделе **Типичные задачи** в меню слева выберите **Редактор запросов (предварительная версия)** . 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/sql-data-warehouse-page.png" alt-text="Страница Azure Synapse Analytics":::
2. Введите имя **пользователя** и **пароль** для SQL Server и щелкните **ОК**. Если появится сообщение о разрешении клиенту доступа к серверу SQL, выполните следующие действия:
    1. Перейдите по ссылке: **Настройка брандмауэра для сервера.** 
    2. На странице **Параметры брандмауэра** щелкните на панели инструментов **Добавить IP-адрес клиента**, а затем щелкните **Сохранить** на панели инструментов. 
    3. Нажмите кнопку **ОК** в окне сообщения об успешном выполнении.
    4. Вернитесь на страницу **Выделенный пул SQL** и выберите в меню слева пункт **Редактор запросов (предварительная версия)** . 
    5. Введите **имя пользователя** и **пароль**, а затем нажмите кнопку **ОК**. 
1. Скопируйте и вставьте следующий код в окно запроса: 

    ```sql
    CREATE TABLE [dbo].[Fact_WindTurbineMetrics] (
        [DeviceId] nvarchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
        [MeasureTime] datetime NULL, 
        [GeneratedPower] float NULL, 
        [WindSpeed] float NULL, 
        [TurbineSpeed] float NULL
    )
    WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
    ```

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/run-sql-query.png" alt-text="Выполнение SQL-запроса":::
5. Не закрывайте эту вкладку или окно, так как в конце руководства это поможет вам проверить успешность создания данных. 

### <a name="update-the-function-runtime-version"></a>Обновление версии среды выполнения функции

1. В веб-браузере откройте новую вкладку и откройте на ней [портал Azure](https://portal.azure.com).
1. На портале Azure в меню слева выберите **Группы ресурсов**.
1. Выберите группу ресурсов, в которой существует приложение-функция. 
1. Выберите **приложение-функцию** в списке ресурсов для группы ресурсов.
1. Выберите **Конфигурация** в меню **Параметры** слева. 
1. Перейдите на вкладку **Параметры среды выполнения функций** в правой области. 
1. Обновите **версию среды выполнения** до **~3**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/function-runtime-version.png" alt-text="Обновление версии среды выполнения функции":::
6. На панели инструментов щелкните **Сохранить**. 
1. Во всплывающем окне с подтверждением для **сохранения изменений** щелкните **Продолжить**. 

## <a name="publish-the-azure-functions-app"></a>Публикация приложения службы "Функции Azure"

1. Запустите Visual Studio.
2. Откройте решение **EventHubsCaptureEventGridDemo.sln**, скачанное с портала [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo) во время выполнения предыдущих шагов. Его можно найти в папке `/samples/e2e/EventHubsCaptureEventGridDemo`. 
3. В обозревателе решений щелкните правой кнопкой мыши проект **FunctionEGDWDumper** и выберите **Публиковать**.
4. Если вы видите такой экран, щелкните **Начало**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/start-publish-button.png" alt-text="Кнопка &quot;Запуск&quot; в разделе &quot;Публикация&quot;.":::
5. В диалоговом окне **Публикация** выберите **Azure** в качестве параметра **Целевой объект** и нажмите **Далее**. 
6. Выберите **Приложение-функция Azure (Windows)** и нажмите **Далее**.
7. На вкладке **Экземпляр функций** выберите свою подписку Azure, разверните группу ресурсов и выберите приложение-функцию, а затем нажмите **Готово**. Войдите в свою учетную запись Azure, если вы еще этого не сделали. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/publish-select-function-app.png" alt-text="Выбор приложения-функции":::
8. На странице **Публикация** в разделе **Зависимости службы** выберите **Настройка** для элемента **Хранилище**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/publish-storage-configure-link.png" alt-text="Щелкните ссылку настройки для зависимости службы хранилища.":::
1. На странице **Настройка зависимостей** выполните следующие действия: 
    1. Выберите **учетную запись хранения**, которую вы создали ранее, затем щелкните **Далее**. 

        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/select-dependency-storage.png" alt-text="Выбор учетной записи хранения":::
    10. Укажите **имя для строки подключения** и выберите **Нет** для параметра **Сохранить строку подключения**, а затем щелкните **Далее**. 
    
        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/dependency-storage-connection-string.png" alt-text="Ввод имени строки подключения":::      
    1. Снимите флажки **Файл кода C#** и **Хранилище секретов**, а затем щелкните **Готово**.  
    
        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/dependency-storage-changes-summary.png" alt-text="Просмотр сводки изменений":::
1. После настройки профиля в Visual Studio выберите **Публиковать**.

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/select-publish.png" alt-text="Выбор публикации":::
2. На вкладке, где открыта страница **Функции Azure**, выберите **Функции** в меню слева. Убедитесь, что в списке отображается функция **EventGridTriggerMigrateData**. Если вы не видите ее, попробуйте повторить публикацию из Visual Studio и обновите страницу на портале. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/confirm-function-creation.png" alt-text="Подтверждение создания функции":::    

После публикации функции вы можете подписаться на событие.

## <a name="subscribe-to-the-event"></a>Оформление подписки на событие.

1. В новой вкладке или новом окне веб-браузера откройте [портал Azure](https://portal.azure.com).
2. На портале Azure в меню слева выберите **Группы ресурсов**. 
3. Отфильтруйте список групп ресурсов, введя имя нужной группы ресурсов в поле поиска. 
4. Выберите в списке свою группу ресурсов.
1. Выберите **Пространство имен Центров событий** из списка ресурсов.
1. На странице **Пространство имен Центров событий** выберите **События** в меню слева, а затем на панели инструментов выберите **+ Подписка на события**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/event-hub-add-subscription-link.png" alt-text="Ссылка для добавления подписки на событие на странице &quot;События&quot; для пространства имен Центров событий":::
1. На странице **Создать подписку на событие** выполните следующие действия:
    1. Укажите имя для **подписки на событие**. 
    1. Введите имя **системного раздела**. Системный раздел предоставляет конечную точку, через которую отправитель может отправлять события. Дополнительные сведения см. в описании [системных разделов](../articles/event-grid/system-topics.md).
    1. В поле **Тип конечной точки** выберите **Функция Azure**.
    1. В разделе **Конечная точка** щелкните ссылку.
    1. На странице **Выберите функцию Azure** выполните следующие действия, если поля не заполнены автоматически:
        1. Выберите подписку Azure, в которой размещена функция Azure. 
        1. Выберите группу ресурсов для этой функции. 
        1. Выберите приложение-функцию.
        1. Выберите слот развертывания. 
        1. Выберите функцию **EventGridTriggerMigrateData**. 
    1. На странице **Выберите функцию Azure** щелкните **Подтвердить выбор**.
    1. Теперь вернитесь на страницу **Создать подписку на события** и щелкните **Создать**. 
    
        :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/event-subscription-select-function.png" alt-text="Создание подписки на событие с помощью функции" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/event-subscription-select-function.png":::
1. Убедитесь, что подписка на событие успешно создана. Перейдите на вкладку **Подписки на события** на странице **События** для пространства имен Центров событий. 
    
    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/confirm-event-subscription.png" alt-text="Подтверждение подписки на событие" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/confirm-event-subscription.png":::
1. Выберите "План службы приложений" (не "Служба приложений") в списке ресурсов в группе ресурсов. 

## <a name="run-the-app-to-generate-data"></a>Запуск приложения для создания данных
Итак, вы завершили настройку концентратора событий, выделенного пула SQL (прежнее название — Хранилище данных SQL), приложения-функции Azure и подписки на события. Прежде чем запустить приложение, создающее данные для концентратора событий, настройте несколько значений.

1. На портале Azure, как и раньше, перейдите к группе ресурсов. 
2. Выберите пространство имен Центров событий.
3. На странице **пространства имен Центров событий** выберите **Политики общего доступа** в меню слева.
4. В списке политик выберите **RootManageSharedAccessKey**. 

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/event-hub-namespace-shared-access-policies.png" alt-text="Страница политик общего доступа для пространства имен Центров событий":::    
1. Щелкните синюю кнопку копирования справа от текстового поля **Connection string-primary key** (Первичный ключ строки подключения). 
1. Вернитесь к решению Visual Studio. 
1. Щелкните правой кнопкой мыши проект **WindTurbineDataGenerator** и выберите **Назначить запускаемым проектом**. 
1. В проекте WindTurbineDataGenerator откройте файл **program.cs**.
1. Замените `<EVENT HUBS NAMESPACE CONNECTION STRING>` строкой подключения, которую вы скопировали на портале. 
1. Замените `<EVENT HUB NAME>` именем концентратора событий. 

   ```cs
   private const string EventHubConnectionString = "Endpoint=sb://demomigrationnamespace.servicebus.windows.net/...";
   private const string EventHubName = "hubdatamigration";
   ```
6. Создайте решение. Запустите приложение **WindTurbineGenerator.exe**. 
7. Через пару минут на другой вкладке браузера, где открыто окно запросов, выполните запрос по перенесенным данным в таблице хранилища данных.

    ```sql
    select * from [dbo].[Fact_WindTurbineMetrics]    
    ```

    ![Результаты запроса](media/event-grid-event-hubs-functions-synapse-analytics/query-results.png)

## <a name="monitor-the-solution"></a>Мониторинг решения
Этот раздел поможет вам выполнять мониторинг и устранение неполадок для решения. 

### <a name="view-captured-data-in-the-storage-account"></a>Просмотр собранных данных в учетной записи хранения
1. Перейдите к группе ресурсов и выберите учетную запись хранения, которая используется для сбора данных о событиях. 
1. На странице **Учетная запись хранения** выберите **Обозреватель службы хранилища (предварительная версия)** в меню слева.
1. Разверните раздел **Контейнеры BLOB-объектов** и выберите **windturbinecapture**. 
1. Откройте в области справа папку с таким же именем, как у **пространства имен Центров событий**. 
1. Откройте папку с таким же именем, как у концентратора событий (**hubdatamigration**). 
1. Просмотрите вложенные папки и найдите файлы AVRO. Ниже приведен пример:

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/storage-captured-file.png" alt-text="Файл собранных данных в хранилище" lightbox="media/event-grid-event-hubs-functions-synapse-analytics/storage-captured-file.png":::
    

### <a name="verify-that-the-event-grid-trigger-invoked-the-function"></a>Проверка того, что триггер Сетки событий вызвал функцию
1. Перейдите к используемой группе ресурсов и выберите приложение-функцию. 
1. Выберите **Функции** в меню слева.
1. В списке выберите функцию **EventGridTriggerMigrateData**. 
1. На странице **Функция** выберите в меню слева пункт **Монитор**. 
1. Выберите **Настроить**, чтобы настроить сбор журналов вызовов в Application Insights. 
1. Создайте новый ресурс **Application Insights** или выберите существующий. 
1. Вернитесь на страницу **Монитор** для этой функции. 
1. Убедитесь, что клиентское приложение (**WindTurbineDataGenerator**), отправляющее события, по-прежнему выполняется. Если это не так, запустите это приложение. 
1. Подождите несколько минут (5 и более), а затем нажмите кнопку **Обновить**, чтобы просмотреть вызовы функций.    

    :::image type="content" source="media/event-grid-event-hubs-functions-synapse-analytics/function-invocations.png" alt-text="Вызовы функций":::
1. Выберите конкретный вызов, чтобы просмотреть сведения о нем.

    Служба "Сетка событий" распределяет данные события между подписчиками. В следующем примере показаны данные событий, сгенерированные при записи потока данных, проходящих через концентратор событий, в большой двоичный объект. В частности, обратите внимание, что свойство `fileUrl` в объекте `data` указывает на большой двоичный объект в хранилище. Приложение-функция использует этот URL-адрес для получения файла большого двоичного объекта с записанными данными.

    ```json
    {
        "topic": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourcegroups/rgDataMigration/providers/Microsoft.EventHub/namespaces/spehubns1207",
        "subject": "hubdatamigration",
        "eventType": "Microsoft.EventHub.CaptureFileCreated",
        "id": "4538f1a5-02d8-4b40-9f20-36301ac976ba",
        "data": {
            "fileUrl": "https://spehubstorage1207.blob.core.windows.net/windturbinecapture/spehubns1207/hubdatamigration/0/2020/12/07/21/49/12.avro",
            "fileType": "AzureBlockBlob",
            "partitionId": "0",
            "sizeInBytes": 473444,
            "eventCount": 2800,
            "firstSequenceNumber": 55500,
            "lastSequenceNumber": 58299,
            "firstEnqueueTime": "2020-12-07T21:49:12.556Z",
            "lastEnqueueTime": "2020-12-07T21:50:11.534Z"
        },
        "dataVersion": "1",
        "metadataVersion": "1",
        "eventTime": "2020-12-07T21:50:12.7065524Z"
    }
    ```

### <a name="verify-that-the-data-is-stored-in-the-dedicated-sql-pool"></a>Проверка того, что данные успешно сохранены в выделенном пуле SQL
На вкладке браузера, где открыто окно запросов, выполните запрос по перенесенным данным в таблице выделенного пула SQL.

![Результаты запроса](media/event-grid-event-hubs-functions-synapse-analytics/query-results.png)

