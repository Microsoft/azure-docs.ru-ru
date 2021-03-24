---
title: Apache Spark & Apache Kafka с Cosmos DB Azure HDInsight
description: Узнайте, как использовать структурированную потоковую передачу Apache Spark для чтения данных из Apache Kafka и как сохранить эти данные в Azure Cosmos DB. В этом примере выполняется потоковая передача данных с помощью Jupyter Notebook из Spark в HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-azurecli
ms.date: 11/18/2019
ms.openlocfilehash: d78b629e90903c58b98de86f425f0c1225d90997
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867055"
---
# <a name="use-apache-spark-structured-streaming-with-apache-kafka-and-azure-cosmos-db"></a>Использование структурированной потоковой передачи Apache Spark с Apache Kafka в Azure Cosmos DB

Узнайте, как использовать [Apache Spark](https://spark.apache.org/) [структурированной потоковой передачи](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html) для чтения данных из [Apache Kafka](https://kafka.apache.org/) в Azure HDInsight, а затем сохранить их в Azure Cosmos DB.

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) является глобально распределенной базой данных с несколькими моделями. В этом примере используется модель базы данных API SQL. Дополнительные сведения см. в документе [Добро пожаловать в базу данных Azure Cosmos DB](../cosmos-db/introduction.md).

Структурированная потоковая передача Spark — это механизм обработки потока, встроенный в Spark SQL. Он позволяет выражать потоковые вычисления так же, как пакетные вычисления статических данных. Дополнительные сведения о структурированной потоковой передаче см. в [руководстве по программированию структурированной потоковой передачи](https://spark.apache.org/docs/2.2.0/structured-streaming-programming-guide.html) на Apache.org.

> [!IMPORTANT]  
> В этом примере используется Spark 2.2 в HDInsight 3.6.
>
> Вы узнаете, как создать группу ресурсов Azure, которая содержит кластеры Spark и Kafka в HDInsight. Оба этих кластера находятся в виртуальной сети Azure, что позволяет кластеру Spark напрямую обмениваться данными с кластером Kafka.
>
> Выполнив инструкции, не забудьте удалить кластеры, чтобы избежать ненужных расходов.

## <a name="create-the-clusters"></a>Создание кластеров

Apache Kafka в HDInsight не предоставляет доступ к брокерам Kafka через общедоступный сегмент Интернета. Все объекты, обращающиеся к Kafka, должны находиться в той же виртуальной сети Azure, что и узлы в кластере Kafka. В этом примере кластеры Kafka и Spark расположены в виртуальной сети Azure. На следующей схеме показано, как взаимодействуют кластеры.

:::image type="content" source="./media/apache-kafka-spark-structured-streaming-cosmosdb/apache-spark-kafka-vnet.png" alt-text="Схема кластеров Spark и Kafka в виртуальной сети Azure" border="false":::

> [!NOTE]  
> Служба Kafka ограничена обменом данными в пределах виртуальной сети. Другие службы в кластере, например SSH и Ambari, могут быть доступны через Интернет. Дополнительные сведения об общих портах, доступных в HDInsight, см. в статье [Порты и универсальные коды ресурсов (URI), используемые кластерами HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Хотя виртуальную сеть Azure, а также кластеры Kafka и Spark можно создать вручную, проще использовать шаблон Azure Resource Manager. Выполните следующие действия, чтобы развернуть виртуальную сеть Azure, а также кластеры Kafka и Spark в подписке Azure.

1. Нажмите эту кнопку, чтобы войти в Azure и открыть шаблон на портале Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-spark-scala-kafka-cosmosdb%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="./media/apache-kafka-spark-structured-streaming-cosmosdb/resource-manager-deploy.png" alt="Deploy to Azure"/>
    </a>

    Шаблон Azure Resource Manager находится в репозитории GitHub для этого проекта ( [https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb](https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb) ).

    Этот шаблон создает следующие ресурсы:

   * Kafka в кластере HDInsight 3.6;

   * Spark в кластере HDInsight 3.6;

   * виртуальную сеть Azure, содержащую кластеры HDInsight. Виртуальная сеть, созданная шаблоном, использует адресное пространство 10.0.0.0/16.

   * базу данных API SQL Azure Cosmos DB.

    > [!IMPORTANT]  
    > Записная книжка структурированной потоковой передачи, используемая в этом примере, требует Spark в HDInsight 3.6. Если используется более ранняя версия Spark в HDInsight, возникнут ошибки при использовании этой записной книжки.

1. Используйте следующие сведения, чтобы заполнить раздел **Настраиваемое развертывание**:

    |Свойство |Значение |
    |---|---|
    |Подписка|Выберите подписку Azure.|
    |Группа ресурсов|Создайте новую группу или выберите существующую. Эта группа содержит кластер HDInsight.|
    |Имя учетной записи Cosmos DB|Это значение используется как имя учетной записи Cosmos DB. Имя может содержать только строчные буквы, цифры и знак дефиса (-). Длина — от 3 до 31 знака.|
    |Имя базового кластера|Это значение будет использоваться в качестве базового имени для кластеров Spark и Kafka. Например, если ввести **myhdi**, будет создан кластер Spark с именем __spark-myhdi__ и кластер Kafka с именем **kafka-myhdi**.|
    |Версия кластера|Версия кластера HDInsight. Этот пример протестирован с HDInsight 3.6 и может не работать с другими типами кластеров.|
    |Имя пользователя для входа в кластер|Имя администратора для кластеров Spark и Kafka.|
    |Пароль для входа в кластер|Пароль администратора для кластеров Spark и Kafka.|
    |Имя пользователя SSH|Создаваемый пользователь SSH для кластеров Spark и Kafka.|
    |Пароль SSH|Пароль пользователя SSH для кластеров Spark и Kafka.|

    :::image type="content" source="./media/apache-kafka-spark-structured-streaming-cosmosdb/hdi-custom-parameters.png" alt-text="Значения настраиваемого развертывания HDInsight":::

1. Прочтите **условия использования** и установите флажок **Я принимаю указанные выше условия**.

1. Наконец, щелкните **Приобрести**. Создание кластеров, виртуальной сети и учетной записи Cosmos DB может занять до 45 минут.

## <a name="create-the-cosmos-db-database-and-collection"></a>Создание базы данных и коллекции Cosmos DB

В этом документе данные проекта сохраняются в Cosmos DB. Перед выполнением кода необходимо сначала создать _базу данных_ и _коллекцию_ в вашем экземпляре Cosmos DB. Также необходимо получить конечную точку документа и _ключ_, используемый для аутентификации запросов к Cosmos DB.

Один из способов сделать это — воспользоваться [Azure CLI 2.0](/cli/azure/). Следующий скрипт создаст базу данных с именем `kafkadata` и коллекцию с именем `kafkacollection`. Затем он возвращает первичный ключ.

```azurecli
#!/bin/bash

# Replace 'myresourcegroup' with the name of your resource group
resourceGroupName='myresourcegroup'
# Replace 'mycosmosaccount' with the name of your Cosmos DB account name
name='mycosmosaccount'

# WARNING: If you change the databaseName or collectionName
#          then you must update the values in the Jupyter Notebook
databaseName='kafkadata'
collectionName='kafkacollection'

# Create the database
az cosmosdb sql database create --account-name $name --name $databaseName --resource-group $resourceGroupName

# Create the collection
az cosmosdb sql container create --account-name $name --database-name $databaseName --name $collectionName --partition-key-path "/my/path" --resource-group $resourceGroupName

# Get the endpoint
az cosmosdb show --name $name --resource-group $resourceGroupName --query documentEndpoint

# Get the primary key
az cosmosdb keys list --name $name --resource-group $resourceGroupName --type keys
```

Конечная точка документа и сведения о первичном ключе подобны следующему тексту:

```text
# endpoint
"https://mycosmosaccount.documents.azure.com:443/"
# key
"YqPXw3RP7TsJoBF5imkYR0QNA02IrreNAlkrUMkL8EW94YHs41bktBhIgWq4pqj6HCGYijQKMRkCTsSaKUO2pw=="
```

> [!IMPORTANT]  
> Сохраните конечную точку и значения ключей. Они понадобятся при работе с записными книжками Jupyter.

## <a name="get-the-notebooks"></a>Получение записных книжек

Код для примера, описанного в этом документе, доступен по адресу [https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb](https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb) .

## <a name="upload-the-notebooks"></a>Отправка записных книжек

Чтобы отправить записные книжки из проекта в кластер Spark в HDInsight, выполните следующие действия.

1. В веб-браузере подключитесь к Jupyter Notebook в кластере Spark. В следующем URL-адресе замените `CLUSTERNAME` именем вашего кластера __Spark__:

    ```http
    https://CLUSTERNAME.azurehdinsight.net/jupyter
    ```

    При появлении запроса введите имя администратора кластера и пароль, которые использовались при создании кластера.

2. В правой верхней части страницы нажмите кнопку __Отправить__, чтобы отправить файл __Stream-taxi-data-to-kafka.ipynb__ в кластер. Нажмите __Открыть__ для запуска отправки.

3. Найдите запись __Stream-taxi-data-to-kafka.ipynb__ в списке записных книжек, а затем нажмите расположенную рядом кнопку __Отправить__.

4. Повторите шаги 1–3, чтобы загрузить записную книжку __Stream-data-from-Kafka-to-Cosmos-DB.ipynb__.

## <a name="load-taxi-data-into-kafka"></a>Загрузка данных о поездках в такси в Kafka

После отправки файлов выберите запись __Stream-taxi-data-to-kafka.ipynb__, чтобы открыть записную книжку. Выполните действия в записной книжке, чтобы загрузить данные в Kafka.

## <a name="process-taxi-data-using-spark-structured-streaming"></a>Обработка данных о поездках в такси с помощью структурированной потоковой передачи Spark

На домашней странице [Jupyter Notebook](https://jupyter.org/) выберите запись __Stream-data-from-Kafka-to-Cosmos-DB.ipynb__. Следуйте инструкциям в записной книжке, чтобы выполнить потоковую передачу данных из Kafka в Azure Cosmos DB с помощью структурированной потоковой передачи Spark.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали, как использовать Apache Spark структурированной потоковой передачи, ознакомьтесь со следующими документами, чтобы узнать больше о работе с Apache Spark, Apache Kafka и Azure Cosmos DB.

* [Пример потоковой передачи Apache Spark (DStream) с использованием Apache Kafka (предварительная версия) в HDInsight](hdinsight-apache-spark-with-kafka.md)
* [Краткое руководство. Создание кластера Apache Spark в HDInsight с помощью шаблона](spark/apache-spark-jupyter-spark-sql.md)
* [Вас приветствует Azure Cosmos DB](../cosmos-db/introduction.md)
