---
title: Руководство. Управление виртуальными машинами с помощью CLI
description: Из этого руководства вы узнаете, как управлять виртуальными машинами Azure с применением Azure RBAC, политик, блокировок и тегов с помощью Azure CLI.
services: virtual-machines
documentationcenter: virtual-machines
author: tfitzmac
manager: gwallace
ms.service: virtual-machines
ms.workload: infrastructure
ms.tgt_pltfrm: vm-linux
ms.topic: tutorial
ms.date: 09/30/2019
ms.author: tomfitz
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: c724f41172ef675f94139cea0ed77fc0232b126b
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102561795"
---
# <a name="tutorial-learn-about-linux-virtual-machine-management-with-azure-cli"></a>Руководство по Управление виртуальными машинами Linux с помощью Azure CLI

[!INCLUDE [Resource Manager governance introduction](../../../includes/resource-manager-governance-intro.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим учебником требуется Azure CLI версии 2.0.30 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="understand-scope"></a>Общие сведения об области

[!INCLUDE [Resource Manager governance scope](../../../includes/resource-manager-governance-scope.md)]

В этом руководстве к группе ресурсов применяются все параметры управления, поэтому вы сможете легко отменить их по завершении работы.

Создайте группу ресурсов.

```azurecli-interactive
az group create --name myResourceGroup --location "East US"
```

Сейчас группа ресурсов пуста.

## <a name="azure-role-based-access-control"></a>Управление доступом на основе ролей в Azure

Вам нужно, чтобы у пользователей вашей организации был необходимый уровень доступа к этим ресурсам. Вы не хотите предоставлять пользователям неограниченный доступ, но при этом требуется обеспечить им возможность работать. [Управление доступом Azure на основе ролей (Azure RBAC)](../../role-based-access-control/overview.md) позволяет предоставлять пользователям разрешения на выполнение определенных действий в той или иной области.

Для создания и удаления назначений ролей пользователи должны иметь доступ `Microsoft.Authorization/roleAssignments/*`. Такой доступ предоставляется с помощью ролей владельца или администратора доступа пользователей.

Для управления решениями виртуальной машины существует три роли для конкретных ресурсов, предоставляющие необходимый тип доступа:

* [Участник виртуальной машины](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)
* [Участник сети](../../role-based-access-control/built-in-roles.md#network-contributor)
* [Участник учетной записи хранения](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

Чтобы не назначать роли для отдельных пользователей, зачастую бывает проще создать группу Azure Active Directory для пользователей, которым нужно выполнять подобные действия. А затем назначить этой группе соответствующую роль. В этой статье описано, как использовать существующую группу, чтобы управлять виртуальной машиной, или портал, чтобы [создать группу Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Создав новую группу или найдя существующую, воспользуйтесь командой [az role assignment create](/cli/azure/policy/assignment#az_policy_assignment_create), чтобы назначить группе Azure Active Directory роль участника виртуальных машин для группы ресурсов.

```azurecli-interactive
adgroupId=$(az ad group show --group <your-group-name> --query objectId --output tsv)

az role assignment create --assignee-object-id $adgroupId --role "Virtual Machine Contributor" --resource-group myResourceGroup
```

Если возникнет ошибка с сообщением о том, что **участник \<guid> не существует в каталоге**, это значит, что новая группа еще не используется в Azure Active Directory. Попробуйте выполнить команду снова.

Как правило, нужно повторить эту процедуру для *участника сети* и *участника учетной записи хранения*, чтобы назначить пользователей для управления развернутыми ресурсами. В текущей статье эти действия можно пропустить.

## <a name="azure-policy"></a>Политика Azure

Служба [Политика Azure](../../governance/policy/overview.md) позволяет следить за тем, чтобы все ресурсы в подписке соответствовали корпоративным стандартам. Ваша подписка уже имеет несколько определений политики. Чтобы просмотреть доступные определения политик, воспользуйтесь командой [az policy definition list](/cli/azure/policy/definition#az_policy_definition_list):

```azurecli-interactive
az policy definition list --query "[].[displayName, policyType, name]" --output table
```

Отобразится список имеющихся определений политики. Политика может относиться к типу **Встроенная** или **Настраиваемая**. Просмотрите определения, которые описывают необходимое вам условие. В этой статье вы назначаете политики, которые:

* ограничивают расположения для всех ресурсов;
* ограничивают номера SKU для виртуальных машин;
* выполняют аудит виртуальных машин, не использующих управляемые диски.

В следующем примере вы получите три определения политик на основе отображаемого имени. Назначьте эти определения группе ресурсов с помощью команды [az policy assignment create](/cli/azure/policy/assignment#az_policy_assignment_create). Для некоторых политик нужно указать допустимые значения, задав значения параметров.

```azurecli-interactive
# Get policy definitions for allowed locations, allowed SKUs, and auditing VMs that don't use managed disks
locationDefinition=$(az policy definition list --query "[?displayName=='Allowed locations'].name | [0]" --output tsv)
skuDefinition=$(az policy definition list --query "[?displayName=='Allowed virtual machine SKUs'].name | [0]" --output tsv)
auditDefinition=$(az policy definition list --query "[?displayName=='Audit VMs that do not use managed disks'].name | [0]" --output tsv)

# Assign policy for allowed locations
az policy assignment create --name "Set permitted locations" \
  --resource-group myResourceGroup \
  --policy $locationDefinition \
  --params '{ 
      "listOfAllowedLocations": {
        "value": [
          "eastus", 
          "eastus2"
        ]
      }
    }'

# Assign policy for allowed SKUs
az policy assignment create --name "Set permitted VM SKUs" \
  --resource-group myResourceGroup \
  --policy $skuDefinition \
  --params '{ 
      "listOfAllowedSKUs": {
        "value": [
          "Standard_DS1_v2", 
          "Standard_E2s_v2"
        ]
      }
    }'

# Assign policy for auditing unmanaged disks
az policy assignment create --name "Audit unmanaged disks" \
  --resource-group myResourceGroup \
  --policy $auditDefinition
```

В предыдущем примере предполагается, что вам уже известны параметры политики. Если требуется просмотреть параметры, выполните следующую команду.

```azurecli-interactive
az policy definition show --name $locationDefinition --query parameters
```

## <a name="deploy-the-virtual-machine"></a>Развертывание виртуальной машины

Роли и политики назначены — теперь все готово для развертывания вашего решения. Размер по умолчанию — Standard_DS1_v2. Он соответствует одному из ваших разрешенных номеров SKU. С помощью этой команды создаются ключи SSH, если они еще не существуют в расположении по умолчанию.

```azurecli-interactive
az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS --generate-ssh-keys
```

После завершения развертывания к решению можно применить дополнительные параметры управления.

## <a name="lock-resources"></a>Блокировка ресурсов

[Блокировки ресурсов](../../azure-resource-manager/management/lock-resources.md) не позволяют пользователям в организации случайно удалить или изменить критически важные ресурсы. В отличие от управления доступом на основе ролей, блокировки ресурсов применяют ограничение для всех пользователей и ролей. Можно установить уровень блокировки *CanNotDelete* или *ReadOnly*.

Для создания или удаления блокировок управления необходим доступ к действиям `Microsoft.Authorization/locks/*`. Из встроенных ролей эти действия предоставляются только **владельцу** и **администратору доступа пользователей**.

Чтобы заблокировать виртуальную машину и группу безопасности сети, выполните команду [az lock create](/cli/azure/resource/lock#az_resource_lock_create).

```azurecli-interactive
# Add CanNotDelete lock to the VM
az lock create --name LockVM \
  --lock-type CanNotDelete \
  --resource-group myResourceGroup \
  --resource-name myVM \
  --resource-type Microsoft.Compute/virtualMachines

# Add CanNotDelete lock to the network security group
az lock create --name LockNSG \
  --lock-type CanNotDelete \
  --resource-group myResourceGroup \
  --resource-name myVMNSG \
  --resource-type Microsoft.Network/networkSecurityGroups
```

Чтобы протестировать блокировки, попробуйте выполнить следующую команду:

```azurecli-interactive 
az group delete --name myResourceGroup
```

Появится ошибка с сообщением о том, что операцию удаления не удалось выполнить из-за блокировки. Группу ресурсов можно удалить, только если намеренно снять блокировки. Этот шаг описан в разделе [Очистка ресурсов](#clean-up-resources).

## <a name="tag-resources"></a>Добавление тегов к ресурсам

К ресурсам Azure можно применять [теги](../../azure-resource-manager/management/tag-resources.md), чтобы логически упорядочивать их по категориям. Каждый тег состоит из имени и значения. Например, имя Environment и значение Production можно применить ко всем ресурсам в рабочей среде.

[!INCLUDE [Resource Manager governance tags CLI](../../../includes/resource-manager-governance-tags-cli.md)]

Чтобы применить теги к виртуальной машине, выполните команду [az resource tag](/cli/azure/resource#az_resource_list). Существующие теги для ресурса не сохраняются.

```azurecli-interactive
az resource tag -n myVM \
  -g myResourceGroup \
  --tags Dept=IT Environment=Test Project=Documentation \
  --resource-type "Microsoft.Compute/virtualMachines"
```

### <a name="find-resources-by-tag"></a>Поиск ресурсов по тегу

Чтобы найти ресурсы по имени и значению тега, выполните команду [az resource list](/cli/azure/resource#az_resource_list):

```azurecli-interactive
az resource list --tag Environment=Test --query [].name
```

Возвращаемые значения можно использовать для выполнения задач управления, например, остановки всех виртуальных машин с определенным значением тега.

```azurecli-interactive
az vm stop --ids $(az resource list --tag Environment=Test --query "[?type=='Microsoft.Compute/virtualMachines'].id" --output tsv)
```

### <a name="view-costs-by-tag-values"></a>Просмотр данных о затратах по значениям тега

[!INCLUDE [Resource Manager governance tags billing](../../../includes/resource-manager-governance-tags-billing.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалить заблокированные группы безопасности сети невозможно до тех пор, пока не будет снята блокировка. Чтобы снять блокировку, получите идентификаторы блокировок и укажите их при выполнении команды [az lock delete](/cli/azure/resource/lock#az_resource_lock_delete).

```azurecli-interactive
vmlock=$(az lock show --name LockVM \
  --resource-group myResourceGroup \
  --resource-type Microsoft.Compute/virtualMachines \
  --resource-name myVM --output tsv --query id)
nsglock=$(az lock show --name LockNSG \
  --resource-group myResourceGroup \
  --resource-type Microsoft.Network/networkSecurityGroups \
  --resource-name myVMNSG --output tsv --query id)
az lock delete --ids $vmlock $nsglock
```

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group#az_group_delete). Завершите сеанс SSH для виртуальной машины, а затем удалите ресурсы, как показано ниже:

```azurecli-interactive 
az group delete --name myResourceGroup
```


## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы создали пользовательский образ виртуальной машины. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * назначение пользователей для роли;
> * применение политик, принудительно внедряющих стандарты;
> * защита важных ресурсов с помощью блокировок;
> * добавление к ресурсам тегов для выставления счетов и управления.

Перейдите к следующему руководству, чтобы узнать, как выявлять изменения и управлять обновлениями пакетов на виртуальной машине.

> [!div class="nextstepaction"]
> [Управление виртуальными машинами](tutorial-config-management.md)
