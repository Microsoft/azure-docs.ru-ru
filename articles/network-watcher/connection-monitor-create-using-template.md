---
title: Создание монитора подключения — шаблон ARM
titleSuffix: Azure Network Watcher
description: Узнайте, как создать монитор подключений с помощью ARMClient.
services: network-watcher
documentationcenter: na
author: vinigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: vinigam
ms.openlocfilehash: 46bdaf932d4224bf97b46e7713d49d815ca1bcdd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99833003"
---
# <a name="create-a-connection-monitor-using-the-arm-template"></a>Создание монитора подключения с помощью шаблона ARM

> [!IMPORTANT]
> Начиная с 1 июля 2021 вы не сможете добавлять новые тесты в существующую рабочую область или включать новую рабочую область в Монитор производительности сети. Кроме того, вы не сможете добавлять новые мониторы подключений в мониторе подключений (классическая модель). Вы можете продолжать использовать тесты и мониторы соединений, созданные до 1 июля 2021. Чтобы минимизировать перерыв в работе служб для текущих рабочих нагрузок, [перенесите тесты из монитор производительности сети ](migrate-to-connection-monitor-from-network-performance-monitor.md) или  [выполните миграцию из монитора подключений (классическая модель)](migrate-to-connection-monitor-from-connection-monitor-classic.md) в новый монитор подключений в наблюдатель за сетями Azure до 29 февраля 2024.

Узнайте, как создать монитор подключения для отслеживания взаимодействия между ресурсами с помощью ARMClient. Он поддерживает Гибридные развертывания в облаке Azure.


## <a name="before-you-begin"></a>Перед началом 

В мониторах подключений, создаваемых в мониторе подключения, можно добавить локальные компьютеры и виртуальные машины Azure в качестве источников. Эти мониторы соединений также могут отслеживать подключение к конечным точкам. Конечные точки могут находиться в Azure или на любом другом URL-адресе или IP.

Монитор подключения включает следующие сущности:

* **Ресурс монитора подключения** — ресурс Azure определенного региона. Все следующие сущности являются свойствами ресурса монитора подключения.
* **Конечная точка** — источник или назначение, участвующие в проверках подключения. Примеры конечных точек: виртуальные машины Azure, локальные агенты, URL-адреса и IP-адрес.
* **Конфигурация теста** — конфигурация, зависящая от протокола, для теста. В зависимости от выбранного протокола можно определить порт, пороговые значения, частоту проверки и другие параметры.
* **Тестовая группа** — группа, содержащая конечные точки источника, конечные точки назначения и конфигурации тестов. Монитор подключения может содержать более одной тестовой группы.
* **Тест** — сочетание исходной конечной точки, конечной точки назначения и конфигурации теста. Тест — это наиболее детализированный уровень доступности данных мониторинга. Данные мониторинга включают в себя процент проверок, завершившихся сбоем, и время приема-передачи (RTT).

    ![Схема, показывающая монитор подключения, определение связи между группами тестов и тестами](./media/connection-monitor-2-preview/cm-tg-2.png)

## <a name="steps-to-create-with-sample-arm-template"></a>Шаги для создания образца шаблона ARM

Используйте следующий код, чтобы создать монитор подключения с помощью ARMClient.

```armclient
$connectionMonitorName = "sampleConnectionMonitor"

$ARM = "https://management.azure.com"

$SUB = "subscriptions/<subscription id 1>;"

$NW = "resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatchers/NetworkWatcher\_<region>"

$body =

"{

location: '<region>',

properties: {

endpoints: [{

name: 'endpoint_workspace_machine',

type: 'MMAWorkspaceMachine',

resourceId: '/subscriptions/<subscription id>/resourcegroups/<resource group>/providers/Microsoft.OperationalInsights/workspaces/sampleWorkspace',

//Example 1: Choose a machine

address : '<non-Azure machine FQDN>'
}

//Example 2: Select IP from chosen machines

address : '<non-Azure machine FQDN>

"scope": {
      "include": [
            {
                  "address": "<IP belonging to machine chosen above>"  
        }
       ]
      }
   }    
   
name: 'endpoint_workspace_network',

type: 'MMAWorkspaceNetwork',

resourceId: '/subscriptions/<subscription id>/resourcegroups/<resource group>/providers/Microsoft.OperationalInsights/workspaces/sampleWorkspace',

 coverage level : 'high', //Optional
 
 //Include subnets. You can also exclude IPs from subnet to exclude from monitoring
 
 scope: {
      "include": [
            {
                  "address": "<subnet 1 mask>" // Eg: 10.10.1.0/28
            },
            {
                  "address": "<subnet 2 mask>" 
            }
      ],
      "exclude": [
            { 
            "address" : "<ip-from-included-subnets-that-should-be-excluded>"
        }
      ]
     }
},

//Use a Azure VM as an endpoint
{

name: 'endpoint_virtualmachine',

resourceId: '/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<vm-name>'

},

//Use an Azure VNET or Subnet as an endpoint

 {

name: 'endpoint_vnet_subnet',

resourceId: '<resource id of VNET or subnet'
coverage level: 'high' //Optional

//Scope is optional.

  "scope": {
      "include": [
            {
                  "address": "<subnet 1 mask>" // Eg: 10.10.1.0/28 .This subnet should match with any existing subnet in vnet
            }
      ],
    "exclude": [
            {
                  "address": "<ip-from-included-subnets-that-should-be-excluded>" // If used with include, IP should be part of the subnet defined above. Without include, this could be any address within vnet range or any specific subnet range as a whole.
            }
      ]
  }
   },

//Endpoint as a URL
{

name: 'azure portal'

address: '<URL>'

   },

//Endpoint as an IP 
 {

    name: 'ip',

     address: '<IP>'

 }

  ],

  testGroups: [{

    name: 'Connectivity to Azure Portal and Public IP',

    testConfigurations: ['http', 'https', 'tcpEnabled', 'icmpEnabled'],

    sources: ['vm1', 'workspace'],

    destinations: ['azure portal', 'ip']

   },

{

    name: 'Connectivty from Azure VM 1 to Azure VM 2',

   // Choose your protocol
   
    testConfigurations: ['http', 'https', 'tcpDisabled', 'icmpDisabled'],

    sources: ['vm1'],

    destinations: ['vm2'],

    disable: true

   }

  ],

  testConfigurations: [{

    name: 'http',

    testFrequencySec: <frequency>,

    protocol: 'HTTP',

    successThreshold: {

     checksFailedPercent: <threshold for checks failed %>,

     roundTripTimeMs: <threshold for RTT>

    }

   }, {

    name: 'https',

    testFrequencySec: <frequency>,

    protocol: 'HTTP',

    httpConfiguration: {
    
     port: '<port of choice>'
  
    preferHTTPS: true // If port chosen is not 80 or 443
    
    method: 'GET', //Choose GET or POST
    
    path: '/', //Specify path for request
         
    requestHeaders: [
            {
              "name": "Content-Type",
              "value": "appication/json"
            }
          ],
          
    validStatusCodeRanges: [ "102", "200-202", "3xx" ], //Samples
          
    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, 
   {

    name: 'tcpEnabled',

    testFrequencySec: <frequency>,

    protocol: 'TCP',

    tcpConfiguration: {

     port: 80

    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, {

    name: 'icmpEnabled',

    testFrequencySec: <frequency>,

    protocol: 'ICMP',

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, {

    name: 'icmpDisabled',

    testFrequencySec: <frequency>,

    protocol: 'ICMP',

    icmpConfiguration: {

     disableTraceRoute: true

    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }, {

    name: 'tcpDisabled',

    testFrequencySec: <frequency>,

    protocol: 'TCP',

    tcpConfiguration: {

     port: 80,

     disableTraceRoute: true

    },

    successThreshold: {

     checksFailedPercent: <choose your checks failed threshold>,

     roundTripTimeMs: <choose your RTT threshold>

    }

   }

  ]

 }

} "
```

Ниже приведена команда развертывания.
```
armclient PUT $ARM/$SUB/$NW/connectionMonitors/$connectionMonitorName/?api-version=2019-07-01 $body -verbose
```

## <a name="description-of-properties"></a>Описание свойств

* connectionMonitorName — имя ресурса монитора подключения.

* Идентификатор подписки, для которой требуется создать монитор подключения

* NW — идентификатор ресурса наблюдателя за сетями, в котором будет создан CM 

* Location — регион, в котором будет создан монитор подключения

* Конечные точки
    * имя — уникальное имя для каждой конечной точки.
    * resourceId — для конечных точек Azure идентификатор ресурса ссылается на Azure Resource Manager идентификатор ресурса для виртуальных машин. Для конечных точек, не относящихся к Azure, идентификатор ресурса относится к Azure Resource Manager ИДЕНТИФИКАТОРу ресурса для Log Analytics рабочей области, связанной с агентами, отличными от агентов Azure.
    * адрес — применяется, только если не указан идентификатор ресурса или идентификатор ресурса Log Analytics рабочей области. Если используется с Log Analyticsным ИДЕНТИФИКАТОРом ресурса, это означает полное доменное имя агента, который можно использовать для мониторинга. Если используется без идентификатора ресурса, это может быть URL-адрес или IP-адреса любой общедоступной конечной точки.
    * фильтр. для конечных точек, не относящихся к Azure, используйте фильтр для выбора агентов из рабочей области Log Analytics, которая будет использоваться для мониторинга в ресурсе монитора подключения. Если фильтры не заданы, для мониторинга можно использовать все агенты, принадлежащие рабочей области Log Analytics.
        * Тип — задайте для параметра Тип значение "адрес агента".
        * адрес — укажите адрес в качестве полного доменного имени локального агента.

* Тестовые группы
    * Имя — имя группы тестов.
    * Тестконфигуратионс — конфигурации тестирования в зависимости от того, какие конечные точки источника подключаются к конечным точкам
    * источники — выберите конечные точки, созданные выше. Для конечных точек источника на основе Azure должно быть установлено расширение наблюдателя за сетями Azure, а в других конечных точках источника, созданных на основе Azure, требуется Хавеазуре Log Analytics Agent. Сведения об установке агента для источника см. в разделе [Install Monitoring](./connection-monitor-overview.md#install-monitoring-agents)Agents.
    * назначения — выберите созданные ранее конечные точки. Вы можете наблюдать за подключением к виртуальным машинам Azure или любой конечной точке (общедоступным IP-АДРЕСом или полным доменным именем), указав их в качестве назначений. В одной тестовой группе можно добавить виртуальные машины Azure, URL-адреса Office 365, URL-адреса Dynamics 365 и пользовательские конечные точки.
    * отключить — используйте это поле, чтобы отключить наблюдение для всех источников и назначений, которые указывает группа тестирования.

* Конфигурации тестов
    * Name — имя конфигурации теста.
    * Тестфрекуенцисек — укажите, как часто источники будут проверять связь с адресатами по указанному протоколу и порту. Можно выбрать 30 секунд, 1 минуту, 5 минут, 15 минут или 30 минут. Источники проверяют возможность подключения к местам назначения на основе выбранного значения. Например, если выбрать 30 секунд, источники будут проверять подключение к назначению по крайней мере один раз в течение 30 секунд.
    * Протокол — можно выбрать TCP, ICMP, HTTP или HTTPS. В зависимости от протокола можно использовать определенные конфигурации протокола.
    
        * Преферхттпс — укажите, следует ли использовать HTTPS через HTTP, если используемый порт не является ни 80, ни 443
        * Порт — укажите выбранный порт назначения.
        * Дисаблетрацерауте — применяется к конфигурациям тестов с протоколом TCP или ICMP. ИТ-отделы не обнаруживают возможности обнаружения топологии и приема-передачи по прыжкам.
        * метод — применяется к конфигурациям тестов, для которых используется протокол HTTP. Выберите метод HTTP-запроса — GET или POST.
        * путь — укажите параметры пути для добавления к URL-адресу
        * Валидстатускодес — выберите соответствующие коды состояния. Если код ответа не соответствует этому списку, вы получите диагностическое сообщение.
        * requestHeaders — укажите строки заголовков настраиваемых запросов, которые будут переданы в назначение.
        
    * Сукцесссрешолд — можно задать пороговые значения для следующих сетевых параметров:
        * Чекксфаиледперцент — задайте процент проверок, которые могут завершаться ошибкой, когда источники проверяют подключение к местам назначения, используя указанные критерии. Для протокола TCP или ICMP процент неудачных проверок может равняться проценту потери пакетов. Для протокола HTTP это поле представляет процент HTTP-запросов, которые не получили ответа.
        * Раундтриптимемс — задайте время приема-передачи в миллисекундах, в течение которого источники могут быть выбраны для подключения к назначению в конфигурации теста.

## <a name="scale-limits"></a> Ограничения масштабирования

Мониторы подключений имеют следующие ограничения масштабирования:

* Максимальное число мониторов подключения на подписку в одном регионе: 100
* Максимальное число групп тестов на монитор каждого подключения: 20
* Максимальное число источников и назначений на монитор подключения: 100
* Максимальное число конфигураций тестов на монитор подключений: 20 через ARMClient

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, [как анализировать данные мониторинга и настраивать оповещения](./connection-monitor-overview.md#analyze-monitoring-data-and-set-alerts) .
* Узнайте [, как диагностировать проблемы в сети](./connection-monitor-overview.md#diagnose-issues-in-your-network)
