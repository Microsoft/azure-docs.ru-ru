---
title: Прекращение использования API Azure Monitor
description: Описание прекращения использования старых версий API поставщика ресурсов OperationalInsights.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/29/2020
ms.openlocfilehash: 7b5d105f6a93102ff1b9142fbdf690c3873d872f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102032030"
---
# <a name="operationalinsights-api-version-retirement"></a>Прекращение использования версии API OperationalInsights
Корпорация Майкрософт предоставляет уведомление не менее чем за 12 месяцев до прекращения использования API для сглаживания перехода до более новой или поддерживаемой версии. Мы выпустили новую версию (2020-08-01) для API поставщика ресурсов **OperationalInsights** и выпустим более ранние версии API-29 февраля 2024.

Мы рекомендуем начать использовать версию 2020-08-01, чтобы получить преимущества новых функций, таких как [выделенный кластер](./logs-dedicated-clusters.md), [управляемые клиентом ключи](../logs/customer-managed-keys.md), [Частная связь](./private-link-security.md) и [Экспорт данных](./logs-data-export.md). Кроме того, новые функции и функции и оптимизации добавляются только к текущему API.

После 29 февраля 2024 Azure Monitor больше не будет поддерживать более ранние версии API, чем 2020-08-01. Если вы предпочитаете не выполнять обновление, запросы, отправленные из более ранних версий, продолжат обслуживаться службой Azure Monitor до 29 февраля 2024.

## <a name="migration-steps"></a>Этапы миграции
В зависимости от используемого метода конфигурации следует обновить новую версию в запросах **RESTful** и **Диспетчер ресурсов шаблонах**. Чтобы обновить версию API, выполните приведенные ниже примеры.

1. Запросы REST API используют версию API в URL-адресе запроса. Замените эту версию последней версией (2020-08-01), как показано в следующем примере.

    ```rest
    https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}?api-version=2020-08-01
    ```

2. Azure Resource Manager шаблоны используют версию API в свойстве **apiVersion** ресурса. Замените эту версию последней версией (2020-08-01), как показано в следующем примере.


    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-08-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "workspaceName": {
                "type": "string",
                "metadata": {
                "description": "Name of the workspace."
                }
            },
            "resources": [
            {
                "type": "Microsoft.OperationalInsights/workspaces",
                "name": "[parameters('workspaceName')]",
                "apiVersion": "2020-08-01",
                "location": "westus",
                "properties": {
                    "sku": {
                        "name": "pergb2018"
                    },
                    "retentionInDays": 30,
                    "features": {
                        "searchVersion": 1,
                        "legacy": 0,
                        "enableLogAccessUsingOnlyResourcePermissions": true
                    }
                }
            }
        ]
    }
    }
    ```


### <a name="more-information"></a>Дополнительные сведения
Если у вас есть вопросы, получите ответы от [наших экспертов сообщества]( https://techcommunity.microsoft.com/t5/azure-monitor/bd-p/AzureMonitor). Если у вас есть план поддержки и вам нужна техническая помощь, создайте [запрос в службу поддержки]( https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest): 
1.  В разделе *тип проблемы* выберите пункт **Техническая**. 
2.  В разделе *Подписка* выберите свою подписку. 
3.  В разделе *Служба* выберите **Мои службы**, а затем выберите **log Analytics**. 
4.  В разделе *Сводка* введите описание проблемы. 
5.  В разделе *тип проблемы* выберите **log Analytics Управление рабочей областью**.  
6.  В разделе *проблемный подтип* выберите **шаблоны ARM, PowerShell и CLI**. 

## <a name="next-steps"></a>Дальнейшие действия

- См. [Справочник по API рабочей области OperationalInsights](/rest/api/loganalytics/workspaces).