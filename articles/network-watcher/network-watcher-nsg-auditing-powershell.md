---
title: Автоматизация NSG Auditing — представление группы безопасности
titleSuffix: Azure Network Watcher
description: На этой странице представлены инструкции по настройке аудита группы безопасности сети
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
ms.openlocfilehash: 177215775c9e83286aa98872eed0ab211a8f36ff
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94948754"
---
# <a name="automate-nsg-auditing-with-azure-network-watcher-security-group-view"></a>Автоматизация аудита группы безопасности сети с помощью представления группы безопасности в Наблюдателе за сетями Azure

Клиенты часто сталкиваются с проблемами при проверке показателей системы безопасности своей инфраструктуры. Это же характерно и для виртуальных машин в Azure. Очень важно иметь одинаковый профиль безопасности, основанный на применении правил группы безопасности сети (NSG). Теперь с помощью представления группы безопасности можно получить список правил, применяемых к виртуальной машине в NSG. Вы можете определить золотой профиль безопасности NSG и инициировать представление группы безопасности с еженедельной периодичностью, а также сравнивать выходные данные в золотом профиле и составлять отчет. Таким образом вы сможете с легкостью определить все виртуальные машины, которые не соответствуют предписанному профилю безопасности.

Чтобы ознакомиться с группами безопасности сети, прочитайте статью [Безопасность сети](../virtual-network/network-security-groups-overview.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Перед началом

В нашем примере вам предлагается сравнить известные базовые показатели и результаты в представлении группы безопасности, возвращаемые для виртуальной машины.

В этом сценарии предполагается, что вы создали Наблюдатель за сетями в соответствии с инструкциями в статье [Create a Network Watcher](network-watcher-create.md) (Создание Наблюдателя за сетями). Предполагается также, что у вас имеется группа ресурсов с допустимой виртуальной машиной.

## <a name="scenario"></a>Сценарий

В рамках сценария, описанного в этой статье, вы получаете представление группы безопасности для виртуальной машины.

Вам предстоит:

- получить набор известных проверенных правил;
- получить виртуальную машину с помощью REST API;
- Получение представления группы безопасности для виртуальной машины
- оценить ответ.

## <a name="retrieve-rule-set"></a>Получение набора правил

Первый шаг в этом примере — это работа с существующими базовыми показателями. В следующем примере показан объект JSON, извлеченный из существующей группы безопасности сети с помощью командлета `Get-AzNetworkSecurityGroup`. Он представляет базовые показатели.

```json
[
    {
        "Description":  null,
        "Protocol":  "TCP",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "3389",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1000,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "default-allow-rdp",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/default-allow-rdp"
    },
    {
        "Description":  null,
        "Protocol":  "*",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "111",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1010,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "MyRuleDoNotDelete",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/MyRuleDoNotDelete"
    },
    {
        "Description":  null,
        "Protocol":  "*",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "112",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1020,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "My2ndRuleDoNotDelete",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/My2ndRuleDoNotDelete"
    },
    {
        "Description":  null,
        "Protocol":  "TCP",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "5672",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Deny",
        "Priority":  1030,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "ThisRuleNeedsToStay",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/ThisRuleNeedsToStay"
    }
]
```

## <a name="convert-rule-set-to-powershell-objects"></a>Преобразование набора правил в объекты PowerShell

На этом шаге необходимо прочитать файл JSON, который создан ранее с помощью правил, которые должны быть включены в группу безопасности сети для этого примера.

```powershell
$nsgbaserules = Get-Content -Path C:\temp\testvm1-nsg.json | ConvertFrom-Json
```

## <a name="retrieve-network-watcher"></a>Извлечение Наблюдателя за сетями

Далее необходимо извлечь экземпляр Наблюдателя за сетями. Переменная `$networkWatcher` передается в командлет `AzNetworkWatcherSecurityGroupView`.

```powershell
$networkWatcher = Get-AzResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" } 
```

## <a name="get-a-vm"></a>Получение виртуальной машины

Для повторного выполнения командлета `Get-AzNetworkWatcherSecurityGroupView` необходима виртуальная машина. Ниже приведен пример получения объекта виртуальной машины.

```powershell
$VM = Get-AzVM -ResourceGroupName "testrg" -Name "testvm1"
```

## <a name="retrieve-security-group-view"></a>Получение представления группы безопасности

Далее необходимо получить результат представления группы безопасности. Этот результат сравнивается с "базовым" объектом JSON (см. выше).

```powershell
$secgroup = Get-AzNetworkWatcherSecurityGroupView -NetworkWatcher $networkWatcher -TargetVirtualMachineId $VM.Id
```

## <a name="analyzing-the-results"></a>Анализ результатов

Ответ группируется по сетевым интерфейсам. Разные типы возвращаемых правил, являются действительными правилами безопасности по умолчанию. Далее результат разбивается на основе особенностей применения: в подсети или на виртуальной сетевой карте.

Следующий сценарий PowerShell сравнивает результаты представления группы безопасности с существующими выходными данными группы NSG. Ниже приведен простой пример того, как можно сравнить результаты с помощью командлета `Compare-Object`.

```powershell
Compare-Object -ReferenceObject $nsgbaserules `
-DifferenceObject $secgroup.NetworkInterfaces[0].NetworkInterfaceSecurityRules `
-Property Name,Description,Protocol,SourcePortRange,DestinationPortRange,SourceAddressPrefix,DestinationAddressPrefix,Access,Priority,Direction
```

Пример ниже является результатом. Вы можете увидеть, что два правила, заданные в первом наборе правил не присутствовали в сравнении.

```
Name                     : My2ndRuleDoNotDelete
Description              : 
Protocol                 : *
SourcePortRange          : *
DestinationPortRange     : 112
SourceAddressPrefix      : *
DestinationAddressPrefix : *
Access                   : Allow
Priority                 : 1020
Direction                : Inbound
SideIndicator            : <=

Name                     : ThisRuleNeedsToStay
Description              : 
Protocol                 : TCP
SourcePortRange          : *
DestinationPortRange     : 5672
SourceAddressPrefix      : *
DestinationAddressPrefix : *
Access                   : Deny
Priority                 : 1030
Direction                : Inbound
SideIndicator            : <=
```

## <a name="next-steps"></a>Дальнейшие действия

Если параметры изменены, см. статью об [управлении группами безопасности сети с помощью портала](../virtual-network/manage-network-security-group.md). В ней содержатся сведения об отслеживании группы безопасности сети и нужных правил безопасности.