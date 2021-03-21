---
title: Настройка интеграции Prometheus контейнеров | Документация Майкрософт
description: В этой статье описывается, как настроить агент контейнеров аналитики для сбора метрик из Prometheus в кластере Kubernetes.
ms.topic: conceptual
ms.date: 04/22/2020
ms.openlocfilehash: 8affeb472b9452e4d234e99e5ea6bb4509770fac
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101731737"
---
# <a name="configure-scraping-of-prometheus-metrics-with-container-insights"></a>Настройка брака/отхода метрик Prometheus с помощью аналитики контейнеров

[Prometheus](https://prometheus.io/) — это популярное решение для мониторинга метрик с открытым исходным кодом, которое входит в состав [облачной среды вычислений машинного](https://www.cncf.io/)кода. Контейнерная аналитика обеспечивает простой процесс адаптации для сбора метрик Prometheus. Как правило, для использования Prometheus необходимо настроить сервер Prometheus и управлять им с помощью магазина. При интеграции с Azure Monitor сервер Prometheus не требуется. Вам нужно просто предоставить конечную точку метрик Prometheus с помощью средств экспорта или модулей (приложения), а контейнерный агент для аналитики контейнера может отбракировать метрики за вас. 

![Архитектура мониторинга контейнеров для Prometheus](./media/container-insights-prometheus-integration/monitoring-kubernetes-architecture.png)

>[!NOTE]
>Минимальная версия агента, поддерживаемая для бракованной Prometheus метрик, — ciprod07092019 или более поздней версии, а версия агента, поддерживаемая для записи ошибок конфигурации и агента в `KubeMonAgentEvents` таблице, — ciprod10112019. Для Azure Red Hat OpenShift и Red Hat OpenShift V4, версия агента ciprod04162020 или более поздняя. 
>
>Дополнительные сведения о версиях агентов и о том, что входит в каждый выпуск, см. в разделе [заметки о выпуске агента](https://github.com/microsoft/Docker-Provider/tree/ci_feature_prod). 
>Чтобы проверить версию агента, на вкладке **узел** выберите узел и в области свойства запишите значение свойства **тег образа агента** .

Отходы метрик Prometheus поддерживаются в кластерах Kubernetes, размещенных на:

- Служба Azure Kubernetes (AKS)
- Azure Stack или локально
- Azure Red Hat OpenShift версии 3. x
- Azure Red Hat OpenShift и Red Hat OpenShift версии 4. x

### <a name="prometheus-scraping-settings"></a>Параметры отхода Prometheus

Активные отходы метрик из Prometheus выполняются с одной из двух перспектив:

* URL-адрес на уровне кластера и обнаружение целей из указанных конечных точек службы. Например, K8S службы, такие как KUBE-DNS и KUBE-State-метрики, и заметки Pod, относящиеся к приложению. Метрики, собранные в этом контексте, будут определены в разделе ConfigMap *[Prometheus data_collection_settings. Cluster]*.
* URL-адрес на уровне узла и обнаружение целей из указанных конечных точек службы. Метрики, собранные в этом контексте, будут определены в разделе ConfigMap *[Prometheus_data_collection_settings. Node]*.

| Конечная точка | Область | Пример |
|----------|-------|---------|
| Аннотация к Pod | На уровне кластера | Примечания <br>`prometheus.io/scrape: "true"` <br>`prometheus.io/path: "/mymetrics"` <br>`prometheus.io/port: "8000"` <br>`prometheus.io/scheme: "http"` |
| Служба Kubernetes | На уровне кластера | `http://my-service-dns.my-namespace:9100/metrics` <br>`https://metrics-server.kube-system.svc.cluster.local/metrics` |
| URL-адрес или конечная точка | На узел или на уровне кластера | `http://myurl:9101/metrics` |

Если указан URL-адрес, то контейнерная аналитика будет передаваться только в конечную точку. Если указана служба Kubernetes, имя службы разрешается с помощью DNS-сервера кластера для получения IP-адреса, а затем разрешенная служба переводится в отходы.

|Область | Ключ | Тип данных | Значение | Описание |
|------|-----|-----------|-------|-------------|
| На уровне кластера | | | | Укажите один из следующих трех методов, чтобы отбракировать конечные точки для метрик. |
| | `urls` | Строка | Массив с разделителями-запятыми | Конечная точка HTTP (указан IP-адрес или допустимый путь URL-адреса). Например, `urls=[$NODE_IP/metrics]`. ($NODE _IP — это конкретный параметр Container Insights. его можно использовать вместо IP-адреса узла. Необходимо использовать все прописные буквы.) |
| | `kubernetes_services` | Строка | Массив с разделителями-запятыми | Массив Kubernetes служб для сбора метрик из KUBE-State-метрик. Например, `kubernetes_services = ["https://metrics-server.kube-system.svc.cluster.local/metrics",http://my-service-dns.my-namespace:9100/metrics]`.|
| | `monitor_kubernetes_pods` | Логическое | true или false | Если задано значение `true` в параметрах на уровне кластера, агент контейнера аналитики будет наделять модули Pod Kubernetes по всему кластеру на наличие следующих заметок Prometheus:<br> `prometheus.io/scrape:`<br> `prometheus.io/scheme:`<br> `prometheus.io/path:`<br> `prometheus.io/port:` |
| | `prometheus.io/scrape` | Логическое | true или false | Включает брак/отход модуля Pod. Для параметра `monitor_kubernetes_pods` нужно задать значение `true`. |
| | `prometheus.io/scheme` | Строка | HTTP или HTTPS | По умолчанию используется отработка отказа по протоколу HTTP. При необходимости задайте значение `https` . | 
| | `prometheus.io/path` | Строка | Массив с разделителями-запятыми | Путь HTTP-ресурса, из которого извлекать метрики. Если путь метрик не равен `/metrics` , определите его с помощью этой заметки. |
| | `prometheus.io/port` | Строка | 9102 | Укажите порт, из которого будет списано значение. Если параметр port не установлен, по умолчанию будет 9102. |
| | `monitor_kubernetes_pods_namespaces` | Строка | Массив с разделителями-запятыми | Список разрешенных пространств имен для брака метрики из модулей Kubernetes.<br> Например: `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]` |
| На уровне узла | `urls` | Строка | Массив с разделителями-запятыми | Конечная точка HTTP (указан IP-адрес или допустимый путь URL-адреса). Например, `urls=[$NODE_IP/metrics]`. ($NODE _IP — это конкретный параметр Container Insights. его можно использовать вместо IP-адреса узла. Необходимо использовать все прописные буквы.) |
| На уровне узла или на уровне кластера | `interval` | Строка | 60 | Интервал сбора по умолчанию равен одной минуте (60 секунд). Коллекцию можно изменить для *[prometheus_data_collection_settings. Node]* и (или) *[prometheus_data_collection_settings. Cluster]* на единицы времени, например s, m, h. |
| На уровне узла или на уровне кластера | `fieldpass`<br> `fielddrop`| Строка | Массив с разделителями-запятыми | Можно указать метрики, которые должны быть собраны или не включены в конечную точку, задав `fieldpass` список разрешений () и запретить ( `fielddrop` ). Сначала необходимо задать список разрешений. |

Конфигмапс является глобальным списком, и к нему может быть применен только один ConfigMap. У вас не может быть другой Конфигмапс.

## <a name="configure-and-deploy-configmaps"></a>Настройка и развертывание Конфигмапс

Выполните следующие действия, чтобы настроить файл конфигурации ConfigMap для следующих кластеров:

* Служба Azure Kubernetes (AKS)
* Azure Stack или локально
* Azure Red Hat OpenShift версии 4. x и Red Hat OpenShift версии 4. x

1. [Скачайте](https://aka.ms/container-azm-ms-agentconfig) файл шаблона ConfigMap YAML и сохраните его как Container-АЗМ-MS-ажентконфиг. YAML.

   >[!NOTE]
   >Этот шаг не требуется при работе с Azure Red Hat OpenShift, так как шаблон ConfigMap уже существует в кластере.

2. Измените файл YAML ConfigMap, дополнив настройки, чтобы забракировать метрики Prometheus.

    >[!NOTE]
    >Если вы редактируете файл YAML ConfigMap для Azure Red Hat OpenShift, сначала выполните команду, `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` чтобы открыть файл в текстовом редакторе.

    >[!NOTE]
    >Следующая аннотация `openshift.io/reconcile-protect: "true"` должна быть добавлена в метаданные *Container-АЗМ-MS-ажентконфиг* ConfigMap для предотвращения сверки. 
    >```
    >metadata:
    >   annotations:
    >       openshift.io/reconcile-protect: "true"
    >```

    - Чтобы собираются службы Kubernetes Services на уровне кластера, настройте файл ConfigMap, используя следующий пример.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        kubernetes_services = ["http://my-service-dns.my-namespace:9102/metrics"]
        ```

    - Чтобы настроить отходы метрик Prometheus по определенному URL-адресу в кластере, настройте файл ConfigMap, используя следующий пример.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        urls = ["http://myurl:9101/metrics"] ## An array of urls to scrape metrics from
        ```

    - Чтобы настроить отходы метрик Prometheus из набора управляющих программ агента для каждого отдельного узла в кластере, настройте следующие параметры в ConfigMap:
    
        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings 
        [prometheus_data_collection_settings.node] 
        interval = "1m"  ## Valid time units are s, m, h. 
        urls = ["http://$NODE_IP:9103/metrics"] 
        fieldpass = ["metric_to_pass1", "metric_to_pass2"] 
        fielddrop = ["metric_to_drop"] 
        ```

        >[!NOTE]
        >$NODE _IP — это конкретный параметр Container Insights. его можно использовать вместо IP-адреса узла. Оно должно быть в верхнем регистре. 

    - Чтобы настроить отходы метрик Prometheus, указав аннотацию Pod, выполните следующие действия.

       1. В ConfigMap укажите следующее:

            ```
            prometheus-data-collection-settings: |- 
            # Custom Prometheus metrics data collection settings
            [prometheus_data_collection_settings.cluster] 
            interval = "1m"  ## Valid time units are s, m, h
            monitor_kubernetes_pods = true 
            ```

       2. Укажите следующую конфигурацию для заметок Pod:

           ```
           - prometheus.io/scrape:"true" #Enable scraping for this pod 
           - prometheus.io/scheme:"http" #If the metrics endpoint is secured then you will need to set this to `https`, if not default ‘http’
           - prometheus.io/path:"/mymetrics" #If the metrics path is not /metrics, define it with this annotation. 
           - prometheus.io/port:"8000" #If port is not 9102 use this annotation
           ```
    
          Если вы хотите ограничить мониторинг определенными пространствами имен для модулей Pod с заметками, например, включайте только модули памяти, выделенные для рабочих нагрузок, установите для параметра значение `monitor_kubernetes_pod` `true` в ConfigMap и добавьте фильтр пространства имен, `monitor_kubernetes_pods_namespaces` указывающий пространства имен, из которых следует избавиться. Например: `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]`

3. Выполните следующую команду kubectl: `kubectl apply -f <configmap_yaml_file.yaml>` .
    
    Например, `kubectl apply -f container-azm-ms-agentconfig.yaml`. 

До вступления в силу изменение конфигурации может занять несколько минут, и все omsagent Pod в кластере будут перезапущены. Перезагрузка является пошаговым перезапуском для всех модулей omsagent Pod, а не всех перезапусков одновременно. После завершения перезагрузки отображается сообщение, похожее на следующее и содержащее результат: `configmap "container-azm-ms-agentconfig" created` .

## <a name="configure-and-deploy-configmaps---azure-red-hat-openshift-v3"></a>Настройка и развертывание Конфигмапс в Azure Red Hat OpenShift v3

В этом разделе содержатся требования и действия для успешной настройки файла конфигурации ConfigMap для кластера Azure Red Hat OpenShift v3. x.

>[!NOTE]
>Для Azure Red Hat OpenShift v3. x в пространстве имен *OpenShift-Azure-Logging* создается файл шаблона ConfigMap. Он не настроен на активное отслеживание метрик или сбор данных от агента.

### <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, подтвердите, что вы являетесь членом роли администратора кластера Azure Red Hat OpenShift, чтобы настроить контейнерные параметры агента и Prometheus. Чтобы убедиться, что вы являетесь членом группы *OSA-Customer-администраторы* , выполните следующую команду:

``` bash
  oc get groups
```

Результат должен выглядеть так:

``` bash
NAME                  USERS
osa-customer-admins   <your-user-account>@<your-tenant-name>.onmicrosoft.com
```

Если вы являетесь членом группы *"OSA — клиент-Администраторы"* , вы можете получить список `container-azm-ms-agentconfig` ConfigMap с помощью следующей команды:

``` bash
oc get configmaps container-azm-ms-agentconfig -n openshift-azure-logging
```

Результат должен выглядеть так:

``` bash
NAME                           DATA      AGE
container-azm-ms-agentconfig   4         56m
```

### <a name="enable-monitoring"></a>Включение мониторинга

Выполните следующие действия, чтобы настроить файл конфигурации ConfigMap для кластера Azure Red Hat OpenShift v3. x.

1. Измените файл YAML ConfigMap, дополнив настройки, чтобы забракировать метрики Prometheus. Шаблон ConfigMap уже существует в кластере Red Hat OpenShift v3. Выполните команду, `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` чтобы открыть файл в текстовом редакторе.

    >[!NOTE]
    >Следующая аннотация `openshift.io/reconcile-protect: "true"` должна быть добавлена в метаданные *Container-АЗМ-MS-ажентконфиг* ConfigMap для предотвращения сверки. 
    >```
    >metadata:
    >   annotations:
    >       openshift.io/reconcile-protect: "true"
    >```

    - Чтобы собираются службы Kubernetes Services на уровне кластера, настройте файл ConfigMap, используя следующий пример.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        kubernetes_services = ["http://my-service-dns.my-namespace:9102/metrics"]
        ```

    - Чтобы настроить отходы метрик Prometheus по определенному URL-адресу в кластере, настройте файл ConfigMap, используя следующий пример.

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        urls = ["http://myurl:9101/metrics"] ## An array of urls to scrape metrics from
        ```

    - Чтобы настроить отходы метрик Prometheus из набора управляющих программ агента для каждого отдельного узла в кластере, настройте следующие параметры в ConfigMap:
    
        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings 
        [prometheus_data_collection_settings.node] 
        interval = "1m"  ## Valid time units are s, m, h. 
        urls = ["http://$NODE_IP:9103/metrics"] 
        fieldpass = ["metric_to_pass1", "metric_to_pass2"] 
        fielddrop = ["metric_to_drop"] 
        ```

        >[!NOTE]
        >$NODE _IP — это конкретный параметр Container Insights. его можно использовать вместо IP-адреса узла. Оно должно быть в верхнем регистре. 

    - Чтобы настроить отходы метрик Prometheus, указав аннотацию Pod, выполните следующие действия.

       1. В ConfigMap укажите следующее:

            ```
            prometheus-data-collection-settings: |- 
            # Custom Prometheus metrics data collection settings
            [prometheus_data_collection_settings.cluster] 
            interval = "1m"  ## Valid time units are s, m, h
            monitor_kubernetes_pods = true 
            ```

       2. Укажите следующую конфигурацию для заметок Pod:

           ```
           - prometheus.io/scrape:"true" #Enable scraping for this pod 
           - prometheus.io/scheme:"http" #If the metrics endpoint is secured then you will need to set this to `https`, if not default ‘http’
           - prometheus.io/path:"/mymetrics" #If the metrics path is not /metrics, define it with this annotation. 
           - prometheus.io/port:"8000" #If port is not 9102 use this annotation
           ```
    
          Если вы хотите ограничить мониторинг определенными пространствами имен для модулей Pod с заметками, например, включайте только модули памяти, выделенные для рабочих нагрузок, установите для параметра значение `monitor_kubernetes_pod` `true` в ConfigMap и добавьте фильтр пространства имен, `monitor_kubernetes_pods_namespaces` указывающий пространства имен, из которых следует избавиться. Например: `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]`

2. Сохраните изменения в редакторе.

До вступления в силу изменение конфигурации может занять несколько минут, и все omsagent Pod в кластере будут перезапущены. Перезагрузка является пошаговым перезапуском для всех модулей omsagent Pod, а не всех перезапусков одновременно. После завершения перезагрузки отображается сообщение, похожее на следующее и содержащее результат: `configmap "container-azm-ms-agentconfig" created` .

Обновленный ConfigMap можно просмотреть, выполнив команду `oc describe configmaps container-azm-ms-agentconfig -n openshift-azure-logging` . 

## <a name="applying-updated-configmap"></a>Применение обновленных ConfigMap

Если вы уже развернули ConfigMap в кластер и хотите обновить его с помощью более новой конфигурации, вы можете изменить ранее использовавшийся файл ConfigMap, а затем применить те же команды, что и раньше.

Для следующих сред Kubernetes:

- Служба Azure Kubernetes (AKS)
- Azure Stack или локально
- Azure Red Hat OpenShift и Red Hat OpenShift версии 4. x

выполните команду `kubectl apply -f <configmap_yaml_file.yaml` . 

Для кластера Azure Red Hat OpenShift v3. x выполните команду, `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` чтобы открыть файл в редакторе по умолчанию, чтобы изменить и сохранить его.

До вступления в силу изменение конфигурации может занять несколько минут, и все omsagent Pod в кластере будут перезапущены. Перезагрузка является пошаговым перезапуском для всех модулей omsagent Pod, а не всех перезапусков одновременно. После завершения перезагрузки отображается сообщение, похожее на следующее и содержащее результат: `configmap "container-azm-ms-agentconfig" updated` .

## <a name="verify-configuration"></a>Проверка конфигурации

Чтобы убедиться, что конфигурация была успешно применена к кластеру, выполните следующую команду, чтобы проверить журналы из модуля Pod агента: `kubectl logs omsagent-fdf58 -n=kube-system` . 

>[!NOTE]
>Эта команда неприменима к кластеру Azure Red Hat OpenShift v3. x.
> 

При наличии ошибок конфигурации из модулей Pod omsagent в выходных данных отобразятся ошибки, аналогичные приведенным ниже.

``` 
***************Start Config Processing******************** 
config::unsupported/missing config schema version - 'v21' , using defaults
```

Ошибки, связанные с применением изменений конфигурации, также доступны для проверки. Следующие параметры позволяют выполнить дополнительные действия по устранению изменений конфигурации и браку в Prometheus метриках.

- Из журналов Pod агента с помощью той же `kubectl logs` команды 
    >[!NOTE]
    >Эта команда неприменима к кластеру Azure Red Hat OpenShift.
    > 

- Из динамических данных (Предварительная версия). В журналах интерактивных данных (Предварительная версия) отображаются ошибки, аналогичные следующим:

    ```
    2019-07-08T18:55:00Z E! [inputs.prometheus]: Error in plugin: error making HTTP request to http://invalidurl:1010/metrics: Get http://invalidurl:1010/metrics: dial tcp: lookup invalidurl on 10.0.0.10:53: no such host
    ```

- Из таблицы **кубемонажентевентс** в рабочей области log Analytics. Данные отправляются каждый час с серьезностью *предупреждения* для ошибок конфигурации и серьезности *ошибки* . Если ошибок нет, запись в таблице будет содержать данные со *сведениями об* уровне серьезности, которые не сообщают об ошибках. Свойство **Tags** содержит дополнительные сведения о Pod и идентификаторе контейнера, в которых произошла ошибка, а также первое вхождение, Последнее вхождение и количество за последний час.

- Для Azure Red Hat OpenShift v3. x и v4. x Проверьте журналы omsagent, выполнив поиск в таблице **контаинерлог** , чтобы проверить, включено ли ведение журнала OpenShift-Azure-Logging.

Ошибки не позволяют omsagent анализировать файл, что приводит к перезапуску и использованию конфигурации по умолчанию. После исправления ошибок в ConfigMap для кластеров, отличных от Azure Red Hat OpenShift v3. x, сохраните файл YAML и примените обновленную Конфигмапс, выполнив команду: `kubectl apply -f <configmap_yaml_file.yaml` . 

Для Azure Red Hat OpenShift v3. x измените и сохраните обновленный Конфигмапс, выполнив команду: `oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging` .

## <a name="query-prometheus-metrics-data"></a>Запрос данных метрик Prometheus

Чтобы просмотреть метрики Prometheus, которые выводятся Azure Monitor и ошибки конфигурации или брака/отхода, о которых сообщил агент, просмотрите [данные метрик запросов Prometheus](container-insights-log-search.md#query-prometheus-metrics-data) и конфигурацию [запроса или ошибки](container-insights-log-search.md#query-config-or-scraping-errors).

## <a name="view-prometheus-metrics-in-grafana"></a>Просмотр метрик Prometheus в Grafana

Аналитика контейнеров поддерживает просмотр метрик, хранящихся в рабочей области Log Analytics, на панелях мониторинга Grafana. Мы создали шаблон, который можно скачать из [репозитория Grafana для панелей мониторинга](https://grafana.com/grafana/dashboards?dataSource=grafana-azure-monitor-datasource&category=docker), чтобы начать работу или уточнить, как выполнять запросы дополнительных данных из включенных в мониторинг кластеров для визуализации на пользовательских панелях мониторинга Grafana. 

## <a name="review-prometheus-data-usage"></a>Проверка использования данных Prometheus

Чтобы определить, является ли объем приема для каждого метрики размером в ГБ в день, чтобы понять, насколько он высокий, предоставляется следующий запрос.

```
InsightsMetrics
| where Namespace contains "prometheus"
| where TimeGenerated > ago(24h)
| summarize VolumeInGB = (sum(_BilledSize) / (1024 * 1024 * 1024)) by Name
| order by VolumeInGB desc
| render barchart
```

В выходных данных отобразятся результаты, аналогичные приведенным ниже.

![Снимок экрана: результаты запроса журнала для тома приема данных](./media/container-insights-prometheus-integration/log-query-example-usage-03.png)

Чтобы оценить, какой размер метрик в Гб соответствует месяцу, чтобы понять, является ли объем данных, полученных в рабочей области, большим, предоставляется следующий запрос.

```
InsightsMetrics
| where Namespace contains "prometheus"
| where TimeGenerated > ago(24h)
| summarize EstimatedGBPer30dayMonth = (sum(_BilledSize) / (1024 * 1024 * 1024)) * 30 by Name
| order by EstimatedGBPer30dayMonth desc
| render barchart
```

В выходных данных отобразятся результаты, аналогичные приведенным ниже.

![Регистрация результатов запроса для тома приема данных](./media/container-insights-prometheus-integration/log-query-example-usage-02.png)

Дополнительные сведения о мониторинге использования данных и анализе затрат см. в руководствах по [управлению использованием и затратами Azure Monitor журналов](../logs/manage-cost-storage.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о настройке параметров коллекции агентов для stdout, stderr и переменных среды из рабочих нагрузок контейнера см. [здесь](container-insights-agent-config.md).