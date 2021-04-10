---
title: Учебник. Мониторинг метрик на уровне приложения Apache Spark с помощью Prometheus и Grafana
description: Учебник. Узнайте, как развернуть решение для метрик приложений Apache Spark в кластере Службы Azure Kubernetes (AKS) и как интегрировать панели мониторинга Grafana.
services: synapse-analytics
author: hrasheed-msft
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 01/22/2021
ms.openlocfilehash: 6a0b63dc7fda25e3911ae713a0bea7ae7a0969f9
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104602304"
---
# <a name="tutorial-monitor-apache-spark-application-level-metrics-with-prometheus-and-grafana"></a>Учебник. Мониторинг метрик на уровне приложения Apache Spark с помощью Prometheus и Grafana

## <a name="overview"></a>Обзор

В этом учебнике описывается, как развернуть решение для метрик приложений Apache Spark в кластере Службы Azure Kubernetes (AKS) и интегрировать панели мониторинга Grafana.

Это решение можно использовать для сбора данных метрик Apache Spark и выполнения запросов к ним в реальном времени. Интегрированные панели мониторинга Grafana позволяют диагностировать и отслеживать приложение Apache Spark. Исходный код и конфигурации предоставляются на портале GitHub в виде открытого кода.

## <a name="prerequisites"></a>Предварительные требования

1.  [Azure CLI](/cli/azure/install-azure-cli)
2.  [Клиент Helm 3.30 или более поздней версии](https://github.com/helm/helm/releases)
3.  [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
4.  [Служба Azure Kubernetes (AKS)](https://azure.microsoft.com/en-us/services/kubernetes-service/)

Или используйте компонент [Azure Cloud Shell](https://shell.azure.com/), в котором уже есть Azure CLI, клиент Helm и kubectl.

## <a name="log-in-to-azure"></a>Вход в Azure

```bash
az login
az account set --subscription "<subscription_id>"
```

## <a name="create-an-azure-kubernetes-service-instance-aks"></a>Создание экземпляра Службы Azure Kubernetes (AKS)

Используйте команду Azure CLI, чтобы создать кластер Kubernetes в своей подписке.

```
az aks create --name <kubernetes_name> --resource-group <kubernetes_resource_group> --location <location> --node-vm-size Standard_D2s_v3
az aks get-credentials --name <kubernetes_name> --resource-group <kubernetes_resource_group>
```

Примечание. Этот шаг можно пропустить, если у вас уже есть кластер AKS.

## <a name="create-a-service-principal-and-grant-permission-to-synapse-workspace"></a>Создание субъекта-службы и предоставление разрешения для рабочей области Synapse

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

Запишите идентификатор appId, пароль и идентификатор tenantID.

[![Снимок экрана: предоставление разрешений для управления доступом на основе ролей](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-grant-permission-srbac-new.png)](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-grant-permission-srbac-new.png#lightbox)

1. Войдите в свою [рабочую область Azure Synapse Analytics](https://web.azuresynapse.net/) в качестве администратора Synapse.

2. В Synapse Studio в области навигации слева выберите **Управление > Управление доступом**.

3. Нажмите кнопку "Добавить" в левом верхнем углу, чтобы **добавить назначение роли**.

4. В качестве области выберите **Рабочая область**.

5. В качестве роли выберите **Оператор вычислительной среды Synapse**.

6. В поле "Выбор пользователя" введите **<имя_субъекта-службы>** и щелкните свой субъект-службу.

7. Нажмите кнопку **Применить** (подождите 3 минуты, чтобы разрешение вступило в силу).

## <a name="install-connector-prometheus-server-grafana-dashboard"></a>Установка соединителя, сервера Prometheus и панели мониторинга Grafana

1. Добавьте репозиторий synapse-charts в клиент Helm.

```bash
helm repo add synapse-charts https://github.com/microsoft/azure-synapse-spark-metrics/releases/download/helm-chart
```

2.  Установите компоненты посредством клиента Helm:

```bash
helm install spo synapse-charts/synapse-prometheus-operator --create-namespace --namespace spo \
    --set synapse.workspaces[0].workspace_name="<workspace_name>" \
    --set synapse.workspaces[0].tenant_id="<tenant_id>" \
    --set synapse.workspaces[0].service_principal_name="<service_principal_app_id>" \
    --set synapse.workspaces[0].service_principal_password="<service_principal_password>" \
    --set synapse.workspaces[0].subscription_id="<subscription_id>" \
    --set synapse.workspaces[0].resource_group="<workspace_resource_group_name>"
```

 - workspace_name: имя рабочей области Synapse;
 - subscription_id: идентификатор подписки рабочей области Synapse;
 - workspace_resource_group_name: имя группы ресурсов рабочей области Synapse;
 - tenant_id: идентификатор арендатора рабочей области Synapse;
 - service_principal_app_id: идентификатор appId субъекта-службы;
 - service_principal_password: созданный вами пароль субъекта-службы.

## <a name="log-in-to-grafana"></a>Вход в Grafana

Получите пароль и адрес Grafana по умолчанию. Вы можете изменить этот пароль в параметрах Grafana.

```bash
kubectl get secret --namespace spo spo-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
kubectl -n spo get svc spo-grafana
```

Получите IP-адрес службы, скопируйте и вставьте этот внешний IP-адрес в браузер, а затем войдите с именем admin и полученным паролем.

## <a name="use-grafana-dashboards"></a>Использование панелей мониторинга Grafana

Найдите панель мониторинга Synapse в левом верхнем углу страницы Grafana (на домашней странице выберите "Synapse Workspace" (Рабочая область Synapse) или "Synapse Application" (Приложение Synapse)). Попробуйте выполнить пример кода в Synapse Studio и подождите несколько секунд, чтобы получить метрики.

Кроме того, вы можете использовать панели мониторинга "Synapse Workspace / Workspace" (Рабочая область Synapse: рабочая область) или "Synapse Workspace / Spark pools" (Рабочая область Synapse: пулы Spark), чтобы получить общие сведения о своей рабочей области и пулах Apache Spark.

## <a name="uninstall"></a>Удаление

Удалите компоненты командой Helm, как показано ниже.

```bash
helm delete <release_name> -n <namespace>
```

Удалите кластер AKS.

```bash
az aks delete --name <kubernetes_cluster_name> --resource-group <kubernetes_cluster_rg>
```

## <a name="components-introduction"></a>Общие сведения о компонентах

Azure Synapse Analytics предоставляет [диаграмму Helm](https://github.com/microsoft/azure-synapse-spark-metrics/tree/main/helm) на основе Prometheus Operator и соединителя Synapse для Prometheus. На этой диаграмме Helm показаны сервер Prometheus, сервер Grafana и панели мониторинга Grafana для метрик на уровне приложения Apache Spark. Вы можете использовать [Prometheus](https://prometheus.io/), популярную систему мониторинга с открытым кодом, для сбора данных этих метрик практически в реальном времени, а для визуализации использовать [Grafana](https://github.com/grafana/grafana).

### <a name="synapse-prometheus-connector"></a>Соединитель Synapse для Prometheus

Соединитель Synapse для Prometheus помогает подключить пул Apache Spark для Azure Synapse к серверу Prometheus. Он реализует следующие функции:

1.  Аутентификация. Это аутентификация на основе AAD, которая позволяет автоматически обновлять токен AAD субъекта-службы для обнаружения приложений, приема данных метрик и других функций.
2.  Обнаружение приложений Apache Spark. Когда вы отправляете приложения в целевую рабочую область, соединитель Synapse для Prometheus может автоматически обнаружить эти приложения.
3.  Метаданные приложения Apache Spark. Выполняется сбор основных сведений о приложении и экспорт этих данных в Prometheus.

Соединитель Synapse для Prometheus выпускается в виде образа Docker, размещаемого в [Microsoft Container Registry](https://github.com/microsoft/containerregistry). Он предоставляется в виде открытого кода и находится в разделе [метрик приложений Spark для Azure Synapse](https://github.com/microsoft/azure-synapse-spark-metrics).

### <a name="prometheus-server"></a>Сервер Prometheus

Prometheus — это набор средств с открытым кодом для мониторинга и оповещения. Набор средств Prometheus был разработан компанией Cloud Native Computing Foundation (CNCF) и де-факто стал стандартом для облачного мониторинга. С помощью Prometheus можно осуществлять сбор, запрос и хранение больших объемов данных временных рядов. Кроме того, это решение легко интегрировать с Grafana. В данном решении мы развертываем компонент Prometheus на основе диаграммы Helm.

### <a name="grafana-and-dashboards"></a>Grafana и панели мониторинга

Grafana — это программное обеспечение с открытым кодом для визуализации и анализа. Оно позволяет запрашивать, визуализировать и исследовать метрики, а также получать оповещения о них. Azure Synapse Analytics предоставляет набор панелей мониторинга Grafana по умолчанию для визуализации метрик на уровне приложения Apache Spark.

Панель мониторинга "Synapse Workspace / Workspace" (Рабочая область Synapse: рабочая область) предоставляет представление на уровне рабочей области, в котором отображаются все пулы Apache Spark, количество приложений, ядер ЦП и т. д.

[![Снимок экрана: рабочая область панели мониторинга](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-dashboard-workspace.png)](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-dashboard-workspace.png#lightbox)

Панель мониторинга "Synapse Workspace / Spark pools" (Рабочая область Synapse: пулы Spark) содержит метрики приложений Apache Spark, выполняющихся в выбранном пуле Apache Spark, за определенный период времени.

[![Снимок экрана: панель мониторинга пула Spark](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-dashboard-sparkpool.png)](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-dashboard-sparkpool.png#lightbox)

Панель мониторинга "Synapse Workspace / Spark Application" (Рабочая область Synapse: приложение Spark) содержит выбранное приложение Apache Spark.

[![Снимок экрана: панель мониторинга приложения](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-dashboard-application.png)](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-dashboard-application.png#lightbox)

Указанные выше шаблоны панелей мониторинга доступны в разделе [метрик приложений Spark для Azure Synapse](https://github.com/microsoft/azure-synapse-spark-metrics/tree/main/helm/synapse-prometheus-operator/grafana_dashboards) в виде открытого кода.