---
title: Анализ безопасности сети — представление группы безопасности — Azure PowerShell
titleSuffix: Azure Network Watcher
description: В этой статье вы узнаете, как проанализировать безопасность виртуальных машин, используя представление группы безопасности, с помощью PowerShell.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 2da4ebfa6a9ce64b03e6e1d29956f740f6cb3c1b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94960654"
---
# <a name="analyze-your-virtual-machine-security-with-security-group-view-using-powershell"></a>Анализ безопасности виртуальной машины с использованием представления группы безопасности (PowerShell)

> [!div class="op_single_selector"]
> - [PowerShell](network-watcher-security-group-view-powershell.md)
> - [Azure CLI](network-watcher-security-group-view-cli.md)
> - [REST API](network-watcher-security-group-view-rest.md)

> [!NOTE]
> API представления группы безопасности больше не поддерживается и скоро станет устаревшим. Используйте [действующие правила безопасности](./network-watcher-security-group-view-overview.md) , которые предоставляют те же функциональные возможности. 

Представление группы безопасности возвращает настроенные и действующие правила сетевой безопасности, применяемые к виртуальной машине. Эта возможность полезна для аудита и диагностики групп безопасности сети и настроенных на виртуальной машине правил, позволяющих обеспечить разрешение или отклонение трафика соответствующим образом. В этой статье мы покажем, как получить настроенные и действующие правила безопасности, применяемые к виртуальной машине, с помощью PowerShell.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Перед началом

В этом сценарии вы получите сведения о правиле безопасности, используя командлет `Get-AzNetworkWatcherSecurityGroupView`.

В этом сценарии предполагается, что вы создали Наблюдатель за сетями в соответствии с инструкциями в статье [Create a Network Watcher](network-watcher-create.md) (Создание Наблюдателя за сетями).

## <a name="scenario"></a>Сценарий

В сценарии, описанном в этой статье, вы получите настроенные и действующие правила безопасности для указанной виртуальной машины.

## <a name="retrieve-network-watcher"></a>Извлечение Наблюдателя за сетями

Сначала необходимо получить экземпляр Наблюдателя за сетями. Эта переменная передается в командлет `Get-AzNetworkWatcherSecurityGroupView`.

```powershell
$networkWatcher = Get-AzResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" }
```

## <a name="get-a-vm"></a>Получение виртуальной машины

Для повторного выполнения командлета `Get-AzNetworkWatcherSecurityGroupView` необходима виртуальная машина. Ниже приведен пример получения объекта виртуальной машины.

```powershell
$VM = Get-AzVM -ResourceGroupName testrg -Name testvm1
```

## <a name="retrieve-security-group-view"></a>Получение представления группы безопасности

Далее необходимо получить результат представления группы безопасности.

```powershell
$secgroup = Get-AzNetworkWatcherSecurityGroupView -NetworkWatcher $networkWatcher -TargetVirtualMachineId $VM.Id
```

## <a name="viewing-the-results"></a>Просмотр результатов

Далее приведен пример сокращенного ответа возвращенных результатов. В результатах приводятся все действующие и применяемые правила безопасности на виртуальной машине, разделенные по группам **NetworkInterfaceSecurityRules**, **DefaultSecurityRules** и **EffectiveSecurityRules**.

```
NetworkInterfaces : [
                      {
                        "NetworkInterfaceSecurityRules": [
                          {
                            "Name": "default-allow-rdp",
                            "Etag": "W/\"d4c411d4-0d62-49dc-8092-3d4b57825740\"",
                            "Id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg2/providers/Microsoft.Network/networkSecurityGroups/testvm2-nsg/securityRules/default-allow-rdp",
                            "Protocol": "TCP",
                            "SourcePortRange": "*",
                            "DestinationPortRange": "3389",
                            "SourceAddressPrefix": "*",
                            "DestinationAddressPrefix": "*",
                            "Access": "Allow",
                            "Priority": 1000,
                            "Direction": "Inbound",
                            "ProvisioningState": "Succeeded"
                          }
                          ...
                        ],
                        "DefaultSecurityRules": [
                          {
                            "Name": "AllowVnetInBound",
                            "Id": "/subscriptions00000000-0000-0000-0000-000000000000/resourceGroups/testrg2/providers/Microsoft.Network/networkSecurityGroups/testvm2-nsg/defaultSecurityRules/",
                            "Description": "Allow inbound traffic from all VMs in VNET",
                            "Protocol": "*",
                            "SourcePortRange": "*",
                            "DestinationPortRange": "*",
                            "SourceAddressPrefix": "VirtualNetwork",
                            "DestinationAddressPrefix": "VirtualNetwork",
                            "Access": "Allow",
                            "Priority": 65000,
                            "Direction": "Inbound",
                            "ProvisioningState": "Succeeded"
                          }
                          ...
                        ],
                        "EffectiveSecurityRules": [
                          {
                            "Name": "DefaultOutboundDenyAll",
                            "Protocol": "All",
                            "SourcePortRange": "0-65535",
                            "DestinationPortRange": "0-65535",
                            "SourceAddressPrefix": "*",
                            "DestinationAddressPrefix": "*",
                            "ExpandedSourceAddressPrefix": [],
                            "ExpandedDestinationAddressPrefix": [],
                            "Access": "Deny",
                            "Priority": 65500,
                            "Direction": "Outbound"
                          },
                          ...
                        ]
                      }
                    ]
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения об автоматизации проверки групп безопасности сети см. в статье [Auditing Network Security Groups (NSG) with Network Watcher](network-watcher-nsg-auditing-powershell.md) (Выполнение аудита групп безопасности сети с помощью Наблюдателя за сетями).