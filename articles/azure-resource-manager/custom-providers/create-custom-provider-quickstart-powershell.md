---
title: Создание настраиваемого поставщика ресурсов Azure с помощью Azure PowerShell
description: Здесь описано, как создать настраиваемый поставщик ресурсов Azure с помощью Azure PowerShell
author: MSEvanhi
ms.author: evanhi
ms.date: 09/22/2020
ms.topic: quickstart
ms.devlang: azurepowershell
ms.custom:
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 9c19eb41210b6fba1935a0d158c8240375f4f8f5
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533913"
---
# <a name="quickstart-create-an-azure-custom-resource-provider-with-azure-powershell"></a>Краткое руководство. Создание настраиваемого поставщика ресурсов Azure с помощью Azure PowerShell

Из этого краткого руководства вы узнаете, как создать собственный настраиваемый поставщик ресурсов Azure с помощью модуля PowerShell [Az.CustomProviders](/powershell/module/az.customproviders).

> [!CAUTION]
> В настоящее время настраиваемые поставщики Azure находятся в общедоступной предварительной версии. Эта предварительная версия предоставляется без соглашения об уровне обслуживания. Эта версия не рекомендуется для использования с рабочими нагрузками в производственной среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если вы решили использовать Cloud Shell, дополнительные сведения см. в статье [Обзор Azure Cloud Shell](../../cloud-shell/overview.md).

> [!IMPORTANT]
> Так как модуль PowerShell **Az.CustomProviders** предоставляется в режиме предварительной версии, его нужно установить отдельно с помощью командлета `Install-Module`. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.CustomProviders
```

Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите требуемую подписку с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов Azure](../../azure-resource-manager/management/overview.md) с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

В следующем примере создается группа ресурсов с указанным именем в выбранном регионе.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location westus2
```

## <a name="create-a-custom-resource-provider"></a>Создание настраиваемого поставщика ресурсов

Чтобы создать или обновить настраиваемый поставщик ресурсов, используйте командлет [New-AzCustomProvider](/powershell/module/az.customproviders/new-azcustomprovider), как показано в следующем примере.

```azurepowershell-interactive
New-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type -Location westus2 -ResourceType @{Name='CustomRoute1'; Endpoint='https://www.contoso.com/'}
```

## <a name="get-the-custom-resource-provider-manifest"></a>Получение манифеста настраиваемого поставщика ресурсов

Чтобы получить сведения о манифесте настраиваемого поставщика ресурсов, используйте командлет [Get-AzCustomProvider](/powershell/module/az.customproviders/get-azcustomprovider), как показано в следующем примере.

```azurepowershell-interactive
Get-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type | Format-List
```

## <a name="create-an-association"></a>Создание ассоциации

Чтобы создать или обновить ассоциацию, используйте командлет [New-AzCustomProviderAssociation](/powershell/module/az.customproviders/new-azcustomproviderassociation), как показано в следующем примере.

```azurepowershell-interactive
$provider = Get-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type
New-AzCustomProviderAssociation -Scope $resourceId -Name MyAssoc -TargetResourceId $provider.Id
```

## <a name="get-an-association"></a>Получение ассоциации

Чтобы получить сведения об ассоциации, используйте командлет [Get-AzCustomProviderAssociation](/powershell/module/az.customproviders/get-azcustomproviderassociation), как показано в следующем примере.

```azurepowershell-interactive
Get-AzCustomProviderAssociation -Scope $resourceId -Name MyAssoc
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если созданные при работе с этой статьей ресурсы не нужны, их можно удалить с помощью команды из следующего примера.

### <a name="delete-an-association"></a>Удаление ассоциации

Чтобы удалить ассоциацию, используйте командлет [Remove-AzCustomProviderAssociation](/powershell/module/az.customproviders/remove-azcustomproviderassociation). В следующем примере удаляется ассоциация.

```azurepowershell
Remove-AzCustomProviderAssociation -Scope $id -Name Namespace.Type
```

### <a name="delete-a-custom-resource-provider"></a>Удаление настраиваемого поставщика ресурсов

Чтобы удалить настраиваемый поставщик ресурсов, используйте командлет [Remove-AzCustomProvider](/powershell/module/az.customproviders/remove-azcustomprovider). В следующем примере удаляется настраиваемый поставщик ресурсов.

```azurepowershell-interactive
Remove-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type
```

### <a name="delete-the-resource-group"></a>удаление группы ресурсов.

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этой статьи, они также будут удалены.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о настраиваемых поставщиках ресурсов Azure см. [здесь](overview.md).
