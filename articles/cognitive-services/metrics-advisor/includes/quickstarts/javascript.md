---
title: Краткое руководство. Помощник по метрикам для JavaScript
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 11/09/2020
ms.author: mbullwin
ms.openlocfilehash: f34720697e545a4d8a5119605db2db70dff6c233
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102444974"
---
[Справочная документация](/java/api/overview/azure/ai-metricsadvisor-readme) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/metricsadvisor/ai-metrics-advisor/README.md) | [Пакет (npm)](https://www.npmjs.com/package/@azure/ai-metrics-advisor) | [Примеры](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/metricsadvisor/ai-metrics-advisor/samples)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Текущая версия [Node.js](https://nodejs.org/)
* Получив подписку Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="Создание ресурса Помощника по метрикам"  target="_blank">создайте ресурс Помощника по метрикам</a> на портале Azure, чтобы развернуть экземпляр Помощника по метрикам.  
* Собственная база данных SQL с данными временных рядов.
  
> [!TIP]
> * Пример Помощника по метрикам для JavaScript можно найти на сайте [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/metricsadvisor/ai-metrics-advisor/samples).
> * Ресурс Помощника по метрикам позволяет развернуть экземпляр службы за период от 10 до 30 минут. Когда развертывание завершится, щелкните **Перейти к ресурсу**. После развертывания можно начать работу с экземпляром Помощника по метрикам, используя веб-портал и REST API. 
> * URL-адрес для REST API на портале Azure можно найти в разделе **Обзор** ресурса. Он будет выглядеть следующим образом.
>    * `https://<instance-name>.cognitiveservices.azure.com/`
    
## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `npm init`, чтобы создать приложение узла с помощью файла `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Установите пакет NPM `@azure/ai-metrics-advisor`.

```console
npm install @azure/ai-metrics-advisor@1.0.0-beta.2
```

Файл `package.json` этого приложения будет дополнен зависимостями.

Создайте файл с именем `index.js` и импортируйте следующие библиотеки:

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/metricsadvisor/ai-metrics-advisor/samples/javascript), где размещены примеры кода для этого краткого руководства.

Создайте переменные для конечной точки Azure и ключа ресурса. 

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Помощника по метрикам, созданный в соответствии с инструкциями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключи подписки и конечная точка доступны на странице **Ключ и конечная точка** ресурса в разделе **Управление ресурсами**. <br><br>Чтобы получить ключ API, перейдите на сайт [https://metricsadvisor.azurewebsites.net](https://metricsadvisor.azurewebsites.net). Выберите нужный вариант: **Каталог**, **Подписки** и **Рабочая область** для своего ресурса и щелкните **Начать работу**. После этого вы сможете получить ключи API на сайте [https://metricsadvisor.azurewebsites.net/api-key](https://metricsadvisor.azurewebsites.net/api-key).   
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

```javascript
subscriptionKey = "<paste-your-metrics-advisor-key-here>";
apiKey ="<paste-your-metrics-advisor-api-key-here>";
endpoint = "<paste-your-metrics-advisor-endpoint-here>";
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы воплощают некоторые основные функции пакета SDK JavaScript для Помощника по метрикам.

|Имя|Описание|
|---|---|
| MetricsAdvisorClient | **Область использования:** <br> – вывод списка инцидентов; <br> – вывод списка основных причин инцидентов; <br> – извлечение исходных данных временных рядов и данных временных рядов, обогащенных службой; <br> – вывод списка оповещений; <br> –- добавление отзывов для настройки модели. |
| MetricsAdvisorAdministrationClient | **Возможные действия:** <br> – управление веб-каналами данных; <br> – создание, настройка, извлечение, вывод и удаление конфигураций оповещения об аномалиях; <br> – управление перехватчиками.  |
| DataFeed | **Данные, принимаемые Помощником по метрикам из источника данных. `DataFeed` содержит записи с такими данными:** <br> – метки времени; <br> – ноль или более измерений; <br> – одна или несколько мер.  |
| DataFeedMetric | `DataFeedMetric` — это количественная мера, используемая для отслеживания и оценки состояния конкретного бизнес-процесса. Это может быть сочетание нескольких значений временных рядов, разделенных на измерения. Например, метрика работоспособности веб-сайта может содержать измерения для числа пользователей и рынка en-us. |

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Помощника по метрикам для JavaScript:

* [аутентификация клиента](#authenticate-the-client);
* [добавление веб-канала данных](#add-a-data-feed);
* [Проверка состояния приема](#check-ingestion-status)
* [настройка обнаружения аномалий](#configure-anomaly-detection);
* [создание перехватчика](#create-a-hook);
* [создание конфигурации оповещения](#create-an-alert-configuration);
* [отправка запроса к оповещению](#query-the-alert).

## <a name="authenticate-the-client"></a>Аутентификация клиента

После получения двух ключей и адреса конечной точки вы можете воспользоваться классом MetricsAdvisorKeyCredential для аутентификации клиентов следующим образом:

```javascript
const {
  MetricsAdvisorKeyCredential,
  MetricsAdvisorClient,
  MetricsAdvisorAdministrationClient
} = require("@azure/ai-metrics-advisor");

const credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);
const client = new MetricsAdvisorClient(endpoint, credential);
const adminClient = new MetricsAdvisorAdministrationClient(endpoint, credential);
```

## <a name="add-a-data-feed"></a>Добавление веб-канала данных

Помощник по метрикам поддерживает подключение источников данных разных типов. Ниже приведен пример приема данных из SQL Server.

Замените `sql_server_connection_string` собственной строкой подключения к серверу SQL, а `query` — запросом, который возвращает ваши данные в виде отдельной метки времени. Вам также нужно скорректировать значения `metric` и `dimension` с учетом своих данных.

> [!IMPORTANT]
> Запрос должен возвращать не более одной записи для каждой комбинации измерения в каждой метке времени. Все записи, возвращаемые запросом, должны иметь одинаковые метки времени. Помощник по метрикам запустит этот запрос для каждой метки времени, чтобы принять данные. Дополнительные сведения и примеры см. в разделе с [вопросами и ответами о запросах](../../faq.md#how-do-i-write-a-valid-query-for-ingesting-my-data). 

```javascript
const {
  MetricsAdvisorKeyCredential,
  MetricsAdvisorAdministrationClient
} = require("@azure/ai-metrics-advisor");

async function main() {
  subscriptionKey = "<paste-your-metrics-advisor-key-here>";
  apiKey ="<paste-your-metrics-advisor-api-key-here>";
  endpoint = "<paste-your-metrics-advisor-endpoint-here>";
  const sqlServerConnectionString ="<sql_server_connection_string>";
  const sqlServerQuery ="<query>";
  const credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);

  const adminClient = new MetricsAdvisorAdministrationClient(endpoint, credential);

  const created = await createDataFeed(adminClient, sqlServerConnectionString, sqlServerQuery);
  console.log(`Data feed created: ${created.id}`);
}

async function createDataFeed(adminClient, sqlServerConnectionString, sqlServerQuery) {
  console.log("Creating Datafeed...");
  const dataFeed = {
    name: "test_datafeed_" + new Date().getTime().toString(),
    source: {
      dataSourceType: "SqlServer",
      dataSourceParameter: {
        connectionString: sqlServerConnectionString,
        query: sqlServerQuery
      }
    },
    granularity: {
      granularityType: "Daily"
    },
    schema: {
      metrics: [
        {
          name: "revenue",
          displayName: "revenue",
          description: "Metric1 description"
        },
        {
          name: "cost",
          displayName: "cost",
          description: "Metric2 description"
        }
      ],
      dimensions: [
        { name: "city", displayName: "city display" },
        { name: "category", displayName: "category display" }
      ],
      timestampColumn: null
    },
    ingestionSettings: {
      ingestionStartTime: new Date(Date.UTC(2020, 5, 1)),
      ingestionStartOffsetInSeconds: 0,
      dataSourceRequestConcurrency: -1,
      ingestionRetryDelayInSeconds: -1,
      stopRetryAfterInSeconds: -1
    },
    rollupSettings: {
      rollupType: "AutoRollup",
      rollupMethod: "Sum",
      rollupIdentificationValue: "__CUSTOM_SUM__"
    },
    missingDataPointFillSettings: {
      fillType: "SmartFilling"
    },
    accessMode: "Private",
    adminEmails: ["xyz@example.com"]
  };
  const result = await adminClient.createDataFeed(dataFeed);

  return result;
}
```

## <a name="check-ingestion-status"></a>Проверка состояния приема

После запуска приема данных мы можем проверить состояние приема.

```javascript
const {
  MetricsAdvisorKeyCredential,
  MetricsAdvisorAdministrationClient
} = require("@azure/ai-metrics-advisor");

async function main() {
  // You will need to set these environment variables or edit the following values
  const endpoint = "<service endpoint>";
  const subscriptionKey = "<subscription key>";
  const apiKey = "<api key>";
  const dataFeedId = "<data feed id>";
  const credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);

  const adminClient = new MetricsAdvisorAdministrationClient(endpoint, credential);
  await checkIngestionStatus(
    adminClient,
    dataFeedId,
    new Date(Date.UTC(2020, 8, 1)),
    new Date(Date.UTC(2020, 8, 12))
  );
}

async function checkIngestionStatus(adminClient, datafeedId, startTime, endTime) {
  // This shows how to use for-await-of syntax to list status
  console.log("Checking ingestion status...");
  const iterator = adminClient.listDataFeedIngestionStatus(datafeedId, startTime, endTime);
  for await (const status of iterator) {
    console.log(`  [${status.timestamp}] ${status.status} - ${status.message}`);
  }
}
```

## <a name="configure-anomaly-detection"></a>Настройка обнаружения аномалий

Нам требуется конфигурация обнаружения аномалий, чтобы определить, является ли точка данных во временных рядах аномалией. Хотя к каждой метрике автоматически применяется конфигурация обнаружения по умолчанию, вы можете отдельно настроить режимы обнаружения для своих данных, создав настраиваемую конфигурацию обнаружения аномалий.

```javascript
const {
  MetricsAdvisorKeyCredential,
  MetricsAdvisorAdministrationClient
} = require("@azure/ai-metrics-advisor");

async function main() {
  const endpoint = "<service endpoint>";
  const subscriptionKey = "<subscription key>";
  const apiKey = "<api key>";
  const metricId =  "<metric id>";
  const credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);

  const adminClient = new MetricsAdvisorAdministrationClient(endpoint, credential);

  const detectionConfig = await configureAnomalyDetectionConfiguration(adminClient, metricId);
  console.log(`Detection configuration created: ${detectionConfig.id}`);
}

async function configureAnomalyDetectionConfiguration(adminClient, metricId) {
  console.log(`Creating an anomaly detection configuration on metric '${metricId}'...`);
  const detectionConfig = {
    name: "test_detection_configuration" + new Date().getTime().toString(),
    metricId,
    wholeSeriesDetectionCondition: {
      smartDetectionCondition: {
        sensitivity: 100,
        anomalyDetectorDirection: "Both",
        suppressCondition: {
          minNumber: 1,
          minRatio: 1
        }
      }
    },
    description: "Detection configuration description"
  };
  return await adminClient.createDetectionConfig(detectionConfig);
}
```

### <a name="create-a-hook"></a>Создание перехватчика

Мы используем перехватчики для оформления подписки на оповещения в реальном времени. В этом примере мы создадим веб-перехватчик для службы "Помощник по метрикам", в который будут передаваться оповещения (с помощью POST).

```javascript

const {
  MetricsAdvisorKeyCredential,
  MetricsAdvisorAdministrationClient
} = require("@azure/ai-metrics-advisor");

async function main() {
  // You will need to set these environment variables or edit the following values
  const endpoint = "<service endpoint>";
  const subscriptionKey = "<subscription key>";
  const apiKey = "<api key>";
  const credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);

  const adminClient = new MetricsAdvisorAdministrationClient(endpoint, credential);
  const hook = await createWebhookHook(adminClient);
  console.log(`Webhook hook created: ${hook.id}`);
}

async function createWebhookHook(adminClient) {
  console.log("Creating a webhook hook");
  const hook = {
    hookType: "Webhook",
    name: "web hook " + new Date().getTime().toFixed(),
    description: "description",
    hookParameter: {
      endpoint: "https://example.com/handleAlerts", // you must enter a valid webhook url to post the alert payload
      username: "username",
      password: "password"
      // certificateKey: "certificate key",
      // certificatePassword: "certificate password"
    }
  };

  return await adminClient.createHook(hook);
}
```

##  <a name="create-an-alert-configuration"></a>Создание конфигурации оповещений

В этом примере показано, как настроить условия для активации оповещения и какие перехватчики использовать в качестве назначения для оповещения.

```javascript
const {
  MetricsAdvisorKeyCredential,
  MetricsAdvisorAdministrationClient
} = require("@azure/ai-metrics-advisor");

async function main() {
  // You will need to set these environment variables or edit the following values
  const endpoint = "<service endpoint>";
  const subscriptionKey = "<subscription key>";
  const apiKey = "<api key>";
  const detectionConfigId = "<detection id>";
  const hookId = "<hook id>";
  const credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);

  const adminClient = new MetricsAdvisorAdministrationClient(endpoint, credential);
  const alertConfig = await configureAlertConfiguration(adminClient, detectionConfigId, [hookId]);
  console.log(`Alert configuration created: ${alertConfig.id}`);
}

async function configureAlertConfiguration(adminClient, detectionConfigId, hookIds) {
  console.log("Creating a new alerting configuration...");
  const anomalyAlertConfig = {
    name: "test_alert_config_" + new Date().getTime().toString(),
    crossMetricsOperator: "AND",
    metricAlertConfigurations: [
      {
        detectionConfigurationId: detectionConfigId,
        alertScope: {
          scopeType: "All"
        },
        alertConditions: {
          severityCondition: { minAlertSeverity: "Medium", maxAlertSeverity: "High" }
        },
        snoozeCondition: {
          autoSnooze: 0,
          snoozeScope: "Metric",
          onlyForSuccessive: true
        }
      }
    ],
    hookIds,
    description: "Alerting config description"
  };
  return await adminClient.createAlertConfig(anomalyAlertConfig);
}
```

## <a name="query-the-alert"></a>Отправка запроса к оповещению

```javascript
const { MetricsAdvisorKeyCredential, MetricsAdvisorClient } = require("@azure/ai-metrics-advisor");

async function main() {
  // You will need to set these environment variables or edit the following values
  const endpoint = "<service endpoint>";
  const subscriptionKey = "<subscription key>";
  const apiKey = "<api key>";
  const alertConfigId = "<alert config id>";
  const credential = new MetricsAdvisorKeyCredential(subscriptionKey, apiKey);

  const client = new MetricsAdvisorClient(endpoint, credential);

  const alertIds = await queryAlerts(
    client,
    alertConfigId,
    new Date(Date.UTC(2020, 8, 1)),
    new Date(Date.UTC(2020, 8, 12))
  );

  if (alertIds.length > 1) {
    // query anomalies using an alert id.
    await queryAnomaliesByAlert(client, alertConfigId, alertIds[0]);
  } else {
    console.log("No alerts during the time period");
  }
}

async function queryAlerts(client, alertConfigId, startTime, endTime) {
  let alerts = [];
  const iterator = client.listAlerts(alertConfigId, startTime, endTime, "AnomalyTime");
  for await (const alert of iterator) {
    alerts.push(alert);
  }

  return alerts;
}

async function queryAnomaliesByAlert(client, alert) {
  console.log(
    `Listing anomalies for alert configuration '${alert.alertConfigId}' and alert '${alert.id}'`
  );
  const iterator = client.listAnomalies(alert);
  for await (const anomaly of iterator) {
    console.log(
      `  Anomaly ${anomaly.severity} ${anomaly.status} ${anomaly.seriesKey.dimension} ${anomaly.timestamp}`
    );
  }
}
```

### <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `node` для файла quickstart.

```console
node index.js
```
