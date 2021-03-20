---
title: Создание SAS делегирования пользователя для контейнера или большого двоичного объекта с помощью PowerShell
titleSuffix: Azure Storage
description: Узнайте, как создать SAS делегирования пользователя с учетными данными Azure Active Directory с помощью PowerShell.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2019
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 875b2a9f35562dd8f0d5df3c631e5ade1e3fbf75
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91714524"
---
# <a name="create-a-user-delegation-sas-for-a-container-or-blob-with-powershell"></a>Создание SAS для делегирования пользователя для контейнера или большого двоичного объекта с помощью PowerShell

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

В этой статье показано, как использовать учетные данные Azure Active Directory (Azure AD) для создания SAS делегирования пользователя для контейнера или большого двоичного объекта с Azure PowerShell.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="install-the-powershell-module"></a>Установка модуля PowerShell

Чтобы создать SAS для делегирования пользователей с помощью PowerShell, установите версию 1.10.0 или более позднюю версию модуля AZ. Storage. Чтобы установить последнюю версию модуля, выполните следующие действия.

1. Удалите все ранее установленные версии Azure PowerShell.

    - Удалите все предыдущие установки Azure PowerShell из Windows с помощью параметра **Apps & features** (Приложения и компоненты) в разделе **Параметры**.
    - Удалите все модули **Azure** из `%Program Files%\WindowsPowerShell\Modules` .

1. Убедитесь, что у вас установлена последняя версия PowerShellGet. Откройте окно Windows PowerShell и выполните следующую команду, чтобы установить последнюю версию:

    ```powershell
    Install-Module PowerShellGet –Repository PSGallery –Force
    ```

1. После установки PowerShellGet закройте и снова откройте окно PowerShell.

1. Установите Azure PowerShell последней версии:

    ```powershell
    Install-Module Az –Repository PSGallery –AllowClobber
    ```

1. Убедитесь, что установлен Azure PowerShell версии 3.2.0 или более поздней. Выполните следующую команду, чтобы установить последнюю версию модуля PowerShell для службы хранилища Azure:

    ```powershell
    Install-Module -Name Az.Storage -Repository PSGallery -Force
    ```

1. Закройте и снова откройте окно PowerShell.

Чтобы проверить, какая версия модуля AZ. Storage установлена, выполните следующую команду:

```powershell
Get-Module -ListAvailable -Name Az.Storage -Refresh
```

Дополнительные сведения об установке Azure PowerShell см. в статье [Установка Azure PowerShell с помощью PowerShellGet](/powershell/azure/install-az-ps).

## <a name="sign-in-to-azure-powershell-with-azure-ad"></a>Вход в Azure PowerShell с помощью Azure AD

Чтобы войти с помощью своей учетной записи Azure AD, вызовите команду [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) :

```powershell
Connect-AzAccount
```

Дополнительные сведения о входе в систему с помощью PowerShell см. [в разделе Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps).

## <a name="assign-permissions-with-azure-rbac"></a>Назначение разрешений с помощью Azure RBAC

Чтобы создать SAS для делегирования пользователей из Azure PowerShell, учетной записи Azure AD, используемой для входа в PowerShell, должна быть назначена роль, включающая действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** . Это разрешение позволяет этой учетной записи Azure AD запрашивать *ключ делегирования пользователя*. Ключ делегирования пользователя используется для подписи SAS делегирования пользователя. Роль, предоставляющая действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** , должна быть назначена на уровне учетной записи хранения, группы ресурсов или подписки. Дополнительные сведения о разрешениях RBAC Azure для создания SAS для делегирования пользователей см. в разделе **Назначение разрешений с помощью Azure RBAC** статьи [Создание SAS для делегирования пользователей](/rest/api/storageservices/create-user-delegation-sas).

Если у вас нет достаточных разрешений для назначения ролей Azure субъекту безопасности Azure AD, может потребоваться попросить владельца или администратора учетной записи назначить необходимые разрешения.

В следующем примере назначается роль **участника данных BLOB-объекта хранилища** , которая включает действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** . Роль ограничивается уровнем учетной записи хранения.

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Blob Data Contributor" `
    -Scope  "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

Дополнительные сведения о встроенных ролях, включающих действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** , см. в статье [встроенные роли Azure](../../role-based-access-control/built-in-roles.md).

## <a name="use-azure-ad-credentials-to-secure-a-sas"></a>Использование учетных данных Azure AD для защиты SAS

При создании SAS делегирования пользователя с Azure PowerShell ключ делегирования пользователя, используемый для подписи SAS, создается неявно. Время начала и окончания срока действия, заданное для SAS, также используются в качестве времени начала и срока действия для ключа делегирования пользователя. 

Так как максимальный интервал, по истечении которого ключ делегирования пользователя является допустимым, — 7 дней с даты начала, необходимо указать время окончания срока действия для SAS в течение 7 дней с момента запуска. После истечения срока действия ключа делегирования пользователя SAS является недействительным, поэтому срок действия SAS с временем окончания срока хранения более 7 дней будет действительным только в течение 7 дней.

Чтобы создать SAS делегирования пользователя для контейнера или большого двоичного объекта с Azure PowerShell, сначала создайте новый объект контекста службы хранилища Azure, указав `-UseConnectedAccount` параметр. `-UseConnectedAccount`Параметр указывает, что команда создает объект контекста в учетной записи Azure AD, с которой вы вошли.

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```powershell
$ctx = New-AzStorageContext -StorageAccountName <storage-account> -UseConnectedAccount
```

### <a name="create-a-user-delegation-sas-for-a-container"></a>Создание SAS для делегирования пользователя для контейнера

Чтобы вернуть маркер SAS делегирования пользователя для контейнера, вызовите команду [New-азсторажеконтаинерсастокен](/powershell/module/az.storage/new-azstoragecontainersastoken) , передав созданный ранее объект контекста службы хранилища Azure.

В следующем примере возвращается маркер SAS для делегирования пользователя для контейнера. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями:

```powershell
New-AzStorageContainerSASToken -Context $ctx `
    -Name <container> `
    -Permission racwdl `
    -ExpiryTime <date-time>
```

Возвращаемый токен SAS для делегирования пользователя будет выглядеть следующим образом:

```output
?sv=2018-11-09&sr=c&sig=<sig>&skoid=<skoid>&sktid=<sktid>&skt=2019-08-05T22%3A24%3A36Z&ske=2019-08-07T07%3A
00%3A00Z&sks=b&skv=2018-11-09&se=2019-08-07T07%3A00%3A00Z&sp=rwdl
```

### <a name="create-a-user-delegation-sas-for-a-blob"></a>Создание SAS делегирования пользователя для большого двоичного объекта

Чтобы вернуть маркер SAS для делегирования пользователя для большого двоичного объекта, вызовите команду [New-азсторажеблобсастокен](/powershell/module/az.storage/new-azstorageblobsastoken) , передав созданный ранее объект контекста службы хранилища Azure.

Следующий синтаксис возвращает SAS делегирования пользователя для большого двоичного объекта. В примере указывается `-FullUri` параметр, который ВОЗВРАЩАЕТ URI большого двоичного объекта с добавленным маркером SAS. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями:

```powershell
New-AzStorageBlobSASToken -Context $ctx `
    -Container <container> `
    -Blob <blob> `
    -Permission racwd `
    -ExpiryTime <date-time>
    -FullUri
```

Возвращаемый URI SAS для делегирования пользователя будет выглядеть примерно так:

```output
https://storagesamples.blob.core.windows.net/sample-container/blob1.txt?sv=2018-11-09&sr=b&sig=<sig>&skoid=<skoid>&sktid=<sktid>&skt=2019-08-06T21%3A16%3A54Z&ske=2019-08-07T07%3A00%3A00Z&sks=b&skv=2018-11-09&se=2019-08-07T07%3A00%3A00Z&sp=racwd
```

> [!NOTE]
> SAS пользователя, поддерживающий делегирование, не поддерживает определение разрешений с помощью хранимой политики доступа.

## <a name="revoke-a-user-delegation-sas"></a>Отзыв SAS для делегирования пользователя

Чтобы отозвать SAS для делегирования пользователя из Azure PowerShell, вызовите команду **REVOKE-азсторажеаккаунтусерделегатионкэйс** . Эта команда отменяет все ключи делегирования пользователя, связанные с указанной учетной записью хранения. Все подписанные URL, связанные с этими ключами, становятся недействительными.

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```powershell
Revoke-AzStorageAccountUserDelegationKeys -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
```

> [!IMPORTANT]
> Как ключ делегирования пользователя, так и назначение ролей Azure кэшируются службой хранилища Azure, поэтому при инициации процесса отзыва может возникнуть задержка, и если существующее сопоставление безопасности делегирования пользователя станет недействительным.

## <a name="next-steps"></a>Дальнейшие действия

- [Создание SAS для делегирования пользователей (REST API)](/rest/api/storageservices/create-user-delegation-sas)
- [Операция получения ключа делегирования пользователя](/rest/api/storageservices/get-user-delegation-key)
