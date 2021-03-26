---
title: Добавление веб-каналов данных из разных источников в помощник по метрикам
titleSuffix: Azure Cognitive Services
description: Добавление различных каналов данных в помощник по метрикам
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: mbullwin
ms.openlocfilehash: 4fd01256d94fbcb18fe8437be00c84e49d98f7d0
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105606153"
---
# <a name="add-data-feeds-from-different-data-sources-to-metrics-advisor"></a>Добавление веб-каналов данных из разных источников данных в помощник по метрикам

Используйте эту статью, чтобы найти параметры и требования для подключения различных типов источников данных к помощнику по метрикам. Ознакомьтесь со сведениями о том, как подключить [свои данные](how-tos/onboard-your-data.md) , чтобы узнать о ключевых концепциях использования данных с помощью помощника по метрикам. \

## <a name="supported-authentication-types"></a>Поддерживаемые типы аутентификации

| Типы проверки подлинности | Описание |
| ---------------------|-------------|
|**Основной** | Необходимо иметь возможность предоставлять основные параметры для доступа к источникам данных. Например, строка подключения или ключ. Администраторы веб-канала данных могут просматривать эти учетные данные. |
| **азуреманажедидентити** | [Управляемые удостоверения](../../active-directory/managed-identities-azure-resources/overview.md) для ресурсов Azure — это функция Azure Active Directory. Он предоставляет службы Azure с автоматически управляемым удостоверением в Azure AD. Удостоверение можно использовать для аутентификации в любой службе, поддерживающей проверку подлинности Azure AD.|
| **азуресклконнектионстринг**| Сохраните строку подключения AzureSQL как **сущность учетных данных** в помощнике по метрикам и используйте ее непосредственно при подключении данных метрик. Только администраторы сущности учетных данных могут просматривать эти учетные данные, но позволяют полномочным читателям создавать веб-каналы данных, не требуя сведений об этих учетных данных. |
| **DataLakeGen2SharedKey**| Храните ключ учетной записи Data Lake в качестве **сущности учетных данных** в советнике по метрикам и используйте его непосредственно при подключении данных метрик. Только администраторы сущности учетных данных могут просматривать эти учетные данные, но позволяют полномочным пользователям создавать веб-канал данных без необходимости знать сведения об учетных данных.|
| **Субъект-служба**| Сохраните субъект-службу в качестве **сущности учетных данных** в помощнике по метрикам и используйте его непосредственно при подключении данных метрик. Только администраторы сущности учетных данных могут просматривать учетные данные, но позволяют полномочным пользователям создавать веб-канал данных без необходимости знать сведения об учетных данных.|
| **Субъект-служба из хранилища ключей**|Сохраните субъект-службу в хранилище ключей как **сущность учетных данных** в Advisor и используйте его непосредственно при подключении данных метрик. Только администраторы **сущности учетных данных** могут просматривать учетные данные, но также не требуют от читателей создавать канал данных, не зная подробных учетных данных. |

## <a name="data-sources-supported-and-corresponding-authentication-types"></a>Поддерживаемые источники данных и соответствующие типы проверки подлинности


| Источники данных | Типы проверки подлинности |
|-------------| ---------------------|
|[**Azure Application Insights**](#appinsights)|  Basic |
|[**Хранилище BLOB-объектов Azure (JSON)**](#blob) | Basic<br>ManagedIdentity|
|[**Azure Cosmos DB (SQL)**](#cosmosdb) | Basic |
|[**Azure Data Explorer (Kusto)**](#kusto) | Basic<br>ManagedIdentity|
|[**Azure Data Lake Storage 2-го поколения**](#adl) | Basic<br>DataLakeGen2SharedKey<br>Субъект-служба<br>Субъект-служба из хранилища ключей<br> |
|[**База данных SQL Azure/SQL Server**](#sql) | Basic<br>ManagedIdentity<br>Субъект-служба<br>Субъект-служба из хранилища ключей<br>азуресклконнектионстринг
|[**Хранилище таблиц Azure**](#table) | Basic | 
|[**ElasticSearch**](#es) | Basic |
|[**HTTP-запрос**](#http) | Basic | 
|[**InfluxDB (Инфлукскл)**](#influxdb) | Basic |
|[**MongoDB**](#mongodb) | Basic |
|[**MySQL**](#mysql) | Basic |
|[**PostgreSQL**](#pgsql)| Basic|

Создайте сущность учетных данных * * и используйте ее для проверки подлинности в источниках данных. В следующих разделах указываются параметры, необходимые для *обычной* проверки подлинности. 

## <a name="span-idappinsightsazure-application-insightsspan"></a><span id="appinsights">Azure Application Insights</span>

* **Идентификатор приложения**: используется для идентификации этого приложения при использовании API Application Insights. Чтобы получить идентификатор приложения, сделайте следующее:

    1. В ресурсе Application Insights щелкните Доступ через API.

    2. Скопируйте идентификатор приложения, созданный в поле **идентификатор приложения** , в помощнике по метрикам. 
    
    Дополнительные сведения см. в [документации по службе Azure Bot](/azure/bot-service/bot-service-resources-app-insights-keys#application-id) .

* **Ключ API**: ключи API используются приложениями вне браузера для доступа к этому ресурсу. Чтобы получить ключ API, сделайте следующее:

    1. Из ресурса Application Insights щелкните Доступ через API.

    2. Щелкните Создание ключа API.

    3. Введите краткое описание, проверьте параметр считывания телеметрии и нажмите кнопку Создать ключ.

    4. Скопируйте ключ API в поле **ключа API** в помощнике по метрикам.

* **Запрос**. журналы Application Insights Azure построены на обозреватель данных Azure, а Azure Monitor запросы журналов используют версию того же языка запросов Kusto. [Документация по языку запросов Kusto](/azure/data-explorer/kusto/query/) содержит все сведения для языка и должна быть основным ресурсом для записи запроса в Application Insights. 


## <a name="span-idblobazure-blob-storage-jsonspan"></a><span id="blob">Хранилище BLOB-объектов Azure (JSON)</span>

* **Строка подключения**. сведения о получении этой строки см. в статье [строка подключения к](../../storage/common/storage-configure-connection-string.md#configure-a-connection-string-for-an-azure-storage-account) хранилищу BLOB-объектов Azure.

* **Контейнер**: помощник по метрикам ждет, что данные временных рядов хранятся в виде файлов BLOB-объектов (один большой двоичный объект на метку времени) в одном контейнере. Это поле имени контейнера.

* **Шаблон большого двоичного объекта**: это шаблон имен файлов BLOB-объектов. Например: `/%Y/%m/X_%Y-%m-%d-%h-%M.json`. Поддерживаются следующие параметры:
  * `%Y` год в формате `yyyy`
  * `%m` — месяц в формате `MM`
  * `%d` — день в формате `dd`
  * `%h` — Это час в формате `HH`
  * `%M` представляет собой минуты в формате `mm`

* **Версия формата JSON**: определяет схему данных в файлах JSON. Сейчас помощник по метрикам поддерживает две версии:
  
  * v1 (значение по умолчанию)

      Принимаются только *имена* и *значения* метрик. Например:
    
      ``` JSON
      {"count":11, "revenue":1.23}
      ```

  * Версия 2

      Также принимаются *измерения* метрик и *отметка времени* . Например:
      
      ``` JSON
      [
        {"date": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23},
        {"date": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}
      ]
      ```

Для каждого JSON-файла допускается только одна метка времени. 

## <a name="span-idcosmosdbazure-cosmos-db-sqlspan"></a><span id="cosmosdb">Azure Cosmos DB (SQL)</span>

* **Строка подключения**: строка подключения для доступа к Azure Cosmos DB. Его можно найти в ресурсе Cosmos DB в поле **ключи**. 
* **База данных**: база данных для запроса. Это можно найти на странице **Обзор** в разделе **контейнеры** .
* **Идентификатор коллекции**: идентификатор коллекции для запроса. Это можно найти на странице **Обзор** в разделе **контейнеры** .
* **SQL**-запрос — запрос SQL, позволяющий получать и формулировать данные в многомерных данных временных рядов. В запросе можно использовать `@StartTime` `@EndTime` переменные и. Они должны быть отформатированы: `yyyy-MM-dd HH:mm:ss` .

    Образец запроса:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = @StartTime
    ```
    
    Пример запроса для среза данных из 2019/12/12:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = '2019-12-12 00:00:00'
    ```

## <a name="span-idkustoazure-data-explorer-kustospan"></a><span id="kusto">Azure Data Explorer (Kusto)</span>

* **Строка подключения**. советник по метрикам поддерживает доступ к Azure обозреватель данных (Kusto) с помощью проверки подлинности приложения Azure AD. Вам потребуется создать и зарегистрировать приложение Azure AD, а затем авторизовать его для доступа к базе данных Azure обозреватель данных. Чтобы получить строку подключения, см. документацию по [Обозреватель данных Azure](/azure/data-explorer/provision-azure-ad-app) .

* **Запрос**. см. раздел [язык запросов Kusto](/azure/data-explorer/kusto/query) , чтобы получить и сформулировать данные в многомерных рядах данных. В запросе можно использовать `@StartTime` `@EndTime` переменные и. Они должны быть отформатированы: `yyyy-MM-dd HH:mm:ss` .

## <a name="span-idadlazure-data-lake-storage-gen2span"></a><span id="adl">Azure Data Lake Storage 2-го поколения</span>

* **Имя учетной записи**. имя учетной записи Azure Data Lake Storage 2-го поколения. Его можно найти в ресурсе учетной записи хранения Azure (Azure Data Lake Storage 2-го поколения) в **ключах доступа**.

* **Ключ учетной записи**: укажите имя учетной записи для доступа к Azure Data Lake Storage 2-го поколения. Это можно найти в ресурсе учетной записи хранения Azure (Azure Data Lake Storage 2-го поколения) в параметре **ключи доступа** .

* **Имя файловой системы (контейнер)**. Помощник по метрикам будет предполагать, что данные временных рядов хранятся в виде файлов BLOB-объектов (один большой двоичный объект на метку времени) в одном контейнере. Это поле имени контейнера. Его можно найти в экземпляре учетной записи хранения Azure (Azure Data Lake Storage 2-го поколения) и щелкнуть "контейнеры" в разделе "Служба BLOB-объектов".

* **Шаблон каталога**: это шаблон каталога для файла большого двоичного объекта. Например: */%и/%м/%д*. Поддерживаются следующие параметры:
  * `%Y` год в формате `yyyy`
  * `%m` — месяц в формате `MM`
  * `%d` — день в формате `dd`
  * `%h` — Это час в формате `HH`
  * `%M` представляет собой минуты в формате `mm`

* **Шаблон файла**: это файл шаблона файла большого двоичного объекта. Например: *X_% Y-% m-% d-% h-% M.json*. Поддерживаются следующие параметры:
  * `%Y` год в формате `yyyy`
  * `%m` — месяц в формате `MM`
  * `%d` — день в формате `dd`
  * `%h` — Это час в формате `HH`
  * `%M` представляет собой минуты в формате `mm`

В настоящее время помощник по метрикам поддерживает схему данных в файлах JSON следующим образом. Например:

``` JSON
[
  {"date": "2018-01-01T00:00:00Z", "market":"en-us", "count":11, "revenue":1.23},
  {"date": "2018-01-01T00:00:00Z", "market":"zh-cn", "count":22, "revenue":4.56}
]
```
<!--
## <span id="eventhubs">Azure Event Hubs</span>

* **Connection String**: This can be found in 'Shared access policies' in your Event Hubs instance. Also for the 'EntityPath', it could be found by clicking into your Event Hubs instance and clicking at 'Event Hubs' in 'Entities' blade. Items that listed can be input as EntityPath. 

* **Consumer Group**: A [consumer group](../../event-hubs/event-hubs-features.md#consumer-groups) is a view (state, position, or offset) of an entire event hub.
Event Hubs use the latest offset of a consumer group to consume (subscribe from) the data from data source. Therefore a dedicated consumer group should be created for one data feed in your Metrics Advisor instance.

* **Timestamp**: Metrics Advisor uses the Event Hubs timestamp as the event timestamp if the user data source does not contain a timestamp field.
The timestamp field must match one of these two formats:

    * "YYYY-MM-DDTHH:MM:SSZ" format;

    * Number of seconds or milliseconds from the epoch of 1970-01-01T00:00:00Z.

    No matter which timestamp field it left aligns to granularity.For example, if timestamp is "2019-01-01T00:03:00Z", granularity is 5 minutes, then Metrics Advisor aligns the timestamp to "2019-01-01T00:00:00Z". If the event timestamp is "2019-01-01T00:10:00Z",  Metrics Advisor uses the timestamp directly without any alignment.
-->
## <a name="span-idsqlazure-sql-database--sql-serverspan"></a><span id="sql">База данных SQL Azure | SQL Server</span>

* **Строка подключения**: советник по метрикам принимает [строку подключения стиля ADO.NET](/dotnet/framework/data/adonet/connection-string-syntax) для источника данных SQL Server.

    Пример строки подключения.

    ```
    Data Source=db-server.database.windows.net:[port];initial catalog=[database];User ID=[username];Password=[password];Connection Timeout=10ms;
    ```

* **Запрос**: SQL-запрос, позволяющий получать и формулировать данные в многомерных данных временных рядов. Вы можете использовать `@StartTime` переменную в запросе, чтобы получить ожидаемое значение метрик.

  * `@StartTime`: значение типа DateTime в формате `yyyy-MM-dd HH:mm:ss`

    Образец запроса:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = @StartTime
    ```
    
    Фактический запрос выполнен для среза данных 2019/12/12:
    
    ``` mssql
    select StartDate, JobStatusId, COUNT(*) AS JobNumber from IngestionJobs WHERE and StartDate = '2019-12-12 00:00:00'
    ```

## <a name="span-idtableazure-table-storagespan"></a><span id="table">хранилище таблиц Azure</span>

* **Строка подключения**: Создайте URL-адрес SAS (подпись общего доступа) и укажите здесь. Самый простой способ создать URL-адрес SAS — использовать портал Azure. С помощью портал Azure можно выполнить навигацию в графическом виде. Чтобы создать URL-адрес SAS с помощью портал Azure, сначала перейдите к учетной записи хранения, к которой вы хотите получить доступ, в разделе Параметры, а затем щелкните подпись общего доступа. Установите флажки "Таблица" и "объект", а затем нажмите кнопку Создать SAS и строку подключения. URL-адрес SAS службы таблиц необходим для копирования и заполнения текстового поля в рабочей области "помощник по метрикам".

* **Имя таблицы**: Укажите таблицу для запроса. Его можно найти в экземпляре учетной записи хранения Azure. Щелкните **таблицы** в разделе **Служба таблиц** .

* **Запрос** В запросе можно использовать `@StartTime` . `@StartTime` заменяется строкой формата yyyy-MM-ddTHH: mm: SS в скрипте. Совет. с помощью обозревателя службы хранилища Azure создайте запрос с указанным диапазоном времени и убедитесь, что он работает правильно, а затем выполните замену.

    ``` mssql
    date ge datetime'@StartTime' and date lt datetime'@EndTime'
    ```

## <a name="span-ideselasticsearchspan"></a><span id="es">Elasticsearch</span>

* **Узел**. Укажите главный узел кластера Elasticsearch.
* **Порт**. Укажите главный порт кластера Elasticsearch.
* **Заголовок авторизации**: Укажите значение заголовка авторизации кластера Elasticsearch.
* **Запрос**: укажите запрос для получения данных. Заполнитель @StartTime поддерживается. ( Например, когда принимается значение 2020-06-21T00:00:00Z, @StartTime = 2020-06-21T00:00:00)

## <a name="span-idhttphttp-requestspan"></a><span id="http">HTTP-запрос</span>

* **URL-адрес запроса**: URL-адрес HTTP, который может возвращать JSON. Заполнители% Y,% m,% d,% h,% M поддерживаются:% Y = year в формате гггг,% m = месяц в формате MM,% d = день в формате DD,% h = час в формате чч,% M = минута в формате mm. Например: `http://microsoft.com/ProjectA/%Y/%m/X_%Y-%m-%d-%h-%M`.
* **HTTP-метод запроса**: используйте Get или POST.
* **Заголовок запроса**: можно добавить обычную проверку подлинности. 
* **Полезные данные запроса**: поддерживаются только полезные данные JSON. @StartTimeВ полезных данных поддерживается заполнитель. Ответ должен быть в следующем формате JSON: [{"timestamp": "2018-01-01T00:00:00Z", "Market": "en-US", "Count": 11, "доход": 1,23}, {"timestamp": "2018-01-01T00:00:00Z", "DataMarket": "zh-CN", "Count": 22, "доход": 4,56}]. (например, когда принимается значение 2020-06-21T00:00:00Z, @StartTime = 2020-06-21T00:00:00.0000000 + 00:00)

## <a name="span-idinfluxdbinfluxdb-influxqlspan"></a><span id="influxdb">InfluxDB (Инфлукскл)</span>

* **Строка подключения**: строка подключения для доступа к InfluxDB.
* **База данных**: база данных для запроса.
* **Запрос**: запрос для получения и формирования данных многомерных временных рядов для приема.
* **Имя пользователя**: это необязательно для проверки подлинности. 
* **Пароль**. Это необязательный параметр для проверки подлинности. 

## <a name="span-idmongodbmongodbspan"></a><span id="mongodb">MongoDB</span>

* **Строка подключения**: строка подключения для доступа к MongoDB.
* **База данных**: база данных для запроса.
* **Команда**: команда для получения и формирования данных в многомерных временных рядах данных для приема.

## <a name="span-idmysqlmysqlspan"></a><span id="mysql">MySQL</span>

* **Строка подключения**: строка подключения для доступа к базе данных MySQL.
* **Запрос**: запрос для получения и формирования данных многомерных временных рядов для приема.

## <a name="span-idpgsqlpostgresqlspan"></a><span id="pgsql">PostgreSQL</span>

* **Строка подключения**: строка подключения для доступа к базе данных PostgreSQL DB.
* **Запрос**: запрос для получения и формирования данных многомерных временных рядов для приема.

## <a name="next-steps"></a>Дальнейшие действия

* При ожидании передачи данных метрик в систему прочитайте о [том, как управлять конфигурациями веб-каналов данных](how-tos/manage-data-feeds.md).
* При приеме данных метрик можно [настроить метрики и выполнить тонкую настройку обнаружения конфигурации](how-tos/configure-metrics.md).
