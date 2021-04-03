---
title: Открытие порта приложения в подсистеме балансировки нагрузки с помощью PowerShell
description: Пример сценария Azure PowerShell — открытие порта в Azure Load Balancer для приложения Service Fabric.
services: service-fabric
documentationcenter: ''
author: athinanthny
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.topic: sample
ms.date: 05/18/2018
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: 4be9944a53aaf0c697d55ff567dee7f2fbb3ed85
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87038112"
---
# <a name="open-an-application-port-in-the-azure-load-balancer"></a>Открытие порта приложения в Azure Load Balancer

Приложения Service Fabric, работающие в Azure, находятся за Azure Load Balancer. Этот пример сценария открывает порт в Azure Load Balancer, чтобы приложение Service Fabric могли взаимодействовать с внешними клиентами. Измените параметры, если это необходимо. Если кластер находится в группе безопасности сети, также [добавьте правило входящего трафика для группы безопасности сети](service-fabric-powershell-add-nsg-rule.md), чтобы разрешить входящий трафик.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

При необходимости установите модуль PowerShell ServiceFabric вместе с [пакетом SDK для Service Fabric](../service-fabric-get-started.md). 

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../powershell_scripts/service-fabric/open-port-in-load-balancer/open-port-in-load-balancer.ps1 "Open a port in the load balancer")]

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Получает ресурс Azure.  |
| [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer) | Получает Azure Load Balancer. |
| [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) | Добавляет конфигурацию пробы для подсистемы балансировки нагрузки.|
| [Get-AzLoadBalancerProbeConfig](/powershell/module/az.network/get-azloadbalancerprobeconfig) | Получает конфигурацию пробы для подсистемы балансировки нагрузки. |
| [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) | Добавляет правило конфигурации в подсистему балансировки нагрузки. |
| [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer) | Задает состояние цели для подсистемы балансировки нагрузки. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев PowerShell для Azure Service Fabric см. в разделе [Примеры сценариев Azure PowerShell](../service-fabric-powershell-samples.md).
