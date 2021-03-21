---
title: Создание правила Azure Load Balancer для кластера
description: Настройте Azure Load Balancer, чтобы открыть порты для кластера Azure Service Fabric.
ms.topic: conceptual
ms.date: 12/06/2017
ms.openlocfilehash: 7e09c7b0b3e2bfa5a5ff834e243f5098cbbd947b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92319899"
---
# <a name="open-ports-for-a-service-fabric-cluster"></a>Открытие портов для кластера Service Fabric

Подсистема балансировки нагрузки, развернутая вместе с кластером Azure Service Fabric, направляет трафик в ваше приложение, работающее на узле. Если изменить настройки приложения, чтобы использовать другой порт, необходимо предоставить этот порт в Azure Load Balancer (или направлять трафик через другой порт).

Подсистема балансировки нагрузки автоматически создается при развертывании кластера Service Fabric в Azure. Если у вас нет подсистемы балансировки нагрузки, ознакомьтесь с разделом [Создание балансировщика нагрузки для Интернета на портале Azure](../load-balancer/quickstart-load-balancer-standard-public-portal.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="configure-service-fabric"></a>Настройка Service Fabric

Файл конфигурации **ServiceManifest.xml** приложения Service Fabric определяет конечные точки, которые будет использовать приложение. После обновления файла конфигурации для определения конечной точки необходимо обновить подсистему балансировки нагрузки, чтобы предоставить этот (или другой) порт. Дополнительные сведения о том, как создать конечную точку Service Fabric, см. в разделе [Указание ресурсов в манифесте службы](service-fabric-service-manifest-resources.md).

## <a name="create-a-load-balancer-rule"></a>Создание правила балансировщика нагрузки

Правило Load Balancer открывает порт с доступом к Интернету и перенаправляет трафик на порт внутреннего узла, используемый вашим приложением. Если у вас нет подсистемы балансировки нагрузки, ознакомьтесь с разделом [Создание балансировщика нагрузки для Интернета на портале Azure](../load-balancer/quickstart-load-balancer-standard-public-portal.md).

Чтобы создать правило Load Balancer, необходимо собрать следующие сведения:

- имя подсистемы балансировки нагрузки;
- группа ресурсов подсистемы балансировки нагрузки и кластера Service Fabric;
- внешний порт;
- внутренний порт.

## <a name="azure-cli"></a>Azure CLI
Для создания правила подсистемы балансировки нагрузки требуется всего одна команда **Azure CLI**. Чтобы создать правило, необходимо знать имя подсистемы балансировки нагрузки и имя группы ресурсов.

>[!NOTE]
>Если требуется определить имя подсистемы балансировки нагрузки, выполните приведенную команду, чтобы быстро получить список всех подсистем балансировки нагрузки и связанных групп ресурсов.
>
>`az network lb list --query "[].{ResourceGroup: resourceGroup, Name: name}"`
>


```azurecli
az network lb rule create --backend-port 40000 --frontend-port 39999 --protocol Tcp --lb-name LB-svcfab3 -g svcfab_cli -n my-app-rule
```

В команде Azure CLI используется несколько параметров, которые описаны в следующей таблице.

| Параметр | Описание |
| --------- | ----------- |
| `--backend-port`  | Порт, через который приложение Service Fabric ожидает передачи данных. |
| `--frontend-port` | Порт, который подсистема балансировки нагрузки предоставляет для внешних подключений. |
| `-lb-name` | Имя изменяемой подсистемы балансировки нагрузки. |
| `-g`       | Группа ресурсов, содержащая подсистему балансировки нагрузки и кластер Service Fabric. |
| `-n`       | Выбранное имя правила. |


>[!NOTE]
>Дополнительные сведения о создании подсистемы балансировки нагрузки с помощью Azure CLI см. в разделе [Создание балансировщика нагрузки для Интернета с помощью Azure CLI](../load-balancer/quickstart-load-balancer-standard-internal-cli.md).

## <a name="powershell"></a>PowerShell

PowerShell немного сложнее в использовании, чем Azure CLI. Чтобы создать правило, сделайте следующее:

1. Получите подсистему балансировки нагрузки из Azure.
2. Создайте правило.
3. Добавьте в подсистему балансировки нагрузки правило.
4. Обновите подсистему балансировки нагрузки.

>[!NOTE]
>Если требуется определить имя подсистемы балансировки нагрузки, выполните приведенную команду, чтобы быстро получить список всех подсистем балансировки нагрузки и связанных групп ресурсов.
>
>`Get-AzLoadBalancer | Select Name, ResourceGroupName`

```powershell
# Get the load balancer
$lb = Get-AzLoadBalancer -Name LB-svcfab3 -ResourceGroupName svcfab_cli

# Create the rule based on information from the load balancer.
$lbrule = New-AzLoadBalancerRuleConfig -Name my-app-rule7 -Protocol Tcp -FrontendPort 39990 -BackendPort 40009 `
                                            -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
                                            -BackendAddressPool  $lb.BackendAddressPools[0] `
                                            -Probe $lb.Probes[0]

# Add the rule to the load balancer
$lb.LoadBalancingRules.Add($lbrule)

# Update the load balancer on Azure
$lb | Set-AzLoadBalancer
```

В команде `New-AzLoadBalancerRuleConfig``-FrontendPort` представляет порт, который подсистема балансировки нагрузки предоставляет для внешних подключений, а `-BackendPort` представляет порт, через который приложение Service Fabric ожидает передачи данных.

>[!NOTE]
>Дополнительные сведения о создании подсистемы балансировки нагрузки с помощью PowerShell см. в разделе [Создание балансировщика нагрузки для Интернета в Resource Manager с помощью PowerShell](../load-balancer/quickstart-load-balancer-standard-internal-powershell.md).

## <a name="next-steps"></a>Дальнейшие действия

См. дополнительные сведения о [параметрах сети в Service Fabric](service-fabric-patterns-networking.md).