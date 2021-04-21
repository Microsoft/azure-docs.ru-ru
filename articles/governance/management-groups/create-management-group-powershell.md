---
title: Краткое руководство. Создание группы управления с помощью Azure PowerShell
description: В этом кратком руководстве показано, как с помощью Azure PowerShell создать группу управления для организации ресурсов в иерархию ресурсов.
ms.date: 02/05/2021
ms.topic: quickstart
ms.custom:
- devx-track-azurepowershell
- mode-api
ms.openlocfilehash: 0291bb2bfb439ad09531066f6bad4e20a3f4c6bd
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107535971"
---
# <a name="quickstart-create-a-management-group-with-azure-powershell"></a>Краткое руководство. Создание группы управления с помощью Azure PowerShell

Группы управления — это контейнеры, которые помогают управлять доступом, политикой и соответствием требованиям в нескольких подписках. Создание таких контейнеров позволяет построить эффективную и экономную иерархию, которую можно использовать с [политикой Azure](../policy/overview.md) и [элементами управления доступом на основе ролей Azure](../../role-based-access-control/overview.md). Дополнительные сведения о группах управления см. в статье [Упорядочение ресурсов с помощью групп управления Azure](overview.md).

Создание первой группы управления в каталоге может занять до 15 минут. Существуют процессы, выполняемые в первый раз при настройке службы групп управления в Azure для вашего каталога. По завершении процесса вы получите уведомление. См. сведения о [начальной настройке групп управления](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Перед началом работы убедитесь, что установлена последняя версия Azure PowerShell. Дополнительные сведения см. в статье [Install the Azure PowerShell module](/powershell/azure/install-az-ps) (Установка модуля Azure PowerShell).

- Любой пользователь Azure AD в арендаторе может создать группу управления без разрешения на запись для группы управления, назначенного этому пользователю, если [защита иерархии](./how-to/protect-resource-hierarchy.md#setting---require-authorization) не включена. Эта новая группа управления становится дочерней по отношению к корневой группе управления или [группе управления по умолчанию](./how-to/protect-resource-hierarchy.md#setting---default-management-group), а ее создателю назначается роль "Владелец". Служба группы управления обеспечивает эту возможность, чтобы назначения ролей не требовались на корневом уровне. У пользователей нет доступа к корневой группе управления при ее создании. Чтобы избежать трудностей при поиске глобальных администраторов Azure AD для настройки работы с группами управления, мы разрешаем создавать исходные группы управления на корневом уровне.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

### <a name="create-in-azure-powershell"></a>Создание с помощью Azure PowerShell

Создайте новую группу управления в PowerShell с помощью командлета [New-AzManagementGroup](/powershell/module/az.resources/new-azmanagementgroup). В нашем примере группа управления **GroupName** — это _Contoso_.

```azurepowershell-interactive
New-AzManagementGroup -GroupName 'Contoso'
```

**GroupName** — уникальный создаваемый идентификатор. Этот идентификатор используется в других командах для обращения к этой группе. Его нельзя изменить позже.

Если нужно, чтобы для группы управления на портале отображалось другое имя, добавьте параметр **DisplayName**. Например, чтобы создать группу управления с именем Contoso и отображаемым именем Contoso Group, используйте следующий командлет:

```azurepowershell-interactive
New-AzManagementGroup -GroupName 'Contoso' -DisplayName 'Contoso Group'
```

В приведенных выше примерах новая группа управления создается в корневой группе управления. Чтобы указать другую группу управления в качестве родительской, используйте параметр **ParentId**.

```azurepowershell-interactive
$parentGroup = Get-AzManagementGroup -GroupName Contoso
New-AzManagementGroup -GroupName 'ContosoSubGroup' -ParentId $parentGroup.id
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить созданную ранее группу управления, используйте командлет [Remove-AzManagementGroup](/powershell/module/az.resources/remove-azmanagementgroup):

```azurepowershell-interactive
Remove-AzManagementGroup -GroupName 'Contoso'
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать группу управления для организации иерархии ресурсов. Группа управления может содержать подписки и другие группы управления.

Чтобы узнать больше о группах управления и управлении иерархией ресурсов, см. следующее руководство:

> [!div class="nextstepaction"]
> [Управление ресурсами с помощью групп управления](./manage.md)
