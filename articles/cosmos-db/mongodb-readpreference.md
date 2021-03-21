---
title: Использование настройки чтения с помощью API Azure Cosmos DB для MongoDB
description: Сведения об использовании параметров чтения MongoDB в API Azure Cosmos DB для MongoDB
author: sivethe
ms.author: sivethe
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: how-to
ms.date: 02/26/2019
ms.custom: devx-track-js
ms.openlocfilehash: 3c78ad6605e927015d35df12cadf0347dd0337cf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96349050"
---
# <a name="how-to-globally-distribute-reads-using-azure-cosmos-dbs-api-for-mongodb"></a>Глобальное распространение операций чтения с помощью API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В этой статье описывается, как глобально распространить операции чтения с применением [параметров чтения MongoDB](https://docs.mongodb.com/manual/core/read-preference/) с помощью API Azure Cosmos DB для MongoDB.

## <a name="prerequisites"></a>Предварительные требования 
Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу. 
[!INCLUDE [cosmos-db-emulator-mongodb](../../includes/cosmos-db-emulator-mongodb.md)]

Перейдите к этому [краткому руководству](tutorial-global-distribution-mongodb.md), чтобы ознакомиться с инструкциями по настройке учетной записи Cosmos с глобальным распределением на портале Azure и последующему подключению.

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Откройте окно терминала Git, например Git Bash, и выполните команду `cd`, чтобы перейти в рабочий каталог.  

Затем выполните следующие команды, чтобы клонировать репозиторий с примером. В зависимости от платформы, которую вы хотите использовать, выберите один из следующих примеров репозиториев:

1. [Пример приложения .NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference)
2. [Пример приложения NodeJS]( https://github.com/Azure-Samples/azure-cosmos-db-mongodb-node-geo-readpreference)
3. [Пример приложения Mongoose](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-mongoose-geo-readpreference).
4. [Пример приложения Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference)
5. [Пример приложения SpringBoot](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-spring).


```bash
git clone <sample repo url>
```

## <a name="run-the-application"></a>Выполнение приложения

В зависимости от используемой платформы установите необходимые пакеты и запустите приложение. Чтобы установить зависимости, выполните инструкции в файле сведений, включенном в репозиторий с примерами приложений. Например, в примере приложения NodeJS выполните команды ниже, чтобы установить необходимые пакеты и запустить приложение.

```bash
cd mean
npm install
node index.js
```
Приложение попытается подключиться к источнику MongoDB, и это подключение завершится сбоем из-за недопустимой строки подключения. Выполните инструкции в файле сведений для обновления строки подключения `url`. Кроме того, укажите для параметра `readFromRegion` регион чтения в своей учетной записи Cosmos. Следующие инструкции взяты из примера приложения NodeJS:

```
* Next, substitute the `url`, `readFromRegion` in App.Config with your Cosmos account's values. 
```

После выполнения этих шагов пример приложения запустится, и мы получим следующие выходные данные:

```
connected!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromNearestfunc query completed!
readFromRegionfunc query completed!
readDefaultfunc query completed!
readFromSecondaryfunc query completed!
```

## <a name="read-using-read-preference-mode"></a>Чтение с использованием режима параметров чтения

Протокол MongoDB предоставляет следующие режимы параметров чтения для клиентов.

1. PRIMARY
2. PRIMARY_PREFERRED.
3. SECONDARY
4. SECONDARY_PREFERRED.
5. NEAREST.

Подробные сведения о поведении всех режимов параметров чтения MongoDB см. в [этой документации](https://docs.mongodb.com/manual/core/read-preference-mechanics/#replica-set-read-preference-behavior). В Cosmos DB основной регион соответствует региону ЗАПИСИ, а дополнительный — региону ЧТЕНИЯ.

На основе общих сценариев мы рекомендуем использовать следующие параметры:

1. Если требуются **операции чтения с низкой задержкой**, используйте режим параметров чтения **NEAREST**. Этот параметр направляет операции чтения в ближайший доступный регион. Обратите внимание, что если ближайшим регионом является регион записи, то эти операции будут направлены в этот регион.
2. Если требуется **Высокая доступность и географическое распределение операций чтения** (задержка не является ограничением), используйте **основной предпочтительный** или **вторичный** режим настройки чтения. Этот параметр направляет операции чтения в доступный регион записи или чтения соответственно. Если регион недоступен, запросы направляются в следующий доступный регион в соответствии с режимом настройки чтения.

В следующем фрагменте кода, взятого из примера приложения, показано, как настроить режим параметров чтения "NEAREST" в NodeJS:

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.NEAREST});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromNearestfunc query completed!");
  });
```

Аналогичным образом в приведенном ниже фрагменте кода показано, как настроить режим параметров чтения "SECONDARY_PREFERRED" в NodeJS:

```javascript
  var query = {};
  var readcoll = client.db('regionDB').collection('regionTest', {readPreference: ReadPreference.SECONDARY_PREFERRED});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromSecondaryPreferredfunc query completed!");
  });
```

Кроме того, этот режим можно задать, передав `readPreference` как параметр в универсальном коде ресурса (URI) строки подключения.

```javascript
const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');

// Connection URL
const url = 'mongodb://localhost:27017?ssl=true&replicaSet=globaldb&readPreference=nearest';

// Database Name
const dbName = 'myproject';

// Use connect method to connect to the Server
MongoClient.connect(url, function(err, client) {
  console.log("Connected correctly to server");

  const db = client.db(dbName);

  client.close();
});
```

Используйте соответствующие репозитории с примерами приложений для других платформ, например [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) и [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference).

## <a name="read-using-tags"></a>Чтение с использованием тегов

Кроме режима параметров чтения, в протоколе MongoDB можно использовать теги для направления операций чтения. В API Cosmos DB для MongoDB тег `region` включен по умолчанию как часть ответа `isMaster`.

```json
"tags": {
         "region": "West US"
      }
```

Таким образом, MongoClient может использовать тег `region` с именем региона, чтобы направлять операции чтения в определенные регионы. Для учетных записей Cosmos названия регионов можно найти на портале Azure в левой области в разделе **Параметры -> Глобальная репликация данных**. Этот параметр позволяет достичь **изоляции операций чтения** — в случаях, когда клиентскому приложению нужно направлять операции чтения только в определенный регион. Этот параметр идеально подходит для сценариев нерабочего или аналитического типа, которые работают в фоновом режиме и не задействуют критически важные рабочие службы.

В следующем фрагменте кода, взятого из примера приложения, показано, как настроить параметры чтения с тегами в NodeJS:

```javascript
 var query = {};
  var readcoll = client.db('regionDB').collection('regionTest',{readPreference: new ReadPreference(ReadPreference.SECONDARY_PREFERRED, {"region": "West US"})});
  readcoll.find(query).toArray(function(err, data) {
    assert.equal(null, err);
    console.log("readFromRegionfunc query completed!");
  });
```

Используйте соответствующие репозитории с примерами приложений для других платформ, например [.NET](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-geo-readpreference) и [Java](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-java-geo-readpreference).

В этой статье вы узнали, как глобально распространить операции чтения с применением параметров чтения с помощью API Azure Cosmos DB для MongoDB.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение дальше, удалите все ресурсы, созданные в ходе работы с этой статьей, на портале Azure, выполнив следующие действия:

1. В меню слева на портале Azure щелкните **Группы ресурсов**, а затем выберите имя созданного ресурса. 
2. На странице группы ресурсов щелкните **Удалить**, в текстовом поле введите имя ресурса для удаления и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

* [Перенос данных MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)
* [Настройка глобальной распределенной базы данных с помощью API Azure Cosmos DB для MongoDB](tutorial-global-distribution-mongodb.md)
* [Разработка с помощью эмулятора Azure Cosmos DB в локальной среде](local-emulator.md)