---
title: Управление доступом к файловым ресурсам Azure — локальная проверка подлинности AD DS
description: Узнайте, как назначить разрешения удостоверению служб домен Active Directory, которое представляет вашу учетную запись хранения. Это позволяет управлять доступом с помощью проверки подлинности на основе удостоверений.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.openlocfilehash: af88f0b3403fb80acbb7dacebe293ac583e35799
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91716038"
---
# <a name="part-two-assign-share-level-permissions-to-an-identity"></a>Часть 2. Назначение удостоверениям разрешений на уровне общего ресурса

Перед началом работы с этой статьей убедитесь, что вы завершили предыдущую статью, [включите проверку подлинности AD DS для своей учетной записи](storage-files-identity-ad-ds-enable.md).

После включения проверки подлинности домен Active Directory Services (AD DS) в учетной записи хранения необходимо настроить разрешения на уровне общего ресурса, чтобы получить доступ к общим файловым ресурсам. Удостоверение, к которому необходимо получить доступ к ресурсам общей папки Azure, должно быть гибридным удостоверением, которое существует как в AD DS, так и в Azure AD. Например, предположим, что у вас есть пользователь в AD DS user1@onprem.contoso.com и вы выполнили синхронизацию с Azure AD в качестве user1@contoso.com Azure AD Connect синхронизации. Чтобы разрешить этому пользователю доступ к службе файлов Azure, необходимо назначить разрешения уровня общего доступа для user1@contoso.com . Та же концепция применима к группам или субъектам-службам. Поэтому необходимо синхронизировать пользователей и группы из AD DS с Azure AD с помощью Azure AD Connect Sync. 

Разрешения уровня общего доступа должны быть назначены удостоверению Azure AD, представляющему того же пользователя или группу в AD DS для поддержки AD DS проверки подлинности в общем файловом ресурсе Azure. Проверка подлинности и авторизация для удостоверений, которые существуют только в Azure AD, такие как управляемые удостоверения Azure (MSI), не поддерживаются при AD DSной проверке подлинности. В этой статье показано, как назначить удостоверению разрешения на уровне общего ресурса для общей папки.


## <a name="share-level-permissions"></a>Разрешения уровня общего доступа

Как правило, рекомендуется использовать разрешения уровня общего доступа для управления доступом высокого уровня к группе Azure AD, представляющей группу пользователей и удостоверений, а затем используя списки управления доступом Windows для детализированного контроля доступа на уровне каталога или файла. 

Существует три встроенных роли Azure для предоставления пользователям разрешений на уровне общего доступа:

- **Файл хранилища. средство чтения общего ресурса SMB** предоставляет доступ на чтение в файловых ресурсах службы хранилища Azure через SMB.
- **Участник общей папки SMB данных файлов хранилища** предоставляет доступ на чтение, запись и удаление в общих файловых ресурсах службы хранилища Azure через SMB.
- **Участник хранилища данных файлового ресурса SMB с повышенными правами** позволяет читать, записывать, удалять и изменять списки управления доступом Windows в общих файловых ресурсах службы хранилища Azure через SMB.

> [!IMPORTANT]
> Полный административный контроль над файловым ресурсом, включая возможность стать владельцем файла, требует использования ключа учетной записи хранения. Административный элемент управления не поддерживается при использовании учетных данных Azure AD.

Вы можете использовать портал Azure, Azure PowerShell или Azure CLI, чтобы назначить встроенные роли удостоверению пользователя Azure AD для предоставления разрешений на уровне общего доступа.

## <a name="assign-an-azure-role"></a>Назначение роли Azure

### <a name="azure-portal"></a>Портал Azure

Чтобы назначить роль Azure удостоверению Azure AD, используйте [портал Azure](https://portal.azure.com)выполните следующие действия.

1. В портал Azure перейдите к общей папке или [Создайте общую папку](storage-how-to-create-file-share.md).
1. Выберите **Управление доступом (IAM)** .
1. Выберите **добавить назначение роли**
1. В колонке **Добавление назначения роли** выберите подходящую встроенную роль (средство чтения общих ресурсов SMB для данных файлов хранилища, участника общего ресурса SMB данных файлов хранилища) из списка **ролей** . Оставьте значение параметра **назначить доступ** в параметре по умолчанию: **пользователь, группа или субъект-служба Azure AD**. Выберите целевое удостоверение Azure AD по имени или адресу электронной почты. **Выбранное удостоверение Azure AD должно быть гибридным удостоверением и не может быть удостоверением только для облака.** Это означает, что одно и то же удостоверение также представлено в AD DS.
1. Нажмите кнопку **сохранить** , чтобы завершить операцию назначения роли.

### <a name="powershell"></a>PowerShell

В следующем примере PowerShell показано, как назначить роль Azure удостоверению Azure AD на основе имени для входа. Дополнительные сведения о назначении ролей Azure с помощью PowerShell см. [в статье Добавление и удаление назначений ролей Azure с использованием модуля Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

Перед запуском следующего примера скрипта замените значения заполнителей, включая квадратные скобки, значениями.

```powershell
#Get the name of the custom role
$FileShareContributorRole = Get-AzRoleDefinition "<role-name>" #Use one of the built-in roles: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
#Constrain the scope to the target file share
$scope = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
#Assign the custom role to the target identity with the specified scope.
New-AzRoleAssignment -SignInName <user-principal-name> -RoleDefinitionName $FileShareContributorRole.Name -Scope $scope
```

### <a name="cli"></a>CLI
  
Следующая команда CLI 2,0 назначает роль Azure удостоверению Azure AD на основе имени для входа. Дополнительные сведения о назначении ролей Azure с Azure CLI см. [в разделе Добавление и удаление назначений ролей Azure с помощью Azure CLI](../../role-based-access-control/role-assignments-cli.md). 

Перед запуском следующего примера сценария не забудьте заменить значения заполнителей, включая квадратные скобки, собственными значениями.

```azurecli-interactive
#Assign the built-in role to the target identity: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
az role assignment create --role "<role-name>" --assignee <user-principal-name> --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы назначили разрешения уровня общего доступа, необходимо настроить разрешения на уровне каталога и файла. Перейдите к следующей статье.

[Часть 3. Настройка разрешений на уровне каталога и файлов по протоколу SMB](storage-files-identity-ad-ds-configure-permissions.md)
