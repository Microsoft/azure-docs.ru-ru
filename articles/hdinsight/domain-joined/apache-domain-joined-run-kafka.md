---
title: Руководство. Использование Apache Kafka с Корпоративным пакетом безопасности в Azure HDInsight
description: 'Учебник: сведения о настройке политик Apache Ranger для Kafka в Azure HDInsight с корпоративным пакетом безопасности.'
ms.service: hdinsight
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: bab3df857dfdac3ca3b9193bda1caea0040a4cbb
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104866987"
---
# <a name="tutorial-configure-apache-kafka-policies-in-hdinsight-with-enterprise-security-package-preview"></a>Руководство по Настройка политик Apache Kafka в HDInsight с Корпоративным пакетом безопасности (предварительная версия)

Сведения о настройке политик Apache Ranger для кластеров Apache Kafka с Корпоративным пакетом безопасности (ESP). Кластеры ESP подключены к домену, благодаря чему пользователи могут проходить аутентификацию с учетными данными домена. В этом руководстве вы создадите две политики Ranger для ограничения доступа к разделам `sales` и `marketingspend`.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание пользователей домена
> * Создание политик Ranger
> * Создание разделов в кластере Kafka
> * Создание политик Ranger

## <a name="prerequisite"></a>Предварительные требования

[Кластер Kafka в HDInsight с Корпоративным пакетом безопасности](./apache-domain-joined-configure-using-azure-adds.md).

## <a name="connect-to-apache-ranger-admin-ui"></a>Подключение к пользовательскому интерфейсу администратора Apache Ranger

1. В браузере подключитесь к интерфейсу администратора Ranger с помощью URL-адреса `https://ClusterName.azurehdinsight.net/Ranger/`. Не забудьте изменить `ClusterName` на имя вашего кластера Kafka. Учетные данные Ranger не совпадают с учетными данными кластера Hadoop. Чтобы браузеры не использовали кэшированные учетные данные Hadoop, подключитесь к пользовательскому интерфейсу администратора Ranger в новом окне браузера в режиме InPrivate.

2. Зарегистрируйтесь, используя свои учетные данные администратора Azure Active Directory (AD). Учетные данные администратора Azure AD отличаются от учетных данных кластера HDInsight или учетных данных SSH узла Linux HDInsight.

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-login.png" alt-text="Пользовательский интерфейс администратора Apache Ranger для HDInsight" border="true":::

## <a name="create-domain-users"></a>Создание пользователей домена

В разделе [Создание кластера HDInsight с корпоративным пакетом безопасности](./apache-domain-joined-configure-using-azure-adds.md), чтобы узнать, как создать пользователей домена **sales_user** и **marketing_user**. В рабочем сценарии пользователи домена берутся из вашего клиента Active Directory.

## <a name="create-ranger-policy"></a>Создание политики Ranger

Создайте политику Ranger для пользователей **sales_user** и **marketing_user**.

1. Откройте **пользовательский интерфейс администратора Ranger**.

2. Щелкните **\<ClusterName>_kafka** а разделе **Kafka**. Может быть указана одна предварительно настроенная политика.

3. Щелкните **Добавить новую политику**, а затем введите следующие значения.

   |Параметр  |Рекомендуемое значение  |
   |---------|---------|
   |Имя политики  |  политика hdi sales*   |
   |Раздел   |  sales* |
   |Выберите пользователя  |  sales_user1 |
   |Разрешения  | публикация, использование, создание |

   Имя раздела может содержать следующие подстановочные знаки:

   * \* обозначает ноль или более вхождений символов.
   * ? означает один символ

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy.png" alt-text="Политика создания пользовательского интерфейса администратора Apache Ranger1" border="true":::

   Подождите несколько минут, пока Ranger синхронизируется с Azure AD, если в поле **Выберите пользователя** автоматически не подставится пользователь домена.

4. Щелкните **Добавить**, чтобы сохранить политику.

5. Щелкните **Добавить новую политику**, а затем введите следующие значения.

   |Параметр  |Рекомендуемое значение  |
   |---------|---------|
   |Имя политики  |  маркетинговая политика hdi   |
   |Раздел   |  marketingspend |
   |Выберите пользователя  |  marketing_user1 |
   |Разрешения  | публикация, использование, создание |

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy-2.png" alt-text="Политика создания пользовательского интерфейса администратора Apache Ranger2" border="true":::  

6. Щелкните **Добавить**, чтобы сохранить политику.

## <a name="create-topics-in-a-kafka-cluster-with-esp"></a>Создание разделов в кластере Kafka с ESP

Чтобы создать разделы `salesevents` и `marketingspend` выполните следующие действия.

1. Чтобы открыть SSH-подключение к кластеру, выполните следующую команду:

   ```cmd
   ssh DOMAINADMIN@CLUSTERNAME-ssh.azurehdinsight.net
   ```

   Замените `DOMAINADMIN` пользователем с правами администратора для вашего кластера, настроенного во время [создания кластера](./apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp), и замените `CLUSTERNAME` именем вашего кластера. При появлении запроса введите пароль для учетной записи администратора. Дополнительные сведения об использовании `SSH` с HDInsight см. в статье [Подключение к HDInsight (Hadoop) с помощью SSH](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

2. Чтобы сохранить имя кластера в переменной и установить служебную программу синтаксического анализа JSON (`jq`), используйте следующие команды. При появлении запроса введите имя кластера Kafka.

   ```bash
   sudo apt -y install jq
   read -p 'Enter your Kafka cluster name:' CLUSTERNAME
   ```

3. Чтобы получить узлы брокера Kafka, используйте следующие команды. При появлении запроса введите пароль для учетной записи администратора кластера.

   ```bash
   export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`; \
   ```

   Прежде чем продолжить, вам может потребоваться настроить среду разработки, если вы еще не сделали это. Требуются такие компоненты, как Java JDK, Apache Maven и SSH-клиент с scp. Дополнительные сведения см. в [инструкциях по настройке](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer).

1. Скачайте [примеры присоединенного к домену производителя и потребителя Apache Kafka](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer).

1. Выполните шаги 2 и 3 в разделе **Создание и развертывание примера** в [руководстве по использованию API производителя и потребителя Apache Kafka](../kafka/apache-kafka-producer-consumer-api.md#build-and-deploy-the-example).

1. Выполните следующие команды:

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar create salesevents $KAFKABROKERS
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar create marketingspend $KAFKABROKERS
   ```

## <a name="test-the-ranger-policies"></a>Тестирование политик Ranger

На основе настроенных политик Ranger **sales_user** может создавать или использовать раздел `salesevents` но не `marketingspend`. И наоборот: **marketing_user** может создавать или использовать раздел `marketingspend` но не `salesevents`.

1. Откройте новое подключение SSH к кластеру. Выполните следующую команду, чтобы войти от имени пользователя **sales_user1**.

   ```bash
   ssh sales_user1@CLUSTERNAME-ssh.azurehdinsight.net
   ```

2. Используйте имена брокера из предыдущего раздела, чтобы настроить следующую переменную среды.

   ```bash
   export KAFKABROKERS=<brokerlist>:9092
   ```

   Например, `export KAFKABROKERS=wn0-khdicl.contoso.com:9092,wn1-khdicl.contoso.com:9092`.

3. Выполните шаг 3 в разделе **Создание и развертывание примера** в статье [Руководство. Используйте API производителя и потребителя Apache Kafka](../kafka/apache-kafka-producer-consumer-api.md#build-and-deploy-the-example), чтобы убедиться, что `kafka-producer-consumer.jar` также доступно для **sales_user**.

   > [!NOTE]  
   > Для работы с этим руководством используйте файл kafka-producer-consumer.jar в проекте DomainJoined-Producer-Consumer (не в проекте Producer-Consumer, который предназначен для сценариев без присоединения к домену).

4. Убедитесь, что **sales_user1** может работать с разделом `salesevents`, выполнив следующую команду.

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar producer salesevents $KAFKABROKERS
   ```

5. Выполните следующую команду, чтобы использовать ресурсы из раздела `salesevents`.

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar consumer salesevents $KAFKABROKERS
   ```

   Убедитесь, что вы можете читать сообщения.

6. Убедитесь, что **sales_user1** не может работать с разделом `marketingspend`, выполнив следующее в том же окне SSH.

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar producer marketingspend $KAFKABROKERS
   ```

   Происходит ошибка авторизации, ее можно игнорировать.

7. Обратите внимание, что пользователь **marketing_user1** не может использовать ресурсы из раздела `salesevents`.

   Повторите шаги 1–3 выше, но на этот раз от имени пользователя **marketing_user1**.

   Выполните следующую команду, чтобы использовать ресурсы из раздела `salesevents`.

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar consumer salesevents $KAFKABROKERS
   ```

   Предыдущие сообщения не будут отображаться.

8. Просмотрите события доступа к ресурсам аудита из интерфейса Ranger.

   :::image type="content" source="./media/apache-domain-joined-run-kafka/apache-ranger-admin-audit.png" alt-text="События доступа к ресурсам аудита политики в пользовательском интерфейсе Ranger" border="true":::
   
## <a name="produce-and-consume-topics-in-esp-kafka-by-using-the-console"></a>Создание и использование разделов в ESP Kafka с помощью консоли

> [!NOTE]
> Использовать команды консоли для создания разделов нельзя. Вместо этого необходимо использовать код Java, показанный в предыдущем разделе. См. раздел [Создание разделов в кластере Kafka с ESP](#create-topics-in-a-kafka-cluster-with-esp).

Для создания и использования разделов в ESP Kafka с помощью консоли выполните следующие действия.

1. Используйте `kinit` с именем пользователя. Введите пароль, когда появится запрос.

   ```bash
   kinit sales_user1
   ```

2. Задайте переменные среды.

   ```bash
   export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf"
   export KAFKABROKERS=<brokerlist>:9092
   ```

3. Создайте сообщения для раздела `salesevents`.

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --topic salesevents --broker-list $KAFKABROKERS --security-protocol SASL_PLAINTEXT
   ```

4. Получите сообщения из раздела `salesevents`.

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --topic salesevents --from-beginning --bootstrap-server $KAFKABROKERS --security-protocol SASL_PLAINTEXT
   ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите созданный кластер Kafka, сделав следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).
1. В поле **Поиск** в верхней части страницы введите **HDInsight**.
1. Выберите **Кластеры HDInsight** в разделе **Службы**.
1. В списке кластеров HDInsight, который отобразится, щелкните **...** рядом с кластером, созданным при работе с этим руководством. 
1. Щелкните **Удалить**. Нажмите кнопку **Да**.

## <a name="troubleshooting"></a>Устранение неполадок
Если файл kafka-producer-consumer.jar не работает в кластере, присоединенном к домену, используйте файл kafka-producer-consumer.jar в проекте DomainJoined-Producer-Consumer (не в проекте Producer-Consumer, который предназначен для сценариев без присоединения к домену).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Шифрование управляемых клиентом ключей](../disk-encryption.md)
