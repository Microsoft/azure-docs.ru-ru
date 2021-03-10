---
title: Краткое руководство. Помощник по метрикам для Java
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 11/09/2020
ms.author: mbullwin
ms.openlocfilehash: 316a27c6b63d71fb87649524adb4b61bd2bd133b
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102444233"
---
[Справочная документация](https://westus2.dev.cognitive.microsoft.com/docs/services/MetricsAdvisor/) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src) | [Артефакт (Maven)](https://search.maven.org/artifact/com.azure/azure-ai-metricsadvisor) | [Примеры](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src/samples)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Текущая версия [пакета средств разработки Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* [Средство сборки Gradle](https://gradle.org/install/) или другой диспетчер зависимостей.
* Получив подписку Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="Создание ресурса Помощника по метрикам"  target="_blank">создайте ресурс Помощника по метрикам</a> на портале Azure, чтобы развернуть экземпляр Помощника по метрикам.  
* Собственная база данных SQL с данными временных рядов.
  
  
> [!TIP]
> * Пример Помощника по метрикам для Java можно найти на сайте [GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src/samples).
> * Ресурс Помощника по метрикам позволяет развернуть экземпляр службы за период от 10 до 30 минут. Когда развертывание завершится, щелкните **Перейти к ресурсу**. После развертывания можно начать работу с экземпляром Помощника по метрикам, используя веб-портал и REST API. 
> * URL-адрес для REST API на портале Azure можно найти в разделе **Обзор** ресурса. Он будет выглядеть следующим образом.
>    * `https://<instance-name>.cognitiveservices.azure.com/`
    
## <a name="setting-up"></a>Настройка

### <a name="create-a-new-gradle-project"></a>Создание проекта Gradle

В этом кратком руководстве используется диспетчер зависимостей Gradle. Подробные сведения о клиентской библиотеке можно найти в [центральном репозитории Maven](https://search.maven.org/artifact/com.azure/azure-ai-metricsadvisor).

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `gradle init` из рабочей папки. Эта команда создает необходимые файлы сборки для Gradle, включая *build.gradle.kts*, который используется во время выполнения для создания и настройки приложения.

```console
gradle init --type basic
```

Когда появится запрос на выбор **предметно-ориентированного языка**, выберите **Kotlin**.

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Найдите файл *build.gradle.kts* и откройте его в предпочитаемой интегрированной среде разработки или текстовом редакторе. Затем скопируйте и вставьте в файл эту конфигурацию сборки. Обязательно включите зависимости проекта.

```kotlin
dependencies {
    compile("com.azure:azure-ai-metricsadvisor:1.0.0-beta.2")
}
```

### <a name="create-a-java-file"></a>Создание файла Java

Создайте папку для примера приложения. Выполните следующие команды из рабочего каталога:

```console
mkdir -p src/main/java
```

Перейдите в новую папку и создайте файл с именем *MetricsAdvisorQuickstarts.java*. Откройте его в предпочитаемом редакторе или интегрированной среде разработки и добавьте следующие операторы `import`:

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src/samples), где размещены примеры кода для этого краткого руководства.

В классе приложения `MetricsAdvisorQuickstarts` создайте переменные для ключа и конечной точки вашего ресурса.


> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Помощника по метрикам, созданный в соответствии с инструкциями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключи подписки и конечная точка доступны на странице **Ключ и конечная точка** ресурса в разделе **Управление ресурсами**. <br><br>Чтобы получить ключ API, перейдите на сайт [https://metricsadvisor.azurewebsites.net](https://metricsadvisor.azurewebsites.net). Выберите нужный вариант: **Каталог**, **Подписки** и **Рабочая область** для своего ресурса и щелкните **Начать работу**. После этого вы сможете получить ключи API на сайте [https://metricsadvisor.azurewebsites.net/api-key](https://metricsadvisor.azurewebsites.net/api-key).   
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

```java
private static String SUBSCRIPTION_KEY = "<replace-with-your-metrics-advisor-subscription-key-here>";
private static String API_KEY = "<replace-with-your-metrics-advisor-api-key-here>"
private static String ENDPOINT = "<replace-with-your-metrics-advisor-endpoint-here>";
```

В методе `Main()` приложения добавьте вызовы методов, используемых в этом кратком руководстве. Они будут созданы позже.

```java
static void Main(string[] args){

    // You will create the below methods later in the quickstart
    exampleTask1();
}
```

## <a name="object-model"></a>Объектная модель

Следующие классы обрабатывают некоторые основные функции пакета SDK Помощника по метрикам для Java.

|Имя|Описание|
|---|---|
| [MetricsAdvisorClient](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/MetricsAdvisorClient.html) | **Область использования:** <br> – вывод списка инцидентов с аномалиями; <br> – вывод списка основных причин инцидентов; <br> – извлечение исходных данных временных рядов и данных временных рядов, обогащенных службой; <br> – вывод списка оповещений; <br> –- добавление отзывов для настройки модели. |
| [MetricsAdvisorAdministrationClient](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/administration/MetricsAdvisorAdministrationClient.html)| **Возможные действия:** <br> – управление веб-каналами данных; <br> – настройка конфигураций обнаружения аномалий; <br> – настройка конфигураций оповещения об аномалиях; <br> – управление перехватчиками.  |
| [DataFeed](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/models/DataFeed.html) | **Данные, принимаемые Помощником по метрикам из источника данных. `DataFeed` содержит записи с такими данными:** <br> – метки времени; <br> – ноль или более измерений; <br> – одна или несколько мер.  |
| [DataFeedMetric](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/models/DataFeedMetric.html) | `DataFeedMetric` — это количественная мера, используемая для отслеживания и оценки состояния конкретного бизнес-процесса. Это может быть сочетание нескольких значений временных рядов, разделенных на измерения. Например, метрика работоспособности веб-сайта может содержать измерения для числа пользователей и рынка en-us. |

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Помощника по метрикам для Java:

* [аутентификация клиента](#authenticate-the-client);
* [добавление веб-канала данных](#add-a-data-feed);
* [проверка состояния приема](#check-the-ingestion-status);
* [настройка обнаружения аномалий](#configure-anomaly-detection);
* [создание перехватчика](#create-a-hook);
* [создание конфигурации оповещения](#create-an-alert-configuration);
* [отправка запроса к оповещению](#query-the-alert).

## <a name="authenticate-the-client"></a>Аутентификация клиента

### <a name="create-a-metrics-advisor-client-using-metricsadvisorkeycredential"></a>Создание клиента Помощника по метрикам с помощью MetricsAdvisorKeyCredential

```java
MetricsAdvisorKeyCredential credential = new MetricsAdvisorKeyCredential(SUBSCRIPTION_KEY, API_KEY);
MetricsAdvisorClient metricsAdvisorClient = new MetricsAdvisorClientBuilder()
    .endpoint(ENDPOINT)
    .credential(credential)
    .buildClient();
```

### <a name="create-a-metrics-administration-client-using-metricsadvisorkeycredential"></a>Создание клиента администрирования метрик с помощью MetricsAdvisorKeyCredential

```java
MetricsAdvisorKeyCredential credential = new MetricsAdvisorKeyCredential(SUBSCRIPTION_KEY, API_KEY);
MetricsAdvisorAdministrationClient metricsAdvisorAdministrationClient =
    new MetricsAdvisorAdministrationClientBuilder()
        .endpoint(ENDPOINT)
        .credential(credential)
        .buildClient();

``` 

## <a name="add-a-data-feed"></a>Добавление веб-канала данных

Замените `sql_server_connection_string` собственной строкой подключения к серверу SQL, а `query` — запросом, который возвращает ваши данные в виде отдельной метки времени. Вам также нужно скорректировать значения `DataFeedMetric` и `DataFeedDimension` с учетом своих данных.

> [!IMPORTANT]
> Запрос должен возвращать не более одной записи для каждой комбинации измерения в каждой метке времени. Все записи, возвращаемые запросом, должны иметь одинаковые метки времени. Помощник по метрикам запустит этот запрос для каждой метки времени, чтобы принять данные. Дополнительные сведения и примеры см. в разделе с [вопросами и ответами о запросах](../../faq.md#how-do-i-write-a-valid-query-for-ingesting-my-data). 

```java
DataFeed dataFeed = new DataFeed()
    .setName("dataFeedName")
    .setSource(new MySqlDataFeedSource("sql_server_connection_string", "query"))
    .setGranularity(new DataFeedGranularity().setGranularityType(DataFeedGranularityType.DAILY))
    .setSchema(new DataFeedSchema(
        Arrays.asList(
            new DataFeedMetric().setName("cost"),
            new DataFeedMetric().setName("revenue")
        )).setDimensions(
        Arrays.asList(
            new DataFeedDimension().setName("city"),
            new DataFeedDimension().setName("category")
        ))
    )
    .setIngestionSettings(new DataFeedIngestionSettings(OffsetDateTime.parse("2020-01-01T00:00:00Z")))
    .setOptions(new DataFeedOptions()
        .setDescription("data feed description")
        .setRollupSettings(new DataFeedRollupSettings()
            .setRollupType(DataFeedRollupType.AUTO_ROLLUP)));
final DataFeed createdSqlDataFeed = metricsAdvisorAdminClient.createDataFeed(dataFeed);

System.out.printf("Data feed Id : %s%n", createdSqlDataFeed.getId());
System.out.printf("Data feed name : %s%n", createdSqlDataFeed.getName());
System.out.printf("Is the query user is one of data feed administrator : %s%n", createdSqlDataFeed.isAdmin());
System.out.printf("Data feed created time : %s%n", createdSqlDataFeed.getCreatedTime());
System.out.printf("Data feed granularity type : %s%n",
    createdSqlDataFeed.getGranularity().getGranularityType());
System.out.printf("Data feed granularity value : %d%n",
    createdSqlDataFeed.getGranularity().getCustomGranularityValue());
System.out.println("Data feed related metric Ids:");
createdSqlDataFeed.getMetricIds().forEach(System.out::println);
System.out.printf("Data feed source type: %s%n", createdSqlDataFeed.getSourceType());

if (SQL_SERVER_DB == createdSqlDataFeed.getSourceType()) {
    System.out.printf("Data feed sql server query: %s%n",
        ((SQLServerDataFeedSource) createdSqlDataFeed.getSource()).getQuery());
}
```

## <a name="check-the-ingestion-status"></a>Проверка состояния приема

Этот пример выполняет проверку состояния приема для ранее указанного источника веб-канала данных.

```java
String dataFeedId = "<use-the-data-feed-id-from-createdSqlDataFeed.getId()"; 

metricsAdvisorAdminClient.listDataFeedIngestionStatus(
    dataFeedId,
    new ListDataFeedIngestionOptions(
        OffsetDateTime.parse("2020-01-01T00:00:00Z"),
        OffsetDateTime.parse("2020-09-09T00:00:00Z"))
).forEach(dataFeedIngestionStatus -> {
    System.out.printf("Message : %s%n", dataFeedIngestionStatus.getMessage());
    System.out.printf("Timestamp value : %s%n", dataFeedIngestionStatus.getTimestamp());
    System.out.printf("Status : %s%n", dataFeedIngestionStatus.getStatus());
});
```
## <a name="configure-anomaly-detection"></a>Настройка обнаружения аномалий 

Этот пример демонстрирует, как пользователь может настроить конфигурацию обнаружения аномалий для своих данных.

```java
String metricId = "<metric-id-from-adding-data-feed>";

ChangeThresholdCondition changeThresholdCondition = new ChangeThresholdCondition()
    .setAnomalyDetectorDirection(AnomalyDetectorDirection.BOTH)
    .setChangePercentage(20)
    .setShiftPoint(10)
    .setWithinRange(true)
    .setSuppressCondition(new SuppressCondition().setMinNumber(1).setMinRatio(2));

HardThresholdCondition hardThresholdCondition = new HardThresholdCondition()
    .setAnomalyDetectorDirection(AnomalyDetectorDirection.DOWN)
    .setLowerBound(5.0)
    .setSuppressCondition(new SuppressCondition().setMinNumber(1).setMinRatio(1));

SmartDetectionCondition smartDetectionCondition = new SmartDetectionCondition()
    .setAnomalyDetectorDirection(AnomalyDetectorDirection.UP)
    .setSensitivity(10.0)
    .setSuppressCondition(new SuppressCondition().setMinNumber(1).setMinRatio(2));

final AnomalyDetectionConfiguration anomalyDetectionConfiguration =
    metricsAdvisorAdminClient.createMetricAnomalyDetectionConfig(
        metricId,
        new AnomalyDetectionConfiguration("My dataPoint anomaly detection configuration")
            .setDescription("anomaly detection config description")
            .setWholeSeriesDetectionCondition(
                new MetricWholeSeriesDetectionCondition()
                    .setChangeThresholdCondition(changeThresholdCondition)
                    .setHardThresholdCondition(hardThresholdCondition)
                    .setSmartDetectionCondition(smartDetectionCondition)
                    .setCrossConditionOperator(DetectionConditionsOperator.OR))
    );
```

## <a name="create-a-hook"></a>Создание перехватчика

Этот пример создает перехватчик электронной почты, который получает оповещения об инцидентах с обнаруженными аномалиями.

```java
NotificationHook emailNotificationHook = new EmailNotificationHook("email Hook")
    .setDescription("my email Hook")
    .addEmailToAlert("alertme@alertme.com")
    .setExternalLink("https://example.com/handleAlerts"); // you must enter a valid webhook url to post the alert payload

final NotificationHook notificationHook = metricsAdvisorAdminClient.createHook(emailNotificationHook);
EmailNotificationHook createdEmailHook = (EmailNotificationHook) notificationHook;
System.out.printf("Email Hook Id: %s%n", createdEmailHook.getId());
System.out.printf("Email Hook name: %s%n", createdEmailHook.getName());
System.out.printf("Email Hook description: %s%n", createdEmailHook.getDescription());
System.out.printf("Email Hook external Link: %s%n", createdEmailHook.getExternalLink());
System.out.printf("Email Hook emails to alert: %s%n",
    String.join(",", createdEmailHook.getEmailsToAlert()));
```

##  <a name="create-an-alert-configuration"></a>Создание конфигурации оповещений

В этом примере показано, как пользователь может настроить конфигурацию оповещений об обнаруженных аномалиях в своих данных.

```java
String detectionConfigurationId1 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
String detectionConfigurationId2 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
String hookId1 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
String hookId2 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

final AnomalyAlertConfiguration anomalyAlertConfiguration
    = metricsAdvisorAdminClient.createAnomalyAlertConfig(
        new AnomalyAlertConfiguration("My anomaly alert config name")
            .setDescription("alert config description")
            .setMetricAlertConfigurations(
                Arrays.asList(
                    new MetricAnomalyAlertConfiguration(detectionConfigurationId1,
                        MetricAnomalyAlertScope.forWholeSeries()),
                    new MetricAnomalyAlertConfiguration(detectionConfigurationId2,
                        MetricAnomalyAlertScope.forWholeSeries())
                        .setAlertConditions(new MetricAnomalyAlertConditions()
                            .setSeverityRangeCondition(new SeverityCondition()
                                .setMaxAlertSeverity(AnomalySeverity.HIGH)))
                ))
            .setCrossMetricsOperator(MetricAnomalyAlertConfigurationsOperator.AND)
            .setIdOfHooksToAlert(Arrays.asList(hookId1, hookId2)));
```

## <a name="query-the-alert"></a>Отправка запроса к оповещению

В этом примере показано, как пользователь может запросить оповещения, активированные для конфигурации обнаружения аномалий, и получить данные об аномалиях для таких оповещений.

```java
String alertConfigurationId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
final OffsetDateTime startTime = OffsetDateTime.parse("2020-01-01T00:00:00Z");
final OffsetDateTime endTime = OffsetDateTime.parse("2020-09-09T00:00:00Z");
metricsAdvisorClient.listAlerts(
    alertConfigurationId,
        startTime, endTime)
    .forEach(alert -> {
        System.out.printf("AnomalyAlert Id: %s%n", alert.getId());
        System.out.printf("AnomalyAlert created on: %s%n", alert.getCreatedTime());

        // List anomalies for returned alerts
        metricsAdvisorClient.listAnomaliesForAlert(
            alertConfigurationId,
            alert.getId())
            .forEach(anomaly -> {
                System.out.printf("DataPoint Anomaly was created on: %s%n", anomaly.getCreatedTime());
                System.out.printf("DataPoint Anomaly severity: %s%n", anomaly.getSeverity().toString());
                System.out.printf("DataPoint Anomaly status: %s%n", anomaly.getStatus());
                System.out.printf("DataPoint Anomaly related series key: %s%n", anomaly.getSeriesKey().asMap());
            });
    });
```

Чтобы создать приложение, выполните следующую команду:

```console
gradle build
```
### <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив цель `run`:

```console
gradle run
```