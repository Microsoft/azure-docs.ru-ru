---
title: Руководство по Миграция из MySQL в Базу данных Azure для MySQL по сети
titleSuffix: Azure Database Migration Service
description: Узнайте, как выполнить интерактивную миграцию из MySQL в локальной среде в Базу данных Azure для MySQL с помощью Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/08/2020
ms.openlocfilehash: 050356dc74641e7e7154d6a3976e6cc8d5f4ce3d
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106063616"
---
# <a name="tutorial-migrate-mysql-to-azure-database-for-mysql-online-using-dms"></a>Руководство. Перенос MySQL в Базу данных Azure для MySQL по сети с помощью DMS

Azure Database Migration Service можно использовать для переноса баз данных из локального экземпляра MySQL в [Базу данных Azure для MySQL](../mysql/index.yml) с минимальным временем простоя. Другими словами, миграцию можно выполнить с минимальным временем простоя для приложения. В этом руководстве описано, как выполнить миграцию примера базы данных **сотрудников** из локального экземпляра MySQL 5.7 в Базу данных Azure для MySQL с помощью интерактивного действия миграции в Azure Database Migration Service.

> [!IMPORTANT]
> 1 июня 2021 года сценарий миграции MySQL в Базу данных Azure для MySQL по сети будет заменен соответствующим высокопроизводительным сценарием миграции без подключения к сети. Для миграции по сети можно использовать это новое предложение с [репликацией входных данных](https://docs.microsoft.com/azure/mysql/concepts-data-in-replication). Для такой миграции также можно использовать средства с открытым кодом, например [MyDumper/MyLoader](https://centminmod.com/mydumper.html), вместе с репликацией входных данных. 

В этом руководстве описано следующее:
> [!div class="checklist"]
>
> * перенос примера схемы с помощью служебной программы mysqldump;
> * Создайте экземпляр Azure Database Migration Service.
> * создание проекта миграции с использованием Azure Database Migration Service.
> * выполнение миграции.
> * Мониторинг миграции.

> [!NOTE]
> Чтобы выполнить сетевую миграцию с помощью Azure Database Migration Service, требуется создать экземпляр ценовой категории "Премиум".

> [!IMPORTANT]
> Чтобы процесс миграции был выполнен без проблем, Майкрософт рекомендует создать экземпляр Azure Database Migration Service в том же регионе Azure, в котором размещена целевая база данных. Перемещение данных между регионами и географическими областями может замедлить процесс миграции и привести к ошибкам.

> [!NOTE]
> Обмен данными без смещения
>
> Корпорация Майкрософт поддерживает принципы инклюзивности, а также этнического и социокультурного многообразия. В версии этой статьи на английском языке используется термин _slave_ (раб, ведомый). В [руководстве по стилю для политкорректной коммуникации](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) корпорации Майкрософт это слово указано как недопустимое. Термин приводится в версии этой статьи на английском языке в целях согласования, так как сейчас он используется в программном обеспечении. После обновления программного обеспечения с удалением этого слова статья будет изменена соответствующим образом.
>


## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

* Скачайте и установите [MySQL Community Edition](https://dev.mysql.com/downloads/mysql/) версии 5.6 или 5.7. Локальная версия MySQL должна соответствовать версии Базы данных Azure для MySQL. Например, MySQL версии 5.6 можно перенести только в Базу данных Azure для MySQL версии 5.6, но не версии 5.7. Миграция в версию MySQL 8.0 или из нее не поддерживается.
* [Создайте экземпляр в Базе данных Azure для MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-portal.md). Ознакомьтесь со статьей [База данных Azure для MySQL: подключение и запрос данных с помощью MySQL Workbench](../mysql/connect-workbench.md), чтобы узнать, как подключить и создать базу данных с помощью портала Azure.  
* Создайте виртуальную сеть Microsoft Azure для Azure Database Migration Service с помощью модели развертывания Azure Resource Manager, которая обеспечивает подключение "сеть — сеть" к локальным исходным серверам с помощью [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Дополнительные сведения о создании виртуальной сети приведены в [документации по виртуальным сетям](../virtual-network/index.yml). В частности, уделите внимание кратким руководствам с пошаговыми инструкциями.

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
* Откройте брандмауэр Windows, чтобы предоставить Azure Database Migration Service доступ к исходному серверу MySQL Server. По умолчанию это TCP-порт 3306.
* Если перед исходными базами данных развернуто устройство брандмауэра, вам может понадобиться добавить правила брандмауэра, чтобы позволить службе Azure Database Migration Service обращаться к исходным базам данных для выполнения миграции.
* Создайте [правило брандмауэра](../azure-sql/database/firewall-configure.md) уровня сервера для Базы данных Azure для MySQL, чтобы предоставить службе Azure Database Migration Service доступ к целевым базам данных. Задайте диапазон подсети в виртуальной сети, которая используется для Azure Database Migration Service.
* Исходный экземпляр MySQL должен находиться в поддерживаемой версии MySQL Community Edition. Чтобы определить версию экземпляра MySQL, в служебной программе MySQL или MySQL Workbench выполните следующую команду:

    ```
    SELECT @@version;
    ```

* База данных Azure для MySQL поддерживает только таблицы InnoDB. Чтобы преобразовать таблицы MyISAM в InnoDB, ознакомьтесь со сведениями на странице о [преобразовании таблиц](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html).

* Включите ведение двоичных журналов в файле my.ini (Windows) или my.cnf (Unix) в исходной базе данных, используя следующую конфигурацию:

  * **server_id** = 1 или больше (применимо только для версии MySQL 5.6)
  * **log-bin** =\<path> (применимо только для версии MySQL 5.6)    Пример: log-bin = E:\MySQL_logs\BinLog
  * **binlog_format** = row
  * **Expire_logs_days** = 5 (рекомендуется не использовать ноль; применимо только для версии MySQL 5.6)
  * **Binlog_row_image** = full (применимо только для версии MySQL 5.6)
  * **log_slave_updates** = 1

* У пользователя должна быть роль ReplicationAdmin со следующими привилегиями:

  * **REPLICATION CLIENT** — требуется только для задач обработки изменений. Другими словами, задачи только с полной загрузкой не требуют этой привилегии.
  * **REPLICATION REPLICA** — требуется только для задач обработки изменений. Другими словами, задачи только с полной загрузкой не требуют этой привилегии.
  * **SUPER** — требуется только в версиях, предшествующих MySQL 5.6.6.

## <a name="migrate-the-sample-schema"></a>Перенос примера схемы

Чтобы подготовить все объекты базы данных, такие как схемы таблицы, индексы и хранимые процедуры, нам нужно извлечь схему из исходной базы данных и применить ее к нужной базе данных. Чтобы извлечь схему, можно использовать mysqldump с параметром `--no-data`.

Предположим, что вы используете пример базы данных **Сотрудники** MySQL в локальной системе. Тогда команда для переноса схемы с помощью mysqldump будет выглядеть так:

```
mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
```

Пример:

```
mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
```

Чтобы импортировать схему в целевой объект Базы данных Azure для MySQL, выполните следующую команду:

```
mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
 ```

Пример:

```
mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
 ```

Если у вас есть внешние ключи в схеме, начальная загрузка и непрерывная синхронизация во время миграции завершатся сбоем.  Выполните следующий сценарий в MySQL Workbench для извлечения сценариев удаления и добавления внешнего ключа.

```sql
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

Запустите сценарий удаления внешнего ключа (второй столбец) в результате запроса, чтобы удалить внешний ключ.

> [!NOTE]
> Azure Database Migration Service не поддерживает ссылочное действие CASCADE, которое позволяет автоматически удалить или обновить совпадающую строку в дочерней таблице при удалении или обновлении строки в родительской таблице. Дополнительные сведения см. в статье [Ограничения для внешних ключей](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html) в документации по MySQL.
> В Azure Database Migration Service нужно снять ограничения для внешних ключей на сервере целевой базы данных во время начальной загрузки данных. При этом нельзя использовать ссылочные действия. Если ваша рабочая нагрузка зависит от обновлений связанной дочерней таблицы, выполняемых с помощью ссылочного действия, рекомендуется переносить базу данных с помощью [дампа и восстановления](../mysql/concepts-migrate-dump-restore.md). 


> [!IMPORTANT]
> Если импортируются данные с помощью резервной копии, удалите команды CREATE DEFINER вручную или с помощью команды --skip-definer при выполнении mysqldump. Чтобы выполнить команду DEFINER, требуются права суперпользователя. Ее запрещено выполнять в службе "База данных Azure для MySQL".

Если у вас есть триггер в данных (триггер вставки или обновления), он будет обеспечивать целостность данных в целевом объекте перед реплицированными данными из источника. Рекомендуется отключить триггеры во всех таблицах целевого объекта во время миграции, а затем включить их после завершения миграции.

Чтобы отключить триггеры в целевой базе данных, используйте следующую команду:

```
SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Регистрация поставщика ресурсов Microsoft.DataMigration

1. Войдите на портал Azure, щелкните **Все службы** и выберите **Подписки**.

   ![Отображение подписок на портале](media/tutorial-mysql-to-azure-mysql-online/portal-select-subscriptions.png)

2. Выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Отображение поставщиков ресурсов](media/tutorial-mysql-to-azure-mysql-online/portal-select-resource-provider.png)

3. В поле поиска введите migration, а затем справа от **Microsoft.DataMigration** щелкните **Зарегистрировать**.

    ![Регистрация поставщика ресурсов](media/tutorial-mysql-to-azure-mysql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Создание экземпляра DMS

1. На портале Azure выберите **+Создать ресурс**, введите в поле поиска "Azure Database Migration Service", а затем в раскрывающемся списке выберите **Azure Database Migration Service**.

    ![Azure Marketplace](media/tutorial-mysql-to-azure-mysql-online/portal-marketplace.png)

2. На экране **Azure Database Migration Service** выберите **Создать**.

    ![Создание экземпляра Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-create1.png)
  
3. На экране **Создание службы миграции** укажите имя службы, подписку и новую или существующую группу ресурсов.

4. Выберите существующую виртуальную сеть или создайте новую.

    Виртуальная сеть предоставляет Azure Database Migration Service доступ к исходному экземпляру SQL Server и целевому экземпляру Базы данных SQL Azure.

    См. статью [Краткое руководство. Создание виртуальной сети с помощью портала Azure](../virtual-network/quick-create-portal.md).

5. Выберите ценовую категорию.

    Дополнительные сведения о ценовых категориях и затратах см. на [странице с описанием цен](https://aka.ms/dms-pricing).

    ![Настройка параметров экземпляра Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-settings3.png)

6. Выберите **Создать**, чтобы создать службу.

## <a name="create-a-migration-project"></a>Создание проекта миграции

После создания службы найдите ее на портале Azure, откройте и создайте проект миграции.

1. На портале Azure щелкните **Все службы**, выполните поиск по запросу "Azure Database Migration Service" и выберите **Azure Database Migration Services** (Службы Azure Database Migration Service).

      ![Поиск всех экземпляров Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-search.png)

2. На экране **Службы миграции баз данных Azure** найдите и выберите имя созданного экземпляра Azure Database Migration Service.

     ![Поиск экземпляра службы Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-instance-search.png)

3. Выберите **+ Новый проект миграции**.
4. В окне **Новый проект миграции** задайте имя для проекта, в текстовом поле **Source server type** (Тип исходного сервера) выберите **MySQL**, а затем в текстовом поле **Target server type** (Тип целевого сервера) выберите **AzureDbForMySQL**.
5. В разделе **Choose type of activity** (Выберите тип действия) выберите **Online data migration** (Оперативное перемещение данных).

    ![Создание проекта Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-create-project4.png)

    > [!NOTE]
    > Кроме того, вы можете выбрать **Создать только проект**, чтобы создать проект миграции для отложенного выполнения.

6. Нажмите кнопку **Сохранить**, обратите внимание на требования для успешного переноса данных с помощью DMS, а затем выберите **Create and run activity** (Создать и запустить действие).

## <a name="specify-source-details"></a>Указание сведений об источнике

1. В окне **Add Source Details** (Добавление сведений об источнике) укажите сведения о подключении для исходного экземпляра MySQL.

    ![Экран добавления сведений об источнике](media/tutorial-mysql-to-azure-mysql-online/dms-add-source-details.png)

## <a name="specify-target-details"></a>Указание сведений о цели

1. Нажмите кнопку **Сохранить**, а затем на экране **данных целевого объекта** укажите сведения о подключении для целевого сервера Базы данных Azure для MySQL, а именно — предварительно подготовленного экземпляра базы данных, для которой была развернута схема **Сотрудники** с помощью mysqldump.

    ![Экран сведений о целевом объекте](media/tutorial-mysql-to-azure-mysql-online/dms-add-target-details.png)

2. Нажмите кнопку **Сохранить**, а затем на экране **Сопоставление с целевыми базами данных** сопоставьте исходную и целевую базы данных для миграции.

    Если в целевой базе данных содержится база данных с тем же именем, что у исходной базы данных, Azure Database Migration Service по умолчанию выберет целевую базу данных.

    ![Сопоставление с целевыми базами данных](media/tutorial-mysql-to-azure-mysql-online/dms-map-target-details.png)
   > [!NOTE] 
   > Хотя на этом шаге вы можете выбрать несколько баз данных, каждый экземпляр Azure Database Migration Service поддерживает до четырех баз данных для параллельной миграции. Кроме того, в каждой подписке действует ограничение в 10 экземпляров Azure Database Migration Service на регион. Например, если вы переносите 80 баз данных, в один регион вы можете параллельно перенести 40 из них и только при наличии 10 экземпляров Azure Database Migration Service.

3. Нажмите кнопку **Сохранить** в окне **Migration summary** (Сводка по миграции) и в поле **Имя действия** введите имя действия миграции, а затем проверьте соответствие сведениям о целевом и исходном сервере, которые вы указали ранее.

    ![Сводка по миграции](media/tutorial-mysql-to-azure-mysql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Выполнение миграции

* Выберите **Запустить миграцию**.

    Появится окно действия миграции и в поле **Состояние** будет указано **Инициализация**.

## <a name="monitor-the-migration"></a>Мониторинг миграции

1. На экране действия миграции нажмите кнопку **Обновить**, чтобы обновить содержимое экрана, пока **состояние** миграции не поменяется на **Завершено**.

     ![Состояние действия "Завершено"](media/tutorial-mysql-to-azure-mysql-online/dms-activity-completed.png)

2. В столбце **Имя базы данных** выберите определенную базу данных, чтобы получить состояние миграции для операций **полной загрузки данных** и **добавочной синхронизации данных**.

    В разделе полной загрузки данных отображается состояние начальной загрузки, а в разделе добавочной синхронизации данных — состояние отслеживания измененных данных (CDC).

     ![Состояние действия "Полная загрузка завершена"](media/tutorial-mysql-to-azure-mysql-online/dms-activity-full-load-completed.png)

     ![Состояние действия "Добавочная синхронизация данных"](media/tutorial-mysql-to-azure-mysql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>Выполнение прямой миграции

После завершения начальной полной загрузки базы данных помечаются как **готовые к прямой миграции**.

1. Когда вы будете готовы выполнить миграцию базы данных, щелкните **Запустить прямую миграцию**.

    ![Запуск прямой миграции](media/tutorial-mysql-to-azure-mysql-online/dms-start-cutover.png)

2. Обязательно остановите все входящие транзакции в исходную базу данных. Подождите, пока в счетчике **Ожидающие изменения** появится значение **0**.
3. Выберите **Подтвердить**, а затем — **Применить**.
4. Когда состояние миграции базы данных изменится на **Завершено**, подключите свои приложения к новой целевой базе данных SQL Azure.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об известных проблемах, ограничениях при выполнении интерактивной миграции в Базу данных Azure для MySQL и их решении см. в [этой](known-issues-azure-mysql-online.md) статье.
* См. дополнительные сведения о [службе Azure Database Migration Service](./dms-overview.md).
* Общие сведения о Базе данных Azure для MySQL см. в [этой статье](../mysql/overview.md).
