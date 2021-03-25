---
title: Развертывание Azure Веснного облака с Azure PowerShell
description: Развертывание Azure Веснного облака с Azure PowerShell
author: bmitchell287
ms.author: brendm
ms.topic: conceptual
ms.service: spring-cloud
ms.devlang: azurepowershell
ms.date: 11/16/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3cb320a37818084f2fbcad22a3cc992655b19c3d
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104878300"
---
# <a name="deploy-azure-spring-cloud-with-azure-powershell"></a>Развертывание Azure Spring Cloud с помощью Azure PowerShell

В этой статье описывается, как создать экземпляр Azure Веснного облака с помощью модуля PowerShell [AZ. спрингклауд](/powershell/module/Az.SpringCloud) .

## <a name="requirements"></a>Требования

* Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

[!INCLUDE [azure-powershell-requirements-no-header.md](../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Пока модуль PowerShell **AZ. спрингклауд** находится на этапе предварительной версии, его необходимо установить отдельно с помощью `Install-Module` командлета. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков модуля Az PowerShell и по умолчанию встроен в Azure Cloud Shell.

  ```azurepowershell-interactive
  Install-Module -Name Az.SpringCloud
  ```

* Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите требуемую подписку с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

  ```azurepowershell-interactive
  Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
  ```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

В следующем примере создается группа ресурсов с указанным именем в выбранном регионе.

```azurepowershell-interactive
New-AzResourceGroup -Name <resource group name> -Location eastus
```

## <a name="provision-a-new-instance-of-azure-spring-cloud"></a>Подготавливает новый экземпляр Azure Веснного облака

Чтобы создать новый экземпляр Azure Веснного облака, используйте командлет [New-азспрингклауд](/powershell/module/az.springcloud/new-azspringcloud) . В следующем примере создается облачная служба Azure весны с указанным именем в ранее созданной группе ресурсов.

```azurepowershell-interactive
New-AzSpringCloud -ResourceGroupName <resource group name> -name <service instance name> -Location eastus
```

## <a name="create-a-new-azure-spring-cloud-app"></a>Создание нового облачного приложения Azure весны

Чтобы создать новое приложение, используйте командлет [New-азспрингклаудапп](/powershell/module/az.springcloud/new-azspringcloudapp) . В следующем примере создается облачное приложение Azure весны с именем `gateway` .

```azurepowershell-interactive
New-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

## <a name="create-a-new-azure-spring-cloud-deployment"></a>Создание нового развертывания Azure весны в облаке

Чтобы создать новое развертывание, используйте командлет [New-азспрингклаудаппдеплоймент](/powershell/module/az.springcloud/new-azspringcloudappdeployment) . В следующем примере создается облачное развертывание Azure весны с именем `default` для `gateway` приложения.

```azurepowershell-interactive
New-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -Name <service instance name> -AppName gateway -DeploymentName default
```

## <a name="get-an-azure-spring-cloud-service"></a>Получение облачной службы Azure весны

Чтобы получить облачную службу Azure весны и ее свойства, используйте командлет [Get-азспрингклауд](/powershell/module/az.springcloud/get-azspringcloud) . В следующем примере извлекаются сведения об указанной облачной службе Azure весны.

```azurepowershell-interactive
Get-AzSpringCloud -ResourceGroupName <resource group name> -ServiceName <service instance name>
```

## <a name="get-an-azure-spring-cloud-app"></a>Получение облачного приложения Azure весны

Чтобы получить облачное приложение Azure весны и его свойства, используйте командлет [Get-азспрингклаудапп](/powershell/module/az.springcloud/get-azspringcloudapp) . В следующем примере извлекаются сведения о `gateway` приложении "Весна Cloud".

```azurepowershell-interactive
Get-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

## <a name="get-an-azure-spring-cloud-deployment"></a>Получение облачного развертывания Azure весны

Чтобы получить облачное развертывание Azure весны и его свойства, используйте командлет [Get-азспрингклаудаппдеплоймент](/powershell/module/az.springcloud/get-azspringcloudappdeployment) . В следующем примере извлекаются сведения о `default` пружинном развертывании в облаке.

```azurepowershell-interactive
Get-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway -DeploymentName default
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если созданные при работе с этой статьей ресурсы не нужны, их можно удалить с помощью команды из следующего примера.

### <a name="delete-an-azure-spring-cloud-deployment"></a>Удаление облачного развертывания Azure весны

Чтобы удалить облачное развертывание Azure весны, используйте командлет [Remove-азспрингклаудаппдеплоймент](/powershell/module/az.springcloud/remove-azspringcloudappdeployment) . В следующем примере удаляется развертывание облачного приложения Azure весны с именем `default` для указанной службы и приложения.

```azurepowershell-interactive
Remove-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway -DeploymentName default
```

### <a name="delete-an-azure-spring-cloud-app"></a>Удаление облачного приложения Azure весны

Чтобы удалить приложение с пружинным облаком, используйте командлет [Remove-азспрингклаудапп](/powershell/module/Az.SpringCloud/remove-azspringcloudapp) . В следующем примере приложение удаляется `gateway` из указанной службы и группы ресурсов.

```azurepowershell
Remove-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

### <a name="delete-an-azure-spring-cloud-service"></a>Удаление облачной службы Azure весны

Чтобы удалить облачную службу Azure весны, используйте командлет [Remove-азспрингклауд](/powershell/module/Az.SpringCloud/remove-azspringcloud) . Следующий пример удаляет указанную облачную службу Azure весны.

```azurepowershell
Remove-AzSpringCloud -ResourceGroupName <resource group name> -ServiceName <service instance name>
```

### <a name="delete-the-resource-group"></a>удаление группы ресурсов.

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этой статьи, они также будут удалены.

```azurepowershell-interactive
Remove-AzResourceGroup -Name <resource group name>
```

## <a name="next-steps"></a>Дальнейшие действия

[Ресурсы для разработчиков Azure весны в облаке](spring-cloud-resources.md).
