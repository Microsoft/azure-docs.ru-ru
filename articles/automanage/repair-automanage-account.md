---
title: Восстановление неработающей учетной записи управления Azure
description: Если вы недавно переместили подписку, содержащую учетную запись для управления, в новый клиент, ее необходимо перенастроить. В этой статье вы узнаете, как это делать.
author: asinn826
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: alsin
ms.openlocfilehash: 4694fa679c7bbff309a0452219ff39bacf2488c4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96183708"
---
# <a name="repair-an-automanage-account"></a>Восстановление учетной записи управления
[Учетная запись автоматического управления Azure](./automanage-virtual-machines.md#automanage-account) — это контекст безопасности или удостоверение, под которым выполняются автоматические операции. Если вы недавно переместили подписку, содержащую учетную запись для управления, в новый клиент, необходимо перенастроить учетную запись. Чтобы перенастроить его, необходимо сбросить тип удостоверения и назначить соответствующие роли для учетной записи.

## <a name="step-1-reset-the-automanage-account-identity-type"></a>Шаг 1. Сброс типа удостоверения учетной записи для управления
Сбросьте тип удостоверения учетной записи управления, используя следующий шаблон Azure Resource Manager (ARM). Сохраните файл локально в armdeploy.jsили с аналогичным именем. Запишите имя и расположение учетной записи, так как они являются обязательными параметрами в шаблоне ARM.

1. Создайте диспетчер ресурсов развертывание с помощью следующего шаблона. Используйте команду `identityType = None`.
    * Развертывание можно создать в Azure CLI с помощью `az deployment sub create` . Дополнительные сведения см. в разделе [AZ Deployment подсистема](/cli/azure/deployment/sub).
    * Вы можете создать развертывание в PowerShell с помощью `New-AzDeployment` модуля. Дополнительные сведения см. в разделе [New-аздеплоймент](/powershell/module/az.resources/new-azdeployment).

1. Снова запустите один и тот же шаблон ARM с помощью `identityType = SystemAssigned` .

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "identityType": {
            "type": "string",
            "allowedValues": [ "None", "SystemAssigned" ]
        }
    },
    "resources": [
        {
            "apiVersion": "2020-06-30-preview",
            "name": "[parameters('accountName')]",
            "location": "[parameters('location')]",
            "type": "Microsoft.Automanage/accounts",
            "identity": {
                "type": "[parameters('identityType')]"
            }
        }
    ]
}

```

## <a name="step-2-assign-appropriate-roles-for-the-automanage-account"></a>Шаг 2. Назначение соответствующих ролей для учетной записи управления
Для учетной записи "управление ими" требуются роли участника и политики ресурсов в подписке, содержащей виртуальные машины, которыми управляет управление. Эти роли можно назначить с помощью портал Azure, шаблонов ARM или Azure CLI.

Если вы используете шаблон ARM или Azure CLI, вам потребуется идентификатор участника (также известный как идентификатор объекта) учетной записи управления. (Идентификатор вам не нужен, если вы используете портал Azure.) Этот идентификатор можно найти с помощью следующих методов:

- [Azure CLI](/cli/azure/ad/sp): используйте команду `az ad sp list --display-name <name of your Automanage Account>` .

- Портал Azure. Перейдите в **Azure Active Directory** и найдите учетную запись для управления ими по имени. В разделе **корпоративные приложения** выберите имя учетной записи для управления, когда она появится.

### <a name="azure-portal"></a>Портал Azure
1. В разделе **подписки** перейдите к подписке, содержащей ваши управляемые виртуальные машины.
1. Перейдите в раздел **Управление доступом (IAM)**.
1. Выберите **добавить назначения ролей**.
1. Выберите роль **участник** и введите имя учетной записи управления.
1. Щелкните **Сохранить**.
1. Повторите шаги с 3 по 5, на этот раз с ролью **участник политики ресурсов** .

### <a name="arm-template"></a>Шаблон ARM
Запустите следующий шаблон ARM. Вам потребуется идентификатор участника учетной записи управления. Инструкции по его выполнению приведены в начале этого раздела. Введите идентификатор при появлении запроса.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "principalId": {
            "type": "string",
            "metadata": {
                "description": "The principal to assign the role to"
            }
        }
    },
    "variables": {
        "Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]",
        "Resource Policy Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', '36243c78-bf99-498c-9df9-86d9f8d28608')]"
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[guid(uniqueString(variables('Contributor')))]",
            "properties": {
                "roleDefinitionId": "[variables('Contributor')]",
                "principalId": "[parameters('principalId')]"
            }
        },
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[guid(uniqueString(variables('Resource Policy Contributor')))]",
            "properties": {
                "roleDefinitionId": "[variables('Resource Policy Contributor')]",
                "principalId": "[parameters('principalId')]"
            }
        }
    ]
}
```

### <a name="azure-cli"></a>Azure CLI
Выполните следующие команды:

```azurecli
az role assignment create --assignee-object-id <your Automanage Account Object ID> --role "Contributor" --scope /subscriptions/<your subscription ID>

az role assignment create --assignee-object-id <your Automanage Account Object ID> --role "Resource Policy Contributor" --scope /subscriptions/<your subscription ID>
```

## <a name="next-steps"></a>Дальнейшие действия
[Дополнительные сведения об управлении Azure](./automanage-virtual-machines.md)