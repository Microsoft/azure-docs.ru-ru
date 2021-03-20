---
title: Предоставление приложению доступа к другим ресурсам Azure
description: В этой статье объясняется, как предоставить управляемому удостоверению приложения Service Fabric доступ к другим ресурсам Azure, поддерживающим проверку подлинности на основе Azure Active Directory.
ms.topic: article
ms.date: 12/09/2019
ms.openlocfilehash: c7560294fbf6d122396b6a5a8ffd3ee93bc89048
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97507461"
---
# <a name="granting-a-service-fabric-applications-managed-identity-access-to-azure-resources"></a>Предоставление управляемому удостоверению приложения Service Fabric доступ к ресурсам Azure

Прежде чем приложение сможет использовать управляемое удостоверение для доступа к другим ресурсам, необходимо предоставить этому удостоверению доступ к защищенному ресурсу Azure. Предоставление разрешений обычно является действием по управлению на плоскости управления службы Azure, владеющей защищенным ресурсом, находящимся через Azure Resource Manager, который обеспечивает все применимые проверки доступа на основе ролей.

Точная последовательность шагов будет зависеть от типа ресурса Azure, к которому осуществляется доступ, а также от языка и клиента, используемого для предоставления разрешений. Оставшаяся часть статьи предполагает, что назначенное приложению удостоверение назначено пользователю и включает несколько типичных примеров для вашего удобства, но не является исчерпывающей ссылкой на этот раздел; Ознакомьтесь с документацией соответствующих служб Azure, чтобы получить последние инструкции по предоставлению разрешений.  

## <a name="granting-access-to-azure-storage"></a>Предоставление доступа к службе хранилища Azure
Для получения данных из большого двоичного объекта службы хранилища Azure можно использовать управляемое удостоверение приложения Service Fabric (назначаемое пользователем в этом случае). Предоставьте удостоверению необходимые разрешения в портал Azure, выполнив следующие действия.

1. Перейдите к учетной записи хранения.
2. Щелкните ссылку Управление доступом (IAM) на панели слева.
3. используемых Проверить существующий доступ: выберите системное или назначенное пользователем управляемое удостоверение в элементе управления "найти"; Выбор соответствующего удостоверения из списка «исходящие результаты»
4. Щелкните + Добавить назначение ролей в верхней части страницы, чтобы добавить новое назначение роли для удостоверения приложения.
В поле Роль из раскрывающегося списка выберите Модуль чтения данных BLOB-объектов хранилища.
5. В следующем раскрывающемся списке в разделе Назначение доступа к выберите `User assigned managed identity` .
6. Убедитесь, что нужная подписка присутствует в раскрывающемся списке Подписка, и установите для параметра Группа ресурсов значение Все группы ресурсов.
7. В разделе Выбрать выберите УАИ, соответствующий приложению Service Fabric, а затем нажмите кнопку Сохранить.

Поддержка назначенных системе Service Fabric управляемых удостоверений не включает интеграцию в портал Azure; Если приложение использует назначенное системой удостоверение, необходимо сначала найти идентификатор клиента для удостоверения приложения, а затем повторить описанные выше действия, но выбрать `Azure AD user, group, or service principal` параметр в элементе управления Find.

## <a name="granting-access-to-azure-key-vault"></a>Предоставление доступа к Azure Key Vault
Аналогично, при доступе к хранилищу можно использовать управляемое удостоверение приложения Service Fabric для доступа к хранилищу ключей Azure. Действия по предоставлению доступа в портал Azure похожи на те, которые перечислены выше, и не будут повторяться здесь. Различия см. на рисунке ниже.

![Политика доступа к Key Vault](../key-vault/media/vs-secure-secret-appsettings/add-keyvault-access-policy.png)

В следующем примере показано предоставление доступа к хранилищу с помощью развертывания шаблона. Добавьте приведенные ниже фрагменты кода в качестве другой записи в `resources` элементе шаблона. В образце демонстрируется предоставление доступа для типов удостоверений, назначаемых пользователем и системой, соответственно, выбор применимого.

```json
    # under 'variables':
  "variables": {
        "userAssignedIdentityResourceId" : "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]",
    }
    # under 'resources':
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2018-02-14",
        "properties": {
            "accessPolicies": [
                {
                    "tenantId": "[reference(variables('userAssignedIdentityResourceId'), '2018-11-30').tenantId]",
                    "objectId": "[reference(variables('userAssignedIdentityResourceId'), '2018-11-30').principalId]",
                    "dependsOn": [
                        "[variables('userAssignedIdentityResourceId')]"
                    ],
                    "permissions": {
                        "keys":         ["get", "list"],
                        "secrets":      ["get", "list"],
                        "certificates": ["get", "list"]
                    }
                }
            ]
        }
    },
```
И для управляемых идентификаторов, назначенных системой:
```json
    # under 'variables':
  "variables": {
        "sfAppSystemAssignedIdentityResourceId": "[concat(resourceId('Microsoft.ServiceFabric/clusters/applications/', parameters('clusterName'), parameters('applicationName')), '/providers/Microsoft.ManagedIdentity/Identities/default')]"
    }
    # under 'resources':
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2018-02-14",
        "properties": {
            "accessPolicies": [
            {
                    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
                    "tenantId": "[reference(variables('sfAppSystemAssignedIdentityResourceId'), '2018-11-30').tenantId]",
                    "objectId": "[reference(variables('sfAppSystemAssignedIdentityResourceId'), '2018-11-30').principalId]",
                    "dependsOn": [
                        "[variables('sfAppSystemAssignedIdentityResourceId')]"
                    ],
                    "permissions": {
                        "secrets": [
                            "get",
                            "list"
                        ],
                        "certificates": 
                        [
                            "get", 
                            "list"
                        ]
                    }
            },
        ]
        }
    }
```

Дополнительные сведения см. в разделе [хранилища — Политика доступа для обновления](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="next-steps"></a>Дальнейшие действия
* [Развертывание приложения Service Fabric Azure с управляемым удостоверением, назначенным системой](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Развертывание приложения Service Fabric Azure с помощью управляемого удостоверения, назначенного пользователем](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
