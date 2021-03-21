---
title: включить файл
description: включить файл
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 08/26/2020
ms.author: rogara
ms.custom: include file
ms.openlocfilehash: 4773446ec0007ffbed99bc01939d1f92f5823d99
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95557331"
---
## <a name="assign-access-permissions-to-an-identity"></a>Назначение разрешений на доступ для удостоверения

Чтобы получить доступ к ресурсам службы файлов Azure с проверкой подлинности на основе удостоверений, удостоверение (пользователь, группа или субъект-служба) должно иметь необходимые разрешения на уровне общего ресурса. Этот процесс аналогичен указанию разрешений для общей папки Windows, где указывается тип доступа, который определенный пользователь имеет в общей папке. В инструкциях в этом разделе описывается, как назначить для удостоверения права доступа на чтение, запись или удаление к файловому ресурсу. 

Мы предоставили три встроенные роли Azure для предоставления пользователям разрешений на уровне общего доступа:

- **Файл хранилища. средство чтения общего ресурса SMB** предоставляет доступ на чтение в файловых ресурсах службы хранилища Azure через SMB.
- **Участник общей папки SMB данных файлов хранилища** предоставляет доступ на чтение, запись и удаление в общих файловых ресурсах службы хранилища Azure через SMB.
- **Участник с повышенными правами общего ресурса SMB для данных файлов хранилища** позволяет читать, записывать, удалять и изменять разрешения NTFS в общих файловых ресурсах службы хранилища Azure по протоколу SMB.

> [!IMPORTANT]
> Полный административный контроль над файловым ресурсом, включая возможность стать владельцем файла, требует использования ключа учетной записи хранения. Административный элемент управления не поддерживается при использовании учетных данных Azure AD.

Вы можете использовать портал Azure, PowerShell или Azure CLI, чтобы назначить встроенные роли удостоверению пользователя Azure AD для предоставления разрешений на уровне общего доступа. Учтите, что для назначения роли Azure уровня общего ресурса может потребоваться некоторое время. 

> [!NOTE]
> Не забудьте [синхронизировать учетные данные AD DS с Azure AD](../articles/active-directory/hybrid/how-to-connect-install-roadmap.md) , если вы планируете использовать локальную AD DS для проверки подлинности. Синхронизация хэша паролей из AD DS Azure AD не является обязательной. Разрешение на уровне общего ресурса будет предоставлено удостоверению Azure AD, которое синхронизируется с локальной AD DS.

Общая рекомендация заключается в использовании разрешения уровня общего доступа для управления доступом высокого уровня к группе AD, представляющей группу пользователей и удостоверений, а затем используйте разрешения NTFS для детального контроля доступа на уровне каталога/файла. 

### <a name="assign-an-azure-role-to-an-ad-identity"></a>Назначение роли Azure удостоверению AD

# <a name="portal"></a>[Портал](#tab/azure-portal)
Чтобы назначить роль Azure удостоверению Azure AD, используйте [портал Azure](https://portal.azure.com)выполните следующие действия.

1. В портал Azure перейдите к общей папке или [Создайте общую папку](../articles/storage/files/storage-how-to-create-file-share.md).
2. Выберите **Управление доступом (IAM)** .
3. Выберите **добавить назначение роли**
4. В колонке **Добавление назначения роли** выберите подходящую встроенную роль (средство чтения общих ресурсов SMB для данных файлов хранилища, участника общего ресурса SMB данных файлов хранилища) из списка **ролей** . Оставьте значение параметра **назначить доступ** в параметре по умолчанию: **пользователь, группа или субъект-служба Azure AD**. Выберите целевое удостоверение Azure AD по имени или адресу электронной почты.
5. Нажмите кнопку **сохранить** , чтобы завершить операцию назначения роли.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

В следующем примере PowerShell показано, как назначить роль Azure удостоверению Azure AD на основе имени для входа. Дополнительные сведения о назначении ролей Azure с помощью PowerShell см. в статье [Управление доступом с использованием RBAC и Azure PowerShell](../articles/role-based-access-control/role-assignments-powershell.md).

Перед запуском следующего примера сценария не забудьте заменить значения заполнителей, включая квадратные скобки, собственными значениями.

```powershell
#Get the name of the custom role
$FileShareContributorRole = Get-AzRoleDefinition "<role-name>" #Use one of the built-in roles: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
#Constrain the scope to the target file share
$scope = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
#Assign the custom role to the target identity with the specified scope.
New-AzRoleAssignment -SignInName <user-principal-name> -RoleDefinitionName $FileShareContributorRole.Name -Scope $scope
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
  
Следующая команда CLI 2,0 показывает, как назначить роль Azure удостоверению Azure AD на основе имени для входа. Дополнительные сведения о назначении ролей Azure с Azure CLI см. в разделе [Управление доступом с помощью RBAC и Azure CLI](../articles/role-based-access-control/role-assignments-cli.md). 

Перед запуском следующего примера сценария не забудьте заменить значения заполнителей, включая квадратные скобки, собственными значениями.

```azurecli-interactive
#Assign the built-in role to the target identity: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
az role assignment create --role "<role-name>" --assignee <user-principal-name> --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
```
---

## <a name="configure-ntfs-permissions-over-smb"></a>Настройка разрешений NTFS по протоколу SMB

После назначения разрешений на уровне общего ресурса с помощью RBAC необходимо назначить соответствующие разрешения NTFS на уровне корневой папки, каталога или на уровне файла. Разрешения уровня общего доступа можно рассматривать как привратник высокого уровня, который определяет, может ли пользователь получить доступ к общей папке. В то время как разрешения NTFS работают на более детализированном уровне, чтобы определить, какие операции пользователь может выполнять на уровне каталога или файла.

Служба файлов Azure поддерживает полный набор основных и дополнительных разрешений NTFS. Вы можете просматривать и настраивать разрешения NTFS для каталогов и файлов в файловом ресурсе Azure, подключив общую папку, а затем используя проводник Windows или выполнив команду Windows [icacls](/windows-server/administration/windows-commands/icacls) или [Set-ACL](/powershell/module/microsoft.powershell.security/set-acl) . 

Чтобы настроить NTFS с разрешениями суперпользователя, необходимо подключить общий ресурс с помощью ключа учетной записи хранения на виртуальной машине, присоединенной к домену. Следуйте инструкциям в следующем разделе, чтобы подключить файловый ресурс Azure из командной строки и соответствующим образом настроить разрешения NTFS.

На уровне корневой папки файлового ресурса поддерживается следующий набор разрешений:

- BUILTIN\Administrators:(OI)(CI)(F);
- NT AUTHORITY\SYSTEM:(OI)(CI)(F);
- BUILTIN\Users:(RX);
- BUILTIN\Users:(OI)(CI)(IO)(GR,GE);
- NT AUTHORITY\Authenticated Users:(OI)(CI)(M);
- NT AUTHORITY\SYSTEM:(F);
- CREATOR OWNER:(OI)(CI)(IO)(F).

### <a name="mount-a-file-share-from-the-command-prompt"></a>Подключение файлового ресурса Azure из командной строки

Используйте команду Windows **net use** для подключения файлового ресурса Azure. Не забудьте заменить значения заполнителей в следующем примере собственными значениями. Дополнительные сведения о подключении общих файловых ресурсов см. [в статье использование файлового ресурса Azure с Windows](../articles/storage/files/storage-how-to-use-files-windows.md). 

```
$connectTestResult = Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded)
{
 net use <desired-drive letter>: \\<storage-account-name>.file.core.windows.net\<fileshare-name>
} 
else 
{
 Write-Error -Message "Unable to reach the Azure storage account via port 445. Check to make sure your organization or ISP is not blocking port 445, or use Azure P2S VPN, Azure S2S VPN, or Express Route to tunnel SMB traffic over a different port."
}

```

Если при подключении к службе файлов Azure возникают проблемы, обратитесь к [средству устранения неполадок, опубликованному для ошибок подключения к службе файлов Azure в Windows](https://azure.microsoft.com/blog/new-troubleshooting-diagnostics-for-azure-files-mounting-errors-on-windows/). Мы также предоставляем [рекомендации](../articles/storage/files/storage-files-faq.md#on-premises-access) по обойти сценарии, когда порт 445 заблокирован. 


### <a name="configure-ntfs-permissions-with-windows-file-explorer"></a>Настройка разрешений NTFS с помощью проводника файлов Windows

Используйте проводник файлов Windows, чтобы предоставить полный доступ ко всем каталогам и файлам в общей папке, включая корневой каталог.

1. Откройте проводник Windows и щелкните правой кнопкой мыши файл или каталог и выберите пункт **Свойства**.
2. Перейдите на вкладку **Безопасность**.
3. Нажмите кнопку **изменить.** для изменения разрешений.
4. Можно изменить разрешения существующих пользователей или нажать кнопку **Добавить...** , чтобы предоставить разрешения новым пользователям.
5. В окне запроса для добавления новых пользователей введите имя целевого пользователя, которому нужно предоставить разрешение, в поле **Введите имена объектов** и выберите **Проверить имена** , чтобы найти полное имя участника-пользователя.
7.    Щелкните **ОК**.
8.    На вкладке **Безопасность** выберите все разрешения, которые вы хотите предоставить новому пользователю.
9.    Нажмите кнопку **Применить**.

### <a name="configure-ntfs-permissions-with-icacls"></a>Настройка разрешений NTFS с помощью icacls

Используйте следующую команду Windows, чтобы предоставить полный набор разрешений для всех каталогов и файлов в файловом ресурсе, включая корневую папку. Не забудьте заменить значения заполнителей в примере собственными значениями.

```
icacls <mounted-drive-letter>: /grant <user-email>:(f)
```

Дополнительные сведения об использовании icacls для установки разрешений NTFS и о различных типах поддерживаемых разрешений см. [в справочнике по командной строке для icacls](/windows-server/administration/windows-commands/icacls).

## <a name="mount-a-file-share-from-a-domain-joined-vm"></a>Подключение файлового ресурса с виртуальной машины, присоединенной к домену

Следующий процесс проверяет правильность настройки общей папки и разрешений на доступ, а также доступ к файловому ресурсу Azure с виртуальной машины, присоединенной к домену. Учтите, что для назначения роли Azure уровня общего ресурса может потребоваться некоторое время. 

Войдите на виртуальную машину с помощью удостоверения Azure AD, которому предоставлены разрешения, как показано на следующем рисунке. Если вы включили локальную проверку подлинности AD DS для службы файлов Azure, используйте учетные данные AD DS. Для аутентификации Azure AD DS выполните вход с использованием учетных данных Azure AD.

![Снимок экрана с окном входа в Azure AD для выполнения аутентификации пользователя](media/storage-files-aad-permissions-and-mounting/azure-active-directory-authentication-dialog.png)

Используйте следующую команду, чтобы подключить файловый ресурс Azure. Не забудьте заменить значения заполнителей собственными значениями. Так как вы прошли проверку подлинности, вам не нужно указывать ключ учетной записи хранения, локальный AD DS учетные данные или учетные данные AD DS Azure. Возможности единого входа поддерживаются для проверки подлинности с помощью локального AD DS или Azure AD DS. Если у вас возникли проблемы при подключении с учетными данными AD DS, см. инструкции по [устранению неполадок с файлами Azure в Windows](../articles/storage/files/storage-troubleshoot-windows-file-connection-problems.md) .

```
$connectTestResult = Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded)
{
 net use <desired-drive letter>: \\<storage-account-name>.file.core.windows.net\<fileshare-name>
} 
else 
{
 Write-Error -Message "Unable to reach the Azure storage account via port 445. Check to make sure your organization or ISP is not blocking port 445, or use Azure P2S VPN, Azure S2S VPN, or Express Route to tunnel SMB traffic over a different port."
}
```