---
title: Управление группами ресурсов — Azure PowerShell
description: Используйте Azure PowerShell для управления группами ресурсов с помощью Azure Resource Manager. Показывает, как создавать, перечислять и удалять группы ресурсов.
author: mumian
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: jgao
ms.openlocfilehash: fb8cd45842e2f9b64048aa939f7b6adc9b0e6e0a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98703688"
---
# <a name="manage-azure-resource-manager-resource-groups-by-using-azure-powershell"></a>Управление группами ресурсов Azure Resource Manager с помощью Azure PowerShell

Узнайте, как использовать Azure PowerShell с [Azure Resource Manager](overview.md) для управления группами ресурсов Azure. Сведения об управлении ресурсами Azure см. в статье [Управление ресурсами Azure с помощью Azure PowerShell](manage-resources-powershell.md).

Другие статьи об управлении группами ресурсов:

- [Управление группами ресурсов Azure с помощью портал Azure](manage-resources-portal.md)
- [Управление группами ресурсов Azure с помощью Azure CLI](manage-resources-cli.md)

## <a name="what-is-a-resource-group"></a>Что такое группа ресурсов

Группа ресурсов — это контейнер, содержащий связанные ресурсы для решения Azure. В группу ресурсов могут входить все ресурсы приложения или только те, которыми необходимо управлять совместно. Кроме того, пользователи могут выбрать оптимальный для своей организации способ распределения ресурсов в группах ресурсов. Как правило, ресурсы с общим жизненным циклом добавляют в одну и ту же группу ресурсов, чтобы их можно было легко развертывать, обновлять и удалять в виде группы.

В группе ресурсов хранятся метаданные о ресурсах. Таким образом, указывая расположение группы ресурсов, вы определяете расположение метаданных. В целях обеспечения соответствия необходимо убедиться, что данные хранятся в определенном регионе.

В группе ресурсов хранятся метаданные о ресурсах. Указывая расположение группы ресурсов, вы определяете расположение метаданных.

## <a name="create-resource-groups"></a>Создание группы ресурсов

Следующий сценарий PowerShell создает группу ресурсов.

```azurepowershell-interactive
New-AzResourceGroup -Name demoResourceGroup -Location westus
```

## <a name="list-resource-groups"></a>Перечислить группы ресурсов

Следующий сценарий PowerShell выводит список групп ресурсов в подписке.

```azurepowershell-interactive
Get-AzResourceGroup
```

Чтобы получить одну группу ресурсов, сделайте следующее:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Get-AzResourceGroup -Name $resourceGroupName
```

## <a name="delete-resource-groups"></a>Удаление групп ресурсов

Следующий сценарий PowerShell удаляет группу ресурсов:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Remove-AzResourceGroup -Name $resourceGroupName
```

Дополнительные сведения о том, как Azure Resource Manager упорядочивают удаление ресурсов, см. в разделе [Azure Resource Manager удаление группы ресурсов](delete-resource-group.md).

## <a name="deploy-resources-to-an-existing-resource-group"></a>Развертывание ресурсов в существующей группе ресурсов

См. раздел [развертывание ресурсов в существующей группе ресурсов](manage-resources-powershell.md#deploy-resources-to-an-existing-resource-group).

Сведения о проверке развертывания группы ресурсов см. в разделе [Test-азресаурцеграупдеплоймент](/powershell/module/Az.Resources/Test-AzResourceGroupDeployment).

## <a name="deploy-a-resource-group-and-resources"></a>Развертывание группы ресурсов и ресурсов

Вы можете создать группу ресурсов и развернуть ресурсы в группе с помощью шаблона диспетчер ресурсов. См. дополнительные сведения в разделе [Создание группы ресурсов и развертывание ресурсов](../templates/deploy-to-subscription.md#resource-groups).

## <a name="redeploy-when-deployment-fails"></a>Повторное развертывание при сбое развертывания

Эта функция также называется *откатом при ошибке*. Дополнительные сведения см. в разделе [Повторное развертывание при сбое развертывания](../templates/rollback-on-error.md).

## <a name="move-to-another-resource-group-or-subscription"></a>Переместить в другую группу ресурсов или подписку

Ресурсы в группе можно переместить в другую группу ресурсов. Дополнительные сведения см. в статье [Перемещение ресурсов в новую группу ресурсов или подписку](move-resource-group-and-subscription.md).

## <a name="lock-resource-groups"></a>Заблокировать группы ресурсов

Блокировка запрещает другим пользователям в организации случайно удалить или изменить критически важные ресурсы, такие как подписка Azure, Группа ресурсов или ресурс. 

Следующий скрипт блокирует группу ресурсов, чтобы группу ресурсов нельзя было удалить.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

New-AzResourceLock -LockName LockGroup -LockLevel CanNotDelete -ResourceGroupName $resourceGroupName 
```

Следующий скрипт получает все блокировки для группы ресурсов:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Get-AzResourceLock -ResourceGroupName $resourceGroupName 
```

Дополнительные сведения см. [в разделе Блокировка ресурсов с помощью Azure Resource Manager](lock-resources.md).

## <a name="tag-resource-groups"></a>Теги для групп ресурсов

К ресурсам и их группам можно добавлять теги. Это позволяет логически их упорядочивать. Дополнительные сведения см. [в статье Использование тегов для Организации ресурсов Azure](tag-resources.md#powershell).

## <a name="export-resource-groups-to-templates"></a>Экспорт групп ресурсов в шаблоны

После настройки группы ресурсов можно просмотреть шаблон диспетчер ресурсов для группы ресурсов. Экспорт шаблона обеспечивает два преимущества:

- Автоматизируйте будущие развертывания решения, поскольку шаблон содержит полную инфраструктуру.
- Познакомьтесь с синтаксисом шаблона, просмотрев нотация объектов JavaScript (JSON), который представляет ваше решение.

Чтобы экспортировать все ресурсы в группе ресурсов, используйте командлет [Export-азресаурцеграуп](/powershell/module/az.resources/Export-AzResourceGroup) и укажите имя группы ресурсов.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Export-AzResourceGroup -ResourceGroupName $resourceGroupName
```

Шаблон сохраняется как локальный файл.

Вместо экспорта всех ресурсов в группе ресурсов можно выбрать ресурсы для экспорта.

Чтобы экспортировать один ресурс, передайте этот идентификатор ресурса.

```azurepowershell-interactive
$resource = Get-AzResource `
  -ResourceGroupName <resource-group-name> `
  -ResourceName <resource-name> `
  -ResourceType <resource-type>
Export-AzResourceGroup `
  -ResourceGroupName <resource-group-name> `
  -Resource $resource.ResourceId
```

Чтобы экспортировать более одного ресурса, передайте идентификаторы ресурсов в массив.

```azurepowershell-interactive
Export-AzResourceGroup `
  -ResourceGroupName <resource-group-name> `
  -Resource @($resource1.ResourceId, $resource2.ResourceId)
```

При экспорте шаблона можно указать, используются ли параметры в шаблоне. По умолчанию параметры для имен ресурсов включены, но не имеют значения по умолчанию. Необходимо передать это значение параметра во время развертывания.

```json
"parameters": {
  "serverfarms_demoHostPlan_name": {
    "defaultValue": null,
    "type": "String"
  },
  "sites_webSite3bwt23ktvdo36_name": {
    "defaultValue": null,
    "type": "String"
  }
}
```

В ресурсе параметр используется для имени.

```json
"resources": [
  {
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2016-09-01",
    "name": "[parameters('serverfarms_demoHostPlan_name')]",
    ...
  }
]
```

При использовании `-IncludeParameterDefaultValue` параметра при экспорте шаблона параметр шаблона включает значение по умолчанию, для которого задано текущее значение. Можно либо использовать это значение по умолчанию, либо перезаписать значение по умолчанию, передав другое значение.

```json
"parameters": {
  "serverfarms_demoHostPlan_name": {
    "defaultValue": "demoHostPlan",
    "type": "String"
  },
  "sites_webSite3bwt23ktvdo36_name": {
    "defaultValue": "webSite3bwt23ktvdo36",
    "type": "String"
  }
}
```

При использовании `-SkipResourceNameParameterization` параметра при экспорте шаблона параметры для имен ресурсов не включаются в шаблон. Вместо этого имя ресурса задается непосредственно в ресурсе с текущим значением. Невозможно настроить имя во время развертывания.

```json
"resources": [
  {
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2016-09-01",
    "name": "demoHostPlan",
    ...
  }
]
```

Функция экспорта шаблона не поддерживает экспорт ресурсов фабрики данных Azure. Дополнительные сведения о том, как можно экспортировать ресурсы фабрики данных, см. в статье [копирование или клонирование фабрики данных в фабрике данных Azure](../../data-factory/copy-clone-data-factory.md).

Чтобы экспортировать ресурсы, созданные с помощью классической модели развертывания, необходимо [перенести их в модель развертывания Диспетчер ресурсов](../../virtual-machines/migration-classic-resource-manager-overview.md).

Дополнительные сведения см. [в разделе один и несколько ресурсов экспорт в шаблон в портал Azure](../templates/export-template-portal.md).

## <a name="manage-access-to-resource-groups"></a>Управление доступом к группам ресурсов

Управление [доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md) — это способ управления доступом к ресурсам в Azure. Дополнительные сведения см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о Azure Resource Manager см. в разделе [Общие сведения о Azure Resource Manager](overview.md).
- Сведения о синтаксисе шаблона диспетчер ресурсов см. в разделе [Общие сведения о структуре и синтаксисе шаблонов Azure Resource Manager](../templates/template-syntax.md).
- Дополнительные сведения о разработке шаблонов см. в пошаговых [руководствах](../index.yml).
- Сведения о том, как просмотреть схемы шаблонов Azure Resource Manager, см. в разделе [Справочник по шаблонам](/azure/templates/).