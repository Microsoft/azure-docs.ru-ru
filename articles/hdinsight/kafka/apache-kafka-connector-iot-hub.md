---
title: Использование Apache Kafka в HDInsight с Центром Интернета вещей
description: Узнайте, как использовать Apache Kafka в HDInsight с Центром Интернета вещей. Проект Центра Интернета вещей Kafka Connect предоставляет соединитель источника и приемника для Kafka. Соединитель источника считывает данные из Центра Интернета вещей, а соединитель приемника записывает данные в Центр Интернета вещей.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/26/2019
ms.openlocfilehash: 7980003dd63e5e51d87f85542029a1f25e7223df
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932870"
---
# <a name="use-apache-kafka-on-hdinsight-with-azure-iot-hub"></a>Использование Apache Kafka в HDInsight с Центром Интернета вещей

Узнайте, как использовать соединитель [Apache Kafka Connect Azure IoT Hub](https://github.com/Azure/toketi-kafka-connect-iothub) для перемещения данных между Apache Kafka в HDInsight и Центром Интернета вещей. В этом документе показано, как запускать соединитель Центра Интернета вещей из граничного узла кластера.

API Kafka Connect позволяет реализовать соединители, которые будут постоянно извлекать данные в Kafka или отправлять их из Kafka в другую систему. [Apache Kafka Connect Azure IoT Hub](https://github.com/Azure/toketi-kafka-connect-iothub) — это соединитель, который извлекает данные из Центра Интернета вещей в Apache Kafka. Он также может отправлять данные из Kafka в Центр Интернета вещей.

При извлечении из Центра Интернета вещей используется соединитель __источника__. При отправке в Центр Интернета вещей используется соединитель __приемника__. Соединитель Центра Интернета вещей может выступать как соединителем источника, так и соединителем приемника.

На следующей схеме показан поток данных между Центром Интернета вещей и Kafka в HDInsight при использовании соединителя.

![Изображение, на котором показан поток данных из Центра Интернета вещей в Kafka через соединитель](./media/apache-kafka-connector-iot-hub/iot-hub-kafka-connector-hdinsight.png)

Дополнительные сведения об API подключения см. в разделе [https://kafka.apache.org/documentation/#connect](https://kafka.apache.org/documentation/#connect) .

## <a name="prerequisites"></a>Предварительные условия

* Кластер Apache Kafka в HDInsight. Дополнительные сведения см. в документе [Приступая к работе с Apache Kafka в HDInsight](apache-kafka-get-started.md).

* Граничный узел в кластере Kafka. Дополнительные сведения см. в статье [Использование пустых граничных узлов в кластерах Hadoop в HDInsight](../hdinsight-apps-use-edge-node.md).

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

* Центр Интернета вещей Azure и устройство. В этой статье рассматривается использование [эмулятора Connect Raspberry Pi Online в центре Интернета вещей Azure](../../iot-hub/iot-hub-raspberry-pi-web-simulator-get-started.md).

* [Средство сборки Scala](https://www.scala-sbt.org/).

## <a name="build-the-connector"></a>Создание соединителя

1. Скачайте источник для соединителя из [https://github.com/Azure/toketi-kafka-connect-iothub/](https://github.com/Azure/toketi-kafka-connect-iothub/) в локальную среду.

2. В командной строке перейдите в `toketi-kafka-connect-iothub-master` каталог. Затем выполните следующую команду, чтобы собрать и упаковать проект:

    ```cmd
    sbt assembly
    ```

    Выполнение сборки займет несколько минут. Команда создает файл с именем `kafka-connect-iothub-assembly_2.11-0.7.0.jar` в `toketi-kafka-connect-iothub-master\target\scala-2.11` каталоге для проекта.

## <a name="install-the-connector"></a>Установка соединителя

1. Отправьте JAR-файл в пограничной узел вашего Kafka в кластере HDInsight. Измените приведенную ниже команду, заменив `CLUSTERNAME` фактическим именем кластера. Значения по умолчанию для учетной записи пользователя SSH и имени [пограничного узла](../hdinsight-apps-use-edge-node.md#access-an-edge-node) используются ниже, при необходимости измените.

    ```cmd
    scp kafka-connect-iothub-assembly*.jar sshuser@new-edgenode.CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. Когда файл будет скопирован, подключитесь к граничному узлу с помощью SSH:

    ```bash
    ssh sshuser@new-edgenode.CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Чтобы установить соединитель в каталог `libs` Kafka, используйте следующую команду:

    ```bash
    sudo mv kafka-connect-iothub-assembly*.jar /usr/hdp/current/kafka-broker/libs/
    ```

Для выполнения оставшихся действий Обеспечьте активное подключение SSH.

## <a name="configure-apache-kafka"></a>Настройка Apache Kafka

Чтобы настроить Kafka для запуска соединителя в автономном режиме, выполните следующие действия из подключения SSH к пограничному узлу:

1. Настройте переменную пароля. Замените PASSWORD паролем для входа в кластер, а затем введите команду:

    ```bash
    export password='PASSWORD'
    ```

1. Установите служебную программу [JQ](https://stedolan.github.io/jq/) . JQ упрощает обработку документов JSON, возвращаемых из запросов Ambari. Введите следующую команду:

    ```bash
    sudo apt -y install jq
    ```

1. Получите адреса брокеров Kafka. В вашем кластере может быть много брокеров, но вам нужно всего лишь ссылаться на один или два. Чтобы получить адрес двух узлов брокера, используйте следующую команду:

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')

    export KAFKABROKERS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    echo $KAFKABROKERS
    ```

    Скопируйте значения для последующего использования. Возвращаемое значение аналогично приведенному ниже тексту.

    `wn0-kafka.w5ijyohcxt5uvdhhuaz5ra4u5f.ex.internal.cloudapp.net:9092,wn1-kafka.w5ijyohcxt5uvdhhuaz5ra4u5f.ex.internal.cloudapp.net:9092`

1. Получите адреса узлов Apache Zookeeper. В кластере имеется несколько узлов Zookeeper, но необходимо ссылаться только на один или два. Используйте следующую команду, чтобы сохранить адреса в переменной `KAFKAZKHOSTS` :

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

1. При запуске соединителя в автономном режиме файл `/usr/hdp/current/kafka-broker/config/connect-standalone.properties` используется для обмена данными с брокерами Kafka. Чтобы изменить файл `connect-standalone.properties`, используйте следующую команду:

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-standalone.properties
    ```

1. Внесите следующие изменения:

    |Текущее значение |Новое значение | Комментировать |
    |---|---|---|
    |`bootstrap.servers=localhost:9092`|Замените `localhost:9092` значение на узлы брокера из предыдущего шага.|Настраивает изолированную конфигурацию для пограничных узлов, чтобы найти брокеры Kafka.|
    |`key.converter=org.apache.kafka.connect.json.JsonConverter`|`key.converter=org.apache.kafka.connect.storage.StringConverter`|Это изменение можно проверить с помощью отправителя консоли, входящего в состав Kafka. Для различных отправителей и потребителей вам могут понадобиться различные преобразователи. Сведения об использовании других значений преобразователей см. в разделе [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md) .|
    |`value.converter=org.apache.kafka.connect.json.JsonConverter`|`value.converter=org.apache.kafka.connect.storage.StringConverter`|То же, что и выше.|
    |Н/Д|`consumer.max.poll.records=10`|Добавить в конец файла. Это изменение предназначено для предотвращения времени ожидания в соединителе приемника, ограничивая его 10 записями за раз. Дополнительные сведения см. на веб-сайте [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).|

1. Чтобы сохранить файл, нажмите клавиши __CTRL+X__, затем — __Y__ и __ВВОД__.

1. Чтобы создать разделы для соединителя, используйте следующие команды:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic iotin --zookeeper $KAFKAZKHOSTS

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic iotout --zookeeper $KAFKAZKHOSTS
    ```

    Чтобы проверить, что разделы `iotin` и `iotout` существуют, используйте следующую команду:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    Раздел `iotin` используется для получения сообщений из Центра Интернета вещей. Раздел `iotout` используется для отправки сообщений в Центр Интернета вещей.

## <a name="get-iot-hub-connection-information"></a>Получение сведений о подключении Центра Интернета вещей

Чтобы извлечь информацию о Центре Интернета вещей, используемую соединителем, выполните следующие действия:

1. Получите конечную точку, совместимую с центрами событий, и ее имя для Центра Интернета вещей. Чтобы получить эту информацию, используйте один из указанных ниже способов:

   * __На [портале Azure](https://portal.azure.com/)__ выполните указанные ниже действия.

     1. Перейдите к Центру Интернета вещей и выберите __Конечные точки__.
     2. В разделе __Встроенные конечные точки__ выберите __События__.
     3. В разделе __Свойства__ скопируйте значения следующих полей:

         * __Имя, совместимое с концентратором событий__
         * __Конечная точка, совместимая с концентраторами событий__
         * __Секции__

        > [!IMPORTANT]  
        > Значение конечной точки на портале может содержать лишний текст, который не требуется в этом примере. Извлеките текст, который соответствует этому шаблону: `sb://<randomnamespace>.servicebus.windows.net/`.

   * __В [интерфейсе командной строки Azure](/cli/azure/get-started-with-azure-cli)__ введите следующую команду:

       ```azurecli
       az iot hub show --name myhubname --query "{EventHubCompatibleName:properties.eventHubEndpoints.events.path,EventHubCompatibleEndpoint:properties.eventHubEndpoints.events.endpoint,Partitions:properties.eventHubEndpoints.events.partitionCount}"
       ```

       Замените переменную `myhubname` именем Центра Интернета вещей. В ответ вы получите примерно такой текст:

       ```json
       "EventHubCompatibleEndpoint": "sb://ihsuprodbnres006dednamespace.servicebus.windows.net/",
       "EventHubCompatibleName": "iothub-ehub-myhub08-207673-d44b2a856e",
       "Partitions": 2
       ```

2. Получите __политику общего доступа__ и __ключ__. Для этого примера используйте __служебный__ ключ. Чтобы получить эту информацию, используйте один из указанных ниже способов:

    * __На [портале Azure](https://portal.azure.com/)__ выполните указанные ниже действия.

        1. Выберите __Политика общего доступа__, а затем выберите __службу__.
        2. Скопируйте значение __первичного ключа__ .
        3. Скопируйте значение поля __Строка подключения — первичный ключ__.

    * __В [интерфейсе командной строки Azure](/cli/azure/get-started-with-azure-cli)__ введите следующую команду:

        1. Чтобы получить значение первичного ключа, используйте следующую команду:

            ```azurecli
            az iot hub policy show --hub-name myhubname --name service --query "primaryKey"
            ```

            Замените переменную `myhubname` именем Центра Интернета вещей. Ответ — это первичный ключ для политики `service` этого центра.

        2. Чтобы получить строку подключения для политики `service`, используйте следующую команду:

            ```azurecli
            az iot hub show-connection-string --name myhubname --policy-name service --query "connectionString"
            ```

            Замените переменную `myhubname` именем Центра Интернета вещей. Ответом является строка подключения для политики `service`.

## <a name="configure-the-source-connection"></a>Настройка подключения к источнику

Чтобы настроить источник для работы с Центром Интернета вещей, выполните следующие действия из SSH-подключения к граничному узлу:

1. Создайте копию файла `connect-iot-source.properties` в каталоге `/usr/hdp/current/kafka-broker/config/`. Скачайте файл из проекта toketi-kafka-connect-iothub, используя следующую команду:

    ```bash
    sudo wget -P /usr/hdp/current/kafka-broker/config/ https://raw.githubusercontent.com/Azure/toketi-kafka-connect-iothub/master/connect-iothub-source.properties
    ```

1. Чтобы изменить файл `connect-iot-source.properties` и добавить информацию о Центре Интернета вещей, используйте следующую команду:

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-iothub-source.properties
    ```

1. В редакторе найдите и измените следующие записи:

    |Текущее значение |Изменить|
    |---|---|
    |`Kafka.Topic=PLACEHOLDER`|Замените `PLACEHOLDER` на `iotin`. Сообщения, полученные из Центра Интернета вещей, размещаются в разделе `iotin`.|
    |`IotHub.EventHubCompatibleName=PLACEHOLDER`|замените `PLACEHOLDER` именем, совместимым с Центрами событий.|
    |`IotHub.EventHubCompatibleEndpoint=PLACEHOLDER`|замените `PLACEHOLDER` конечной точкой, совместимой с Центрами событий.|
    |`IotHub.AccessKeyName=PLACEHOLDER`|Замените `PLACEHOLDER` на `service`.|
    |`IotHub.AccessKeyValue=PLACEHOLDER`|замените `PLACEHOLDER` первичным ключом политики `service`.|
    |`IotHub.Partitions=PLACEHOLDER`|замените `PLACEHOLDER` на количество секций из предыдущего шага.|
    |`IotHub.StartTime=PLACEHOLDER`|замените `PLACEHOLDER` датой в формате UTC. Это время и дата, когда соединитель начинает проверку на наличие сообщений. Формат даты — `yyyy-mm-ddThh:mm:ssZ`.|
    |`BatchSize=100`|Замените `100` на `5`. В таком случае соединитель считывает сообщения в Kafka, когда в Центре Интернета вещей появилось пять новых сообщений.|

    Пример конфигурации см. в статье [Kafka Connect Source Connector для центра Интернета вещей Azure](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md).

1. Чтобы сохранить изменения, нажмите клавиши __CTRL+X__, затем — __Y__ и __ВВОД__.

Дополнительные сведения о настройке источника соединителя см. в разделе [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Source.md) .

## <a name="configure-the-sink-connection"></a>Настройка подключения приемника

Чтобы настроить подключение приемника для работы с Центром Интернета вещей, выполните следующие действия из SSH-подключения к граничному узлу:

1. Создайте копию файла `connect-iothub-sink.properties` в каталоге `/usr/hdp/current/kafka-broker/config/`. Скачайте файл из проекта toketi-kafka-connect-iothub, используя следующую команду:

    ```bash
    sudo wget -P /usr/hdp/current/kafka-broker/config/ https://raw.githubusercontent.com/Azure/toketi-kafka-connect-iothub/master/connect-iothub-sink.properties
    ```

1. Чтобы изменить файл `connect-iothub-sink.properties` и добавить информацию о Центре Интернета вещей, используйте следующую команду:

    ```bash
    sudo nano /usr/hdp/current/kafka-broker/config/connect-iothub-sink.properties
    ```

1. В редакторе найдите и измените следующие записи:

    |Текущее значение |Изменить|
    |---|---|
    |`topics=PLACEHOLDER`|Замените `PLACEHOLDER` на `iotout`. Сообщение, записанные в раздел `iotout`, переадресовываются в Центр Интернета вещей.|
    |`IotHub.ConnectionString=PLACEHOLDER`|замените `PLACEHOLDER` строкой подключения политики `service`.|

    Пример конфигурации см. в статье [соединитель приемника Kafka Connect для центра Интернета вещей Azure](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md).

1. Чтобы сохранить изменения, нажмите клавиши __CTRL+X__, затем — __Y__ и __ВВОД__.

Дополнительные сведения о настройке приемника соединителя см. в разделе [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md) .

## <a name="start-the-source-connector"></a>Запуск соединителя источника

1. Чтобы запустить соединитель источника, используйте следующую команду из SSH-подключения к пограничному узлу:

    ```bash
    /usr/hdp/current/kafka-broker/bin/connect-standalone.sh /usr/hdp/current/kafka-broker/config/connect-standalone.properties /usr/hdp/current/kafka-broker/config/connect-iothub-source.properties
    ```

    После запуска соединителя отправьте сообщения в Центр Интернета вещей со своих устройств. Когда соединитель считывает сообщения из Центра Интернета вещей и сохраняет их в разделе Kafka, он регистрирует информацию на консоли:

    ```output
    [2017-08-29 20:15:46,112] INFO Polling for data - Obtained 5 SourceRecords from IotHub (com.microsoft.azure.iot.kafka.connect.IotHubSourceTask:39)
    [2017-08-29 20:15:54,106] INFO Finished WorkerSourceTask{id=AzureIotHubConnector-0} commitOffsets successfully in 4 ms (org.apache.kafka.connect.runtime.WorkerSourceTask:356)
    ```

    > [!NOTE]  
    > При запуске соединителя может появиться несколько предупреждений. Эти предупреждения не вызывают проблем с получением сообщений из Центра Интернета вещей.

1. Отключите соединитель через несколько минут, дважды нажав **клавиши CTRL + C** . Для завершения соединителя потребуется несколько минут.

## <a name="start-the-sink-connector"></a>Запуск соединителя приемника

Чтобы запустить соединитель приемника в автономном режиме, используйте следующую команду из SSH-подключения к граничному узлу:

```bash
/usr/hdp/current/kafka-broker/bin/connect-standalone.sh /usr/hdp/current/kafka-broker/config/connect-standalone.properties /usr/hdp/current/kafka-broker/config/connect-iothub-sink.properties
```

После запуска соединителя отобразится похожая информация:

```output
[2017-08-30 17:49:16,150] INFO Started tasks to send 1 messages to devices. (com.microsoft.azure.iot.kafka.connect.sink.
IotHubSinkTask:47)
[2017-08-30 17:49:16,150] INFO WorkerSinkTask{id=AzureIotHubSinkConnector-0} Committing offsets (org.apache.kafka.connect.runtime.WorkerSinkTask:262)
```

> [!NOTE]  
> При запуске соединителя может появиться несколько предупреждений. Эти предупреждения можно игнорировать.

## <a name="send-messages"></a>Отправка сообщений

Для отправки сообщений через соединитель, выполните следующие действия:

1. Откройте *второй* сеанс SSH в кластере Kafka:

    ```bash
    ssh sshuser@new-edgenode.CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Получите адрес брокеров Kafka для нового сеанса SSH. Замените PASSWORD паролем для входа в кластер, а затем введите команду:

    ```bash
    export password='PASSWORD'

    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')

    export KAFKABROKERS=`curl -sS -u admin:$password -G http://headnodehost:8080/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    ```

1. Для отправки сообщений в раздел `iotout` используйте следующую команду:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic iotout
    ```

    Эта команда не вернет вас к обычной командной строке Bash. Вместо этого он отправляет ввод с клавиатуры в раздел `iotout`.

1. Чтобы отправить сообщение на устройство, вставьте документ JSON в сеанс SSH для `kafka-console-producer`.

    > [!IMPORTANT]  
    > Для записи `"deviceId"` необходимо задать идентификатор устройства. В следующем примере устройство называется `myDeviceId`:

    ```json
    {"messageId":"msg1","message":"Turn On","deviceId":"myDeviceId"}
    ```

    Схема для этого документа JSON подробно описана в разделе [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md) .

    Если вы используете имитируемое устройство Raspberry Pi и оно работает, устройство регистрирует следующее сообщение:

    ```output
    Receive message: Turn On
    ```

    Повторно отправьте документ JSON, но измените значение записи `"message"`. Новое значение записывается на устройстве.

Дополнительные сведения об использовании соединителя приемника см. в разделе [https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md](https://github.com/Azure/toketi-kafka-connect-iothub/blob/master/README_Sink.md) .

## <a name="next-steps"></a>Дальнейшие действия

Из этого документа вы узнали, как использовать API Apache Kafka Connect для запуска соединителя IoT Kafka в HDInsight. Другие материалы, посвященные работе с Kafka, доступны по следующим ссылкам:

* [Использование Apache Spark с Apache Kafka в HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [Использование Apache Storm с Apache Kafka в HDInsight](../hdinsight-apache-storm-with-kafka.md)
