---
title: Apache Kafka шифрование TLS & аутентификация — Azure HDInsight
description: Настройте шифрование TLS для обмена данными между клиентами Kafka и брокерами Kafka, а также между брокерами Kafka. Настройте проверку подлинности клиентов по протоколу SSL.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 05/01/2019
ms.openlocfilehash: d061832022b983e4d5fd55e72c1d4789b82f6633
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104863230"
---
# <a name="set-up-tls-encryption-and-authentication-for-apache-kafka-in-azure-hdinsight"></a>Настройка шифрования и проверки подлинности TLS для Apache Kafka в Azure HDInsight

В этой статье показано, как настроить шифрование TLS, ранее известное как шифрование SSL (SSL) между Apache Kafka клиентами и брокерами Apache Kafka. Здесь также показано, как настроить проверку подлинности клиентов (иногда это называется двусторонним протоколом TLS).

> [!Important]
> Для приложений Kafka можно использовать два клиента: клиент Java и клиент консоли. Только клиент Java `ProducerConsumer.java` может использовать TLS как для создания, так и для использования. Клиент производителя консоли `console-producer.sh` не работает с протоколом TLS.

> [!Note]
> Производитель консоли HDInsight Kafka с версией 1,1 не поддерживает SSL.

## <a name="apache-kafka-broker-setup"></a>Установка компонента Service Broker Apache Kafka

Программа установки посредника Kafka TLS будет использовать четыре виртуальных машины кластера HDInsight следующим образом:

* головного узла 0 — центр сертификации (ЦС)
* Рабочий узел 0, 1 и 2 — брокеры

> [!Note] 
> В этом руководстве будут использоваться самозаверяющие сертификаты, но наиболее безопасными являются сертификаты, выданные доверенными центрами сертификации.

Ниже приведена сводка процесса установки компонента Service Broker.

1. Следующие шаги повторяются на каждом из трех рабочих узлов:

    1. Создайте сертификат.
    1. Создайте запрос подписи сертификата.
    1. Отправьте запрос подписи сертификата в центр сертификации (ЦС).
    1. Войдите в центр сертификации и подпишите запрос.
    1. Наscp подписанный сертификат обратно на рабочий узел.
    1. Наscp открытый сертификат ЦС на рабочий узел.

1. Получив все сертификаты, поставьте сертификаты в хранилище сертификатов.
1. Перейдите по адресу Ambari и измените настройки.

Используйте следующие подробные инструкции для завершения установки компонента Service Broker.

> [!Important]
> В следующих фрагментах кода Внкс — это сокращение для одного из трех рабочих узлов, которое должно быть заменено `wn0` `wn1` или `wn2` соответствующим образом. `WorkerNode0_Name` и `HeadNode0_Name` должны быть заменены именами соответствующих компьютеров.

1. Выполните начальную настройку на головном узле 0, который для HDInsight будет заполнять роль центра сертификации (ЦС).

    ```bash
    # Create a new directory 'ssl' and change into it
    mkdir ssl
    cd ssl
    ```

1. Выполните ту же начальную настройку для каждого брокера (рабочие узлы 0, 1 и 2).

    ```bash
    # Create a new directory 'ssl' and change into it
    mkdir ssl
    cd ssl
    ```

1. На каждом из рабочих узлов выполните следующие действия с помощью приведенного ниже фрагмента кода.
    1. Создайте хранилище ключей и заполните его новым частным сертификатом.
    1. Создайте запрос подписи сертификата.
    1. SCP — запрос подписи сертификата в ЦС (headnode0)

    ```bash
    keytool -genkey -keystore kafka.server.keystore.jks -validity 365 -storepass "MyServerPassword123" -keypass "MyServerPassword123" -dname "CN=FQDN_WORKER_NODE" -storetype pkcs12
    keytool -keystore kafka.server.keystore.jks -certreq -file cert-file -storepass "MyServerPassword123" -keypass "MyServerPassword123"
    scp cert-file sshuser@HeadNode0_Name:~/ssl/wnX-cert-sign-request
    ```

1. На компьютере ЦС выполните следующую команду, чтобы создать файлы сертификатов и ключей ЦС:

    ```bash
    openssl req -new -newkey rsa:4096 -days 365 -x509 -subj "/CN=Kafka-Security-CA" -keyout ca-key -out ca-cert -nodes
    ```

1. Перейдите на компьютер ЦС и подпишите все полученные запросы на подписывание сертификатов:

    ```bash
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn0-cert-sign-request -out wn0-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn1-cert-sign-request -out wn1-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn2-cert-sign-request -out wn2-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    ```

1. Отправьте подписанные сертификаты обратно рабочим узлам из центра сертификации (headnode0).

    ```bash
    scp wn0-cert-signed sshuser@WorkerNode0_Name:~/ssl/cert-signed
    scp wn1-cert-signed sshuser@WorkerNode1_Name:~/ssl/cert-signed
    scp wn2-cert-signed sshuser@WorkerNode2_Name:~/ssl/cert-signed
    ```

1. Отправьте открытый сертификат ЦС на каждый рабочий узел.

    ```bash
    scp ca-cert sshuser@WorkerNode0_Name:~/ssl/ca-cert
    scp ca-cert sshuser@WorkerNode1_Name:~/ssl/ca-cert
    scp ca-cert sshuser@WorkerNode2_Name:~/ssl/ca-cert
    ```

1. На каждом рабочем узле добавьте общедоступный сертификат CAs в trustStore и хранилище ключей. Затем добавьте в хранилище ключей собственный подписанный сертификат рабочего узла.

    ```bash
    keytool -keystore kafka.server.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt
    keytool -keystore kafka.server.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt
    keytool -keystore kafka.server.keystore.jks -import -file cert-signed -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt

    ```

## <a name="update-kafka-configuration-to-use-tls-and-restart-brokers"></a>Обновление конфигурации Kafka для использования TLS и перезапуска брокеров

Теперь вы настроили каждый брокер Kafka с хранилищем ключей и trustStore и импортировали правильные сертификаты. Далее измените связанные свойства конфигурации Kafka с помощью Ambari, а затем перезапустите брокеры Kafka.

Выполните следующие действия для завершения конфигурации:

1. Войдите на портал Azure и выберите свой кластер Azure HDInsight Apache Kafka.
1. Перейдите в пользовательский интерфейс Ambari, щелкнув **домашнюю страницу Ambari** в разделе **Панели мониторинга кластера**.
1. В разделе **Kafka Broker** (Брокер Kafka) задайте для свойства **listeners** значение `PLAINTEXT://localhost:9092,SSL://localhost:9093`.
1. В разделе **Advanced kafka-broker** (Расширенный брокер Kafka) задайте для свойства **security.inter.broker.protocol** значение `SSL`.

    :::image type="content" source="./media/apache-kafka-ssl-encryption-authentication/editing-configuration-ambari.png" alt-text="Изменение свойств конфигурации Kafka SSL в Ambari" border="true":::

1. В разделе **Custom kafka-broker** (Пользовательский брокер Kafka) задайте для свойства **ssl.client.auth** значение `required`. Этот шаг необходим только при настройке проверки подлинности и шифрования.

    :::image type="content" source="./media/apache-kafka-ssl-encryption-authentication/editing-configuration-ambari2.png" alt-text="Изменение свойств конфигурации SSL для Kafka в Ambari" border="true":::

1. Для HDI версии 3,6 Перейдите в пользовательский интерфейс Ambari и добавьте следующие конфигурации в разделе **Advanced Kafka-env** и свойство **шаблона Kafka-env** .

    ```bash
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092,SSL://$IP_ADDRESS:9093" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.keystore.location=/home/sshuser/ssl/kafka.server.keystore.jks" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.keystore.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.key.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.truststore.location=/home/sshuser/ssl/kafka.server.truststore.jks" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.truststore.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

1. Ниже приведен снимок экрана, на котором показан пользовательский интерфейс настройки Ambari с этими изменениями.

    Для HDI версии 3,6:

    :::image type="content" source="./media/apache-kafka-ssl-encryption-authentication/editing-configuration-kafka-env.png" alt-text="Изменение свойства шаблона Kafka-env в Ambari" border="true":::

    Для HDI версии 4,0:

     :::image type="content" source="./media/apache-kafka-ssl-encryption-authentication/editing-configuration-kafka-env-four.png" alt-text="Изменение свойства шаблона Kafka-env в Ambari 4" border="true":::

1. Перезапустите все брокеры Kafka.

## <a name="client-setup-without-authentication"></a>Установка клиента (без проверки подлинности)

Если проверка подлинности не требуется, то сводка действий по настройке только шифрования TLS:

1. Войдите в центр сертификации (активный головной узел).
1. Скопируйте сертификат ЦС на клиентский компьютер с компьютера ЦС (WN0).
1. Войдите на клиентский компьютер (HN1) и перейдите в `~/ssl` папку.
1. Импортируйте сертификат ЦС в trustStore.
1. Импортируйте сертификат ЦС в хранилище ключей.

Эти действия подробно описаны в следующих фрагментах кода.

1. Войдите в узел центра сертификации.

    ```bash
    ssh sshuser@HeadNode0_Name
    cd ssl
    ```

1. Копирование сертификата CA-CERT на клиентский компьютер

    ```bash
    scp ca-cert sshuser@HeadNode1_Name:~/ssl/ca-cert
    ```

1. Войдите на клиентский компьютер (резервный головной узел).

    ```bash
    ssh sshuser@HeadNode1_Name
    cd ssl
    ```

1. Импортируйте сертификат ЦС в trustStore.

    ```bash
    keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    ```

1. Импортируйте сертификат ЦС в хранилище ключей.
    
    ```bash
    keytool -keystore kafka.client.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    ```

1. Создайте файл `client-ssl-auth.properties` на клиентском компьютере (HN1). В нем должны присутствовать следующие строки.

    ```config
    security.protocol=SSL
    ssl.truststore.location=/home/sshuser/ssl/kafka.client.truststore.jks
    ssl.truststore.password=MyClientPassword123
    ```

1. Запустите клиент администрирования с параметрами Producer и Consumer, чтобы убедиться, что оба производителя и потребитель работают с портом 9093. Шаги, необходимые для проверки установки с помощью Console Producer или Consumer, см. в разделе " [Проверка](apache-kafka-ssl-encryption-authentication.md#verification) " ниже.

## <a name="client-setup-with-authentication"></a>Установка клиента (с проверкой подлинности)

> [!Note]
> Следующие шаги необходимы только при настройке шифрования **и** проверки подлинности TLS. Если вы настраиваете шифрование только, то см. раздел [Настройка клиента без проверки подлинности](apache-kafka-ssl-encryption-authentication.md#client-setup-without-authentication).

Следующие четыре шага обобщаются задачи, необходимые для завершения установки клиента.

1. Войдите на клиентский компьютер (резервный головной узел).
1. Создайте хранилище ключей Java и получите подписанный сертификат для брокера. Затем скопируйте сертификат на виртуальную машину, где запущен центр сертификации.
1. Чтобы подписать сертификат клиента, переключитесь на компьютер ЦС (активный головной узел).
1. Перейдите на клиентский компьютер (резервный головной узел) и перейдите к `~/ssl` папке. Скопируйте подписанный сертификат на клиентский компьютер.

Подробные сведения о каждом шаге приведены ниже.

1. Войдите на клиентский компьютер (резервный головной узел).

    ```bash
    ssh sshuser@HeadNode1_Name
    ```

1. Удалите любой существующий каталог SSL.

    ```bash
    rm -R ~/ssl
    mkdir ssl
    cd ssl
    ```

1. Создайте хранилище ключей Java и создайте запрос подписи сертификата. 

    ```bash
    keytool -genkey -keystore kafka.client.keystore.jks -validity 365 -storepass "MyClientPassword123" -keypass "MyClientPassword123" -dname "CN=HEADNODE1_FQDN" -storetype pkcs12
    
    keytool -keystore kafka.client.keystore.jks -certreq -file client-cert-sign-request -storepass "MyClientPassword123" -keypass "MyClientPassword123"
    ```

1. Копирование запроса подписи сертификата в ЦС

    ```bash
    scp client-cert-sign-request sshuser@HeadNode0_Name:~/ssl/client-cert-sign-request
    ```

1. Переключитесь на компьютер ЦС (активный головной узел) и подпишите сертификат клиента.

    ```bash
    ssh sshuser@HeadNode0_Name
    cd ssl
    openssl x509 -req -CA ca-cert -CAkey ca-key -in ~/ssl/client-cert-sign-request -out ~/ssl/client-cert-signed -days 365 -CAcreateserial -passin pass:MyClientPassword123
    ```

1. Скопируйте подписанный сертификат клиента из центра сертификации (активного головного узла) на клиентский компьютер.

    ```bash
    scp client-cert-signed sshuser@HeadNode1_Name:~/ssl/client-signed-cert
    ```

1. Копирование сертификата CA-CERT на клиентский компьютер

    ```bash
    scp ca-cert sshuser@HeadNode1_Name:~/ssl/ca-cert
    ```

    1. Войдите на клиентский компьютер (резервный головной узел) и перейдите к каталогу SSL.

    ```bash
    ssh sshuser@HeadNode1_Name
    cd ssl
    ```

1. Создайте хранилище клиента с подписанным сертификатом и импортируйте сертификат ЦС в хранилище ключей и trustStore на клиентском компьютере (HN1):

    ```bash
    keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    
    keytool -keystore kafka.client.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    
    keytool -keystore kafka.client.keystore.jks -import -file client-cert-signed -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    ```

1. Создайте файл `client-ssl-auth.properties` на клиентском компьютере (HN1). В нем должны присутствовать следующие строки.

    ```bash
    security.protocol=SSL
    ssl.truststore.location=/home/sshuser/ssl/kafka.client.truststore.jks
    ssl.truststore.password=MyClientPassword123
    ssl.keystore.location=/home/sshuser/ssl/kafka.client.keystore.jks
    ssl.keystore.password=MyClientPassword123
    ssl.key.password=MyClientPassword123
    ```

## <a name="verification"></a>Проверка

Выполните эти действия на клиентском компьютере.

> [!Note]
> Если установлены HDInsight 4,0 и Kafka 2,1, для проверки установки можно воспользоваться производителем или потребителями консоли. В противном случае запустите производитель Kafka на порте 9092 и отправьте сообщения в раздел, а затем используйте потребитель Kafka на порте 9093, который использует TLS.

### <a name="kafka-21-or-above"></a>Kafka 2,1 или более поздней версии

1. Создайте раздел, если он еще не существует.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --zookeeper <ZOOKEEPER_NODE>:2181 --create --topic topic1 --partitions 2 --replication-factor 2
    ```

1. Запустите Producer Console и укажите путь в `client-ssl-auth.properties` качестве файла конфигурации для производителя.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list <FQDN_WORKER_NODE>:9093 --topic topic1 --producer.config ~/ssl/client-ssl-auth.properties
    ```

1. Откройте другое SSH-подключение к клиентскому компьютеру и запустите консоль-потребитель и укажите путь к `client-ssl-auth.properties` файлу конфигурации для потребителя.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server <FQDN_WORKER_NODE>:9093 --topic topic1 --consumer.config ~/ssl/client-ssl-auth.properties --from-beginning
    ```

### <a name="kafka-11"></a>Kafka 1,1

1. Создайте раздел, если он еще не существует.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --zookeeper <ZOOKEEPER_NODE_0>:2181 --create --topic topic1 --partitions 2 --replication-factor 2
    ```

1. Запустите Producer Console и укажите путь к свойствам Client-SSL-auth. Properties в качестве файла конфигурации для производителя.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list <FQDN_WORKER_NODE>:9092 --topic topic1 
    ```

1. Откройте другое SSH-подключение к клиентскому компьютеру и запустите консоль-потребитель и укажите путь к `client-ssl-auth.properties` файлу конфигурации для потребителя.

    ```bash
    $ /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server <FQDN_WORKER_NODE>:9093 --topic topic1 --consumer.config ~/ssl/client-ssl-auth.properties --from-beginning
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Что такое Apache Kafka в HDInsight?](apache-kafka-introduction.md)
