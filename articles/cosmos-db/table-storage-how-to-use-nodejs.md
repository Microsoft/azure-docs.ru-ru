---
title: Использование Хранилища таблиц Azure или API таблиц Azure Cosmos DB в Node.js
description: Хранение структурированных данных в облаке с помощью Хранилища таблиц Azure или API таблиц Azure Cosmos DB в Node.js.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: nodejs
ms.topic: sample
ms.date: 07/23/2020
author: sakash279
ms.author: akshanka
ms.custom: devx-track-js
ms.openlocfilehash: 2d40b70d49b1934c9dd2d911369245b1b2e4f2ff
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93079714"
---
# <a name="how-to-use-azure-table-storage-or-the-azure-cosmos-db-table-api-from-nodejs"></a>Как использовать в Node.js Хранилище таблиц Azure и API таблиц Azure Cosmos DB
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

В этой статье показано, как создавать таблицы, сохранять данные и выполнять операции CRUD с данными. Вы можете использовать службу таблиц Azure или API таблиц Azure Cosmos DB. Примеры кода написаны на языке Node.js.

## <a name="create-an-azure-service-account"></a>Создание учетной записи службы Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Create an Azure Storage account** (Создание учетной записи хранения Azure)

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Создание учетной записи API таблиц Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="configure-your-application-to-access-azure-storage-or-the-azure-cosmos-db-table-api"></a>Настройка приложения для доступа к службе хранилища Azure или API таблиц Azure Cosmos DB

Чтобы использовать службу хранилища Azure или Azure Cosmos DB, вам понадобится пакет SDK службы хранилища Azure для Node.js, который содержит набор вспомогательных библиотек, взаимодействующих со службами REST хранилища.

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Использование диспетчера пакета Node (NPM) для установки пакета

1. Используя интерфейс командной строки, например **PowerShell** (Windows), **Terminal** (Mac) или **Bash** (Unix), перейдите в папку, в которой вы создали приложение.
2. Введите в командной строке **npm install azure-storage** . Результат выполнения этой команды будет аналогичен следующему примеру:

   ```bash
    azure-storage@0.5.0 node_modules\azure-storage
    +-- extend@1.2.1
    +-- xmlbuilder@0.4.3
    +-- mime@1.2.11
    +-- node-uuid@1.4.3
    +-- validator@3.22.2
    +-- underscore@1.4.4
    +-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
    +-- xml2js@0.2.7 (sax@0.5.2)
    +-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)
   ```

3. Выполнив команду **ls** вручную, можно убедиться, что папка **node_modules** создана. В этой папке находится пакет **azure-storage** , содержащий библиотеки, необходимые для доступа к хранилищу.

### <a name="import-the-package"></a>Импорт пакета

Добавьте следующий код в начало файла **server.js** в приложении:

```javascript
var azure = require('azure-storage');
```

## <a name="add-your-connection-string"></a>Добавление строки подключения

Вы можете подключиться к учетной записи хранения Azure или учетной записи API таблиц Azure Cosmos DB. Получите строку подключения на основе типа используемой учетной записи.

### <a name="add-an-azure-storage-connection"></a>Добавление подключения к службе хранилища Azure

Модуль Azure считывает переменные среды AZURE_STORAGE_ACCOUNT и AZURE_STORAGE_ACCESS_KEY или AZURE_STORAGE_CONNECTION_STRING, чтобы получить информацию, необходимую для подключения к учетной записи хранения Azure. Если эти переменные среды не заданы, при вызове `TableService` необходимо указать сведения об учетной записи. Например, следующий код создает объект `TableService`.

```javascript
var tableSvc = azure.createTableService('myaccount', 'myaccesskey');
```

### <a name="add-an-azure-cosmos-db-connection"></a>Добавление подключения к Azure Cosmos DB

Чтобы добавить подключение к Azure Cosmos DB, создайте объект `TableService` и укажите имя учетной записи, первичный ключ и конечную точку. Эти значения можно скопировать из раздела **Параметры** > **Строка подключения** на портале Azure для вашей учетной записи Cosmos DB. Пример:

```javascript
var tableSvc = azure.createTableService('myaccount', 'myprimarykey', 'myendpoint');
```

## <a name="create-a-table"></a>Создание таблицы

Следующий код создает объект `TableService` и использует его для создания таблицы.

```javascript
var tableSvc = azure.createTableService();
```

Вызов `createTableIfNotExists` создает таблицу с определенным именем, если она еще не существует. В следующем примере создается новая таблица с именем "mytable", если она еще не создана:

```javascript
tableSvc.createTableIfNotExists('mytable', function(error, result, response){
  if(!error){
    // Table exists or created
  }
});
```

`result.created` имеет значение `true`, если создана новая таблица, и `false`, если таблица уже существует. `response` содержит информацию о запросе.

### <a name="apply-filters"></a>Применение фильтров

К операциям, выполняемым с помощью `TableService`, можно применить дополнительную фильтрацию. Фильтровать можно операции ведения журнала, автоматические повторы и т. д. Фильтры являются объектами, реализующими метод со следующей сигнатурой:

```javascript
function handle (requestOptions, next)
```

Выполнив предварительную обработку параметров запроса, метод должен вызвать функцию **next**, передав ему обратный вызов со следующей сигнатурой.

```javascript
function (returnObject, finalCallback, next)
```

В этой функции обратного вызова и после обработки `returnObject` (ответ на запрос к серверу) функция обратного вызова должна либо вызвать `next` (при наличии), чтобы продолжить обработку других фильтров, либо в противном случае просто вызвать `finalCallback` для завершения обращения к службе.

Два фильтра (`ExponentialRetryPolicyFilter` и `LinearRetryPolicyFilter`), которые позволяют реализовать логику повторных попыток, входят в состав пакета Azure SDK для Node.js. Следующий код создает объект `TableService`, который использует метод `ExponentialRetryPolicyFilter`:

```javascript
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var tableSvc = azure.createTableService().withFilter(retryOperations);
```

## <a name="add-an-entity-to-a-table"></a>Добавление сущности в таблицу

Чтобы добавить сущность, сначала создайте объект, который определяет свойства сущности. Все сущности должны содержать ключи **PartitionKey** и **RowKey**, которые выступают ее уникальными идентификаторами.

* **PartitionKey** — определяет секцию, в которой хранится сущность.
* **RowKey** — уникально определяет сущность в секции.

**PartitionKey** и **RowKey** должны быть строковыми значениями. Дополнительные сведения см. в статье [Understanding the Table Service Data Model](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model) (Общие сведения о модели данных службы таблиц).

Ниже приводится пример задания сущности. Параметр **dueDate** определяется как тип `Edm.DateTime`. Задание типа необязательно, типы будут выведены, если они не заданы.

```javascript
var task = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'},
  description: {'_':'take out the trash'},
  dueDate: {'_':new Date(2015, 6, 20), '$':'Edm.DateTime'}
};
```

> [!NOTE]
> Для каждой записи есть поле `Timestamp`, значение которого задается Azure при вставке или обновлении сущности.

Чтобы создать сущность, можно также использовать `entityGenerator`. В следующем примере код создает сущность для той же задачи с использованием `entityGenerator`.

```javascript
var entGen = azure.TableUtilities.entityGenerator;
var task = {
  PartitionKey: entGen.String('hometasks'),
  RowKey: entGen.String('1'),
  description: entGen.String('take out the trash'),
  dueDate: entGen.DateTime(new Date(Date.UTC(2015, 6, 20))),
};
```

Чтобы добавить сущность в таблицу, передайте объект сущности в метод `insertEntity`.

```javascript
tableSvc.insertEntity('mytable',task, function (error, result, response) {
  if(!error){
    // Entity inserted
  }
});
```

Если операция успешна, `result` содержит [ETag](https://en.wikipedia.org/wiki/HTTP_ETag) вставленной записи, а `response` — информацию об операции.

Пример ответа:

```javascript
{ '.metadata': { etag: 'W/"datetime\'2015-02-25T01%3A22%3A22.5Z\'"' } }
```

> [!NOTE]
> По умолчанию метод `insertEntity` не возвращает вставленную сущность как часть информации, содержащейся в `response`. Если вы хотите выполнить другие операции с этой сущностью или кэшировать информацию, необходимо, чтобы она была возвращена как часть `result`. Это можно сделать следующим образом, включив `echoContent`:
>
> `tableSvc.insertEntity('mytable', task, {echoContent: true}, function (error, result, response) {...}`

## <a name="update-an-entity"></a>Обновление сущности

Для обновления имеющейся сущности доступно несколько методов:

* `replaceEntity` — обновляет имеющуюся сущность путем ее замены.
* `mergeEntity` — обновляет сущность посредством объединения новых значений свойств с имеющейся сущностью.
* `insertOrReplaceEntity` — обновляет имеющуюся сущность путем ее замены. Если сущность не существует, будет вставлена новая сущность.
* `insertOrMergeEntity` — обновляет сущность посредством объединения новых значений свойств с имеющейся сущностью. Если сущность не существует, будет вставлена новая сущность.

В следующем примере показано обновление сущности с помощью метода `replaceEntity`.

```javascript
tableSvc.replaceEntity('mytable', updatedTask, function(error, result, response){
  if(!error) {
    // Entity updated
  }
});
```

> [!NOTE]
> По умолчанию обновление сущности не проверяет, были ли обновляемые данные ранее изменены другим процессом. Поддержка одновременных обновлений:
>
> 1. Получите ETag обновляемого объекта. Он возвращается в составе `response` для всех операций с сущностями и может быть извлечен с помощью `response['.metadata'].etag`.
> 2. При выполнении операции обновления с сущностью предварительно добавьте информацию ETag, извлеченную для новой сущности. Пример:
>
>       entity2['.metadata'].etag = currentEtag;
> 3. Выполните операцию обновления. Если сущность была изменена с момента получения значения ETag, например, другим экземпляром вашего приложения, будет возвращена ошибка `error`, указывающая, что определенное в запросе условие обновления не выполнено.
>
>

Если при использовании `replaceEntity` и `mergeEntity` сущность, которая обновляется, не существует, то произойдет сбой операции обновления. Таким образом, если вы хотите сохранить сущность независимо от того, существует она уже или нет, используйте `insertOrReplaceEntity` или `insertOrMergeEntity`.

`result` содержит значение **Etag** обновленной сущности в случае успешного выполнения операций.

## <a name="work-with-groups-of-entities"></a>Работа с группами сущностей

Иногда имеет смысл отправлять совместно несколько операций в пакете для атомарной обработки сервером. Чтобы сделать это, используйте класс `TableBatch` для создания пакета, а затем метод `executeBatch` из `TableService` для выполнения пакетных операций.

 В следующем примере показана отправка двух сущностей в пакете:

```javascript
var task1 = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'},
  description: {'_':'Take out the trash'},
  dueDate: {'_':new Date(2015, 6, 20)}
};
var task2 = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '2'},
  description: {'_':'Wash the dishes'},
  dueDate: {'_':new Date(2015, 6, 20)}
};

var batch = new azure.TableBatch();

batch.insertEntity(task1, {echoContent: true});
batch.insertEntity(task2, {echoContent: true});

tableSvc.executeBatch('mytable', batch, function (error, result, response) {
  if(!error) {
    // Batch completed
  }
});
```

В успешных пакетных операциях `result` содержит информацию о всех операциях в пакете.

### <a name="work-with-batched-operations"></a>Работа с пакетными операциями

Операции, добавленные в пакет, можно проверить, просмотрев свойство `operations`. Для работы с операциями можно также использовать следующие методы:

* **clear** — удаляет все операции из пакета.
* **getOperations** — получает операцию из пакета.
* **hasOperations** — возвращает значение true, если пакет содержит операции.
* **removeOperations** — удаляет операцию.
* **size** — возвращает количество операций в пакете.

## <a name="retrieve-an-entity-by-key"></a>Получение сущности по ключу

Чтобы возвратить определенную сущность на основе значений ключей **PartitionKey** и **RowKey**, используйте метод **retrieveEntity**.

```javascript
tableSvc.retrieveEntity('mytable', 'hometasks', '1', function(error, result, response){
  if(!error){
    // result contains the entity
  }
});
```

После завершения этой операции `result` будет содержать сущность.

## <a name="query-a-set-of-entities"></a>Запрос набора сущностей

Чтобы запросить таблицу, используйте объект **TableQuery** для создания выражения запроса с помощью следующих предложений:

* **select** — поля, возвращаемые из запроса.
* **where** — предложение where.

  * **and** — условие `and` в предложении where.
  * **or** — условие `or` в предложении where.
* **top** — количество извлекаемых элементов.

Следующий пример собирает запрос, который возвращает первые пять элементов с использованием ключа PartitionKey со значением "hometasks".

```javascript
var query = new azure.TableQuery()
  .top(5)
  .where('PartitionKey eq ?', 'hometasks');
```

Так как параметр **select** не используется, возвращаются все поля. Чтобы выполнить запрос сущности в таблице, используйте **queryEntities**. В следующем примере используется запрос для возврата сущностей из таблицы 'mytable'.

```javascript
tableSvc.queryEntities('mytable',query, null, function(error, result, response) {
  if(!error) {
    // query was successful
  }
});
```

В случае успешного выполнения `result.entries` будет содержать массив сущностей, соответствующих запросу. Если запросу не удалось вернуть все сущности, `result.continuationToken` не будет иметь значение *NULL* и его можно будет использовать в качестве третьего параметра **queryEntities** для получения других результатов. В начальном запросе третий параметр должен иметь значение *null* .

### <a name="query-a-subset-of-entity-properties"></a>Запрос подмножества свойств сущности

Запрос к таблице может получить лишь несколько полей сущности.
Этот позволяет снизить потребление пропускной способности и может повысить производительность запросов, особенно для крупных сущностей. С помощью предложения **select** передайте имена возвращаемых полей. Например, следующий запрос возвратит только поля **description** и **dueDate**.

```javascript
var query = new azure.TableQuery()
  .select(['description', 'dueDate'])
  .top(5)
  .where('PartitionKey eq ?', 'hometasks');
```

## <a name="delete-an-entity"></a>Удаление сущности

Сущность можно удалить с помощью ее ключей раздела и строки. В этом примере объект **task1** содержит значения ключей **RowKey** и **PartitionKey** удаляемой сущности. Затем этот объект передается в метод **deleteEntity** .

```javascript
var task = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'}
};

tableSvc.deleteEntity('mytable', task, function(error, response){
  if(!error) {
    // Entity deleted
  }
});
```

> [!NOTE]
> Следует рассмотреть использование тегов ETag при удалении элементов, чтобы гарантировать отсутствие в них изменений, внесенных другим процессом. Сведения об использовании тегов ETag см. в разделе [Обновление сущности](#update-an-entity).
>
>

## <a name="delete-a-table"></a>Удаление таблицы

Следующий код удаляет таблицу из учетной записи хранения.

```javascript
tableSvc.deleteTable('mytable', function(error, response){
    if(!error){
        // Table deleted
    }
});
```

Если неизвестно, существует ли таблица, используйте **deleteTableIfExists**.

## <a name="use-continuation-tokens"></a>Использование маркеров продолжения

При выполнении запросов к таблицам для получения больших объемов результатов следует искать маркеры продолжения. По вашему запросу может быть найден большой объем данных, который, возможно, не удастся реализовать, если не создать метод определения наличия маркера продолжения.

При наличии такого маркера объект **results**, возвращаемый при запросе сущностей, задает свойство `continuationToken`. В последствии его можно использовать при выполнении запроса для продолжения и перемещения между разделами и сущностями таблицы.

При выполнении запросов для экземпляра объекта запроса и функции обратного вызова можно указать параметр `continuationToken`.

```javascript
var nextContinuationToken = null;
dc.table.queryEntities(tableName,
    query,
    nextContinuationToken,
    function (error, results) {
        if (error) throw error;

        // iterate through results.entries with results

        if (results.continuationToken) {
            nextContinuationToken = results.continuationToken;
        }

    });
```

Если обратиться к объекту `continuationToken`, то вы обнаружите, что он имеет такие свойства, как `nextPartitionKey`, `nextRowKey` и `targetLocation`, которые можно использовать для итерации по всем результатам.

Можно также использовать `top` вместе с `continuationToken`, чтобы задать размер страницы.

## <a name="work-with-shared-access-signatures"></a>Работа с подписями общего доступа

Подписанные URL-адреса (SAS) — безопасный способ предоставить детализированный доступ к таблицам без указания имени или ключей своей учетной записи хранения. SAS часто используется для предоставления ограниченного доступа к данным, например, позволяет мобильному приложению запрашивать записи.

Надежное приложение, например облачная служба, создает подписанный URL-адрес с помощью метода **generateSharedAccessSignature** из **TableService** и передает этот адрес ненадежному или частично надежному приложению, например мобильному приложению. Подпись SAS создается с использованием политики, которая описывает даты начала и окончания срока действия SAS, а также уровень доступа, который предоставляется держателю подписи SAS.

В следующем примере создается новая общая политика, которая позволяет держателю подписи SAS запрашивать ('r') в таблице в течение 100 минут с момента своего создания.

```javascript
var startDate = new Date();
var expiryDate = new Date(startDate);
expiryDate.setMinutes(startDate.getMinutes() + 100);
startDate.setMinutes(startDate.getMinutes() - 100);

var sharedAccessPolicy = {
  AccessPolicy: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
    Start: startDate,
    Expiry: expiryDate
  },
};

var tableSAS = tableSvc.generateSharedAccessSignature('mytable', sharedAccessPolicy);
var host = tableSvc.host;
```

Обратите внимание, что также должна быть предоставлена информация об узле, так как она требуется владельцу SAS для совершения попыток доступа к таблице.

Клиентское приложение далее использует подпись SAS с помощью **TableServiceWithSAS** для выполнения операций с таблицей. Следующий пример выполняет подключение к таблице и выполняет запрос. Дополнительные сведения о формате подписанных URL-адресов для таблиц см. в статье об [использование подписанных URL-адресов SAS в службе хранилища Azure](../storage/common/storage-sas-overview.md).

```javascript
// Note in the following command, host is in the format: `https://<your_storage_account_name>.table.core.windows.net` and the tableSAS is in the format: `sv=2018-03-28&si=saspolicy&tn=mytable&sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D`;

var sharedTableService = azure.createTableServiceWithSas(host, tableSAS);
var query = azure.TableQuery()
  .where('PartitionKey eq ?', 'hometasks');

sharedTableService.queryEntities(query, null, function(error, result, response) {
  if(!error) {
    // result contains the entities
  }
});
```

Так как подписанный URL-адрес был создан только для выполнения запросов, при попытке вставки, обновления или удаления сущностей будет возвращена ошибка.

### <a name="access-control-lists"></a>Списки управления доступом

Можно также использовать список управления доступом (ACL) для задания политики доступа подписи SAS. Это может оказаться удобным, когда необходимо предоставить доступ к таблице нескольким клиентам, но с различной политикой доступа для каждого из них.

ACL реализуется с помощью массива политик доступа, каждая из которых связана со своим идентификатором. В следующем примере определяются две политики, по одной для пользователей user1 и user2:

```javascript
var sharedAccessPolicy = {
  user1: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
    Start: startDate,
    Expiry: expiryDate
  },
  user2: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.ADD,
    Start: startDate,
    Expiry: expiryDate
  }
};
```

В этом примере код получает текущий список ACL для таблицы **hometasks**, а затем добавляет новые политики с помощью **setTableAcl**. Такой подход допускает выполнение:

```javascript
var extend = require('extend');
tableSvc.getTableAcl('hometasks', function(error, result, response) {
if(!error){
    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
    tableSvc.setTableAcl('hometasks', newSignedIdentifiers, function(error, result, response){
      if(!error){
        // ACL set
      }
    });
  }
});
```

После задания списка управления доступом можно создать подписанный URL-адрес на основе идентификатора политики. В следующем примере создается новая подпись SAS для пользователя 'user2':

```javascript
tableSAS = tableSvc.generateSharedAccessSignature('hometasks', { Id: 'user2' });
```

## <a name="next-steps"></a>Дальнейшие действия

Для получения дополнительных сведений см. следующие ресурсы.

* [Обозреватель хранилищ Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.
* Репозиторий [пакета SDK службы хранилища Azure для Node](https://github.com/Azure/azure-storage-node) на сайте GitHub.
* [Azure для разработчиков Node.js](/azure/developer/javascript/)
* [Создание веб-приложений Node.js в Azure](../app-service/quickstart-nodejs.md)
* [Создание и развертывание приложения Node.js в облачной службе Azure](../cloud-services/cloud-services-nodejs-develop-deploy-app.md) (с помощью Windows PowerShell)