---
title: Собирайте & анализа журналов ресурсов
description: Узнайте, как отправить журналы ресурсов и данные событий из групп контейнеров в службе "экземпляры контейнеров Azure" в журналы Azure Monitor
ms.topic: article
ms.date: 07/13/2020
ms.openlocfilehash: cfdcd1cc8e36a118c4e3c4435eaa002e4d3b1b93
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100579321"
---
# <a name="container-group-and-instance-logging-with-azure-monitor-logs"></a>Ведение журнала группы контейнеров и экземпляра с Azure Monitor журналами

Log Analytics рабочие области предоставляют централизованное расположение для хранения и запроса данных журнала не только из ресурсов Azure, но также и локальных ресурсов и ресурсов в других облаках. Экземпляры контейнеров Azure по умолчанию поддерживают отправку журналов и данных событий в журналы Azure Monitor.

Чтобы отправить журнал и данные событий группы контейнеров в Azure Monitor журналы, укажите существующий идентификатор рабочей области Log Analytics и ключ рабочей области при настройке группы контейнеров. 

В следующих разделах описано, как создать группу контейнеров с поддержкой ведения журналов и как запрашивать журналы. Вы также можете [Обновить группу контейнеров](container-instances-update.md) , указав идентификатор и ключ рабочей области, чтобы включить ведение журнала.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

> [!NOTE]
> Сейчас данные событий можно отправлять только из экземпляров контейнера Linux в Log Analytics.

## <a name="prerequisites"></a>Предварительные условия

Чтобы включить ведение журнала в экземпляре контейнера, необходимо следующее:

* [Рабочая область Log Analytics](../azure-monitor/logs/quick-create-workspace.md)
* [Azure CLI](/cli/azure/install-azure-cli) (или [Cloud Shell](../cloud-shell/overview.md))

## <a name="get-log-analytics-credentials"></a>Получение учетных данных Log Analytics

Экземплярам контейнеров Azure нужны разрешения на отправку данных в рабочую область Log Analytics. Чтобы предоставить такие разрешения и включить ведение журнала, необходимо указать идентификатор рабочей области Log Analytics и один из ключей для нее (первичный или вторичный) при создании группы контейнеров.

Чтобы получить идентификатор и первичный ключ рабочей области Log Analytics, сделайте следующее:

1. На портале Azure перейдите к рабочей области Log Analytics.
1. В разделе **Параметры** выберите **Управление агентами** .
1. Запишите следующие значения:
   * **Идентификатор рабочей области**
   * **Первичный ключ**

## <a name="create-container-group"></a>Создание группы контейнеров

Теперь у вас есть идентификатор и первичный ключ рабочей области Log Analytics, а значит вы можете создать группу контейнеров с поддержкой ведения журнала.

В следующих примерах демонстрируются два способа создания группы контейнеров, состоящей из одного [свободно][fluentd] созданного контейнера: Azure CLI и Azure CLI с помощью шаблона YAML. Контейнер fluentd создает несколько строк выходных данных в конфигурации по умолчанию. Так как эти выходные данные направляются в рабочую область Log Analytics, такой контейнер отлично подходит для демонстрации процессов просмотра журналов и обращения к ним.

### <a name="deploy-with-azure-cli"></a>Развертывание с помощью интерфейса командной строки Azure

Для развертывания с помощью Azure CLI следует указать параметры `--log-analytics-workspace` и `--log-analytics-workspace-key` при вызове команды [az container create][az-container-create]. Замените два значения рабочей области значениями, полученными на предыдущем шаге (и обновите имя группы ресурсов), прежде чем выполнять следующую команду.

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainergroup001 \
    --image fluent/fluentd \
    --log-analytics-workspace <WORKSPACE_ID> \
    --log-analytics-workspace-key <WORKSPACE_KEY>
```

### <a name="deploy-with-yaml"></a>Развертывание с помощью YAML

Используйте этот метод, чтобы развернуть группы контейнеров с YAML. Этот пример YAML определяет группу контейнеров с одним контейнером. Скопируйте код YAML в новый файл, а затем замените `LOG_ANALYTICS_WORKSPACE_ID` и `LOG_ANALYTICS_WORKSPACE_KEY` значениями, полученными на предыдущем шаге. Сохраните файл как **deploy-aci.yaml**.

```yaml
apiVersion: 2019-12-01
location: eastus
name: mycontainergroup001
properties:
  containers:
  - name: mycontainer001
    properties:
      environmentVariables: []
      image: fluent/fluentd
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
  diagnostics:
    logAnalytics:
      workspaceId: LOG_ANALYTICS_WORKSPACE_ID
      workspaceKey: LOG_ANALYTICS_WORKSPACE_KEY
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Затем выполните следующую команду, чтобы развернуть группу контейнеров. Замените в ней `myResourceGroup` именем группы ресурсов в вашей подписке (или создайте группу ресурсов с именем myResourceGroup перед выполнением команды).

```azurecli-interactive
az container create --resource-group myResourceGroup --name mycontainergroup001 --file deploy-aci.yaml
```

Вскоре после выполнения команды вы получите от Azure ответ с информацией о развертывании.

## <a name="view-logs"></a>Просмотр журналов

После развертывания группы контейнеров может потребоваться до 10 минут, чтобы на портале Azure появились первые записи журнала. 

Чтобы просмотреть журналы группы контейнеров в таблице `ContainerInstanceLog_CL`, выполните следующие действия:

1. На портале Azure перейдите к рабочей области Log Analytics.
1. В разделе **Общие** выберите **журналы** .  
1. Введите следующий запрос: `ContainerInstanceLog_CL | limit 50`.
1. Выбор **запуска**

Вы увидите несколько результатов, отображаемых запросом. Если на первый раз вы не видите никаких результатов, подождите несколько минут, а затем нажмите кнопку **выполнить** , чтобы снова выполнить запрос. По умолчанию записи журнала отображаются в формате **Таблица**. Вы также можете развернуть любую строку, чтобы просмотреть содержимое отдельной записи журнала.

![Параметры поиска по журналам на портале Azure][log-search-01]

## <a name="view-events"></a>Просмотр событий

Вы также можете просмотреть события для экземпляров контейнеров на портале Azure. События содержат время создания экземпляра и время его запуска. Чтобы просмотреть данные события в таблице `ContainerEvent_CL`, выполните следующие действия:

1. На портале Azure перейдите к рабочей области Log Analytics.
1. В разделе **Общие** выберите **журналы** .  
1. Введите следующий запрос: `ContainerEvent_CL | limit 50`.
1. Выбор **запуска**

Вы увидите несколько результатов, отображаемых запросом. Если на первый раз вы не видите никаких результатов, подождите несколько минут, а затем нажмите кнопку **выполнить** , чтобы снова выполнить запрос. По умолчанию записи отображаются в формате **таблицы**. Вы также можете развернуть любую строку, чтобы просмотреть содержимое отдельной записи.

![Результаты поиска событий на портале Azure][log-search-02]

## <a name="query-container-logs"></a>Запрос по журналам контейнера

Журналы Azure Monitor поддерживают расширенный [язык запросов][query_lang] для извлечения информации из выходных данных журналов, размер которых может достигать тысяч строк.

Базовая структура запроса включает имя таблицы с исходными данными (в этой статье `ContainerInstanceLog_CL` или `ContainerEvent_CL`) и ряд операторов, разделенных символом вертикальной черты (`|`). Вы можете объединить несколько операторов в цепочку, чтобы получить более точные результаты или применить более сложные функции.

Чтобы просмотреть пример результатов запроса, вставьте следующий запрос в текстовое поле запрос и нажмите кнопку **выполнить** , чтобы выполнить запрос. Этот запрос отображает все записи журнала, у которых поле Message (Сообщение) которого содержит слово warn (Предупреждение):

```query
ContainerInstanceLog_CL
| where Message contains "warn"
```

Также поддерживаются более сложные запросы. Например, такой запрос отображает записи из журнала группы контейнеров mycontainergroup001, которые были созданы за последний час:

```query
ContainerInstanceLog_CL
| where (ContainerGroup_s == "mycontainergroup001")
| where (TimeGenerated > ago(1h))
```

## <a name="next-steps"></a>Дальнейшие действия

### <a name="azure-monitor-logs"></a>Журналы Azure Monitor

Дополнительные сведения о запросах по журналам и настройке предупреждений в журналах Azure Monitor см. в следующих статьях:

* [Основные сведения о поиске по журналам в Azure Monitor журналах](../azure-monitor/logs/log-query-overview.md)
* [Новые возможности оповещений в Azure Monitor](../azure-monitor/alerts/alerts-overview.md)


### <a name="monitor-container-cpu-and-memory"></a>Мониторинг использования ЦП и памяти

Дополнительные сведения о мониторинге ресурсов ЦП и памяти для экземпляра контейнера см. в следующей статье:

* [Мониторинг ресурсов контейнеров в службе "Экземпляры контейнеров Azure"](container-instances-monitor.md).

<!-- IMAGES -->
[log-search-01]: ./media/container-instances-log-analytics/portal-query-01.png
[log-search-02]: ./media/container-instances-log-analytics/portal-query-02.png

<!-- LINKS - External -->
[fluentd]: https://hub.docker.com/r/fluent/fluentd/
[query_lang]: /azure/data-explorer/

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create