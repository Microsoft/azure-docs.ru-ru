---
title: Краткое руководство. Помощник по метрикам для Python
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 11/09/2020
ms.author: mbullwin
ms.openlocfilehash: b6bea3fdc47bfbb74c18161a400d59453540b14d
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106081016"
---
[Справочная документация](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-ai-metricsadvisor/latest/azure.ai.metricsadvisor.html) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/metricsadvisor/azure-ai-metricsadvisor/README.md) | [Пакет (PiPy)](https://pypi.org/project/azure-ai-metricsadvisor/) | [Примеры](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/metricsadvisor/azure-ai-metricsadvisor/samples/README.md)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [Python 3.x](https://www.python.org/)
* Получив подписку Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="Создание ресурса Помощника по метрикам"  target="_blank">создайте ресурс Помощника по метрикам</a> на портале Azure, чтобы развернуть экземпляр Помощника по метрикам.  
* Собственная база данных SQL с данными временных рядов.
  
> [!TIP]
> * Пример Помощника по метрикам для Python можно найти на сайте [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/samples).
> * Ресурс Помощника по метрикам позволяет развернуть экземпляр службы за период от 10 до 30 минут. Когда развертывание завершится, щелкните **Перейти к ресурсу**. После развертывания можно начать работу с экземпляром Помощника по метрикам, используя веб-портал и REST API.
> * URL-адрес для REST API на портале Azure можно найти в разделе **Обзор** ресурса. Он будет выглядеть следующим образом.
>    * `https://<instance-name>.cognitiveservices.azure.com/` 
 
## <a name="setting-up"></a>Настройка

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

После установки Python вы можете установить клиентскую библиотеку с помощью следующей команды:

```console
pip install azure-ai-metricsadvisor --pre
```

### <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте файл Python и импортируйте следующие библиотеки.

```python
import os
import datetime
```

Создайте переменные для конечной точки Azure и ключа ресурса.

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Помощника по метрикам, созданный в соответствии с инструкциями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключи подписки и конечная точка доступны на странице **Ключ и конечная точка** ресурса в разделе **Управление ресурсами**. <br><br>Чтобы получить ключ API, перейдите на сайт [https://metricsadvisor.azurewebsites.net](https://metricsadvisor.azurewebsites.net). Выберите нужный вариант: **Каталог**, **Подписки** и **Рабочая область** для своего ресурса и щелкните **Начать работу**. После этого вы сможете получить ключи API на сайте [https://metricsadvisor.azurewebsites.net/api-key](https://metricsadvisor.azurewebsites.net/api-key).   
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

```python
subscription_key = "<paste-your-metrics-advisor-subscription-key-here>"
api_key = "<paste-your-metrics-advisor-api-key-here>"
service_endpoint = "<paste-your-metrics-advisor-endpoint-here>"
```

## <a name="object-model"></a>Объектная модель

Следующие классы обрабатывают некоторые основные функции пакета SDK Python для Помощника по метрикам.

|Имя|Описание|
|---|---|
| [MetricsAdvisorClient](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-ai-metricsadvisor/latest/azure.ai.metricsadvisor.html#azure.ai.metricsadvisor.MetricsAdvisorClient) | **Область использования:** <br> – вывод списка инцидентов; <br> – вывод списка основных причин инцидентов; <br> – извлечение исходных данных временных рядов и данных временных рядов, обогащенных службой; <br> – вывод списка оповещений; <br> –- добавление отзывов для настройки модели. |
| [MetricsAdvisorAdministrationClient](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-ai-metricsadvisor/latest/azure.ai.metricsadvisor.html#azure.ai.metricsadvisor.MetricsAdvisorAdministrationClient) | **Возможные действия:** <br> – управление веб-каналами данных; <br> – создание, настройка, извлечение, вывод и удаление конфигураций обнаружения аномалий; <br> – создание, настройка, извлечение, вывод и удаление конфигураций оповещения об аномалиях; <br> – управление перехватчиками.  |
| [DataFeed](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-ai-metricsadvisor/latest/azure.ai.metricsadvisor.models.html?highlight=datafeed#azure.ai.metricsadvisor.models.DataFeed)| **Данные, принимаемые Помощником по метрикам из источника данных. `DataFeed` содержит записи с такими данными:** <br> – метки времени; <br> – ноль или более измерений; <br> – одна или несколько мер.  |
| [DataFeedMetric](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-ai-metricsadvisor/latest/azure.ai.metricsadvisor.models.html?highlight=datafeedmetric#azure.ai.metricsadvisor.models.DataFeedMetric) | `DataFeedMetric` — это количественная мера, используемая для отслеживания и оценки состояния конкретного бизнес-процесса. Это может быть сочетание нескольких значений временных рядов, разделенных на измерения. Например, метрика работоспособности веб-сайта может содержать измерения для числа пользователей и рынка en-us. |

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Помощника по метрикам для Python:

* [аутентификация клиента](#authenticate-the-client);
* [добавление веб-канала данных](#add-a-data-feed);
* [Проверка состояния приема](#check-the-ingestion-status)
* [Настройка конфигурации обнаружения и конфигурации оповещения](#configure-anomaly-detection)
* [Создание конфигурации оповещения](#create-an-alert-configuration)
* [Запрашивание результатов обнаружения аномалий](#query-the-alert)

## <a name="authenticate-the-client"></a>Аутентификация клиента

В этом примере клиент является объектом `MetricsAdvisorAdministrationClient`, который использует вашу конечную точку и объект `MetricsAdvisorKeyCredential`, содержащий ваши ключи. Вам не нужно копировать этот пример кода. Методы, которые вы добавите позже, создадут экземпляр клиента. Альтернативный клиент называется `MetricsAdvisorClient`. Дополнительные сведения об этом клиенте можно найти в [справочной документации](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-ai-metricsadvisor/latest/azure.ai.metricsadvisor.html#azure.ai.metricsadvisor.MetricsAdvisorClient).

```python
client = MetricsAdvisorAdministrationClient(service_endpoint,
                                MetricsAdvisorKeyCredential(subscription_key, api_key))
```

## <a name="add-a-data-feed"></a>Добавление веб-канала данных

Создайте операторы импорта в новом методе, как показано в примере ниже. Замените `sql_server_connection_string` собственной строкой подключения к серверу SQL, а `query` — запросом, который возвращает ваши данные в виде отдельной метки времени. Вам также нужно скорректировать значения `DataFeedmetric` и `DataFeedDimension` с учетом своих данных.

> [!IMPORTANT]
> Запрос должен возвращать не более одной записи для каждой комбинации измерения в каждой метке времени. Все записи, возвращаемые запросом, должны иметь одинаковые метки времени. Помощник по метрикам запустит этот запрос для каждой метки времени, чтобы принять данные. Дополнительные сведения и примеры см. в разделе с [вопросами и ответами о запросах](../../faq.md#how-do-i-write-a-valid-query-for-ingesting-my-data). 

Создайте клиент со своими ключами и конечной точкой и воспользуйтесь методом `client.create_data_feed()` для настройки имени, источника, степени детализации и схемы. Вы также можете задать время приема, параметры свертки и пр.


```python
def sample_create_data_feed():
    from azure.ai.metricsadvisor import MetricsAdvisorKeyCredential, MetricsAdvisorAdministrationClient
    from azure.ai.metricsadvisor.models import (
        SQLServerDataFeed,
        DataFeedSchema,
        DataFeedMetric,
        DataFeedDimension,
        DataFeedOptions,
        DataFeedRollupSettings
    )
    sql_server_connection_string = "<replace-with-your-sql-server-connection-string>"
    query = "<replace-with-metrics-advisor-sql-server-query>"

    client = MetricsAdvisorAdministrationClient(service_endpoint,
                                  MetricsAdvisorKeyCredential(subscription_key, api_key))

    data_feed = client.create_data_feed(
    name="My data feed",
    source=SQLServerDataFeed(
        connection_string=sql_server_connection_string,
        query=query,
    ),
    granularity="Daily",
    schema=DataFeedSchema(
        metrics=[
            DataFeedMetric(name="cost", display_name="Cost"),
            DataFeedMetric(name="revenue", display_name="Revenue")
        ],
        dimensions=[
            DataFeedDimension(name="category", display_name="Category"),
            DataFeedDimension(name="city", display_name="City")
        ],
        timestamp_column="Timestamp"
    ),
    ingestion_settings=datetime.datetime(2019, 10, 1),
    options=DataFeedOptions(
        data_feed_description="cost/revenue data feed",
        rollup_settings=DataFeedRollupSettings(
            rollup_type="AutoRollup",
            rollup_method="Sum",
            rollup_identification_value="__CUSTOM_SUM__"
        ),
        missing_data_point_fill_settings=DataFeedMissingDataPointFillSettings(
            fill_type="SmartFilling"
        ),
        access_mode="Private"
    )
)

return data_feed
sample_create_data_feed()
```

## <a name="check-the-ingestion-status"></a>Проверка состояния приема

Создайте операторы импорта в новом методе, как показано в примере ниже. Замените `data_feed_id` идентификатором для созданного вами веб-канала данных. Создайте клиент со своими ключами и конечной точкой и воспользуйтесь методом `client.list_data_feed_ingestion_status()` для получения данных о ходе приема. Выведите подробные данные, например метки времени для последнего действия и успешного выполнения.


```python
    from azure.ai.metricsadvisor import MetricsAdvisorKeyCredential, MetricsAdvisorAdministrationClient

    data_feed_id = "<replace-with-your-metrics-advisor-data-feed-id>"

   client = MetricsAdvisorAdministrationClient(service_endpoint,
    MetricsAdvisorKeyCredential(subscription_key, api_key)
)

ingestion_status = client.list_data_feed_ingestion_status(
    data_feed_id,
    datetime.datetime(2020, 9, 20),
    datetime.datetime(2020, 9, 25)
)
for status in ingestion_status:
    print("Timestamp: {}".format(status.timestamp))
    print("Status: {}".format(status.status))
    print("Message: {}\n".format(status.message))

```

## <a name="configure-anomaly-detection"></a>Настройка обнаружения аномалий

Создайте операторы импорта в новом методе, как показано в примере ниже. Замените `metric_id` идентификатором метрики, которую вы хотите настроить. Создайте клиент со своими ключами и конечной точкой и воспользуйтесь `client.create_detection_configuration` для создания новой конфигурации обнаружения. В условиях пороговых значений заданы параметры для обнаружения аномалий.

```python
    from azure.ai.metricsadvisor import MetricsAdvisorKeyCredential, MetricsAdvisorAdministrationClient
    from azure.ai.metricsadvisor.models import (
        ChangeThresholdCondition,
        HardThresholdCondition,
        SmartDetectionCondition,
        SuppressCondition,
        MetricDetectionCondition,
    )
    metric_id = "replace-with-your-metric-id"

    
client = MetricsAdvisorAdministrationClient(
    service_endpoint,
    MetricsAdvisorKeyCredential(subscription_key, api_key)
)

change_threshold_condition = ChangeThresholdCondition(
    anomaly_detector_direction="Both",
    change_percentage=20,
    shift_point=10,
    within_range=True,
    suppress_condition=SuppressCondition(
        min_number=5,
        min_ratio=2
    )
)
hard_threshold_condition = HardThresholdCondition(
    anomaly_detector_direction="Up",
    upper_bound=100,
    suppress_condition=SuppressCondition(
        min_number=2,
        min_ratio=2
    )
)
smart_detection_condition = SmartDetectionCondition(
    anomaly_detector_direction="Up",
    sensitivity=10,
    suppress_condition=SuppressCondition(
        min_number=2,
        min_ratio=2
    )
)

detection_config = client.create_detection_configuration(
    name="my_detection_config",
    metric_id=metric_id,
    description="anomaly detection config for metric",
    whole_series_detection_condition=MetricDetectionCondition(
        cross_conditions_operator="OR",
        change_threshold_condition=change_threshold_condition,
        hard_threshold_condition=hard_threshold_condition,
        smart_detection_condition=smart_detection_condition
    )
)
return detection_config
```

## <a name="create-a-hook"></a>Создание перехватчика

Создайте операторы импорта в новом методе, как показано в примере ниже. Создайте клиент со своими ключами и конечной точкой и воспользуйтесь методом `client.create_hook()` для создания перехватчика. Введите описание, список адресов электронной почты, на которые будут отправляться оповещения, и внешнюю ссылку для получения оповещений.  

```python
def sample_create_hook():

    from azure.ai.metricsadvisor import MetricsAdvisorKeyCredential, MetricsAdvisorAdministrationClient
    from azure.ai.metricsadvisor.models import EmailNotificationHook

    client = MetricsAdvisorAdministrationClient(service_endpoint,
                                  MetricsAdvisorKeyCredential(subscription_key, api_key))

client = MetricsAdvisorAdministrationClient(service_endpoint,
    MetricsAdvisorKeyCredential(subscription_key, api_key))

hook = client.create_hook(
    hook=EmailNotificationHook(
        name="email hook",
        description="my email hook",
        emails_to_alert=["alertme@alertme.com"],
        external_link="https://example.com/handleAlerts", # you must enter a valid webhook url to post the alert payload
    )
)
```

##  <a name="create-an-alert-configuration"></a>Создание конфигурации оповещений

Создайте операторы импорта в новом методе, как показано в примере ниже. Замените `detection_configuration_id` идентификатором своей конфигурации обнаружения аномалий, а `hook_id` — созданным ранее перехватчиком. Создайте клиент со своими ключами и конечной точкой и воспользуйтесь методом `client.create_alert_configuration()` для создания конфигурации оповещения. 

```python
def sample_create_alert_config():
    from azure.ai.metricsadvisor import MetricsAdvisorKeyCredential, MetricsAdvisorAdministrationClient
    from azure.ai.metricsadvisor.models import (
        MetricAlertConfiguration,
        MetricAnomalyAlertScope,
        TopNGroupScope,
        MetricAnomalyAlertConditions,
        SeverityCondition,
        MetricBoundaryCondition,
        MetricAnomalyAlertSnoozeCondition
    )
    anomaly_detection_configuration_id = "<replace-with-your-detection-configuration-id"
    hook_id = "<replace-with-your-hook-id>"

    client = MetricsAdvisorAdministrationClient(
    service_endpoint,
    MetricsAdvisorKeyCredential(subscription_key, api_key)
)

alert_config = client.create_alert_configuration(
        name="my alert config",
        description="alert config description",
        cross_metrics_operator="AND",
        metric_alert_configurations=[
            MetricAlertConfiguration(
                detection_configuration_id=detection_configuration_id,
                alert_scope=MetricAnomalyAlertScope(
                    scope_type="WholeSeries"
                ),
                alert_conditions=MetricAnomalyAlertConditions(
                    severity_condition=SeverityCondition(
                        min_alert_severity="Low",
                        max_alert_severity="High"
                    )
                )
            ),
            MetricAlertConfiguration(
                detection_configuration_id=detection_configuration_id,
                alert_scope=MetricAnomalyAlertScope(
                    scope_type="TopN",
                    top_n_group_in_scope=TopNGroupScope(
                        top=10,
                        period=5,
                        min_top_count=5
                    )
                ),
                alert_conditions=MetricAnomalyAlertConditions(
                    metric_boundary_condition=MetricBoundaryCondition(
                        direction="Up",
                        upper=50
                    )
                ),
                alert_snooze_condition=MetricAnomalyAlertSnoozeCondition(
                    auto_snooze=2,
                    snooze_scope="Metric",
                    only_for_successive=True
                )
            ),
        ],
        hook_ids=[hook_id]
    )

    return alert_config
```

### <a name="query-the-alert"></a>Отправка запроса к оповещению

Создайте операторы импорта в новом методе, как показано в примере ниже. Замените `alert_id` идентификатором своего оповещения, а `alert_config_id` — идентификатором конфигурации оповещения. Создайте клиент со своими ключами и конечной точкой и воспользуйтесь методом `client.list_anomalies` для вывода аномалий по оповещению. 

```python
from azure.ai.metricsadvisor import MetricsAdvisorKeyCredential, MetricsAdvisorClient
    
alert_id = "<replace-with-your-alert-id>"
alert_config_id = "<replace-with-your-alert-configuration-id"

client = MetricsAdvisorClient(service_endpoint,
    MetricsAdvisorKeyCredential(subscription_key, api_key)
)

results = client.list_alerts(
    alert_configuration_id=alert_config_id,
    start_time=datetime.datetime(2020, 1, 1),
    end_time=datetime.datetime(2020, 9, 9),
    time_mode="AnomalyTime",
)
for result in results:
    print("Alert id: {}".format(result.id))
    print("Create on: {}".format(result.created_on))

results = client.list_anomalies(
    alert_configuration_id=alert_config_id,
    alert_id=alert_id,
)
for result in results:
    print("Create on: {}".format(result.created_on))
    print("Severity: {}".format(result.severity))
    print("Status: {}".format(result.status))
```

### <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `python` для файла quickstart.

```console
python quickstart-file.py
```
