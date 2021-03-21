---
title: Функции и возможности SQL Управляемый экземпляр с поддержкой Arc
description: Функции и возможности SQL Управляемый экземпляр с поддержкой Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 9c42acb69e13cc1eb0fbba3fcafaec1451bc4d77
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97589225"
---
# <a name="features-and-capabilities-of-azure-arc-enabled-sql-managed-instance"></a>Функции и возможности SQL Управляемый экземпляр с поддержкой Arc

Служба "Дуга Azure" с поддержкой SQL Управляемый экземпляр совместно использовать общую базу кода с последней стабильной версией SQL Server. Большинство стандартных функций языка SQL, обработки запросов и управления базами данных идентичны. Ниже перечислены наиболее распространенные функции SQL Server и базы данных SQL, а также Управляемый экземпляр SQL.

- Языковые функции — [Управление ключевыми словами языка потока](/sql/t-sql/language-elements/control-of-flow), [курсорами](/sql/t-sql/language-elements/cursors-transact-sql), [типами данных](/sql/t-sql/data-types/data-types-transact-sql), [инструкциями DML](/sql/t-sql/queries/queries), [предикатами](/sql/t-sql/queries/predicates), [порядковыми номерами](/sql/relational-databases/sequence-numbers/sequence-numbers), [хранимыми процедурами](/sql/relational-databases/stored-procedures/stored-procedures-database-engine)и [переменными](/sql/t-sql/language-elements/variables-transact-sql).
- Функции базы данных — [Автоматическая настройка (форсирование плана)](/sql/relational-databases/automatic-tuning/automatic-tuning), [Отслеживание изменений](/sql/relational-databases/track-changes/about-change-tracking-sql-server), параметры [сортировки базы данных](/sql/relational-databases/collations/set-or-change-the-database-collation), [автономные базы](/sql/relational-databases/databases/contained-databases)данных, [автономные пользователи](/sql/relational-databases/security/contained-database-users-making-your-database-portable), [Сжатие данных](/sql/relational-databases/data-compression/data-compression), [настройки конфигурации базы данных](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql), [операции с индексами в сети](/sql/relational-databases/indexes/perform-index-operations-online), [секционирование](/sql/relational-databases/partitions/partitioned-tables-and-indexes)и [временные таблицы](/sql/relational-databases/tables/temporal-tables) ([см. Руководство по началу работы](/sql/relational-databases/tables/getting-started-with-system-versioned-temporal-tables)).
- Функции безопасности — [роли приложений](/sql/relational-databases/security/authentication-access/application-roles), [Динамическое маскирование данных](/sql/relational-databases/security/dynamic-data-masking) ([Начало работы с динамическим маскированием данных в базе данных SQL с помощью портал Azure](../../azure-sql/database/dynamic-data-masking-configure-portal.md)) [безопасность на уровне строк](/sql/relational-databases/security/row-level-security)
- Возможности с несколькими моделями — [Обработка графов](/sql/relational-databases/graphs/sql-graph-overview), [данные JSON](/sql/relational-databases/json/json-data-sql-server), [OPENXML](/sql/t-sql/functions/openxml-transact-sql), [пространственные](/sql/relational-databases/spatial/spatial-data-sql-server), [OPENJSON](/sql/t-sql/functions/openjson-transact-sql)и [XML-индексы](/sql/t-sql/statements/create-xml-index-transact-sql).



[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="features-of-azure-arc-enabled-sql-managed-instance"></a>Возможности SQL Управляемый экземпляр с поддержкой ARC в Azure

###  <a name="rdbms-high-availability"></a><a name="RDBMSHA"></a> RDBMS High Availability  
  
|Компонент|Управляемый экземпляр SQL с поддержкой Azure Arc|
|-------------|----------------|
|доставка журналов;|Да| 
|Сжатие резервных копий|Да|
|Моментальный снимок базы данных|Да|
|Экземпляр отказоустойчивого кластера Always On <sup>1</sup>| Неприменимо. Аналогичные возможности доступны |
|Группы доступности Always On <sup>2</sup>|Возможности высокой доступности планируются.|
|Базовые группы доступности <sup>2</sup>|Возможности высокой доступности планируются.|
|Минимальная группа доступности фиксации реплики <sup>2</sup>|Возможности высокой доступности планируются.|
|Группа доступности без кластеров|Да|
|Восстановление страниц и файлов в режиме «в сети»|Да|
|Индексирование в сети|Да|
|Возобновляемая перестройка индексов в подключенном режиме|Да|
|Изменение схемы в режиме «в сети»|Да|
|Быстрое восстановление|Да|
|Зеркальные резервные копии|Да|
|Поддержка памяти и ЦП с "горячей" заменой|Да|
|Зашифрованная резервная копия|Да|
|Гибридное резервное копирование в Azure (резервное копирование по URL-адресу)|Да|

<sup>1</sup> в сценарии, в котором произошел сбой Pod, новый управляемый экземпляр SQL запустится и повторно подключится к постоянному тому, содержащему ваши данные. Дополнительные [сведения о постоянных томах Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes)см. здесь.

<sup>2</sup> в будущих выпусках будут предоставлены возможности группы доступности 

###  <a name="rdbms-scalability-and-performance"></a><a name="RDBMSSP"></a> RDBMS Scalability and Performance  

| Компонент | Управляемый экземпляр SQL с поддержкой Azure Arc |
|--|--|
| columnstore | Да |
| Большие двоичные объекты в кластеризованных индексах columnstore | Да |
| Перестройка некластеризованных индексов columnstore в подключенном режиме | Да |
| Выполняющаяся в памяти OLTP | Да |
| Постоянная основная память | Да |
| Секционирование таблиц и индексов | Да |
| Сжатие данных | Да |
| Resource Governor | Да |
| Параллелизм секционированных таблиц | Да |
| Поддержка NUMA, выделение памяти больших страниц и массива буфера | Да |
| Управление ресурсами ввода-вывода | Да |
| Отложенная устойчивость | Да |
| Автоматическая настройка | Да |
| Адаптивные соединения в пакетном режиме | Да |
| Обратная связь по временно предоставляемому буферу памяти в пакетном режиме | Да |
| Выполнение с чередованием для функций с табличным значением с несколькими инструкциями | Да |
| Улучшения массовой вставки | Да |

###  <a name="rdbms-security"></a><a name="RDBMSS"></a> RDBMS Security

| Компонент | Управляемый экземпляр SQL с поддержкой Azure Arc |
|--|--|
| Безопасность на уровне строк | Да |
| Always Encrypted | Да |
| Always Encrypted с безопасными анклавами | Нет |
| Динамическое маскирование данных | Да |
| Основные возможности аудита | Да |
| Аудит мелких фрагментов данных | Да |
| Прозрачное шифрование в базе данных | Да |
| Определяемые пользователем роли | Да |
| Автономные базы данных | Да |
| Шифрование для резервного копирования | Да |

###  <a name="rdbms-manageability"></a><a name="RDBMSM"></a> RDBMS Manageability  

| Компонент | Управляемый экземпляр SQL с поддержкой Azure Arc |
|--|--|
| Выделенное административное соединение | Да |
| Поддержка скриптов PowerShell | Да |
| Поддержка операций с компонентами приложения уровня данных — извлечение, развертывание, обновление, удаление | Да |
| Автоматизация политики (проверка по расписанию и изменение) | Да |
| Сборщик данных производительности | Да |
| Стандартный производительности отчет | Да |
| Структуры планов и закрепление плана для структур планов | Да |
| Прямой запрос индексированных представлений (с использованием указания NOEXPAND) | Да |
| Автоматическое сопровождение индексированного представления | Да |
| Распределенные секционированные представления | Да |
| Параллельные операции с индексами | Да |
| Автоматическое использование индексированного представления оптимизатором запросов | Да |
| Проверка согласованности параллелизма | Да |

### <a name="programmability"></a><a name="Programmability"></a> Programmability  

| Компонент | Управляемый экземпляр SQL с поддержкой Azure Arc |
|--|--|
| JSON | Да |
| Хранилище запросов | Да |  |
| Temporal | Да |  |
| Собственная поддержка XML | Да |  |
| Индексирование XML | Да |  |
| Возможности MERGE & UPSERT | Да |  |
| Типы данных даты и времени | Да |  |
| Поддержка международного использования | Да |  |
| Семантический поиск и полнотекстовый поиск | Нет |
| Определение языка в запросе | Да |  |
| Компонент Service Broker (сообщения) | Да |  |
| конечные точки в языке Transact-SQL | Да |  |
| График | Да |  |
| Служба машинного обучения | Нет |  |
| PolyBase | Нет |


### <a name="tools"></a>Инструменты

SQL Управляемый экземпляр с поддержкой службы "Дуга Azure" поддерживает различные инструменты для работы с данными, которые могут помочь в управлении данными.

| **Инструмент** | Управляемый экземпляр SQL с поддержкой Azure Arc|
| --- | --- | --- |
| Портал Azure <sup>1</sup> | Нет |
| Azure CLI | Нет |
| [Azure Data Studio](/sql/azure-data-studio/what-is) | Да |
| Azure PowerShell | Да |
| [BACPAC-файл (экспорт)](/sql/relational-databases/data-tier-applications/export-a-data-tier-application) | Да |
| [BACPAC-файл (импорт)](/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database) | Да |
| [SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt) | Да |
| [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) | Да |
| [SQL Server PowerShell](/sql/relational-databases/scripting/sql-server-powershell) | Да |
| [Приложение SQL Server Profiler](/sql/tools/sql-server-profiler/sql-server-profiler) | Да |

<sup>1</sup> портал Azure используется только для просмотра УПРАВЛЯЕМЫХ экземпляров SQL с поддержкой ARC в режиме только для чтения во время предварительной версии.


### <a name="unsupported-features--services"></a><a name="Unsupported"></a> Неподдерживаемые функции & Services

Следующие функции и службы недоступны для SQL Управляемый экземпляр с поддержкой ARC. Поддержка этих функций будет постепенно реализовываться с течением времени.

| Область | Неподдерживаемая функция или служба |
|-----|-----|
| **Ядро СУБД** | Репликация слиянием |
| &nbsp; | База данных Stretch |
| &nbsp; | Распределенный запрос с сторонними подключениями |
| &nbsp; | Связанные серверы с источниками данных, отличными от SQL Server и продуктов SQL Azure |
| &nbsp; | Системные расширенные хранимые процедуры (XP_CMDSHELL и т. п.) |
| &nbsp; | FileTable, FILESTREAM |
| &nbsp; | Сборки CLR с набором разрешений EXTERNAL_ACCESS или UNSAFE |
| &nbsp; | Buffer Pool Extension |
| **Агент SQL Server** |  Подсистемы: CmdExec, PowerShell, средство чтения очереди, SSIS, SSAS, SSRS |
| &nbsp; | видны узлы |
| &nbsp; | Управляемое резервное копирование |
| **Обеспечение высокого уровня доступности** | Зеркальное отображение базы данных  |
| **Безопасность** | расширенное управление ключами |
| &nbsp; | Проверка подлинности AD для связанных серверов | 
| &nbsp; | Проверка подлинности AD для групп доступности | 