---
title: Учебник. Подключение и мониторинг метрик на уровне приложения Spark для Azure Synapse
description: Учебник. Узнайте, как интегрировать имеющийся локальный сервер Prometheus с рабочей областью Azure Synapse, чтобы использовать метрики приложения Spark для Azure практически в реальном времени, с помощью соединителя Synapse для Prometheus.
services: synapse-analytics
author: jejiang
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 01/22/2021
ms.openlocfilehash: d22975199eedae353f2dc12588671ae4b54c85ab
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105109324"
---
# <a name="tutorial-connect-and-monitor-azure-synapse-spark-application-level-metrics"></a>Учебник. Подключение и мониторинг метрик на уровне приложения Spark для Azure Synapse

## <a name="overview"></a>Обзор

В этом учебнике описывается, как интегрировать имеющийся локальный сервер Prometheus с рабочей областью Azure Synapse, чтобы использовать метрики приложения Spark для Azure практически в реальном времени, с помощью соединителя Synapse для Prometheus. 

В нем также представлены интерфейсы REST API метрик для Azure Synapse. Данные метрик приложения Spark можно получать с помощью интерфейсов REST API, чтобы создать собственный набор средств мониторинга и диагностики или интегрировать их в свои системы мониторинга.

## <a name="use-azure-synapse-prometheus-connector-for-your-on-premises-prometheus-servers"></a>Использование соединителя Azure Synapse для Prometheus для локальных серверов Prometheus

[Соединитель Azure Synapse для Prometheus](https://github.com/microsoft/azure-synapse-spark-metrics) — это проект с открытым кодом. Соединитель Synapse для Prometheus использует метод обнаружения служб на основе файлов, обеспечивающий следующее:
 - Аутентификация в рабочей области Synapse с помощью субъекта-службы AAD.
 - Передача списка приложений Apache Spark в рабочую область. 
 - Получение данных метрик приложения Spark с помощью конфигурации на основе файлов Prometheus. 

### <a name="1-prerequisite"></a>1. Предварительные требования

На виртуальной машине Linux должен быть развернут сервер Prometheus.

### <a name="2-create-a-service-principal"></a>2. Создание субъекта-службы

Чтобы использовать соединитель Azure Synapse для Prometheus на локальном сервере Prometheus, выполните следующие действия для создания субъекта-службы.

#### <a name="21-create-a-service-principal"></a>2.1 Создание субъекта-службы

```bash
az ad sp create-for-rbac --name <service_principal_name>
```

Результат должен выглядеть следующим образом.

```json
{
  "appId": "abcdef...",
  "displayName": "<service_principal_name>",
  "name": "http://<service_principal_name>",
  "password": "abc....",
  "tenant": "<tenant_id>"
}
```

Запишите идентификатор appId, пароль и идентификатор арендатора.

#### <a name="22-add-corresponding-permissions-to-the-service-principal-created-in-the-above-step"></a>2.2. Добавление соответствующих разрешений для субъекта-службы, созданного на предыдущем шаге

![Снимок экрана: предоставление разрешений для управления доступом на основе ролей](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-grant-permission-srbac-new.png)

1. Войдите в свою [рабочую область Azure Synapse Analytics](https://web.azuresynapse.net/) в качестве администратора Synapse.

2. В Synapse Studio в области навигации слева выберите **Управление > Управление доступом**.

3. Нажмите кнопку "Добавить" в левом верхнем углу, чтобы **добавить назначение роли**.

4. В качестве области выберите **Рабочая область**.

5. В качестве роли выберите **Оператор вычислительной среды Synapse**.

6. В поле "Выбор пользователя" введите **<имя_субъекта-службы>** и щелкните свой субъект-службу.

7. Нажмите кнопку **Применить** (подождите 3 минуты, чтобы разрешение вступило в силу).


### <a name="3-download-the-azure-synapse-prometheus-connector"></a>3. Скачивание соединителя Azure Synapse для Prometheus

Используйте приведенные команды для установки соединителя Azure Synapse для Prometheus.

```bash
git clone https://github.com/microsoft/azure-synapse-spark-metrics.git
cd ./azure-synapse-spark-metrics/synapse-prometheus-connector/src
python pip install -r requirements.txt
```

### <a name="4-create-a-config-file-for-azure-synapse-workspaces"></a>4. Создание файла конфигурации для рабочих областей Azure Synapse

Создайте файл config.yaml в папке config и заполните следующие значения: workspace_name, tenant_id, service_principal_name и service_principal_password.
В конфигурацию YAML можно добавить несколько рабочих областей.

```yaml
workspaces:
  - workspace_name: <your_workspace_name>
    tenant_id: <tenant_id>
    service_principal_name: <service_principal_app_id>
    service_principal_password: "<service_principal_password>"
```

### <a name="5-update-the-prometheus-config"></a>5. Обновление конфигурации Prometheus

Добавьте приведенный ниже раздел конфигурации в scrape_config Prometheus и замените <your_workspace_name> именем рабочей области, а <path_to_synapse_connector> — именем клонированной папки synapse-prometheus-connector.

```yaml
- job_name: synapse-prometheus-connector
  static_configs:
  - labels:
      __metrics_path__: /metrics
      __scheme__: http
    targets:
    - localhost:8000
- job_name: synapse-workspace-<your_workspace_name>
  bearer_token_file: <path_to_synapse_connector>/output/workspace/<your_workspace_name>/bearer_token
  file_sd_configs:
  - files:
    - <path_to_synapse_connector>/output/workspace/<your_workspace_name>/application_discovery.json
    refresh_interval: 10s
  metric_relabel_configs:
  - source_labels: [ __name__ ]
    target_label: __name__
    regex: metrics_application_[0-9]+_[0-9]+_(.+)
    replacement: spark_$1
  - source_labels: [ __name__ ]
    target_label: __name__
    regex: metrics_(.+)
    replacement: spark_$1
```


### <a name="6-start-the-connector-in-the-prometheus-server-vm"></a>6. Запуск соединителя на виртуальной машине сервера Prometheus

Запустите сервер соединителя на виртуальной машине сервера Prometheus, как показано ниже.

```
python main.py
```

Подождите несколько секунд, после чего соединитель начнет работать. На странице обнаружения служб Prometheus можно увидеть "synapse-prometheus-connector".

## <a name="use-azure-synapse-prometheus-or-rest-metrics-apis-to-collect-metrics-data"></a>Использование интерфейсов API Azure Synapse для Prometheus или REST API метрик для сбора данных метрик

### <a name="1-authentication"></a>1. Аутентификация
Для получения маркера доступа можно использовать поток учетных данных клиента. Для доступа к API метрик необходимо получить маркер доступа Azure AD для субъекта-службы, который предоставляет необходимые разрешения для доступа к интерфейсам API.

| Параметр     | Обязательно | Описание                                                                                                   |
| ------------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| tenant_id     | True     | Идентификатор арендатора субъекта-службы Azure (приложения).                                                          |
| grant_type    | True     | Указывает запрашиваемый тип предоставления. В потоке предоставления учетных данных клиента этот параметр должен иметь значение client_credentials. |
| client_id     | True     | Идентификатор приложения (субъекта-службы), зарегистрированного с помощью портала Azure или Azure CLI.        |
| client_secret | True     | Секрет, созданный для приложения (субъекта-службы).                                                  |
| ресурс      | True     | Универсальный код ресурса (URI) Synapse должен иметь значение "https://dev.azuresynapse.net"                                                  |

```bash
curl -X GET -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials&client_id=<service_principal_app_id>&resource=<azure_synapse_resource_id>&client_secret=<service_principal_secret>' \
  https://login.microsoftonline.com/<tenant_id>/oauth2/token
```

Ответ выглядит следующим образом:

```json
{
  "token_type": "Bearer",
  "expires_in": "599",
  "ext_expires_in": "599",
  "expires_on": "1575500666",
  "not_before": "1575499766",
  "resource": "2ff8...f879c1d",
  "access_token": "ABC0eXAiOiJKV1Q......un_f1mSgCHlA"
}
```

### <a name="2-list-running-applications-in-the-azure-synapse-workspace"></a>2. Вывод списка выполняющихся приложений в рабочей области Azure Synapse

Чтобы получить список приложений Spark для рабочей области Synapse, можно следовать указаниям в документе [Monitoring - Get Spark Job List](/rest/api/synapse/data-plane/monitoring/getsparkjoblist) (Мониторинг: получение списка заданий Spark).


### <a name="3-collect-spark-application-metrics-with-the-prometheus-or-rest-apis"></a>3. Сбор данных метрик приложений Spark с помощью интерфейсов API Prometheus или REST API


#### <a name="collect-spark-application-metrics-with-the-prometheus-api"></a>Сбор данных метрик приложений Spark с помощью интерфейсов API Prometheus

Получите последние данные метрик указанного приложения Spark с помощью API Prometheus.

```
GET https://{endpoint}/livyApi/versions/{livyApiVersion}/sparkpools/{sparkPoolName}/sessions/{sessionId}/applications/{sparkApplicationId}/metrics/executors/prometheus?format=html
```

| Параметр          | Обязательно | Описание                                                                               |
| ------------------ | -------- | ----------------------------------------------------------------------------------------- |
| endpoint           | True     | Конечная точка разработки рабочей области, например https://myworkspace.dev.azuresynapse.net. |
| livyApiVersion     | True     | Допустимая версия API для запроса. В настоящее время это 2019-11-01-preview.                    |
| sparkPoolName      | True     | Имя пула Spark.                                                                   |
| sessionID          | True     | Идентификатор сеанса.                                                               |
| sparkApplicationId | True     | Идентификатор приложения Spark.                                                                      |

Пример запроса: 

```
GET https://myworkspace.dev.azuresynapse.net/livyApi/versions/2019-11-01-preview/sparkpools/mysparkpool/sessions/1/applications/application_1605509647837_0001/metrics/executors/prometheus?format=html
```

Пример ответа:

Код состояния: 200. Ответ выглядит следующим образом:

```
metrics_executor_rddBlocks{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_memoryUsed_bytes{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 74992
metrics_executor_diskUsed_bytes{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_totalCores{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_maxTasks{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_activeTasks{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 1
metrics_executor_failedTasks_total{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_completedTasks_total{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 2
...

```

#### <a name="collect-spark-application-metrics-with-the-rest-api"></a>Сбор данных метрик приложений Spark с помощью интерфейсов REST API

```
GET https://{endpoint}/livyApi/versions/{livyApiVersion}/sparkpools/{sparkPoolName}/sessions/{sessionId}/applications/{sparkApplicationId}/executors
```

| Параметр          | Обязательно | Описание                                                                               |
| ------------------ | -------- | ----------------------------------------------------------------------------------------- |
| endpoint           | True     | Конечная точка разработки рабочей области, например https://myworkspace.dev.azuresynapse.net. |
| livyApiVersion     | True     | Допустимая версия API для запроса. В настоящее время это 2019-11-01-preview.                    |
| sparkPoolName      | True     | Имя пула Spark.                                                                   |
| sessionID          | True     | Идентификатор сеанса.                                                               |
| sparkApplicationId | True     | Идентификатор приложения Spark.                                                                      |

Пример запроса

```
GET https://myworkspace.dev.azuresynapse.net/livyApi/versions/2019-11-01-preview/sparkpools/mysparkpool/sessions/1/applications/application_1605509647837_0001/executors
```

Пример ответа для кода состояния 200.

```json
[
    {
        "id": "driver",
        "hostPort": "f98b8fc2aea84e9095bf2616208eb672007bde57624:45889",
        "isActive": true,
        "rddBlocks": 0,
        "memoryUsed": 75014,
        "diskUsed": 0,
        "totalCores": 0,
        "maxTasks": 0,
        "activeTasks": 0,
        "failedTasks": 0,
        "completedTasks": 0,
        "totalTasks": 0,
        "totalDuration": 0,
        "totalGCTime": 0,
        "totalInputBytes": 0,
        "totalShuffleRead": 0,
        "totalShuffleWrite": 0,
        "isBlacklisted": false,
        "maxMemory": 15845975654,
        "addTime": "2020-11-16T06:55:06.718GMT",
        "executorLogs": {
            "stdout": "http://f98b8fc2aea84e9095bf2616208eb672007bde57624:8042/node/containerlogs/container_1605509647837_0001_01_000001/trusted-service-user/stdout?start=-4096",
            "stderr": "http://f98b8fc2aea84e9095bf2616208eb672007bde57624:8042/node/containerlogs/container_1605509647837_0001_01_000001/trusted-service-user/stderr?start=-4096"
        },
        "memoryMetrics": {
            "usedOnHeapStorageMemory": 75014,
            "usedOffHeapStorageMemory": 0,
            "totalOnHeapStorageMemory": 15845975654,
            "totalOffHeapStorageMemory": 0
        },
        "blacklistedInStages": []
    },
    // ...
]
```


### <a name="4-build-your-own-diagnosis-and-monitoring-tools"></a>4. Создание собственных средств диагностики и мониторинга

Интерфейсы API Prometheus и REST API позволяют получать подробные данные метрик о выполняемом приложении Spark. Данные метрик приложения можно получать с помощью интерфейсов API Prometheus и REST API. Вы можете также создать собственные средства диагностики и мониторинга, которые лучше подходят для ваших задач.
