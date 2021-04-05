---
title: Центры событий Azure. Обработка событий Apache Kafka
description: Руководство по В этой статье показано, как обрабатывать события Kafka, которые принимаются через Центры событий, с помощью Azure Stream Analytics
ms.topic: tutorial
ms.date: 06/23/2020
ms.openlocfilehash: 328537eebd05391fc4c8138395a9c10f0a5d072f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92319394"
---
# <a name="tutorial-process-apache-kafka-for-event-hubs-events-using-stream-analytics"></a>Руководство по Обработка событий Центров событий в Apache Kafka с использованием Stream Analytics 
В этой статье показано, как выполняется потоковая передача данных в Центры событий и обработка этих данных с помощью Azure Stream Analytics. Здесь подробно описаны следующие действия: 

1. Создайте пространство имен Центров событий.
2. создание клиента Kafka, который отправляет сообщения в концентратор событий;
3. создание задания Stream Analytics, которое копирует данные из концентратора событий в хранилище BLOB-объектов Azure. 

Вам не обязательно изменять протокол клиентов или запускать собственные кластеры при использовании конечной точки Kafka, предоставленной концентратором событий. Центры событий Azure поддерживают [Apache Kafka 1.0.](https://kafka.apache.org/10/documentation.html) и более поздние версии. 


## <a name="prerequisites"></a>Предварительные требования

Ниже указаны требования для работы с этим кратким руководством.

* Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начать работу.
* [Комплект разработчика Java (JDK) 1.7+](/azure/developer/java/fundamentals/java-jdk-long-term-support).
* [Скачайте](https://maven.apache.org/download.cgi) и [установите](https://maven.apache.org/install.html) двоичный архив Maven.
* [Git](https://www.git-scm.com/);
* **Учетная запись хранения Azure**. Если ее у вас нет, то, прежде чем продолжить, [создайте учетную запись хранения Azure](../storage/common/storage-account-create.md). В этом пошаговом руководстве задание Stream Analytics состоит в сохранении выходных данных в хранилище BLOB-объектов Azure. 


## <a name="create-an-event-hubs-namespace"></a>Создание пространства имен в Центрах событий
Если вы создаете пространство имен в Центрах событий уровня **Стандартный**, конечная точка Kafka для пространства имен включается автоматически. Вы можете выполнять потоковую передачу событий из приложений, использующих протокол Kafka, в Центры событий уровня "Стандартный". Выполните пошаговые инструкции в статье [Краткое руководство. Создание концентратора событий с помощью портала Azure](event-hubs-create.md), чтобы создать пространство имен концентраторов событий уровня **Стандартный**. 

> [!NOTE]
> Центры событий Azure для Kafka доступны только на уровнях **Стандартный** и **Выделенный**. Уровень **Базовый** не поддерживает Kafka в Центрах событий.

## <a name="send-messages-with-kafka-in-event-hubs"></a>Отправка сообщений с использованием Kafka в Центрах событий

1. Клонируйте [репозиторий Центров событий Azure для Kafka](https://github.com/Azure/azure-event-hubs-for-kafka) на компьютер.
2. Перейдите в папку `azure-event-hubs-for-kafka/quickstart/java/producer`. 
4. Обновите сведения о конфигурации для отправителя в файле по адресу `src/main/resources/producer.config`. Укажите **имя** и **строку подключения** для **пространства имен концентратора событий**. 

    ```xml
    bootstrap.servers={EVENT HUB NAMESPACE}.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{CONNECTION STRING for EVENT HUB NAMESPACE}";
    ```

5. Перейдите в `azure-event-hubs-for-kafka/quickstart/java/producer/src/main/java/` и откройте файл **TestDataReporter.java** в любом редакторе по своему усмотрению. 
6. Закомментируйте следующую строку кода.

    ```java
                //final ProducerRecord<Long, String> record = new ProducerRecord<Long, String>(TOPIC, time, "Test Data " + i);
    ```
3. Добавьте следующую строку кода вместо закомментированного кода. 

    ```java
                final ProducerRecord<Long, String> record = new ProducerRecord<Long, String>(TOPIC, time, "{ \"eventData\": \"Test Data " + i + "\" }");            
    ```

    Этот код отправляет данные события в формате **JSON**. При настройке входных данных для задания Stream Analytics необходимо указать JSON как формат входных данных. 
7. **Выполните код отправителя** и потоковую передачу данных в Центры событий. При использовании **командной строки Node.js** перед выполнением этих команд на компьютере Windows переключитесь на папку `azure-event-hubs-for-kafka/quickstart/java/producer`. 
   
    ```shell
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestProducer"                                    
    ```

## <a name="verify-that-event-hub-receives-the-data"></a>Проверка получения данных концентратором событий

1. Выберите **Центры событий** в разделе **Сущности**. Убедитесь, что вы видите концентратор событий с именем **test**. 

    ![Концентратор событий. Тестирование](./media/event-hubs-kafka-stream-analytics/test-event-hub.png)
2. Убедитесь, что вы видите сообщения, входящие в концентратор событий. 

    ![Концентратор событий. Сообщения](./media/event-hubs-kafka-stream-analytics/confirm-event-hub-messages.png)

## <a name="process-event-data-using-a-stream-analytics-job"></a>Обработка данных событий с помощью задания Stream Analytics
В этом разделе будет создано задание Azure Stream Analytics. Клиент Kafka отправит события в концентратор событий. Вы создадите задание Stream Analytics, которое принимает данные о событиях в качестве входных данных и выводит их в хранилище BLOB-объектов Azure. Если у вас нет **учетной записи службы хранилища Azure**, то потребуется [ее создать](../storage/common/storage-account-create.md).

Запрос в задании Stream Analytics проходит через данные без выполнения какой-либо аналитики. Можно создать запрос, преобразующий входные данные в выходные данные в другом формате или с полученными аналитическими сведениями.  

### <a name="create-a-stream-analytics-job"></a>Создание задания Stream Analytics 

1. Выберите **+ Создать ресурс**  на [портале Azure](https://portal.azure.com).
2. Выберите **Analytics** в меню **Azure Marketplace**, и затем выберите **задание Stream Analytics**. 
3. На странице **Новое задание Stream Analytics** выполните следующие действия. 
    1. Введите **имя** для задания. 
    2. Выберите свою **подписку**.
    3. Выберите **Создать новую** для **группы ресурсов** и введите имя. Вы также можете **использовать существующую** группу ресурсов. 
    4. Выберите **расположение** для задания.
    5. Щелкните **Создать**, чтобы создать задание. 

        ![Новое задание Stream Analytics](./media/event-hubs-kafka-stream-analytics/new-stream-analytics-job.png)

### <a name="configure-job-input"></a>Настройка входных данных для задания

1. В сообщении уведомления, щелкните **Перейти к ресурсу**, чтобы открыть страницу **задания Stream Analytics**. 
2. Выберите **Входные данные** в разделе **Топология задания** в меню слева.
3. Выберите **Добавить потоковый вход**, а затем выберите **Концентратор событий**. 

    ![Добавление концентратора событий в качестве входных данных](./media/event-hubs-kafka-stream-analytics/select-event-hub-input.png)
4. На странице конфигурации **входных данных концентратора событий** выполните следующие действия. 

    1. Укажите **псевдоним** для входных данных. 
    2. Выберите свою **подписку Azure**.
    3. Выберите **пространство имен концентратора событий** созданное ранее. 
    4. Выберите **тестирование** для **концентратора событий**. 
    5. Щелкните **Сохранить**. 

        ![Конфигурация входных данных концентратора событий](./media/event-hubs-kafka-stream-analytics/event-hub-input-configuration.png)

### <a name="configure-job-output"></a>Настройка выходных данных для задания 

1. Выберите **Выходные данные** в меню раздела **Топология задания**. 
2. Выберите **+ Добавить** на панели инструментов и выберите **Хранилище BLOB-объектов**
3. На странице параметров выходных данных хранилища BLOB-объектов выполните следующие действия. 
    1. Укажите **псевдоним** для выходных данных. 
    2. Выберите свою **подписку Azure**. 
    3. Выберите свою **учетную запись службы хранилища Azure**. 
    4. Введите **имя для контейнера**, в котором хранятся выходные данные из запроса Stream Analytics.
    5. Щелкните **Сохранить**.

        ![Конфигурация выходных данных хранилища BLOB-объектов](./media/event-hubs-kafka-stream-analytics/output-blob-settings.png)
 

### <a name="define-a-query"></a>Определение запроса
Настроив задание Stream Analytics для считывания входящего потока данных, необходимо создать преобразование, которое анализирует данные в реальном времени. Определите запрос преобразования с помощью [языка запросов Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference). В этом пошаговом руководстве необходимо определить запрос, который проходит через данные без выполнения какого-либо преобразования.

1. Выберите **Запрос**.
2. В окне запроса замените `[YourOutputAlias]` псевдонимом выходных данных, созданным ранее.
3. Замените `[YourInputAlias]` псевдонимом входных данных, созданным ранее. 
4. На панели инструментов щелкните **Сохранить**. 

    ![Снимок экрана: окно запроса со значениями для входных и выходных переменных.](./media/event-hubs-kafka-stream-analytics/query.png)


### <a name="run-the-stream-analytics-job"></a>Выполнение задания Stream Analytics

1. В меню слева выберите **Обзор**. 
2. Щелкните **Запуск**. 

    ![Меню «Пуск»](./media/event-hubs-kafka-stream-analytics/start-menu.png)
1. На странице **Запуск задания** выберите **Запуск**. 

    ![Страница запуска задания](./media/event-hubs-kafka-stream-analytics/start-job-page.png)
1. Подождите, пока состояние задания не изменится с **запуска** на **выполняется**. 

    ![Состояние задания. Выполняется](./media/event-hubs-kafka-stream-analytics/running.png)

## <a name="test-the-scenario"></a>Тестирование сценария
1. Запустите еще раз **отправитель Kafka**, чтобы отправить события в концентратор событий. 

    ```shell
    mvn exec:java -Dexec.mainClass="TestProducer"                                    
    ```
1. Убедитесь, что вы видите **выходные данные** сгенерированные в **хранилище BLOB-объектов Azure**. Вы увидите JSON-файл в контейнере с сотней строк, которые выглядят следующим образом. 

    ```
    {"eventData":"Test Data 0","EventProcessedUtcTime":"2018-08-30T03:27:23.1592910Z","PartitionId":0,"EventEnqueuedUtcTime":"2018-08-30T03:27:22.9220000Z"}
    {"eventData":"Test Data 1","EventProcessedUtcTime":"2018-08-30T03:27:23.3936511Z","PartitionId":0,"EventEnqueuedUtcTime":"2018-08-30T03:27:22.9220000Z"}
    {"eventData":"Test Data 2","EventProcessedUtcTime":"2018-08-30T03:27:23.3936511Z","PartitionId":0,"EventEnqueuedUtcTime":"2018-08-30T03:27:22.9220000Z"}
    ```

    Задание Azure Stream Analytics получило входные данные из концентратора событий и в этом сценарии сохранило их в хранилище BLOB-объектов Azure. 



## <a name="next-steps"></a>Дальнейшие действия
Из этой статьи вы узнали, как выполнять потоковую передачу данных в Центры событий без необходимости менять клиенты протоколов или запускать собственные кластеры. Сведения о Центрах событий Azure для Apache Kafka см. в [этом руководстве для разработчиков](apache-kafka-developer-guide.md).