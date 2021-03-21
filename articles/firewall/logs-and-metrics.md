---
title: Общие сведения о журналах и метриках брандмауэра Azure
description: Работу брандмауэра Azure можно отслеживать с помощью журналов брандмауэра. Также журналы действий можно использовать для аудита операций на ресурсах брандмауэра Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 02/16/2021
ms.author: victorh
ms.openlocfilehash: 8922e6e0d5137a3a900e0f57f685d449c08b3f47
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100596960"
---
# <a name="azure-firewall-logs-and-metrics"></a>Журналы и метрики Брандмауэра Azure

Работу брандмауэра Azure можно отслеживать с помощью журналов брандмауэра. Также журналы действий можно использовать для аудита операций на ресурсах брандмауэра Azure.

Доступ к некоторым из этих журналов можно получить через портал. Журналы можно отправлять в [Azure Monitor журналы](../azure-monitor/insights/azure-networking-analytics.md), хранилище и концентраторы событий, а также анализировать в журналах Azure Monitor или с помощью различных средств, таких как Excel и Power BI.

Метрики являются упрощенными и могут поддерживать сценарии практически в реальном времени, что позволяет использовать их для оповещений и быстрого обнаружения проблем.

## <a name="diagnostic-logs"></a>Журналы диагностики

 Для брандмауэра Azure доступны следующие журналы диагностики.

* **Журнал правил приложений**

   Журнал правил приложений сохраняется в учетной записи хранения, передается в концентраторы событий и (или) отправляются в журналы Azure Monitor, только если вы включили его для каждого брандмауэра Azure. Результаты каждого нового подключения, которое соответствует одному из настроенных правил приложения, находятся в журнале принятого или отклоненного подключения. Данные записываются в формате JSON, как показано в следующих примерах:

   ```
   Category: application rule logs.
   Time: log timestamp.
   Properties: currently contains the full message.
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   ```json
   {
    "category": "AzureFirewallApplicationRule",
    "time": "2018-04-16T23:45:04.8295030Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallApplicationRuleLog",
    "properties": {
        "msg": "HTTPS request from 10.1.0.5:55640 to mydestination.com:443. Action: Allow. Rule Collection: collection1000. Rule: rule1002"
    }
   }
   ```

   ```json
   {
     "category": "AzureFirewallApplicationRule",
     "time": "2018-04-16T23:45:04.8295030Z",
     "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
     "operationName": "AzureFirewallApplicationRuleLog",
     "properties": {
         "msg": "HTTPS request from 10.11.2.4:53344 to www.bing.com:443. Action: Allow. Rule Collection: ExampleRuleCollection. Rule: ExampleRule. Web Category: SearchEnginesAndPortals"
     }
   }
   ```

* **Журнал правил сети**

   Журнал сетевых правил сохраняется в учетной записи хранения, передается в концентраторы событий и (или) отправляются в журналы Azure Monitor, только если вы включили его для каждого брандмауэра Azure. Каждое новое подключение, которое соответствует одному из настроенных правил сети, регистрируется в журнале как принятое или отклоненное подключение. Данные регистрируются в журнале в формате JSON, как показано в примере ниже.

   ```
   Category: network rule logs.
   Time: log timestamp.
   Properties: currently contains the full message.
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   ```json
  {
    "category": "AzureFirewallNetworkRule",
    "time": "2018-06-14T23:44:11.0590400Z",
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
    "operationName": "AzureFirewallNetworkRuleLog",
    "properties": {
        "msg": "TCP request from 111.35.136.173:12518 to 13.78.143.217:2323. Action: Deny"
    }
   }

   ```

* **Журнал DNS-прокси**

   Журнал прокси-сервера DNS сохраняется в учетной записи хранения, передается в концентраторы событий и/или отправляются в журналы Azure Monitor только в том случае, если вы включили его для каждого брандмауэра Azure. Этот журнал отслеживает сообщения DNS на DNS-сервере, настроенном с помощью DNS-прокси. Данные записываются в формате JSON, как показано в следующих примерах:


   ```
   Category: DNS proxy logs.
   Time: log timestamp.
   Properties: currently contains the full message.
   note: this field will be parsed to specific fields in the future, while maintaining backward compatibility with the existing properties field.
   ```

   Выполнение:
   ```json
   {
     "category": "AzureFirewallDnsProxy",
     "time": "2020-09-02T19:12:33.751Z",
     "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
     "operationName": "AzureFirewallDnsProxyLog",
     "properties": {
         "msg": "DNS Request: 11.5.0.7:48197 – 15676 AAA IN md-l1l1pg5lcmkq.blob.core.windows.net. udp 55 false 512 NOERROR - 0 2.000301956s"
     }
   }
   ```

   Сбой:

   ```json
   {
     "category": "AzureFirewallDnsProxy",
     "time": "2020-09-02T19:12:33.751Z",
     "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/AZUREFIREWALLS/{resourceName}",
     "operationName": "AzureFirewallDnsProxyLog",
     "properties": {
         "msg": " Error: 2 time.windows.com.reddog.microsoft.com. A: read udp 10.0.1.5:49126->168.63.129.160:53: i/o timeout”
     }
   }
   ```

   формат сообщения:

   `[client’s IP address]:[client’s port] – [query ID] [type of the request] [class of the request] [name of the request] [protocol used] [request size in bytes] [EDNS0 DO (DNSSEC OK) bit set in the query] [EDNS0 buffer size advertised in the query] [response CODE] [response flags] [response size] [response duration]`

Существует три способа хранения журналов:

* **Учетная запись хранения** лучше всего подходит для длительного хранения журналов и их просмотра по мере необходимости.
* **Центры событий** — это отличный вариант для интеграции с другими инструментами управления событиями и сведениями о безопасности (SEIM), позволяющий получать оповещения о ваших ресурсах.
* **Журналы Azure Monitor**. журналы Azure Monitor лучше использовать для общего мониторинга приложения в режиме реального времени или для анализа тенденций.

## <a name="activity-logs"></a>Журналы действий

   Записи этого журнала собираются по умолчанию, и их можно просмотреть на портале Azure.

   [Журналы действий Azure](../azure-resource-manager/management/view-activity-logs.md) (ранее известные как операционные журналы и журналы аудита) можно использовать для просмотра всех операций, отправленных в подписку Azure.

## <a name="metrics"></a>Метрики

Метрики в Azure Monitor представляют собой числовые значения, которые описывают некоторый аспект системы в определенный момент времени. Метрики собираются каждую минуту и полезны для оповещений, так как их можно часто выдавать. Предупреждение может быть быстро запущено с относительно простой логикой.

Для брандмауэра Azure доступны следующие метрики.

- Число **попаданий правил приложения** — количество попаданий в правило приложения.

    Единица измерения: количество

- **Число попаданий сетевых правил** — количество попаданий в сетевое правило.

    Единица измерения: количество

- **Обработанные данные** — сумма данных, проходящих через брандмауэр в течение заданного временного периода.

    Единица: байт

- **Пропускная способность** для передачи данных в брандмауэре в секунду.

    Единица: бит в секунду

- **Состояние работоспособности брандмауэра** — указывает работоспособность брандмауэра на основе доступности порта SNAT.

    Единица: процент

   В этой метрике есть два измерения.
  - Состояние. Возможные значения: *Исправен*, *пониженный*, *неработоспособный*.
  - Причина: указывает причину соответствующего состояния брандмауэра. 

     Если порты SNAT используются > 95%, они считаются исчерпанными, а работоспособность — 50% с состоянием "**снижено работоспособность** " и "причина" —**порт SNAT**. Брандмауэр продолжает обрабатывать трафик, и существующие подключения не затрагиваются. Но возможны временные проблемы с установкой новых подключений.

     Если порты SNAT используются < 95%, брандмауэр считается работоспособным, а работоспособность отображается как 100%.

     Если сведения об использовании портов SNAT отсутствуют, отображается состояние работоспособности 0 %. 

- **Использование портов SNAT** — процент портов SNAT, используемых брандмауэром.

    Единица: процент

   Добавив дополнительные общедоступные IP-адреса для брандмауэра, вы сделаете доступными больше портов SNAT и снизите уровень их использования. Кроме того, дополнительные порты SNAT станут доступными при масштабировании брандмауэра по любой другой причине (например, для оптимизации работы ЦП или пропускной способности сети). Таким образом, заданный процент использования портов SNAT может быть отключен без добавления общедоступных IP-адресов, так как служба масштабируется. Можно напрямую управлять количеством доступных общедоступных IP-адресов, чтобы увеличить число доступных портов в брандмауэре. Но вы не можете напрямую управлять масштабированием брандмауэра.


## <a name="next-steps"></a>Дальнейшие действия

- См. дополнительные сведения об [отслеживании метрик и журналов Брандмауэра Azure](./firewall-diagnostics.md).

- Дополнительные сведения о метриках в Azure Monitor см. в разделе [метрики в Azure Monitor](../azure-monitor/essentials/data-platform-metrics.md).