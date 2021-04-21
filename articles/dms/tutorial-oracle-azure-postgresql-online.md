---
title: Руководство по Перенос Oracle по сети в Базу данных Azure для PostgreSQL
titleSuffix: Azure Database Migration Service
description: Из этой статьи вы узнаете, как выполнить сетевой перенос из Oracle в локальной среде или на виртуальной машине в Базу данных Azure для PostgreSQL с помощью Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/24/2020
ms.openlocfilehash: 76ab2204a81d3cbfb559bfa4591f1f332b1707ee
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107388080"
---
# <a name="tutorial-migrate-oracle-to-azure-database-for-postgresql-online-using-dms-preview"></a>Руководство по Сетевой перенос Oracle в Базу данных Azure для PostgreSQL с помощью DMS (предварительная версия)

> [!IMPORTANT]
> Сценарий **Миграции с Oracle на Базу данных Azure для PostgreSQL** (сейчас на этапе предварительной версии) станет недоступен после 1 мая 2021 г. Мы продолжим предоставлять поддержку через альтернативные средства (например, Ora2pg) и обеспечивать оптимальный способ миграции с Oracle на PostgreSQL. Рекомендации по миграции см. в [руководстве по миграции с Oracle на Базу данных Azure для PostgreSQL](https://aka.ms/OracletoPGguide). 

Службу Azure Database Migration Service можно использовать для переноса баз данных Oracle из локальной среды или виртуальных машин в [Базу данных Azure для PostgreSQL](../postgresql/index.yml) с минимальным временем простоя. Другими словами, миграцию можно завершить с минимальным временем простоя для приложения. В этом руководстве выполняется перенос примера базы данных **отдела кадров** из экземпляра Oracle 11g, размещенного в локальной среде или на виртуальной машине, в Базу данных Azure для PostgreSQL с помощью операции сетевого переноса в Azure Database Migration Service.

В этом руководстве описано следующее:
> [!div class="checklist"]
>
> * оценка мероприятий по миграции с помощью средства ora2pg;
> * перенос примера схемы с помощью средства ora2pg;
> * Создайте экземпляр Azure Database Migration Service.
> * создание проекта миграции с использованием Azure Database Migration Service.
> * выполнение миграции.
> * Мониторинг миграции.

> [!NOTE]
> Чтобы выполнить сетевую миграцию с помощью Azure Database Migration Service, требуется создать экземпляр ценовой категории "Премиум".

> [!IMPORTANT]
> Чтобы процесс миграции был выполнен без проблем, Майкрософт рекомендует создать экземпляр Azure Database Migration Service в том же регионе Azure, в котором размещена целевая база данных. Перемещение данных между регионами и географическими областями может замедлить процесс миграции и привести к ошибкам.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

В этой статье описывается выполнение сетевого переноса из Oracle в Базу данных Azure для PostgreSQL.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

* Скачайте и установите [Oracle 11g с выпуском 2 (Standard Edition, Standard Edition One или Enterprise Edition)](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html).
* Скачайте базу данных **отдела кадров** из [этого примера](https://docs.oracle.com/database/121/COMSC/installation.htm#COMSC00002).
* Скачайте и [установите ora2pg в системе Windows или Linux](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf).
* [Создайте экземпляр в Базе данных Azure для PostgreSQL](../postgresql/quickstart-create-server-database-portal.md).
* Подключитесь к этому экземпляру и создайте базу данных, следуя инструкциям из этого [документа](../postgresql/tutorial-design-database-using-azure-portal.md).
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
* Откройте брандмауэр Windows, чтобы предоставить Azure Database Migration Service доступ к исходному серверу Oracle. По умолчанию это TCP-порт 1521.
* Если перед исходными базами данных развернуто устройство брандмауэра, вам может понадобиться добавить правила брандмауэра, чтобы позволить службе Azure Database Migration Service обращаться к исходным базам данных для выполнения миграции.
* Создайте [правило брандмауэра](../azure-sql/database/firewall-configure.md) на уровне сервера для Базы данных Azure для PostgreSQL, чтобы предоставить службе Azure Database Migration Service доступ к целевым базам данных. Задайте диапазон подсети в виртуальной сети, которая используется для Azure Database Migration Service.
* Разрешите доступ к исходным базам данных Oracle.

  > [!NOTE]
  > Для подключения пользователя к источнику данных Oracle требуется роль DBA.

  * Журналы повторной архивации нужны для добавочной синхронизации измененных данных в Azure Database Migration Service. Чтобы настроить источник Oracle, выполните следующие действия:
    * Войдите в систему с правами SYSDBA, выполнив следующую команду:

      ```
      sqlplus (user)/(password) as sysdba
      ```

    * Завершите работу экземпляра базы данных, выполнив следующую команду:

      ```
      SHUTDOWN IMMEDIATE;
      ```

      Дождитесь подтверждения `'ORACLE instance shut down'`.

    * Запустите новый экземпляр и подключите базу данных (но не открывайте ее), чтобы включить или отключить архивирование, выполнив следующую команду:

      ```
      STARTUP MOUNT;
      ```

      База данных должна быть подключена. Дождитесь подтверждения "Запущен экземпляр Oracle".

    * Измените режим архивации базы данных, выполнив следующую команду:

      ```
      ALTER DATABASE ARCHIVELOG;
      ```

    * Откройте базу данных для обычной работы, выполнив следующую команду:

      ```
      ALTER DATABASE OPEN;
      ```

      Возможно, потребуется перезапустить систему, чтобы отобразился файл ARC.

    * Чтобы проверить, так ли это, выполните следующую команду:

      ```
      SELECT log_mode FROM v$database;
      ```

      Вы должны получить ответ `'ARCHIVELOG'`. Если получен ответ `'NOARCHIVELOG'`, значит требование не соблюдается.

  * Включите дополнительное ведение журнала для репликации, выполнив любую из следующих процедур.

    * **Вариант 1**.
      Измените уровень дополнительного ведения журнала для базы данных, чтобы охватить все таблицы с первичными ключами и уникальными индексами. Запрос обнаружения будет возвращать ответ `'IMPLICIT'`.

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY, UNIQUE) COLUMNS;
      ```

      Измените уровень дополнительного ведения журнала на уровне таблиц. Выполняйте его только для таблиц, в которых изменяются данные, но нет первичных или уникальных индексов.

      ```
      ALTER TABLE [TABLENAME] ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    * **Вариант 2**.
      Измените уровень дополнительного ведения журнала для базы данных, чтобы охватить все таблицы, и запрос обнаружения возвратит ответ `'YES'`.

      ```
      ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
      ```

      Измените уровень дополнительного ведения журнала на уровне таблиц. Выполните действия по описанной ниже логике, чтобы запустить только одну инструкцию для каждой таблицы.

      Если таблица имеет первичный ключ:

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
      ```

      Если таблица имеет уникальный индекс:

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG GROUP (first unique index columns) ALWAYS;
      ```

      Для остальных таблиц используйте следующую команду:

      ```
      ALTER TABLE xxx ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
      ```

    Чтобы проверить, так ли это, выполните следующую команду:

      ```
      SELECT supplemental_log_data_min FROM v$database;
      ```

    Вы должны получить ответ `'YES'`.

## <a name="assess-the-effort-for-an-oracle-to-azure-database-for-postgresql-migration"></a>Оценка мероприятий для перехода с Oracle на Базу данных Azure для PostgreSQL

Мы рекомендуем использовать ora2pg для оценки требуемых мероприятий с целью перехода с Oracle на Базу данных Azure для PostgreSQL. С помощью директивы `ora2pg -t SHOW_REPORT` создайте отчет, перечисляющий все объекты Oracle с оценкой стоимости миграции (в днях работы разработчика) и некоторые объекты базы данных, для которых потребуется особое внимание в процессе миграции.

Большинство клиентов тратят немало времени на изучение отчета об оценке и планирование мероприятий по миграции в автоматическом и ручном режиме.

Чтобы настроить и запустить ora2pg для подготовки отчета об оценке, воспользуйтесь разделом **Premigration: Assessment** (Перед миграцией: оценка) в [учебнике по переносу Oracle в Базу данных Azure для PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf). Пример создаваемого ora2pg отчета об оценке можно получить [здесь](https://ora2pg.darold.net/report.html).

## <a name="export-the-oracle-schema"></a>Экспорт схемы Oracle

Мы рекомендуем использовать ora2pg для преобразования схемы Oracle и других объектов Oracle (типы, процедуры, функции, и т. д.) в схему, совместимую с Базой данных Azure для PostgreSQL. ora2pg предоставляет множество директив для предварительного определения некоторых типов данных. Например, вы можете использовать директиву `DATA_TYPE`, чтобы заменить все NUMBER(*,0) на тип bigint вместо NUMERIC(38).

С помощью ora2pg можно выполнить экспорт всех объектов базы данных в формат SQL-файлов. Вы сможете просмотреть SQL-файлы, а затем импортировать их в Базу данных Azure для PostgreSQL с помощью psql или выполнить скрипт .sql в PgAdmin.

```
psql -f [FILENAME] -h [AzurePostgreConnection] -p 5432 -U [AzurePostgreUser] -d database 
```

Пример:

```
psql -f %namespace%\schema\sequences\sequence.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database
```

Чтобы настроить и запустить ora2pg для преобразования схемы, воспользуйтесь разделом **Migration: Schema and data** (Миграция: схема и данные) в [учебнике по переносу Oracle в Базу данных Azure для PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf).

## <a name="set-up-the-schema-in-azure-database-for-postgresql"></a>Настройка схемы в Базе данных Azure для PostgreSQL

Схемы таблиц Oracle, хранимые процедуры, пакеты и другие объекты базы данных можно преобразовать, чтобы сделать их совместимыми c Postgres, используя ora2pg перед запуском конвейера миграции в Azure Database Migration Service. Ниже приведены ссылки на сведения о работе с ora2pg:

* [Step-by-Step Guide to Install ora2pg on Windows](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf) (Пошаговое руководство по установке ora2pg в Windows)
* [Oracle to Azure Database for PostgreSQL Migration Cookbook](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf) (Руководство по миграции Oracle в Базу данных Azure для PostgreSQL)

Azure Database Migration Service также может создать схему таблицы PostgreSQL. Служба обращается к схеме таблицы в подключенном источнике Oracle и создает совместимую схему таблицы в Базе данных Azure для PostgreSQL. Не забудьте проверить формат схемы в Базе данных Azure для PostgreSQL после того, как Azure Database Migration Service завершит создание схемы и перемещение данных.

> [!IMPORTANT]
> Azure Database Migration Service создает только схему таблицы; другие объекты базы данных, такие как хранимые процедуры, пакеты, индексы и т. д., не создаются.

Удалите внешний ключ в целевой базе данных, чтобы выполнить полную загрузку данных. В разделе **Перенос примера схемы**[этой статьи](./tutorial-postgresql-azure-postgresql-online.md) приводится скрипт, который можно использовать для удаления внешнего ключа. Примените Azure Database Migration Service для запуска полной загрузки и синхронизации.

### <a name="when-the-postgresql-table-schema-already-exists"></a>Если схема таблицы PostgreSQL уже существует

При создании схемы PostgreSQL с помощью таких средств, как ora2pg, перед началом перемещения данных с помощью Azure Database Migration Service сопоставьте исходные таблицы с целевыми в Azure Database Migration Service.

1. При создании проекта миграции Oracle в Базу данных Azure для PostgreSQL вам будет предложено выбрать целевую базу данных и целевую схему на шаге выбора схем. Заполните целевую базу данных и целевую схему.

   ![Снимок экрана: сопоставление с целевыми базами данных.](media/tutorial-oracle-azure-postgresql-online/dms-map-to-target-databases.png)

2. На экране **Параметры миграции** представлен список таблиц в источнике Oracle. Azure Database Migration Service пытается сопоставить таблицы в исходной и целевой таблицах на основе имени таблицы. Если существует несколько совпадающих целевых таблиц с различными регистрами, можно выбрать целевую таблицу для сопоставления.

    ![Снимок экрана: параметры миграции.](media/tutorial-oracle-azure-postgresql-online/dms-migration-settings.png)

> [!NOTE]
> Если необходимо сопоставлять имена исходных таблиц с таблицами с разными именами, напишите по адресу [dmsfeedback@microsoft.com](mailto:dmsfeedbac@microsoft.com) и мы можем предоставить скрипт для автоматизации процесса.

### <a name="when-the-postgresql-table-schema-doesnt-exist"></a>Если схема таблицы PostgreSQL не существует

Если целевая база данных PostgreSQL не содержит никаких сведений о схеме таблицы, Azure Database Migration Service преобразует исходную схему и воссоздает ее в целевой базе данных. Помните, Azure Database Migration Service создает только схему таблицы, а не другие объекты базы данных, такие как хранимые процедуры, пакеты и индексы.
Чтобы служба Azure Database Migration Service создала схему, убедитесь, что целевая среда включает схему без таблиц. Если Azure Database Migration Service обнаруживает любую таблицу, служба предполагает, что схема была создана внешним средством, например ora2pg.

> [!IMPORTANT]
> Azure Database Migration Service требует, чтобы все таблицы создавались одинаково, используя Azure Database Migration Service или такое средство, как ora2pg, но не оба.

Чтобы начать работу:

1. Создайте схему в целевой базе данных в соответствии с требованиями приложения. По умолчанию имена схемы и столбцов таблицы PostgreSQL написаны в нижнем регистре. С другой стороны, названия схемы и столбцов таблицы Oracle по умолчанию записаны в верхнем регистре.
2. На шаге выбора схем укажите целевую базу данных и целевую схему.
3. На основе схемы, создаваемой в Базе данных Azure для PostgreSQL, Azure Database Migration Service использует следующие правила преобразования:

    Если имя схемы в источнике Oracle совпадает с именем в Базе данных Azure для PostgreSQL, то Azure Database Migration Service *создает схему таблицы, используя тот же регистр, что и в целевом объекте*.

    Пример:

    | Исходная схема Oracle | Целевая PostgreSQL Database.Schema | schema.table.column, созданная DMS |
    | ------------- | ------------- | ------------- |
    | HR | targetHR.public | public.countries.country_id |
    | HR | targetHR.trgthr | trgthr.countries.country_id |
    | HR | targetHR.TARGETHR | "TARGETHR"."COUNTRIES"."COUNTRY_ID" |
    | HR | targetHR.HR | "HR"."COUNTRIES"."COUNTRY_ID" |
    | HR | targetHR.Hr | *Невозможно сопоставить смешанные регистры |

    *Чтобы создать схему со смешанными регистрами и имена таблиц в целевом PostgreSQL, свяжитесь с нами по адресу [dmsfeedback@microsoft.com](mailto:dmsfeedback@microsoft.com). Мы можем предоставить скрипт для настройки схемы таблицы со смешанными регистрами в целевой базе данных PostgreSQL.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Регистрация поставщика ресурсов Microsoft.DataMigration

1. Войдите на портал Azure, щелкните **Все службы** и выберите **Подписки**.

   ![Отображение подписок на портале](media/tutorial-oracle-azure-postgresql-online/portal-select-subscriptions.png)

2. Выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Отображение поставщиков ресурсов](media/tutorial-oracle-azure-postgresql-online/portal-select-resource-provider.png)

3. В поле поиска введите migration, а затем справа от **Microsoft.DataMigration** щелкните **Зарегистрировать**.

    ![Регистрация поставщика ресурсов](media/tutorial-oracle-azure-postgresql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Создание экземпляра DMS

1. На портале Azure выберите **+Создать ресурс**, введите в поле поиска "Azure Database Migration Service", а затем в раскрывающемся списке выберите **Azure Database Migration Service**.

    ![Azure Marketplace](media/tutorial-oracle-azure-postgresql-online/portal-marketplace.png)

2. На экране **Azure Database Migration Service** выберите **Создать**.

    ![Создание экземпляра Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-create2.png)
  
3. На экране **Создание службы миграции** укажите имя службы, подписку и новую или существующую группу ресурсов.

4. Выберите существующую виртуальную сеть или создайте новую.

    Виртуальная сеть предоставляет Azure Database Migration Service доступ к исходному экземпляру Oracle и целевому экземпляру Базы данных Azure для PostgreSQL.

    См. статью [Краткое руководство. Создание виртуальной сети с помощью портала Azure](../virtual-network/quick-create-portal.md).

5. Выберите ценовую категорию.

    Дополнительные сведения о ценовых категориях и затратах см. на [странице с описанием цен](https://aka.ms/dms-pricing).

    ![Настройка параметров экземпляра Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-settings5.png)

6. Выберите **Создать**, чтобы создать службу.

## <a name="create-a-migration-project"></a>Создание проекта миграции

После создания службы найдите ее на портале Azure, откройте и создайте проект миграции.

1. На портале Azure щелкните **Все службы**, выполните поиск по запросу "Azure Database Migration Service" и выберите **Azure Database Migration Services** (Службы Azure Database Migration Service).

    ![Поиск всех экземпляров Azure Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-search.png)

2. На экране **Azure Database Migration Services** найдите имя созданного экземпляра Azure Database Migration Service и выберите его.

    ![Поиск экземпляра службы миграции базы данных Azure](media/tutorial-oracle-azure-postgresql-online/dms-instance-search.png)

3. Выберите **+ Новый проект миграции**.
4. На экране **Новый проект миграции** задайте имя проекта, в текстовом поле **Тип исходного сервера** выберите **Oracle**, а в текстовом поле **Тип целевого сервера** — **База данных Azure для PostgreSQL**.
5. В разделе **Выберите тип действия** выберите **Миграция данных по сети**.

   ![Создание проекта Database Migration Service](media/tutorial-oracle-azure-postgresql-online/dms-create-project5.png)

   > [!NOTE]
   > Кроме того, вы можете выбрать **Создать только проект**, чтобы создать проект миграции для отложенного выполнения.

6. Выберите **Сохранить**, изучите требования для успешного использования Azure Database Migration Service для сетевой миграции, а затем щелкните **Создать и выполнить действие**.

## <a name="specify-source-details"></a>Указание сведений об источнике

* В окне **Добавление сведений об источнике** укажите сведения о подключении для исходного экземпляра Oracle.

  ![Экран добавления сведений об источнике](media/tutorial-oracle-azure-postgresql-online/dms-add-source-details.png)

## <a name="upload-oracle-oci-driver"></a>Загрузка драйвера Oracle OCI

1. Выберите **Сохранить**, а затем на экране **Установка драйвера OCI** войдите в учетную запись Oracle и загрузите драйвер **instantclient-basiclite-windows.x64-12.2.0.1.0.zip**(размер: 37 128 586 байт; контрольная сумма SHA1: 865082268), который можно найти [здесь](https://www.oracle.com/technetwork/topics/winx64soft-089540.html#ic_winx64_inst).
2. Поместите драйвер в общую папку.

   Убедитесь, что общий доступ к этой папке предоставляется от имени пользователя с минимальным доступом только для чтения. Azure Database Migration Service обращается к этой папке и считывает драйвер OCI для загрузки в Azure, олицетворяя пользователя с указанным именем.

   Это обязательно должна быть учетная запись пользователя Windows.

   ![Установка драйвера OCI](media/tutorial-oracle-azure-postgresql-online/dms-oci-driver-install.png)

## <a name="specify-target-details"></a>Указание сведений о цели

1. Нажмите кнопку **Сохранить**, а затем на экране **данных целевого объекта** укажите сведения о подключении для целевого сервера Базы данных Azure для PostgreSQL, то есть предварительно подготовленного экземпляра Базы данных Azure для PostgreSQL, для которого вы развернули схему **HR**.

    ![Экран сведений о целевом объекте](media/tutorial-oracle-azure-postgresql-online/dms-add-target-details1.png)

2. Нажмите кнопку **Сохранить**, а затем на экране **Сопоставление с целевыми базами данных** сопоставьте исходную и целевую базы данных для миграции.

    Если в целевой базе данных содержится база данных с тем же именем, что у исходной базы данных, Azure Database Migration Service по умолчанию выберет целевую базу данных.

    ![Сопоставление с целевыми базами данных](media/tutorial-oracle-azure-postgresql-online/dms-map-target-details.png)

3. Нажмите кнопку **Сохранить** в окне **Migration summary** (Сводка по миграции) и в поле **Имя действия** введите имя действия миграции, а затем проверьте соответствие сведениям о целевом и исходном сервере, которые вы указали ранее.

    ![Сводка по миграции](media/tutorial-oracle-azure-postgresql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Выполнение миграции

* Выберите **Запустить миграцию**.

  Появится окно действия миграции и в поле **Состояние** будет указано **Инициализация**.

## <a name="monitor-the-migration"></a>Мониторинг миграции

1. На экране действия миграции нажимайте **Обновить**, чтобы периодически обновлять содержимое экрана, пока в поле **Состояние** не будет указано **Выполнение**.

     ![Состояние действия "Выполняется"](media/tutorial-oracle-azure-postgresql-online/dms-activity-running.png)

2. В столбце **Имя базы данных** выберите базу данных, чтобы получить состояние миграции для операций **полной загрузки данных** и **добавочной синхронизации данных**.

    В разделе полной загрузки данных отображается состояние начальной загрузки, а в разделе добавочной синхронизации данных — состояние отслеживания измененных данных (CDC).

     ![Состояние действия "Полная загрузка завершена"](media/tutorial-oracle-azure-postgresql-online/dms-activity-full-load-completed.png)

     ![Состояние действия "Добавочная синхронизация данных"](media/tutorial-oracle-azure-postgresql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>Выполнение прямой миграции

После завершения начальной полной загрузки базы данных помечаются как **готовые к прямой миграции**.

1. Когда вы будете готовы выполнить миграцию базы данных, щелкните **Запустить прямую миграцию**.

2. Обязательно остановите все входящие транзакции в исходную базу данных. Подождите, пока в счетчике **Ожидающие изменения** появится значение **0**.

   ![Запуск прямой миграции](media/tutorial-oracle-azure-postgresql-online/dms-start-cutover.png)

3. Щелкните **Подтвердить**, а затем — **Apply** (Применить).
4. Когда состояние переноса базы данных изменится на **Завершено**, подключите свои приложения к новой целевой Базе данных Azure для PostgreSQL.

 > [!NOTE]
 > Так как PostgreSQL по умолчанию хранит schema.table.column в нижнем регистре, преобразуйте имена из верхнего в нижний регистр с помощью скрипта из раздела **Настройка схемы в Базе данных Azure для PostgreSQL** выше в этой статье.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об известных проблемах, ограничениях при выполнении сетевой миграции в Базу данных Azure для PostgreSQL см. в [этой](known-issues-azure-postgresql-online.md) статье.
* См. дополнительные сведения о [службе Azure Database Migration Service](./dms-overview.md).
* Общие сведения о Базе данных Azure для PostgreSQL см. в статье [Что такое база данных Azure для PostgreSQL](../postgresql/overview.md).
