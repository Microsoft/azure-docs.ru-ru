---
title: Анализ безопасности сети — представление группы безопасности — Azure REST API
titleSuffix: Azure Network Watcher
description: В этой статье описывается, как REST API Azure проанализировать безопасность виртуальных машин с помощью представления группы безопасности.
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
ms.openlocfilehash: 2efd3e9c9ca97ea3d94b03bd5e440cd24d5da5da
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94960611"
---
# <a name="analyze-your-virtual-machine-security-with-security-group-view-using-rest-api"></a>Анализ безопасности виртуальной машины с использованием представления группы безопасности в REST API

> [!div class="op_single_selector"]
> - [PowerShell](network-watcher-security-group-view-powershell.md)
> - [Azure CLI](network-watcher-security-group-view-cli.md)
> - [REST API](network-watcher-security-group-view-rest.md)

> [!NOTE]
> API представления группы безопасности больше не поддерживается и скоро станет устаревшим. Используйте [действующие правила безопасности](./network-watcher-security-group-view-overview.md) , которые предоставляют те же функциональные возможности. 

Представление группы безопасности возвращает настроенные и действующие правила сетевой безопасности, применяемые к виртуальной машине. Эта возможность полезна для аудита и диагностики групп безопасности сети и настроенных на виртуальной машине правил, позволяющих обеспечить разрешение или отклонение трафика соответствующим образом. В этой статье мы покажем, как получить эффективные правила безопасности и применить их к виртуальной машине с помощью REST API.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Перед началом

В этом сценарии вы вызовите REST API Наблюдателя за сетями, чтобы получить представление группы безопасности для виртуальной машины. Чтобы вызвать REST API при помощи командлетов PowerShell, вам потребуется ARMClient. ARMClient находится на шоколадном [ARMClientе в шоколаде](https://chocolatey.org/packages/ARMClient)

В этом сценарии предполагается, что вы создали Наблюдатель за сетями в соответствии с инструкциями в статье [Create a Network Watcher](network-watcher-create.md) (Создание Наблюдателя за сетями). Предполагается также, что у вас имеется группа ресурсов с допустимой виртуальной машиной.

## <a name="scenario"></a>Сценарий

В сценарии, описанном в этой статье, вы получите эффективные правила безопасности и примените их к указанной виртуальной машине.

## <a name="log-in-with-armclient"></a>выполните вход с помощью ARMClient;

```powershell
armclient login
```

## <a name="retrieve-a-virtual-machine"></a>Получение виртуальной машины

Чтобы получить сведения о виртуальной машине, выполните приведенный ниже скрипт, указав в нем следующие переменные:

- **SubscriptionId** — идентификатор подписки можно также получить с помощью командлета **Get-азсубскриптион** .
- **resourceGroupName** — имя группы ресурсов, в которой содержатся виртуальные машины.

```powershell
$subscriptionId = '<subscription id>'
$resourceGroupName = '<resource group name>'

armclient get https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Compute/virtualMachines?api-version=2015-05-01-preview
```

Вам потребуется **идентификатор**, расположенный под строкой `Microsoft.Compute/virtualMachines` в ответе, как показано в примере ниже.

```json
...,
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft
.Network/networkInterfaces/{nicName}"
            }
          ]
        },
        "provisioningState": "Succeeded"
      },
      "resources": [
        {
          "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Com
pute/virtualMachines/{vmName}/extensions/CustomScriptExtension"
        }
      ],
      "type": "Microsoft.Compute/virtualMachines",
      "location": "westcentralus",
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute
/virtualMachines/{vmName}",
      "name": "{vmName}"
    }
  ]
}
```

## <a name="get-security-group-view-for-virtual-machine"></a>Получение представления группы безопасности для виртуальной машины

Приведенный ниже пример запрашивает представление группы безопасности целевой виртуальной машины. Полученный после выполнения результат можно использовать, чтобы сравнить правила и параметры безопасности, определенные источником. Это позволит оценить изменения конфигурации.

```powershell
$subscriptionId = "<subscription id>"
$resourceGroupName = "<resource group name>"
$networkWatcherName = "<network watcher name>"
$targetUri = "<uri of target resource>" # Example: /subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.compute/virtualMachine/$vmName

$requestBody = @"
{
    'targetResourceId': '${targetUri}'

}
"@
armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/securityGroupView?api-version=2016-12-01" $requestBody -verbose
```

## <a name="view-the-response"></a>Просмотр ответа

Ниже приведен пример ответа, возвращенного после выполнения предыдущей команды. В результатах приводятся все действующие и применяемые правила безопасности на виртуальной машине, разделенные по группам **NetworkInterfaceSecurityRules**, **DefaultSecurityRules** и **EffectiveSecurityRules**.

```json

{
  "networkInterfaces": [
    {
      "securityRuleAssociations": {
        "networkInterfaceAssociation": {
          "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkInterfaces/{nicName}",
          "securityRules": [
            {
              "name": "default-allow-rdp",
              "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}/securityRules/default-allow-rdp",
              "etag": "W/\"d4c411d4-0d62-49dc-8092-3d4b57825740\"",
              "properties": {
                "provisioningState": "Succeeded",
                "protocol": "TCP",
                "sourcePortRange": "*",
                "destinationPortRange": "3389",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 1000,
                "direction": "Inbound"
              }
            }
          ]
        },
        "defaultSecurityRules": [
          {
            "name": "AllowVnetInBound",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}/defaultSecurityRules/",
            "properties": {
              "provisioningState": "Succeeded",
              "description": "Allow inbound traffic from all VMs in VNET",
              "protocol": "*",
              "sourcePortRange": "*",
              "destinationPortRange": "*",
              "sourceAddressPrefix": "VirtualNetwork",
              "destinationAddressPrefix": "VirtualNetwork",
              "access": "Allow",
              "priority": 65000,
              "direction": "Inbound"
            }
          },
          ...
        ],
        "effectiveSecurityRules": [
          {
            "name": "DefaultOutboundDenyAll",
            "protocol": "All",
            "sourcePortRange": "0-65535",
            "destinationPortRange": "0-65535",
            "sourceAddressPrefix": "*",
            "destinationAddressPrefix": "*",
            "access": "Deny",
            "priority": 65500,
            "direction": "Outbound"
          },
          ...
        ]
      }
    }
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения об автоматизации проверки групп безопасности сети см. в статье [Auditing Network Security Groups (NSG) with Network Watcher](network-watcher-security-group-view-powershell.md) (Выполнение аудита групп безопасности сети с помощью Наблюдателя за сетями).