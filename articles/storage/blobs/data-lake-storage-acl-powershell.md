---
title: Использование PowerShell для управления списками ACL в Azure Data Lake Storage 2-го поколения
description: Используйте командлеты PowerShell для управления списками управления доступом (ACL) в учетных записях хранения с включенным иерархическое пространством имен (HNS).
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 02/17/2021
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: dd522355d30564d84fec15bdc57c7397c1e6cfe4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104702547"
---
# <a name="use-powershell-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Использование PowerShell для управления списками ACL в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать PowerShell для получения, установки и обновления списков управления доступом к каталогам и файлам. 

Наследование ACL уже доступно для новых дочерних элементов, созданных в родительском каталоге. Но можно также добавлять, обновлять и удалять списки управления доступом для существующих дочерних элементов родительского каталога без необходимости вносить эти изменения отдельно для каждого дочернего элемента. 

[Справочные материалы](/powershell/module/Az.Storage/)  |  [Рекурсивные примеры ACL](https://recursiveaclpr.blob.core.windows.net/privatedrop/samplePS.ps1?sv=2019-02-02&st=2020-08-24T17%3A04%3A44Z&se=2021-08-25T17%3A04%3A00Z&sr=b&sp=r&sig=dNNKS%2BZcp%2F1gl6yOx6QLZ6OpmXkN88ZjBeBtym1Mejo%3D)  |  [Отправить отзыв](https://github.com/Azure/azure-powershell/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- В учетной записи хранения включено иерархическое пространство имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Azure CLI версии`2.6.0` или выше.

- Одно из следующих разрешений безопасности:

  - [Участник безопасности](../../role-based-access-control/overview.md#security-principal) подготовленного Azure Active Directory (AD), которому назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) в области целевого контейнера, родительской группы ресурсов или подписки.  

  - Пользователь-владелец целевого контейнера или каталога, к которому планируется применять параметры ACL. Чтобы настроить списки управления доступом рекурсивно, сюда входят все дочерние элементы в целевом контейнере или каталоге.
  
  - Ключ учетной записи хранения.

## <a name="install-the-powershell-module"></a>Установка модуля PowerShell

1. Убедитесь, что установленная версия PowerShell `5.1` или более поздняя, с помощью следующей команды.    

   ```powershell
   echo $PSVersionTable.PSVersion.ToString() 
   ```

   Сведения об обновлении версии PowerShell см. в разделе [обновление существующих Windows PowerShell](/powershell/scripting/install/installing-windows-powershell#upgrading-existing-windows-powershell) .

2. Установите **AZ. Storage** Module.

   ```powershell
   Install-Module Az.Storage -Repository PSGallery -Force  
   ```

   Дополнительные сведения об установке модулей PowerShell см. [в статье Установка модуля Azure PowerShell](/powershell/azure/install-az-ps) .

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Выберите способ, которым команды должны получать права доступа к учетной записи хранения. 

### <a name="option-1-obtain-authorization-by-using-azure-active-directory-ad"></a>Вариант 1. получение авторизации с помощью Azure Active Directory (AD)

> [!NOTE]
> Если вы используете Azure Active Directory (Azure AD) для авторизации доступа, убедитесь, что участнику безопасности назначена [роль владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Дополнительные сведения о применении разрешений ACL и их влиянии на их изменение см. в разделе  [модель контроля доступа в Azure Data Lake Storage 2-го поколения](./data-lake-storage-access-control-model.md).

При таком подходе система гарантирует, что ваша учетная запись пользователя имеет соответствующие разрешения на управление доступом на основе ролей Azure (Azure RBAC) и ACL.

1. Откройте командное окно Windows PowerShell, а затем войдите в подписку Azure с помощью `Connect-AzAccount` команды и следуйте инструкциям на экране.

   ```powershell
   Connect-AzAccount
   ```

2. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку на подписку учетной записи хранения, которая будет создавать каталоги и управлять ими в. В этом примере замените `<subscription-id>` значение ЗАПОЛНИТЕЛЯ идентификатором подписки.

   ```powershell
   Select-AzSubscription -SubscriptionId <subscription-id>
   ``` 

3. Получение контекста учетной записи хранения.

   ```powershell
   $ctx = New-AzStorageContext -StorageAccountName '<storage-account-name>' -UseConnectedAccount
   ```

### <a name="option-2-obtain-authorization-by-using-the-storage-account-key"></a>Вариант 2. получение авторизации с помощью ключа учетной записи хранения

При таком подходе система не проверяет разрешения Azure RBAC или ACL. Получите контекст учетной записи хранения с помощью ключа учетной записи.

```powershell
$ctx = New-AzStorageContext -StorageAccountName '<storage-account-name>' -StorageAccountKey '<storage-account-key>'
```

## <a name="get-acls"></a>Получение списков управления доступом

Получите список ACL для каталога или файла с помощью `Get-AzDataLakeGen2Item` командлета.

Этот пример возвращает список ACL корневого каталога **контейнера** , а затем ВЫВОДИТ список ACL на консоль.

```powershell
$filesystemName = "my-file-system"
$filesystem = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName
$filesystem.ACL
```

Этот пример возвращает список ACL для **каталога**, а затем ВЫВОДИТ список ACL на консоль.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$dir = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
```

Этот пример возвращает список ACL для **файла** , а затем ВЫВОДИТ список ACL на консоль.

```powershell
$filePath = "my-directory/upload.txt"
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath
$file.ACL
```

На следующем изображении показаны выходные данные после получения списка ACL для каталога.

![Получение выходных данных ACL для каталога](./media/data-lake-storage-directory-file-acl-powershell/get-acl.png)

В этом примере владелец имеет разрешения на чтение, запись и выполнение. Группа-владелец имеет разрешения только на чтение и выполнение. Дополнительные сведения о списках управления доступом в Azure Data Lake Storage 2-го поколения см. на [этой странице](data-lake-storage-access-control.md).

## <a name="set-acls"></a>Настройка списков управления доступом

При *задании* ACL **заменяется** весь список ACL, включая все его записи. Если вы хотите изменить уровень разрешений субъекта безопасности или добавить новый субъект безопасности в список управления доступом, не затрагивая другие существующие записи, следует *Обновить* список управления доступом. Чтобы обновить список управления доступом вместо его замены, см. раздел [Обновление списков управления доступом](#update-acls) этой статьи.  

Если вы решили *задать* список управления доступом, необходимо добавить запись для пользователя-владельца, запись для группы владельцев и запись для всех других пользователей. Дополнительные сведения о пользователях-владельце, группе владельцев и других пользователях см. в разделе [Пользователи и удостоверения](data-lake-storage-access-control.md#users-and-identities).

В этом разделе показано, как:

- Настройка списка ACL
- Рекурсивная настройка ACL

### <a name="set-an-acl"></a>Настройка списка ACL

Используйте `set-AzDataLakeGen2ItemAclObject` командлет, чтобы создать список ACL для пользователя-владельца, группы-владельца или других пользователей. Затем используйте `Update-AzDataLakeGen2Item` командлет для фиксации ACL.

Этот пример задает список управления доступом для корневого каталога **контейнера** для пользователя-владельца, группы-владельца или других пользователей, а затем ВЫВОДИТ список ACL на консоль.

```powershell
$filesystemName = "my-file-system"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission -wx -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Acl $acl
$filesystem = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName
$filesystem.ACL
```

Этот пример задает список ACL для **каталога** пользователя-владельца, группы-владельца или других пользователей, а затем ВЫВОДИТ список ACL на консоль.

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission -wx -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
$dir = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname
$dir.ACL
```

> [!NOTE]
> Если вы хотите задать запись ACL **по умолчанию** , используйте параметр **-DefaultScope** при выполнении команды **Set-AzDataLakeGen2ItemAclObject** . Например, `$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rwx -DefaultScope`. 

В этом примере список ACL задается для **файла** , владеющего пользователем, группой-владельцем или другими пользователями, а затем на консоль выводится список ACL.

```powershell
$filesystemName = "my-file-system"
$filePath = "my-directory/upload.txt"
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rw- 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission rw- -InputObject $acl 
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission "-wx" -InputObject $acl
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath -Acl $acl
$file = Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $filePath
$file.ACL
```

> [!NOTE]
> Чтобы задать список управления доступом для определенной группы или пользователя, используйте соответствующие идентификаторы объектов. Например, `group:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` или `user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

На следующем изображении показаны выходные данные после установки списка ACL для файла.

![Получить выходные данные ACL для файла](./media/data-lake-storage-directory-file-acl-powershell/set-acl.png)

В этом примере владелец и группа-владелец имеют разрешения только на чтение и запись. У всех остальных пользователей есть разрешения на запись и выполнение. Дополнительные сведения о списках управления доступом в Azure Data Lake Storage 2-го поколения см. на [этой странице](data-lake-storage-access-control.md).

### <a name="set-acls-recursively"></a>Рекурсивная настройка ACL

Настройте списки управления доступом рекурсивно с помощью командлета **Set-AzDataLakeGen2AclRecursive** .

В этом примере задается список ACL для каталога с именем `my-parent-directory` . Эти записи предоставляют владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ. Последняя запись ACL в этом примере предоставляет пользователю с ИДЕНТИФИКАТОРом "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" разрешения на чтение и выполнение.

```powershell
$filesystemName = "my-container"
$dirname = "my-parent-directory/"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rwx 
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType group -Permission r-x -InputObject $acl 
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType other -Permission "---" -InputObject $acl
$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission r-x -InputObject $acl 

Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl

```

> [!NOTE]
> Если вы хотите задать запись ACL **по умолчанию** , используйте параметр **-DefaultScope** при выполнении команды **Set-AzDataLakeGen2ItemAclObject** . Например, `$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -Permission rwx -DefaultScope`.

Пример рекурсивного задания списков ACL в пакетах путем указания размера пакета см. в справочной статье по [Set-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/set-azdatalakegen2aclrecursive) .

## <a name="update-acls"></a>Обновление списков управления доступом

При *обновлении* списка управления доступом необходимо изменить список управления доступом вместо замены списка управления доступом. Например, можно добавить нового субъекта безопасности в список управления доступом, не затрагивая другие субъекты безопасности, перечисленные в списке ACL.  Чтобы заменить список управления доступом вместо обновления, см. раздел [Настройка списков управления доступом](#set-acls) этой статьи.

Чтобы обновить список управления доступом, создайте новый объект ACL с записью ACL, которую требуется обновить, а затем используйте этот объект в операции Update ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для обновления.

В этом разделе показано, как:

- Обновление списка ACL
- Рекурсивно обновлять списки ACL

### <a name="update-an-acl"></a>Обновление списка ACL

Сначала получите список ACL. Затем с помощью `set-AzDataLakeGen2ItemAclObject` командлета добавьте или обновите запись ACL. Используйте `Update-AzDataLakeGen2Item` командлет для фиксации ACL.

В этом примере создается или обновляется список ACL для пользователя в **каталоге** .

```powershell
$filesystemName = "my-file-system"
$dirname = "my-directory/"
$acl = (Get-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname).ACL
$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityID xxxxxxxx-xxxx-xxxxxxxxxxx -Permission r-x -InputObject $acl 
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
```

> [!NOTE]
> Если вы хотите обновить запись ACL **по умолчанию** , используйте параметр **-DefaultScope** при выполнении команды **Set-AzDataLakeGen2ItemAclObject** . Например, `$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityID xxxxxxxx-xxxx-xxxxxxxxxxx -Permission r-x -DefaultScope`.

### <a name="update-acls-recursively"></a>Рекурсивно обновлять списки ACL

Рекурсивно обновите списки управления доступом с помощью командлета  **Update-AzDataLakeGen2AclRecursive** .

В этом примере обновляется запись ACL с разрешением на запись.

```powershell
$filesystemName = "my-container"
$dirname = "my-parent-directory/"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission rwx

Update-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl

```

> [!NOTE]
> Чтобы задать список управления доступом для определенной группы или пользователя, используйте соответствующие идентификаторы объектов. Например, `group:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` или `user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

Пример рекурсивного обновления списков ACL в пакетах путем указания размера пакета см. в справочной статье по [Update-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/update-azdatalakegen2aclrecursive) .

## <a name="remove-acl-entries"></a>Удалить записи ACL

В этом разделе показано, как:

- Удаление записи ACL
- Рекурсивное удаление записей ACL

### <a name="remove-an-acl-entry"></a>Удаление записи ACL

В этом примере удаляется запись из существующего списка ACL.

```powershell
$id = "xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Create the new ACL object.
[Collections.Generic.List[System.Object]]$aclnew =$acl

foreach ($a in $aclnew)
{
    if ($a.AccessControlType -eq "User"-and $a.DefaultScope -eq $false -and $a.EntityId -eq $id)
    {
        $aclnew.Remove($a);
        break;
    }
}
Update-AzDataLakeGen2Item -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $aclnew
```

### <a name="remove-acl-entries-recursively"></a>Рекурсивное удаление записей ACL

Можно рекурсивно удалить одну или несколько записей списка ACL. Чтобы удалить запись ACL, создайте новый объект ACL для удаляемой записи ACL, а затем используйте этот объект в операции удаления списка ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для удаления.

Удалите записи ACL с помощью командлета **Remove-AzDataLakeGen2AclRecursive** .

В этом примере удаляется запись ACL из корневого каталога контейнера.  

```powershell
$filesystemName = "my-container"
$userID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

$acl = Set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission "---" 

Remove-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName  -Acl $acl
```

> [!NOTE]
> Если вы хотите удалить запись ACL **по умолчанию** , используйте параметр **-DefaultScope** при выполнении команды **Set-AzDataLakeGen2ItemAclObject** . Например, `$acl = set-AzDataLakeGen2ItemAclObject -AccessControlType user -EntityId $userID -Permission "---" -DefaultScope`.

Пример рекурсивного удаления списков ACL в пакетах путем указания размера пакета см. в справочной статье [recurs-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/remove-azdatalakegen2aclrecursive) .

## <a name="recover-from-failures"></a>Восстановление после сбоев

При рекурсивном изменении списков ACL могут возникать ошибки времени выполнения или разрешения. 

Для ошибок времени выполнения перезапустите процесс с самого начала. Ошибки разрешений могут возникать, если субъект безопасности не имеет достаточных разрешений для изменения списка управления доступом к каталогу или файлу в изменяемой иерархии каталогов. Устраните проблему с разрешениями, а затем выберите либо возобновление процесса с точки сбоя с помощью токена продолжения, либо перезапуск процесса с начала. Если вы предпочитаете перезапустить с самого начала, вам не нужно использовать токен продолжения. Вы можете повторно применить записи ACL без отрицательного воздействия.

Этот пример возвращает результаты в переменную, а затем передает неудачные записи в отформатированную таблицу.

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl
$result
$result.FailedEntries | ft 
```

На основе выходных данных таблицы можно исправить любые ошибки разрешений, а затем возобновить выполнение с помощью токена продолжения.

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl -ContinuationToken $result.ContinuationToken
$result

```

Пример рекурсивного задания списков ACL в пакетах путем указания размера пакета см. в справочной статье по [Set-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/set-azdatalakegen2aclrecursive) .

Если вы хотите, чтобы процесс завершился без прерывания с ошибками разрешений, можно указать это.

В этом примере используется `ContinueOnFailure` параметр, чтобы выполнение продолжалось, даже если операция встречает ошибку разрешения. 

```powershell
$result = Set-AzDataLakeGen2AclRecursive -Context $ctx -FileSystem $filesystemName -Path $dirname -Acl $acl -ContinueOnFailure

echo "[Result Summary]"
echo "TotalDirectoriesSuccessfulCount: `t$($result.TotalFilesSuccessfulCount)"
echo "TotalFilesSuccessfulCount: `t`t`t$($result.TotalDirectoriesSuccessfulCount)"
echo "TotalFailureCount: `t`t`t`t`t$($result.TotalFailureCount)"
echo "FailedEntries:"$($result.FailedEntries | ft) 
```

Пример рекурсивного задания списков ACL в пакетах путем указания размера пакета см. в справочной статье по [Set-AzDataLakeGen2AclRecursive](/powershell/module/az.storage/set-azdatalakegen2aclrecursive) .

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>См. также раздел

- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Командлеты PowerShell для службы хранилища](/powershell/module/az.storage)
- [Модель контроля доступа в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
- [Списки управления доступом (ACL) в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
