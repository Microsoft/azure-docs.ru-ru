---
title: Устранение временных ошибок
description: Узнайте, как устранять, диагностировать и предотвращать ошибки подключения SQL или временные ошибки при подключении к базе данных SQL Azure, Управляемый экземпляр Azure SQL и Azure синапсе Analytics.
keywords: подключение sql,строка подключения,проблемы с подключением, временная ошибка,ошибка подключения
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: troubleshooting
author: dalechen
ms.author: ninarn
ms.reviewer: sstein, vanto
ms.date: 01/14/2020
ms.openlocfilehash: 9f2e755047910aefa89c2f187cda956aca608b98
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99093763"
---
# <a name="troubleshoot-transient-connection-errors-in-sql-database-and-sql-managed-instance"></a>Устранение временных ошибок подключения в базе данных SQL и Управляемый экземпляр SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

В этой статье описывается, как предотвратить, диагностировать и устранить ошибки подключения и временные ошибки, возникающие при взаимодействии клиентского приложения с базой данных SQL Azure, Azure SQL Управляемый экземпляр и Azure синапсе Analytics. Узнайте, как настроить логику повторных попыток, создать строку подключения и настроить другие параметры подключения.

<a id="i-transient-faults" name="i-transient-faults"></a>

## <a name="transient-errors-transient-faults"></a>Временные ошибки (временные сбои)

Временные ошибки (или временные сбои) возникают из-за причин, которые вскоре устраняются автоматически. Иногда временная ошибка возникает потому, что система Azure быстро сменяет аппаратные ресурсы, чтобы лучше распределить нагрузку, связанную с разными рабочими нагрузками. Большинство этих событий перенастройки завершаются менее чем за 60 секунд. Во время этого периода перенастройки могут возникнуть проблемы с подключением к базе данных в базе данных SQL. Приложения, подключающиеся к базе данных, должны быть построены для ожидания этих временных ошибок. и предусматривать их обработку путем реализации логики повторных попыток в коде вместо вывода пользователю сообщений об ошибках приложения.

Если клиентская программа использует пакет ADO.NET, ваша программа получает исключение **SqlException** и таким образом узнает о временной ошибке.

<a id="connection-versus-command" name="connection-versus-command"></a>

### <a name="connection-vs-command"></a>Подключение и команда

Повторите попытку подключения к базе данных SQL и SQL Управляемый экземпляр или установите ее снова в зависимости от следующих действий.

- **Временная ошибка возникает при попытке подключения**.

Через несколько секунд повторите попытку подключения.

- **Временная ошибка возникает во время выполнения команды запроса базы данных SQL и SQL Управляемый экземпляр**

Не повторяйте команду сразу. Лучше заново установите подключение после небольшой задержки. Затем снова выполните команду.

<a id="j-retry-logic-transient-faults" name="j-retry-logic-transient-faults"></a>

## <a name="retry-logic-for-transient-errors"></a>Повтор логики для временных ошибок

Клиентские программы, в которых иногда происходят временные ошибки, являются более надежными, если используют логику повторных попыток. Когда программа взаимодействует с базой данных в базе данных SQL с помощью по промежуточного слоя стороннего производителя, спросите у поставщика, содержит ли по промежуточного слоя логику повторных попыток для временных ошибок.

<a id="principles-for-retry" name="principles-for-retry"></a>

### <a name="principles-for-retry"></a>Принципы повторных попыток

- Если ошибка является временной, попробуйте установить соединение снова.
- Не пытайтесь напрямую повторно использовать базу данных SQL или `SELECT` инструкцию sql управляемый экземпляр, которая завершилась сбоем с временной ошибкой. Установите новое подключение и повторите `SELECT`.
- Когда база данных SQL или инструкция SQL Управляемый экземпляр `UPDATE` завершается с временной ошибкой, установите новое подключение, прежде чем повторять попытку обновления. Логика повторных попыток обеспечивает полноценное выполнение транзакции базы данных или откат всей транзакции.

### <a name="other-considerations-for-retry"></a>Другие соображения по поводу повторных попыток

- Пакетная программа, которая автоматически запускается после окончания рабочего дня и завершает свою работу до наступления утра, может выполнять повторные попытки с продолжительными интервалами.
- Программа, использующая пользовательский интерфейс, должна учитывать, что человек не любит ждать очень долго. Это не значит, что нужно повторять попытку каждые несколько секунд, так как подобная политика может переполнить систему запросами.

### <a name="interval-increase-between-retries"></a>Увеличение интервала между повторными попытками

Рекомендуется подождать 5 секунд, прежде чем выполнять первую повторную попытку. Повторная попытка после ожидания менее 5 секунд может привести к перегрузке облачной службы. Для каждой последующей повторной попытки ожидание должно увеличиваться экспоненциально, но не более чем до 60 секунд.

Обсуждение периода блокировки для клиентов, использующих ADO.NET, см. в разделе [Организация пулов соединений (ADO.NET)](/dotnet/framework/data/adonet/sql-server-connection-pooling).

Вы также можете задать максимальное количество повторных попыток, которые программа должна выполнить перед автоматическим завершением работы.

### <a name="code-samples-with-retry-logic"></a>Образцы кода с логикой повторных попыток

Образцы кода с логикой повторных попыток доступны по ссылкам:

- [Устойчивое подключение к Azure SQL с помощью ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n]
- [Устойчивое подключение к Azure SQL с помощью PHP][step-4-connect-resiliently-to-sql-with-php-p42h]

<a id="k-test-retry-logic" name="k-test-retry-logic"></a>

### <a name="test-your-retry-logic"></a>Тестирование логики повторных попыток

Чтобы протестировать логику повторных попыток, нужно имитировать или вызвать ошибку, которую можно исправить во время работы программы.

#### <a name="test-by-disconnecting-from-the-network"></a>Тестирование с отключением от сети

Один из способов протестировать логику повторных попыток — это отключить клиентский компьютер от сети во время работы программы. Ошибка:

- **SqlException.Number** = 11001
- Сообщение: "Данный узел неизвестен."

В рамках первой повторной попытки можно подключить клиентский компьютер к сети и попытаться подключиться.

Чтобы сделать это тестирование практичным, следует отключить компьютер от сети, прежде чем запускать программу. Программа распознает параметр среды выполнения, после чего делает следующее.

- Временно добавляет 11001 в свой список ошибок, чтобы эта ошибка считалась временной.
- Первый раз пытается подключиться как обычно.
- После выявления ошибки удаляет 11001 из списка.
- Выводит сообщение с просьбой подключить компьютер к сети.
- Программа приостанавливает дальнейшее выполнение с помощью метода **Console.ReadLine** или диалогового окна с кнопкой "ОК". Клавишу ВВОД пользователю нужно нажимать после подключения компьютера к сети.
- Попытаться подключиться еще раз. В этот раз попытка должна завершиться успехом.

#### <a name="test-by-misspelling-the-user-name-when-connecting"></a>Проверка путем неправильного написания имени пользователя при подключении

Программа может намеренно сделать опечатку в имени пользователя перед первой попыткой подключения. Ошибка:

- **SqlException.Number** = 18456
- Сообщение: "Ошибка входа пользователя WRONG_MyUserName."

В рамках первой повторной попытки программа может исправить опечатку, а затем попытаться подключиться.

Чтобы сделать это тестирование практичным, программа должна распознать параметр времени выполнения и сделать следующее:

- Временно добавить 18456 в свой список ошибок, чтобы эта ошибка считалась временной.
- Добавить элемент WRONG_ к имени пользователя.
- После выявления ошибки удалить 18456 из списка.
- Удалить WRONG_ из имени пользователя.
- Попытаться подключиться еще раз. В этот раз попытка должна завершиться успехом.

<a id="net-sqlconnection-parameters-for-connection-retry" name="net-sqlconnection-parameters-for-connection-retry"></a>

## <a name="net-sqlconnection-parameters-for-connection-retry"></a>Параметры .NET SqlConnection для повторной попытки подключения

Если клиентская программа подключается к базе данных в базе данных SQL с помощью платформа .NET Framework класса **System. Data. SqlClient. SqlConnection**, используйте .NET 4.6.1 или более поздней версии (или .NET Core), чтобы можно было использовать функцию повторного подключения. Дополнительные сведения об этой функции см. в разделе [свойство SqlConnection. ConnectionString](/dotnet/api/system.data.sqlclient.sqlconnection.connectionstring?view=netframework-4.8&preserve-view=true).

<!--
2015-11-30, FwLink 393996 points to dn632678.aspx, which links to a downloadable .docx related to SqlClient and SQL Server 2014.
-->

При создании [строки подключения](/dotnet/api/system.data.sqlclient.sqlconnection.connectionstring) для объекта **SqlConnection** нужно правильно настроить значения следующих параметров.

- **ConnectRetryCount**: &nbsp; &nbsp; значение по умолчанию — 1. Диапазон — от 0 до 255.
- **ConnectRetryInterval**: &nbsp; &nbsp; значение по умолчанию — 10 секунд. Диапазон — от 1 до 60.
- **Время ожидания подключения**: &nbsp; &nbsp; по умолчанию — 15 секунд. Диапазон — от 0 до 2147483647.

В частности, выбранные значения должны обеспечивать равенство Connection Timeout = ConnectRetryCount * ConnectionRetryIntervall.

Например, если количество попыток — 3, интервал между ними составляет 10 секунд, а время ожидания подключения только 29 секунд, то у системы не останется времени для последней (третьей) попытки подключения: 29 < 3 * 10.

<a id="connection-versus-command" name="connection-versus-command"></a>

## <a name="connection-vs-command"></a>Подключение и команда

Параметры **ConnectRetryCount** и **ConnectRetryInterval** позволяют объекту **SqlConnection** повторять операцию подключения, не извещая об этом программу и не вынуждая ее управлять этим процессом. Повторные попытки будут выполняться в следующих ситуациях:

- Вызов метода SqlConnection. Open
- SqlConnection.Exeвызов метода милые

Есть один важный нюанс. Если во время выполнения *запроса* возникает временная ошибка, объект **SqlConnection** не пытается подключиться еще раз. Как следствие, запрос повторно не выполняется. Но перед отправкой запроса для выполнения объект **SqlConnection** очень быстро проверит наличие соединения. Если эта быстрая проверка обнаружит проблемы с подключением, **SqlConnection** повторит операцию подключения. Если повторная попытка подключения завершится успешно, запрос отправится на выполнение.

### <a name="should-connectretrycount-be-combined-with-application-retry-logic"></a>Нужно ли сочетать ConnectRetryCount с логикой повторных попыток в приложении?

Предположим, что в вашем приложении реализована надежная настраиваемая логика повторных попыток. Допустим, приложение повторяет операцию подключения четыре раза. Если вы добавите в строку подключения значения **ConnectRetryInterval** и **ConnectRetryCount** = 3, общее количество повторных попыток составит 4 * 3 = 12. Вряд ли вам действительно нужно такое количество повторных попыток.

<a id="a-connection-connection-string" name="a-connection-connection-string"></a>

## <a name="connections-to-your-database-in-sql-database"></a>Подключения к базе данных в базе данных SQL

<a id="c-connection-string" name="c-connection-string"></a>

### <a name="connection-connection-string"></a>Подключение: строка подключения

Строка подключения, необходимая для подключения к базе данных, немного отличается от строки, используемой для подключения к SQL Server. Строку подключения для своей базы данных вы можете скопировать на [портале Azure](https://portal.azure.com/).

[!INCLUDE [sql-database-include-connection-string-20-portalshots](../../../includes/sql-database-include-connection-string-20-portalshots.md)]

<a id="b-connection-ip-address" name="b-connection-ip-address"></a>

### <a name="connection-ip-address"></a>Подключение: IP-адрес

Необходимо настроить базу данных SQL для приема подключения с IP-адреса компьютера, на котором размещена клиентская программа. Чтобы настроить эту конфигурацию, измените параметры брандмауэра с помощью [портала Azure](https://portal.azure.com/).

Если не задать IP-адрес, произойдет сбой программы и отобразится сообщение об ошибке, содержащее необходимый IP-адрес.

[!INCLUDE [sql-database-include-ip-address-22-portal](../../../includes/sql-database-include-ip-address-22-v12portal.md)]

Дополнительные сведения см. [в разделе Настройка параметров брандмауэра в базе данных SQL](firewall-configure.md).
<a id="c-connection-ports" name="c-connection-ports"></a>

### <a name="connection-ports"></a>Подключение: порты

Обычно нужно убедиться лишь в том, что на компьютере, на котором установлено клиентское приложение, для исходящей связи открыт порт 1433.

Например, если ваша клиентская программа размещена на компьютере Windows, можно использовать брандмауэр Windows на этом узле, чтобы открыть порт 1433.

1. Откройте панель управления.
2. Выберите **все элементы панели управления**  >  **Брандмауэр Windows**  >  **Дополнительные параметры**  >  **правила для исходящих подключений**  >  **действия**  >  **новое правило**.

Если клиентская программа находится на виртуальной машине Azure, см. статью [Порты для ADO.NET 4.5, отличные от порта 1433](adonet-v12-develop-direct-route-ports.md).

Общие сведения о настройке портов и IP-адресов в базе данных см. в разделе [брандмауэр базы данных SQL Azure](firewall-configure.md).

<a id="d-connection-ado-net-4-5" name="d-connection-ado-net-4-5"></a>

### <a name="connection-adonet-462-or-later"></a>Подключение: ADO.NET 4.6.2 или более поздней версии

Если для подключения к Базе данных SQL программа применяет классы ADO.NET, например **System.Data.SqlClient.SqlConnection**, мы рекомендуем использовать .NET Framework 4.6.2 или более позднюю версию.

#### <a name="starting-with-adonet-462"></a>Начиная с ADO.NET 4.6.2:

- Попытка повторного открытия подключения для SQL Azure будет выполнена немедленно, что повышает производительность приложений с поддержкой облака.

#### <a name="starting-with-adonet-461"></a>Начиная с ADO.NET 4.6.1:

- Предлагает повышенную надежность подключения к базам данных SQL с помощью метода **SqlConnection.Open**. В методе **Open** теперь реализованы лучшие механизмы повторных попыток при временных сбоях, в частности для некоторых ошибок, которые возникают при ожидании соединения.
- Поддерживается работа с пулами подключений, а также эффективная проверка функционирования объекта соединения, который видит ваша программа.

Когда вы используете объект соединения, входящий в пул подключений, программе следует временно закрывать подключение, если она его сейчас не использует. Повторное открытие подключения не требует высоких затрат, нужно просто создать новое подключение.

Если вы используете пакет ADO.NET 4.0 или более раннюю версию, мы рекомендуем обновить его до последней версии. С августа 2018 года доступна [версия ADO.NET 4.6.2](https://blogs.msdn.microsoft.com/dotnet/20../../announcing-the-net-framework-4-7-2/).

<a id="e-diagnostics-test-utilities-connect" name="e-diagnostics-test-utilities-connect"></a>

## <a name="diagnostics"></a>Диагностика

<a id="d-test-whether-utilities-can-connect" name="d-test-whether-utilities-can-connect"></a>

### <a name="diagnostics-test-whether-utilities-can-connect"></a>Диагностика: проверяет, могут ли служебные программы подключаться

Если программе не удается подключиться к базе данных в базе данных SQL, то один из вариантов диагностики заключается в попытке подключиться с помощью служебной программы. В идеале служебная программа подключается с помощью библиотеки, которую использует ваша программа.

На компьютерах под управлением Windows можно использовать следующие служебные программы:

- SQL Server Management Studio (ssms.exe) подключается с помощью ADO.NET.
- `sqlcmd.exe` подключается с помощью [ODBC](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server).

После подключения программы проверьте, работает ли короткий SQL-запрос SELECT.

<a id="f-diagnostics-check-open-ports" name="f-diagnostics-check-open-ports"></a>

### <a name="diagnostics-check-the-open-ports"></a>Диагностика: проверка открытых портов

Если есть подозрение, что попытки подключения не удались из-за проблем с портом, запустите на компьютере служебную программу, которая сообщает о конфигурациях порта.

На компьютерах Linux можно использовать следующие служебные программы:

- `netstat -nap`
- `nmap -sS -O 127.0.0.1` — вместо указанного в примере IP-адреса укажите свой.

В Windows можно использовать служебную программу [PortQry.exe](https://www.microsoft.com/download/details.aspx?id=17148). Ниже приведен пример выполнения, который запросил ситуацию с портами в базе данных SQL и был запущен на переносном компьютере:

```cmd
[C:\Users\johndoe\]
>> portqry.exe -n johndoesvr9.database.windows.net -p tcp -e 1433

Querying target system called: johndoesvr9.database.windows.net

Attempting to resolve name to IP address...
Name resolved to 23.100.117.95

querying...
TCP port 1433 (ms-sql-s service): LISTENING

[C:\Users\johndoe\]
>>
```

<a id="g-diagnostics-log-your-errors" name="g-diagnostics-log-your-errors"></a>

### <a name="diagnostics-log-your-errors"></a>Диагностика: внесение ошибок в журнал

Эпизодические неисправности иногда лучше всего диагностировать, выявляя общий шаблон за несколько дней или недель подряд.

Клиент может помочь в диагностике. Для этого ему следует вносить в журнал все ошибки, с которыми он сталкивается. У вас может получиться коррелировать записи журнала со сведениями об ошибках, которые база данных SQL вносит в журнал для внутренних целей.

Для облегчения ведения журналов можно использовать Enterprise Library 6 (EntLib60), где используются классы .NET. Дополнительные сведения см. в статье [5 - As Easy As Falling Off a Log: Using the Logging Application Block](/previous-versions/msp-n-p/dn440731(v=pandp.60)) (5. Простой вариант: использование решения Logging Application Block).

<a id="h-diagnostics-examine-logs-errors" name="h-diagnostics-examine-logs-errors"></a>

### <a name="diagnostics-examine-system-logs-for-errors"></a>Диагностика: проверка системных журналов на наличие ошибок

Ниже приведены некоторые Transact-SQL-инструкции SELECT, которые запрашивают в журналах сведения об ошибках и прочую информацию.

| Запрос у журнала | Описание |
|:--- |:--- |
| `SELECT e.*`<br/>`FROM sys.event_log AS e`<br/>`WHERE e.database_name = 'myDbName'`<br/>`AND e.event_category = 'connectivity'`<br/>`AND 2 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, e.end_time, GetUtcDate())`<br/>`ORDER BY e.event_category,`<br/>&nbsp;&nbsp;`e.event_type, e.end_time;` |В представлении [sys.event_log](/sql/relational-databases/system-catalog-views/sys-event-log-azure-sql-database) приводятся сведения об отдельных событиях, включая те, которые могут привести к временным ошибкам или проблемам с подключением.<br/><br/>В идеале значения **start_time** или **end_time** можно сопоставить с временем возникновения ошибок в клиентской программе.<br/><br/>Для выполнения этого запроса необходимо подключиться к базе данных *master*. |
| `SELECT c.*`<br/>`FROM sys.database_connection_stats AS c`<br/>`WHERE c.database_name = 'myDbName'`<br/>`AND 24 >= DateDiff`<br/>&nbsp;&nbsp;`(hour, c.end_time, GetUtcDate())`<br/>`ORDER BY c.end_time;` |Представление [sys.database_connection_stats](/sql/relational-databases/system-catalog-views/sys-database-connection-stats-azure-sql-database) отображает суммарное количество событий каждого типа, что также бывает полезно при дополнительной диагностике.<br/><br/>Для выполнения этого запроса необходимо подключиться к базе данных *master*. |

<a id="d-search-for-problem-events-in-the-sql-database-log" name="d-search-for-problem-events-in-the-sql-database-log"></a>

### <a name="diagnostics-search-for-problem-events-in-the-sql-database-log"></a>Диагностика: поиск событий проблемы в журнале базы данных SQL

Искать записи о событиях проблемы можно в журнале базы данных SQL. Попробуйте выполнить в базе данных *master* такую Transact-SQL-инструкцию SELECT:

```sql
SELECT
   object_name
  ,CAST(f.event_data as XML).value
      ('(/event/@timestamp)[1]', 'datetime2')                      AS [timestamp]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="error"]/value)[1]', 'int')             AS [error]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="state"]/value)[1]', 'int')             AS [state]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="is_success"]/value)[1]', 'bit')        AS [is_success]
  ,CAST(f.event_data as XML).value
      ('(/event/data[@name="database_name"]/value)[1]', 'sysname') AS [database_name]
FROM
  sys.fn_xe_telemetry_blob_target_read_file('el', null, null, null) AS f
WHERE
  object_name != 'login_event'  -- Login events are numerous.
  and
  '2015-06-21' < CAST(f.event_data as XML).value
        ('(/event/@timestamp)[1]', 'datetime2')
ORDER BY
  [timestamp] DESC
;
```

#### <a name="a-few-returned-rows-from-sysfn_xe_telemetry_blob_target_read_file"></a>Несколько возвращенных строк из sys.fn_xe_telemetry_blob_target_read_file

В следующем примере показано, как может выглядеть возвращаемая строка. Отображаемые пустые значения могут не быть пустыми в других строках.

```
object_name                   timestamp                    error  state  is_success  database_name

database_xml_deadlock_report  2015-10-16 20:28:01.0090000  NULL   NULL   NULL        AdventureWorks
```

<a id="l-enterprise-library-6" name="l-enterprise-library-6"></a>

## <a name="enterprise-library-6"></a>Enterprise Library 6

Enterprise Library 6 (EntLib60) — это платформа классов .NET, которая помогает реализовать надежные клиенты облачных служб, одна из которых является базой данных SQL. Дополнительные сведения обо всех полезных возможностях EntLib60 вы найдете в [этой](/previous-versions/msp-n-p/dn169621(v=pandp.10)) статье.

Одна из областей, в которой может помочь EntLib60, — логика повторных попыток для обработки временных ошибок. Дополнительные сведения см. в статье [4 - Perseverance, Secret of All Triumphs: Using the Transient Fault Handling Application Block](/previous-versions/msp-n-p/dn440719(v=pandp.60)) (4. Настойчивость — секрет всех побед. Использование блока приложения для обработки временных ошибок).

> [!NOTE]
> Исходный код для EntLib60 доступен для открытого скачивания в [Центре загрузки](https://github.com/MicrosoftArchive/enterprise-library). Корпорация Майкрософт не планирует обновлять функции и менять характер обслуживания библиотеки EntLib.

<a id="entlib60-classes-for-transient-errors-and-retry" name="entlib60-classes-for-transient-errors-and-retry"></a>

### <a name="entlib60-classes-for-transient-errors-and-retry"></a>Классы EntLib60 для временных ошибок и повторов

Следующие классы EntLib60 особенно полезны для логики повторных ошибок. Все эти классы входят в пространство имен **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling** или во вложенные пространства.

В пространстве имен **Microsoft. Practices. EnterpriseLibrary. TransientFaultHandling**:

- **RetryPolicy** ;
  - **ExecuteAction** ;
- Класс **exponentialbackoff;**
- **SqlDatabaseTransientErrorDetectionStrategy** ;
- **ReliableSqlConnection** ;
  - **ExecuteCommand** ;

В пространстве имен **Microsoft.Practices.EnterpriseLibrary.TransientFaultHandling.TestSupport**:

- **AlwaysTransientErrorDetectionStrategy** ;
- **NeverTransientErrorDetectionStrategy** ;

Ниже приведены некоторые ссылки на сведения об EntLib60.

- Бесплатная загрузка 2-го издания книги [Руководство разработчика по Microsoft Enterprise Library](https://www.microsoft.com/download/details.aspx?id=41145).
- Рекомендации: в статье [Общие рекомендации по повторным попыткам](/azure/architecture/best-practices/transient-faults) содержится подробное обсуждение логики повторных попыток.
- Загрузка на сайте NuGet компонента [Enterprise Library 6.0: Transient Fault Handling application block](https://www.nuget.org/packages/EnterpriseLibrary.TransientFaultHandling/)

<a id="entlib60-the-logging-block" name="entlib60-the-logging-block"></a>

### <a name="entlib60-the-logging-block"></a>EntLib60: блок ведения журнала

- Блок ведения журнала — это очень гибкое и настраиваемое решение, которое позволяет выполнять такие действия:
  - создавать и хранить сообщения журнала в разных расположениях;
  - классифицировать и фильтровать сообщения;
  - собирать контекстную информацию, полезную для отладки, трассировки, аудита и выполнения общих требований к ведению журнала.
- Это решение извлекает функции ведения журнала из расположения журнала, что обеспечивает согласованность кода приложения независимо от расположения и типа хранилища журнала.

Дополнительные сведения см. в статье [5 - As Easy As Falling Off a Log: Using the Logging Application Block](/previous-versions/msp-n-p/dn440731(v=pandp.60)) (5. Простой вариант: использование решения Logging Application Block).

<a id="entlib60-istransient-method-source-code" name="entlib60-istransient-method-source-code"></a>

### <a name="entlib60-istransient-method-source-code"></a>Исходный код метода EntLib60 IsTransient

Ниже приведен исходный код (на языке C#) метода **IsTransient** из класса **SqlDatabaseTransientErrorDetectionStrategy**. Исходный код поясняет, какие ошибки считаются временными и приемлемыми для повторной попытки (версия за апрель 2013 г.).

```csharp
public bool IsTransient(Exception ex)
{
  if (ex != null)
  {
    SqlException sqlException;
    if ((sqlException = ex as SqlException) != null)
    {
      // Enumerate through all errors found in the exception.
      foreach (SqlError err in sqlException.Errors)
      {
        switch (err.Number)
        {
            // SQL Error Code: 40501
            // The service is currently busy. Retry the request after 10 seconds.
            // Code: (reason code to be decoded).
          case ThrottlingCondition.ThrottlingErrorNumber:
            // Decode the reason code from the error message to
            // determine the grounds for throttling.
            var condition = ThrottlingCondition.FromError(err);

            // Attach the decoded values as additional attributes to
            // the original SQL exception.
            sqlException.Data[condition.ThrottlingMode.GetType().Name] =
              condition.ThrottlingMode.ToString();
            sqlException.Data[condition.GetType().Name] = condition;

            return true;

          case 10928:
          case 10929:
          case 10053:
          case 10054:
          case 10060:
          case 40197:
          case 40540:
          case 40613:
          case 40143:
          case 233:
          case 64:
            // DBNETLIB Error Code: 20
            // The instance of SQL Server you attempted to connect to
            // does not support encryption.
          case (int)ProcessNetLibErrorCode.EncryptionNotSupported:
            return true;
        }
      }
    }
    else if (ex is TimeoutException)
    {
      return true;
    }
    else
    {
      EntityException entityException;
      if ((entityException = ex as EntityException) != null)
      {
        return this.IsTransient(entityException.InnerException);
      }
    }
  }

  return false;
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Библиотеки подключений для Базы данных SQL и SQL Server](connect-query-content-reference-guide.md#libraries)
- [Организация пулов соединений (ADO.NET)](/dotnet/framework/data/adonet/sql-server-connection-pooling)
- [*Retrying* — это лицензированная общая библиотека Apache 2.0 для повторных попыток, написанная на языке Python](https://pypi.python.org/pypi/retrying), которая позволяет легко добавить режим повтора куда угодно.

<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: /sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net

[step-4-connect-resiliently-to-sql-with-php-p42h]: /sql/connect/php/step-4-connect-resiliently-to-sql-with-php