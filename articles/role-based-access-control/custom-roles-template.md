---
title: Создание или обновление пользовательских ролей Azure с помощью шаблона Azure Resource Manager — Azure RBAC
description: Узнайте, как создать или обновить пользовательские роли Azure с помощью шаблона Azure Resource Manager (шаблон ARM) и управления доступом на основе ролей Azure (Azure RBAC).
services: role-based-access-control,azure-resource-manager
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 12/16/2020
ms.author: rolyon
ms.openlocfilehash: 0626a9e36d05ac9cb51f62652dbe6f3133bbc6d7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101095907"
---
# <a name="create-or-update-azure-custom-roles-using-an-arm-template"></a>Создание или обновление пользовательских ролей Azure с помощью шаблона ARM

Если [встроенные роли Azure](built-in-roles.md) не отвечают конкретным потребностям Организации, можно создать собственные [пользовательские роли](custom-roles.md). В этой статье описывается создание или обновление пользовательской роли с помощью шаблона Azure Resource Manager (шаблон ARM).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Чтобы создать пользовательскую роль, необходимо указать имя роли, разрешения и место, где можно использовать эту роль. В этой статье вы создадите роль _Custom Role-RG читатель_ с разрешениями ресурсов, которые могут быть назначены в области действия подписки или ниже.

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsubscription-deployments%2Fcreate-role-def%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Чтобы создать настраиваемую роль, необходимо иметь следующее:

- Разрешения на создание пользовательских ролей, таких как [владелец](built-in-roles.md#owner) или [администратор доступа пользователей](built-in-roles.md#user-access-administrator).

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этой статье, относится к [шаблонам](https://azure.microsoft.com/resources/templates/create-role-def)быстрого запуска Azure. Шаблон содержит четыре параметра и раздел ресурсов. Четыре параметра:

- Массив действий со значением по умолчанию `["Microsoft.Resources/subscriptions/resourceGroups/read"]` .
- Массив `notActions` с пустым значением по умолчанию.
- Имя роли со значением по умолчанию `Custom Role - RG Reader` .
- Описание роли со значением по умолчанию `Subscription Level Deployment of a Role Definition` .

Областью, в которой можно назначить эту настраиваемую роль, назначается текущая подписка.

:::code language="json" source="~/quickstart-templates/subscription-deployments/create-role-def/azuredeploy.json":::

В шаблоне определен следующий ресурс:

- [Microsoft. Authorization/roleDefinitions](/azure/templates/Microsoft.Authorization/roleDefinitions)

## <a name="deploy-the-template"></a>Развертывание шаблона

Выполните следующие действия, чтобы развернуть предыдущий шаблон.

1. Войдите на [портал Azure](https://portal.azure.com).

1. Откройте Azure Cloud Shell для PowerShell.

1. Скопируйте и вставьте следующий скрипт в Cloud Shell.

    ```azurepowershell-interactive
    $location = Read-Host -Prompt "Enter a location (i.e. centralus)"
    [string[]]$actions = Read-Host -Prompt "Enter actions as a comma-separated list (i.e. action1,action2)"
    $actions = $actions.Split(',')
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/create-role-def/azuredeploy.json"
    New-AzDeployment -Location $location -TemplateUri $templateUri -actions $actions
    ```

1. Введите расположение для развертывания, например `centralus` .

1. Введите список действий для пользовательской роли в виде списка с разделителями-запятыми, например `Microsoft.Resources/resources/read,Microsoft.Resources/subscriptions/resourceGroups/read` .

1. При необходимости нажмите клавишу ВВОД, чтобы выполнить `New-AzDeployment` команду.

    Команда [New-аздеплоймент](/powershell/module/az.resources/new-azdeployment) развертывает шаблон для создания настраиваемой роли.

    Выходные данные должны иметь следующий вид.

    ```azurepowershell-interactive
    PS> New-AzDeployment -Location $location -TemplateUri $templateUri -actions $actions

    Id                      : /subscriptions/{subscriptionId}/providers/Microsoft.Resources/deployments/azuredeploy
    DeploymentName          : azuredeploy
    Location                : centralus
    ProvisioningState       : Succeeded
    Timestamp               : 6/25/2020 8:08:32 PM
    Mode                    : Incremental
    TemplateLink            :
                              Uri            : https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/subscription-deployments/create-role-def/azuredeploy.json
                              ContentVersion : 1.0.0.0

    Parameters              :
                              Name               Type                       Value
                              =================  =========================  ==========
                              actions            Array                      [
                                "Microsoft.Resources/resources/read",
                                "Microsoft.Resources/subscriptions/resourceGroups/read"
                              ]
                              notActions         Array                      []
                              roleName           String                     Custom Role - RG Reader
                              roleDescription    String                     Subscription Level Deployment of a Role Definition

    Outputs                 :
    DeploymentDebugLogLevel :
    ```

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

Выполните следующие действия, чтобы убедиться, что пользовательская роль создана.

1. Выполните команду [Get-азроледефинитион](/powershell/module/az.resources/get-azroledefinition) , чтобы получить список настраиваемой роли.

    ```azurepowershell-interactive
    Get-AzRoleDefinition "Custom Role - RG Reader" | ConvertTo-Json
    ```

    Выходные данные должны иметь следующий вид.

    ```azurepowershell-interactive
    {
      "Name": "Custom Role - RG Reader",
      "Id": "11111111-1111-1111-1111-111111111111",
      "IsCustom": true,
      "Description": "Subscription Level Deployment of a Role Definition",
      "Actions": [
        "Microsoft.Resources/resources/read",
        "Microsoft.Resources/subscriptions/resourceGroups/read"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/{subscriptionId}"
      ]
    }
    ```

1. В портал Azure откройте подписку.

1. В меню слева выберите **Управление доступом (IAM)**.

1. Перейдите на вкладку **роли** .

1. Задайте для списка **типов** значение **кустомроле**.

1. Убедитесь, что в списке указана **пользовательская роль читателя RG** .

   ![Новая настраиваемая роль в портал Azure](./media/custom-roles-template/custom-role-template-portal.png)

## <a name="update-a-custom-role"></a>Обновление пользовательской роли

Аналогично созданию пользовательской роли можно обновить существующую пользовательскую роль с помощью шаблона. Чтобы обновить пользовательскую роль, необходимо указать роль, которую требуется обновить.

Ниже приведены изменения, которые необходимо внести в предыдущий шаблон быстрого запуска для обновления пользовательской роли.

- Включите идентификатор роли в качестве параметра.
    ```json
        ...
        "roleDefName": {
          "type": "string",
          "metadata": {
            "description": "ID of the role definition"
          }
        ...
    ```

- Включите параметр идентификатора роли в определение роли.

    ```json
      ...
      "resources": [
        {
          "type": "Microsoft.Authorization/roleDefinitions",
          "apiVersion": "2018-07-01",
          "name": "[parameters('roleDefName')]",
          "properties": {
            ...
    ```

Ниже приведен пример развертывания шаблона.

```azurepowershell
$location = Read-Host -Prompt "Enter a location (i.e. centralus)"
[string[]]$actions = Read-Host -Prompt "Enter actions as a comma-separated list (i.e. action1,action2)"
$actions = $actions.Split(',')
$roleDefName = Read-Host -Prompt "Enter the role ID to update"
$templateFile = "rg-reader-update.json"
New-AzDeployment -Location $location -TemplateFile $templateFile -actions $actions -roleDefName $roleDefName
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить настраиваемую роль, выполните следующие действия.

1. Выполните следующую команду, чтобы удалить настраиваемую роль.

    ```azurepowershell-interactive
    Get-AzRoleDefinition -Name "Custom Role - RG Reader" | Remove-AzRoleDefinition
    ```

1. Введите **Y** , чтобы подтвердить, что вы хотите удалить настраиваемую роль.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о определениях ролей Azure](role-definitions.md)
- [Краткое руководство. Назначение роли Azure с помощью шаблона Azure Resource Manager](quickstart-role-assignments-template.md)
- [Документация по шаблонам ARM](../azure-resource-manager/templates/index.yml)
