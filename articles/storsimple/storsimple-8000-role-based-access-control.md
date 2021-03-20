---
title: Использование управления доступом на основе ролей Azure для StorSimple | Документация Майкрософт
description: В этой статье описывается, как использовать управление доступом на основе ролей Azure (Azure RBAC) в контексте StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: jconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/11/2017
ms.author: alkohli
ms.openlocfilehash: 49c38e23ddbbfe983ff82ad25363c744292d4d69
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92518982"
---
# <a name="azure-role-based-access-control-for-storsimple"></a>Управление доступом на основе ролей в Azure для StorSimple

В этой статье приводится краткое описание использования управления доступом на основе ролей Azure (Azure RBAC) для устройства StorSimple. Azure RBAC обеспечивает детальное управление доступом для Azure. Используйте Azure RBAC, чтобы предоставить доступ к пользователям StorSimple только для выполнения своих заданий, вместо того чтобы предоставлять всем неограниченный доступ. Дополнительные сведения об основах управления доступом в Azure см. в статье [что такое управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md).

Эта статья относится к устройствам StorSimple серии 8000 с обновлением 3.0 или более поздней версии на портале Azure.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="azure-roles-for-storsimple"></a>Роли Azure для StorSimple

Azure RBAC можно назначить на основе ролей. Роли предоставляют определенные уровни разрешений, исходя из доступных ресурсов в среде. Существует два типа ролей, которые пользователи StorSimple могут выбрать: встроенная и пользовательская.

* **Встроенные роли**. Доступны такие встроенные роли, как владелец, участник, читатель или администратор доступа пользователей. Дополнительные сведения см. [в статье встроенные роли для управления доступом на основе ролей в Azure](../role-based-access-control/built-in-roles.md).

* **Пользовательские роли** . Если встроенные роли не соответствуют вашим потребностям, можно создать пользовательские роли Azure для StorSimple. Чтобы создать настраиваемую роль Azure, начните с встроенной роли, измените ее, а затем импортируйте обратно в среду. Скачивание и передача роли осуществляется с помощью Azure PowerShell или интерфейса командной строки Azure. Дополнительные сведения см. в статье [Создание пользовательских ролей для управления доступом на основе ролей в Azure](../role-based-access-control/custom-roles.md).

Чтобы просмотреть роли, доступные для пользователя устройства StorSimple на портале Azure, войдите в службу диспетчера устройств StorSimple, а затем выберите **Управление доступом (IAM) > Роли**.


## <a name="create-a-custom-role-for-storsimple-infrastructure-administrator"></a>Создание пользовательской роли для администратора инфраструктуры StorSimple

В следующем примере мы начнем со встроенной роли **Читатель**, которая позволяет пользователям просматривать все области действия ресурсов, но не изменять или создавать их. Затем мы расширяем эту роль, чтобы создать новую настраиваемую роль Администратор инфраструктуры StorSimple. Эта роль назначается пользователям, которые могут управлять инфраструктурой для устройств StorSimple.

1. Запустите Windows PowerShell от имени администратора.

2. Войдите в Azure.

    `Connect-AzAccount`

3. Экспортируйте роль "Читатель" в качестве шаблона JSON на компьютер.

    ```powershell
    Get-AzRoleDefinition -Name "Reader"

    Get-AzRoleDefinition -Name "Reader" | ConvertTo-Json | Out-File C:\ssrbaccustom.json
    ```

4. Откройте JSON-файл в Visual Studio. Вы видите, что типичная роль Azure состоит из трех основных разделов: **Actions** **, No и** **AssignableScopes**.

    В разделе **Actions** перечислены все разрешенные операции этой роли. Каждое действие назначено из поставщика ресурсов. Для администратора инфраструктуры StorSimple используйте поставщик ресурсов `Microsoft.StorSimple`.

    Чтобы просмотреть все доступные и зарегистрированные в подписке поставщики ресурсов, используйте PowerShell.

    `Get-AzResourceProvider`

    Вы также можете проверить все доступные командлеты PowerShell для управления поставщиками ресурсов.

    В разделе **"** незащищенные" перечислены все действия, ограниченные для конкретной роли Azure. В этом примере нет запрещенных действий.
    
    В разделе **AssignableScopes** указаны идентификаторы подписок. Убедитесь, что роль Azure содержит явный идентификатор подписки, где она используется. Если указан неправильный идентификатор подписки, вы не сможете импортировать роль в подписке.

    Измените файл, учитывая предыдущие рекомендации.

    ```json
    {
        "Name":  "StorSimple Infrastructure Admin",
        "Id":  "<guid>",
        "IsCustom":  true,
        "Description":  "Lets you view everything, but not make any changes except for Clear alerts, Clear settings, install, download etc.",
        "Actions":  [
                        "Microsoft.StorSimple/managers/alerts/read",
                        "Microsoft.StorSimple/managers/devices/volumeContainers/read",
                        "Microsoft.StorSimple/managers/devices/jobs/read",
                        "Microsoft.StorSimple/managers/devices/alertSettings/read",
                        "Microsoft.StorSimple/managers/devices/alertSettings/write",
                        "Microsoft.StorSimple/managers/clearAlerts/action",
                        "Microsoft.StorSimple/managers/devices/networkSettings/read",
                        "Microsoft.StorSimple/managers/devices/publishSupportPackage/action",
                        "Microsoft.StorSimple/managers/devices/scanForUpdates/action",
                        "Microsoft.StorSimple/managers/devices/metrics/read"

                    ],
        "NotActions":  [

                    ],
        "AssignableScopes":  [
                                "/subscriptions/<subscription_ID>/"
                            ]
    }
    ```

6. Импортируйте настраиваемую роль Azure обратно в среду.

    `New-AzRoleDefinition -InputFile "C:\ssrbaccustom.json"`


Эта роль должна отобразиться в списке ролей в колонке **Управление доступом**.

![Просмотр ролей Azure](./media/storsimple-8000-role-based-access-control/rbac-role-types.png)

Дополнительную информацию см. в разделе [Пользовательские роли](../role-based-access-control/custom-roles.md).

### <a name="sample-output-for-custom-role-creation-via-the-powershell"></a>Пример выходных данных для создания пользовательской роли через PowerShell

```powershell
Connect-AzAccount
```

```Output
Environment           : AzureCloud
Account               : john.doe@contoso.com
TenantId              : <tenant_ID>
SubscriptionId        : <subscription_ID>
SubscriptionName      : Internal Consumption
CurrentStorageAccount :
```

```powershell
Get-AzRoleDefinition -Name "Reader"
```

```Output
Name             : Reader
Id               : <guid>
IsCustom         : False
Description      : Lets you view everything, but not make any changes.
Actions          : {*/read}
NotActions       : {}
AssignableScopes : {/}
```

```powershell
Get-AzRoleDefinition -Name "Reader" | ConvertTo-Json | Out-File C:\ssrbaccustom.json
New-AzRoleDefinition -InputFile "C:\ssrbaccustom.json"
```

```Output
Name             : StorSimple Infrastructure Admin
Id               : <tenant_ID>
IsCustom         : True
Description      : Lets you view everything, but not make any changes except for Clear alerts, Clear settings, install,
                   download etc.
Actions          : {Microsoft.StorSimple/managers/alerts/read,
                   Microsoft.StorSimple/managers/devices/volumeContainers/read,
                   Microsoft.StorSimple/managers/devices/jobs/read,
                   Microsoft.StorSimple/managers/devices/alertSettings/read...}
NotActions       : {}
AssignableScopes : {/subscriptions/<subscription_ID>/}
```

## <a name="add-users-to-the-custom-role"></a>Добавление пользователей в пользовательскую роль

Вы можете предоставить доступ в рамках ресурса, группы ресурсов или подписки, которые входят в область назначения роли. При предоставлении доступа учитывайте, что права доступа, предоставленные в родительском узле, наследуются дочерним узлом. Дополнительные сведения см. в подразделах [Управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md).

1. Перейдите в раздел **Управление доступом (IAM)**. В колонке "Управление доступом" щелкните **+ Добавить**.

    ![Добавление доступа к роли Azure](./media/storsimple-8000-role-based-access-control/rbac-add-role.png)

2. Выберите роль, которую необходимо назначить. В данном случае это роль **администратора инфраструктуры StorSimple**.

3. В каталоге выберите пользователя, группу или приложение, которым нужно предоставить доступ. Поиск в каталоге можно выполнить по отображаемым именам, адресам электронной почты и идентификаторам объектов.

4. Нажмите кнопку **Сохранить**, чтобы создать назначение.

    ![Добавление разрешений в роль Azure](./media/storsimple-8000-role-based-access-control/rbac-create-role-infra-admin.png)

Уведомление **Добавление пользователя** показывает ход выполнения. После успешного добавления пользователя список пользователей в колонке управления доступом обновляется.

## <a name="view-permissions-for-the-custom-role"></a>Просмотр разрешений для пользовательской роли

После создания этой роли разрешения, связанные с ней, можно просмотреть на портале Azure.

1. Чтобы просмотреть разрешения, связанные с этой ролью, перейдите в раздел **Управление доступом (IAM) > роли > администратор инфраструктуры StorSimple**. Отобразится список пользователей в этой роли.

2. Выберите администратора инфраструктуры StorSimple и нажмите кнопку **Permissions** (Разрешения).

    ![Просмотр разрешений для роли администратора инфраструктуры StorSimple](./media/storsimple-8000-role-based-access-control/rbac-roles-view-permissions.png)

3. Отображаются разрешения, связанные с этой ролью.

    ![Просмотр пользователей в роли администратора инфраструктуры StorSimple](./media/storsimple-8000-role-based-access-control/rbac-infra-admin-permissions1.png)


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Общие сведения об управлении доступом на основе ролей](../role-based-access-control/role-assignments-external-users.md).
