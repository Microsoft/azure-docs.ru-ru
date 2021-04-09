---
title: Перенос MongoDB в API Azure Cosmos DB для MongoDB в автономном режиме с использованием нативных средств MongoDB
description: Сведения о том, как с помощью нативных средств MongoDB можно перенести небольшие наборы данных из экземпляров MongoDB в Azure Cosmos DB.
author: anfeldma-ms
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 02/10/2021
ms.reviewer: sngun
ms.openlocfilehash: 2e9f3c877a5c4650d3e31fa414cac76837f4c9e8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101655757"
---
# <a name="tutorial-migrate-mongodb-to-azure-cosmos-dbs-api-for-mongodb-offline-using-mongodb-native-tools"></a>Руководство по переносу MongoDB в API Azure Cosmos DB для MongoDB в автономном режиме с использованием нативных средств MongoDB

С помощью нативных средств MongoDB можно выполнить автономный (однократный) перенос баз данных из локального или облачного экземпляра MongoDB в API Azure Cosmos DB для MongoDB.

В этом руководстве описано следующее:
> [!div class="checklist"]
>
> * выбор подходящего нативного средства MongoDB для вашего варианта использования;
> * выполнение миграции.
> * Мониторинг миграции.
> * проверка успешного выполнения миграции.

В этом руководстве описано, как перенести набор данных из базы данных MongoDB, размещенной на Виртуальной машине Azure, в API Azure Cosmos DB для MongoDB с помощью нативных средств MongoDB. Нативные средства MongoDB — это набор двоичных файлов, упрощающих обработку данных в существующем экземпляре MongoDB. Так как Azure Cosmos DB предоставляет API Mongo, нативные средства MongoDB могут вставлять данные в Azure Cosmos DB. В этом документе основное внимание уделяется переносу данных из экземпляра MongoDB с помощью *mongoexport/mongoimport* или *mongodump/mongorestore*. Так как нативные средства подключаются к MongoDB с помощью строк подключения, вы можете запускать их в любом месте, но мы рекомендуем делать это в той же сети, в которой размещен экземпляр MongoDB, чтобы избежать проблем с брандмауэром. 

Нативные средства MongoDB могут перемещать данные не быстрее, чем это позволяет оборудование. Они могут быть самым простым решением для небольших наборов данных, для которых общее время миграции не имеет значения. [Соединитель Spark для MongoDB](https://docs.mongodb.com/spark-connector/current/), [Azure Data Migration Service (DMS)](../dms/tutorial-mongodb-cosmos-db.md) или [Фабрика данных Azure (ADF)](../data-factory/connector-azure-cosmos-db-mongodb-api.md) могут быть более полезны, если вам требуется масштабируемый конвейер миграции.

Если у вас еще не настроен источник MongoDB, обратитесь к статье [Установка и настройка базы данных MongoDB на виртуальной машине Windows в Azure](/previous-versions/azure/virtual-machines/windows/install-mongodb).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

* [Завершите шаги, которые необходимо выполнить перед миграцией](../cosmos-db/mongodb-pre-migration.md), включая оценку пропускной способности, а также выбор ключа секции и политики индексирования.
* [Создать API Azure Cosmos DB для учетной записи MongoDB.](https://ms.portal.azure.com/#create/Microsoft.DocumentDB)
* Войти в экземпляр MongoDB.
    * [Скачать нативные средства MongoDB по этой ссылке и установить их.](https://www.mongodb.com/try/download/database-tools)
        * **Убедиться, что версия нативных средств MongoDB соответствует версии имеющегося экземпляра MongoDB.**
        * Если версия экземпляра MongoDB отличается от версии API Azure Cosmos DB для MongoDB, **установите обе версии нативных средств MongoDB и используйте соответствующую версию средства MongoDB и API Azure Cosmos DB для MongoDB соответственно.**
    * Добавить пользователя с разрешениями `readWrite` (при его отсутствии). Далее при работе с этим руководством вам нужно будет указать имя пользователя и пароль в средствах *mongoexport* и *mongodump*.

## <a name="configure-azure-cosmos-db-server-side-retries"></a>Настройка повторных попыток в Azure Cosmos DB на стороне сервера

Клиенты, которые переходят с MongoDB на Azure Cosmos DB получают преимущества возможностей управления ресурсами, позволяющие полностью использовать подготовленные ЕЗ/с пропускной способности. Azure Cosmos DB может регулировать определенный запрос в процессе миграции, если размер этого запроса превышает число подготовленных ЕЗ/с контейнера. После этого такой запрос нужно повторить. Время кругового пути между средством миграции и Azure Cosmos DB влияет на общее время отклика этого запроса. Более того, нативные средства MongoDB могут не справиться с повторными попытками. Функция *повторных попыток на стороне сервера* Azure Cosmos DB позволяет службе перехватывать коды ошибок регулирования и повторять попытки с уменьшенным временем кругового пути, что значительно сокращает время отклика для запроса. С точки зрения нативных средств MongoDB необходимость в обработке повторных попыток сводиться к минимуму, что положительно сказывается на процессе миграции.

Функцию повторных попыток на стороне сервера можно найти в колонке *Функции* на портале Azure Cosmos DB.

![Снимок экрана: функция SSR в MongoDB.](../dms/media/tutorial-mongodb-to-cosmosdb/mongo-server-side-retry-feature.png)

Если функция *отключена*, рекомендуется включить ее, как показано ниже.

![Снимок экрана: функция SSR в MongoDB включена.](../dms/media/tutorial-mongodb-to-cosmosdb/mongo-server-side-retry-enable.png)

## <a name="choose-the-proper-mongodb-native-tool"></a>Выбор надлежащего нативного средства MongoDB

![Схема выбора оптимального нативного средства MongoDB.](media/tutorial-mongotools-cosmos-db/mongodb-native-tool-selection-table.png)

* *mongoexport/mongoimport* — это лучшая пара средств миграции для переноса подмножества вашей базы данных MongoDB.
    * *mongoexport* экспортирует существующие данные в JSON- или CSV-файл в понятном для человека формате. *mongoexport* принимает аргумент, указывающий подмножество существующих данных для экспорта. 
    * *mongoimport* открывает JSON- или CSV-файл и вставляет содержимое в целевой экземпляр базы данных (в нашем случае — Azure Cosmos DB). 
    * Обратите внимание, что JSON и CSV — это не компактные форматы. Если *mongoimport* отправляет данные в Azure Cosmos DB, может взиматься дополнительная плата за использование сети.
* *mongodump/mongorestore* — это лучшая пара средств миграции для переноса всей базы данных MongoDB. Компактный формат BSON позволяет более эффективно использовать ресурсы сети при вставке данных в Azure Cosmos DB.
    * *mongodump* экспортирует существующие данные в виде BSON-файла.
    * *mongorestore* импортирует дамп BSON-файла в Azure Cosmos DB.
* Небольшое замечание: если вы хотите просто импортировать в API Azure Cosmos DB для Mongo небольшой JSON-файл, для быстрого приема данных можно использовать средство *mongoimport*.

## <a name="collect-the-azure-cosmos-db-mongo-api-credentials"></a>Сбор учетных данных API Azure Cosmos DB для Mongo

API Azure Cosmos DB для Mongo предоставляет совместимые учетные данные доступа, которые могут использоваться нативными средствами MongoDB. Для переноса данных в API Azure Cosmos DB для Mongo вам потребуются такие учетные данные доступа. Чтобы найти их, сделайте следующее:

1. Откройте портал Azure.
1. Перейдите к своей учетной записи API Azure Cosmos DB для Mongo.
1. На панели навигации слева выберите колонку *Строка подключения*, и вы увидите экран, похожий на приведенный ниже:

    ![Снимок экрана: учетные данные Azure Cosmos DB.](media/tutorial-mongotools-cosmos-db/cosmos-mongo-credentials.png)

    * *HOST* (Узел) — конечная точка Azure Cosmos DB, которая выступает в роли имени узла MongoDB.
    * *PORT* (Порт) — при подключении нативных средств MongoDB к Azure Cosmos DB вам необходимо явно указать этот порт.
    * *USERNAME* (Имя пользователя) — префикс доменного имени конечной точки Azure Cosmos DB, который выступает в роли имени пользователя MongoDB.
    * *PASSWORD* (Пароль) — главный ключ Azure Cosmos DB, который выступает в роли пароля MongoDB.
    * Кроме того, обратите внимание на поле *SSL* (со значением `true`), так как нативное средство MongoDB **должно** включить SSL при записи данных в Azure Cosmos DB.

## <a name="perform-the-migration"></a>Выполнение миграции

1. Выберите базы данных и коллекции для переноса. В этом примере выполняется миграция коллекции *query* в базе данных *edx* из MongoDB в Azure Cosmos DB.

В остальной части этого раздела показано, как использовать пару средств, которую вы выбрали при изучении предыдущего раздела.

### <a name="mongoexportmongoimport"></a>*mongoexport/mongoimport*

1. Чтобы экспортировать данные из исходного экземпляра MongoDB, откройте терминал на компьютере экземпляра MongoDB. Если это компьютер Linux, введите следующее:

    `mongoexport --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection query --out edx.json`

    В Windows исполняемый файл будет иметь имя `mongoexport.exe`. Значения в полях *HOST* (Узел), *PORT* (Порт), *USERNAME* (Имя пользователя) и *PASSWORD* (Пароль) необходимо указать с учетом свойств существующего экземпляра базы данных MongoDB. 
    
    Вы также можете экспортировать подмножество набора данных MongoDB, а не всю базу данных. Один из способов сделать это — добавить аргумент фильтра:
    
    `mongoexport --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection query --out edx.json --query '{"field1":"value1"}'`

    При этом будут экспортированы только те документы, которые соответствуют фильтру `{"field1":"value1"}`.

    После выполнения вызова вы должны увидеть, что создается файл `edx.json`:

    ![Снимок экрана: вызов mongoexport.](media/tutorial-mongotools-cosmos-db/mongo-export-output.png)
1. Вы можете использовать этот же терминал для импорта `edx.json` в Azure Cosmos DB. Если вы используете `mongoimport` на компьютере Linux, введите следующее:

    `mongoimport --host HOST:PORT -u USERNAME -p PASSWORD --db edx --collection importedQuery --ssl --type json --writeConcern="{w:0}" --file edx.json`

    В Windows исполняемый файл будет иметь имя `mongoimport.exe`. Значения в полях *HOST* (Узел), *PORT* (Порт), *USERNAME* (Имя пользователя) и *PASSWORD* (Пароль) нужно указать с учетом собранных раннее учетных данных Azure Cosmos DB. 
1. **Просматривайте** выходные данные терминала от *mongoimport*. Вы увидите, что средство выводит строки текста в терминал с данными о состоянии миграции:

    ![Снимок экрана: вызов mongoimport.](media/tutorial-mongotools-cosmos-db/mongo-import-output.png)    

1. Наконец, **проверьте** успешность миграции в Azure Cosmos DB. Откройте портал Azure Cosmos DB и перейдите к обозревателю данных. Вы должны увидеть следующее: 1) была создана база данных *edx* с коллекцией *importedQuery*; 2) если вы экспортировали только подмножество данных, *importedQuery* должен содержать *только* документы, соответствующие нужному подмножеству данных. В примере ниже только один документ соответствует фильтру `{"field1":"value1"}`:

    ![Снимок экрана: проверка данных Cosmos DB.](media/tutorial-mongotools-cosmos-db/mongo-review-cosmos.png)    

### <a name="mongodumpmongorestore"></a>*mongodump/mongorestore*

1. Чтобы создать дамп данных BSON для экземпляра MongoDB, откройте терминал на компьютере с экземпляром MongoDB. Если это компьютер Linux, введите следующее:

    `mongodump --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection query --out edx-dump`

    Значения в полях *HOST* (Узел), *PORT* (Порт), *USERNAME* (Имя пользователя) и *PASSWORD* (Пароль) необходимо указать с учетом свойств существующего экземпляра базы данных MongoDB. Вы должны увидеть, что создается каталог `edx-dump` и что структура `edx-dump` воспроизводит иерархию ресурсов (структуру базы данных и коллекции) в исходном экземпляре MongoDB. Каждая коллекция представлена отдельным BSON-файлом:

    ![Снимок экрана: вызов mongodump.](media/tutorial-mongotools-cosmos-db/mongo-dump-output.png)
1. Вы можете использовать этот же терминал для восстановления содержимого `edx-dump` в Azure Cosmos DB. Если вы используете `mongorestore` на компьютере Linux, введите следующее:

    `mongorestore --host HOST:PORT --authenticationDatabase admin -u USERNAME -p PASSWORD --db edx --collection importedQuery --ssl edx-dump/edx/query.bson`

    В Windows исполняемый файл будет иметь имя `mongorestore.exe`. Значения в полях *HOST* (Узел), *PORT* (Порт), *USERNAME* (Имя пользователя) и *PASSWORD* (Пароль) нужно указать с учетом собранных раннее учетных данных Azure Cosmos DB. 
1. **Просматривайте** выходные данные терминала от *mongorestore*. Вы увидите, что средство выводит строки текста в терминал с данными о состоянии миграции:

    ![Снимок экрана: вызов mongorestore.](media/tutorial-mongotools-cosmos-db/mongo-restore-output.png)    

1. Наконец, **проверьте** успешность миграции в Azure Cosmos DB. Откройте портал Azure Cosmos DB и перейдите к обозревателю данных. Вы должны увидеть следующее: 1) что создана база данных *edx* с коллекцией *importedQuery*; 2) *importedQuery* должен содержать *весь* набор данных из исходной коллекции:

    ![Снимок экрана: проверка данных в Cosmos DB с помощью mongorestore.](media/tutorial-mongotools-cosmos-db/mongo-review-cosmos-restore.png)    

## <a name="post-migration-optimization"></a>Оптимизация после переноса

Для управления данными, перенесенными из базы данных MongoDB в API Azure Cosmos DB для MongoDB, можно подключиться к Azure Cosmos DB. После переноса можно также выполнить другие действия, включая оптимизацию политики индексирования, обновление уровня согласованности по умолчанию и настройку глобального распределения для своей учетной записи Azure Cosmos DB. См. подробнее об [оптимизации после переноса](../cosmos-db/mongodb-post-migration.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Сведения о службе Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)
* [Документация по средствам для работы с базой данных MongoDB](https://docs.mongodb.com/database-tools/)

## <a name="next-steps"></a>Дальнейшие действия

* Просмотрите другие сценарии в [руководстве по миграции базы данных Майкрософт](https://datamigration.microsoft.com/).