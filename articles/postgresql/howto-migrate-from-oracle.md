---
title: Миграция из Oracle
titleSuffix: Azure Database for PostgreSQL
description: В этом руководстве описывается перенос схемы Oracle в базу данных Azure для PostgreSQL.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.subservice: migration-guide
ms.topic: how-to
ms.date: 03/18/2021
ms.openlocfilehash: ec6cf87b3fd326c905b4843dc30ae6ce15379305
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609454"
---
# <a name="migrate-oracle-to-azure-database-for-postgresql"></a>Перенос Oracle в базу данных Azure для PostgreSQL

В этом руководстве описывается перенос схемы Oracle в базу данных Azure для PostgreSQL. 

Подробные и исчерпывающие инструкции по миграции см. в разделе [материалы руководства по миграции](https://github.com/microsoft/OrcasNinjaTeam/blob/master/Oracle%20to%20PostgreSQL%20Migration%20Guide/Oracle%20to%20Azure%20Database%20for%20PostgreSQL%20Migration%20Guide.pdf). 

## <a name="prerequisites"></a>Предварительные требования

Чтобы перенести схему Oracle в базу данных Azure для PostgreSQL, вам потребуется: 

- Убедитесь, что исходная среда поддерживается. 
- Скачайте последнюю версию [ora2pg](https://ora2pg.darold.net/). 
- Последняя версия [модуля ДБД](https://www.cpan.org/modules/by-module/DBD/). 


## <a name="overview"></a>Обзор

PostgreSQL — одна из самых сложных баз данных с открытым исходным кодом в мире. В этой статье описывается, как использовать бесплатную служебную программу ora2pg для переноса базы данных Oracle в PostgreSQL. Для переноса базы данных Oracle или MySQL в схему, совместимую с PostgreSQL, можно использовать ora2pg, бесплатный инструмент. Программа подключает базу данных Oracle, сканирует ее автоматически и извлекает ее структуру или данные. Затем ora2pg создает скрипты SQL, которые можно загрузить в базу данных PostgreSQL. ora2pg можно использовать для задач из реконструирования базы данных Oracle, выполнения огромного переноса базы данных предприятия или просто репликации некоторых данных Oracle в базу данных PostgreSQL. Он прост в использовании и не требует никаких знаний базы данных Oracle, Кроме возможности предоставлять параметры, необходимые для подключения к базе данных Oracle.

> [!NOTE]
> Дополнительные сведения об использовании последней версии ora2pg см. в [документации по ora2pg](https://ora2pg.darold.net/documentation.html).

### <a name="typical-ora2pg-migration-architecture"></a>Стандартная архитектура миграции ora2pg

![Снимок экрана с архитектурой миграции ora2pg.](media/howto-migrate-from-oracle/ora2pg-migration-architecture.png)

После подготовки виртуальной машины и базы данных Azure для PostgreSQL требуется две конфигурации для включения подключения между ними: **разрешите использование служб Azure** и **Принудительное подключение SSL**, как показано ниже.

- Колонка " **Безопасность подключения** " — > **Разрешить доступ к службам Azure** — >

- Колонка " **Безопасность подключения** " — > **Параметры SSL**  ->  **применение SSL-подключения** — > отключено

### <a name="recommendations"></a>Рекомендации

- Чтобы повысить производительность операций оценки или экспорта на сервере Oracle, собирайте статистику следующим образом:

   ```
   BEGIN
   
     DBMS_STATS.GATHER_SCHEMA_STATS
     DBMS_STATS.GATHER_DATABASE_STATS
     DBMS_STATS.GATHER_DICTIONARY_STATS
     END;
   ```

- Экспортируйте данные с помощью команды COPY вместо INSERT.

- Избегайте экспорта таблиц с их внешние ключи, ограничениями и индексами. это приведет к более медленному выполнению импорта данных в PostgreSQL.

- Создавайте материализованные представления с помощью **предложения No Data** и обновляйте его позже.

- Если это возможно, реализуйте уникальные индексы в материализованных представлениях, что сделает обновление быстрее с помощью синтаксиса `REFRESH MATERIALIZED VIEW CONCURRENTLY` .



## <a name="pre-migration"></a>Подготовка к миграции 

Убедившись в том, что исходная среда поддерживается и что вы устранили все необходимые условия, можно приступить к этапу перед переносом. Эта часть процесса включает в себя проведение инвентаризации баз данных, которые необходимо перенести, оценка этих баз данных на наличие потенциальных проблем миграции или блокирования, а также разрешение любых элементов, которые могли быть обнаружены. Для разнородных миграций, таких как Oracle в базу данных Azure для PostgreSQL, этот этап также включает преобразование схем в базах данных-источниках для совместимости с целевой средой.

### <a name="discover"></a>Обнаружить

Цель фазы обнаружения — определить существующие источники данных и сведения о функциях, которые используются для лучшего понимания и планирования миграции. Этот процесс включает проверку сети для обнаружения всех экземпляров Oracle организации вместе с используемой версией и функциями.

Скрипты предварительной оценки Microsoft Oracle выполняются в базе данных Oracle. Скрипты предварительной оценки — это набор запросов, который обращается к метаданным Oracle и предоставляет следующие сведения:

- Инвентаризация базы данных, включая количество объектов по схеме, типу и состоянию.

- Приблизительная оценка необработанных данных в каждой схеме (на основе статистики).

- Размер таблиц в каждой схеме.

- Число строк кода на пакет, функция, процедура и т. д.

Скачайте связанные сценарии с [веб-сайта ora2pg](http://ora2pg.darold.net/).

### <a name="assess"></a>Оценка

После завершения инвентаризации баз данных Oracle, чтобы получить представление о размере базы данных и о проблемах, следует выполнить оценку следующим шагом.

Оценка стоимости процесса миграции с Oracle на PostgreSQL непроста. Чтобы получить хорошую оценку этой стоимости миграции, ora2pg проверит все объекты базы данных, все функции и хранимые процедуры, чтобы определить, остались ли некоторые объекты и код PL/SQL, которые не могут быть автоматически преобразованы ora2pg.

ora2pg имеет режим анализа содержимого, который проверяет базу данных Oracle на создание текстового отчета о том, что содержится в базе данных Oracle и что не может быть экспортировано.

Чтобы активировать режим **анализа и отчета** , используйте экспортированный тип, `SHOW_REPORT` как показано в следующей команде:

```
ora2pg -t SHOW_REPORT
```

После анализа базы данных ora2pg с возможностью преобразования кода SQL и PL/SQL из синтаксиса Oracle в PostgreSQL, может пройти дальнейшее выполнение, оценивая проблемы с кодом и время, необходимое для выполнения полной миграции базы данных.

Чтобы оценить стоимость миграции в человек-днях, ora2pg позволяет использовать директиву конфигурации с именем ESTIMATE_COST, которая также может быть включена в командной строке:

```
ora2pg -t SHOW_REPORT --estimate_cost
```

Единица миграции по умолчанию — около пяти минут для эксперта PostgreSQL. Если это первая миграция, ее можно получить выше с помощью директивы конфигурации COST_UNIT_VALUE или параметра командной строки--cost_unit_value.

В последней строке отчета отображается общий предполагаемый код миграции в мужчина-днях после числа единиц миграции, оцениваемых для каждого объекта.

Эта единица миграции представляет около пяти минут для эксперта PostgreSQL. Если это первая миграция, можно увеличить значение по умолчанию с помощью директивы конфигурации COST_UNIT_VALUE или параметра командной строки--cost_unit_value. Найдите ниже некоторые виды оценки а) оценки таблиц. б) Оценка схемы в c) с использованием cost_unit по умолчанию (5 минут) d) оценки схемы с 10-минутной единицей стоимости.

```
ora2pg -t SHOW_TABLE -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\tables.txt ora2pg -t SHOW_COLUMN -c c:\ora2pg\ora2pg_hr.conf > c:\ts303\hr_migration\reports\columns.txt
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf --dump_as_html --estimate_cost > c:\ts303\hr
_migration\reports\report.html
ora2pg -t SHOW_REPORT -c c:\ora2pg\ora2pg_hr.conf –-cost_unit_value 10 --dump_as_html --estimate_cost > c:\ts303\hr_migration\reports\report2.html
```

Результат оценки схемы показан ниже:

**Уровень миграции: B-5**

Уровни миграции:

A — миграция, которая может выполняться автоматически

B-миграция с переписыванием кода и стоимостью в человеко-дни до 5 дней

C-миграция с переписыванием кода и стоимостью в человеко-день более 5 дней

Технические уровни:

1 = тривиальный: нет хранимых функций и триггеров

2 = простой: нет хранимых функций, но с триггерами, перезапись вручную не производится

3 = простые: хранимые функции и/или триггеры, без ручной перезаписи

4 = вручную: нет хранимых функций, но с триггерами или представлениями с перезаписью кода

5 = сложная функция: хранимые функции и триггеры с перезаписью кода

Оценка состоит из буквы (A или B), чтобы указать, требуется ли для миграции ручная перезапись, а также число от 1 до 5, указывающее на уровень технических сложностей. У вас есть дополнительный параметр-human_days_limit, чтобы указать количество человеко-дневного ограничения, в течение которого уровень миграции должен быть установлен в C, чтобы указать, что требуется огромный объем работы и полное управление проектом с поддержкой миграции. Значение по умолчанию — 10 человеко-дней. Вы можете использовать директиву конфигурации HUMAN_DAYS_LIMIT, чтобы изменить это значение по умолчанию без возможности восстановления.

Эта функция была разработана, чтобы помочь выбрать базу данных, которую можно перенести в первую очередь, а также группу, которой необходимо присвоить мобильные решения.

### <a name="convert"></a>Convert

При миграции с минимальным временем простоя переданный источник продолжит изменяться, отменяя целевой объект с точки зрения данных и схемы после однократной миграции. На этапе **синхронизации данных** необходимо убедиться, что все изменения в источнике фиксируются и применяются к целевому объекту практически в реальном времени. Убедившись, что все изменения в источнике применены к целевому объекту, можно прямую миграцию из источника в целевую среду.

На этом шаге миграции выполняется преобразование или перевод кода Oracle + ОЗНАЧАЮЩЕГО в PostgreSQL. Средство ora2pg экспортирует объекты Oracle в формате PostgreSQL автоматически. Для создаваемых объектов часть не будет компилироваться в базе данных PostgreSQL без внесения изменений вручную.  
Процесс понимания того, какие элементы требуют вмешательства вручную, состоит в компиляции файлов, созданных ora2pg, в базе данных PostgreSQL, проверке журнала и внесении необходимых изменений до того, как вся структура схемы совместима с синтаксисом PostgreSQL.


#### <a name="create-migration-template"></a>Создание шаблона миграции 

Во-первых, рекомендуется создать шаблон миграции, поставляемый с ora2pg. Два параметра — project_base и--init_project при использовании указывают, чтобы ora2pg, что он должен создать шаблон проекта с рабочим деревом, файл конфигурации и скрипт для экспорта всех объектов из базы данных Oracle. Дополнительные сведения см. в [документации по ora2pg](https://ora2pg.darold.net/documentation.html).

   Используйте следующую команду: 

   ```
   ora2pg --project_base /app/migration/ --init_project test_project
   
   ora2pg --project_base /app/migration/ --init_project test_project
   ```

Выходные данные примера: 
   
   ```
   Creating project test_project. /app/migration/test_project/ schema/ dblinks/ directories/ functions/ grants/ mviews/ packages/ partitions/ procedures/ sequences/ synonyms/    tables/ tablespaces/ triggers/ types/ views/ sources/ functions/ mviews/ packages/ partitions/ procedures/ triggers/ types/ views/ data/ config/ reports/
   
   Generating generic configuration file
   
   Creating script export_schema.sh to automate all exports.
   
   Creating script import_all.sh to automate all imports.
   ```

Sources/Directory содержит код Oracle, схема или каталог содержит код, перенесенный в PostgreSQL. Отчеты и каталог содержат отчеты в формате HTML с оценкой затрат на миграцию.


После создания структуры проекта создается универсальный файл конфигурации. Определите подключение к базе данных Oracle, а также соответствующие параметры конфигурации в файле конфигурации.  Сведения о том, что можно настроить в файле конфигурации и о том, как это сделать, см. в документации по ora2pg.


#### <a name="export-oracle-objects"></a>Экспорт объектов Oracle

Затем экспортируйте объекты Oracle как объекты PostgreSQL, запустив файл export_schema. sh.

   ```
   cd /app/migration/mig_project
   ./export_schema.sh
   
   Run the following command manually:
   
   SET namespace="/app/migration/mig_project"
   
   ora2pg -t DBLINK -p -o dblink.sql -b %namespace%/schema/dblinks -c
   %namespace%/config/ora2pg.conf
   ora2pg -t DIRECTORY -p -o directory.sql -b %namespace%/schema/directories -c
   %namespace%/config/ora2pg.conf
   ora2pg -p -t FUNCTION -o functions2.sql -b %namespace%/schema/functions -c
   %namespace%/config/ora2pg.conf ora2pg -t GRANT -o grants.sql -b %namespace%/schema/grants -c %namespace%/config/ora2pg.conf ora2pg -t MVIEW -o mview.sql -b %namespace%/schema/   mviews -c %namespace%/config/ora2pg.conf
   ora2pg -p -t PACKAGE -o packages.sql
   %namespace%/config/ora2pg.conf -b %namespace%/schema/packages -c
   ora2pg -p -t PARTITION -o partitions.sql %namespace%/config/ora2pg.conf -b %namespace%/schema/partitions -c
   ora2pg -p -t PROCEDURE -o procs.sql
   %namespace%/config/ora2pg.conf -b %namespace%/schema/procedures -c
   ora2pg -t SEQUENCE -o sequences.sql
   %namespace%/config/ora2pg.conf -b %namespace%/schema/sequences -c
   ora2pg -p -t SYNONYM -o synonym.sql -b %namespace%/schema/synonyms -c
   %namespace%/config/ora2pg.conf
   ora2pg -t TABLE -o table.sql -b %namespace%/schema/tables -c %namespace%/config/ora2pg.conf ora2pg -t TABLESPACE -o tablespaces.sql -b %namespace%/schema/tablespaces -c
   %namespace%/config/ora2pg.conf
   ora2pg -p -t TRIGGER -o triggers.sql -b %namespace%/schema/triggers -c
   %namespace%/config/ora2pg.conf ora2pg -p -t TYPE -o types.sql -b %namespace%/schema/types -c %namespace%/config/ora2pg.conf ora2pg -p -t VIEW -o views.sql -b %namespace%/   schema/views -c %namespace%/config/ora2pg.conf
   ```

   Чтобы извлечь данные, используйте следующую команду:

   ```
   ora2pg -t COPY -o data.sql -b %namespace/data -c %namespace/config/ora2pg.conf
   ```

#### <a name="compile-files"></a>Компиляция файлов

Наконец, Скомпилируйте все файлы на сервере базы данных Azure для PostgreSQL. Теперь можно загрузить файлы DDL, созданные вручную, или использовать второй скрипт import_all. sh, чтобы импортировать эти файлы в интерактивном режиме.

   ```
   psql -f %namespace%\schema\sequences\sequence.sql -h server1-
   
   server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -l
   
   %namespace%\ schema\sequences\create_sequences.log
   
   psql -f %namespace%\schema\tables\table.sql -h server1-server.postgres.database.azure.com p 5432 -U username@server1-server -d database -l    %namespace%\schema\tables\create_table.log
   ```

   Команда импорта данных:

   ```
   psql -f %namespace%\data\table1.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -l %namespace%\data\table1.log
   
   psql -f %namespace%\data\table2.sql -h server1-server.postgres.database.azure.com -p 5432 -U username@server1-server -d database -l %namespace%\data\table2.log
   ```

Во время компиляции файлов проверьте журналы и исправьте необходимые синтаксисы, которые ora2pg не удалось преобразовать из поля.

Дополнительные сведения о решении проблем см. в техническом документе [Oracle to Azure Database PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20Database%20for%20PostgreSQL%20Migration%20Workarounds.pdf) reworks.

## <a name="migrate"></a>Миграция 

После выполнения необходимых условий и выполнения задач, связанных с стадией **перед миграцией** , можно приступать к миграции схемы и данных.

### <a name="migrate-schema-and-data"></a>Перенос схемы и данных

После устранения этих ошибок стабильная сборка базы данных готова к развертыванию.

На этом этапе все, что требуется, — это выполнить команды импорта *psql* , указывающие на файлы, содержащие измененный код, для компиляции объектов базы данных в базе данных PostgreSQL и импорта данных.

На этом шаге можно реализовать некоторый уровень параллелизма при импорте данных.

### <a name="data-sync-and-cutover"></a>Синхронизация данных и прямую миграцию

При переходе в оперативный режим (с минимальным временем простоя) переданный источник продолжит изменяться, отменяя целевой объект с точки зрения данных и схемы после однократной миграции. На этапе **синхронизации данных** необходимо убедиться, что все изменения в источнике фиксируются и применяются к целевому объекту практически в реальном времени. Убедившись, что все изменения в источнике применены к целевому объекту, можно прямую миграцию из источника в целевую среду.

По состоянию на март 2019, если вы хотите выполнить оперативную миграцию, рассмотрите возможность использования Attunity replicate для миграции Майкрософт или Стриим.

Для *разностной или добавочной* миграции с использованием ora2pg методика состоит в применении для каждой таблицы. запрос, который применяет фильтр (вырезание) по дате или времени и т. д., и после этого завершает перенос, применяя второй запрос, который перенесет остальные данные (оставшиеся).

В таблице исходных данных сначала перенесите все исторические данные. Пример:

```
select * from table1 where filter_data < 01/01/2019
```

Вы можете запросить изменения, внесенные после первоначальной миграции, выполнив команду, аналогичную следующей:

```
select * from table1 where filter_data >= 01/01/2019
```

В этом случае рекомендуется улучшить проверку, проверив четность данных на обеих сторонах, источнике и целевом объекте.

## <a name="post-migration"></a>После миграции 

После успешного завершения этапа **миграции** необходимо выполнить ряд задач, выполняемых после миграции, чтобы обеспечить бесперебойную и эффективную работу всех компонентов.

### <a name="remediate-applications"></a>Исправление приложений

После переноса данных в целевую среду все приложения, которые раньше использовали источник, должны приступить к приему целевого объекта. Для этого в некоторых случаях потребуется внести изменения в приложения.

### <a name="perform-tests"></a>Выполнение тестов

После переноса данных в целевой объект выполните тесты для баз данных, чтобы убедиться, что приложения хорошо работают с целевым объектом после миграции.

Чтобы гарантировать правильную миграцию исходного и целевого объектов, выполните скрипты проверки данных вручную в отношении источника Oracle и целевых баз данных PostgreSQL.

В идеале, если исходная и целевая базы данных имеют сетевой путь, для проверки данных следует использовать ora2pg. С помощью действия теста можно проверить, были ли созданы все объекты базы данных Oracle в разделе PostgreSQL. Выполните команду, как показано ниже.

```
ora2pg -t TEST -c config/ora2pg.conf > migration_diff.txt
```

### <a name="optimize"></a>Оптимизация

Этап, выполняемый после миграции, крайне важен для согласования любых проблем с точностью данных и проверки полноты, а также для устранения проблем с производительностью рабочей нагрузки.

## <a name="migration-assets"></a>Ресурсы, посвященные миграции 

Дополнительную помощь по выполнению этого сценария миграции см. в следующих ресурсах, которые были разработаны с учетом поддержки реальных проектов миграции.

| **Ссылка на заголовок** | **Описание**    |
| -------------- | ------------------ |
| [Oracle to Azure Database for PostgreSQL Migration Cookbook](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20PostgreSQL%20Migration%20Cookbook.pdf) (Руководство по миграции Oracle в Базу данных Azure для PostgreSQL) | Эта цель документа — предоставить архитекторам, консультантам, администраторам баз данных и связанным ролям рекомендации по быстрому переносу рабочих нагрузок из Oracle в службу "база данных Azure для PostgreSQL" с помощью средства ora2pg. |
| [Решения для миграции с Oracle на Azure PostgreSQL](https://github.com/Microsoft/DataMigrationTeam/blob/master/Whitepapers/Oracle%20to%20Azure%20Database%20for%20PostgreSQL%20Migration%20Workarounds.pdf) | Эта цель документа — предоставить архитекторам, консультантам, администраторам баз данных и связанным ролям руководство по быстрому исправлению и обработку проблем при переносе рабочих нагрузок из Oracle в службу "база данных Azure для PostgreSQL". |
| [Инструкции по установке ora2pg в Windows или Linux](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Steps%20to%20Install%20ora2pg%20on%20Windows%20and%20Linux.pdf)                       | Этот документ предназначен для быстрой установки с целью включения миграции схемы & данных из Oracle в базу данных Azure для PostgreSQL с помощью средства ora2pg в Windows или Linux. Подробные сведения об этом инструменте можно найти по адресу http://ora2pg.darold.net/documentation.html . |

Эти ресурсы были разработаны в рамках программы Data SQL Ninja, которая спонсируется группой разработчиков Azure Data Group. Основная часть программы Data SQL Ninja заключается в разрешении и ускорении сложной модернизации и реализация перехода на платформу данных Microsoft Azure. Если вы считаете, что ваша организация заинтересована в участии в программе Data SQL Ninja, обратитесь в службу поддержки своей учетной записи и попросите отправить заявку.


### <a name="contact-support"></a>Обращение в службу поддержки

Если вам нужна помощь с миграцией, помимо средств ora2pg, обратитесь к псевдониму [ @Ask Azure DB для PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com) , чтобы получить сведения о других вариантах миграции.

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы получить доступ к матрице Microsoft и сторонним службам и средствам, доступным для различных сценариев переноса баз данных и данных (и специальных задач), см. статью [служба и средства для переноса данных](https://docs.microsoft.com/azure/dms/dms-tools-matrix).

Дополнительные сведения см. на следующих ресурсах: 
- [Документация по Базе данных Azure для PostgreSQL](https://docs.microsoft.com/azure/postgresql/)
- [Документация по ora2pg](https://ora2pg.darold.net/documentation.html)
- [Веб-сайт PostgreSQL](https://www.postgresql.org/)
- [Поддержка автономных транзакций в PostgreSQL](http://blog.dalibo.com/2016/08/19/Autonoumous_transactions_support_in_PostgreSQL.html) 

Для видеоматериала: 
- [Общие сведения о процессе миграции и средствах и службах, рекомендованных для выполнения оценки и миграции](https://azure.microsoft.com/resources/videos/overview-of-migration-and-recommended-tools-services/).