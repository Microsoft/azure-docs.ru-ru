---
title: Примеры для Azure PowerShell. Получение сведений об Облачной службе Azure (расширенная поддержка)
description: Примеры скриптов для получения сведений о развертывании Облачных служб Azure (расширенная поддержка)
ms.topic: sample
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: fc16c9e0d47cfe015f7ec5ceb766053635840abb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99475341"
---
# <a name="retrieve-information-about-your-azure-cloud-service-extended-support-deployments"></a>Получение сведений о развертываниях Облачной службы Azure (расширенная поддержка)

В этим примерах приведены различные способы для получения сведений о существующих развертываниях Облачной службы Azure (расширенная поддержка).

## <a name="get-all-cloud-services-under-a-resource-group"></a>Получение всех Облачных служб в группе ресурсов

```powershell
Get-AzCloudService -ResourceGroupName "ContosOrg"

ResourceGroupName Name              Location    ProvisioningState
----------------- ----              --------    -----------------
ContosOrg         ContosoCS         eastus2euap Succeeded
ContosOrg         ContosoCSTest     eastus2euap Failed
```

## <a name="get-single-cloud-service"></a>Получение отдельной Облачной службы
```powershell
Get-AzCloudService -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS"

ResourceGroupName Name              Location    ProvisioningState
----------------- ----              --------    -----------------
ContosOrg         ContosoCS         eastus2euap Succeeded

PS C:\> $cloudService = Get-AzCloudService -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS"
PS C:\> $cloudService | Format-List
ResourceGroupName : ContosOrg
Configuration     : xxxxxxxx
ConfigurationUrl  :
ExtensionProfile  : xxxxxxxx
Id                : xxxxxxxx
Location          : East US
Name              : ContosoCS
NetworkProfile    : xxxxxxxx
OSProfile         : xxxxxxxx
PackageUrl        : xxxxxxxx
ProvisioningState : Succeeded
RoleProfile       : xxxxxxxx
StartCloudService :
Tag               : {
                      "Owner": "Contos"
                    }
Type              : Microsoft.Compute/cloudServices
UniqueId          : xxxxxxxx
UpgradeMode       : Auto
```

## <a name="get-cloud-service-instance-view"></a>Получение представления экземпляра Облачной службы
```powershell
PS C:\>$cloudServiceInstanceView = Get-AzCloudServiceInstanceView -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS"

PS C:\>$cloudServiceInstanceView
RoleInstanceStatusesSummary                                   Statuses
---------------------------                                   --------
{{ProvisioningState/succeeded : 4}, {PowerState/started : 4}} {Provisioning succeeded, Started, Current Upgrade Domain of cloud service is -1., Max Upgrade Domain of cloud service is 1.}

PS C:\>$cloudServiceInstanceView.ToJsonString()
{
  "roleInstance": {
    "statusesSummary": [
      {
        "code": "ProvisioningState/succeeded",
        "count": 4
      },
      {
        "code": "PowerState/started",
        "count": 4
      }
    ]
  },
  "statuses": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "time": "2020-10-28T13:26:48.8109686Z"
    },
    {
      "code": "PowerState/started",
      "displayStatus": "Started",
      "level": "Info",
      "time": "2020-10-28T13:26:48.8109686Z"
    },
    {
      "code": "CurrentUpgradeDomain/-1",
      "displayStatus": "Current Upgrade Domain of cloud service is -1.",
      "level": "Info"
    },
    {
      "code": "MaxUpgradeDomain/1",
      "displayStatus": "Max Upgrade Domain of cloud service is 1.",
      "level": "Info"
    }
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об Облачных службах Azure (расширенная поддержка) см. в статье [Общие сведения об Облачных службах Azure (расширенная поддержка)](overview.md).
- Перейдите в [репозиторий примеров для Облачных служб (расширенная поддержка) ](https://github.com/Azure-Samples/cloud-services-extended-support)
