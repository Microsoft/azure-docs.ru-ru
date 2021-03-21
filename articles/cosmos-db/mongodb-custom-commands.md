---
title: Команды расширения MongoDB для управления данными в API Azure Cosmos DB для MongoDB
description: В этой статье описывается, как использовать команды расширения MongoDB для управления данными, хранящимися в API Azure Cosmos DB для MongoDB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/02/2021
ms.author: chrande
ms.custom: devx-track-js
ms.openlocfilehash: deba6696eb71287902fa3970ed2d83d0b09ac08d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101658492"
---
# <a name="use-mongodb-extension-commands-to-manage-data-stored-in-azure-cosmos-dbs-api-for-mongodb"></a>Используйте команды расширения MongoDB для управления данными, хранящимися в API-интерфейсе Azure Cosmos DB для MongoDB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Следующий документ содержит команды настраиваемого действия, относящиеся к API Azure Cosmos DB для MongoDB. Эти команды можно использовать для создания и получения ресурсов базы данных, относящихся к [модели Azure Cosmos DB емкости](account-databases-containers-items.md).

Используя API Azure Cosmos DB для MongoDB, вы можете воспользоваться преимуществами Cosmos DB таких, как глобальное распространение, автоматическое сегментирование, высокая доступность, гарантия задержки, автоматическое шифрование при хранении, резервное копирование и многое другое, сохраняя инвестиции в приложение MongoDB. Вы можете взаимодействовать с API Azure Cosmos DB для MongoDB с помощью любого из [драйверов клиента](https://docs.mongodb.org/ecosystem/drivers)с открытым кодом MongoDB. API Azure Cosmos DB для MongoDB позволяет использовать существующие клиентские драйверы, применяя [протокол MongoDB Wire](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol).

## <a name="mongodb-protocol-support"></a>Поддержка протокола MongoDB

API Azure Cosmos DB для MongoDB совместим с MongoDB Server версии 4,0, 3,6 и 3,2. Дополнительные сведения см. в статье поддерживаемые функции и синтаксис в статьях [4,0](mongodb-feature-support-40.md), [3,6](mongodb-feature-support-36.md)и [3,2](mongodb-feature-support.md) . 

Следующие команды расширения предоставляют возможность создания и изменения ресурсов, связанных с Azure Cosmos DB, с помощью запросов к базе данных.

* [Создание базы данных](#create-database)
* [Обновление базы данных](#update-database)
* [Получение базы данных](#get-database)
* [Создать коллекцию](#create-collection)
* [Обновить коллекцию](#update-collection)
* [Получить коллекцию](#get-collection)

## <a name="create-database"></a><a id="create-database"></a> Создание базы данных

Команда создания расширения базы данных создает новую базу данных MongoDB. Имя базы данных можно использовать из контекста базы данных, установленного `use database` командой. В следующей таблице описаны параметры в команде.

|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
| `customAction`   |  `string`  |   Имя пользовательской команды должно быть "CreateDatabase".      |
| `offerThroughput` | `int`  | Подготовленная пропускная способность, заданная для базы данных. Это необязательный параметр. |
| `autoScaleSettings` | `Object` | Требуется для [режима автомасштабирования](provision-throughput-autoscale.md). Этот объект содержит параметры, связанные с режимом автомасштабирования емкость. Можно настроить `maxThroughput` значение, которое описывает наибольшее количество единиц запросов, которое будет динамически увеличиваться в коллекции. |

### <a name="output"></a>Выходные данные

Если команда выполнена успешно, будет возвращен следующий ответ:

```javascript
{ "ok" : 1 }
```

Просмотрите [выходные данные пользовательской команды по умолчанию](#default-output) для параметров в выходных данных.

### <a name="examples"></a>Примеры

#### <a name="create-a-database"></a>Создание базы данных

Чтобы создать базу данных с именем `"test"` , использующую все значения по умолчанию, используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "CreateDatabase"});
```

Эта команда создаст базу данных без пропускной способности на уровне базы данных. Это означает, что коллекции в этой базе данных должны будут указывать требуемый объем пропускной способности.

#### <a name="create-a-database-with-throughput"></a>Создание базы данных с пропускной способностью

Чтобы создать базу данных с именем `"test"` и указать подготовленную пропускную способность [уровня базы данных](set-throughput.md#set-throughput-on-a-database) 1000 RUs, используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "CreateDatabase", offerThroughput: 1000 });
```

Это приведет к созданию базы данных и настройке пропускной способности для нее. Все коллекции в этой базе данных будут совместно использовать заданную пропускную способность, если только коллекции не будут созданы с [конкретным уровнем пропускной способности](set-throughput.md#set-throughput-on-a-database-and-a-container).

#### <a name="create-a-database-with-autoscale-throughput"></a>Создание базы данных с пропускной способностью автомасштабирования

Чтобы создать базу данных с именем `"test"` и указать максимальную пропускную способность автомасштабирования 20 000 единиц запросов в секунду на [уровне базы данных](set-throughput.md#set-throughput-on-a-database), используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "CreateDatabase", autoScaleSettings: { maxThroughput: 20000 } });
```

## <a name="update-database"></a><a id="update-database"></a> Обновить базу данных

Команда обновить расширение базы данных обновляет свойства, связанные с указанной базой данных. Изменение базы данных с подготовленной пропускной способности на Автомасштабирование и наоборот поддерживается только на портале Azure. В следующей таблице описаны параметры в команде.

|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
| `customAction`    |    `string`     |   Имя пользовательской команды. Должно быть "Упдатедатабасе".      |
|  `offerThroughput`   |  `int`       |     Новая подготовленная пропускная способность, которую необходимо задать для базы данных, если база данных использует [пропускную способность уровня базы](set-throughput.md#set-throughput-on-a-database) данных  |
| `autoScaleSettings` | `Object` | Требуется для [режима автомасштабирования](provision-throughput-autoscale.md). Этот объект содержит параметры, связанные с режимом автомасштабирования емкость. Можно настроить `maxThroughput` значение, которое описывает максимальный объем единиц запросов, которые будут динамически увеличиваться в базе данных. |

Эта команда использует базу данных, указанную в контексте сеанса. Это база данных, которая использовалась в `use <database>` команде. В настоящее время имя базы данных не может быть изменено с помощью этой команды.

### <a name="output"></a>Выходные данные

Если команда выполнена успешно, будет возвращен следующий ответ:

```javascript
{ "ok" : 1 }
```

Просмотрите [выходные данные пользовательской команды по умолчанию](#default-output) для параметров в выходных данных.

### <a name="examples"></a>Примеры

#### <a name="update-the-provisioned-throughput-associated-with-a-database"></a>Обновление подготовленной пропускной способности, связанной с базой данных

Чтобы обновить подготовленную пропускную способность базы данных с именем `"test"` в 1200 RUs, используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "UpdateDatabase", offerThroughput: 1200 });
```

#### <a name="update-the-autoscale-throughput-associated-with-a-database"></a>Обновление пропускной способности автомасштабирования, связанной с базой данных

Чтобы обновить подготовленную пропускную способность базы данных с именем `"test"` в 20 000 RUs или преобразовать ее в [уровень пропускной способности автомасштабирования](provision-throughput-autoscale.md), используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "UpdateDatabase", autoScaleSettings: { maxThroughput: 20000 } });
```


## <a name="get-database"></a><a id="get-database"></a> Получение базы данных

Команда получения расширения базы данных возвращает объект базы данных. Имя базы данных используется в контексте базы данных, для которой выполняется команда.

```javascript
{
  customAction: "GetDatabase"
}
```

В следующей таблице описаны параметры в команде.


|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
|  `customAction`   |   `string`      |   Имя пользовательской команды. Необходимо указать "".|
        
### <a name="output"></a>Выходные данные

Если команда выполнена, ответ содержит документ со следующими полями:

|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
|  `ok`   |   `int`     |   Состояние ответа. 1 = = успешное завершение. 0 = = сбой.      |
| `database`    |    `string`        |   Имя базы данных.      |
|   `provisionedThroughput`  |    `int`      |    Подготовленная пропускная способность, заданная для базы данных, если база данных использует  [пропускную способность уровня базы данных вручную](set-throughput.md#set-throughput-on-a-database)     |
| `autoScaleSettings` | `Object` | Этот объект содержит параметры емкости, связанные с базой данных, если она использует [режим автомасштабирования](provision-throughput-autoscale.md). `maxThroughput`Значение описывает наибольшее количество единиц запросов, которое будет динамически увеличиваться в базе данных. |

Если команда завершается ошибкой, возвращается пользовательский ответ команды по умолчанию. Просмотрите [выходные данные пользовательской команды по умолчанию](#default-output) для параметров в выходных данных.

### <a name="examples"></a>Примеры

#### <a name="get-the-database"></a>Получение базы данных

Чтобы получить объект базы данных с именем `"test"` , используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "GetDatabase"});
```

Если у базы данных нет связанной пропускной способности, выходные данные будут выглядеть следующим образом:

```javascript
{ "database" : "test", "ok" : 1 }
```

Если в базе данных имеется связанная с ней [пропускная способность вручную](set-throughput.md#set-throughput-on-a-database) , то выходные данные будут содержать `provisionedThroughput` следующие значения:

```javascript
{ "database" : "test", "provisionedThroughput" : 20000, "ok" : 1 }
```

Если с базой данных связана [пропускная способность автомасштабирования на уровне базы](provision-throughput-autoscale.md) данных, то выходные данные покажут `provisionedThroughput` , что описывает минимальный коэффициент единиц запросов в базе данных, и `autoScaleSettings` объект, включая `maxThroughput` , который описывает максимальный коэффициент единиц запросов в секунду для базы данных.

```javascript
{
        "database" : "test",
        "provisionedThroughput" : 2000,
        "autoScaleSettings" : {
                "maxThroughput" : 20000
        },
        "ok" : 1
}
```

## <a name="create-collection"></a><a id="create-collection"></a> Создать коллекцию

Команда Create Collection создает новую коллекцию MongoDB. Имя базы данных используется из контекста базы данных, установленного `use database` командой. Команда CreateCollection имеет следующий формат:

```javascript
{
  customAction: "CreateCollection",
  collection: "<Collection Name>",
  shardKey: "<Shard key path>",
  // Replace the line below with "autoScaleSettings: { maxThroughput: (int) }" to use Autoscale instead of Provisioned Throughput. Fill the required Autoscale max throughput setting.
  offerThroughput: (int) // Provisioned Throughput enabled with required throughput amount set
}
```

В следующей таблице описаны параметры в команде.

| **Поле** | **Тип** | **Обязательно** | **Описание** |
|---------|---------|---------|---------|
| `customAction` | `string` | Обязательно | Имя пользовательской команды. Должно быть "CreateCollection".|
| `collection` | `string` | Обязательно | Имя коллекции. Специальные символы и пробелы не допускаются.|
| `offerThroughput` | `int` | Необязательно | Подготовленная пропускная способность для установки в базе данных. Если этот параметр не указан, по умолчанию он будет иметь минимальный 400 единиц запросов в секунду. * Чтобы указать пропускную способность свыше 10 000 единиц запросов в секунду, `shardKey` параметр является обязательным.|
| `shardKey` | `string` | Требуется для коллекций с большой пропускной способностью | Путь к ключу сегмента для сегментированной коллекции. Этот параметр является обязательным, если задано более 10 000 единиц запросов в секунду `offerThroughput` .  Если он указан, для всех вставленных документов потребуется этот ключ и значение. |
| `autoScaleSettings` | `Object` | Требуется для [режима автомасштабирования](provision-throughput-autoscale.md) | Этот объект содержит параметры, связанные с режимом автомасштабирования емкость. Можно настроить `maxThroughput` значение, которое описывает наибольшее количество единиц запросов, которое будет динамически увеличиваться в коллекции. |

### <a name="output"></a>Выходные данные

Возвращает ответ пользовательской команды по умолчанию. Просмотрите [выходные данные пользовательской команды по умолчанию](#default-output) для параметров в выходных данных.

### <a name="examples"></a>Примеры

#### <a name="create-a-collection-with-the-minimum-configuration"></a>Создание коллекции с минимальной конфигурацией

Чтобы создать новую коллекцию с именем `"testCollection"` и значениями по умолчанию, используйте следующую команду: 

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection"});
```

Это приведет к созданию новой фиксированной, несегментированной коллекции с 400RU/s и `_id` автоматически созданного индекса в поле. Этот тип конфигурации также будет применяться при создании новых коллекций с помощью `insert()` функции. Пример: 

```javascript
use test
db.newCollection.insert({});
```

#### <a name="create-a-unsharded-collection"></a>Создание несегментированной коллекции

Чтобы создать несегментированную коллекцию с именем `"testCollection"` и подготовленной пропускной способностью 1000 RUs, используйте следующую команду: 

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", offerThroughput: 1000});
``` 

Можно создать коллекцию размером до 10 000 единиц запросов в секунду `offerThroughput` без необходимости указывать ключ сегмента. Для коллекций с большей пропускной способностью ознакомьтесь со следующим разделом.

#### <a name="create-a-sharded-collection"></a>Создание сегментированной коллекции

Чтобы создать сегментированную коллекцию с именем `"testCollection"` и подготовленной пропускной способностью 11 000 RUs, а также `shardkey` свойство "a. b", используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", offerThroughput: 11000, shardKey: "a.b" });
```

Теперь для этой команды требуется `shardKey` параметр, так как в задано более 10 000 единиц запросов/с `offerThroughput` .

#### <a name="create-an-unsharded-autoscale-collection"></a>Создание несегментированной коллекции автомасштабирования

Чтобы создать несегментированную коллекцию с именем `'testCollection'` , которая использует [пропускную способность автомасштабирования](provision-throughput-autoscale.md) , равную 4 000 единиц запросов в секунду, используйте следующую команду:

```javascript
use test
db.runCommand({ 
    customAction: "CreateCollection", collection: "testCollection", 
    autoScaleSettings:{
      maxThroughput: 4000
    } 
});
```

Для `autoScaleSettings.maxThroughput` значения можно указать диапазон от 4 000 единиц запросов в секунду до 10 000 единиц запросов в секунду без ключа сегмента. Для более высокой пропускной способности автомасштабирования необходимо указать `shardKey` параметр.

#### <a name="create-a-sharded-autoscale-collection"></a>Создание коллекции сегментированного автомасштабирования

Чтобы создать сегментированную коллекцию с именем `'testCollection'` и ключом сегмента с именем `'a.b'` , а также использовать [пропускную способность автомасштабирования](provision-throughput-autoscale.md) , равную 20 000 единиц запросов в секунду, используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", shardKey: "a.b", autoScaleSettings: { maxThroughput: 20000 }});
```

## <a name="update-collection"></a><a id="update-collection"></a> Обновить коллекцию

Команда Update Collection обновляет свойства, связанные с указанной коллекцией. Изменение коллекции с подготовленной пропускной способности на Автомасштабирование и наоборот поддерживается только на портале Azure.

```javascript
{
  customAction: "UpdateCollection",
  collection: "<Name of the collection that you want to update>",
  // Replace the line below with "autoScaleSettings: { maxThroughput: (int) }" if using Autoscale instead of Provisioned Throughput. Fill the required Autoscale max throughput setting. Changing between Autoscale and Provisioned throughput is only supported in the Azure Portal.
  offerThroughput: (int) // Provisioned Throughput enabled with required throughput amount set
}
```

В следующей таблице описаны параметры в команде.

|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
|  `customAction`   |   `string`      |   Имя пользовательской команды. Должно быть "Упдатеколлектион".      |
|  `collection`   |   `string`      |   Имя коллекции.       |
| `offerThroughput` | `int` |   Подготовленная пропускная способность для установки в коллекции.|
| `autoScaleSettings` | `Object` | Требуется для [режима автомасштабирования](provision-throughput-autoscale.md). Этот объект содержит параметры, связанные с режимом автомасштабирования емкость. `maxThroughput`Значение описывает наибольшее количество единиц запросов, которое будет динамически увеличиваться в коллекции. |

## <a name="output"></a>Выходные данные

Возвращает ответ пользовательской команды по умолчанию. Просмотрите [выходные данные пользовательской команды по умолчанию](#default-output) для параметров в выходных данных.

### <a name="examples"></a>Примеры

#### <a name="update-the-provisioned-throughput-associated-with-a-collection"></a>Обновление подготовленной пропускной способности, связанной с коллекцией

Чтобы обновить подготовленную пропускную способность коллекции с именем `"testCollection"` в 1200 RUs, используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "UpdateCollection", collection: "testCollection", offerThroughput: 1200 });
```

## <a name="get-collection"></a><a id="get-collection"></a> Получить коллекцию

Настраиваемая команда Get Collection возвращает объект коллекции.

```javascript
{
  customAction: "GetCollection",
  collection: "<Name of the collection>"
}
```

В следующей таблице описаны параметры в команде.


|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
| `customAction`    |   `string`      |   Имя пользовательской команды. Должен быть "IsCollection".      |
| `collection`    |    `string`     |    Имя коллекции.     |

### <a name="output"></a>Выходные данные

Если команда выполнена, ответ содержит документ со следующими полями


|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
|  `ok`   |    `int`     |   Состояние ответа. 1 = = успешное завершение. 0 = = сбой.      |
| `database`    |    `string`     |   Имя базы данных.      |
| `collection`    |    `string`     |    Имя коллекции.     |
|  `shardKeyDefinition`   |   `document`      |  Документ спецификации индекса, используемый в качестве ключа сегмента. Это необязательный параметр ответа.       |
|  `provisionedThroughput`   |   `int`      |    Подготовленная пропускная способность для установки в коллекции. Это необязательный параметр ответа.     |
| `autoScaleSettings` | `Object` | Этот объект содержит параметры емкости, связанные с базой данных, если она использует [режим автомасштабирования](provision-throughput-autoscale.md). `maxThroughput`Значение описывает наибольшее количество единиц запросов, которое будет динамически увеличиваться в коллекции. |

Если команда завершается ошибкой, возвращается пользовательский ответ команды по умолчанию. Просмотрите [выходные данные пользовательской команды по умолчанию](#default-output) для параметров в выходных данных.

### <a name="examples"></a>Примеры

#### <a name="get-the-collection"></a>Получение коллекции

Чтобы получить объект коллекции с именем `"testCollection"` , используйте следующую команду:

```javascript
use test
db.runCommand({customAction: "GetCollection", collection: "testCollection"});
```

Если с коллекцией связанная пропускная способность, она будет включать это `provisionedThroughput` значение, а выходные данные будут выглядеть следующим образом:

```javascript
{
        "database" : "test",
        "collection" : "testCollection",
        "provisionedThroughput" : 400,
        "ok" : 1
}
```

Если у коллекции есть связанная пропускная способность автомасштабирования, она будет включать `autoScaleSettings` объект с `maxThroughput` параметром, который определяет максимальную пропускную способность, которую коллекция будет увеличивать динамически. Кроме того, он также включает `provisionedThroughput` значение, определяющее минимальную пропускную способность, которую будет уменьшать эта коллекция при отсутствии запросов в коллекции. 

```javascript
{
        "database" : "test",
        "collection" : "testCollection",
        "provisionedThroughput" : 1000,
        "autoScaleSettings" : {
            "maxThroughput" : 10000
        },
        "ok" : 1
}
```

Если коллекция является общей [пропускной способностью на уровне базы данных](set-throughput.md#set-throughput-on-a-database)в режиме автомасштабирования или вручную, выходные данные будут выглядеть следующим образом:

```javascript
{ "database" : "test", "collection" : "testCollection", "ok" : 1 }
```

```javascript
{
        "database" : "test",
        "provisionedThroughput" : 2000,
        "autoScaleSettings" : {
            "maxThroughput" : 20000
        },
        "ok" : 1
}
```


## <a name="default-output-of-a-custom-command"></a><a id="default-output"></a> Выходные данные пользовательской команды по умолчанию

Если не указано, пользовательский ответ содержит документ со следующими полями:

|**Поле**|**Тип** |**Описание** |
|---------|---------|---------|
|  `ok`   |    `int`     |   Состояние ответа. 1 = = успешное завершение. 0 = = сбой.      |
| `code`    |   `int`      |   Возвращается только при сбое команды (т. е. ОК = = 0). Содержит код ошибки MongoDB. Это необязательный параметр ответа.      |
|  `errMsg`   |  `string`      |    Возвращается только при сбое команды (т. е. ОК = = 0). Содержит понятное сообщение об ошибке. Это необязательный параметр ответа.      |

Пример:

```javascript
{ "ok" : 1 }
```

## <a name="next-steps"></a>Дальнейшие действия

Далее можно перейти к рассмотрению следующих понятий Azure Cosmos DB: 

* [Индексирование в Azure Cosmos DB](../cosmos-db/index-policy.md)
* [Срок жизни для данных Azure Cosmos DB](../cosmos-db/time-to-live.md)
