---
title: Защита зоны DNS и записей — Azure DNS
description: В этой схеме обучения приступите к защите зон и наборов записей DNS в Microsoft Azure DNS.
services: dns
author: asudbring
ms.service: dns
ms.topic: how-to
ms.date: 2/20/2020
ms.author: allensu
ms.openlocfilehash: 2280d6243f468708269569cd24cb8c7a3e2a8191
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102202303"
---
# <a name="how-to-protect-dns-zones-and-records"></a>Как защитить зоны и записи DNS

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Зоны и записи DNS являются критически важными ресурсами. Удаление зоны DNS или отдельной записи DNS может привести к сбою службы. Важно, чтобы зоны и записи DNS были защищены от несанкционированных или случайных изменений.

В этой статье объясняется, как Azure DNS позволяет защищать частные зоны и записи DNS для таких изменений.  Мы применяем две эффективные функции для ценных бумаг, предоставляемые Azure Resource Manager: [Управление доступом на основе ролей Azure (Azure RBAC)](../role-based-access-control/overview.md) и [блокировки ресурсов](../azure-resource-manager/management/lock-resources.md).

## <a name="azure-role-based-access-control"></a>Управление доступом на основе ролей в Azure

Управление доступом на основе ролей Azure (Azure RBAC) обеспечивает точное управление доступом для пользователей, групп и ресурсов Azure. С помощью Azure RBAC можно предоставить пользователям необходимый уровень доступа. Дополнительные сведения об управлении доступом с помощью Azure RBAC см. в статье [что такое управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md).

### <a name="the-dns-zone-contributor-role"></a>Роль "Участник зоны DNS"

Роль участника зоны DNS — это встроенная роль для управления частными ресурсами DNS. Эта роль, примененная к пользователю или группе, позволяет им управлять ресурсами DNS.

Группа ресурсов *myResourceGroup* содержит пять зон для Contoso Corporation. Если предоставить администратору DNS разрешения "Участник зоны DNS" для этой группы ресурсов, то он получит полный контроль над этими зонами DNS. Это позволяет избежать предоставления ненужных разрешений. Администратор DNS не может создавать или прекращать работу виртуальных машин.

Самый простой способ назначения разрешений Azure RBAC — [с помощью портал Azure](../role-based-access-control/role-assignments-portal.md).  

Откройте **элемент управления доступом (IAM)** для группы ресурсов, затем выберите **Добавить**, а затем роль **участник зоны DNS** . Выберите необходимых пользователей или группы для предоставления разрешений.

![Azure RBAC уровня группы ресурсов через портал Azure](./media/dns-protect-zones-recordsets/rbac1.png)

Разрешения также можно [предоставить с помощью Azure PowerShell](../role-based-access-control/role-assignments-powershell.md):

```azurepowershell
# Grant 'DNS Zone Contributor' permissions to all zones in a resource group

$usr = "<user email address>"
$rol = "DNS Zone Contributor"
$rsg = "<resource group name>"

New-AzRoleAssignment -SignInName $usr -RoleDefinitionName $rol -ResourceGroupName $rsg
```

Аналогичную команду также [можно выполнить через интерфейс командной строки Azure](../role-based-access-control/role-assignments-cli.md):

```azurecli
# Grant 'DNS Zone Contributor' permissions to all zones in a resource group

az role assignment create \
--assignee "<user email address>" \
--role "DNS Zone Contributor" \
--resource-group "<resource group name>"
```

### <a name="zone-level-azure-rbac"></a>Azure RBAC уровня зоны

Правила RBAC Azure могут применяться к подписке, группе ресурсов или к отдельному ресурсу. Этот ресурс может быть отдельной зоной DNS или отдельным набором записей.

Например, Группа ресурсов *myResourceGroup* содержит зону *contoso.com* и подзону *Customers.contoso.com*. Записи CNAME создаются для каждой учетной записи клиента. Учетной записи администратора, используемой для управления записями CNAME, присваиваются разрешения на создание записей в зоне *Customers.contoso.com* . Учетная запись может управлять только *Customers.contoso.com* .

Разрешения RBAC уровня зоны Azure можно предоставить с помощью портал Azure.  Откройте **элемент управления доступом (IAM)** для зоны, выберите **Добавить**, а затем роль **участника зоны DNS** и выберите пользователей или группы, которым требуется предоставить разрешения.

![Azure RBAC уровня зоны DNS через портал Azure](./media/dns-protect-zones-recordsets/rbac2.png)

Разрешения также можно [предоставить с помощью Azure PowerShell](../role-based-access-control/role-assignments-powershell.md):

```azurepowershell
# Grant 'DNS Zone Contributor' permissions to a specific zone

$usr = "<user email address>"
$rol = "DNS Zone Contributor"
$rsg = "<resource group name>"
$zon = "<zone name>"
$typ = "Microsoft.Network/DNSZones"

New-AzRoleAssignment -SignInName $usr -RoleDefinitionName $rol -ResourceGroupName $rsg -ResourceName $zon -ResourceType $typ
```

Аналогичную команду также [можно выполнить через интерфейс командной строки Azure](../role-based-access-control/role-assignments-cli.md):

```azurecli
# Grant 'DNS Zone Contributor' permissions to a specific zone

az role assignment create \
--assignee <user email address> \
--role "DNS Zone Contributor" \
--scope "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/DnsZones/<zone name>/"
```

### <a name="record-set-level-azure-rbac"></a>Azure RBAC на уровне набора записей

Разрешения применяются на уровне набора записей.  Пользователю предоставляется доступ к записям, которые им нужны, и не могут вносить какие-либо другие изменения.

Разрешения RBAC на уровне набора записей можно настроить с помощью портал Azure, используя кнопку **управления доступом (IAM)** на странице "набор записей":

![На уровне набора записей Azure RBAC через портал Azure](./media/dns-protect-zones-recordsets/rbac3.png)

Разрешения RBAC на уровне набора записей также можно [предоставлять с помощью Azure PowerShell](../role-based-access-control/role-assignments-powershell.md):

```azurepowershell
# Grant permissions to a specific record set

$usr = "<user email address>"
$rol = "DNS Zone Contributor"
$sco = 
"/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/dnszones/<zone name>/<record type>/<record name>"

New-AzRoleAssignment -SignInName $usr -RoleDefinitionName $rol -Scope $sco
```

Аналогичную команду также [можно выполнить через интерфейс командной строки Azure](../role-based-access-control/role-assignments-cli.md):

```azurecli
# Grant permissions to a specific record set

az role assignment create \
--assignee "<user email address>" \
--role "DNS Zone Contributor" \
--scope "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/dnszones/<zone name>/<record type>/<record name>"
```

### <a name="custom-roles"></a>Пользовательские роли

Встроенная роль "Участник зоны DNS" разрешает полный контроль над ресурсом DNS. Можно создать собственные пользовательские роли Azure, чтобы обеспечить детальный контроль.

Учетной записи, используемой для управления CNAME, предоставляется разрешение на управление только записями CNAME. Учетная запись не может изменять записи других типов. Учетная запись не может выполнять операции на уровне зоны, такие как удаление зоны.

В следующем примере показано определение пользовательской роли для управления только записями CNAME:

```json
{
    "Name": "DNS CNAME Contributor",
    "Id": "",
    "IsCustom": true,
    "Description": "Can manage DNS CNAME records only.",
    "Actions": [
        "Microsoft.Network/dnsZones/CNAME/*",
        "Microsoft.Network/dnsZones/read",
        "Microsoft.Authorization/*/read",
        "Microsoft.Insights/alertRules/*",
        "Microsoft.ResourceHealth/availabilityStatuses/read",
        "Microsoft.Resources/deployments/*",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.Support/*"
    ],
    "NotActions": [
    ],
    "AssignableScopes": [
        "/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e"
    ]
}
```

Свойство Actions определяет следующие разрешения для DNS:

* `Microsoft.Network/dnsZones/CNAME/*` предоставляет полный контроль над записями CNAME.
* `Microsoft.Network/dnsZones/read` предоставляет разрешение на чтение зон DNS, но не позволяет их изменять. Таким образом вы можете видеть зону, в которой создается запись CNAME.

Остальные элементы свойства Actions копируются из [встроенной роли участника зоны DNS](../role-based-access-control/built-in-roles.md#dns-zone-contributor).

> [!NOTE]
> Использование настраиваемой роли Azure для предотвращения удаления наборов записей с тем, чтобы разрешить их обновление, не является эффективным элементом управления. Наборы записей нельзя удалить, но ничто не препятствует их изменению.  К разрешенным изменениям относятся добавление и удаление записей из набора записей, включая удаление всех записей (при этом остается "пустой" набор записей). Это действует так же, как удаление набора записей с точки зрения разрешения DNS.

В настоящее время невозможно определить пользовательские определения ролей с помощью портал Azure. Пользовательскую роль на основе этого определения роли можно создать с помощью Azure PowerShell:

```azurepowershell
# Create new role definition based on input file
New-AzRoleDefinition -InputFile <file path>
```

Ее также можно создать с помощью интерфейса командной строки Azure:

```azurecli
# Create new role definition based on input file
az role create -inputfile <file path>
```

Затем роль можно назначить таким же образом, как и встроенные роли. Этот процесс описан ранее в этой статье.

Дополнительные сведения о создании настраиваемых ролей, управлении ими и их назначении см. в статье [пользовательские роли Azure](../role-based-access-control/custom-roles.md).

## <a name="resource-locks"></a>Блокировки ресурсов

Azure Resource Manager поддерживает другой тип управления безопасностью, возможность блокировки ресурсов. Блокировки ресурсов применяются к ресурсу и действуют для всех пользователей и ролей. Дополнительные сведения см. [в разделе Блокировка ресурсов с помощью Azure Resource Manager](../azure-resource-manager/management/lock-resources.md).

Существует два типа блокировки ресурсов: **CanNotDelete** и **ReadOnly**. Эти типы блокировки можно применять либо к Частная зона DNSной зоне, либо к отдельному набору записей. Следующие разделы описывают несколько распространенных сценариев и способы их поддержки с помощью блокировок ресурсов.

### <a name="protecting-against-all-changes"></a>Защита от любых изменений

Чтобы предотвратить внесенные изменения, примените к зоне блокировку только для чтения. Эта блокировка предотвращает создание новых наборов записей и изменение или удаление существующих наборов записей.

Блокировки ресурсов на уровне зоны можно создавать через портал Azure.  На странице зоны DNS выберите **Блокировки**, затем щелкните **+ Добавить**:

![Блокировка ресурсов на уровне зоны через портал Azure](./media/dns-protect-zones-recordsets/locks1.png)

Блокировки ресурсов на уровне зоны также можно создавать с помощью [Azure PowerShell](/powershell/module/az.resources/new-azresourcelock?view=latest):

```azurepowershell
# Lock a DNS zone

$lvl = "<lock level>"
$lnm = "<lock name>"
$rsc = "<zone name>"
$rty = "Microsoft.Network/DNSZones"
$rsg = "<resource group name>"

New-AzResourceLock -LockLevel $lvl -LockName $lnm -ResourceName $rsc -ResourceType $rty -ResourceGroupName $rsg
```

Аналогичную команду также [можно выполнить через интерфейс командной строки Azure](/cli/azure/lock#az-lock-create):

```azurecli
# Lock a DNS zone

az lock create \
--lock-type "<lock level>" \
--name "<lock name>" \
--resource-name "<zone name>" \
--namespace "Microsoft.Network" \
--resource-type "DnsZones" \
--resource-group "<resource group name>"
```

### <a name="protecting-individual-records"></a>Защита отдельных записей

Чтобы предотвратить внесение изменений в существующий набор записей DNS, примените к нему блокировку ReadOnly.

> [!NOTE]
> Применение к набору записей блокировки CanNotDelete (Запрет удаления) не является эффективным. Набор записей нельзя удалить, но ничто не препятствует его изменению.  К разрешенным изменениям относятся добавление и удаление записей из набора записей, включая удаление всех записей (при этом остается "пустой" набор записей). Это действует так же, как удаление набора записей с точки зрения разрешения DNS.

В настоящее время блокировки ресурсов на уровне наборов записей можно настраивать только с помощью Azure PowerShell.  Они не поддерживаются в портал Azure или Azure CLI.

```azurepowershell
# Lock a DNS record set

$lvl = "<lock level>"
$lnm = "<lock name>"
$rsc = "<zone name>/<record set name>"
$rty = "Microsoft.Network/DNSZones/<record type>"
$rsg = "<resource group name>"

New-AzResourceLock -LockLevel $lvl -LockName $lnm -ResourceName $rsc -ResourceType $rty -ResourceGroupName $rsg
```

### <a name="protecting-against-zone-deletion"></a>Защита зоны от удаления

При удалении зоны в Azure DNS все наборы записей в зоне удаляются.  Эта операция не может быть отменена. Случайное удаление критически важной зоны может оказать серьезное влияние на коммерческую деятельность.  Важно защититься от случайного удаления зоны.

Применение блокировки CanNotDelete (Запрет удаления) предотвращает удаление зоны. Блокировки наследуются дочерними ресурсами. Блокировка предотвращает удаление наборов записей в зоне. Как описано в заметке выше, это неэффективно, так как записи по-прежнему могут быть удалены из существующих наборов записей.

В качестве альтернативы примените блокировку CanNotDelete к набору записей в зоне, например к набору записей SOA. Зона не удаляется, и наборы записей также не удаляются. Эта блокировка защищает от удаления зоны, а также позволяет свободно изменять наборы записей в зоне. При попытке удалить зону Azure Resource Manager обнаруживает это удаление. Удаление также приведет к удалению набора записей SOA, Azure Resource Manager блокирует вызов, так как SOA заблокирован.  Наборы записей не будут удалены.

Следующая команда PowerShell создает блокировку CanNotDelete (Запрет удаления) для записи типа SOA в указанной зоне:

```azurepowershell
# Protect against zone delete with CanNotDelete lock on the record set

$lvl = "CanNotDelete"
$lnm = "<lock name>"
$rsc = "<zone name>/@"
$rty = "Microsoft.Network/DNSZones/SOA"
$rsg = "<resource group name>"

New-AzResourceLock -LockLevel $lvl -LockName $lnm -ResourceName $rsc -ResourceType $rty -ResourceGroupName $rsg
```

Другим вариантом предотвращения случайного удаления зоны является использование настраиваемой роли. Эта роль гарантирует, что учетные записи, используемые для управления зонами, не имеют разрешений на удаление зоны. 

При необходимости удаления зоны можно выполнить два шага удаления:

 - Сначала предоставьте разрешения на удаление зоны
 - Во вторых, предоставьте разрешения на удаление зоны.

Настраиваемая роль работает для всех зон, к которым обращаются эти учетные записи. Учетные записи с разрешениями на удаление зоны, такие как владелец подписки, по-прежнему могут случайно удалить зону.

В то же время можно одновременно использовать как блокировки ресурсов, так и пользовательские роли в качестве подхода к защите зоны DNS.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о работе с Azure RBAC см. в статье [что такое управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md).
* Дополнительные сведения о работе с блокировками ресурсов см. в статье [Блокировка ресурсов с помощью диспетчера ресурсов Azure](../azure-resource-manager/management/lock-resources.md).
