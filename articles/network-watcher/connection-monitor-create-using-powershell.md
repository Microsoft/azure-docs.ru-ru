---
title: Создание монитора подключения с помощью PowerShell
titleSuffix: Azure Network Watcher
description: Узнайте, как создать монитор подключения с помощью PowerShell.
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
ms.openlocfilehash: 7f175d82b650871437a506ea4513f0ae28360f68
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99833037"
---
# <a name="create-a-connection-monitor-by-using-powershell"></a>Создание монитора подключения с помощью PowerShell

> [!IMPORTANT]
> Начиная с 1 июля 2021 вы не сможете добавлять новые тесты в существующую рабочую область или включать новую рабочую область в Монитор производительности сети. Кроме того, вы не сможете добавлять новые мониторы подключений в мониторе подключений (классическая модель). Вы можете продолжать использовать тесты и мониторы соединений, созданные до 1 июля 2021. Чтобы минимизировать перерыв в работе служб для текущих рабочих нагрузок, [перенесите тесты из монитор производительности сети ](migrate-to-connection-monitor-from-network-performance-monitor.md) или  [выполните миграцию из монитора подключений (классическая модель)](migrate-to-connection-monitor-from-connection-monitor-classic.md) в новый монитор подключений в наблюдатель за сетями Azure до 29 февраля 2024.


Узнайте, как использовать функцию монитора подключений наблюдателя за сетями Azure для отслеживания взаимодействия между ресурсами.


## <a name="before-you-begin"></a>Перед началом

В мониторах подключений, создаваемых с помощью монитора подключения, в качестве источников можно добавить локальные компьютеры и виртуальные машины Azure (ВМ). Эти мониторы соединений также могут отслеживать подключение к конечным точкам. Конечные точки могут находиться в Azure или на любом другом URL-адресе или IP.

Монитор подключения включает следующие сущности:

* **Ресурс монитора подключения**: ресурс Azure определенного региона. Все следующие сущности являются свойствами ресурса монитора подключения.
* **Конечная точка**: источник или назначение, участвующие в проверках подключения. Примеры конечных точек: виртуальные машины Azure, локальные агенты, URL-адреса и IP-адрес.
* **Конфигурация теста**: конфигурация, зависящая от протокола, для теста. В зависимости от выбранного протокола можно определить порт, пороговые значения, частоту проверки и другие параметры.
* **Тестовая группа**: группа, содержащая конечные точки источника, конечные точки назначения и конфигурации тестов. Монитор подключения может содержать более одной тестовой группы.
* **Тест**: сочетание исходной конечной точки, конечной точки назначения и конфигурации теста. Тест — это наиболее детализированный уровень доступности данных мониторинга. Данные мониторинга включают в себя процент проверок, завершившихся сбоем, и время приема-передачи (RTT).

    ![Схема, на которой показан монитор подключения, определяющий связь между группами тестов и тестами.](./media/connection-monitor-2-preview/cm-tg-2.png)

## <a name="steps-to-create-a-connection-monitor"></a>Шаги для создания монитора подключения

Используйте следующие команды для создания монитора подключения с помощью PowerShell.

```powershell

//Connect to your Azure account with the subscription
Connect-AzAccount
Select-AzSubscription -SubscriptionId <your-subscription>
//Select region
$nw = "NetworkWatcher_centraluseuap"
//Declare endpoints like Azure VM below. You can also give VNET,Subnet,Log Analytics workspace
$sourcevmid1 = New-AzNetworkWatcherConnectionMonitorEndpointObject -Name MyAzureVm -ResourceID /subscriptions/<your-subscription>/resourceGroups/<your resourceGroup>/providers/Microsoft.Compute/virtualMachines/<vm-name>
//Declare endpoints like URL, IPs
$bingEndpoint = New-AzNetworkWatcherConnectionMonitorEndpointObject -name Bing -Address www.bing.com # Destination URL
//Create test configuration.Choose Protocol and parametersSample configs below.

$IcmpProtocolConfiguration = New-AzNetworkWatcherConnectionMonitorProtocolConfigurationObject -IcmpProtocol
$TcpProtocolConfiguration = New-AzNetworkWatcherConnectionMonitorProtocolConfigurationObject -TcpProtocol -Port 80
$httpProtocolConfiguration = New-AzNetworkWatcherConnectionMonitorProtocolConfigurationObject -HttpProtocol -Port 443 -Method GET -RequestHeader @{Allow = "GET"} -ValidStatusCodeRange 2xx, 300-308 -PreferHTTPS
$httpTestConfiguration = New-AzNetworkWatcherConnectionMonitorTestConfigurationObject -Name http-tc -TestFrequencySec 60 -ProtocolConfiguration $httpProtocolConfiguration -SuccessThresholdChecksFailedPercent 20 -SuccessThresholdRoundTripTimeMs 30
$icmpTestConfiguration = New-AzNetworkWatcherConnectionMonitorTestConfigurationObject -Name icmp-tc -TestFrequencySec 30 -ProtocolConfiguration $icmpProtocolConfiguration -SuccessThresholdChecksFailedPercent 5 -SuccessThresholdRoundTripTimeMs 500
$tcpTestConfiguration = New-AzNetworkWatcherConnectionMonitorTestConfigurationObject -Name tcp-tc -TestFrequencySec 60 -ProtocolConfiguration $TcpProtocolConfiguration -SuccessThresholdChecksFailedPercent 20 -SuccessThresholdRoundTripTimeMs 30
//Create Test Group
$testGroup1 = New-AzNetworkWatcherConnectionMonitorTestGroupObject -Name testGroup1 -TestConfiguration $httpTestConfiguration, $tcpTestConfiguration, $icmpTestConfiguration -Source $sourcevmid1 -Destination $bingEndpoint,
$testname = "cmtest9"
//Create Connection Monitor
New-AzNetworkWatcherConnectionMonitor -NetworkWatcherName $nw -ResourceGroupName NetworkWatcherRG -Name $testname -TestGroup $testGroup1

```

## <a name="description-of-properties"></a>Описание свойств

* **ConnectionMonitorName**: имя ресурса монитора подключения.

* **Подписку: идентификатор** подписки, в которой вы хотите создать монитор подключения.

* **NW**: идентификатор ресурса наблюдателя за сетями, в котором создается монитор подключения.

* **Location**— регион, в котором создается монитор подключения.

* **Конечные точки**
    * **Name**: уникальное имя для каждой конечной точки.
    * **Идентификатор ресурса**. для конечных точек Azure идентификатор ресурса ссылается на Azure Resource Manager идентификатор ресурса для виртуальных машин. Для конечных точек, не относящихся к Azure, идентификатор ресурса относится к Azure Resource Manager ИДЕНТИФИКАТОРу ресурса для Log Analytics рабочей области, связанной с агентами, отличными от агентов Azure.
    * **Адрес**: применяется, только если идентификатор ресурса не указан или если идентификатор ресурса находится в log Analytics рабочей области. Если используется без идентификатора ресурса, это может быть URL или IP-адрес любой общедоступной конечной точки. Если используется с Log Analytics ИДЕНТИФИКАТОРом ресурса, это означает полное доменное имя агента мониторинга.
    * **Фильтр**. для конечных точек, не относящихся к Azure, используйте фильтры для выбора агентов мониторинга из рабочей области log Analytics в ресурсе монитора подключения. Если фильтры не заданы, для мониторинга можно использовать все агенты, принадлежащие к рабочей области Log Analytics.
        * **Тип**: укажите **адрес агента**.
        * **Адрес**: задается в качестве полного доменного имени локального агента.

* **Тестовые группы**
    * **Имя**: имя группы тестов.
    * **Источники**: Выберите конечные точки, созданные ранее. Для конечных точек источника на основе Azure необходимо установить расширение наблюдателя за сетями Azure. на конечных точках источника, отличных от Azure, должен быть установлен агент Azure Log Analytics. Сведения об установке агента для источника см. в разделе [Install Monitoring](./connection-monitor-overview.md#install-monitoring-agents)Agents.
    * **Назначения**: Выберите конечные точки, созданные ранее. Вы можете наблюдать за подключением к виртуальным машинам Azure или любой конечной точке (общедоступным IP-АДРЕСом или полным доменным именем), указав их в качестве назначений. В одной тестовой группе можно добавить виртуальные машины Azure, URL-адреса Office 365, URL-адреса Dynamics 365 и пользовательские конечные точки.
    * **Disable**: отключение мониторинга для всех источников и назначений, которые указывает группа тестирования.

* **Конфигурации тестов**
    * **Имя**: имя конфигурации теста.
    * **Тестфрекуенцисек**. Укажите, как часто источники направляются с помощью ping для указанного протокола и порта. Можно выбрать 30 секунд, 1 минуту, 5 минут, 15 минут или 30 минут. Источники проверяют возможность подключения к местам назначения на основе выбранного значения. Например, если выбрать 30 секунд, источники будут проверять подключение к назначению по крайней мере один раз в течение 30 секунд.
    * **Протокол**: выберите TCP, ICMP, HTTP или HTTPS. В зависимости от протокола можно также выбрать следующие конфигурации, относящиеся к протоколу.
        * **преферхттпс**: укажите, следует ли использовать HTTPS через HTTP.
        * **порт**. Укажите выбранный порт назначения.
        * **дисаблетрацерауте**: останавливаются источники от обнаружения ТОПОЛОГИИ и приема-передачи по прыжкам. Это относится к группам тестов с протоколом TCP или ICMP.
        * **метод**: выберите метод HTTP-запроса (Get или POST). Это относится к конфигурациям тестов с HTTP.
        * **путь**: укажите параметры пути, которые нужно добавить к URL-адресу.
        * **валидстатускодес**: выберите применимые коды состояния. Если код ответа не совпадает, отображается диагностическое сообщение.
        * **requestHeaders**: укажите строки заголовков настраиваемых запросов, которые передаются в назначение.
    * **Порог успеха**: Задайте пороговые значения для следующих сетевых параметров:
        * **чекксфаиледперцент**: задайте процент проверок, которые могут завершаться ошибкой, когда источники проверяют подключение к местам назначения, используя указанные критерии. Для протокола TCP или ICMP процент неудачных проверок может равняться проценту потери пакетов. Для протокола HTTP это поле представляет процент HTTP-запросов, которые не получили ответа.
        * **раундтриптимемс**: укажите, как долго могут выполняться источники для подключения к целевому объекту через конфигурацию теста в миллисекундах.

## <a name="scale-limits"></a> Ограничения масштабирования

Мониторы подключений имеют следующие ограничения масштабирования:

* Максимальное число мониторов подключения на подписку в одном регионе: 100
* Максимальное число групп тестов на монитор каждого подключения: 20
* Максимальное число источников и назначений на монитор подключения: 100
* Максимальное число конфигураций тестов на монитор каждого подключения: 20

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, [как анализировать данные мониторинга и настраивать оповещения](./connection-monitor-overview.md#analyze-monitoring-data-and-set-alerts).
* Узнайте [, как диагностировать проблемы в сети](./connection-monitor-overview.md#diagnose-issues-in-your-network).
