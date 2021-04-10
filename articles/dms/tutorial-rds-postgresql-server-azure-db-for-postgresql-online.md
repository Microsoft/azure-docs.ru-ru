---
title: Руководство по миграции из RDS PostgreSQL в Базу данных Azure для PostgreSQL по сети
titleSuffix: Azure Database Migration Service
description: Узнайте, как выполнить сетевую миграцию из RDS PostgreSQL в Базу данных Azure для PostgreSQL с помощью Azure Database Migration Service.
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
ms.openlocfilehash: c2a6a71365b48fa4349306ce632f5762c38dacf7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104597476"
---
# <a name="tutorial-migrate-rds-postgresql-to-azure-db-for-postgresql-online-using-dms"></a>Руководство по Миграция из PostgreSQL RDS в Базу данных Azure для PostgreSQL по сети с использованием DMS

С помощью службы Azure Database Migration Service базы данных из экземпляра RDS PostgreSQL можно перенести в [Базу данных Azure для PostgreSQL](../postgresql/index.yml), не отключая базу данных-источник от сети. Другими словами, миграцию можно выполнить с минимальным временем простоя для приложения. В этом руководстве выполняется миграция примера базы данных **Прокат DVD** из экземпляра RDS PostgreSQL 9.6 в Базу данных Azure для PostgreSQL с помощью действия сетевой миграции в Azure Database Migration Service.

В этом руководстве описано следующее:
> [!div class="checklist"]
>
> * перенос примера схемы с помощью служебной программы pg_dump;
> * Создайте экземпляр Azure Database Migration Service.
> * создание проекта миграции с использованием Azure Database Migration Service.
> * выполнение миграции.
> * Мониторинг миграции.
> * выполнить переключение после миграции.

> [!NOTE]
> Чтобы выполнить подключенную миграцию с помощью Azure Database Migration Service, требуется создать экземпляр ценовой категории "Премиум". Дополнительные сведения см. на странице с [ценами](https://azure.microsoft.com/pricing/details/database-migration/) на Azure Database Migration Service. Мы шифруем диск для защиты данных от кражи при миграции.

> [!IMPORTANT]
> Чтобы оптимизировать процесс миграции, Майкрософт рекомендует создать экземпляр Azure Database Migration Service в том же регионе Azure, в котором размещена целевая база данных. Перемещение данных между регионами и географическими областями может замедлить процесс миграции и привести к ошибкам.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

В этой статье описывается сетевая миграция из экземпляра RDS PostgreSQL в Базу данных Azure для PostgreSQL.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

* Скачайте и установите [PostgreSQL Community Edition](https://www.postgresql.org/download/) версии 9.5, 9.6 или 10. На исходном сервере должна быть установлена PostgreSQL версии 9.5.11, 9.6.7, 10 или более поздней. Дополнительные сведения см. в статье [Поддерживаемые версии базы данных PostgreSQL](../postgresql/concepts-supported-versions.md).

   Также обратите внимание, что версия целевой Базы данных Azure для PostgreSQL не может быть более ранней, чем версия RDS PostgreSQL. Например, RDS PostgreSQL 9.6 можно перенести только в Базу данных Azure для PostgreSQL 9.6, 10 или 11, но не в Базу данных Azure для PostgreSQL 9.5.

* Создайте экземпляр [Базы данных Azure для PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) или [Базы данных Azure для PostgreSQL с Гипермасштабированием (Citus)](../postgresql/quickstart-create-hyperscale-portal.md). В [этом разделе](../postgresql/quickstart-create-server-database-portal.md#connect-to-the-server-with-psql) документа приводятся сведения о том, как подключиться к серверу PostgreSQL с помощью pgAdmin.
* Создайте виртуальную сеть Microsoft Azure для Azure Database Migration Service с помощью модели развертывания Azure Resource Manager, которая обеспечивает подключение "сеть — сеть" к локальным исходным серверам с помощью [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Дополнительные сведения о создании виртуальной сети приведены в [документации по виртуальным сетям](../virtual-network/index.yml). В частности, уделите внимание кратким руководствам с пошаговыми инструкциями.
* Убедитесь, что правила группы безопасности сети для виртуальной сети не блокируют исходящий порт 443 ServiceTag для Служебной шины, службы хранилища и Azure Monitor. См. дополнительные сведения о [фильтрации трафика, предназначенного для виртуальной сети, с помощью групп безопасности сети](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Настройте [брандмауэр Windows для доступа к ядру СУБД](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Откройте брандмауэр Windows, чтобы предоставить Azure Database Migration Service доступ к исходному серверу PostgreSQL. По умолчанию это TCP-порт 5432.
* Если перед исходными базами данных развернуто устройство брандмауэра, вам может понадобиться добавить правила брандмауэра, чтобы позволить службе Azure Database Migration Service обращаться к исходным базам данных для выполнения миграции.
* Создайте [правило брандмауэра](../postgresql/concepts-firewall-rules.md) уровня сервера для сервера Базы данных Azure для PostgreSQL, чтобы предоставить службе Azure Database Migration Service доступ к целевым базам данных. Задайте диапазон подсети в виртуальной сети, которая используется для Azure Database Migration Service.

### <a name="set-up-aws-rds-postgresql-for-replication"></a>Настройка репликации для AWS RDS PostgreSQL

1. Чтобы создать новую группу параметров, выполните инструкции AWS из статьи [о работе с группами параметров базы данных](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html).
2. Используйте имя главного пользователя, чтобы подключиться к источнику из Azure Database Migration Service. Если вы используете учетную запись, отличную от учетной записи главного пользователя, она должна иметь роли rds_superuser и rds_replication. Роль rds_replication предоставляет разрешения на управление логическими слотами и потоковую передачу данных через логические слоты.
3. Создайте группу параметров со следующей конфигурацией:

    a. Для параметра rds.logical_replication в группе параметров базы данных задайте значение 1.

    b. max_wal_senders = [количество параллельных задач]. Параметр max_wal_senders задает число параллельных задач, которые можно выполнить, рекомендуемое значение — 10 задач.

    c. max_replication_slots = [количество слотов], рекомендуемое значение — до 5 слотов.

4. Свяжите созданную группу параметров с экземпляром RDS PostgreSQL.

## <a name="migrate-the-schema"></a>Перенос схемы

1. Чтобы перенести все объекты базы данных, такие как схемы таблицы, индексы и хранимые процедуры, извлеките схему из исходной базы данных и примените ее к нужной базе данных.

    Чтобы перенести только схему, проще всего выполнить pg_dump с параметром -s. Дополнительные сведения см. в разделе [примеров](https://www.postgresql.org/docs/9.6/app-pgdump.html#PG-DUMP-EXAMPLES) в руководстве по pg_dump для Postgres.

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Например, чтобы выгрузить файл схемы дампа для базы данных **dvdrental**, используйте следующую команду:

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

2. Создайте пустую базу данных в целевой службе, то есть в Базе данных Azure для PostgreSQL. Чтобы подключиться и создать базу данных, воспользуйтесь одной из следующих статей.

    * [Create an Azure Database for PostgreSQL by using the Azure portal](../postgresql/quickstart-create-server-database-portal.md) (Создание базы данных Azure для PostgreSQL с помощью портала Azure)
    * [Создание сервера Базы данных Azure для PostgreSQL с Гипермасштабированием (Citus) на портале Azure](../postgresql/quickstart-create-hyperscale-portal.md)

3. Импортируйте схему в целевую службу, то есть в Базу данных Azure для PostgreSQL. Чтобы восстановить файл дампа схемы, выполните следующую команду:

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    Пример:

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

4. Если у вас есть внешние ключи в схеме, начальная загрузка и непрерывная синхронизация во время миграции завершатся сбоем. Выполните следующий сценарий в PgAdmin или psql, чтобы извлечь внешний ключ сценария удаления и добавить сценарий внешнего ключа в назначении (База данных Azure для PostgreSQL).
  
    ```
    SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ',
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema,
        tc.constraint_name as foreignkey,
        tc.table_name as tableName,
        kcu.column_name,
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name
    FROM
        information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
    ```

5. Запустите команду drop foreign key в результате запроса, чтобы удалить внешний ключ (второй столбец).

6. Если у вас есть триггеры (триггер вставки или обновления), они будут обеспечивать целостность данных в целевом объекте перед реплицированием данных из источника. Рекомендуется отключить триггеры во всех таблицах *целевого объекта* на время миграции, а затем включить их после завершения миграции.

    Чтобы удалить триггеры в целевой базе данных, выполните:

    ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name,' ON ', event_object_table, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
    ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Регистрация поставщика ресурсов Microsoft.DataMigration

1. Войдите на портал Azure, щелкните **Все службы** и выберите **Подписки**.

   ![Отображение подписок на портале](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-subscription1.png)

2. Выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Отображение поставщиков ресурсов](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-resource-provider.png)

3. В поле поиска введите migration, а затем справа от **Microsoft.DataMigration** щелкните **Зарегистрировать**.

    ![Регистрация поставщика ресурсов](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>Создание экземпляра Azure Database Migration Service

1. На портале Azure выберите **+Создать ресурс**, введите в поле поиска "Azure Database Migration Service", а затем в раскрывающемся списке выберите **Azure Database Migration Service**.

    ![Azure Marketplace](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-marketplace.png)

2. На экране **Azure Database Migration Service** выберите **Создать**.

    ![Создание экземпляра Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. На экране **Создание службы миграции** укажите имя службы, подписку и новую или существующую группу ресурсов.

4. Выберите расположение, в котором нужно создать экземпляр Azure Database Migration Service.

5. Выберите существующую виртуальную сеть или создайте новую.

    Виртуальная сеть предоставляет Azure Database Migration Service доступ к исходному экземпляру PostgreSQL и целевому экземпляру Базы данных Azure для PostgreSQL.

    См. статью [Краткое руководство. Создание виртуальной сети с помощью портала Azure](../virtual-network/quick-create-portal.md).

6. Выберите ценовую категорию. Для этой миграции по сети необходимо выбрать ценовую категорию "Премиум: 4 виртуальных ядра".

    ![Настройка параметров экземпляра Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-settings5.png)

7. Выберите **Создать**, чтобы создать службу.

## <a name="create-a-migration-project"></a>Создание проекта миграции

После создания службы найдите ее на портале Azure, откройте и создайте проект миграции.

1. На портале Azure щелкните **Все службы**, выполните поиск по запросу "Azure Database Migration Service" и выберите **Azure Database Migration Services** (Службы Azure Database Migration Service).

      ![Поиск всех экземпляров Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-search.png)

2. На экране **Службы Azure Database Migration Service** найдите и выберите имя созданного экземпляра Azure Database Migration Service, а затем щелкните элемент **Новый проект миграции**.
3. На экране **Новый проект миграции** задайте имя проекта, в текстовом поле **Тип исходного сервера** выберите **AWS RDS для PostgreSQL**, а в текстовом поле **Тип целевого сервера** — **База данных Azure для PostgreSQL**.
4. В разделе **Выберите тип действия** выберите **Миграция данных по сети**.

    > [!IMPORTANT]
    > Не забудьте выбрать режим **миграции данных по сети**, так как автономная миграция не поддерживается для этого сценария.

    ![Создание проекта Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-create-project5.png)

    > [!NOTE]
    > Кроме того, вы можете выбрать **Создать только проект**, чтобы создать проект миграции для отложенного выполнения.

5. Щелкните **Сохранить**.

6. Выберите **Создать и выполнить действие**, чтобы создать проект и выполнить действие миграции.

    > [!NOTE]
    > Запишите предварительные требования, необходимые для настройки интерактивной миграции, в колонке создания проекта.

## <a name="specify-source-details"></a>Указание сведений об источнике

* В окне **Добавление сведений об источнике** укажите сведения о подключении для исходного экземпляра PostgreSQL.

   ![Сведения об источнике](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-source-details5.png)

## <a name="specify-target-details"></a>Указание сведений о цели

1. Нажмите кнопку **Сохранить**, а затем на экране **Сведения о целевом объекте** укажите сведения о подключении для предварительно подготовленного целевого сервера Базы данных Azure для PostgreSQL, для которого была развернута схема **Прокат DVD** с помощью PostgreSQL.

    ![Сведения о целевом объекте](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-target-details.png)

2. Нажмите кнопку **Сохранить**, а затем на экране **Сопоставление с целевыми базами данных** сопоставьте исходную и целевую базы данных для миграции.

    Если в целевой базе данных содержится база данных с тем же именем, что у исходной базы данных, Azure Database Migration Service по умолчанию выберет целевую базу данных.

    ![Сопоставление с целевыми базами данных](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-map-target-databases.png)

3. Нажмите кнопку **Сохранить** в окне **Migration summary** (Сводка по миграции) и в поле **Имя действия** введите имя действия миграции, а затем проверьте соответствие сведениям о целевом и исходном сервере, которые вы указали ранее.

    ![Сводка по миграции](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-migration-summary1.png)

## <a name="run-the-migration"></a>Выполнение миграции

* Выберите **Запустить миграцию**.

    Появится окно действия миграции и в поле **Состояние** будет указано **Инициализация**.

## <a name="monitor-the-migration"></a>Мониторинг миграции

1. На экране действия миграции нажимайте **Обновить**, чтобы периодически обновлять содержимое экрана, пока в поле **Состояние** не будет указано **Выполнение**.

    ![Состояние действия "Выполняется"](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-activity-status3.png)

2. В столбце **Имя базы данных** выберите базу данных, чтобы узнать состояние миграции для операций **полной загрузки данных** и **добавочной синхронизации данных**.

    В разделе **полной загрузки данных** отображается состояние начальной загрузки, а в разделе **добавочной синхронизации данных** — состояние отслеживания измененных данных (CDC).

    ![Экран инвентаризации — полная загрузка данных](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-full-load.png)

    ![Экран инвентаризации — добавочная синхронизация данных](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>Выполнение прямой миграции

После завершения начальной полной загрузки базы данных помечаются как **готовые к прямой миграции**.

1. Когда вы будете готовы выполнить миграцию базы данных, щелкните **Запустить прямую миграцию**.

2. Подождите, пока счетчик **Ожидающие изменения** не дойдет до **0**, что означает полное прекращение всех входящих транзакций в исходной базе данных, затем установите флажок **Подтвердить** и щелкните элемент **Применить**.

    ![Экран завершения миграции](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-complete-cutover.png)

3. Когда состояние переноса базы данных изменится на **Завершено**, подключите свои приложения к новому целевому экземпляру Базы данных Azure для PostgreSQL.

Теперь перенос локального экземпляра RDS PostgreSQL в Базу данных Azure для PostgreSQL по сети завершен.

## <a name="next-steps"></a>Дальнейшие действия

* См. дополнительные сведения о [службе Azure Database Migration Service](./dms-overview.md).
* Общие сведения о Базе данных Azure для PostgreSQL см. в статье [Что такое база данных Azure для PostgreSQL](../postgresql/overview.md).
* По другим вопросам отправляйте сообщения электронной почты на псевдоним [Спросить о миграции базы данных Azure](mailto:AskAzureDatabaseMigrations@service.microsoft.com).
