---
title: Примеры CLI для Azure Monitor
description: Примеры команд интерфейса командной строки для функций монитора Azure Monitor. Azure Monitor — это служба Microsoft Azure, которая позволяет отправлять оповещения и осуществлять вызов URL-адресов на основе значений настроенных данных телеметрии, а также выполнять автоматическое масштабирование облачных служб, виртуальных машин и веб-приложений.
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 05/16/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8eaf8c2e140f0b323db0c20a2e9946884c51df04
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102039178"
---
# <a name="azure-monitor-cli-samples"></a>Примеры CLI для Azure Monitor
В этой статье приведены примеры команд интерфейса командной строки для работы с функциями Azure Monitor. Azure Monitor позволяет выполнять автомасштабирование облачных служб, виртуальных машин и веб-приложений, отправлять оповещения и осуществлять вызов URL-адресов на основе значений настроенных данных телеметрии.

## <a name="prerequisites"></a>Предварительные требования

Если вы еще не установили Azure CLI, выполните инструкции по [установке Azure CLI](/cli/azure/install-azure-cli). Для запуска CLI в качестве интерактивного интерфейса в браузере вы также можете использовать [Azure Cloud Shell](/azure/cloud-shell). Сведения обо всех доступных командах см. в [справке по интерфейсу командной строки Azure Monitor](/cli/azure/monitor). 

## <a name="log-in-to-azure"></a>Вход в Azure
Прежде всего войдите в учетную запись Azure.

```azurecli
az login
```

После выполнения этой команды необходимо будет войти, следуя инструкциям на экране. Все команды работают в контексте подписки по умолчанию.

Получение сведений о текущей подписке.

```azurecli
az account show
```

Изменение подписки для рабочего контекста.

```azurecli
az account set -s <Subscription ID or name>
```

Просмотр списка всех поддерживаемых команд Azure Monitor.

```azurecli
az monitor -h
```

## <a name="view-activity-log"></a>Просмотр журнала действий

Просмотр списка событий в журнале действий.

```azurecli
az monitor activity-log list
```

Просмотр всех доступных вариантов.

```azurecli
az monitor activity-log list -h
```

Получение списка журналов по группе ресурсов.

```azurecli
az monitor activity-log list --resource-group <group name>
```

Получение списка журналов по вызывающему объекту.

```azurecli
az monitor activity-log list --caller myname@company.com
```

Получение списка журналов с указанием вызывающего объекта, типа ресурса и диапазона дат.

```azurecli
az monitor activity-log list --resource-provider Microsoft.Web \
    --caller myname@company.com \
    --start-time 2016-03-08T00:00:00Z \
    --end-time 2016-03-16T00:00:00Z
```

## <a name="work-with-alerts"></a>Работа с оповещениями 
> [!NOTE]
> Сейчас в интерфейсе командной строки поддерживаются только оповещения (классические). 

### <a name="get-alert-classic-rules-in-a-resource-group"></a>Получение правил оповещений (классических) в группе ресурсов

```azurecli
az monitor activity-log alert list --resource-group <group name>
az monitor activity-log alert show --resource-group <group name> --name <alert name>
```

### <a name="create-a-metric-alert-classic-rule"></a>Создание метрики правил оповещения (классические)

```azurecli
az monitor alert create --name <alert name> --resource-group <group name> \
    --action email <email1 email2 ...> \
    --action webhook <URI> \
    --target <target object ID> \
    --condition "<METRIC> {>,>=,<,<=} <THRESHOLD> {avg,min,max,total,last} ##h##m##s"
```

### <a name="delete-an-alert-classic-rule"></a>Удаление правил оповещений (классических)

```azurecli
az monitor alert delete --name <alert name> --resource-group <group name>
```

## <a name="log-profiles"></a>Профили журнала

В этом разделе описана работа с профилями журнала.

### <a name="get-a-log-profile"></a>Получение профиля журнала

```azurecli
az monitor log-profiles list
az monitor log-profiles show --name <profile name>
```

### <a name="add-a-log-profile-with-retention"></a>Добавление профиля журнала с хранением

```azurecli
az monitor log-profiles create --name <profile name> --location <location of profile> \
    --locations <locations to monitor activity in: location1 location2 ...> \
    --categories <categoryName1 categoryName2 ...> \
    --days <# days to retain> \
    --enabled true \
    --storage-account-id <storage account ID to store the logs in>
```

### <a name="add-a-log-profile-with-retention-and-eventhub"></a>Добавление профиля журнала с хранением и концентратором событий

```azurecli
az monitor log-profiles create --name <profile name> --location <location of profile> \
    --locations <locations to monitor activity in: location1 location2 ...> \
    --categories <categoryName1 categoryName2 ...> \
    --days <# days to retain> \
    --enabled true
    --storage-account-id <storage account ID to store the logs in>
    --service-bus-rule-id <service bus rule ID to stream to>
```

### <a name="remove-a-log-profile"></a>Удаление профиля журнала

```azurecli
az monitor log-profiles delete --name <profile name>
```

## <a name="diagnostics"></a>Диагностика

В этом разделе описана работа с параметрами диагностики.

### <a name="get-a-diagnostic-setting"></a>Получение параметра диагностики

```azurecli
az monitor diagnostic-settings list --resource <target resource ID>
```

### <a name="create-a-diagnostic-setting"></a>Создание параметра диагностики 

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <storage account ID> \
    --resource <target resource object ID> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

### <a name="delete-a-diagnostic-setting"></a>Удаление параметра диагностики

```azurecli
az monitor diagnostic-settings delete --name <diagnostic name> \
    --resource <target resource ID>
```

## <a name="autoscale"></a>Автомасштабирование

В этом разделе описана работа с параметрами автомасштабирования. Эти примеры необходимо изменить.

### <a name="get-autoscale-settings-for-a-resource-group"></a>Получение параметров автомасштабирования для группы ресурсов

```azurecli
az monitor autoscale list --resource-group <group name>
```

### <a name="get-autoscale-settings-by-name-in-a-resource-group"></a>Получение параметров автомасштабирования по имени в группе ресурсов

```azurecli
az monitor autoscale show --name <settings name> --resource-group <group name>
```

### <a name="set-autoscale-settings"></a>Настройка параметров автоматического масштабирования

```azurecli
az monitor autoscale create --name <settings name> --resource-group <group name> \
    --count <# instances> \
    --resource <target resource ID>
```
