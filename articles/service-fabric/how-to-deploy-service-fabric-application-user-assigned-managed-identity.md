---
title: Развертывание приложения с управляемым удостоверением, назначенным пользователем
description: В этой статье показано, как развернуть приложение Service Fabric с управляемым удостоверением, назначенным пользователем.
ms.topic: article
ms.date: 12/09/2019
ms.openlocfilehash: 79d8654733b580be96d59e78f31105077929ac78
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86260081"
---
# <a name="deploy-service-fabric-application-with-a-user-assigned-managed-identity"></a>Развертывание Service Fabric приложения с помощью управляемого User-Assigned удостоверения

Чтобы развернуть приложение Service Fabric с управляемым удостоверением, приложение должно быть развернуто с помощью Azure Resource Manager, обычно с шаблоном Azure Resource Manager. Дополнительные сведения о развертывании приложения Service Fabric с помощью Azure Resource Manager см. в статье [Управление приложениями и службами в качестве Azure Resource Managerных ресурсов](service-fabric-application-arm-resource.md).

> [!NOTE] 
> 
> Приложения, которые не развертываются как ресурсы Azure, **не могут** иметь управляемые удостоверения. 
>
> Service Fabric развертывание приложения с управляемым удостоверением поддерживается в версии API `"2019-06-01-preview"` . Можно также использовать ту же версию API для типа приложения, версию типа приложения и ресурсы службы.
>

## <a name="user-assigned-identity"></a>Удостоверение User-Assigned

Чтобы включить приложение с удостоверением User-Assigned, сначала добавьте свойство **Identity** в ресурс приложения с типом **усерассигнед** и назначенными пользователем удостоверениями. Затем добавьте раздел **манажедидентитиес** в раздел **Properties** для ресурса **приложения** , который содержит список понятных имен для сопоставления principalId для каждого назначенного пользователем удостоверения. Дополнительные сведения о назначенных пользователем удостоверениях см. [в статье Создание, перечисление или удаление назначенного пользователем управляемого удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md).

### <a name="application-template"></a>Шаблон приложения

Чтобы включить приложение с пользовательским удостоверением, сначала добавьте свойство **Identity** в ресурс приложения с типом **усерассигнед** и назначенными пользователем удостоверениями, а затем добавьте объект **манажедидентитиес** в раздел **Properties** , содержащий список понятного имени для сопоставления principalId для каждого назначенного пользователя удостоверения.

```json
{
  "apiVersion": "2019-06-01-preview",
  "type": "Microsoft.ServiceFabric/clusters/applications",
  "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'), '/versions/', parameters('applicationTypeVersion'))]",
    "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]"
  ],
  "identity": {
    "type" : "userAssigned",
    "userAssignedIdentities": {
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]": {}
    }
  },
  "properties": {
    "typeName": "[parameters('applicationTypeName')]",
    "typeVersion": "[parameters('applicationTypeVersion')]",
    "parameters": {
    },
    "managedIdentities": [
      {
        "name" : "[parameters('userAssignedIdentityName')]",
        "principalId" : "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName')), '2018-11-30').principalId]"
      }
    ]
  }
}
```

В приведенном выше примере имя ресурса назначенного пользователя используется в качестве понятного имени управляемого удостоверения для приложения. В следующих примерах предполагается, что фактически понятное имя — «AdminUser».

### <a name="application-package"></a>Пакет приложения

1. Для каждого удостоверения, определенного в `managedIdentities` разделе шаблона Azure Resource Manager, добавьте `<ManagedIdentity>` тег в манифест приложения в разделе **участников** . `Name`Атрибут должен соответствовать `name` свойству, определенному в `managedIdentities` разделе.

    **ApplicationManifest.xml**

    ```xml
      <Principals>
        <ManagedIdentities>
          <ManagedIdentity Name="AdminUser" />
        </ManagedIdentities>
      </Principals>
    ```

2. В разделе **ServiceManifestImport** добавьте **идентитибиндингполици** для службы, которая использует управляемое удостоверение. Эта политика сопоставляет `AdminUser` удостоверение с именем удостоверения, зависящего от службы, которое необходимо добавить в манифест службы позже.

    **ApplicationManifest.xml**

    ```xml
      <ServiceManifestImport>
        <Policies>
          <IdentityBindingPolicy ServiceIdentityRef="WebAdmin" ApplicationIdentityRef="AdminUser" />
        </Policies>
      </ServiceManifestImport>
    ```

3. Обновите манифест службы, чтобы добавить **ManagedIdentity** в раздел **Resources** с именем, соответствующим `ServiceIdentityRef` в `IdentityBindingPolicy` манифесте приложения:

    **ServiceManifest.xml**

    ```xml
      <Resources>
        ...
        <ManagedIdentities DefaultIdentity="WebAdmin">
          <ManagedIdentity Name="WebAdmin" />
        </ManagedIdentities>
      </Resources>
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Использование управляемого удостоверения в коде приложения Service Fabric](how-to-managed-identity-service-fabric-app-code.md)
* [Как предоставить Service Fabric доступ к другим ресурсам Azure для приложений](how-to-grant-access-other-resources.md)
