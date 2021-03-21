---
title: Включение проверки подлинности AD DS для файловых ресурсов Azure
description: Узнайте, как включить проверку подлинности домен Active Directory служб по протоколу SMB для файловых ресурсов Azure. Затем присоединенные к домену виртуальные машины Windows могут получить доступ к файловым ресурсам Azure с помощью учетных данных AD DS.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 09/13/2020
ms.author: rogarana
ms.openlocfilehash: 5ee4481b3151e28d5d37760e486a43adbc194994
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102553227"
---
# <a name="part-one-enable-ad-ds-authentication-for-your-azure-file-shares"></a>Часть 1. Включение проверки подлинности AD DS для файловых ресурсов Azure 

Перед включением проверки подлинности домен Active Directory служб (AD DS) убедитесь, что вы прочитали [обзорную статью](storage-files-identity-auth-active-directory-enable.md) , чтобы понять Поддерживаемые сценарии и требования.

В этой статье описывается процесс, необходимый для включения проверки подлинности домен Active Directory Services (AD DS) в учетной записи хранения. После включения этой функции необходимо настроить учетную запись хранения и AD DS, чтобы использовать AD DS учетные данные для проверки подлинности в общем файловом ресурсе Azure. Чтобы включить проверку подлинности AD DS по протоколу SMB для файловых ресурсов Azure, необходимо зарегистрировать учетную запись хранения в AD DS, а затем задать необходимые свойства домена в учетной записи хранения.

Чтобы зарегистрировать учетную запись хранения в AD DS, создайте учетную запись, представляющую ее в AD DS. Этот процесс можно рассматривать как создание учетной записи, представляющей локальный файловый сервер Windows в AD DS. Если эта функция включена в учетной записи хранения, она применяется ко всем новым и существующим файловым ресурсам в учетной записи.

## <a name="option-one-recommended-use-azfileshybrid-powershell-module"></a>Вариант 1 (рекомендуется): использование модуля PowerShell Азфилешибрид

Командлеты в модуле PowerShell Азфилешибрид выполняют необходимые изменения и включают эту функцию. Так как некоторые части командлетов взаимодействуют с локальными AD DS, мы объясним, что делают командлеты, чтобы вы могли определить, соответствуют ли изменения требованиям и политикам безопасности, и убедитесь, что у вас есть необходимые разрешения для выполнения командлетов. Хотя мы рекомендуем использовать модуль Азфилешибрид, если это не удается сделать, мы предоставляющи эти действия, чтобы вы могли выполнить их вручную.

### <a name="download-azfileshybrid-module"></a>Скачать модуль Азфилешибрид

- [Скачайте и Распакуйте модуль азфилешибрид (общедоступный модуль: v 0.2.0 +).](https://github.com/Azure-Samples/azure-files-samples/releases) Обратите внимание, что шифрование AES 256 Kerberos поддерживается на v 0.2.2 или более поздней версии. Если вы включили функцию с версией Азфилешибрид ниже v 0.2.2 и хотите обновить для поддержки шифрования AES 256 Kerberos, обратитесь к [этой статье](./storage-troubleshoot-windows-file-connection-problems.md#azure-files-on-premises-ad-ds-authentication-support-for-aes-256-kerberos-encryption). 
- Установите и выполните модуль на устройстве, присоединенном к локальному AD DS с учетными данными AD DS, имеющими разрешения на создание учетной записи входа в службу или учетной записи компьютера в целевом AD.
-  Запустите скрипт, используя локальные учетные данные AD DS, которые синхронизируются с Azure AD. Локальные учетные данные AD DS должны иметь разрешения владельца учетной записи хранения или роли участника Azure.

### <a name="run-join-azstorageaccountforauth"></a>Запустить Join-AzStorageAccountForAuth

`Join-AzStorageAccountForAuth`Командлет выполняет эквивалентное автономное присоединение к домену от имени указанной учетной записи хранения. Сценарий использует командлет для создания [учетной записи компьютера](/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) в домене AD. Если по какой-либо причине нельзя использовать учетную запись компьютера, можно изменить скрипт, чтобы создать [учетную запись входа в службу](/windows/win32/ad/about-service-logon-accounts) . Если вы решили выполнить команду вручную, следует выбрать учетную запись, подходящую для вашей среды.

Учетная запись AD DS, созданная командлетом, представляет учетную запись хранения. Если учетная запись AD DS создана в подразделении (OU), которая обеспечивает истечение срока действия пароля, необходимо обновить пароль до максимального срока действия пароля. Не удается обновить пароль учетной записи до того, как эта дата приведет к ошибкам проверки подлинности при доступе к файловым ресурсам Azure. Сведения об обновлении пароля см. в разделе [Обновление пароля учетной записи AD DS](storage-files-identity-ad-ds-update-password.md).

Перед выполнением в PowerShell замените значения заполнителей собственными параметрами в приведенных ниже параметрах.
> [!IMPORTANT]
> Командлет присоединение к домену создаст учетную запись AD для представления учетной записи хранения (общей папки) в AD. Вы можете зарегистрировать учетную запись компьютера или учетную запись входа службы. Дополнительные сведения см. в разделе [часто задаваемые вопросы](./storage-files-faq.md#security-authentication-and-access-control) . Для учетных записей компьютеров срок действия пароля по умолчанию задается в AD через 30 дней. Аналогичным образом для учетной записи входа в службу может быть задан срок действия пароля по умолчанию в домене AD или подразделении (OU).
> Для обоих типов учетных записей рекомендуется проверить срок действия пароля, настроенный в среде AD, и запланировать [Обновление пароля учетной записи хранения](storage-files-identity-ad-ds-update-password.md) учетной записи AD до максимального срока действия пароля. Вы можете [создать новое подразделение Active Directory (OU) в AD](/powershell/module/addsadministration/new-adorganizationalunit) и отключить политику истечения срока действия паролей для [учетных записей компьютеров](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj852252(v=ws.11)) или учетных записей службы входа в систему. 

```PowerShell
#Change the execution policy to unblock importing AzFilesHybrid.psm1 module
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser

# Navigate to where AzFilesHybrid is unzipped and stored and run to copy the files into your path
.\CopyToPSPath.ps1 

#Import AzFilesHybrid module
Import-Module -Name AzFilesHybrid

#Login with an Azure AD credential that has either storage account owner or contributer Azure role assignment
Connect-AzAccount

#Define parameters
$SubscriptionId = "<your-subscription-id-here>"
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

#Select the target subscription for the current session
Select-AzSubscription -SubscriptionId $SubscriptionId 

# Register the target storage account with your active directory environment under the target OU (for example: specify the OU with Name as "UserAccounts" or DistinguishedName as "OU=UserAccounts,DC=CONTOSO,DC=COM"). 
# You can use to this PowerShell cmdlet: Get-ADOrganizationalUnit to find the Name and DistinguishedName of your target OU. If you are using the OU Name, specify it with -OrganizationalUnitName as shown below. If you are using the OU DistinguishedName, you can set it with -OrganizationalUnitDistinguishedName. You can choose to provide one of the two names to specify the target OU.
# You can choose to create the identity that represents the storage account as either a Service Logon Account or Computer Account (default parameter value), depends on the AD permission you have and preference. 
# Run Get-Help Join-AzStorageAccountForAuth for more details on this cmdlet.

Join-AzStorageAccountForAuth `
        -ResourceGroupName $ResourceGroupName `
        -StorageAccountName $StorageAccountName `
        -DomainAccountType "<ComputerAccount|ServiceLogonAccount>" <# Default is set as ComputerAccount #> `
        -OrganizationalUnitDistinguishedName "<ou-distinguishedname-here>" <# If you don't provide the OU name as an input parameter, the AD identity that represents the storage account is created under the root directory. #> `
        -EncryptionType "<AES256/RC4/AES256,RC4>" <# Specify the encryption agorithm used for Kerberos authentication. Default is configured as "'RC4','AES256'" which supports both 'RC4' and 'AES256' encryption. #>

#Run the command below if you want to enable AES 256 authentication. If you plan to use RC4, you can skip this step.
Update-AzStorageAccountAuthForAES256 -ResourceGroupName $ResourceGroupName -StorageAccountName $StorageAccountName

#You can run the Debug-AzStorageAccountAuth cmdlet to conduct a set of basic checks on your AD configuration with the logged on AD user. This cmdlet is supported on AzFilesHybrid v0.1.2+ version. For more details on the checks performed in this cmdlet, see Azure Files Windows troubleshooting guide.
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## <a name="option-2-manually-perform-the-enablement-actions"></a>Вариант 2. выполнение действий по включению вручную

Если вы уже выполнили `Join-AzStorageAccountForAuth` приведенный выше сценарий, перейдите к разделу [подтверждение включения компонента](#confirm-the-feature-is-enabled) . Вам не нужно выполнять следующие действия вручную.

### <a name="checking-environment"></a>Проверка среды

Сначала необходимо проверить состояние среды. В частности, необходимо проверить, установлен ли [Active Directory PowerShell](/powershell/module/addsadministration/) и выполняется ли оболочка с правами администратора. Затем проверьте, установлен ли модуль [Az.Storage 2.0](https://www.powershellgallery.com/packages/Az.Storage/2.0.0). Если нет, установите его. После выполнения этих проверок проверьте AD DS, чтобы узнать, есть ли [учетная запись компьютера](/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) (по умолчанию) или [учетная запись для входа в службу](/windows/win32/ad/about-service-logon-accounts) , которая уже создана с именем субъекта-службы или UPN, как CIFS/My-Storage-Account-Name-WHERE. File. Core. Windows. NET. Если учетная запись не существует, создайте ее, как описано в следующем разделе.

### <a name="creating-an-identity-representing-the-storage-account-in-your-ad-manually"></a>Создание удостоверения, представляющего учетную запись хранения в AD вручную

Чтобы создать эту учетную запись вручную, создайте новый ключ Kerberos для своей учетной записи хранения. Затем используйте этот ключ Kerberos в качестве пароля для учетной записи с помощью командлетов PowerShell ниже. Этот ключ используется только во время установки и не может использоваться для операций управления или плоскости данных в учетной записи хранения. 

```PowerShell
# Create the Kerberos key on the storage account and get the Kerb1 key as the password for the AD identity to represent the storage account
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

New-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -KeyName kerb1
Get-AzStorageAccountKey -ResourceGroupName $ResourceGroupName -Name $StorageAccountName -ListKerbKey | where-object{$_.Keyname -contains "kerb1"}
```

Получив этот ключ, создайте учетную запись службы или компьютера в подразделении. Используйте следующую спецификацию (не забудьте заменить текст примера на имя учетной записи хранения):

Имя участника-службы: "CIFS/My-Storage-Account-Name —" пароль. файл. Core. Windows. NET "— ключ Kerberos для вашей учетной записи хранения.

Если подразделение использует срок действия пароля, необходимо обновить пароль до максимального срока действия пароля, чтобы предотвратить сбои проверки подлинности при доступе к файловым ресурсам Azure. Дополнительные сведения см. [в статье об обновлении пароля для удостоверения учетной записи хранения в AD](storage-files-identity-ad-ds-update-password.md) .

Сохраним идентификатор безопасности вновь созданного удостоверения, он понадобится для следующего шага. Созданное удостоверение, представляющее учетную запись хранения, не требуется синхронизировать с Azure AD.

### <a name="enable-the-feature-on-your-storage-account"></a>Включение функции в учетной записи хранения

Теперь можно включить функцию в учетной записи хранения. Укажите сведения о конфигурации для свойств домена в следующей команде, а затем запустите ее. Идентификатор безопасности учетной записи хранения, необходимый для следующей команды, — это идентификатор безопасности удостоверения, созданного в AD DS в [предыдущем разделе](#creating-an-identity-representing-the-storage-account-in-your-ad-manually).

```PowerShell
# Set the feature flag on the target storage account and provide the required AD domain information
Set-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>" `
        -EnableActiveDirectoryDomainServicesForFile $true `
        -ActiveDirectoryDomainName "<your-domain-name-here>" `
        -ActiveDirectoryNetBiosDomainName "<your-netbios-domain-name-here>" `
        -ActiveDirectoryForestName "<your-forest-name-here>" `
        -ActiveDirectoryDomainGuid "<your-guid-here>" `
        -ActiveDirectoryDomainsid "<your-domain-sid-here>" `
        -ActiveDirectoryAzureStorageSid "<your-storage-account-sid>"
```

### <a name="debugging"></a>Отладка

Вы можете запустить командлет Debug-AzStorageAccountAuth, чтобы выполнить ряд основных проверок конфигурации AD с вошедшим в систему пользователем AD. Этот командлет поддерживается в AzFilesHybrid версии 0.1.2+. Дополнительные сведения о проверках, выполненных в этом командлете, см. в разделе [не удается подключить службу файлов Azure с учетными данными AD](storage-troubleshoot-windows-file-connection-problems.md#unable-to-mount-azure-files-with-ad-credentials) в статье Руководство по устранению неполадок для Windows.

```PowerShell
Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```

## <a name="confirm-the-feature-is-enabled"></a>Убедитесь, что функция включена

Проверить, включена ли эта функция в вашей учетной записи хранения, можно с помощью следующего скрипта:

```PowerShell
# Get the target storage account
$storageaccount = Get-AzStorageAccount `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -Name "<your-storage-account-name-here>"

# List the directory service of the selected service account
$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions

# List the directory domain information if the storage account has enabled AD DS authentication for file shares
$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
```

В случае успеха выходные данные должны выглядеть следующим образом:

```PowerShell
DomainName:<yourDomainHere>
NetBiosDomainName:<yourNetBiosDomainNameHere>
ForestName:<yourForestNameHere>
DomainGuid:<yourGUIDHere>
DomainSid:<yourSIDHere>
AzureStorageID:<yourStorageSIDHere>
```
## <a name="next-steps"></a>Дальнейшие действия

Вы успешно включили функцию в учетной записи хранения. Чтобы использовать эту функцию, необходимо назначить разрешения на уровне общего ресурса. Перейдите к следующему разделу.

[Часть 2. Назначение удостоверениям разрешений на уровне общего ресурса](storage-files-identity-ad-ds-assign-permissions.md)