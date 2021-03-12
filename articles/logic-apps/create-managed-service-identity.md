---
title: Аутентификация на основе управляемых удостоверений
description: Доступ к ресурсам, защищенным Azure Active Directory, без входа с использованием учетных данных или секретов с помощью управляемого удостоверения
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, azla
ms.topic: article
ms.date: 03/09/2021
ms.openlocfilehash: 7796fc7e2032559ca3ff5c738c46fe025719942d
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102556627"
---
# <a name="authenticate-access-to-azure-resources-by-using-managed-identities-in-azure-logic-apps"></a>Проверка подлинности и получение доступа к ресурсам Azure с помощью управляемых удостоверений в Azure Logic Apps.

Чтобы легко получить доступ к другим ресурсам, защищенным Azure Active Directory (Azure AD), и проверить подлинность удостоверения, приложение логики может использовать [управляемое удостоверение](../active-directory/managed-identities-azure-resources/overview.md) (прежнее управляемое удостоверение службы или MSI), а не учетные данные, секреты или токены Azure AD. Azure управляет этим удостоверением и помогает защитить учетные данные, так как вам не нужно управлять секретами или напрямую использовать токены Azure AD.

Azure Logic Apps поддерживает [*назначенные системой*](../active-directory/managed-identities-azure-resources/overview.md) и [*назначенные пользователем*](../active-directory/managed-identities-azure-resources/overview.md) управляемые удостоверения. Приложение логики или отдельные подключения могут использовать либо назначенное системой удостоверение, либо *одно* пользовательское удостоверение, которое можно совместно использовать в группе приложений логики, но не в обеих.

## <a name="where-can-logic-apps-use-managed-identities"></a>Где можно использовать управляемые удостоверения для приложений логики?

Сейчас только [определенные встроенные триггеры и действия](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions) и [определенные управляемые соединители](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions) , которые поддерживают Azure AD OAuth, могут использовать управляемое удостоверение для проверки подлинности. Вот пример выбора:

**Встроенные триггеры и действия**

* Cлужба управления Azure API
* Службы приложений Azure
* Функции Azure
* HTTP
* HTTP + веб-перехватчик

> [!NOTE]
> Хотя триггер и действие HTTP могут проверять подлинность подключений к учетным записям хранения Azure за брандмауэром Azure, используя управляемое системой удостоверение, они не могут использовать управляемое пользователем удостоверение для проверки подлинности тех же подключений.

**Управляемые соединители**

* Служба автоматизации Azure
* Сетка событий Azure
* Azure Key Vault
* Azure Resource Manager
* HTTP с Azure AD

Поддержка управляемых соединителей сейчас доступна в предварительной версии. Текущий список см. в разделе [типы проверки подлинности для триггеров и действий, поддерживающих проверку подлинности](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

В этой статье показано, как настроить оба типа управляемых удостоверений для приложения логики. Дополнительные сведения см. в следующих статьях:

* [Триггеры и действия, поддерживающие управляемые удостоверения](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions)
* [Ограничения управляемых удостоверений для приложений логики](../logic-apps/logic-apps-limits-and-config.md#managed-identity)
* [Службы, которые поддерживают управляемые удостоверения для проверки подлинности Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication)

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись и подписка Azure. Если у вас нет ее, вы можете [зарегистрироваться для получения бесплатной учетной записи Azure](https://azure.microsoft.com/free/). Как назначенное системой управляемое удостоверение, так и ресурс, к которому необходимо назначить доступ, должны находиться в одной подписке Azure.

* Чтобы предоставить управляемому удостоверению доступ к ресурсу Azure, необходимо добавить роль в целевой ресурс для этого удостоверения. Чтобы добавить роли, необходимо иметь [разрешения администратора Azure AD](../active-directory/roles/permissions-reference.md), которые могут назначать роли удостоверениям в соответствующем клиенте Azure AD.

* Целевой ресурс Azure, к которому требуется получить доступ. В этом ресурсе вы добавите роль для управляемого удостоверения, которая помогает приложению логики подтвердить доступ к целевому ресурсу.

* Приложение логики, в котором вы хотите использовать [триггер или действия, поддерживающие управляемые удостоверения](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

## <a name="enable-managed-identity"></a>Включение управляемого удостоверения

Чтобы настроить управляемое удостоверение, которое вы хотите использовать, перейдите по ссылке для этого удостоверения:

* [Назначаемое системой удостоверение](#system-assigned)
* [Назначаемое пользователем удостоверение](#user-assigned)

<a name="system-assigned"></a>

### <a name="enable-system-assigned-identity"></a>Включить назначаемое системой удостоверение

В отличие от назначаемых пользователем удостоверений, создаваемое системой удостоверение не нужно создавать самостоятельно. Чтобы настроить для приложения логики удостоверение, назначенное системой, можно использовать следующие варианты:

* [Портал Azure](#azure-portal-system-logic-app)
* [Шаблоны диспетчера ресурсов Azure](#template-system-logic-app)

<a name="azure-portal-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-azure-portal"></a>Включить назначаемое системой удостоверение через портал Azure

1. На [портале Azure](https://portal.azure.com) откройте приложение логики в конструкторе приложений логики.

1. В меню приложения логики в разделе **Параметры** выберите **Идентификатор**. Выберите **Назначенное системой** > **Включено** > **Сохранить**. Когда Azure запросит подтверждение, выберите **Да**.

   ![Включение назначаемого системой удостоверения](./media/create-managed-service-identity/enable-system-assigned-identity.png)

   > [!NOTE]
   > Если вы получаете сообщение об ошибке, уведомляющее, что может быть активно только одно управляемое удостоверение, это значит, что приложение логики уже связано с назначенным пользователем удостоверением. Прежде чем можно будет добавить назначенное системой удостоверение, необходимо сначала *удалить* назначенное пользователем удостоверение из приложения логики.

   Теперь приложение логики может использовать назначенное системой удостоверение, зарегистрированное в Azure AD и представленное ИДЕНТИФИКАТОРом объекта.

   ![Идентификатор объекта для назначенного системой удостоверения](./media/create-managed-service-identity/object-id-system-assigned-identity.png)

   | Свойство | Значение | Описание |
   |----------|-------|-------------|
   | **Идентификатор объекта** | <*identity-resource-ID*> | Глобальный уникальный идентификатор (GUID), который представляет присвоенное системой управляемое удостоверение приложения логики в клиенте Azure AD |
   ||||

1. Теперь выполните действия [, которые предоставляют удостоверению доступ к ресурсу](#access-other-resources), описанные далее в этом разделе.

<a name="template-system-logic-app"></a>

#### <a name="enable-system-assigned-identity-in-azure-resource-manager-template"></a>Включение назначенного системой удостоверения в шаблоне Azure Resource Manager

Чтобы автоматизировать создание и развертывание таких ресурсов Azure, как приложения логики, можно настроить [шаблоны Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md). Чтобы включить управляемое системой удостоверение в приложение логики в шаблоне, добавьте объект `identity` и свойство дочернего элемента `type` в определение ресурса приложения логики в шаблоне, например:

```json
{
   "apiVersion": "2016-06-01",
   "type": "Microsoft.logic/workflows",
   "name": "[variables('logicappName')]",
   "location": "[resourceGroup().location]",
   "identity": {
      "type": "SystemAssigned"
   },
   "properties": {
      "definition": {
         "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
         "actions": {},
         "parameters": {},
         "triggers": {},
         "contentVersion": "1.0.0.0",
         "outputs": {}
   },
   "parameters": {},
   "dependsOn": []
}
```

При создании Azure определения ресурса для приложения логики объект `identity` получает следующие дополнительные свойства:

```json
"identity": {
   "type": "SystemAssigned",
   "principalId": "<principal-ID>",
   "tenantId": "<Azure-AD-tenant-ID>"
}
```

| Свойство (JSON) | Значение | Описание |
|-----------------|-------|-------------|
| `principalId` | <*ИД субъекта*> | Глобальный уникальный идентификатор (GUID) объекта субъекта-службы для управляемого удостоверения, представляющего ваше приложение логики в клиенте Azure AD. Этот идентификатор GUID иногда отображается как "идентификатор объекта" или `objectID`. |
| `tenantId` | <*ИД клиента Azure AD*> | Глобальный уникальный идентификатор (GUID), представляющий клиент Azure AD, участником которого теперь является приложение логики. В клиенте Azure AD субъект-служба имеет то же имя, что и экземпляр приложения логики. |
||||

<a name="user-assigned"></a>

### <a name="enable-user-assigned-identity"></a>Подключение назначенного пользователем удостоверения

Чтобы настроить назначенное пользователем удостоверение для приложения логики, необходимо сначала создать это удостоверение в отдельном автономном ресурсе Azure. Воспользоваться можно следующими вариантами.

* [Портал Azure](#azure-portal-user-identity)
* [Шаблоны диспетчера ресурсов Azure](#template-user-identity)
* Azure PowerShell
  * [Создание назначаемого пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
  * [Добавление назначения роли](../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md)
* Azure CLI
  * [Создание назначаемого пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
  * [Добавление назначения роли](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md)
* REST API Azure
  * [Создание назначаемого пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)
  * [Добавление назначения роли](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-user-identity"></a>

#### <a name="create-user-assigned-identity-in-the-azure-portal"></a>Создание назначаемого пользователем управляемого удостоверения через портал Azure

1. На [портале Azure](https://portal.azure.com) в поле поиска на любой странице введите `managed identities` и затем выберите пункт **Управляемые удостоверения**.

   ![Найдите и выберите пункт "Управляемые удостоверения".](./media/create-managed-service-identity/find-select-managed-identities.png)

1. В разделе **Управляемые удостоверения** нажмите **Добавить**.

   ![Добавление управляемого удостоверения](./media/create-managed-service-identity/add-user-assigned-identity.png)

1. Укажите сведения об управляемом удостоверении, а затем выберите **Проверка + создать**, например:

   ![Создание управляемого удостоверения, назначаемого пользователем](./media/create-managed-service-identity/create-user-assigned-identity.png)

   | Свойство | Обязательно | Значение | Описание |
   |----------|----------|-------|-------------|
   | **Подписка** | Да | <*Azure-subscription-name*> | Имя подписки Azure, которую нужно использовать. |
   | **Группа ресурсов** | Да | <*имя_группы_ресурсов_Azure*> | Имя, используемое группой ресурсов. Создайте группу или выберите имеющуюся. В этом примере создается новая группа с именем `fabrikam-managed-identities-RG` . |
   | **Регион** | Да | <*Azure-region*> | Регион Azure, в котором будут храниться сведения о приложении логики. В этом примере используется регион "Западная часть США". |
   | **имя**; | Да | <*user-assigned-identity-name*> | Имя, присваиваемое удостоверению. В этом примере используется `Fabrikam-user-assigned-identity`. |
   |||||

   После проверки этих сведений Azure создаст управляемое удостоверение. Теперь можно добавить назначенное пользователем удостоверение в приложение логики. Невозможно добавить более одного назначенного пользователем удостоверения в приложение логики.

1. На портале Azure найдите ваше приложение логики и откройте его в конструкторе приложений логики.

1. В меню приложения логики в разделе **Параметры** выберите пункт **Удостоверение**, затем нажмите на **Назначенное пользователем** > **Добавить**.

   ![Добавление назначенного пользователем управляемого удостоверения](./media/create-managed-service-identity/add-user-assigned-identity-logic-app.png)

1. Выберите подписку Azure из списка **Подписки** (если она уже не была выбрана) на панели **Добавление назначенного пользователем управляемого удостоверения**. Выберите необходимое назначенное пользователем управляемое удостоверение из списка, в котором отображаются *все* управляемые удостоверения на данной подписке. Чтобы отфильтровать список, в поле поиска **Назначенные пользователем управляемые удостоверения** нужно будет ввести имя удостоверения или группы ресурсов. Закончив, нажмите кнопку **Добавить**.

   ![Выберите назначаемое пользователем удостоверение для использования](./media/create-managed-service-identity/select-user-assigned-identity.png)

   > [!NOTE]
   > Если вы получаете сообщение об ошибке, уведомляющее, что может быть активно только одно управляемое удостоверение, это значит, что приложение логики уже связано с назначенным системой удостоверением. Прежде чем можно будет добавить назначенное пользователем удостоверение, необходимо сначала удалить назначенное системой удостоверение из приложения логики.

   Теперь приложение логики связано с управляемым удостоверением, назначенным пользователем.

   ![Привязка назначаемого пользователем удостоверения](./media/create-managed-service-identity/added-user-assigned-identity.png)

1. Теперь выполните действия [, которые предоставляют удостоверению доступ к ресурсу](#access-other-resources), описанные далее в этом разделе.

<a name="template-user-identity"></a>

#### <a name="create-user-assigned-identity-in-an-azure-resource-manager-template"></a>Создание назначенного пользователем удостоверения в шаблоне Azure Resource Manager

Если вы хотите автоматизировать создание и развертывание таких ресурсов Azure, как приложения логики, можно настроить [шаблоны Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md), поддерживающие [установленные пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md) для аутентификации. В разделе `resources` шаблона вам потребуются следующие элементы для определения ресурса приложения логики:

* Объект `identity` со свойством `type`, для которого задано значение `UserAssigned`.

* Дочерний `userAssignedIdentities` объект, указывающий назначенный пользователю ресурс и имя

В этом примере показано определение ресурса приложения логики для запроса HTTP-размещения, а также объект `identity` без параметров. Ответ на запрос размещения и последующие операции GET также имеют объект `identity`:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicappName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user-assigned-identity-name>": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": []
      },
   ],
   "outputs": {}
}
```

Если шаблон также включает определение ресурса управляемого удостоверения, можно параметризовать объект `identity`. В этом примере показано, как дочерний объект `userAssignedIdentities` ссылается на переменную `userAssignedIdentity`, определенную в разделе `variables` шаблона. Эта переменная ссылается на идентификатор ресурса для назначенного пользователю удостоверения.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "Template_LogicAppName": {
         "type": "string"
      },
      "Template_UserAssignedIdentityName": {
         "type": "securestring"
      }
   },
   "variables": {
      "logicAppName": "[parameters(`Template_LogicAppName')]",
      "userAssignedIdentityName": "[parameters('Template_UserAssignedIdentityName')]"
   },
   "resources": [
      {
         "apiVersion": "2016-06-01",
         "type": "Microsoft.logic/workflows",
         "name": "[variables('logicAppName')]",
         "location": "[resourceGroup().location]",
         "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
               "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]": {}
            }
         },
         "properties": {
            "definition": {<logic-app-workflow-definition>}
         },
         "parameters": {},
         "dependsOn": [
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', variables('userAssignedIdentityName'))]"
         ]
      },
      {
         "apiVersion": "2018-11-30",
         "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
         "name": "[parameters('Template_UserAssignedIdentityName')]",
         "location": "[resourceGroup().location]",
         "properties": {}
      }
  ]
}
```

<a name="access-other-resources"></a>

## <a name="give-identity-access-to-resources"></a>Предоставление удостоверению доступа к ресурсам

Прежде чем использовать управляемое удостоверение приложения логики для аутентификации, настройте доступ для этого удостоверения в ресурсе Azure, где вы планируете использовать удостоверение. Чтобы выполнить эту задачу, назначьте удостоверению соответствующую роль в целевом ресурсе Azure. Воспользоваться можно следующими вариантами.

* [Портал Azure](#azure-portal-assign-access)
* [Шаблон Azure Resource Manager](../role-based-access-control/role-assignments-template.md)
* Azure PowerShell ([New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment)). Дополнительные сведения см. в разделе [Добавление назначения ролей с помощью Azure RBAC и Azure PowerShell](../role-based-access-control/role-assignments-powershell.md).
* Azure CLI ([az role assignment create](/cli/azure/role/assignment#az-role-assignment-create)). Дополнительные сведения см. в разделе [Добавление назначения ролей с помощью Azure RBAC и Azure CLI](../role-based-access-control/role-assignments-cli.md).
* [Azure REST API](../role-based-access-control/role-assignments-rest.md)

<a name="azure-portal-assign-access"></a>

### <a name="assign-access-in-the-azure-portal"></a>Назначение доступа на портале Azure

В целевом ресурсе Azure, к которому должно быть предоставлено управляемое удостоверение, предоставьте этому удостоверению доступ на основе ролей к целевому ресурсу.

1. На [портале Azure](https://portal.azure.com) перейдите к ресурсу Azure, к которому вы хотите назначить доступ для управляемого удостоверения.

1. В меню ресурса выберите **Управление доступом (IAM)**  > **Назначение ролей**. Там можно будет проверить текущие назначения ролей для этого ресурса. Выберите на панели инструментов **Добавить** > **Добавить назначение ролей**.

   ![Выберите "Добавить" > "Добавить назначение ролей"](./media/create-managed-service-identity/add-role-to-resource.png)

   > [!TIP]
   > Если параметр **Добавить назначение ролей** отключен, скорее всего, у вас нет разрешений. Дополнительные сведения о различных ролях администраторов см. в статье [Разрешения роли администратора в Azure Active Directory](../active-directory/roles/permissions-reference.md).

1. В разделе **Добавить назначение ролей** выберите **роль**, которая предоставит удостоверению необходимый доступ к целевому ресурсу.

   В этом примере для удостоверения требуется [роль, которая может получить доступ к большому двоичному объекту в контейнере службы хранилища Azure](../storage/common/storage-auth-aad.md#assign-azure-roles-for-access-rights), поэтому выберите роль **участника данных BLOB-объекта хранилища** для управляемого удостоверения.

   ![Выберите роль "Участник хранилища для больших двоичных объектов"](./media/create-managed-service-identity/select-role-for-identity.png)

1. Выполните следующие действия для управляемого удостоверения.

   * **Назначаемое системой удостоверение**

     1. В поле **Назначить доступ к** выберите **Приложение логики**. Когда появится свойство **Подписка**, выберите подписку Azure, связанную с вашим удостоверением.

        ![Выбор доступа для назначенного системой удостоверения](./media/create-managed-service-identity/assign-access-system.png)

     1. В поле **Выбор** выберите приложение логики из списка. Если список слишком длинный, воспользуйтесь полем **Выбор**, чтобы отфильтровать список.

        ![Выбор приложения логики для назначенного системой удостоверения](./media/create-managed-service-identity/add-permissions-select-logic-app.png)

   * **Назначаемое пользователем удостоверение**

     1. В поле **Назначить доступ** выберите **Управляемое удостоверение, назначаемое пользователем**. Когда появится свойство **Подписка**, выберите подписку Azure, связанную с вашим удостоверением.

        ![Выбор доступа для назначаемого пользователем удостоверения](./media/create-managed-service-identity/assign-access-user.png)

     1. В поле **Выбор** выберите удостоверение из списка. Если список слишком длинный, воспользуйтесь полем **Выбор**, чтобы отфильтровать список.

        ![Выберите назначаемое пользователем удостоверение](./media/create-managed-service-identity/add-permissions-select-user-assigned-identity.png)

1. Когда все будет готово, нажмите кнопку **Сохранить**.

   Список ролей целевого ресурса теперь будет отображать выбранное удостоверение и роль. В этом примере показано, как можно использовать назначенное системой удостоверение для одного приложения логики и назначенное пользователем удостоверение для группы других приложений логики.

   ![В целевой ресурс добавлены управляемые удостоверения и роли](./media/create-managed-service-identity/added-roles-for-identities.png)

   Чтобы узнать больше, [назначьте управляемому удостоверению доступ к ресурсу через портал Azure](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md).

1. Теперь выполните [действия, чтобы проверить подлинность доступа с помощью удостоверения](#authenticate-access-with-identity) в триггере или действии, поддерживающем управляемые удостоверения.

<a name="authenticate-access-with-identity"></a>

## <a name="authenticate-access-with-managed-identity"></a>Подтверждение подлинности доступа на основе управляемых удостоверений

После [включения управляемого удостоверения в приложение логики](#azure-portal-system-logic-app) и [предоставления этому удостоверению доступа к целевому ресурсу или сущности](#access-other-resources) вы можете использовать это удостоверение в [триггерах и действиях, поддерживающих управляемые удостоверения](logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

> [!IMPORTANT]
> Если у вас есть функция Azure, в которой вы хотите использовать назначенное системой удостоверение, сначала [включите проверку подлинности для функций Azure](../logic-apps/logic-apps-azure-functions.md#enable-authentication-for-functions).

В этой пошаговой инструкции показано, как использовать управляемое удостоверение с триггером или действием через портал Azure. Сведения об указании управляемого удостоверения в базовом определении JSON триггера или действия см. в разделе [проверки подлинности управляемого удостоверения](../logic-apps/logic-apps-securing-a-logic-app.md#managed-identity-authentication).

1. Откройте приложение логики в конструкторе приложений логики на [портале Azure](https://portal.azure.com).

1. Если вы еще не сделали этого, добавьте [триггер или действие, поддерживающее управляемые удостоверения](logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

   > [!NOTE]
   > Не все триггеры и действия поддерживают добавление типа проверки подлинности. Дополнительные сведения см. в разделе [типы проверки подлинности для триггеров и действий, поддерживающих проверку подлинности](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

1. В добавленном триггере или действии выполните следующие действия.

   * **Встроенные триггеры и действия, поддерживающие использование управляемого удостоверения**

     1. Добавьте свойство **проверки подлинности** , если свойство еще не отображается.

     1. В разделе **тип проверки подлинности** выберите **управляемое удостоверение**.

     Дополнительные сведения см. в разделе [Пример: проверка подлинности встроенного триггера или действия с управляемым удостоверением](#authenticate-built-in-managed-identity).
 
   * **Триггеры управляемого соединителя и действия, поддерживающие использование управляемого удостоверения**

     1. На странице Выбор клиента выберите **подключиться с управляемым удостоверением**.

     1. На следующей странице Укажите имя подключения.

        По умолчанию в списке управляемых удостоверений отображается только управляемое удостоверение, так как приложение логики поддерживает включение только одного управляемого удостоверения за раз, например:

        ![Снимок экрана, на котором показана страница имени подключения и выбранное управляемое удостоверение.](./media/create-managed-service-identity/system-assigned-managed-identity.png)

     Дополнительные сведения см. в статье [пример. Проверка подлинности управляемого соединителя триггера или действия с управляемым удостоверением](#authenticate-managed-connector-managed-identity).

     Подключения, создаваемые для использования управляемого удостоверения, являются специальным типом соединения, который работает только с управляемым удостоверением. Во время выполнения подключение использует управляемое удостоверение, включенное в приложении логики. Эта конфигурация сохраняется в объекте определения ресурса приложения логики `parameters` , который содержит `$connections` объект, который содержит указатели на идентификатор ресурса подключения, а также идентификатор ресурса удостоверения, если разрешено назначенное пользователем удостоверение.

     В этом примере показано, как выглядит конфигурация, если приложение логики включает управляемое системой удостоверение:

     ```json
     "parameters": {
        "$connections": {
           "value": {
              "<action-name>": {
                 "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/connections/{connection-name}",
                 "connectionName": "{connection-name}",
                 "connectionProperties": {
                    "authentication": {
                       "type": "ManagedServiceIdentity"
                    }
                 },
                 "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{managed-connector-type}"
              }
           }
        }
     }
     ```

     В этом примере показано, как выглядит конфигурация, если приложение логики включает управляемое пользователем удостоверение:

     ```json
     "parameters": {
        "$connections": {
           "value": {
              "<action-name>": {
                 "connectionId": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/connections/{connection-name}",
                 "connectionName": "{connection-name}",
                 "connectionProperties": {
                    "authentication": {
                       "identity": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{resourceGroupName}/providers/microsoft.managedidentity/userassignedidentities/{managed-identity-name}",
                       "type": "ManagedServiceIdentity"
                    }
                 },
                 "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/{managed-connector-type}"
              }
           }
        }
     }
     ```

     Во время выполнения служба Logic Apps проверяет, настроено ли для триггера управляемого соединителя и действий в приложении логики использование управляемого удостоверения, и что все необходимые разрешения настроены на использование управляемого удостоверения для доступа к целевым ресурсам, заданным триггером и действиями. В случае успеха служба Logic Apps Извлекает маркер Azure AD, связанный с управляемым удостоверением, и использует это удостоверение для проверки подлинности доступа к целевому ресурсу и выполнения настроенной операции в триггере и действиях.

<a name="authenticate-built-in-managed-identity"></a>

#### <a name="example-authenticate-built-in-trigger-or-action-with-a-managed-identity"></a>Пример. Проверка подлинности встроенного триггера или действия с управляемым удостоверением

Триггер или действие HTTP могут использовать назначенное системой удостоверение, включенное для приложения логики. Как правило, триггер или действие HTTP используют эти свойства для указания ресурса или сущности, к которым требуется получить доступ:

| Свойство | Обязательно | Описание |
|----------|----------|-------------|
| **Метод** | Да | Метод HTTP, используемый операцией, которую необходимо выполнить |
| **URI** | Да | URL-адрес конечной точки для доступа к целевому ресурсу или сущности Azure. Синтаксис URI обычно включает в себя [идентификатор ресурса](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) для ресурса или службы Azure. |
| **Заголовки** | нет | Любые значения заголовка, которые требуется включить в исходящий запрос, например тип содержимого |
| **Запросы** | нет | Любые параметры запроса, которые требуется включить в запрос, например параметр для определенной операции или версия API для операции, которую требуется выполнить. |
| **Аутентификация** | Да | Тип проверки подлинности, используемый для аутентификации доступа к целевому ресурсу или сущности |
||||

В качестве конкретного примера предположим, что вы хотите выполнить операцию [со снимком BLOB-объекта](/rest/api/storageservices/snapshot-blob) на большом двоичном объекте в учетной записи хранилища Azure, где вы ранее настроили доступ к удостоверению. Однако [соединитель хранилища BLOB-объектов](/connectors/azureblob/) Azure в настоящее время не поддерживает эту операцию. Вместо этого операцию можно выполнить с помощью [действия HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action) или другой операции [REST API службы BLOB-объектов](/rest/api/storageservices/operations-on-blobs).

> [!IMPORTANT]
> Чтобы получить доступ к учетным записям хранения Azure за брандмауэром с помощью HTTP-запросов и управляемых удостоверений, убедитесь, что вы также настроили учетную запись хранения с [исключением, которое разрешает доступ доверенным службам от Майкрософт](../connectors/connectors-create-api-azureblobstorage.md#access-trusted-service).

Для запуска [операции снимка BLOB-объекта](/rest/api/storageservices/snapshot-blob) в действии HTTP указываются следующие свойства:

| Свойство | Обязательно | Пример значения | Описание |
|----------|----------|---------------|-------------|
| **Метод** | Да | `PUT`| Метод HTTP, используемый операцией снимка BLOB-объекта |
| **URI** | Да | `https://{storage-account-name}.blob.core.windows.net/{blob-container-name}/{folder-name-if-any}/{blob-file-name-with-extension}` | Идентификатор ресурса для файла хранилища BLOB-объектов Azure в глобальной (общедоступной) среде Azure, который использует этот синтаксис |
| **Заголовки** | Для службы хранилища Azure | `x-ms-blob-type` = `BlockBlob` <p>`x-ms-version` = `2019-02-02` <p>`x-ms-date` = `@{formatDateTime(utcNow(),'r'}` | `x-ms-blob-type` `x-ms-version` Значения заголовка, и `x-ms-date` необходимы для операций службы хранилища Azure. <p><p>**Важно!** В исходящих HTTP-триггерах и запросах действий для службы хранилища Azure для заголовка требуется свойство `x-ms-version` и версия API для операции, которую требуется выполнить. Значение `x-ms-date` должно быть текущей датой. В противном случае приложение логики завершится ошибкой `403 FORBIDDEN` . Чтобы получить текущую дату в требуемом формате, можно использовать выражение в примере значения. <p>Дополнительные сведения см. в следующих статьях: <p><p>- [Заголовок запроса — снимок BLOB-объекта](/rest/api/storageservices/snapshot-blob#request) <br>- [Управление версиями для служб хранилища Azure](/rest/api/storageservices/versioning-for-the-azure-storage-services#specifying-service-versions-in-requests) |
| **Запросы** | Только для операции моментального снимка BLOB | `comp` = `snapshot` | Имя параметра запроса и значение для операции. |
|||||

Ниже приведен пример действия HTTP, в котором показаны все эти значения свойств.

![Добавьте действие HTTP для доступа к ресурсу Azure](./media/create-managed-service-identity/http-action-example.png)

1. После добавления действия HTTP добавьте свойство **authentication** в действие HTTP. Из списка **Добавить новый параметр** выберите **Проверка подлинности**.

   ![Добавьте свойство "Проверка подлинности" в действие HTTP.](./media/create-managed-service-identity/add-authentication-property.png)

   > [!NOTE]
   > Не все триггеры и действия поддерживают добавление типа проверки подлинности. Дополнительные сведения см. в разделе [типы проверки подлинности для триггеров и действий, поддерживающих проверку подлинности](../logic-apps/logic-apps-securing-a-logic-app.md#authentication-types-supported-triggers-actions).

1. В списке **Проверка подлинности** выберите **Управляемое удостоверение**.

   ![В списке "Проверка подлинности" выберите "Управляемое удостоверение".](./media/create-managed-service-identity/select-managed-identity.png)

1. В списке управляемых удостоверений выберите необходимые параметры из списка доступных для нужд вашего сценария.

   * Если вы настроили назначенное системой удостоверение, выберите пункт **Назначенное системой управляемое удостоверение**, если он еще не выбран.

     ![Выберите "Управляемое удостоверение, назначенное системой"](./media/create-managed-service-identity/select-system-assigned-identity-for-action.png)

   * Если вы настроили назначенное пользователем удостоверение, выберите это удостоверение, если оно еще не выбрано.

     ![Выберите назначаемое пользователем удостоверение](./media/create-managed-service-identity/select-user-assigned-identity-for-action.png)

   Этот пример продолжится с **управляемым удостоверением, назначаемым системой**.

1. Для некоторых триггеров и действий также отображается свойство **Аудитория** для задания идентификатора целевого ресурса. Задайте для свойства **аудитории** значение [идентификатора ресурса](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) целевого ресурса или службы. В противном случае по умолчанию свойство **аудитории** будет использовать идентификатор ресурса `https://management.azure.com/`, который представляет собой идентификатор ресурса для Azure Resource Manager.

   > [!IMPORTANT]
   > Убедитесь, что значение идентификатора ресурса *в точности совпадает* со значением, ожидаемым Azure AD, включая все необходимые знаки косой черты. Например, для идентификатора ресурса для всех учетных записей хранилища BLOB-объектов Azure требуется завершающая косая черта. Но для идентификатора ресурса конкретной учетной записи хранения завершающая косая черта не требуется. Сведения о том, как найти эти идентификаторы ресурсов, см. в разделе [Службы Azure, которые поддерживают Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

   В этом примере свойству **аудитории** присваивается значение `https://storage.azure.com/`, чтобы маркеры доступа, используемые для проверки подлинности, были допустимы для всех учетных записей хранения. Однако можно также указать URL-адрес корневой службы для конкретной учетной записи хранения, `https://fabrikamstorageaccount.blob.core.windows.net`.

   ![Задание для свойства "Аудитория" идентификатора целевого ресурса](./media/create-managed-service-identity/specify-audience-url-target-resource.png)

   Дополнительные сведения об авторизации доступа с помощью Azure AD для службы хранилища Azure см. в следующих разделах:

   * [Авторизация доступа к BLOB-объектам и очередям с помощью Azure Active Directory](../storage/common/storage-auth-aad.md)
   * [Авторизация доступа к хранилищу Azure с помощью Azure Active Directory](/rest/api/storageservices/authorize-with-azure-active-directory#use-oauth-access-tokens-for-authentication)

1. Продолжите создание приложения логики нужным образом.

<a name="authenticate-managed-connector-managed-identity"></a>

#### <a name="example-authenticate-managed-connector-trigger-or-action-with-a-managed-identity"></a>Пример. Проверка подлинности управляемого соединителя триггера или действия с управляемым удостоверением

Действие Azure Resource Manager **чтение ресурса** может использовать управляемое удостоверение, включенное для приложения логики. В этом примере показано, как использовать управляемое системой удостоверение.

1. После добавления действия в рабочий процесс на странице Выбор клиента выберите **подключиться с управляемым удостоверением**.

   ![Снимок экрана, на котором показано Azure Resource Manager действие и выбран параметр "подключиться с управляемым удостоверением".](./media/create-managed-service-identity/select-connect-managed-identity.png)

   Теперь действие отображает страницу имя подключения с управляемым списком удостоверений, включающим тип управляемого удостоверения, который в настоящее время включен в приложении логики.

1. На странице имя соединения укажите имя соединения. В списке управляемое удостоверение выберите управляемое удостоверение, которому **назначено управляемое системой удостоверение** в этом примере, и нажмите кнопку **создать**. Если вы включили управляемое удостоверение, назначенное пользователем, выберите это удостоверение.

   ![Снимок экрана, показывающий Azure Resource Manager действия с указанным именем подключения и выбранным параметром "управляемое удостоверение, назначенное системой".](./media/create-managed-service-identity/system-assigned-managed-identity.png)

   Если управляемое удостоверение не включено, при попытке создать подключение появляется следующая ошибка:

   *Необходимо включить управляемое удостоверение для приложения логики, а затем предоставить требуемый доступ к удостоверению в целевом ресурсе.*

   ![Снимок экрана, на котором показано Azure Resource Manager действие с ошибкой, если управляемое удостоверение не включено.](./media/create-managed-service-identity/system-assigned-managed-identity-disabled.png)

1. После успешного создания соединения конструктор может получить любые динамические значения, содержимое или схему, используя проверку подлинности с помощью управляемого удостоверения.

1. Продолжите создание приложения логики нужным образом.

<a name="remove-identity"></a>

## <a name="disable-managed-identity"></a>Отключение управляемого удостоверения

Чтобы прекратить использование управляемого удостоверения в приложении логики, можно воспользоваться следующими вариантами действий.

* [Портал Azure](#azure-portal-disable)
* [Шаблоны диспетчера ресурсов Azure](#template-disable)
* Azure PowerShell
  * [Удаление назначений ролей](../role-based-access-control/role-assignments-powershell.md)
  * [Удаление назначаемого пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md)
* Azure CLI
  * [Удаление назначений ролей](../role-based-access-control/role-assignments-cli.md)
  * [Удаление назначаемого пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md)
* REST API Azure
  * [Удаление назначений ролей](../role-based-access-control/role-assignments-rest.md)
  * [Удаление назначаемого пользователем удостоверения](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-rest.md)

При удалении приложения логики Azure автоматически удаляет присвоенный системой идентификатор приложения логики из Azure AD.

<a name="azure-portal-disable"></a>

### <a name="disable-managed-identity-in-the-azure-portal"></a>Отключение управляемого удостоверения на портале Azure

На портале Azure сначала удалите доступ удостоверения к [целевому ресурсу](#disable-identity-target-resource). Затем отключите назначенное системой удостоверение или удалите назначаемое пользователем удостоверение из [приложения логики](#disable-identity-logic-app).

<a name="disable-identity-target-resource"></a>

#### <a name="remove-identity-access-from-resources"></a>Удаление доступа удостоверения к ресурсам

1. На [портале Azure](https://portal.azure.com) перейдите к ресурсу Azure, у которого вы хотите удалить доступ для управляемого удостоверения.

1. В меню целевого ресурса выберите **Управление доступом (IAM)** . Выберите под панелью инструментов пункт **Назначение ролей**.

1. В списке ролей выберите управляемые удостоверения, которые требуется удалить. На панели инструментов нажмите кнопку **Удалить**.

   > [!TIP]
   > Если кнопка **Удалить** недоступна, скорее всего, у вас нет разрешений. Дополнительные сведения о различных ролях администраторов см. в статье [Разрешения роли администратора в Azure Active Directory](../active-directory/roles/permissions-reference.md).

Управляемое удостоверение теперь удалено и больше не имеет доступа к целевому ресурсу.

<a name="disable-identity-logic-app"></a>

#### <a name="disable-managed-identity-on-logic-app"></a>Отключение управляемого удостоверения в приложении логики

1. На [портале Azure](https://portal.azure.com) откройте приложение логики в конструкторе приложений логики.

1. В меню приложения логики в разделе **Параметры** выберите пункт **Удостоверение**, затем выполните следующее.

   * Выберите **Назначенное системой** > **Включено** > **Сохранить**. Когда Azure запросит подтверждение, выберите **Да**.

     ![Отключение назначаемого системой удостоверения](./media/create-managed-service-identity/disable-system-assigned-identity.png)

   * Выберите **Назначаемое пользователем** и нужное управляемое удостоверение, а затем нажмите **Удалить**. Когда Azure запросит подтверждение, выберите **Да**.

     ![Удаление назначаемого пользователем удостоверения](./media/create-managed-service-identity/remove-user-assigned-identity.png)

Управляемое удостоверение теперь отключено в приложении логики.

<a name="template-disable"></a>

### <a name="disable-managed-identity-in-azure-resource-manager-template"></a>Отключение управляемого удостоверения с помощью шаблона Resource Manager

Если вы создали управляемое удостоверение приложения логики с помощью шаблона развертывания Azure Resource Manager, задайте свойству `type` элемента `identity` значение `None`.

```json
"identity": {
   "type": "None"
}
```

## <a name="next-steps"></a>Дальнейшие действия

* [Безопасный доступ и данные в Azure Logic Apps](../logic-apps/logic-apps-securing-a-logic-app.md)
