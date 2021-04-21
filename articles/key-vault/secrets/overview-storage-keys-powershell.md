---
title: 'Версия PowerShell: учетная запись хранения, управляемая с помощью Azure Key Vault'
description: Функция управляемой учетной записи хранения обеспечивает простую интеграцию между Azure Key Vault и учетной записью хранения Azure.
ms.topic: tutorial
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/10/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: aa4daf7c8d951a7b42479533a3a5a50c06c5c144
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107748490"
---
# <a name="manage-storage-account-keys-with-key-vault-and-azure-powershell"></a>Управление ключами учетной записи хранения с помощью Key Vault и Azure PowerShell
> [!IMPORTANT]
> Мы рекомендуем использовать интеграцию службы хранилища Azure с Azure Active Directory (Azure AD), облачной службой корпорации Майкрософт для управления удостоверениями и доступом. Интеграция с Azure AD доступна для [BLOB-объектов и очередей Azure](../../storage/common/storage-auth-aad.md). Она обеспечивает доступ к службе хранилища Azure на основе токенов OAuth2 (точно так же, как Azure Key Vault).
> Azure AD позволяет проверять подлинность клиентского приложения, используя удостоверение приложения или пользователя вместо учетных данных учетной записи хранения. Вы можете использовать [управляемое удостоверение Azure AD](../../active-directory/managed-identities-azure-resources/index.yml) при запуске в Azure. Управляемые удостоверения устраняют необходимость проверки подлинности клиента и хранения учетных данных в приложении или вместе с ним. Используйте приведенное ниже решение только в том случае, если аутентификация Azure AD невозможна.

Учетная запись хранения Azure использует учетные данные, состоящие из имени учетной записи и ключа. Ключ генерируется автоматически и служит скорее паролем, чем криптографическим ключом. Key Vault управляет ключами учетной записи хранения, периодически создавая их заново в учетной записи хранения, и предоставляет маркеры подписанного URL-адреса в случае делегированного доступа к ресурсам в учетной записи хранения.

С помощью функции ключа управляемой учетной записи хранения Key Vault можно перечислять (синхронизировать) ключи с учетными записями хранения Azure и периодически повторно создавать (заменять) ключи. Вы можете управлять ключами как для учетных записей хранения, так и для классических учетных записей хранения.

При использовании функции ключей для управляемой учетной записи хранения следует учитывать следующие моменты:

- Значения ключей никогда не возвращаются в ответе вызывающему объекту.
- Управлять ключами учетной записи хранения должно только решение Key Vault. Не управляйте ключами самостоятельно и избегайте конфликтов с процессами Key Vault.
- Управлять ключами учетной записи хранения должен только один объект Key Vault. Не разрешайте управление ключами из нескольких объектов.
- Создавайте ключи повторно только с помощью Key Vault. Не создавайте повторно ключи своей учетной записи хранения вручную.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="service-principal-application-id"></a>Идентификатор приложения субъекта-службы

Арендатор Azure AD предоставляет каждому зарегистрированному приложению [субъект-службу](../../active-directory/develop/developer-glossary.md#service-principal-object). Субъект-служба выступает в качестве идентификатора приложения, который используется во время настройки авторизации для доступа к другим ресурсам Azure с использованием Azure RBAC.

Key Vault — это приложение Майкрософт, которое предварительно зарегистрировано во всех клиентах Azure AD. Key Vault регистрируется с одним и тем же идентификатором приложения в каждом облаке Azure.

| Клиенты | Cloud | Идентификатор приложения |
| --- | --- | --- |
| Azure AD | Azure для государственных организаций | `7e7c393b-45d0-48b1-a35e-2905ddf8183c` |
| Azure AD | общедоступный пиринг Azure; | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |
| Другое  | Любой | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сначала вам следует выполнить перечисленные ниже действия.

- [Установите модуль Azure PowerShell](/powershell/azure/install-az-ps).
- [Создайте хранилище ключей.](quick-create-powershell.md)
- [Создайте учетную запись хранения Azure](../../storage/common/storage-account-create.md?tabs=azure-powershell). В имени учетной записи хранения должны использоваться только строчные буквы и цифры. Имя должно содержать от 3 до 24 знаков.


## <a name="manage-storage-account-keys"></a>Управление ключами учетной записи хранения

### <a name="connect-to-your-azure-account"></a>Подключение к учетной записи Azure

Проверьте подлинность сеанса PowerShell с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

```azurepowershell-interactive
Connect-AzAccount
```
Если у вас несколько подписок Azure, можно перечислить их с помощью командлета [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) и указать подписку, которую необходимо использовать, с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId <subscriptionId>
```

### <a name="set-variables"></a>Задание переменных

Сначала задайте переменные, которые будут использоваться командлетами PowerShell на последующих этапах. Замените заполнители YourResourceGroupName, YourStorageAccountName и YourKeyVaultName, а затем задайте для $keyVaultSpAppId значение `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` (как указано в разделе [Идентификатор приложения субъекта-службы](#service-principal-application-id) выше).

Мы также будем использовать командлеты Azure PowerShell [Get-AzContext](/powershell/module/az.accounts/get-azcontext) и [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount), чтобы получить идентификатор пользователя и контекст учетной записи хранения Azure.

```azurepowershell-interactive
$resourceGroupName = <YourResourceGroupName>
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>
$keyVaultSpAppId = "cfa8b339-82a2-471a-a3c9-0fc0be7a4093"
$storageAccountKey = "key1" #(key1 or key2 are allowed)

# Get your User Id
$userId = (Get-AzContext).Account.Id

# Get a reference to your Azure storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName

```
>[!Note]
> Для классической учетной записи хранения в качестве значения $storageAccountKey используйте "primary" и "secondary". <br>
> Вместо "Get-AzStorageAccount" для классической учетной записи хранения используйте "Get-AzResource -Name "ClassicStorageAccountName" -ResourceGroupName $resourceGroupName".

### <a name="give-key-vault-access-to-your-storage-account"></a>Предоставление доступа Key Vault к учетной записи хранения

Прежде чем приложение Key Vault сможет получить доступ к ключам учетной записи хранения и возможность управления ими, вы должны авторизовать его доступ к вашей учетной записи хранения. Приложению Key Vault нужны разрешения для *перечисления* и *повторного создания* ключей для учетной записи хранения. Эти разрешения включаются с помощью встроенной роли Azure [Роль службы оператора ключей учетных записей хранения](../../role-based-access-control/built-in-roles.md#storage-account-key-operator-service-role).

Назначьте эту роль субъекту-службе Key Vault, ограничив область учетной записью хранения с помощью командлета Azure PowerShell [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment).

```azurepowershell-interactive
# Assign Azure role "Storage Account Key Operator Service Role" to Key Vault, limiting the access scope to your storage account. For a classic storage account, use "Classic Storage Account Key Operator Service Role."
New-AzRoleAssignment -ApplicationId $keyVaultSpAppId -RoleDefinitionName 'Storage Account Key Operator Service Role' -Scope $storageAccount.Id
```

После успешного назначения ролей должен отобразиться результат, аналогичный приведенному ниже.

```console
RoleAssignmentId   : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso/providers/Microsoft.Authorization/roleAssignments/189cblll-12fb-406e-8699-4eef8b2b9ecz
Scope              : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
DisplayName        : Azure Key Vault
SignInName         :
RoleDefinitionName : storage account Key Operator Service Role
RoleDefinitionId   : 81a9662b-bebf-436f-a333-f67b29880f12
ObjectId           : 93c27d83-f79b-4cb2-8dd4-4aa716542e74
ObjectType         : ServicePrincipal
CanDelegate        : False
```

Если приложение Key Vault уже было добавлено к роли в вашей учетной записи хранения, сообщение об ошибке *Назначение роли уже существует* ошибка". Можно также проверить назначение роли, используя страницу учетной записи хранения "Управление доступом (IAM)" для портала Azure.

### <a name="give-your-user-account-permission-to-managed-storage-accounts"></a>Предоставление разрешения учетной записи пользователя для управляемых учетных записей хранения

Используйте командлет Azure PowerShell [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy), чтобы обновить политику доступа Key Vault и предоставить разрешения учетной записи хранения учетной записи пользователя.

```azurepowershell-interactive
# Give your user principal access to all storage account permissions, on your Key Vault instance

Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -UserPrincipalName $userId -PermissionsToStorage get, list, delete, set, update, regeneratekey, getsas, listsas, deletesas, setsas, recover, backup, restore, purge
```

Обратите внимание, что разрешения для учетных записей хранения не доступны на странице "Политики доступа" учетной записи хранения на портале Azure.

### <a name="add-a-managed-storage-account-to-your-key-vault-instance"></a>Добавление управляемой учетной записи хранения к экземпляру Key Vault

Используйте командлет Azure PowerShell [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount), чтобы создать управляемую учетную запись хранения в экземпляре Key Vault. Переключатель `-DisableAutoRegenerateKey` НЕ указывает на повторное создание ключей учетной записи хранения.

```azurepowershell-interactive
# Add your storage account to your Key Vault's managed storage accounts

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -DisableAutoRegenerateKey
```

После успешного добавления учетной записи хранения без повторного создания ключей должен отобразиться результат, аналогичный приведенному ниже.

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : False
Regeneration Period : 90.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                :
```

### <a name="enable-key-regeneration"></a>Включение повторного создания ключа

Чтобы хранилище Key Vault периодически повторно создавало ключи учетной записи хранения, можно использовать командлет Azure PowerShell [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount), задающий период повторного создания. В этом примере задан трехдневный период повторного создания. Когда настанет время замены, Key Vault повторно создаст ключ. Сначала он будет неактивен, но затем Key Vault задаст его в качестве активного. В любой момент времени для выпусков маркеров SAS используется только один ключ. Именно он и является активным.

```azurepowershell-interactive
$regenPeriod = [System.Timespan]::FromDays(3)

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -RegenerationPeriod $regenPeriod
```

После успешного добавления учетной записи хранения без повторного создания ключа должен отобразиться результат, аналогичный приведенному ниже.

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : True
Regeneration Period : 3.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                :
```

## <a name="shared-access-signature-tokens"></a>Маркер подписанного URL-адреса

Key Vault также может создавать маркеры подписанного URL-адреса. Подпись общего доступа обеспечивает делегированный доступ к ресурсам в вашей учетной записи хранения. Вы можете предоставить клиентам доступ к ресурсам в учетной записи хранения, не предоставляя общий доступ к ключам учетной записи. Подписанный URL-адрес обеспечивает возможность безопасно предоставлять общий доступ к ресурсам хранилища, не раскрывая ключи учетной записи.

С помощью команд в этом разделе можно выполнить следующие действия:

- Задать определение подписанного URL-адреса учетной записи.
- Создать маркер подписанного URL-адреса учетной записи для служб BLOB-объектов, файлов, таблиц и очередей. Токен создается для служб, контейнеров и объектов типов ресурсов по протоколу HTTPS со всеми разрешениями и заданными датами начала и окончания.
- Задать в хранилище определение подписанного URL-адреса управляемого хранилища Key Vault. Определение содержит шаблон универсального кода ресурса (URI) созданного маркера подписанного URL-адреса. Этот подписанный URL-адрес относится к типу `account` и действителен в течение N дней.
- Убедиться, что подписанный URL-адрес сохранен в хранилище ключей в качестве секрета.
-
### <a name="set-variables"></a>Задание переменных

Сначала задайте переменные, которые будут использоваться командлетами PowerShell на последующих этапах. Не забудьте обновить заполнители <YourStorageAccountName> и <YourKeyVaultName>.

Мы также будем использовать командлет Azure PowerShell [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext), чтобы получить идентификатор пользователя и контекст учетной записи хранения Azure.

```azurepowershell-interactive
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>

$storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -Protocol Https -StorageAccountKey Key1 #(or "Primary" for Classic Storage Account)
```

### <a name="create-a-shared-access-signature-token"></a>Создание маркера подписанного URL-адреса

Маркер подписанного URL-адреса можно создать с помощью командлета Azure PowerShell [New-AzStorageAccountSASToken](/powershell/module/az.storage/new-azstorageaccountsastoken).

```azurepowershell-interactive
$start = [System.DateTime]::Now.AddDays(-1)
$end = [System.DateTime]::Now.AddMonths(1)

$sasToken = New-AzStorageAccountSasToken -Service blob,file,Table,Queue -ResourceType Service,Container,Object -Permission "racwdlup" -Protocol HttpsOnly -StartTime $start -ExpiryTime $end -Context $storageContext
```
Значение $sasToken будет аналогичным этому.

```console
?sv=2018-11-09&sig=5GWqHFkEOtM7W9alOgoXSCOJO%2B55qJr4J7tHQjCId9S%3D&spr=https&st=2019-09-18T18%3A25%3A00Z&se=2019-10-19T18%3A25%3A00Z&srt=sco&ss=bfqt&sp=racupwdl
```

### <a name="generate-a-shared-access-signature-definition"></a>Создание определения подписанного URL-адреса

Чтобы создать определение подписанного URL-адреса, используйте командлет Azure PowerShell [Set-AzKeyVaultManagedStorageSasDefinition](/powershell/module/az.keyvault/set-azkeyvaultmanagedstoragesasdefinition).  С помощью параметра `-Name` можно задать выбранное имя.

```azurepowershell-interactive
Set-AzKeyVaultManagedStorageSasDefinition -AccountName $storageAccountName -VaultName $keyVaultName -Name <YourSASDefinitionName> -TemplateUri $sasToken -SasType 'account' -ValidityPeriod ([System.Timespan]::FromDays(30))
```

### <a name="verify-the-shared-access-signature-definition"></a>Проверка определения подписанного URL-адреса

Убедиться в том, что определение подписанного URL-адреса сохранено в хранилище ключей, можно с помощью командлета Azure PowerShell [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret).

Сначала найдите определение подписанного URL-адреса в хранилище ключей.

```azurepowershell-interactive
Get-AzKeyVaultSecret -VaultName <YourKeyVaultName>
```

Секрет, соответствующий определению SAS, будет иметь следующие свойства:

```console
Vault Name   : <YourKeyVaultName>
Name         : <SecretName>
...
Content Type : application/vnd.ms-sastoken-storage
Tags         :
```

Теперь можно воспользоваться командлетом [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret) с параметром `VaultName` и `Name` для просмотра содержимого этого секрета.

```azurepowershell-interactive
$secret = Get-AzKeyVaultSecret -VaultName <YourKeyVaultName> -Name <SecretName>
$ssPtr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secret.SecretValue)
try {
   $secretValueText = [System.Runtime.InteropServices.Marshal]::PtrToStringBSTR($ssPtr)
} finally {
   [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ssPtr)
}
Write-Output $secretValueText
```

В выходных данных этой команды будет отображаться строка определения SAS.

## <a name="next-steps"></a>Дальнейшие действия

- [Примеры ключей управляемой учетной записи хранения](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=)
- [Ссылка на PowerShell для Key Vault](/powershell/module/az.keyvault/#key_vault)
