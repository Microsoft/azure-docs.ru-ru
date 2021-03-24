---
title: Использование службы таблиц службы хранилища Azure или API таблиц Azure Cosmos DB в PHP
description: Хранение структурированных данных в облаке с помощью Хранилища таблиц Azure или API таблиц Azure Cosmos DB в PHP.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: php
ms.topic: sample
ms.date: 07/23/2020
ms.openlocfilehash: 9d059c899e4a64d4d2c1b880b2a1d0f89258f33b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93079637"
---
# <a name="how-to-use-azure-storage-table-service-or-the-azure-cosmos-db-table-api-from-php"></a>Использование Хранилища таблиц службы хранилища Azure или API таблиц Azure Cosmos DB в PHP
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

В этой статье показано, как создавать таблицы, сохранять данные и выполнять операции CRUD с данными. Вы можете использовать либо службу таблиц Azure, либо API таблиц Azure Cosmos DB. В приведенных здесь примерах, написанных на PHP, используется [клиентская библиотека PHP таблиц службы хранилища Azure][download]. Здесь рассматриваются такие сценарии, как **создание и удаление таблицы**, а также **вставка, удаление и запрашивание сущностей в таблице**. Дополнительные сведения о службе таблиц Azure см. в разделе [Дальнейшие действия](#next-steps).

## <a name="create-an-azure-service-account"></a>Создание учетной записи службы Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Create an Azure Storage account** (Создание учетной записи хранения Azure)

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Создание учетной записи API таблиц Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="create-a-php-application"></a>Создание приложения PHP

Единственное требование при создании приложения PHP для доступа к хранилищу таблиц службы хранилища Azure или API-интерфейсу таблиц Azure Cosmos DB заключается в том, чтобы ссылаться в коде на классы пакета SDK azure-storage-table для PHP. Можно использовать любые средства разработки для создания приложения, включая программу "Блокнот".

В этом руководстве используются компоненты хранилища таблиц службы хранилища Azure или Azure Cosmos DB, которые могут вызываться локально из приложения PHP или в коде, работающем в веб-роли, рабочей роли или на веб-сайте Azure.

## <a name="get-the-client-library"></a>Получение клиентской библиотеки

1. Создайте файл с именем composer.json в корневой папке проекта и добавьте в него следующий код:
   ```json
   {
   "require": {
    "microsoft/azure-storage-table": "*"
   }
   }
   ```
2. Скачайте файл [composer.phar](https://getcomposer.org/composer.phar) в корневой каталог. 
3. Откройте командную строку и выполните следующую команду в корневом каталоге проекта:
   ```
   php composer.phar install
   ```
   Также можно перейти в [клиентскую библиотеку таблиц службы хранилища Azure для PHP](https://github.com/Azure/azure-storage-php/tree/master/azure-storage-table) на GitHub, чтобы клонировать исходный код.

## <a name="add-required-references"></a>Добавление обязательных ссылок

Чтобы использовать хранилище таблиц службы хранилища Azure или API-интерфейсы Azure Cosmos DB, необходимо указать следующее:

* Ссылка на файл автозагрузчика с использованием инструкции [require_once][require_once] и
* ссылку на все используемые классы.

В следующем примере показано, как включить файл автозагрузчика и добавить ссылку на класс **TableRestProxy**.

```php
require_once 'vendor/autoload.php';
use MicrosoftAzure\Storage\Table\TableRestProxy;
```

В приведенных ниже примерах всегда отображается оператор `require_once` , однако ссылки приводятся только на классы, которые необходимы для выполнения этого примера.

## <a name="add-your-connection-string"></a>Добавление строки подключения

Вы можете подключиться к учетной записи хранения Azure или учетной записи API таблиц Azure Cosmos DB. Получите строку подключения на основе типа используемой учетной записи.

### <a name="add-a-storage-table-service-connection"></a>Добавление подключения к хранилищу таблиц службы хранилища Azure

Для создания экземпляра клиента хранилища таблиц необходимо сначала сформировать правильную строку подключения. Для строки подключения к хранилищу таблиц используется следующий формат:

```php
$connectionString = "DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]"
```

### <a name="add-a-storage-emulator-connection"></a>Добавление подключения к эмулятору хранения

Для доступа к эмулятору хранения используйте следующий код:

```php
UseDevelopmentStorage = true
```

### <a name="add-an-azure-cosmos-db-connection"></a>Добавление подключения к Azure Cosmos DB

Для создания экземпляра клиента таблиц Azure Cosmos DB необходимо сначала сформировать правильную строку подключения. Для строки подключения Azure Cosmos DB используется следующий формат:

```php
$connectionString = "DefaultEndpointsProtocol=[https];AccountName=[myaccount];AccountKey=[myaccountkey];TableEndpoint=[https://myendpoint/]";
```

Для создания клиента хранилища таблиц Azure или клиента Azure Cosmos DB необходимо использовать класс **TableRestProxy**. Вы можете:

* передать строку подключения напрямую или
* использовать **CloudConfigurationManager (CCM)** для проверки нескольких внешних источников на наличие строки подключения:
  * по умолчанию предоставляется поддержка одного внешнего источника — переменных среды;
  * Можно добавить новые источники, расширив класс `ConnectionStringSource`.

В приведенных здесь примерах строка подключения передается напрямую.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;

$tableClient = TableRestProxy::createTableService($connectionString);
```

## <a name="create-a-table"></a>Создание таблицы

Объект **TableRestProxy** позволяет создать таблицу с помощью метода **createTable**. При создании таблицы можно задать время ожидания службы таблиц. (Дополнительные сведения о времени ожидания службы таблиц см. в статье [Setting Timeouts for Table Service Operations][table-service-timeouts] (Настройка времени ожидания для операций службы таблиц).)

```php
require_once 'vendor\autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create Table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    // Create table.
    $tableClient->createTable("mytable");
}
catch(ServiceException $e){
    $code = $e->getCode();
    $error_message = $e->getMessage();
    // Handle exception based on error codes and messages.
    // Error codes and messages can be found here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
}
```

Дополнительные сведения об ограничениях имен таблиц см. в статье [Understanding the Table Service Data Model][table-data-model] (Основные сведения о модели данных службы таблиц).

## <a name="add-an-entity-to-a-table"></a>Добавление сущности в таблицу

Чтобы добавить сущность в таблицу, создайте новый объект **Entity** и передайте его в **TableRestProxy->insertEntity**. Обратите внимание, что при создании сущности необходимо задать свойства `PartitionKey` и `RowKey`. Это уникальные идентификаторы сущности и значения, которые можно запросить гораздо быстрее, чем другие свойства сущности. Система использует `PartitionKey`, чтобы автоматически распределять сущности таблицы на несколько узлов хранилища. Сущности с одинаковым значением `PartitionKey` хранятся на одном узле. (Операции с несколькими сущностями, хранящимися на одном узле, выполняются эффективнее, чем с сущностями с разных узлов). `RowKey` — это уникальный идентификатор сущности в разделе.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$entity = new Entity();
$entity->setPartitionKey("tasksSeattle");
$entity->setRowKey("1");
$entity->addProperty("Description", null, "Take out the trash.");
$entity->addProperty("DueDate",
                        EdmType::DATETIME,
                        new DateTime("2012-11-05T08:15:00-08:00"));
$entity->addProperty("Location", EdmType::STRING, "Home");

try{
    $tableClient->insertEntity("mytable", $entity);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
}
```

Дополнительные сведения о типах и свойствах таблиц см. в статье [Understanding the Table Service Data Model][table-data-model] (Основные сведения о модели данных службы таблиц).

Класс **TableRestProxy** предлагает два альтернативных способа вставки сущностей: **insertOrMergeEntity** и **insertOrReplaceEntity**. Чтобы использовать эти методы, создайте новый объект **Entity** и передайте его в качестве параметра в любой из этих методов. Каждый метод вставит эту сущность, если она не существует. Если сущность уже существует, **insertOrMergeEntity** обновляет значения свойств, если свойства уже существуют, и добавляет новые свойства, если они не существуют, в то время как **insertOrReplaceEntity** полностью заменяет существующую сущность. Следующий пример показывает, как использовать **insertOrMergeEntity**. Если сущность со значением `PartitionKey` "tasksSeattle" и значением `RowKey` "1" еще не существует, она будет вставлена. Но если она была вставлена ранее (как показано в приведенном выше примере), будет обновлено свойство `DueDate` и добавлено свойство `Status`. Свойства `Description` и `Location` также будут обновлены, однако до таких значений, которые фактически равны предыдущим. Если эти два последних свойства не были добавлены, как показано в примере, но существовали в целевой сущности, их текущие значения остаются без изменений.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

//Create new entity.
$entity = new Entity();

// PartitionKey and RowKey are required.
$entity->setPartitionKey("tasksSeattle");
$entity->setRowKey("1");

// If entity exists, existing properties are updated with new values and
// new properties are added. Missing properties are unchanged.
$entity->addProperty("Description", null, "Take out the trash.");
$entity->addProperty("DueDate", EdmType::DATETIME, new DateTime()); // Modified the DueDate field.
$entity->addProperty("Location", EdmType::STRING, "Home");
$entity->addProperty("Status", EdmType::STRING, "Complete"); // Added Status field.

try    {
    // Calling insertOrReplaceEntity, instead of insertOrMergeEntity as shown,
    // would simply replace the entity with PartitionKey "tasksSeattle" and RowKey "1".
    $tableClient->insertOrMergeEntity("mytable", $entity);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="retrieve-a-single-entity"></a>Извлечение одной сущности

Метод **TableRestProxy->getEntity** позволяет получить одну сущность, запрашивая ее значения `PartitionKey` и `RowKey`. В приведенном ниже примере ключ раздела `tasksSeattle` и ключ строки `1` передаются в метод **getEntity**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    $result = $tableClient->getEntity("mytable", "tasksSeattle", 1);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$entity = $result->getEntity();

echo $entity->getPartitionKey().":".$entity->getRowKey();
```

## <a name="retrieve-all-entities-in-a-partition"></a>Получение всех сущностей в разделе

Запросы сущностей создаются с помощью фильтров (дополнительные сведения см. в статье [Querying Tables and Entities][filters] (Запросы к таблицам и сущностям)). Для получения всех сущностей в разделе используйте фильтр "PartitionKey eq *имя_раздела*". В следующем примере показано, как получить все сущности в разделе `tasksSeattle`, передав фильтр в метод **queryEntities**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$filter = "PartitionKey eq 'tasksSeattle'";

try    {
    $result = $tableClient->queryEntities("mytable", $filter);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$entities = $result->getEntities();

foreach($entities as $entity){
    echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
}
```

## <a name="retrieve-a-subset-of-entities-in-a-partition"></a>Получение диапазона сущностей в разделе

Процедуру, аналогичную приведенной в предыдущем примере, можно использовать для получения любого подмножества сущностей в разделе. Получаемое подмножество сущностей определяется используемым фильтром (дополнительные сведения см. в статье [Querying Tables and Entities][filters] (Запросы к таблицам и сущностям)). В следующем примере показано, как использовать фильтр для получения всех сущностей с определенным значением `Location` и значением `DueDate`, которое меньше указанной даты.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$filter = "Location eq 'Office' and DueDate lt '2012-11-5'";

try    {
    $result = $tableClient->queryEntities("mytable", $filter);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$entities = $result->getEntities();

foreach($entities as $entity){
    echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
}
```

## <a name="retrieve-a-subset-of-entity-properties"></a>Запрос подмножества свойств сущности

Запрос позволяет получить подмножество свойств сущности. Этот метод, который называется *проекцией*, снижает потребление полосы пропускания и может повысить производительность запросов, особенно для крупных сущностей. Чтобы задать свойство для извлечения, передайте имя этого свойства в метод **Query->addSelectField**. Этот метод можно вызывать несколько раз для добавления дополнительных свойств. После выполнения **TableRestProxy->queryEntities** возвращаемые сущности будут иметь только выбранные свойства. (Если вы хотите вернуть подмножество сущностей таблицы, используйте фильтр из приведенных выше запросов.)

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\QueryEntitiesOptions;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$options = new QueryEntitiesOptions();
$options->addSelectField("Description");

try    {
    $result = $tableClient->queryEntities("mytable", $options);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

// All entities in the table are returned, regardless of whether
// they have the Description field.
// To limit the results returned, use a filter.
$entities = $result->getEntities();

foreach($entities as $entity){
    $description = $entity->getProperty("Description")->getValue();
    echo $description."<br />";
}
```

## <a name="update-an-entity"></a>Обновление сущности

Имеющуюся сущность можно обновить, воспользовавшись методами **Entity->setProperty** и **Entity->addProperty** для сущности, а затем вызвав **TableRestProxy->updateEntity**. Следующий пример извлекает сущность, изменяет одно свойство, удаляет другое свойство и добавляет новое свойство. Обратите внимание, что свойство можно удалить, установив для него значение **null**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

$result = $tableClient->getEntity("mytable", "tasksSeattle", 1);

$entity = $result->getEntity();
$entity->setPropertyValue("DueDate", new DateTime()); //Modified DueDate.
$entity->setPropertyValue("Location", null); //Removed Location.
$entity->addProperty("Status", EdmType::STRING, "In progress"); //Added Status.

try    {
    $tableClient->updateEntity("mytable", $entity);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="delete-an-entity"></a>Удаление сущности

Чтобы удалить сущность, передайте имя таблицы и значения `PartitionKey` и `RowKey` сущности в метод **TableRestProxy->deleteEntity**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    // Delete entity.
    $tableClient->deleteEntity("mytable", "tasksSeattle", "2");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Для проверки на параллелизм можно задать Etag для удаляемой сущности, воспользовавшись методом **DeleteEntityOptions->setEtag** и передав объект **DeleteEntityOptions** в **deleteEntity** в качестве четвертого параметра.

## <a name="batch-table-operations"></a>Пакетные операции с таблицами

Метод **TableRestProxy->batch** позволяет выполнять несколько операций в одном запросе. Здесь шаблон включает в себя добавление операций в объект **BatchRequest** и последующую передачу объекта **BatchRequest** в метод **TableRestProxy->batch**. Чтобы добавить операцию в объект **BatchRequest** , можно вызвать любой из следующих методов несколько раз:

* **addInsertEntity** (добавляет операцию insertEntity)
* **addUpdateEntity** (добавляет операцию updateEntity)
* **addMergeEntity** (добавляет операцию mergeEntity)
* **addInsertOrReplaceEntity** (добавляет операцию insertOrReplaceEntity)
* **addInsertOrMergeEntity** (добавляет операцию insertOrMergeEntity)
* **addDeleteEntity** (добавляет операцию deleteEntity)

В следующем примере показано, как выполнить операции **insertEntity** и **deleteEntity** в одном запросе. 

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Table\Models\Entity;
use MicrosoftAzure\Storage\Table\Models\EdmType;
use MicrosoftAzure\Storage\Table\Models\BatchOperations;

// Configure a connection string for Storage Table service.
$connectionString = "DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]"

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

// Create list of batch operation.
$operations = new BatchOperations();

$entity1 = new Entity();
$entity1->setPartitionKey("tasksSeattle");
$entity1->setRowKey("2");
$entity1->addProperty("Description", null, "Clean roof gutters.");
$entity1->addProperty("DueDate",
                        EdmType::DATETIME,
                        new DateTime("2012-11-05T08:15:00-08:00"));
$entity1->addProperty("Location", EdmType::STRING, "Home");

// Add operation to list of batch operations.
$operations->addInsertEntity("mytable", $entity1);

// Add operation to list of batch operations.
$operations->addDeleteEntity("mytable", "tasksSeattle", "1");

try    {
    $tableClient->batch($operations);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Дополнительные сведения о пакетной обработке операций с таблицами см. в статье [Performing Entity Group Transactions][entity-group-transactions] (Выполнение транзакций для группы сущностей).

## <a name="delete-a-table"></a>Удаление таблицы

Наконец, чтобы удалить таблицу, следует передать имя таблицы в метод **TableRestProxy->deleteTable**.

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Table\TableRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create table REST proxy.
$tableClient = TableRestProxy::createTableService($connectionString);

try    {
    // Delete table.
    $tableClient->deleteTable("mytable");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Table-Service-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="next-steps"></a>Дальнейшие действия

Вы изучили основные сведения о хранилище таблиц Azure и Azure Cosmos DB. Дополнительные сведения можно найти по приведенным ниже ссылкам.

* [Обозреватель хранилищ Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.

* [Центр разработчиков PHP](https://azure.microsoft.com/develop/php/)

[download]: https://packagist.org/packages/microsoft/azure-storage-table
[require_once]: https://php.net/require_once
[table-service-timeouts]: /rest/api/storageservices/setting-timeouts-for-table-service-operations

[table-data-model]: /rest/api/storageservices/Understanding-the-Table-Service-Data-Model
[filters]: /rest/api/storageservices/Querying-Tables-and-Entities
[entity-group-transactions]: /rest/api/storageservices/Performing-Entity-Group-Transactions
