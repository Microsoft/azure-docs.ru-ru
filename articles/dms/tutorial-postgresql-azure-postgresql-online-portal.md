---
title: Руководство по миграции из PostgreSQL в Базу данных Azure для PostgreSQL по сети с помощью портала Azure
titleSuffix: Azure Database Migration Service
description: Узнайте, как выполнить миграцию по сети из локальной среды PostgreSQL в Базу данных Azure для PostgreSQL с помощью Azure Database Migration Service через портал Azure.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 04/11/2020
ms.openlocfilehash: d28c45b2d0fc1a123f44020f42c4d2c59c593cb2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101709926"
---
# <a name="tutorial-migrate-postgresql-to-azure-db-for-postgresql-online-using-dms-via-the-azure-portal"></a>Руководство по миграции из PostgreSQL в Базу данных Azure для PostgreSQL по сети с помощью DMS через портал Azure

Azure Database Migration Service Azure Database Migration Service позволяет переносить базы данных из локального экземпляра PostgreSQL в [Базу данных Azure для PostgreSQL](../postgresql/index.yml) с минимальным простоем для приложения. В этом руководстве выполняется миграция примера базы данных **Прокат DVD** из локального экземпляра PostgreSQL 9.6 в Базу данных Azure для PostgreSQL с помощью действия сетевой миграции в Azure Database Migration Service.

В этом руководстве вы узнаете, как:
> [!div class="checklist"]
>
> * перенести пример схемы с помощью служебной программы pg_dump;
> * Создайте экземпляр Azure Database Migration Service.
> * создать проект миграции в Azure Database Migration Service;
> * выполнение миграции.
> * Мониторинг миграции.
> * выполнить переключение после миграции.

> [!NOTE]
> Чтобы выполнить сетевую миграцию с помощью Azure Database Migration Service, требуется создать экземпляр ценовой категории "Премиум". Мы шифруем диск для защиты данных от кражи при миграции.

> [!IMPORTANT]
> Чтобы процесс миграции был выполнен без проблем, Майкрософт рекомендует создать экземпляр Azure Database Migration Service в том же регионе Azure, в котором размещена целевая база данных. Перемещение данных между регионами и географическими областями может замедлить процесс миграции и привести к ошибкам.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

* Скачайте и установите [PostgreSQL Community Edition](https://www.postgresql.org/download/) 9.4, 9.5, 9.6 или 10. На исходном сервере должна быть установлена PostgreSQL версии 9.4, 9.5, 9.6, 10 или 11. Дополнительные сведения см. в статье [Поддерживаемые версии базы данных PostgreSQL](../postgresql/concepts-supported-versions.md).

    Также обратите внимание, что версия целевой Базы данных Azure для PostgreSQL не может быть более ранней, чем версия исходного экземпляра. Например, PostgreSQL 9.6 можно перенести в Базу данных Azure для PostgreSQL 9.6, 10 или 11, но не в Базу данных Azure для PostgreSQL 9.5.

* Создайте [сервер Базы данных Azure для PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) или [сервер базы данных Azure для PostgreSQL (Гипермасштабирование (Citus))](../postgresql/quickstart-create-hyperscale-portal.md).
* Создайте виртуальную сеть Microsoft Azure для Azure Database Migration Service с помощью модели развертывания Azure Resource Manager. Она обеспечивает подключение "сеть — сеть" к локальным исходным серверам через [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Дополнительные сведения о создании виртуальной сети приведены в [документации по виртуальным сетям](../virtual-network/index.yml). В частности, уделите внимание кратким руководствам с пошаговыми инструкциями.

    > [!NOTE]
    > Если вы используете ExpressRoute с пиринговым подключением к сети, управляемой Майкрософт, во время настройки виртуальной сети добавьте в подсеть, в которой будет подготовлена служба, следующие [конечные точки](../virtual-network/virtual-network-service-endpoints-overview.md):
    >
    > * целевую конечную точку базы данных (например, конечная точка SQL, конечная точка Cosmos DB и т. д.);
    > * конечную точку службы хранилища;
    > * конечную точку служебной шины.
    >
    > Такая конфигурация вызвана тем, что у Azure Database Migration Service нет подключения к Интернету.

* Убедитесь, что правила группы безопасности сети для виртуальной сети не блокируют исходящий порт 443 ServiceTag для Служебной шины, службы хранилища и Azure Monitor. См. дополнительные сведения о [фильтрации трафика, предназначенного для виртуальной сети, с помощью групп безопасности сети](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Настройте [брандмауэр Windows для доступа к ядру СУБД](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Откройте брандмауэр Windows, чтобы предоставить Azure Database Migration Service доступ к исходному серверу PostgreSQL Server. По умолчанию это TCP-порт 5432.
* Если перед исходными базами данных развернуто устройство брандмауэра, вам может понадобиться добавить правила брандмауэра, чтобы позволить службе Azure Database Migration Service обращаться к исходным базам данных для выполнения миграции.
* Создайте [правило брандмауэра](../postgresql/concepts-firewall-rules.md) уровня сервера для Базы данных Azure для PostgreSQL, чтобы предоставить службе Azure Database Migration Service доступ к целевым базам данных. Задайте диапазон подсети в виртуальной сети, которая используется для Azure Database Migration Service.
* Чтобы включить логическую репликацию в файле postgresql.config, задайте параметры, приведенные ниже.

  * wal_level = **logical**
  * max_replication_slots = [количество слотов], рекомендуемое значение — до **5 слотов**
  * max_wal_senders = [количество параллельных задач]. Параметр max_wal_senders задает число параллельных задач, которые можно выполнить: рекомендуемый параметр — до **10 задач**

> [!IMPORTANT]
> Чтобы поддерживать синхронизацию в целевую базу данных, все таблицы в существующей базе данных должны иметь первичный ключ.

## <a name="migrate-the-sample-schema"></a>Перенос примера схемы

Чтобы подготовить все объекты базы данных, такие как схемы таблицы, индексы и хранимые процедуры, нам нужно извлечь схему из исходной базы данных и применить ее к нужной базе данных.

1. Используйте команду pg_dump -s, чтобы создать схемы файла дампа для базы данных.

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Например, для создания файла дампа схемы для базы данных **dvdrental** выполните команду:

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s -O -x > dvdrentalSchema.sql
    ```

    Дополнительные сведения об использовании программы pg_dump см. в примерах руководства о [pg-dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html#PG-DUMP-EXAMPLES).

2. Создайте пустую базу данных в целевой среде, которая является Базой данных Azure для PostgreSQL.

    Дополнительные сведения о подключении к базе данных и о создании базы данных см. в кратких руководствах [Создание сервера Базы данных Azure для PostgreSQL с помощью портала Azure](../postgresql/quickstart-create-server-database-portal.md) или [Создание серверной группы Гипермасштабирования (Citus) на портале Azure](../postgresql/quickstart-create-hyperscale-portal.md).

    > [!NOTE]
    > Экземпляр Базы данных Azure для PostgreSQL (Гипермасштабирование (Citus)) содержит только одну базу данных с именем **citus**.

3. Импортируйте схемы в целевую базу данных, созданную путем восстановления схемы файла дампа.

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    Пример:

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental citus < dvdrentalSchema.sql
    ```

4. Выполните следующий сценарий в PgAdmin или psql, чтобы извлечь внешний ключ сценария удаления и добавить его в целевом экземпляре (База данных Azure для PostgreSQL).

   > [!IMPORTANT]
   > Внешние ключи в схеме приводят к сбою начальной загрузки и непрерывной синхронизации в процессе миграции.

    ```
    SELECT Q.table_name
        ,CONCAT('ALTER TABLE ', table_schema, '.', table_name, STRING_AGG(DISTINCT CONCAT(' DROP CONSTRAINT ', foreignkey), ','), ';') as DropQuery
            ,CONCAT('ALTER TABLE ', table_schema, '.', table_name, STRING_AGG(DISTINCT CONCAT(' ADD CONSTRAINT ', foreignkey, ' FOREIGN KEY (', column_name, ')', ' REFERENCES ', foreign_table_schema, '.', foreign_table_name, '(', foreign_column_name, ')' ), ','), ';') as AddQuery
    FROM
        (SELECT
        S.table_schema,
        S.foreignkey,
        S.table_name,
        STRING_AGG(DISTINCT S.column_name, ',') AS column_name,
        S.foreign_table_schema,
        S.foreign_table_name,
        STRING_AGG(DISTINCT S.foreign_column_name, ',') AS foreign_column_name
    FROM
        (SELECT DISTINCT
        tc.table_schema,
        tc.constraint_name AS foreignkey,
        tc.table_name,
        kcu.column_name,
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name
        FROM information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu ON tc.constraint_name = kcu.constraint_name AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu ON ccu.constraint_name = tc.constraint_name AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY'
        ) S
        GROUP BY S.table_schema, S.foreignkey, S.table_name, S.foreign_table_schema, S.foreign_table_name
        ) Q
        GROUP BY Q.table_schema, Q.table_name;
    ```

5. Запустите сценарий удаления внешнего ключа (второй столбец) в результате запроса.

6. Чтобы отключить триггеры в целевой базе данных, выполните приведенный ниже скрипт.

   > [!IMPORTANT]
   > Триггер в данных (вставки или обновления) поддерживает целостность данных в целевом объекте перед репликацией данных из источника. Поэтому мы рекомендуем отключить триггеры во всех таблицах **в целевом объекте** на период миграции, а затем снова включить триггеры по ее завершении.

    ```
    SELECT DISTINCT CONCAT('ALTER TABLE ', event_object_schema, '.', event_object_table, ' DISABLE TRIGGER ', trigger_name, ';')
    FROM information_schema.triggers
    ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Регистрация поставщика ресурсов Microsoft.DataMigration

1. Войдите на портал Azure, щелкните **Все службы** и выберите **Подписки**.

   ![Отображение подписок на портале](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-select-subscriptions.png)

2. Выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Отображение поставщиков ресурсов](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-select-resource-provider.png)

3. В поле поиска введите migration, а затем справа от **Microsoft.DataMigration** щелкните **Зарегистрировать**.

    ![Регистрация поставщика ресурсов](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Создание экземпляра DMS

1. На портале Azure выберите **+Создать ресурс**, введите в поле поиска "Azure Database Migration Service", а затем в раскрывающемся списке выберите **Azure Database Migration Service**.

    ![Azure Marketplace](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-marketplace.png)

2. На экране **Azure Database Migration Service** выберите **Создать**.

    ![Создание экземпляра Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-create1.png)
  
3. На экране **Создание службы миграции** укажите имя службы, подписку, новую или существующую группу ресурсов, а также расположение службы.

4. Выберите существующую виртуальную сеть или создайте новую.

    Виртуальная сеть предоставляет Azure Database Migration Service доступ к исходному экземпляру сервера PostgreSQL и целевому экземпляру Базы данных Azure для PostgreSQL.

    См. статью [Краткое руководство. Создание виртуальной сети с помощью портала Azure](../virtual-network/quick-create-portal.md).

5. Выберите ценовую категорию.

    Дополнительные сведения о ценовых категориях и затратах см. на [странице с описанием цен](https://aka.ms/dms-pricing).

    ![Настройка параметров экземпляра Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-settings4.png)

6. Щелкните **Просмотреть и создать**, чтобы создать службу.

   Создание службы потребует примерно 10 – 15 минут.

## <a name="create-a-migration-project"></a>Создание проекта миграции

После создания службы найдите ее на портале Azure, откройте и создайте проект миграции.

1. На портале Azure щелкните **Все службы**, выполните поиск по запросу "Azure Database Migration Service" и выберите **Azure Database Migration Services** (Службы Azure Database Migration Service).

      ![Поиск всех экземпляров Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-search.png)

2. На экране **Службы Azure Database Migration Service** найдите и выберите имя созданного экземпляра Azure Database Migration Service, а затем щелкните элемент **Новый проект миграции**.

3. На экране **Новый проект миграции** задайте имя проекта, в текстовом поле **Тип исходного сервера** выберите элемент **PostgreSQL**, а в текстовом поле **Тип целевого сервера** — элемент **База данных Azure для PostgreSQL**.

4. В разделе **Выберите тип действия** выберите **Миграция данных по сети**.

    ![Создание проекта Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-create-project.png)

    > [!NOTE]
    > Кроме того, вы можете выбрать **Создать только проект**, чтобы создать проект миграции для отложенного выполнения.

5. Щелкните элемент **Сохранить**, изучите требования для успешного использования Azure Database Migration Service для переноса данных, а затем щелкните элемент **Создать и выполнить действие**.

## <a name="specify-source-details"></a>Указание сведений об источнике

1. В окне **Добавление сведений об источнике** укажите сведения о подключении для исходного экземпляра PostgreSQL.

    ![Экран добавления сведений об источнике](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-add-source-details.png)

2. Щелкните **Сохранить**.

## <a name="specify-target-details"></a>Указание сведений о цели

1. На экране **Сведения о целевом объекте** укажите сведения для подключения к целевому серверу Гипермасштабирования (Citus), в качестве которого используется подготовленный ранее экземпляр Гипермасштабирования (Citus), где вы развернули схему **DVD** с помощью команды pg_dump.

    ![Экран сведений о целевом объекте](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-add-target-details.png)

2. Нажмите кнопку **Сохранить**, а затем на экране **Сопоставление с целевыми базами данных** сопоставьте исходную и целевую базы данных для миграции.

    Если в целевой базе данных содержится база данных с тем же именем, что у исходной базы данных, Azure Database Migration Service по умолчанию выберет целевую базу данных.

    ![Экран сопоставления с целевыми базами данных](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-map-target-databases.png)

3. Щелкните элемент **Сохранить**, а затем на экране **Параметры миграции** подтвердите значения по умолчанию.

    ![Экран параметров миграции](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-migration-settings.png)

4. Нажмите кнопку **Сохранить** в окне **Migration summary** (Сводка по миграции) и в поле **Имя действия** введите имя действия миграции, а затем проверьте соответствие сведениям о целевом и исходном сервере, которые вы указали ранее.

    ![Экран сводных данных по миграции](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-migration-summary.png)

## <a name="run-the-migration"></a>Выполнение миграции

* Выберите **Запустить миграцию**.

    Откроется окно "Действие миграции", а для действия в поле **Состояние** отобразится надпись **Выполняется резервное копирование**.

## <a name="monitor-the-migration"></a>Мониторинг миграции

1. На экране действия миграции нажмите кнопку **Обновить**, чтобы обновить содержимое экрана, пока **состояние** миграции не поменяется на **Завершено**.

     ![Мониторинг миграции](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-monitor-migration.png)

2. Когда миграция завершится, выберите в столбце **Имя базы данных** конкретную базу данных, чтобы узнать состояние миграции для операций **полной загрузки данных** и **добавочной синхронизации данных**.

   > [!NOTE]
   > В разделе **полной загрузки данных** отображается состояние начальной загрузки, а в разделе **добавочной синхронизации данных** — состояние отслеживания измененных данных (CDC).

     ![Сведения о полной загрузке данных](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-full-data-load-details.png)

     ![Сведения о добавочной синхронизации данных](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-incremental-data-sync-details.png)

## <a name="perform-migration-cutover"></a>Выполнение прямой миграции

После завершения начальной полной загрузки базы данных помечаются как **готовые к прямой миграции**.

1. Когда вы будете готовы выполнить миграцию базы данных, щелкните **Запустить прямую миграцию**.

2. Подождите, пока счетчик **Ожидающие изменения** не дойдет до **0**, что означает полное прекращение всех входящих транзакций в исходной базе данных, затем установите флажок **Подтвердить** и щелкните элемент **Применить**.

    ![Экран завершения миграции](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-complete-cutover.png)

3. Когда состояние переноса базы данных изменится на **Завершено**, [повторно создайте последовательности](https://wiki.postgresql.org/wiki/Fixing_Sequences) (если это необходимо) и подключите свои приложения к новому целевому экземпляру Базы данных Azure для PostgreSQL.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об известных проблемах, ограничениях при выполнении сетевой миграции в Базу данных Azure для PostgreSQL см. в [этой](known-issues-azure-postgresql-online.md) статье.
* См. дополнительные сведения о [службе Azure Database Migration Service](./dms-overview.md).
* Общие сведения о Базе данных Azure для PostgreSQL см. в статье [Что такое база данных Azure для PostgreSQL](../postgresql/overview.md).