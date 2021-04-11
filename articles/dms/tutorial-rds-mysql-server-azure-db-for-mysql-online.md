---
title: Руководство по сетевой миграции из RDS MySQL в Базу данных Azure для MySQL
titleSuffix: Azure Database Migration Service
description: Узнайте, как выполнить интерактивную миграцию из RDS MySQL в Базу данных Azure для MySQL с помощью Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 06/09/2020
ms.openlocfilehash: 13095824122f1d4b59ecde445efa1ea8bfa60dcd
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106063582"
---
# <a name="tutorial-migrate-rds-mysql-to-azure-database-for-mysql-online-using-dms"></a>Руководство. Миграция из MySQL RDS в Базу данных Azure для MySQL через Интернет

С помощью службы Azure Database Migration Service базы данных из экземпляра RDS MySQL можно перенести в [Базу данных Azure для MySQL](../mysql/index.yml), не отключая базу данных-источник от сети. Другими словами, миграцию можно выполнить с минимальным временем простоя для приложения. В этом руководстве выполняется перенос примера базы данных **сотрудников** из экземпляра RDS MySQL в Базу данных Azure для MySQL с помощью интерактивного действия миграции в Azure Database Migration Service.

> [!IMPORTANT]
> 1 июня 2021 года сценарий миграции RDS MySQL в Базу данных Azure для MySQL по сети будет заменен соответствующим высокопроизводительным сценарием миграции без подключения к сети. Для миграции по сети можно использовать это новое предложение с [репликацией входных данных](https://docs.microsoft.com/azure/mysql/concepts-data-in-replication). Для такой миграции также можно использовать средства с открытым кодом, например [MyDumper/MyLoader](https://centminmod.com/mydumper.html), вместе с репликацией входных данных. 

В этом руководстве описано следующее:
> [!div class="checklist"]
>
> * Перенос примера схемы с помощью служебных программ mysqldump и mysql.
> * Создайте экземпляр Azure Database Migration Service.
> * создание проекта миграции с использованием Azure Database Migration Service.
> * выполнение миграции.
> * Мониторинг миграции.

> [!NOTE]
> Чтобы выполнить сетевую миграцию с помощью Azure Database Migration Service, требуется создать экземпляр ценовой категории "Премиум". Дополнительные сведения см. на странице с [ценами](https://azure.microsoft.com/pricing/details/database-migration/) на Azure Database Migration Service.

> [!IMPORTANT]
> Чтобы оптимизировать процесс миграции, Майкрософт рекомендует создать экземпляр Azure Database Migration Service в том же регионе Azure, в котором размещена целевая база данных. Перемещение данных между регионами и географическими областями может замедлить процесс миграции и привести к ошибкам.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

В этой статье описывается выполнение интерактивного переноса из экземпляра RDS MySQL в Базу данных Azure для MySQL.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

* Убедитесь, что на исходном сервере MySQL работает поддерживаемая версия MySQL Community Edition. Чтобы определить версию экземпляра MySQL, в служебной программе mysql или MySQL Workbench выполните команду:

    ```
    SELECT @@version;
    ```

    Дополнительные сведения см. в статье [Supported Azure Database for MySQL server versions](../mysql/concepts-supported-versions.md) (Поддерживаемые версии серверов Базы данных Azure для MySQL).

* Скачайте и установите пример базы данных [MySQL **Сотрудники**](https://dev.mysql.com/doc/employee/en/employees-installation.html).
* Создайте экземпляр [Базы данных Azure для MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-portal.md).
* Создайте виртуальную сеть Microsoft Azure для Azure Database Migration Service с помощью модели развертывания Azure Resource Manager, которая обеспечивает подключение "сеть — сеть" к локальным исходным серверам с помощью [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Дополнительные сведения о создании виртуальной сети приведены в [документации по виртуальным сетям](../virtual-network/index.yml). В частности, уделите внимание кратким руководствам с пошаговыми инструкциями.
* Убедитесь, что правила группы безопасности сети для виртуальной сети не блокируют исходящий порт 443 ServiceTag для Служебной шины, службы хранилища и Azure Monitor. Дополнительные сведения о фильтрации трафика, предназначенного для виртуальной сети, с помощью NSG см. в статье [Планирование виртуальных сетей](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Настройте [брандмауэр Windows](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access) (или Linux) для доступа к ядру СУБД. Для сервера MySQL разрешите подключение через порт 3306.

> [!NOTE]
> База данных Azure для MySQL поддерживает только таблицы InnoDB. Чтобы преобразовать таблицы MyISAM в InnoDB, ознакомьтесь со сведениями на странице о [преобразовании таблиц](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html).

### <a name="set-up-aws-rds-mysql-for-replication"></a>Настройка AWS RDS MySQL для репликации

1. Чтобы создать группу параметров, выполните инструкции, предоставленные AWS в статье о [файлах журнала Базы данных MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.MySQL.html) в разделе о **формате ведения двоичного журнала**.
2. Создайте группу параметров со следующей конфигурацией:
    * log_bin = ON
    * binlog_format = row
    * binlog_checksum = NONE
3. Сохраните новую группу параметров.
4. Свяжите новую группу параметров с экземпляром MySQL RDS. Может потребоваться перезагрузка.
5. Создав группу параметров, подключитесь к экземпляру MySQL и [задайте для параметра хранения двоичных файлов журналов](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/mysql_rds_set_configuration.html#mysql_rds_set_configuration-usage-notes.binlog-retention-hours) значение, равное не менее 5 дням.
```
call mysql.rds_set_configuration('binlog retention hours', 120);
```

## <a name="migrate-the-schema"></a>Перенос схемы

1. Чтобы перенести все объекты базы данных, такие как схемы таблицы, индексы и хранимые процедуры, извлеките схему из исходной базы данных и примените ее к нужной базе данных.

    Самый простой способ переноса только схемы заключается в использовании mysqldump с параметром --no-data. Команда для переноса схемы выглядит так:

    ```
    mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
    ```
    
    Например, чтобы выгрузить файл схемы дампа для базы данных **Сотрудники**, используйте следующую команду:
    
    ```
    mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
    ```

2. Импортируйте схему в целевую службу, которая является Базой данных Azure для MySQL. Чтобы восстановить файл дампа схемы, выполните следующую команду:

    ```
    mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
    ```

    Например, чтобы импортировать схему для базы данных **Сотрудники**:

    ```
    mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
    ```

3. Если у вас есть внешние ключи в схеме, начальная загрузка и непрерывная синхронизация во время миграции завершатся сбоем. Чтобы извлечь сценарий удаления и сценарий добавления внешнего ключа в назначении (База данных Azure для MySQL), выполните следующий сценарий в MySQL Workbench.

    ```
    SET group_concat_max_len = 8192;
        SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
        FROM
        (SELECT
        KCU.REFERENCED_TABLE_SCHEMA as SchemaName,
                    KCU.TABLE_NAME,
                    KCU.COLUMN_NAME,
                    CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
                    FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
                    WHERE
                      KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
                      AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
      AND KCU.REFERENCED_TABLE_SCHEMA = 'SchemaName') Queries
      GROUP BY SchemaName;
    ```

4. Запустите команду drop foreign key в результате запроса, чтобы удалить внешний ключ (второй столбец).

> [!NOTE]
> Azure Database Migration Service не поддерживает ссылочное действие CASCADE, которое позволяет автоматически удалить или обновить совпадающую строку в дочерней таблице при удалении или обновлении строки в родительской таблице. Дополнительные сведения см. в статье [Ограничения для внешних ключей](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html) в документации по MySQL.
> В Azure Database Migration Service нужно снять ограничения для внешних ключей на сервере целевой базы данных во время начальной загрузки данных. При этом нельзя использовать ссылочные действия. Если ваша рабочая нагрузка зависит от обновлений связанной дочерней таблицы, выполняемых с помощью ссылочного действия, рекомендуется переносить базу данных с помощью [дампа и восстановления](../mysql/concepts-migrate-dump-restore.md). 

5. Если у вас есть триггеры (триггер вставки или обновления), они будут обеспечивать целостность данных в целевом объекте перед реплицированием данных из источника. Рекомендуется отключить триггеры во всех таблицах *целевого объекта* на время миграции, а затем включить их после завершения миграции.

    Чтобы удалить триггеры в целевой базе данных, выполните:

    ```
    select concat ('alter table ', event_object_table, ' disable trigger ', trigger_name)
    from information_schema.triggers;
    ```

6. Если в таблицах есть экземпляры типа данных ENUM, мы рекомендуем временно обновить его до типа данных character varying в целевой таблице. После завершения репликации данных отмените изменение типа данных и снова преобразуйте их в ENUM.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Регистрация поставщика ресурсов Microsoft.DataMigration

1. Войдите на портал Azure, щелкните **Все службы** и выберите **Подписки**.

   ![Отображение подписок на портале](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-select-subscription1.png)

2. Выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Отображение поставщиков ресурсов](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-select-resource-provider.png)

3. В поле поиска введите migration, а затем справа от **Microsoft.DataMigration** щелкните **Зарегистрировать**.

    ![Регистрация поставщика ресурсов](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>Создание экземпляра Azure Database Migration Service

1. На портале Azure выберите **+Создать ресурс**, введите в поле поиска "Azure Database Migration Service", а затем в раскрывающемся списке выберите **Azure Database Migration Service**.

    ![Azure Marketplace](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-marketplace.png)

2. На экране **Azure Database Migration Service** выберите **Создать**.

    ![Создание экземпляра Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-create1.png)
  
3. На экране **Создание службы миграции** укажите имя службы, подписку и новую или существующую группу ресурсов.

4. Выберите расположение, в котором нужно создать экземпляр Azure Database Migration Service.

5. Выберите существующую виртуальную сеть или создайте новую.

    Виртуальная сеть предоставляет Azure Database Migration Service доступ к исходному экземпляру MySQL и целевому экземпляру Базы данных Azure для MySQL.

    См. статью [Краткое руководство. Создание виртуальной сети с помощью портала Azure](../virtual-network/quick-create-portal.md).

6. Выберите ценовую категорию. Для этой миграции по сети необходимо выбрать ценовую категорию "Премиум: 4 виртуальных ядра".

    ![Настройка параметров экземпляра Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-settings3.png)

7. Выберите **Создать**, чтобы создать службу.

## <a name="create-a-migration-project"></a>Создание проекта миграции

После создания службы найдите ее на портале Azure, откройте и создайте проект миграции.

1. На портале Azure щелкните **Все службы**, выполните поиск по запросу "Azure Database Migration Service" и выберите **Azure Database Migration Services** (Службы Azure Database Migration Service).

      ![Поиск всех экземпляров Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-search.png)

2. На экране **Azure Database Migration Services** найдите имя созданного экземпляра Azure Database Migration Service и выберите его.

     ![Поиск экземпляра службы миграции базы данных Azure](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-instance-search.png)

3. Выберите **+ Новый проект миграции**.
4. В окне **New migration project** (Новый проект миграции) задайте имя для проекта, в текстовом поле **Source server type** (Тип исходного сервера) выберите **MySQL**, а затем в текстовом поле **Target server type** (Тип целевого сервера) выберите **AzureDbForMySQL**.
5. В разделе **Выберите тип действия** выберите **Миграция данных по сети**.

    > [!IMPORTANT]
    > Не забудьте выбрать режим **миграции данных по сети**, так как автономная миграция не поддерживается для этого сценария.

    ![Создание проекта Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-create-project6.png)

    > [!NOTE]
    > Кроме того, вы можете выбрать **Создать только проект**, чтобы создать проект миграции для отложенного выполнения.

6. Щелкните **Сохранить**.

7. Выберите **Создать и выполнить действие**, чтобы создать проект и выполнить действие миграции.

    > [!NOTE]
    > Запишите предварительные требования, необходимые для настройки интерактивной миграции, в колонке создания проекта.

## <a name="specify-source-details"></a>Указание сведений об источнике

* На экране **Migration source detail** (Сведения об источнике миграции) задайте сведения о подключении для исходного экземпляра MySQL.

   ![Сведения об источнике](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-source-details5.png)

## <a name="specify-target-details"></a>Указание сведений о цели

1. Нажмите кнопку **Сохранить**, а затем на экране **Сведения о целевом объекте** укажите сведения о подключении для целевого сервера Базы данных Azure для MySQL, а именно — предварительно подготовленного и для которого была развернута схема **Сотрудники** с помощью MySQLDump.

    ![Выбор цели](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-select-target5.png)

2. Нажмите кнопку **Сохранить**, а затем на экране **Сопоставление с целевыми базами данных** сопоставьте исходную и целевую базы данных для миграции.

    Если в целевой базе данных содержится база данных с тем же именем, что у исходной базы данных, Azure Database Migration Service по умолчанию выберет целевую базу данных.

    ![Сопоставление с целевыми базами данных](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-map-targets-activity5.png)

3. Нажмите кнопку **Сохранить** в окне **Migration summary** (Сводка по миграции) и в поле **Имя действия** введите имя действия миграции, а затем проверьте соответствие сведениям о целевом и исходном сервере, которые вы указали ранее.

    ![Сводка по миграции](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-migration-summary2.png)

## <a name="run-the-migration"></a>Выполнение миграции

* Выберите **Запустить миграцию**.

    Появится окно действия миграции и в поле **Состояние** будет указано **Инициализация**.

## <a name="monitor-the-migration"></a>Мониторинг миграции

1. На экране действия миграции нажимайте **Обновить**, чтобы периодически обновлять содержимое экрана, пока в поле **Состояние** не будет указано **Выполнение**.

    ![Состояние действия "Выполняется"](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-activity-status4.png)

2. В столбце **Имя базы данных** выберите базу данных, чтобы узнать состояние миграции для операций **полной загрузки данных** и **добавочной синхронизации данных**.

    В разделе **полной загрузки данных** отображается состояние начальной загрузки, а в разделе **добавочной синхронизации данных** — состояние отслеживания измененных данных (CDC).

    ![Экран инвентаризации — полная загрузка данных](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-full-load.png)

    ![Экран инвентаризации — добавочная синхронизация данных](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>Выполнение прямой миграции

После завершения начальной полной загрузки базы данных помечаются как **готовые к прямой миграции**.

1. Когда вы будете готовы выполнить миграцию базы данных, щелкните **Запустить прямую миграцию**.

    ![Начало перехода](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-start-cutover.png)

2. Обязательно остановите все входящие транзакции в исходную базу данных. Подождите, пока в счетчике **Ожидающие изменения** появится значение **0**.
3. Выберите **Подтвердить**, а затем — **Применить**.
4. Когда состояние переноса базы данных изменится на **Завершено**, подключите свои приложения к новой целевой Базе данных Azure для MySQL.

Интерактивный перенос локального экземпляра MySQL в Базу данных Azure для MySQL выполнен.

## <a name="next-steps"></a>Следующие шаги

* См. дополнительные сведения о [службе Azure Database Migration Service](./dms-overview.md).
* Общие сведения о Базе данных Azure для MySQL см. в [этой статье](../mysql/overview.md).
* По другим вопросам отправляйте сообщения электронной почты на псевдоним [Спросить о миграции базы данных Azure](mailto:AskAzureDatabaseMigrations@service.microsoft.com).
