---
title: Развертывание политики, которую можно исправить
description: Чтобы развернуть политики, использующие задачу исправления с помощью Azure Лигхсаусе, необходимо создать управляемое удостоверение в клиенте клиента.
ms.date: 01/14/2021
ms.topic: how-to
ms.openlocfilehash: 01070133241117596bdf2b8e1e7c3fa101fc656c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98233888"
---
# <a name="deploy-a-policy-that-can-be-remediated-within-a-delegated-subscription"></a>Развертывание политики, которую можно исправить в рамках делегированной подписки

[Azure Lighthouse](../overview.md) позволяет поставщикам служб создавать и редактировать определения политики в рамках делегированной подписки. Однако для развертывания политик, использующих [задачу по исправлению](../../governance/policy/how-to/remediate-resources.md) (то есть политики с [deployIfNotExists](../../governance/policy/concepts/effects.md#deployifnotexists) или [изменением](../../governance/policy/concepts/effects.md#modify) ), необходимо создать [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) в клиенте клиента. Это управляемое удостоверение может использоваться политикой Azure для развертывания шаблона в политике. Для включения этого сценария, как при подключении клиента к управлению делегированными ресурсами Azure, так и при развертывании самой политики необходимо выполнять некоторые шаги.

> [!TIP]
> Хотя в этом разделе мы будем называть поставщиков услуг и клиентов, [предприятия, управляющие несколькими клиентами](../concepts/enterprise.md) , могут использовать одни и те же процессы.

## <a name="create-a-user-who-can-assign-roles-to-a-managed-identity-in-the-customer-tenant"></a>Создание пользователя, который может назначать роли управляемому удостоверению в клиенте клиента

При подключении клиента к Azure Лигхсаусе используется [шаблон Azure Resource Manager](onboard-customer.md#create-an-azure-resource-manager-template) вместе с файлом параметров для определения авторизаций, предоставляющих доступ к делегированным ресурсам в клиенте клиента. Каждая авторизация указывает **principalId** , соответствующий пользователю Azure AD, группе или субъекту-службе в управляющем клиенте, а также **roleDefinitionId** , который соответствует [встроенной роли Azure](../../role-based-access-control/built-in-roles.md) , которая будет предоставлена.

Чтобы разрешить **principalId** создать управляемое удостоверение в клиенте клиента, необходимо задать для его **roleDefinitionId** значение **Администратор доступа пользователя**. Хотя эта роль обычно не поддерживается, ее можно использовать в этом конкретном сценарии, позволяя учетным записям пользователей с этим разрешением назначать одну или несколько встроенных ролей управляемым удостоверениям. Эти роли определяются в свойстве **делегатедроледефинитионидс** и могут включать все [Поддерживаемые встроенные роли Azure](../concepts/tenants-users-roles.md#role-support-for-azure-lighthouse) , за исключением администратора доступа пользователей или владельца.

После подключения клиента **principalId**, созданный в этой авторизации, сможет назначать эти встроенные роли управляемым удостоверениям в клиенте клиента. Однако у них не будет никаких других разрешений, обычно связанных с ролью администратора доступа пользователей.

В следующем примере показан **principalId**, которому будет назначена роль администратора доступа пользователей. Этот пользователь будет иметь возможность назначать две встроенные роли управляемым удостоверениям в клиенте клиента: участник и Log Analytics участник.

```json
{
    "principalId": "3kl47fff-5655-4779-b726-2cf02b05c7c4",
    "principalIdDisplayName": "Policy Automation Account",
    "roleDefinitionId": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "delegatedRoleDefinitionIds": [
         "b24988ac-6180-42a0-ab88-20f7382dd24c",
         "92aaf0da-9dab-42b6-94a3-d43ce8d16293"
    ]
}
```

## <a name="deploy-policies-that-can-be-remediated"></a>Развертывание политик, которые можно исправить

После создания пользователя с необходимыми разрешениями, как описано выше, этот пользователь может развернуть политики, использующие задачи исправления в делегированных подписках клиента.

Например, предположим, что вы хотите включить диагностику для Azure Key Vault ресурсов в клиенте клиента, как показано в этом [примере](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-enforce-keyvault-monitoring). Пользователь в управляющем клиенте с соответствующими разрешениями (как описано выше) развернет [шаблон Azure Resource Manager](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/policy-enforce-keyvault-monitoring/enforceAzureMonitoredKeyVault.json), чтобы включить этот сценарий.

Обратите внимание, что создание назначения политики для использования с делегированной подпиской в настоящее время должно осуществляться через API, а не на портале Azure. При этом для **apiVersion** должно быть задано **2019-04-01-preview**, включающее новое свойство **DelegatedManagedIdentityResourceId**. Это свойство позволяет включить управляемое удостоверение, которое находится в клиенте клиента (в подписке или группе ресурсов, которая была подключена к Azure Лигхсаусе).

В следующем примере показано назначение роли с **delegatedManagedIdentityResourceId**.

```json
"type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2019-04-01-preview",
            "name": "[parameters('rbacGuid')]",
            "dependsOn": [
                "[variables('policyAssignment')]"
            ],
            "properties": {
                "roleDefinitionId": "[concat(subscription().id, '/providers/Microsoft.Authorization/roleDefinitions/', variables('rbacContributor'))]",
                "principalType": "ServicePrincipal",
                "delegatedManagedIdentityResourceId": "[concat(subscription().id, '/providers/Microsoft.Authorization/policyAssignments/', variables('policyAssignment'))]",
                "principalId": "[toLower(reference(concat('/providers/Microsoft.Authorization/policyAssignments/', variables('policyAssignment')), '2018-05-01', 'Full' ).identity.principalId)]"
            }
```

> [!TIP]
> Доступен [аналогичный пример](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-add-or-replace-tag), демонстрирующий, как развернуть политику, которая добавляет или удаляет тег (используя эффект изменения) в делегированной подписке.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о службе [Политика Azure](../../governance/policy/index.yml).
- Сведения об [управляемых удостоверениях для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md).
