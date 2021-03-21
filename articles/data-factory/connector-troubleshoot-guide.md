---
title: Устранение неполадок соединителей фабрики данных Azure
description: Узнайте, как устранять неполадки с соединителями Фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 02/08/2021
ms.author: jingwang
ms.custom: has-adal-ref
ms.openlocfilehash: 9d8f940e3900c00b1c6f6623dfeff2d92ca85aa3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102042442"
---
# <a name="troubleshoot-azure-data-factory-connectors"></a>Устранение неполадок соединителей фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье рассматриваются распространенные способы устранения проблем с соединителями фабрики данных Azure.

## <a name="azure-blob-storage"></a>хранилище BLOB-объектов Azure

### <a name="error-code-azurebloboperationfailed"></a>Код ошибки: Азуреблобоператионфаилед

- **Сообщение**: ошибка операции BLOB-объекта. ContainerName:% containerName;, путь:% path;. "

- **Причина**: проблема с операцией с хранилищем BLOB-объектов.

- **Рекомендация**: чтобы проверить сведения об ошибке, см. раздел [коды ошибок хранилища BLOB-объектов](/rest/api/storageservices/blob-service-error-codes). За дальнейшей помощью обратитесь к команде хранилища BLOB-объектов.


### <a name="invalid-property-during-copy-activity"></a>Недопустимое свойство во время действия копирования

- **Сообщение.** `Copy activity \<Activity Name> has an invalid "source" property. The source type is not compatible with the dataset \<Dataset Name> and its linked service \<Linked Service Name>. Please verify your input against.`

- **Причина**: тип, определенный в наборе данных, не согласуется с типом источника или приемника, определенным в действии копирования.

- **Решение**. Измените определение набора данных или конвейера, чтобы обеспечить единообразие типов, а затем повторно запустите развертывание.


## <a name="azure-cosmos-db"></a>Azure Cosmos DB

### <a name="error-message-request-size-is-too-large"></a>Сообщение об ошибке: Слишком большой размер запроса.

- **Симптомы**. при копировании данных в Azure Cosmos DB с размером пакета записи по умолчанию возникает следующая ошибка: `Request size is too large.`

- **Причина**: Azure Cosmos DB ограничивает размер одного запроса до 2 МБ. Формула имеет *размер запроса = размер отдельного документа \* для записи размера пакета*. Если размер документа большой, поведение по умолчанию приведет к тому, что размер запроса будет слишком большим. Размер пакета для записи можно настроить.

- **Решение**. в приемнике действия копирования уменьшите значение *размера пакета записи* (значение по умолчанию — 10000).

### <a name="error-message-unique-index-constraint-violation"></a>Сообщение об ошибке: Нарушение уникального ограничения индекса

- **Симптомы**. при копировании данных в Azure Cosmos DB появляется следующее сообщение об ошибке:

    `Message=Partition range id 0 | Failed to import mini-batch. 
    Exception was Message: {"Errors":["Encountered exception while executing function. Exception = Error: {\"Errors\":[\"Unique index constraint violation.\"]}...`

- **Причина.** Существуют две возможные причины.

    - Причина 1. при использовании инструкции **INSERT** в качестве поведения записи эта ошибка означает, что исходные данные имеют строки или объекты с ОДИНАКОВым идентификатором.
    - Причина 2. Если в качестве поведения записи используется **Upsert** , а для контейнера задается другой уникальный ключ, эта ошибка означает, что исходные данные имеют строки или объекты с разными идентификаторами, но имеют одинаковое значение для определенного уникального ключа.

- **Решение**. 

    - Для причины 1 задайте **Upsert** в качестве поведения записи.
    - Для причины 2 Убедитесь, что каждый документ имеет другое значение для определенного уникального ключа.

### <a name="error-message-request-rate-is-large"></a>Сообщение об ошибке: Высокая частота запросов

- **Симптомы**. при копировании данных в Azure Cosmos DB появляется следующее сообщение об ошибке:

    `Type=Microsoft.Azure.Documents.DocumentClientException,
    Message=Message: {"Errors":["Request rate is large"]}`

- **Причина**: число используемых единиц запросов (RUS) превышает доступное значение, настроенное в Azure Cosmos DB. Сведения о том, как Azure Cosmos DB вычислится [в Azure Cosmos DB](../cosmos-db/request-units.md#request-unit-considerations), см. в разделе единицы запросов.

- **Решение**. Попробуйте одно из следующих двух решений:

    - Увеличьте значение *контейнера RUs* в Azure Cosmos DB. Это решение повысит производительность действия копирования, но в Azure Cosmos DB будет взиматься дополнительная стоимость. 
    - Уменьшите *writeBatchSize* до меньшего значения, например 1000, и уменьшите *parallelCopies* до меньшего значения, например 1. Это решение сокращает производительность выполнения копирования, но не требует дополнительных затрат на Azure Cosmos DB.

### <a name="columns-missing-in-column-mapping"></a>Столбцы, отсутствующие в сопоставлении столбцов

- **Симптомы**. при импорте схемы для Azure Cosmos DB сопоставления столбцов отсутствуют некоторые столбцы. 

- **Причина**. фабрика данных выводит схему из первых 10 Azure Cosmos DB документов. Если некоторые столбцы или свойства документа не содержат значений, схема не обнаруживается фабрикой данных и, следовательно, не отображается.

- **Решение**. можно настроить запрос, как показано в следующем коде, чтобы принудительно отображать значения столбцов в результирующем наборе с пустыми значениями. Предположим, что в первых 10 документах отсутствует *невозможность* столбца. Кроме того, можно вручную добавить столбец для сопоставления.

    ```sql
    select c.company, c.category, c.comments, (c.impossible??'') as impossible from c
    ```

### <a name="error-message-the-guidrepresentation-for-the-reader-is-csharplegacy"></a>Сообщение об ошибке: GuidRepresentation для модуля чтения имеет значение CSharpLegacy

- **Симптомы**. при копировании данных из Azure Cosmos DB Монгоапи или MongoDB с помощью поля универсального уникального идентификатора (UUID) появляется следующее сообщение об ошибке:

    `Failed to read data via MongoDB client., 
    Source=Microsoft.DataTransfer.Runtime.MongoDbV2Connector,Type=System.FormatException, 
    Message=The GuidRepresentation for the reader is CSharpLegacy which requires the binary sub type to be UuidLegacy not UuidStandard.,Source=MongoDB.Bson,’“,`

- **Причина**: существует два способа представления UUID в двоичном JSON (BSON): Ууидстардард и ууидлегаци. По умолчанию UuidLegacy используется для чтения данных. Если данные UUID в MongoDB имеют значение Ууидстандард, появится сообщение об ошибке.

- **Решение**. в строке подключения MongoDB добавьте параметр *ууидрепресентатион = Standard* . Дополнительные сведения см. в разделе о [строке подключения MongoDB](connector-mongodb.md#linked-service-properties).
            
## <a name="azure-cosmos-db-sql-api"></a>Azure Cosmos DB (SQL API)

### <a name="error-code-cosmosdbsqlapioperationfailed"></a>Код ошибки: Космосдбсклапиоператионфаилед

- **Сообщение.** `CosmosDbSqlApi operation Failed. ErrorMessage: %msg;.`

- **Причина**: проблема с операцией космосдбсклапи.

- **Рекомендация**: сведения об ошибке см. в разделе [Azure Cosmos DB справочный документ](../cosmos-db/troubleshoot-dot-net-sdk.md). За дальнейшей помощью обратитесь к группе Azure Cosmos DB.

## <a name="azure-data-lake-storage-gen1"></a>Хранилище Azure Data Lake Storage 1-го поколения

### <a name="error-message-the-underlying-connection-was-closed-could-not-establish-trust-relationship-for-the-ssltls-secure-channel"></a>Сообщение об ошибке: базовое соединение закрыто: не удалось установить доверительные отношения для защищенного канала SSL/TLS.

- **Симптомы**. сбой действия копирования со следующей ошибкой: 

    `Message: ErrorCode = UserErrorFailedFileOperation, Error Message = The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.`

- **Причина**: сбой проверки сертификата во время подтверждения TLS.

- **Решение**. в качестве обходного решения используйте промежуточное копирование, чтобы пропустить проверку протокола tls для Azure Data Lake Storage 1-го поколения. Необходимо воспроизвести эту ошибку и собрать трассировку сетевого монитора (Netmon), а затем обратиться в сетевую группу, чтобы проверить конфигурацию локальной сети.

    ![Схема Azure Data Lake Storage 1-го поколения подключений для устранения неполадок.](./media/connector-troubleshoot-guide/adls-troubleshoot.png)


### <a name="error-message-the-remote-server-returned-an-error-403-forbidden"></a>Сообщение об ошибке: Удаленный сервер вернул ошибку: (403) Запрещено

- **Проблема.** Сбой действия копирования со следующей ошибкой: 

   `Message: The remote server returned an error: (403) Forbidden.   
    Response details: {"RemoteException":{"exception":"AccessControlException""message":"CREATE failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.)....`

- **Причина**: одна из возможных причин заключается в том, что используемый субъект-служба или управляемое удостоверение не имеет разрешения на доступ к определенным папкам или файлам.

- **Решение**. Предоставьте соответствующие разрешения всем папкам и вложенным папкам, которые необходимо скопировать. Дополнительные сведения см. [в статье копирование данных в Azure Data Lake Storage 1-го поколения с помощью фабрики данных Azure или из](connector-azure-data-lake-store.md#linked-service-properties)нее.

### <a name="error-message-failed-to-get-access-token-by-using-service-principal-adal-error-service_unavailable"></a>Сообщение об ошибке: Не удалось получить токен доступа с помощью субъекта-службы. Ошибка ADAL: service_unavailable

- **Симптомы**. сбой действия копирования со следующей ошибкой:

    `Failed to get access token by using service principal.  
    ADAL Error: service_unavailable, The remote server returned an error: (503) Server Unavailable.`

- **Причина**: Если сервер маркеров службы (STS), принадлежащий Azure Active Directory, недоступен, это означает, что он слишком занят для обработки запросов и ВОЗВРАЩАЕТ ошибку HTTP 503. 

- **Решение**. Повторите действие копирования через несколько минут.


## <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage 2-го поколения

### <a name="error-code-adlsgen2operationfailed"></a>Код ошибки: ADLSGen2OperationFailed

- **Сообщение.** `ADLS Gen2 operation failed for: %adlsGen2Message;.%exceptionData;.`

- **Причины и рекомендации**. различные причины могут привести к этой ошибке. Ниже приведен список возможных причин и соответствующие рекомендации.

  | Анализ причин                                               | Рекомендация                                               |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | Если Azure Data Lake Storage 2-го поколения выдает ошибку, указывающую на сбой некоторой операции.| Проверьте подробное сообщение об ошибке, выдаваемое Azure Data Lake Storage 2-го поколения. Если ошибка является временным сбоем, повторите операцию. Для получения дополнительной помощи обратитесь в службу поддержки хранилища Azure и укажите идентификатор запроса в сообщении об ошибке. |
  | Если сообщение об ошибке содержит строку "запрещено", возможно, субъект-служба или управляемое удостоверение, которые вы используете, не имеют достаточных разрешений для доступа к Azure Data Lake Storage 2-го поколения. | Сведения об устранении этой ошибки см. [в статье копирование и преобразование данных в Azure Data Lake Storage 2-го поколения с помощью фабрики данных Azure](./connector-azure-data-lake-storage.md#service-principal-authentication). |
  | Если сообщение об ошибке содержит строку "InternalServerError", то ошибка возвращается Azure Data Lake Storage 2-го поколения. | Эта ошибка может быть вызвана временным сбоем. Если это так, повторите операцию. Если проблема будет повторяться, обратитесь в службу поддержки хранилища Azure и укажите идентификатор запроса из сообщения об ошибке. |

### <a name="request-to-azure-data-lake-storage-gen2-account-caused-a-timeout-error"></a>Запрос к Azure Data Lake Storage 2-го поколения учетной записи привел к ошибке времени ожидания

- **Сообщение**. 
  * Код ошибки = `UserErrorFailedBlobFSOperation`
  * Сообщение об ошибке = `BlobFS operation failed for: A task was canceled.`

- **Причина**. проблема вызвана ошибкой времени ожидания приемника Azure Data Lake Storage 2-го поколения, которая обычно возникает на компьютере с автономной Integration Runtime (IR).

- **Рекомендация**. 

    - Поместите собственный IR-компьютер и целевую учетную запись Azure Data Lake Storage 2-го поколения в том же регионе, если это возможно. Это может помочь избежать ошибки случайного времени ожидания и повысить производительность.

    - Проверьте наличие специальных параметров сети, например ExpressRoute, и убедитесь, что пропускная способность сети достаточно велика. Мы рекомендуем уменьшить значение параметра одновременных параллельных заданий, если общая пропускная способность мала. Это позволит избежать конкуренции между сетевыми ресурсами в нескольких параллельных заданиях.

    - Если размер файла умеренный или малый, используйте размер блока меньшего размера для недвоичного копирования, чтобы устранить такую ошибку времени ожидания. Дополнительные сведения см. в разделе [Размещение блока хранилища BLOB-объектов](/rest/api/storageservices/put-block).

       Чтобы указать размер пользовательского блока, измените свойство в редакторе файлов JSON, как показано ниже:
    
        ```
        "sink": {
            "type": "DelimitedTextSink",
            "storeSettings": {
                "type": "AzureBlobFSWriteSettings",
                "blockSizeInMB": 8
            }
        }
        ```

                  
## <a name="azure-files-storage"></a>Хранилище файлов Azure

### <a name="error-code-azurefileoperationfailed"></a>Код ошибки: Азурефилеоператионфаилед

- **Сообщение.** `Azure File operation Failed. Path: %path;. ErrorMessage: %msg;.`

- **Причина**: проблема с операцией с хранилищем файлов Azure.

- **Рекомендация**: чтобы проверить сведения об ошибке, см. [раздел Справка по файлам Azure](/rest/api/storageservices/file-service-error-codes). За дальнейшей помощью обратитесь в службу поддержки файлов Azure.


## <a name="azure-synapse-analytics-azure-sql-database-and-sql-server"></a>Azure синапсе Analytics, база данных SQL Azure и SQL Server

### <a name="error-code-sqlfailedtoconnect"></a>Код ошибки: Склфаиледтоконнект

- **Сообщение.** `Cannot connect to SQL Database: '%server;', Database: '%database;', User: '%user;'. Check the linked service configuration is correct, and make sure the SQL Database firewall allows the integration runtime to access.`
- **Причины и рекомендации**. различные причины могут привести к этой ошибке. Ниже приведен список возможных причин и соответствующие рекомендации.

    | Анализ причин                                               | Рекомендация                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | В случае с Azure SQL, если сообщение об ошибке содержит строку "Склеррорнумбер = 47073", это означает, что доступ к общедоступной сети запрещен в параметре подключения. | В брандмауэре Azure SQL установите для параметра **запретить доступ к общедоступной сети** значение *нет*. Дополнительные сведения см. в статье [Параметры подключения к SQL Azure](../azure-sql/database/connectivity-settings.md#deny-public-network-access). |
    | Для Azure SQL, если сообщение об ошибке содержит код ошибки SQL, например "Склеррорнумбер = [ErrorCode]", см. Руководство по устранению неполадок SQL Azure. | Рекомендации см. в статьях [Устранение неполадок подключения и другие ошибки в базе данных SQL Azure и управляемый экземпляр Azure SQL](../azure-sql/database/troubleshoot-common-errors-issues.md). |
    | Проверьте, включен ли порт 1433 в список разрешений брандмауэра. | Дополнительные сведения см. в разделе [порты, используемые SQL Server](/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access#ports-used-by-). |
    | Если сообщение об ошибке содержит строку «SqlException», то база данных SQL. Эта ошибка указывает на сбой некоторой конкретной операции. | Для получения дополнительных сведений выполните поиск по коду ошибки SQL в [ошибках ядра](/sql/relational-databases/errors-events/database-engine-events-and-errors)СУБД. Для получения дополнительной помощи обратитесь в службу поддержки Azure SQL. |
    | Если это временная проблема (например, нестабильное сетевое подключение), добавьте повторную попытку в политике действий, чтобы устранить неполадки. | См. дополнительные сведения о [конвейерах и действиях в фабрике данных Azure](./concepts-pipelines-activities.md#activity-policy). |
    | Если сообщение об ошибке содержит строку "клиент с IP-адресом"... " не разрешен доступ к серверу ", и вы пытаетесь подключиться к базе данных SQL Azure, то ошибка обычно вызвана проблемой с брандмауэром базы данных SQL Azure. | В конфигурации брандмауэра Azure SQL Server включите параметр **Разрешить службам Azure и ресурсам доступ к этому серверу** . Дополнительные сведения см. в статьях [база данных SQL Azure и правила брандмауэра IP-адресов Azure синапсе](../azure-sql/database/firewall-configure.md). |
    
### <a name="error-code-sqloperationfailed"></a>Код ошибки: Склоператионфаилед

- **Сообщение.** `A database operation failed. Please search error to get more details.`

- **Причины и рекомендации**. различные причины могут привести к этой ошибке. Ниже приведен список возможных причин и соответствующие рекомендации.

    | Анализ причин                                               | Рекомендация                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | Если сообщение об ошибке содержит строку «SqlException», то база данных SQL выдает ошибку, указывающую на сбой некоторой конкретной операции. | Если ошибка SQL не была понятна, попробуйте изменить базу данных до последнего уровня совместимости "150". Это может вызвать ошибки SQL последней версии. Дополнительные сведения см. в этой [документации](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level#backwardCompat). <br/> Для получения дополнительных сведений об устранении неполадок SQL выполните поиск по коду ошибки SQL в [ошибках ядра](/sql/relational-databases/errors-events/database-engine-events-and-errors)СУБД. Для получения дополнительной помощи обратитесь в службу поддержки Azure SQL. |
    | Если сообщение об ошибке содержит строку «Пдвманажедтонативеинтеропексцептион», это обычно вызвано несоответствием размеров столбцов источника и приемника. | Проверьте размер столбцов источника и приемника. Для получения дополнительной помощи обратитесь в службу поддержки Azure SQL. |
    | Если сообщение об ошибке содержит строку "InvalidOperationException", это обычно вызвано недопустимыми входными данными. | Чтобы узнать, какая строка столкнулась с проблемой, включите функцию отказоустойчивости в действии копирования, которая может переназначать проблемные строки в хранилище для дальнейшего изучения. Дополнительные сведения см. [в статье отказоустойчивость действия копирования в фабрике данных Azure](./copy-activity-fault-tolerance.md). |


### <a name="error-code-sqlunauthorizedaccess"></a>Код ошибки: Склунаусоризедакцесс

- **Сообщение.** `Cannot connect to '%connectorName;'. Detail Message: '%message;'`

- **Причина**: учетные данные неверны, или учетная запись входа не может получить доступ к базе данных SQL.

- **Рекомендация**: Убедитесь, что учетная запись входа имеет достаточные разрешения для доступа к базе данных SQL.


### <a name="error-code-sqlopenconnectiontimeout"></a>Код ошибки: Склопенконнектионтимеаут

- **Сообщение.** `Open connection to database timeout after '%timeoutValue;' seconds.`

- **Причина**: проблема может быть временным сбоем базы данных SQL.

- **Рекомендация**: повторите операцию, чтобы обновить строку подключения связанной службы с большим значением времени ожидания соединения.


### <a name="error-code-sqlautocreatetabletypemapfailed"></a>Код ошибки: Склаутокреатетаблетипемапфаилед

- **Сообщение.** `Type '%dataType;' in source side cannot be mapped to a type that supported by sink side(column name:'%columnName;') in autocreate table.`

- **Причина**: таблица автосоздания не может соответствовать исходному требованию.

- **Рекомендация**: обновите тип столбца в *сопоставлениях* или вручную создайте таблицу приемника на целевом сервере.


### <a name="error-code-sqldatatypenotsupported"></a>Код ошибки: Склдататипенотсуппортед

- **Сообщение.** `A database operation failed. Check the SQL errors.`

- **Причина**: Если проблема возникает в ИСТОЧНИКе SQL, а ошибка связана с переполнением SqlDateTime, значение данных превышает диапазон типов логики (1/1/1753 12:00:00 AM-12/31/9999 11:59:59 PM).

- **Рекомендация**: приведите тип к строке в исходном SQL-запросе или в сопоставлении столбцов действия копирования измените тип столбца на *String*.

- **Причина**: Если проблема возникает в ПРИЕМНИКе SQL, а ошибка связана с переполнением SqlDateTime, значение данных превышает допустимый диапазон в таблице приемника.

- **Рекомендация**: обновите соответствующий тип столбца до типа *datetime2* в таблице приемника.


### <a name="error-code-sqlinvaliddbstoredprocedure"></a>Код ошибки: Склинвалиддбсторедпроцедуре

- **Сообщение.** `The specified Stored Procedure is not valid. It could be caused by that the stored procedure doesn't return any data. Invalid Stored Procedure script: '%scriptName;'.`

- **Причина**: указанная хранимая процедура недопустима. Причиной может быть то, что хранимая процедура не возвращает никаких данных.

- **Рекомендация**: Проверьте хранимую процедуру с помощью средств SQL. Убедитесь, что хранимая процедура может возвращать данные.


### <a name="error-code-sqlinvaliddbquerystring"></a>Код ошибки: Склинвалиддбкуеристринг

- **Сообщение.** `The specified SQL Query is not valid. It could be caused by that the query doesn't return any data. Invalid query: '%query;'`

- **Причина**: указан недопустимый SQL-запрос. Причиной может быть то, что запрос не возвращает никаких данных.

- **Рекомендация**: Проверка SQL запроса с помощью средств SQL. Убедитесь, что запрос может возвращать данные.


### <a name="error-code-sqlinvalidcolumnname"></a>Код ошибки: Склинвалидколумннаме

- **Сообщение.** `Column '%column;' does not exist in the table '%tableName;', ServerName: '%serverName;', DatabaseName: '%dbName;'.`

- **Причина**: не удается найти столбец, так как конфигурация может быть неверной.

- **Рекомендация**: Проверьте столбец в запросе, *структуру* в наборе данных и *сопоставления* в действии.


### <a name="error-code-sqlbatchwritetimeout"></a>Код ошибки: Склбатчвритетимеаут

- **Сообщение.** `Timeouts in SQL write operation.`

- **Причина**: проблема может быть вызвана временным сбоем базы данных SQL.

- **Рекомендация**: повторите операцию. Если проблема будет повторяться, обратитесь в службу поддержки SQL Azure.


### <a name="error-code-sqlbatchwritetransactionfailed"></a>Код ошибки: Склбатчвритетрансактионфаилед

- **Сообщение.** `SQL transaction commits failed.`

- **Причина**: Если сведения об исключении постоянно указывают на время ожидания транзакции, задержка в сети между средой выполнения интеграции и базой данных превышает пороговое значение по умолчанию (30 секунд).

- **Рекомендация**. Обновите строку подключения к связанной службе SQL, указав значение *времени ожидания соединения* , которое равно или больше 120, и повторите действие.

- **Причина**. Если сведения об исключении периодически указывают на то, что подключение SQL разорвано, это может быть временный сбой сети или проблема с базой данных SQL.

- **Рекомендация**: Повторите действие и проверьте метрики на стороне базы данных SQL.


### <a name="error-code-sqlbulkcopyinvalidcolumnlength"></a>Код ошибки: Склбулккопинвалидколумнленгс

- **Сообщение.** `SQL Bulk Copy failed due to receive an invalid column length from the bcp client.`

- **Причина**: не удалось выполнить групповое копирование SQL из-за того, что в клиенте программы пакетного копирования (BCP) получена Недопустимая длина столбца.

- **Рекомендация**: чтобы узнать, какая строка столкнулась с проблемой, включите функцию отказоустойчивости для действия копирования. Это позволяет перенаправлять проблемные строки в хранилище для дальнейшего изучения. Дополнительные сведения см. [в статье отказоустойчивость действия копирования в фабрике данных Azure](./copy-activity-fault-tolerance.md).


### <a name="error-code-sqlconnectionisclosed"></a>Код ошибки: Склконнектионисклосед

- **Сообщение.** `The connection is closed by SQL Database.`

- **Причина**: соединение SQL закрывается базой данных SQL при высоком параллельном выполнении и при разрыве соединения с сервером.

- **Рекомендация**: повторите попытку подключения. Если проблема будет повторяться, обратитесь в службу поддержки SQL Azure.


### <a name="error-message-conversion-failed-when-converting-from-a-character-string-to-uniqueidentifier"></a>Сообщение об ошибке: Ошибка при преобразовании строки символов в тип uniqueidentifier

- **Симптомы**. при копировании данных из источника табличных данных (например, SQL Server) в Azure синапсе Analytics с помощью промежуточного копирования и polybase появляется следующее сообщение об ошибке:

   `ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException, 
    Message=Error happened when loading data into Azure Synapse Analytics., 
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException, 
    Message=Conversion failed when converting from a character string to uniqueidentifier...`

- **Причина**: Azure синапсе Analytics polybase не может преобразовать пустую строку в GUID.

- **Решение**. в приемнике действия копирования в разделе Параметры polybase задайте для параметра **использовать тип по умолчанию** *значение false*.


### <a name="error-message-expected-data-type-decimalxx-offending-value"></a>Сообщение об ошибке: Ожидаемый тип данных: DECIMAL(x,x), ошибочное значение

- **Симптомы**. при копировании данных из источника табличных данных (например, SQL Server) в Azure синапсе Analytics с помощью промежуточного копирования и polybase появляется следующее сообщение об ошибке:

    `ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException, 
    Message=Error happened when loading data into Azure Synapse Analytics., 
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException, 
    Message=Query aborted-- the maximum reject threshold (0 rows) was reached while reading from an external source: 1 rows rejected out of total 415 rows processed. (/file_name.txt)  
    Column ordinal: 18, Expected data type: DECIMAL(x,x), Offending value:..`

- **Причина**: Azure синапсе Analytics polybase не может вставить пустую строку (значение null) в десятичный столбец.

- **Решение**. в приемнике действия копирования в разделе Параметры polybase задайте для параметра **использовать тип по умолчанию** значение false.


### <a name="error-message-java-exception-message-hdfsbridgecreaterecordreader"></a>Сообщение об ошибке: сообщение исключения Java: Хдфсбридже:: Креатерекордреадер

- **Симптомы**. вы копируете данные в Azure синапсе Analytics с помощью polybase и получаете следующую ошибку:

    `Message=110802;An internal DMS error occurred that caused this operation to fail.  
    Details: Exception: Microsoft.SqlServer.DataWarehouse.DataMovement.Common.ExternalAccess.HdfsAccessException, 
    Message: Java exception raised on call to HdfsBridge_CreateRecordReader.  
    Java exception message:HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.: Error [HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.] occurred while accessing external file.....`

- **Причина**. причиной может быть то, что схема (общая ширина столбца) слишком велика (больше 1 МБ). Проверьте схему целевой таблицы Azure синапсе Analytics, добавив размер всех столбцов:

    - Int = 4 байта
    - Bigint = 8 байт
    - Varchar (n), char (n), binary (n), varbinary (n) = n bytes
    - Nvarchar (n), nchar (n) = n * 2 байта
    - Дата = 6 байт
    - DateTime/(2), smalldatetime = 16 байт
    - DateTimeOffset = 20 байт
    - Десятичное число = 19 байт
    - Float = 8 байт
    - Деньги = 8 байт
    - Smallmoney = 4 байта
    - Real = 4 байта
    - Smallint = 2 байта
    - Время = 12 байт
    - Tinyint = 1 байт

- **Решение**. 
    - Уменьшите ширину столбца до значения меньше 1 МБ.
    - Или используйте метод с массовыми вставками, отключив Polybase.


### <a name="error-message-the-condition-specified-using-http-conditional-headers-is-not-met"></a>Сообщение об ошибке: Условие, указанное с помощью условных заголовков HTTP, не выполнено.

- **Симптомы**. вы используете SQL Query для извлечения данных из Azure синапсе Analytics и получаете следующую ошибку:

    `...StorageException: The condition specified using HTTP conditional header(s) is not met...`

- **Причина**: в Azure синапсе Analytics возникла ошибка при запросе внешней таблицы в службе хранилища Azure.

- **Решение**. Выполните тот же запрос в SQL Server Management Studio (SSMS) и проверьте, получен ли тот же результат. Если вы сделаете это, отправьте запрос в службу поддержки в Azure синапсе Analytics и укажите имя сервера и базы данных Azure синапсе Analytics.


### <a name="performance-tier-is-low-and-leads-to-copy-failure"></a>Низкий уровень производительности, что приводит к сбою копирования

- **Симптомы**. вы копируете данные в базу данных SQL Azure и получаете следующую ошибку: `Database operation failed. Error message from database execution : ExecuteNonQuery requires an open and available Connection. The connection's current state is closed.`

- **Причина**: в базе данных SQL Azure S1 превышены ограничения ввода-вывода.

- **Решение**. чтобы устранить проблему, обновите уровень производительности базы данных SQL Azure. 


### <a name="sql-table-cant-be-found"></a>Не удается найти таблицу SQL 

- **Симптомы**. вы копируете данные из гибридной среды в локальную SQL Server таблицу и получаете следующую ошибку:`Cannot find the object "dbo.Contoso" because it does not exist or you do not have permissions.`

- **Причина**: текущая учетная запись SQL не имеет достаточных разрешений для выполнения запросов, выданных .NET SqlBulkCopy. WriteToServer.

- **Решение**. Переключитесь на более привилегированную учетную запись SQL.


### <a name="error-message-string-or-binary-data-is-truncated"></a>Сообщение об ошибке: строка или двоичные данные усекаются

- **Симптомы**. при копировании данных в локальную таблицу Azure SQL Server возникает ошибка. 

- **Причина**: определение схемы таблицы SQL CX содержит один или несколько столбцов с меньшей длиной, чем ожидалось.

- **Решение**. чтобы устранить проблему, попробуйте выполнить следующие действия.

    1. Чтобы устранить неполадки в строках, примените [отказоустойчивость](./copy-activity-fault-tolerance.md)приемника SQL, особенно "redirectIncompatibleRowSettings".

        > [!NOTE]
        > Отказоустойчивость может потребовать дополнительного времени выполнения, что может привести к увеличению затрат.

    2. Дважды проверьте перенаправленные данные по длине столбца схемы таблицы SQL, чтобы узнать, какие столбцы необходимо обновить.

    3. Соответствующим образом обновите схему таблицы.


## <a name="azure-table-storage"></a>Хранилище таблиц Azure

### <a name="error-code-azuretableduplicatecolumnsfromsource"></a>Код ошибки: Азуретабледупликатеколумнсфромсаурце

- **Сообщение.** `Duplicate columns with same name '%name;' are detected from source. This is NOT supported by Azure Table Storage sink.`

- **Причина**: дублирующиеся исходные столбцы могут возникать по одной из следующих причин:
   * Вы используете базу данных в качестве источника и присоединяетесь к таблицам.
   * У вас есть неструктурированные CSV-файлы с повторяющимися именами столбцов в строке заголовка.

- **Рекомендация**: при необходимости дважды проверьте и исправьте исходные столбцы.


## <a name="db2"></a>DB2

### <a name="error-code-db2driverrunfailed"></a>Код ошибки: DB2DriverRunFailed

- **Сообщение.** `Error thrown from driver. Sql code: '%code;'`

- **Причина**: Если сообщение об ошибке содержит строку "SQLSTATE = 51002 SQLCODE =-805", следуйте указаниям в подсказке в [Копировать данные из DB2 с помощью фабрики данных Azure](./connector-db2.md#linked-service-properties).

- **Рекомендация**: попытайтесь задать "нуллид" в `packageCollection`  свойстве.


## <a name="delimited-text-format"></a>Формат текста с разделителями

### <a name="error-code-delimitedtextcolumnnamenotallownull"></a>Код ошибки: Делимитедтекстколумннаменоталловнулл

- **Сообщение.** `The name of column index %index; is empty. Make sure column name is properly specified in the header row.`

- **Причина**: Если в действии задано значение "firstRowAsHeader", в качестве имени столбца используется первая строка. Эта ошибка означает, что первая строка содержит пустое значение (например, "Column a, Столбецb").

- **Рекомендация**: Проверьте первую строку и исправьте значение, если оно пустое.


### <a name="error-code-delimitedtextmorecolumnsthandefined"></a>Код ошибки: Делимитедтекстмореколумнссандефинед

- **Сообщение.** `Error found when processing '%function;' source '%name;' with row number %rowCount;: found more columns than expected column count: %expectedColumnCount;.`

- **Причины и рекомендации**. различные причины могут привести к этой ошибке. Ниже приведен список возможных причин и соответствующие рекомендации.

  | Анализ причин                                               | Рекомендация                                               |
  | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | Число столбцов в проблемной строке превышает число столбцов в первой строке. Это может быть вызвано проблемой с данными или неправильным разделителем столбцов или параметрами символов кавычек. | Получите количество строк из сообщения об ошибке, проверьте столбец строки и исправьте данные. |
  | Если ожидаемое число столбцов равно "1" в сообщении об ошибке, возможно, было указано неправильное сжатие или параметры форматирования, что привело к неправильному анализу файлов в фабрике данных. | Проверьте параметры формата, чтобы убедиться, что они соответствуют исходным файлам. |
  | Если источником является папка, то файлы в указанной папке могут иметь другую схему. | Убедитесь, что файлы в указанной папке имеют одинаковую схему. |


## <a name="dynamics-365-common-data-service-and-dynamics-crm"></a>Dynamics 365, Common Data Service и Dynamics CRM

### <a name="error-code-dynamicscreateserviceclienterror"></a>Код ошибки: Динамикскреатесервицеклиентеррор

- **Сообщение.** `This is a transient issue on Dynamics server side. Try to rerun the pipeline.`

- **Причина**: проблема является временной проблемой на стороне Dynamics Server.

- **Рекомендация**.  Перезапустите конвейер. В случае повторного сбоя попробуйте уменьшить параллелизм. Если проблема будет повторяться, обратитесь в службу поддержки Dynamics.


### <a name="missing-columns-when-you-import-a-schema-or-preview-data"></a>Отсутствующие столбцы при импорте схемы или предварительном просмотре данных

- **Симптомы**. при импорте схемы или предварительном просмотре данных некоторые столбцы отсутствуют. Сообщение об ошибке: `The valid structure information (column name and type) are required for Dynamics source.`

- **Причина**: Эта проблема обусловлена проектированием, поскольку фабрике данных не удается отобразить столбцы, которые не содержат значений, в первых 10 записях. Убедитесь, что добавленные столбцы имеют правильный формат. 

- **Рекомендация**: вручную добавьте столбцы на вкладке Сопоставление.


### <a name="error-code-dynamicsmissingtargetformultitargetlookupfield"></a>Код ошибки: Динамиксмиссингтаржетформултитаржетлукупфиелд

- **Сообщение.** `Cannot find the target column for multi-target lookup field: '%fieldName;'.`

- **Причина**: целевой столбец не существует в источнике или в сопоставлении столбцов.

- **Рекомендация**.  
  1. Убедитесь, что источник содержит целевой столбец. 
  2. Добавьте целевой столбец в сопоставление столбцов. Убедитесь, что столбец приемника имеет формат *{fieldname} @EntityReference*.


### <a name="error-code-dynamicsinvalidtargetformultitargetlookupfield"></a>Код ошибки: Динамиксинвалидтаржетформултитаржетлукупфиелд

- **Сообщение.** `The provided target: '%targetName;' is not a valid target of field: '%fieldName;'. Valid targets are: '%validTargetNames;'`

- **Причина**: неправильное имя сущности указано в качестве целевой сущности для поля подстановки с несколькими целевыми адресами.

- **Рекомендация**: Укажите допустимое имя сущности для поля поиска с несколькими целевыми именами.


### <a name="error-code-dynamicsinvalidtypeformultitargetlookupfield"></a>Код ошибки: Динамиксинвалидтипеформултитаржетлукупфиелд

- **Сообщение.** `The provided target type is not a valid string. Field: '%fieldName;'.`

- **Причина**: значение в целевом столбце не является строкой.

- **Рекомендация**: Укажите допустимую строку в целевом столбце с несколькими целевыми столбцами поиска.


### <a name="error-code-dynamicsfailedtorequetserver"></a>Код ошибки: Динамиксфаиледторекуетсервер

- **Сообщение.** `The Dynamics server or the network is experiencing issues. Check network connectivity or check Dynamics server log for more details.`

- **Причина**: сервер Dynamics нестабильн или недоступен, или в сети возникли проблемы.

- **Рекомендация**. для получения дополнительных сведений проверьте подключение к сети или проверьте журнал сервера Dynamics. Для получения дополнительной помощи обратитесь в службу поддержки Dynamics.
  

## <a name="ftp"></a>FTP

### <a name="error-code-ftpfailedtoconnecttoftpserver"></a>Код ошибки: Фтпфаиледтоконнекттофтпсервер

- **Сообщение.** `Failed to connect to FTP server. Please make sure the provided server information is correct, and try again.`

- **Причина**: для FTP-сервера может использоваться неверный тип связанной службы, например, с помощью связанной службы Secure FTP (SFTP) для подключения к FTP-серверу.

- **Рекомендация**: проверьте порт целевого сервера. Протокол FTP использует порт 21.


## <a name="http"></a>HTTP

### <a name="error-code-httpfilefailedtoread"></a>Код ошибки: Хттпфилефаиледтореад

- **Сообщение.** `Failed to read data from http server. Check the error from http server：%message;`

- **Причина**: Эта ошибка возникает, когда фабрика данных Azure обращается к серверу HTTP, но операция запроса HTTP завершается ошибкой.

- **Рекомендация**: Проверьте код состояния HTTP в сообщении об ошибке и устраните проблему с удаленным сервером.


## <a name="oracle"></a>Oracle;

### <a name="error-code-argumentoutofrangeexception"></a>Код ошибки: ArgumentOutOfRangeException

- **Сообщение.** `Hour, Minute, and Second parameters describe an un-representable DateTime.`

- **Причина**: в фабрике данных значения DateTime поддерживаются в диапазоне от 0001-01-01 00:00:00 до 9999-12-31 23:59:59. Однако Oracle поддерживает более широкий диапазон значений DateTime, например BC век или min/sec>59, что приводит к сбою в фабрике данных.

- **Рекомендация**. 

    Чтобы узнать, находится ли значение в Oracle в диапазоне фабрики данных, выполните команду `select dump(<column name>)` . 

    Чтобы узнать последовательность байтов в результате, см. раздел [как даты хранятся в Oracle?](https://stackoverflow.com/questions/13568193/how-are-dates-stored-in-oracle).


## <a name="orc-format"></a>Формат ORC

### <a name="error-code-orcjavainvocationexception"></a>Код ошибки: Оркжаваинвокатионексцептион

- **Сообщение.** `An error occurred when invoking Java, message: %javaException;.`
- **Причины и рекомендации**. различные причины могут привести к этой ошибке. Ниже приведен список возможных причин и соответствующие рекомендации.

    | Анализ причин                                               | Рекомендация                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | Если сообщение об ошибке содержит строки «Java. lang. OutOfMemory», «пространство кучи Java» и «ДаублекапаЦити», это обычно является проблемой управления памятью в старой версии среды выполнения интеграции. | Если вы используете локально размещенную Integration Runtime, рекомендуется выполнить обновление до последней версии. |
    | Если сообщение об ошибке содержит строку "Java. lang. OutOfMemory", среда выполнения интеграции не имеет достаточно ресурсов для обработки файлов. | Ограничьте количество параллельных запусков в среде выполнения интеграции. Для локальной среды IR увеличивайте масштаб до мощной машины с объемом памяти, равным или превышающим 8 ГБ. |
    |Если сообщение об ошибке содержит строку "Нуллпоинтерреференце", причиной может быть временная ошибка. | Повторите операцию. Если проблема не исчезнет, обратитесь в службу поддержки. |
    | Если сообщение об ошибке содержит строку "Буффероверфловексцептион", причиной может быть временная ошибка. | Повторите операцию. Если проблема не исчезнет, обратитесь в службу поддержки. |
    | Если сообщение об ошибке содержит строку "Java. lang. Класскастексцептион:орг. Apache. Hadoop. Hive. serde2. IO. Хивечарвритабле невозможно привести к org. Apache. Hadoop. IO. Text", причиной может быть ошибка преобразования типа в среде выполнения Java. Обычно это означает, что исходные данные не могут быть хорошо обработаны в среде выполнения Java. | Это проблема с данными. Попробуйте использовать строку вместо Char или varchar в данных формата ORC. |

### <a name="error-code-orcdatetimeexceedlimit"></a>Код ошибки: Оркдатетимиксцеедлимит

- **Сообщение.** `The Ticks value '%ticks;' for the datetime column must be between valid datetime ticks range -621355968000000000 and 2534022144000000000.`

- **Причина**: Если значение DateTime равно "0001-01-01 00:00:00", это может быть вызвано различиями между [Юлианским календарем и григорианского календарем](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar#Difference_between_Julian_and_proleptic_Gregorian_calendar_dates).

- **Рекомендация**: проверьте значение тактов и Избегайте использования значения datetime "0001-01-01 00:00:00".


## <a name="parquet-format"></a>Формат Parquet

### <a name="error-code-parquetjavainvocationexception"></a>Код ошибки: Паркуетжаваинвокатионексцептион

- **Сообщение.** `An error occurred when invoking java, message: %javaException;.`

- **Причины и рекомендации**. различные причины могут привести к этой ошибке. Ниже приведен список возможных причин и соответствующие рекомендации.

    | Анализ причин                                               | Рекомендация                                               |
    | :----------------------------------------------------------- | :----------------------------------------------------------- |
    | Если сообщение об ошибке содержит строки "Java. lang. OutOfMemory", "пространство кучи Java" и "ДаублекапаЦити", это обычно является проблемой управления памятью в старой версии Integration Runtime. | Если используется локальная среда IR, а версия более ранняя, чем 3.20.7159.1, рекомендуется выполнить обновление до последней версии. |
    | Если сообщение об ошибке содержит строку "Java. lang. OutOfMemory", среда выполнения интеграции не имеет достаточно ресурсов для обработки файлов. | Ограничьте количество параллельных запусков в среде выполнения интеграции. Для локальной среды IR можно увеличить масштаб до мощной машины с памятью, которая больше или равна 8 ГБ. |
    | Если сообщение об ошибке содержит строку "Нуллпоинтерреференце", это может быть временной ошибкой. | Повторите операцию. Если проблема не исчезнет, обратитесь в службу поддержки. |

### <a name="error-code-parquetinvalidfile"></a>Код ошибки: Паркуетинвалидфиле

- **Сообщение.** `File is not a valid Parquet file.`

- **Причина**: это проблема с файлом Parquet.

- **Рекомендация**: Проверьте, является ли входные данные допустимым файлом Parquet.


### <a name="error-code-parquetnotsupportedtype"></a>Код ошибки: Паркуетнотсуппортедтипе

- **Сообщение.** `Unsupported Parquet type. PrimitiveType: %primitiveType; OriginalType: %originalType;.`

- **Причина**: формат Parquet не поддерживается в фабрике данных Azure.

- **Рекомендация**: дважды проверьте исходные данные, перейдя к [поддерживаемым форматам файлов и кодекам сжатия по действию копирования в фабрике данных Azure](./supported-file-formats-and-compression-codecs.md).


### <a name="error-code-parquetmisseddecimalprecisionscale"></a>Код ошибки: ПаркуетмисседдеЦималпреЦисионскале

- **Сообщение.** `Decimal Precision or Scale information is not found in schema for column: %column;.`

- **Причина**: была проанализирована точность и масштаб числа, но эти сведения не были предоставлены.

- **Рекомендация**: источник не возвращает правильные сведения о точности и масштабировании. Просмотрите сведения в столбце проблемы.


### <a name="error-code-parquetinvaliddecimalprecisionscale"></a>Код ошибки: ПаркуетинвалиддеЦималпреЦисионскале

- **Сообщение**: `Invalid Decimal Precision or Scale. Precision: %precision; Scale: %scale;.`

- **Причина.** Недопустимая схема.

- **Рекомендация**: проверьте наличие точности и масштаба в столбце "проблемы".


### <a name="error-code-parquetcolumnnotfound"></a>Код ошибки: Паркуетколумннотфаунд

- **Сообщение.** `Column %column; does not exist in Parquet file.`

- **Причина**: исходная схема не соответствует схеме приемника.

- **Рекомендация**: проверьте сопоставления в действии. Убедитесь, что исходный столбец может быть сопоставлен с правильным столбцом приемника.


### <a name="error-code-parquetinvaliddataformat"></a>Код ошибки: Паркуетинвалиддатаформат

- **Сообщение.** `Incorrect format of %srcValue; for converting to %dstType;.`

- **Причина**: данные не могут быть преобразованы в тип, указанный в поле сопоставления. источник.

- **Рекомендация**: дважды проверьте исходные данные или укажите правильный тип данных для этого столбца в сопоставлении столбцов действия копирования. Дополнительные сведения см. [в разделе Поддерживаемые форматы файлов и кодеки сжатия по действию копирования в фабрике данных Azure](./supported-file-formats-and-compression-codecs.md).


### <a name="error-code-parquetdatacountnotmatchcolumncount"></a>Код ошибки: Паркуетдатакаунтнотматчколумнкаунт

- **Сообщение.** `The data count in a row '%sourceColumnCount;' does not match the column count '%sinkColumnCount;' in given schema.`

- **Причина**: несоответствие между числом исходных столбцов и числом столбцов приемника.

- **Рекомендация**: дважды проверьте, что количество исходных столбцов совпадает с числом столбцов приемника в "Mapping".


### <a name="error-code-parquetdatatypenotmatchcolumntype"></a>Код ошибки: Паркуетдататипенотматчколумнтипе

- **Сообщение.** `The data type %srcType; is not match given column type %dstType; at column '%columnIndex;'.`

- **Причина**: данные из источника не могут быть преобразованы в тип, определенный в приемнике.

- **Рекомендация**: укажите правильный тип в сопоставлении. Sink.


### <a name="error-code-parquetbridgeinvaliddata"></a>Код ошибки: Паркуетбриджеинвалиддата

- **Сообщение.** `%message;`

- **Причина**: значение данных превысило предел.

- **Рекомендация**: повторите операцию. Если проблема будет повторяться, свяжитесь с нами.


### <a name="error-code-parquetunsupportedinterpretation"></a>Код ошибки: Паркуетунсуппортединтерпретатион

- **Сообщение.** `The given interpretation '%interpretation;' of Parquet format is not supported.`

- **Причина**. Этот сценарий не поддерживается.

- **Рекомендация**.  Параметр ParquetInterpretFor не должен иметь значение sparkSql.


### <a name="error-code-parquetunsupportfilelevelcompressionoption"></a>Код ошибки: Паркуетунсуппортфилелевелкомпрессионоптион

- **Сообщение.** `File level compression is not supported for Parquet.`

- **Причина**. Этот сценарий не поддерживается.

- **Рекомендация**: удалите "компрессионтипе" в полезных данных.


### <a name="error-code-usererrorjniexception"></a>Код ошибки: Усерерроржниексцептион

- **Сообщение.** `Cannot create JVM: JNI return code [-6][JNI call failed: Invalid arguments.]`

- **Причина**: невозможно создать виртуальная машина Java (виртуальной машины Java), так как заданы недопустимые (глобальные) аргументы.

- **Рекомендация**: Войдите на компьютер, на котором размещается *каждый узел* в локальной среде IR. Убедитесь, что системная переменная задана правильно, следующим образом: `_JAVA_OPTIONS "-Xms256m -Xmx16g" with memory bigger than 8 G` . Перезапустите все ИНФРАКРАСные узлы, а затем повторно запустите конвейер.


### <a name="arithmetic-overflow"></a>Переполнение при арифметической операции

- **Симптомы**: при копировании файлов Parquet возникла ошибка: `Message = Arithmetic Overflow., Source = Microsoft.DataTransfer.Common`

- **Причина**: в настоящее время при копировании файлов из Oracle в Parquet поддерживаются только десятичные значения точности <= 38 и длины целочисленной части <= 20. 

- **Решение**. в качестве обходного пути можно преобразовать любые столбцы с этой проблемой в VARCHAR2.


### <a name="no-enum-constant"></a>Нет константы перечисления

- **Симптомы**: при копировании данных в формат Parquet произошла ошибка: `java.lang.IllegalArgumentException:field ended by &apos;;&apos;` , или: `java.lang.IllegalArgumentException:No enum constant org.apache.parquet.schema.OriginalType.test` .

- **Причина**. 

    Эта ошибка может быть вызвана пробелами или неподдерживаемыми специальными символами (например,,; {} () \n\t =) в имени столбца, так как Parquet не поддерживает такой формат. 

    Например, имя столбца, например *contoso (Test)* , будет анализировать тип в квадратных скобках из [кода](https://github.com/apache/parquet-mr/blob/master/parquet-column/src/main/java/org/apache/parquet/schema/MessageTypeParser.java) `Tokenizer st = new Tokenizer(schemaString, " ;{}()\n\t");` . Ошибка возникает из-за отсутствия такого типа "Test".

    Чтобы проверить поддерживаемые типы, перейдите на сайт GitHub [Apache/Parquet-MR](https://github.com/apache/parquet-mr/blob/master/parquet-column/src/main/java/org/apache/parquet/schema/OriginalType.java).

- **Решение**. 

    Проверьте наличие следующих двух проверок:
    - В имени столбца приемника есть пробелы.
    - В качестве имени столбца используется первая строка с пробелами.
    - Тип Оригиналтипе поддерживается. Старайтесь не использовать эти специальные символы: `,;{}()\n\t=` . 


## <a name="rest"></a>REST

### <a name="error-code-restsinkcallfailed"></a>Код ошибки: Рестсинккаллфаилед

- **Сообщение.** `Rest Endpoint responded with Failure from server. Check the error from server:%message;`

- **Причина**: Эта ошибка возникает, когда фабрика данных Azure обращается к КОНЕЧНОЙ точке RESTful по протоколу HTTP, и операция запроса завершается ошибкой.

- **Рекомендация**: Проверьте код состояния HTTP или сообщение в сообщении об ошибке и устраните проблему с удаленным сервером.

### <a name="unexpected-network-response-from-the-rest-connector"></a>Непредвиденный сетевой ответ от соединителя RESTFUL

- **Симптомы**. Конечная точка иногда получает непредвиденный ответ (400, 401, 403, 500) из соединителя RESTful.

- **Причина**. Соединитель источника RESTful использует URL-адрес и метод HTTP/заголовок/текст из связанной службы, набора данных или источника копирования в качестве параметров при формировании HTTP-запроса. Скорее всего, эта неполадка вызвана некоторыми ошибками в одном или нескольких указанных параметрах.

- **Решение**. 
    - Используйте "изогнутое" в окне командной строки, чтобы определить, является ли параметр причиной ("**принять** " и " **User-Agent** " всегда должен быть включен):
    
      `curl -i -X <HTTP method> -H <HTTP header1> -H <HTTP header2> -H "Accept: application/json" -H "User-Agent: azure-data-factory/2.0" -d '<HTTP body>' <URL>`
      
      Если команда возвращает один и тот же непредвиденный ответ, исправьте приведенные выше параметры с помощью параметра "листывание", пока не вернет ожидаемый ответ. 

      Для более подробного использования команды можно также использовать параметр ««-Help».

    - Если только соединитель RESTFUL фабрики данных возвращает непредвиденный ответ, обратитесь в службу поддержки Майкрософт для дальнейшего устранения неполадок.
    
    - Обратите внимание, что "изогнутое" может оказаться непригодным для воспроизведения проблемы проверки SSL-сертификата. В некоторых сценариях команда "перелистывание" была выполнена успешно без возникновения каких-либо проблем с проверкой SSL-сертификата. Но если один и тот же URL-адрес выполняется в браузере, на самом деле сертификат SSL не возвращается, чтобы клиент установил отношение доверия с сервером.

      В предыдущем случае рекомендуется использовать такие средства, как **posts** и **Fiddler** .


## <a name="sftp"></a>SFTP

#### <a name="error-code-sftpoperationfail"></a>Код ошибки: Сфтпоператионфаил

- **Сообщение.** `Failed to '%operation;'. Check detailed error from SFTP.`

- **Причина**: проблема с операцией SFTP.

- **Рекомендация**: Проверьте сведения об ошибке в SFTP.


### <a name="error-code-sftprenameoperationfail"></a>Код ошибки: Сфтпренамеоператионфаил

- **Сообщение.** `Failed to rename the temp file. Your SFTP server doesn't support renaming temp file, set "useTempFileRename" as false in copy sink to disable uploading to temp file.`

- **Причина**: сервер SFTP не поддерживает переименование временного файла.

- **Рекомендация**: задайте для параметра "усетемпфилеренаме" значение false в приемнике копирования, чтобы отключить отправку во временный файл.


### <a name="error-code-sftpinvalidsftpcredential"></a>Код ошибки: Сфтпинвалидсфтпкредентиал

- **Сообщение.** `Invalid SFTP credential provided for '%type;' authentication type.`

- **Причина**: содержимое закрытого ключа извлекается из хранилища ключей Azure или пакета SDK, но не кодируется правильно.

- **Рекомендация**.  

    Если содержимое закрытого ключа находится в хранилище ключей, исходный файл ключа может работать при его передаче непосредственно в связанную службу SFTP.

    Дополнительные сведения см. в статье [копирование данных с сервера SFTP и обратно с помощью фабрики данных Azure](./connector-sftp.md#use-ssh-public-key-authentication). Содержимое закрытого ключа — это содержимое закрытого ключа SSH в кодировке Base64.

    Кодирование *всего* исходного файла закрытого ключа в кодировке Base64 и сохранение закодированной строки в хранилище ключей. Исходный закрытый ключ является файлом, который может работать с связанной службой SFTP, если в файле выбрано значение **Отправить** .

    Ниже приведены некоторые примеры, которые можно использовать для создания строки.

    - Использовать код C#:

        ```
        byte[] keyContentBytes = File.ReadAllBytes(Private Key Path);
        string keyContent = Convert.ToBase64String(keyContentBytes, Base64FormattingOptions.None);
        ```

    - Использовать код Python:

        ```
        import base64
        rfd = open(r'{Private Key Path}', 'rb')
        keyContent = rfd.read()
        rfd.close()
        print base64.b64encode(Key Content)
        ```

    - Используйте сторонние средства преобразования в формат Base64. Рекомендуется использовать средство [кодирования в формат Base64](https://www.base64encode.org/) .

- **Причина**: выбран неправильный формат содержимого ключа.

- **Рекомендация**.  

    Формат PKCS # 8 закрытый ключ SSH (начинающийся с "-----BEGIN ENCRYPTed закрытый ключ-----") сейчас не поддерживается для доступа к серверу SFTP в фабрике данных. 

    Чтобы преобразовать ключ в формат традиционного SSH-ключа, начиная с "-----начать работу с ЗАКРЫТым ключом RSA-----", выполните следующие команды:

    ```
    openssl pkcs8 -in pkcs8_format_key_file -out traditional_format_key_file
    chmod 600 traditional_format_key_file
    ssh-keygen -f traditional_format_key_file -p
    ```

- **Причина**: недопустимые учетные данные или содержимое закрытого ключа.

- **Рекомендация**: чтобы проверить правильность файла ключа или пароля, дважды проверьте их с помощью таких средств, как WinSCP.

### <a name="sftp-copy-activity-failed"></a>Сбой действия SFTP Copy

- **Симптомы**: 
  * Код ошибки: Усерерроринвалидколумнмаппингколумннотфаунд 
  * Сообщение об ошибке: `Column 'AccMngr' specified in column mapping cannot be found in source data.`

- **Причина**: источник не включает столбец с именем "аккмнгр".

- **Решение**. чтобы определить, существует ли столбец "аккмнгр", проверьте конфигурацию набора данных, сопоставив столбец целевого набора данных.


### <a name="error-code-sftpfailedtoconnecttosftpserver"></a>Код ошибки: Сфтпфаиледтоконнекттосфтпсервер

- **Сообщение.** `Failed to connect to SFTP server '%server;'.`

- **Причина**: Если сообщение об ошибке содержит строку "истекло время ожидания операции чтения сокета после 30 000 миллисекунд", одна из возможных причин заключается в том, что для SFTP-сервера используется неверный тип связанной службы. Например, вы можете использовать связанную службу FTP для подключения к SFTP-серверу.

- **Рекомендация**: проверьте порт целевого сервера. По умолчанию SFTP использует порт 22.

- **Причина**: Если сообщение об ошибке содержит строку "ответ сервера не содержит идентификатор SSH Protocol", одна из возможных причин заключается в том, что SFTP-сервер регулирует соединение. Фабрика данных будет создавать несколько подключений для скачивания с сервера SFTP параллельно, и иногда это приведет к регулированию сервера SFTP. Обычно различные серверы возвращают различные ошибки при регулировании.

- **Рекомендация**.  

    Укажите максимальное число одновременных подключений к набору данных SFTP, равное 1, и повторно запустите действие копирования. Если действие выполнено, можно убедиться в том, что это вызвано регулированием.

    Если вы хотите повысить пропускную способность, обратитесь к администратору SFTP, чтобы увеличить число одновременных подключений, или выполните одно из следующих действий.

    * Если используется локальная среда IR, добавьте IP-адрес саморазмещенного ИНФРАКРАСного компьютера в список разрешений.
    * Если вы используете Azure IR, добавьте [Azure Integration Runtime IP-адреса](./azure-integration-runtime-ip-addresses.md). Если вы не хотите добавлять диапазон IP-адресов в список разрешенных серверов SFTP, используйте вместо этого локальную IR.

## <a name="sharepoint-online-list"></a>Список SharePoint Online

### <a name="error-code-sharepointonlineauthfailed"></a>Код ошибки: Шарепоинтонлинеаусфаилед

- **Сообщение.** `The access token generated failed, status code: %code;, error message: %message;.`

- **Причина**: идентификатор субъекта-службы и ключ могут быть заданы неправильно.

- **Рекомендация**. Проверьте зарегистрированное приложение (идентификатор субъекта-службы) и ключ, чтобы проверить, правильно ли заданы.


## <a name="xml-format"></a>Формат XML

### <a name="error-code-xmlsinknotsupported"></a>Код ошибки: Ксмлсинкнотсуппортед

- **Сообщение.** `Write data in XML format is not supported yet, choose a different format!`

- **Причина**: набор данных XML использовался в качестве набора данных-приемника в действии копирования.

- **Рекомендация**: используйте набор данных в другом формате из набора данных приемника.


### <a name="error-code-xmlattributecolumnnameconflict"></a>Код ошибки: Ксмлаттрибутеколумннамеконфликт

- **Сообщение.** `Column names %attrNames;' for attributes of element '%element;' conflict with that for corresponding child elements, and the attribute prefix used is '%prefix;'.`

- **Причина**: использовался префикс атрибута, вызвавший конфликт.

- **Рекомендация**: задайте другое значение для свойства "аттрибутепрефикс".


### <a name="error-code-xmlvaluecolumnnameconflict"></a>Код ошибки: Ксмлвалуеколумннамеконфликт

- **Сообщение.** `Column name for the value of element '%element;' is '%columnName;' and it conflicts with the child element having the same name.`

- **Причина**: одно из имен дочерних элементов было использовано в качестве имени столбца для значения элемента.

- **Рекомендация**: задайте другое значение для свойства "valueColumn".


### <a name="error-code-xmlinvalid"></a>Код ошибки: Ксмлинвалид

- **Сообщение.** `Input XML file '%file;' is invalid with parsing error '%error;'.`

- **Причина**: входной XML-файл имеет некорректный формат.

- **Рекомендация**: ИСПРАВЬТЕ XML-файл, чтобы сделать его правильным.


## <a name="general-copy-activity-error"></a>Общая ошибка действия копирования

### <a name="error-code-jrenotfound"></a>Код ошибки: Жренотфаунд

- **Сообщение.** `Java Runtime Environment cannot be found on the Self-hosted Integration Runtime machine. It is required for parsing or writing to Parquet/ORC files. Make sure Java Runtime Environment has been installed on the Self-hosted Integration Runtime machine.`

- **Причина**: локальная IR не может найти среду выполнения Java. Для чтения определенных источников требуется среда выполнения Java.

- **Рекомендации**. Проверьте среду выполнения интеграции. см. раздел [Использование Integration Runtime с автономным размещением](./format-parquet.md#using-self-hosted-integration-runtime).


### <a name="error-code-wildcardpathsinknotsupported"></a>Код ошибки: Вилдкардпассинкнотсуппортед

- **Сообщение.** `Wildcard in path is not supported in sink dataset. Fix the path: '%setting;'.`

- **Причина**: в наборе данных приемника не поддерживаются значения с подстановочными знаками.

- **Рекомендация**: Проверьте набор данных приемника и перепишите путь, не используя подстановочное значение.


### <a name="fips-issue"></a>Выпуск FIPS

- **Симптомы**. сбой действия копирования на автономном IR-компьютере с поддержкой FIPS со следующим сообщением об ошибке: `This implementation is not part of the Windows Platform FIPS validated cryptographic algorithms.` 

- **Причина**: Эта ошибка может возникать при копировании данных с помощью соединителей, таких как хранилище BLOB-объектов Azure, SFTP и т. д. Федеральный стандарт обработки информации (FIPS) определяет определенный набор алгоритмов шифрования, которые разрешено использовать. Когда режим FIPS включен на компьютере, некоторые криптографические классы, от которых зависит действие копирования, блокируются в некоторых сценариях.

- **Решение**. Узнайте, [почему мы больше не рекомендуем "режим FIPS"](https://techcommunity.microsoft.com/t5/microsoft-security-baselines/why-we-8217-re-not-recommending-8220-fips-mode-8221-anymore/ba-p/701037), и оцените, можно ли отключить FIPS на своем собственном IR-компьютере.

    Кроме того, если вы хотите разрешить в фабрике данных Azure обход FIPS и сделать действие выполненным, выполните следующие действия.

    1. Откройте папку, в которой установлена локальная среда IR. Путь обычно является *C:\Program Files\Microsoft Integration Runtime \<IR version> \шаред*.

    2. Откройте файл *diawp.exe.config* , а затем в конце `<runtime>` раздела добавьте `<enforceFIPSPolicy enabled="false"/>` , как показано ниже:

        ![Снимок экрана раздела файла diawp.exe.config, в котором отключено FIPS.](./media/connector-troubleshoot-guide/disable-fips-policy.png)

    3. Сохраните файл, а затем перезапустите автономный IR-компьютер.


## <a name="next-steps"></a>Дальнейшие действия

Для получения дополнительных сведений об устранении неполадок воспользуйтесь следующими ресурсами:

*  [Блог о Фабрике данных](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Запросы на добавление функции в Фабрику данных](https://feedback.azure.com/forums/270578-data-factory)
*  [Видео по Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Microsoft Q&странице](/answers/topics/azure-data-factory.html)
*  [Форум Stack Overflow по Фабрике данных](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Сведения о Фабрике данных в Twitter](https://twitter.com/hashtag/DataFactory)