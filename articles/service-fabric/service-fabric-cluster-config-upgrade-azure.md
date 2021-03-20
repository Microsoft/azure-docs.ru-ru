---
title: Обновление конфигурации кластера Azure Service Fabric
description: Узнайте, как обновить конфигурацию, которая работает в кластере Service Fabric в Azure, с помощью шаблона Resource Manager.
ms.topic: conceptual
ms.date: 11/09/2018
ms.openlocfilehash: 692dc2162159ab61a3ac527e12dac43438084a60
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91842722"
---
# <a name="upgrade-the-configuration-of-a-cluster-in-azure"></a>Обновление конфигурации кластера в Azure 

В этой статье описывается, как настроить различные параметры структуры для кластера Service Fabric. Для кластеров, размещенных в Azure, можно настроить параметры на [портале Azure](https://portal.azure.com) или использовать шаблон Azure Resource Manager.

> [!NOTE]
> Не все параметры доступны на портале, поэтому [рекомендуется выполнять настройку с помощью шаблона Azure Resource Manager](./service-fabric-best-practices-infrastructure-as-code.md). Портал используется только для сценариев разработки и тестирования Service Fabric.
> 


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="customize-cluster-settings-using-resource-manager-templates"></a>Настройка параметров кластера с помощью шаблонов Resource Manager
Кластеры Azure можно настроить с помощью JSON-шаблона Resource Manager. Дополнительные сведения см. в статье о [параметрах конфигурации для кластеров](service-fabric-cluster-fabric-settings.md). В качестве примера ниже приведены инструкции, с помощью которых можно добавить новый параметр *MaxDiskQuotaInMB* в раздел *Diagnostics*, используя обозреватель ресурсов Azure.

1. Перейдите на страницу https://resources.azure.com.
2. Перейдите к своей подписке, развернув **подписки**  ->  **\<Your Subscription>**  ->  **resourceGroups**  ->  **\<Your Resource Group>**  ->  **providers**  ->  **Microsoft. ServiceFabric**  ->  **Clusters** . -> **\<Your Cluster Name>**
3. В правом верхнем углу выберите **чтение и запись.**
4. Выберите **изменить** и обновите `fabricSettings` элемент JSON и добавьте новый элемент:

```json
      {
        "name": "Diagnostics",
        "parameters": [
          {
            "name": "MaxDiskQuotaInMB",
            "value": "65536"
          }
        ]
      }
```

Вы также можете настроить параметры кластера с помощью Azure Resource Manager одним из следующих способов:

- Экспортируйте и обновите шаблон Resource Manager с помощью [портала Azure](../azure-resource-manager/templates/export-template-portal.md).
- Экспортируйте и обновите шаблон Resource Manager с помощью [PowerShell](../azure-resource-manager/management/manage-resources-powershell.md).
- Экспортируйте и обновите шаблон Resource Manager с помощью [Azure CLI](../azure-resource-manager/management/manage-resources-cli.md).
- Чтобы изменить параметр напрямую, используйте команды Azure PowerShell [Set-азсервицефабриксеттинг](/powershell/module/az.servicefabric/set-azservicefabricsetting) и [Remove-азсервицефабриксеттинг](/powershell/module/az.servicefabric/remove-azservicefabricsetting) .
- чтобы изменить параметр напрямую, используйте команды Azure CLI [az sf cluster setting](/cli/azure/sf/cluster/setting).

## <a name="next-steps"></a>Дальнейшие действия
* Изучите сведения о [параметрах кластера Service Fabric](service-fabric-cluster-fabric-settings.md).
* Ознакомьтесь с концепцией [масштабирования кластера](service-fabric-cluster-scale-in-out.md).
