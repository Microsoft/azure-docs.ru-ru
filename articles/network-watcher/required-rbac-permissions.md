---
title: Разрешения RBAC Azure, необходимые для использования возможностей
titleSuffix: Azure Network Watcher
description: Узнайте, какие разрешения на управление доступом на основе ролей доступны для работы с возможностями Наблюдателя за сетями.
services: network-watcher
documentationcenter: ''
author: damendo
ms.service: network-watcher
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/07/2021
ms.author: damendo
ms.openlocfilehash: 47418b9c5235255ff7dbf4a1a151e51e4c9aba58
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019675"
---
# <a name="azure-role-based-access-control-permissions-required-to-use-network-watcher-capabilities"></a>Разрешения на управление доступом на основе ролей в Azure, необходимые для использования возможностей наблюдателя за сетями

Управление доступом на основе ролей в Azure (Azure RBAC) позволяет назначать членам вашей организации только определенные действия, необходимые для выполнения назначенных им обязанностей. Чтобы использовать возможности Наблюдателя за сетями, учетной записи, в которую вы вошли, должна быть назначена встроенная роль [Владелец](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#owner), [Участник](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#contributor) или [Участник сетей](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#network-contributor) или назначена [пользовательская роль](../role-based-access-control/custom-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json), которая обладает свойствами, перечисленными для каждой возможности Наблюдателя за сетями в следующем разделе. Дополнительные сведения о Наблюдателе за сетями см. в [этой статье](network-watcher-monitoring-overview.md).

## <a name="network-watcher"></a>Наблюдатель за сетями

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/read                              | Получение Наблюдателя за сетями                                          |
| Microsoft.Network/networkWatchers/write                             | Создание или обновление Наблюдателя за сетями                             |
| Microsoft.Network/networkWatchers/delete                            | Удаление Наблюдателя за сетями                                       |

## <a name="nsg-flow-logs"></a>Журналы потоков NSG

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/configureFlowLog/action           | Настройка журнала потока                                           |
| Microsoft.Network/networkWatchers/queryFlowLogStatus/action         | Запрос состояния для журнала потока                                    |

## <a name="connection-troubleshoot"></a>Устранение неполадок подключения

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/connectivityCheck/action          | Инициирование теста на устранение неполадок подключения
| Microsoft.Network/networkWatchers/queryTroubleshootResult/action    | Запрос результатов теста устранения неполадок подключения                |
| Microsoft.Network/networkWatchers/troubleshoot/action               | Выполнение теста на устранение неполадок подключения                             |

## <a name="connection-monitor"></a>Монитор подключения

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/connectionMonitors/start/action   | Запуск монитора подключения                                     |
| Microsoft.Network/networkWatchers/connectionMonitors/stop/action    | Остановка монитора подключения                                      |
| Microsoft.Network/networkWatchers/connectionMonitors/query/action   | Запрос монитора подключения                                     |
| Microsoft.Network/networkWatchers/connectionMonitors/read           | Получение монитора подключения                                       |
| Microsoft.Network/networkWatchers/connectionMonitors/write          | Создание монитора подключения                                    |
| Microsoft.Network/networkWatchers/connectionMonitors/delete         | Удаление монитора подключения                                    |

## <a name="packet-capture"></a>Запись пакетов

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/packetCaptures/queryStatus/action | Запрос состояния записи пакета                           |
| Microsoft.Network/networkWatchers/packetCaptures/stop/action        | Прекращение записи пакета                                          |
| Microsoft.Network/networkWatchers/packetCaptures/read               | Получение записи пакета                                           |
| Microsoft.Network/networkWatchers/packetCaptures/write              | Создание записи пакетов                                        |
| Microsoft.Network/networkWatchers/packetCaptures/delete             | Удаление записи пакета                                        |

## <a name="ip-flow-verify"></a>Проверка IP-потока

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/ipFlowVerify/action               | Проверка IP-потока                                              |

## <a name="next-hop"></a>Следующий прыжок

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/nextHop/action                    | Получение следующего прыжка с виртуальной машины                                     |

## <a name="network-security-group-view"></a>Представление группы безопасности сети

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/securityGroupView/action          | Просмотр групп безопасности                                           |

## <a name="topology"></a>Топология

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/topology/action                   | Получение топологии                                                   |
| Microsoft. Network/Нетворкватчерс/топология/чтение                     | То же, что и выше                                                  |

## <a name="reachability-report"></a>Отчет о возможности доступа

| Действие                                                              | Описание                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/azureReachabilityReport/action    | Получение отчета о возможности доступа Azure                               |


## <a name="additional-actions"></a>Дополнительные действия

Возможностям Наблюдателя за сетью также требуются следующие действия:

| Действия                                                           | Описание                                                    |
| ---------                                                           | -------------                                                  |
| Microsoft.Authorization/\*/Read                                     | Используется для выборки назначений ролей и определений политик Azure          |
| Microsoft.Resources/subscriptions/resourceGroups/Read;               | Используется для перечисления всех групп ресурсов в подписке    |
| Microsoft.Storage/storageAccounts/Read                              | Используется для получения свойств указанной учетной записи хранения   |
| Microsoft. Storage/storageAccounts/Листсервицесас/действие, </br> Microsoft. Storage/storageAccounts/Листаккаунтсас/действие, <br> Microsoft.Storage/storageAccounts/listKeys/Action;| Используется для получения подписанных URL-адресов (SAS), обеспечивающих [безопасный доступ к учетной записи хранения](../storage/common/storage-sas-overview.md) и записи в учетную запись хранения. |
| Microsoft. COMPUTE/virtualMachines/Read, </br> Microsoft.Compute/virtualMachines/Write;| Используется для входа в виртуальную машину, для записи пакетов и их передачи в учетную запись хранения|
| Microsoft.Compute/virtualMachines/extensions/Read </br> Microsoft.Compute/virtualMachines/extensions/Write| Используется для проверки наличия расширения наблюдателя за сетями и установки при необходимости |
| Microsoft. COMPUTE/virtualMachineScaleSets/Read, </br> Microsoft.Compute/virtualMachineScaleSets/Write;| Используется для доступа к масштабируемым наборам виртуальных машин, для записи пакетов и их передачи в учетную запись хранения|
| Microsoft. COMPUTE/virtualMachineScaleSets/Extensions/Read, </br> Microsoft.Compute/virtualMachineScaleSets/extensions/Write| Используется для проверки наличия расширения наблюдателя за сетями и установки при необходимости |
| Microsoft.Insights/alertRules/*                                     | Используется для настройки оповещений метрик                                     |
| Microsoft.Support/*                                                 | Используется для создания и обновления билетов на службу поддержки в службе "наблюдатель за сетями" |