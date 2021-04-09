---
title: Создание графовой базы данных в Azure Cosmos DB с помощью Java
description: В этой статье представлен пример кода Java, который можно использовать для подключения и выполнения запросов к данным графа в Azure Cosmos DB с помощью Gremlin.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.devlang: java
ms.topic: quickstart
ms.date: 03/26/2019
ms.author: chrande
ms.custom: seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 5a793eb69b344e361679759e46db47c78885831e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93360962"
---
# <a name="quickstart-build-a-graph-database-with-the-java-sdk-and-the-azure-cosmos-db-gremlin-api"></a>Краткое руководство. Создание графовой базы данных с помощью пакета SDK для Java и API Gremlin для Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

> [!div class="op_single_selector"]
> * [Консоль Gremlin](create-graph-gremlin-console.md)
> * [.NET](create-graph-dotnet.md)
> * [Java](create-graph-java.md)
> * [Node.js](create-graph-nodejs.md)
> * [Python](create-graph-python.md)
> * [PHP](create-graph-php.md)
>  

В этом кратком руководстве вы создадите учетную запись API Gremlin (граф) для Azure Cosmos DB и будете управлять ею из портала Azure, после чего добавите данные с помощью приложения Java, клонированного из GitHub. Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования.

## <a name="prerequisites"></a>Предварительные требования
- Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). 
- [Комплект SDK для Java (JDK) версии 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk). Укажите для переменной среды `JAVA_HOME` папку, в которой установлен комплект JDK.
- [Двоичный архив Maven](https://maven.apache.org/download.cgi). 
- [Git](https://www.git-scm.com/downloads). 

## <a name="create-a-database-account"></a>Создание учетной записи базы данных

Перед созданием графовой базы данных необходимо создать учетную запись графовой базы данных Gremlin с Azure Cosmos DB.

[!INCLUDE [cosmos-db-create-dbaccount-graph](../../includes/cosmos-db-create-dbaccount-graph.md)]

## <a name="add-a-graph"></a>Добавление графа

[!INCLUDE [cosmos-db-create-graph](../../includes/cosmos-db-create-graph.md)]

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Теперь перейдем к работе с кодом. Мы клонируем приложение API Gremlin из GitHub, зададим строку подключения и запустим приложение. Вы узнаете, как можно упростить работу с данными программным способом.  

1. Откройте командную строку, создайте папку git-samples, а затем закройте окно командной строки.

    ```bash
    md "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в папку для установки примера приложения.  

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере. 

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-graph-java-getting-started.git
    ```

## <a name="review-the-code"></a>Просмотр кода

Это необязательный шаг. Если вы хотите узнать, как создать в коде ресурсы базы данных, изучите приведенные ниже фрагменты кода. Если вас это не интересует, можете сразу переходить к разделу [Обновление строки подключения](#update-your-connection-information).

Следующие фрагменты кода взяты из файла *C:\git-samples\azure-cosmos-db-graph-java-getting-started\src\GetStarted\Program.java*.

Это консольное приложение Java использует базу данных [API Gremlin](graph-introduction.md) с драйвером OSS [Apache TinkerPop](https://tinkerpop.apache.org/). 

- `Client` Gremlin инициализируется на основе конфигурации в файле *C:\git-samples\azure-cosmos-db-graph-java-getting-started\src\remote.yaml*.

    ```java
    cluster = Cluster.build(new File("src/remote.yaml")).create();
    ...
    client = cluster.connect();
    ```

- Ряд шагов Gremlin выполняется с использованием метода `client.submit`.

    ```java
    ResultSet results = client.submit(gremlin);

    CompletableFuture<List<Result>> completableFutureResults = results.all();
    List<Result> resultList = completableFutureResults.get();

    for (Result result : resultList) {
        System.out.println(result.toString());
    }
    ```

## <a name="update-your-connection-information"></a>Обновление информации о подключении

Теперь вернитесь на портал Azure, чтобы получить данные для подключения. Скопируйте эти данные в приложение. Эти настройки обеспечат обмен данными между вашим приложением и размещенной базой данных.

1. В учетной записи Azure Cosmos DB на [портале Azure](https://portal.azure.com/) выберите элемент **Ключи**. 

    Скопируйте первую часть значения URI.

    :::image type="content" source="./media/create-graph-java/copy-access-key-azure-portal.png" alt-text="Просмотр и копирование ключа доступа на портале Azure, страница Ключи":::

2. Откройте файл *src/remote.yaml* и вставьте значение уникального идентификатора вместо `$name$` в `hosts: [$name$.graphs.azure.com]`.

    Теперь строка 1 в файле *remote.yaml* будет выглядеть примерно так: 

    `hosts: [test-graph.graphs.azure.com]`

3. В значении `endpoint` измените `graphs` на `gremlin.cosmosdb`. (Если учетная запись базы данных графа создана до 20 декабря 2017 г., оставьте значение конечной точки без изменений и перейдите к следующему шагу.)

    Значение конечной точки должно выглядеть так:

    `"endpoint": "https://testgraphacct.gremlin.cosmosdb.azure.com:443/"`

4. На портале Azure используйте кнопку "Копировать", чтобы скопировать первичный ключ и вставить его вместо `$masterKey$` в `password: $masterKey$`.

    Теперь строка 4 в файле *remote.yaml* будет выглядеть примерно так: 

    `password: 2Ggkr662ifxz2Mg==`

5. В файле *remote.yaml* измените строку 3. Укажите вместо

    `username: /dbs/$database$/colls/$collection$`

    значение 

    `username: /dbs/sample-database/colls/sample-graph`

    Если вы использовали уникальное имя для примера базы данных или графа, обновите соответствующие значения.

6. Сохраните файл *remote.yaml*.

## <a name="run-the-console-app"></a>Запуск консольного приложения

1. В окне терминала Git перейдите в папку azure-cosmos-db-graph-java-getting-started с помощью команды `cd`.

    ```git
    cd "C:\git-samples\azure-cosmos-db-graph-java-getting-started"
    ```

2. В окне терминала Git введите приведенную ниже команду, чтобы установить необходимые пакеты Java.

   ```git
   mvn package
   ```

3. В окне терминала Git выполните следующую команду, чтобы запустить приложение Java.
    
    ```git
    mvn exec:java -D exec.mainClass=GetStarted.Program
    ```

    В окне терминала появятся вершины, добавляемые в граф. 
    
    Если возникают ошибки времени ожидания, проверьте, правильно ли вы [указали сведения о подключении](#update-your-connection-information), и попробуйте еще раз выполнить последнюю команду. 
    
    Когда программа завершит работу, нажмите клавишу ВВОД и откройте окно веб-браузера с порталом Azure. 

<a id="add-sample-data"></a>
## <a name="review-and-add-sample-data"></a>Просмотр и добавление демонстрационных данных

Теперь можно вернуться в обозреватель данных, чтобы просмотреть вершины, добавленные в граф, и добавить дополнительные точки данных.

1. В учетной записи Azure Cosmos DB на портале Azure выберите **Обозреватель данных**, разверните **sample-graph**, а затем выберите **Граф** и нажмите кнопку **Применить фильтр**. 

   :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-expanded.png" alt-text="Снимок экрана: выбран пункт &quot;Граф&quot; в разделе API и выделена кнопка &quot;Применить фильтр&quot;.":::

2. В списке **результатов** обратите внимание на новых пользователей, добавленных в граф. Выберите пользователя **ben**. Вы увидите, что он связан с пользователем robin. Здесь можно перетаскивать вершины мышью, увеличивать или уменьшать масштаб колесиком мыши, а также увеличивать размер графа с помощью двойной стрелки. 

   :::image type="content" source="./media/create-graph-java/azure-cosmosdb-graph-explorer-new.png" alt-text="Новые вершины в графе в обозревателе данных на портале Azure":::

3. Давайте добавим несколько новых пользователей. Щелкните **Создать вершину**, чтобы добавить данные в граф.

   :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-new-vertex.png" alt-text="Снимок экрана: панель &quot;Новая вершина&quot; для ввода значений.":::

4. В поле метки укажите *пользователя*.

5. Щелкните **Добавить свойство**, чтобы добавить каждое из указанных ниже свойств. Обратите внимание, что вы можете создать уникальные свойства для каждого пользователя в графе. Требуется только ключ идентификатора.

    ключ|value|Примечания
    ----|----|----
    идентификатор|ashley|Уникальный идентификатор вершины. Если не указать идентификатор, он создастся автоматически.
    gender|Женский| 
    Технология | java | 

    > [!NOTE]
    > С помощью этого краткого руководства вы создадите несекционированную коллекцию. Но если вы создаете секционированную коллекцию, указав ключ раздела во время создания коллекции, включите ключ раздела в качестве ключа каждой новой вершины. 

6. Щелкните **ОК**. Чтобы увидеть кнопку **ОК** в нижней части экрана, может потребоваться развернуть экран.

7. Еще раз щелкните **Создать вершину**, чтобы добавить нового пользователя. 

8. Введите метку *person*.

9. Щелкните **Добавить свойство**, чтобы добавить каждое из указанных ниже свойств:

    ключ|value|Примечания
    ----|----|----
    идентификатор|rakesh|Уникальный идентификатор вершины. Если не указать идентификатор, он создастся автоматически.
    gender|Мужской| 
    Учебное заведение|MIT| 

10. Щелкните **ОК**. 

11. Нажмите кнопку **Применить фильтр** со стандартным значением фильтра `g.V()`, чтобы отобразить все значения графа. Все пользователи теперь отображаются в списке **Результаты**. 

    По мере добавления новых данных используйте фильтры для ограничения результатов. По умолчанию обозреватель данных использует `g.V()` для получения всех вершин в графе. Вы можете задать для него другой [запрос графа](tutorial-query-graph.md), например `g.V().count()`, который возвращает число всех вершин графа в формате JSON. Если вы изменили фильтр, снова установите фильтр `g.V()` и щелкните **Применить фильтр**, чтобы снова отобразить полный список результатов.

12. Теперь вы можете соединить пользователей rakesh и ashley. Выберите в списке **Результаты** пользователя **ashley** и щелкните :::image type="content" source="./media/create-graph-java/edit-pencil-button.png" alt-text="Изменение целевого объекта вершины в графе"::: рядом с пунктом **Целевые объекты** внизу справа. Чтобы отобразить кнопку, может потребоваться развернуть окно.

    :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-edit-target.png" alt-text="Изменение целевого объекта вершины в графе в Azure Cosmos DB":::

13. В поле **Целевой объект** введите *rakesh*, затем в поле **Граничная метка** введите слово *знает* и установите флажок.

    :::image type="content" source="./media/create-graph-java/azure-cosmosdb-data-explorer-set-target.png" alt-text="Добавление подключения в обозревателе данных в Azure Cosmos DB":::

14. Теперь выберите пользователя **rakesh** в списке результатов. Вы увидите, что пользователи ashley и rakesh связаны. 

    :::image type="content" source="./media/create-graph-java/azure-cosmosdb-graph-explorer.png" alt-text="Две соединенных вершины в обозревателе данных в Azure Cosmos DB":::

На этом часть руководства, посвященная созданию ресурсов, завершена. Вы можете дополнить граф новыми вершинами, а также изменить существующие вершины или запросы. Теперь давайте изучим метрики, которые предоставляет Azure Cosmos DB, а затем очистим все ресурсы. 

## <a name="review-slas-in-the-azure-portal"></a>Просмотр соглашений об уровне обслуживания на портале Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать учетную запись Azure Cosmos DB, построить граф с помощью обозревателя данных, а также запустить приложение Java, которое добавляет данные в граф. Теперь вы можете создавать более сложные запросы и внедрять эффективную логику обхода графа с помощью Gremlin. 

> [!div class="nextstepaction"]
> [Как выполнять запросы к данным в базе данных Azure Cosmos DB с помощью API Graph (предварительная версия)](tutorial-query-graph.md)

