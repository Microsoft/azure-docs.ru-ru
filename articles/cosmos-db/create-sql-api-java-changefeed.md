---
title: Создание примера полного приложения пакета SDK Java для Azure Cosmos DB версии 4 с использованием канала изменений
description: В этом руководстве описано простое приложение API SQL на языке Java, которое вставляет документы в контейнер Azure Cosmos DB, сохраняя материализованное представление контейнера с помощью канала изменений.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 06/11/2020
ms.author: anfeldma
ms.custom: devx-track-java
ms.openlocfilehash: 765fd3afc7fe688d3e6b0e3394e7dc8c39af69b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93096858"
---
# <a name="how-to-create-a-java-application-that-uses-azure-cosmos-db-sql-api-and-change-feed-processor"></a>Создание приложения Java, которое использует API SQL Azure Cosmos DB и обработчик канала изменений
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этом практическом руководстве описано простое приложение на языке Java, которое применяет API SQL Azure Cosmos DB для передачи документов в контейнер Azure Cosmos DB и поддерживает материализованное представление этого контейнера с помощью канала изменений и обработчика канала изменений. Это приложение Java взаимодействует с API SQL Azure Cosmos DB через пакет SDK Java для Azure Cosmos DB версии 4.

> [!IMPORTANT]  
> Это руководство применимо только к пакету SDK Java для Azure Cosmos DB версии 4. Дополнительные сведения см. в [заметках о выпуске](sql-api-sdk-java-v4.md) для пакета средств разработки Java для Azure Cosmos DB версии 4, [репозитории Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), [рекомендациях по повышению производительности](performance-tips-java-sdk-v4-sql.md) и [руководстве по устранению неполадок](troubleshoot-java-sdk-v4-sql.md) для пакета средств разработки Java для Azure Cosmos DB версии 4. Если сейчас вы используете более раннюю версию, чем версия 4, руководство [Перевод приложения на использование пакета средств разработки Java для Azure Cosmos DB версии 4](migrate-java-v4-sdk.md) поможет вам обновить его до версии 4.
>

## <a name="prerequisites"></a>Предварительные требования

* Универсальный код ресурса (URI) и ключ для учетной записи Azure Cosmos DB.

* Maven

* Java 8

## <a name="background"></a>Историческая справка

Канал изменений Azure Cosmos DB предоставляет основанный на событиях интерфейс для активации действий при вставке документа. Этот канал изменений можно использовать множеством различных способов. Например, в приложениях, которые доступны как для чтения, так и для записи, главное применение канала изменений — создание **материализованных представлений** контейнера по мере приема документов. Контейнер в материализованном представлении будет содержать те же данные, но секционированные для эффективного чтения. Благодаря этому приложение будет эффективно выполнять операции чтения и записи.

Работа по управлению событиями канала изменений выполняется в основном встроенной в пакет SDK библиотекой обработчика канала изменений. Эта библиотека даже умеет распределять события канала изменений между несколькими рабочими ролями, если это потребуется. Вам достаточно лишь предоставить ей обратный вызов.

В этом простом примере показана библиотека обработчика канала изменений с одной рабочей ролью, которая позволяет создавать документы и удалять их из материализованного представления.

## <a name="setup"></a>Настройка

Если вы еще не сделали этого, клонируйте репозиторий примера приложения:

```bash
git clone https://github.com/Azure-Samples/azure-cosmos-java-sql-app-example.git
```

Откройте терминал в каталоге репозитория. Выполните сборку приложения с помощью следующей команды:

```bash
mvn clean package
```

## <a name="walkthrough"></a>Пошаговое руководство

1. В первую очередь проверьте наличие учетной записи Azure Cosmos DB. Откройте **портал Azure** в браузере, перейдите к учетной записи Azure Cosmos DB и в области слева перейдите к **обозревателю данных**.

   :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_account_empty.JPG" alt-text="Учетная запись Azure Cosmos DB":::

1. Запустите приложение в окне терминала с помощью следующей команды:

    ```bash
    mvn exec:java -Dexec.mainClass="com.azure.cosmos.workedappexample.SampleGroceryStore" -DACCOUNT_HOST="your-account-uri" -DACCOUNT_KEY="your-account-key" -Dexec.cleanupDaemonThreads=false
    ```

1. Нажмите клавишу ВВОД, когда на экране появится следующее сообщение.

    ```bash
    Press enter to create the grocery store inventory system...
    ```

    Затем вернитесь к обозревателю данных на портале Azure через браузер. Вы увидите, что добавлена база данных **GroceryStoreDatabase** с тремя пустыми контейнерами: 

    * **InventoryContainer** — запись инвентаризации для нашего примера продуктового магазина, секционированная по элементу ```id```, который является UUID.
    * **InventoryContainer-pktype** — материализованное представление записи инвентаризации, оптимизированное для запросов по элементу ```type```
    * **InventoryContainer-leases** — для канала изменений обязательно нужен контейнер аренд. Аренды позволяют отслеживать чтение канала изменений приложением.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_account_resources_lease_empty.JPG" alt-text="Пустые контейнеры":::

1. В окне терминала появится следующий запрос:

    ```bash
    Press enter to start creating the materialized view...
    ```

    Нажмите клавишу ВВОД. Теперь будет выполнен следующий блок кода, который инициализирует обработчик канала изменений в другом потоке: 

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=InitializeCFP)]

    ```"SampleHost_1"``` — это имя рабочей роли обработчика канала изменений. ```changeFeedProcessorInstance.start()``` фактически запускает обработчик канала изменений.

    Вернитесь к обозревателю данных на портале Azure через браузер. В разделе контейнера **InventoryContainer-leases** щелкните **элементы**, чтобы просмотреть его содержимое. Вы увидите, что обработчик канала изменений заполнил контейнер аренд. Это означает, что обработчик назначил рабочей роли ```SampleHost_1``` аренду некоторых секций **InventoryContainer**.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_leases.JPG" alt-text="Аренда":::

1. Снова нажмите клавишу ВВОД в окне терминала. При этом активируется вставка 10 документов в **InventoryContainer**. Каждая вставка документа отображается в канале изменений в формате JSON. Следующий код обратного вызова обрабатывает эти события путем зеркального отображения документов JSON в материализованном представлении:

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=CFPCallback)]

1. Подождите 5–10 секунд, пока код выполняется. Затем вернитесь к обозревателю данных на портале Azure и выберите раздел **InventoryContainer > элементы**. Там должно быть показано, что выполняется вставка элементов в контейнер инвентаризации. Обратите внимание на ключ секции (```id```).

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_items.JPG" alt-text="Контейнер канала":::

1. Теперь в обозревателе данных перейдите в раздел **InventoryContainer-pktype > элементы**. Это материализованное представление. Элементы в этом контейнере являются зеркальным отображением **InventoryContainer**, так как они были вставлены программными средствами с помощью канала изменений. Обратите внимание на ключ секции (```type```). Таким образом, это материализованное представление оптимизировано для фильтрации запросов по ```type```, что будет неэффективным для контейнера **InventoryContainer**, так как он секционирован по ```id```.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_materializedview2.JPG" alt-text="На снимке экрана показана обозреватель данных страница для учетной записи Azure Cosmos D B с выбранными элементами.":::

1. Мы удалим документ из **InventoryContainer** и **InventoryContainer-pktype** с помощью одного вызова ```upsertItem()```. Сначала обратите внимание на обозреватель данных на портале Azure. Мы удалим документ, для которого ```/type == "plums"```. Он обведен красной рамкой.

    :::image type="content" source="media/create-sql-api-java-changefeed/cosmos_materializedview-emph-todelete.JPG" alt-text="На снимке экрана показана обозреватель данных страница для учетной записи Azure Cosmos D B с выбранным элементом I D.":::

    Нажмите клавишу ВВОД еще раз, чтобы вызвать функцию ```deleteDocument()``` в примере кода. Эта функция, показанная ниже, выполняет для новой версии документа операцию upsert, устанавливая ```/ttl == 5```. Теперь срок жизни документа равен 5 секундам. 
    
    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-app-example/src/main/java/com/azure/cosmos/workedappexample/SampleGroceryStore.java?name=DeleteWithTTL)]

    Значение параметра ```feedPollDelay``` для канала изменений имеет значение 100 мс. Поэтому канал изменений реагирует на обновление практически мгновенно и вызывает показанную выше функцию ```updateInventoryTypeMaterializedView()```. При вызове этой функции для новой версии документа выполняется операция upsert, устанавливающая 5-секундный срок жизни документа в **InventoryContainer-pktype**.

    В результате приблизительно через 5 секунд срок жизни документа истечет и он будет удален из обоих контейнеров.

    Эта процедура необходима, так как канал изменений выдает события только при вставке или обновлении элементов, но не при их удалении.

1. Нажмите клавишу ВВОД еще раз, чтобы закрыть программу и очистить ее ресурсы.
