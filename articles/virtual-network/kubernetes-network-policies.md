---
title: Политики сети Azure Kubernetes | Документация Майкрософт
description: Узнайте о сетевых политиках Kubernetes для защиты кластера Kubernetes.
services: virtual-network
documentationcenter: na
author: aanandr
manager: NarayanAnnamalai
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/25/2018
ms.author: aanandr
ms.custom: ''
ms.openlocfilehash: a68e1a3f60930e290e97084ff2ec9350b18e2873
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100594971"
---
# <a name="azure-kubernetes-network-policies-overview"></a>Общие сведения о сетевых политиках Azure Kubernetes

Политики сети предоставляют микросегментацию для модулей Pod так же, как и группы безопасности сети (группы безопасности сети), обеспечивающие микросегментацию для виртуальных машин. Реализация диспетчера политики сети Azure (также известная как Azure NPM) поддерживает стандартную спецификацию политики сети Kubernetes. Метки можно использовать для выбора группы модулей Pod и определения списка входящих и исходящих правил для фильтрации трафика из этих модулей. Дополнительные сведения о политиках сети Kubernetes см. в [документации по Kubernetes](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

![Обзор политик сети Kubernetes](./media/kubernetes-network-policies/kubernetes-network-policies-overview.png)

Реализация Azure NPM работает совместно с Azure CNI, которая обеспечивает интеграцию виртуальной сети для контейнеров. NPM поддерживается только в Linux уже сегодня. Реализация обеспечивает фильтрацию трафика, настраивая правила IP-адресов allow и Deny в Linux IPTables на основе определенных политик. Эти правила группируются совместно с помощью Linux Ипсетс.

## <a name="planning-security-for-your-kubernetes-cluster"></a>Планирование безопасности для кластера Kubernetes
При реализации безопасности для кластера используйте группы безопасности сети (группы безопасности сети) для фильтрации трафика, поступающего в подсеть кластера (север-Южный трафик). Используйте Azure NPM для трафика между модулями Pod в кластере (трафик "Восток-Западная").

## <a name="using-azure-npm"></a>Использование Azure NPM
Azure NPM можно использовать в следующих способах для обеспечения микросегментации для модулей Pod.

### <a name="azure-kubernetes-service-aks"></a>Служба Azure Kubernetes (AKS)
NPM доступен в AKS и может быть включен во время создания кластера. Узнайте больше об этом в [безопасном трафике между модулями Pod с помощью сетевых политик в службе Azure Kubernetes Service (AKS)](../aks/use-network-policies.md).

### <a name="aks-engine"></a>AKS-Engine
является средством, которое создает шаблон Azure Resource Manager для развертывания кластера Kubernetes в Azure. Конфигурация кластера указана в файле JSON, который передается в средство при создании шаблона. Полный список поддерживаемых параметров кластера и их описание см. в разделе "Обработчик службы контейнеров Azure Microsoft — определения кластера".

Чтобы включить политики в кластерах, развернутых с помощью обработчика ACS, укажите для параметра networkPolicy в файле определения кластера значение "azure".

#### <a name="example-configuration"></a>Пример конфигурации

Ниже приведен пример конфигурации JSON, где создаются новая виртуальная сеть и подсеть, в которой с помощью Azure CNI развертывается кластер Kubernetes. Рекомендуется использовать Блокнот для изменения файла JSON. 
```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "kubernetesConfig": {
         "networkPolicy": "azure"
       }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "<specify a cluster name>",
      "vmSize": "Standard_D2s_v3"
    },
    "agentPoolProfiles": [
      {
        "name": "agentpool",
        "count": 2,
        "vmSize": "Standard_D2s_v3",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
   "linuxProfile": {
      "adminUsername": "<specify admin username>",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "<cut and paste your ssh key here>"
          }
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "<enter the client ID of your service principal here >",
      "secret": "<enter the password of your service principal here>"
    }
  }
}

```
### <a name="do-it-yourself-diy-kubernetes-clusters-in-azure"></a>Самостоятельное (DIY) кластеры Kubernetes в Azure
 Для кластеров DIY сначала установите подключаемый модуль CNI и включите его на каждой виртуальной машине в кластере. Подробные инструкции см. в разделе [Развертывание подключаемого модуля для кластера Kubernetes](deploy-container-networking.md#deploy-plug-in-for-a-kubernetes-cluster).

После развертывания кластера выполните следующую `kubectl` команду, чтобы скачать и применить *набор управляющих* программ Azure NPM к кластеру.

  ```
  kubectl apply -f https://raw.githubusercontent.com/Azure/acs-engine/master/parts/k8s/addons/kubernetesmasteraddons-azure-npm-daemonset.yaml

  ```
Это решение с открытым исходным кодом, и код доступен в [репозитории работы с контейнером Azure в сети](https://github.com/Azure/azure-container-networking/tree/master/npm).

## <a name="monitor-and-visualize-network-configurations-with-azure-npm"></a>Мониторинг и визуализация конфигураций сети с помощью Azure NPM
Azure NPM включает информативные метрики Prometheus, которые позволяют отслеживать и лучше понимать конфигурации. Она предоставляет встроенные визуализации в портал Azure или в Grafana Labs. Вы можете начать сбор этих метрик с помощью Azure Monitor или сервера Prometheus.

### <a name="benefits-of-azure-npm-metrics"></a>Преимущества метрик Azure NPM
Ранее пользователи могли только узнать о своей конфигурации сети с помощью команды `iptables -L` , выполняемой внутри узла кластера, что приводит к подробному и сложному понимании выходных данных. Метрики NPM обеспечивают следующие преимущества, связанные с политиками сети, правилами IPTables и Ипсетс.
- Предоставляет аналитические сведения о связи между тремя и измерениями времени для отладки конфигурации.
- Количество записей во всех Ипсетс и каждой Ипсет.
- Время, затраченное на применение политики с детализацией уровня Иптабле или Ипсет.
 
### <a name="supported-metrics"></a>Поддерживаемые метрики
Ниже приведен список поддерживаемых метрик.

|Имя метрики |Описание  |Тип метрики Prometheus  |Метки  |
|---------|---------|---------|---------|
|`npm_num_policies`     |число сетевых политик          |Датчик         |-         |
|`npm_num_iptables_rules`     | число правил IPTables     | Датчик        |-         |         
|`npm_num_ipsets`     |число Ипсетс         |Датчик            |-         |
|`npm_num_ipset_entries`     |число записей IP-адресов во всех Ипсетс         |Датчик         |-         |
|`npm_add_policy_exec_time`     |среда выполнения для добавления сетевой политики         |Итоги         |квантилей (0,5, 0,9 или 0,99)         |
|`npm_add_iptables_rule_exec_time`     |среда выполнения для добавления правила IPTables         |Итоги         |квантилей (0,5, 0,9 или 0,99)         |
|`npm_add_ipset_exec_time`     |среда выполнения для добавления Ипсет         |Итоги         |квантилей (0,5, 0,9 или 0,99)         |
|`npm_ipset_counts` продвинут     |число записей в каждом отдельном Ипсет         |гаужевек         |задать имя & хэш         |

Различные уровни квантилей в метриках "exec_time" помогают различать общие и наихудшие ситуации.

Есть также Метрика "exec_time_count" и "exec_time_sum" для каждой сводной метрики "exec_time".

Метрики могут быть списаны с помощью Azure Monitor для контейнеров или через Prometheus.

### <a name="setup-for-azure-monitor"></a>Настройка для Azure Monitor
Первым шагом является включение Azure Monitor для контейнеров в кластере Kubernetes. Действия можно найти в разделе [Общие сведения о Azure Monitor для контейнеров](../azure-monitor/containers/container-insights-overview.md). После включения Azure Monitor для контейнеров настройте [Azure Monitor для контейнеров ConfigMap](https://aka.ms/container-azm-ms-agentconfig) , чтобы включить интеграцию NPM и сбор МЕТРИК Prometheus NPM. В Azure Monitor для контейнеров ConfigMap есть ```integrations``` раздел с параметрами для сбора МЕТРИК NPM. Эти параметры по умолчанию отключены в ConfigMap. При включении базового параметра ```collect_basic_metrics = true``` будут собраны базовые метрики NPM. При включении дополнительных параметров ```collect_advanced_metrics = true``` будут собираются дополнительные метрики в дополнение к основным метрикам. 

После изменения ConfigMap сохраните его локально и примените ConfigMap к кластеру следующим образом.

```kubectl apply -f container-azm-ms-agentconfig.yaml``` Ниже приведен фрагмент кода из [Azure Monitor для контейнеров ConfigMap](https://aka.ms/container-azm-ms-agentconfig), в котором показана интеграция NPM с поддержкой расширенной коллекции метрик.
```
integrations: |-
    [integrations.azure_network_policy_manager]
        collect_basic_metrics = false
        collect_advanced_metrics = true
```
Дополнительные метрики необязательны, и их включение автоматически приводит к автоматическому включению сбора основных метрик. В настоящее время Расширенная метрика включает только `npm_ipset_counts`

Дополнительные сведения о [параметрах коллекции "Azure Monitor для контейнеров" в карте конфигурации](../azure-monitor/containers/container-insights-agent-config.md)

### <a name="visualization-options-for-azure-monitor"></a>Параметры визуализации для Azure Monitor
После включения сбора метрик NPM можно просмотреть метрики в портал Azure с помощью Container Insights или в Grafana.

#### <a name="viewing-in-azure-portal-under-insights-for-the-cluster"></a>Просмотр в портал Azure в разделе "аналитика" для кластера
Откройте портал Azure. В аналитике кластера перейдите в раздел "книги" и откройте "Конфигурация диспетчера сетевых политик (NPM)".

Помимо просмотра книги (рисунков ниже), можно также напрямую запрашивать метрики Prometheus в разделе "журналы" раздела Insights. Например, этот запрос возвратит все собираемые метрики.
| где TimeGenerated > назад (5H) | где name содержит "npm_"

Можно также запрашивать Log Analytics непосредственно для метрик. Дополнительные сведения о [Начало работы с помощью запросов log Analytics](../azure-monitor/containers/container-insights-log-search.md) 

#### <a name="viewing-in-grafana-dashboard"></a>Просмотр на панели мониторинга Grafana
Настройте сервер Grafana и настройте источник данных Log Analytics, как описано [здесь](https://grafana.com/grafana/plugins/grafana-azure-monitor-datasource). Затем импортируйте [панель мониторинга Grafana с log Analyticsной серверной частью](https://grafana.com/grafana/dashboards/10956) в Grafana Labs.

Панель мониторинга содержит визуальные элементы, аналогичные книге Azure. На диаграмму можно добавлять панели & визуализировать метрики NPM из таблицы Инсигхтсметрикс.

### <a name="setup-for-prometheus-server"></a>Установка сервера Prometheus
Некоторые пользователи могут использовать для сбора метрики с сервером Prometheus, а не Azure Monitor для контейнеров. Вам просто нужно добавить два задания в конфигурацию отходов для сбора метрик NPM.

Чтобы установить простой сервер Prometheus, добавьте этот репозиторий Helm в кластер.
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm repo update
```
Затем добавьте сервер.
```
helm install prometheus stable/prometheus -n monitoring \
--set pushgateway.enabled=false,alertmanager.enabled=false, \
--set-file extraScrapeConfigs=prometheus-server-scrape-config.yaml
```
где `prometheus-server-scrape-config.yaml` состоит из
```
- job_name: "azure-npm-node-metrics"
  metrics_path: /node-metrics
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - source_labels: [__address__]
    action: replace
    regex: ([^:]+)(?::\d+)?
    replacement: "$1:10091"
    target_label: __address__
- job_name: "azure-npm-cluster-metrics"
  metrics_path: /cluster-metrics
  kubernetes_sd_configs:
  - role: service
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    regex: kube-system
    action: keep
  - source_labels: [__meta_kubernetes_service_name]
    regex: npm-metrics-cluster-service
    action: keep
# Comment from here to the end to collect advanced metrics: number of entries for each IPSet
  metric_relabel_configs:
  - source_labels: [__name__]
    regex: npm_ipset_counts
    action: drop
```


Вы также можете заменить `azure-npm-node-metrics` Задание содержимым ниже или включить его в уже существующее задание для Kubernetes Pod:
```
- job_name: "azure-npm-node-metrics-from-pod-config"
  metrics_path: /node-metrics
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    regex: kube-system
    action: keep
  - source_labels: [__meta_kubernetes_pod_annotationpresent_azure_npm_scrapeable]
    action: keep
  - source_labels: [__address__]
    action: replace
    regex: ([^:]+)(?::\d+)?
    replacement: "$1:10091"
    target_label: __address__
```

### <a name="visualization-options-for-prometheus"></a>Параметры визуализации для Prometheus
При использовании Prometheus Server поддерживается только панель мониторинга Grafana. 

Если вы еще не сделали этого, настройте сервер Grafana и настройте источник данных Prometheus. Затем импортируйте нашу [панель мониторинга Grafana с серверной частью Prometheus](https://grafana.com/grafana/dashboards/13000) в свои Grafana Labs.

Визуальные элементы для этой панели мониторинга идентичны панели мониторинга с внутренним контейнером "аналитика/Log Analytics".

### <a name="sample-dashboards"></a>Примеры информационных панелей
Ниже приведен пример панели мониторинга для метрик NPM в контейнере "аналитика" (CI) и Grafana

#### <a name="ci-summary-counts"></a>Сводные счетчики CI
![Сводный счетчик книг Azure](media/kubernetes-network-policies/workbook-summary-counts.png)

#### <a name="ci-counts-over-time"></a>Счетчики CI с течением времени
[![Количество книг Azure с течением времени](media/kubernetes-network-policies/workbook-counts-over-time.png)](media/kubernetes-network-policies/workbook-counts-over-time.png#lightbox)

#### <a name="ci-ipset-entries"></a>Элементы CI Ипсет
[![Записи Ипсет в книге Azure](media/kubernetes-network-policies/workbook-ipset-entries.png)](media/kubernetes-network-policies/workbook-ipset-entries.png#lightbox)

#### <a name="ci-runtime-quantiles"></a>Квантилей среды выполнения CI
![Квантилей среды выполнения книги Azure](media/kubernetes-network-policies/workbook-runtime-quantiles.png)

#### <a name="grafana-dashboard-summary-counts"></a>Счетчики сводки панели мониторинга Grafana
![Счетчики сводки панели мониторинга Grafana](media/kubernetes-network-policies/grafana-summary-counts.png)

#### <a name="grafana-dashboard-counts-over-time"></a>Счетчики панели мониторинга Grafana с течением времени
[![Счетчики панели мониторинга Grafana с течением времени](media/kubernetes-network-policies/grafana-counts-over-time.png)](media/kubernetes-network-policies/grafana-counts-over-time.png#lightbox)

#### <a name="grafana-dashboard-ipset-entries"></a>Записи Ипсет панели мониторинга Grafana
[![Записи Ипсет панели мониторинга Grafana](media/kubernetes-network-policies/grafana-ipset-entries.png)](media/kubernetes-network-policies/grafana-ipset-entries.png#lightbox)

#### <a name="grafana-dashboard-runtime-quantiles"></a>Квантилей среды выполнения Grafana Dashboard
[![Квантилей среды выполнения Grafana Dashboard](media/kubernetes-network-policies/grafana-runtime-quantiles.png)](media/kubernetes-network-policies/grafana-runtime-quantiles.png#lightbox)



## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [Службе Azure Kubernetes](../aks/intro-kubernetes.md).
-  Дополнительные сведения о [работе с контейнерами в сети](container-networking-overview.md).
- [Разверните подключаемый модуль](deploy-container-networking.md) для кластеров Kubernetes или контейнеров DOCKER.

