---
title: Использование сертификатов и безопасный доступ к Azure Key Vault с помощью пакетной службы
description: Сведения о программном доступе к учетным данным в Key Vault с помощью пакетной службы Azure.
ms.topic: how-to
ms.date: 10/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: eaaeaa05caca7897eb649b56504b643038f08d53
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99260135"
---
# <a name="use-certificates-and-securely-access-azure-key-vault-with-batch"></a>Использование сертификатов и безопасный доступ к Azure Key Vault с помощью пакетной службы

В этой статье вы узнаете, как настроить узлы пакетной службы для безопасного доступа к учетным данным, хранящимся в [Azure Key Vault](../key-vault/general/overview.md). Нет никакого смысла помещать учетные данные администратора в Key Vault, если сведения для доступа к этому Key Vault будут жестко запрограммированы прямо в коде скрипта. Правильное решение — использовать сертификат, который предоставляет узлам пакетной службы доступ к Key Vault.

Чтобы пройти аутентификацию в Azure Key Vault с узла пакетной службы, вам потребуется следующее:

- учетные данные Azure Active Directory (Azure AD);
- сертификат;
- учетная запись пакетной службы;
- пул пакетной службы хотя бы с одним узлом.

## <a name="obtain-a-certificate"></a>Получение сертификата

Если у вас еще нет сертификата, проще всего будет создать самозаверяющий сертификат с помощью программы командной строки `makecert`.

Обычно `makecert` размещается в папке `C:\Program Files (x86)\Windows Kits\10\bin\<arch>`. Откройте командную строку с правами администратора и перейдите к папке `makecert`, как показано в следующем примере.

```console
cd C:\Program Files (x86)\Windows Kits\10\bin\x64
```

Затем с помощью средства `makecert` создайте файлы самозаверяющего сертификата `batchcertificate.cer` и `batchcertificate.pvk`. Общее имя сертификата для нашего примера не имеет значения, но лучше присваивать такое имя, которое будет напоминать о назначении сертификата.

```console
makecert -sv batchcertificate.pvk -n "cn=batch.cert.mydomain.org" batchcertificate.cer -b 09/23/2019 -e 09/23/2019 -r -pe -a sha256 -len 2048
```

Для пакетной службы требуется файл `.pfx`. С помощью средства [pvk2pfx](/windows-hardware/drivers/devtest/pvk2pfx) преобразуйте файлы `.cer` и `.pvk`, созданные `makecert`, в единый файл `.pfx`.

```console
pvk2pfx -pvk batchcertificate.pvk -spc batchcertificate.cer -pfx batchcertificate.pfx -po
```

## <a name="create-a-service-principal"></a>Создание субъекта-службы

Доступ к Key Vault предоставляется для **пользователя** или **субъекта-службы**. Для программного доступа к Key Vault используйте [субъект-службу](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) с сертификатом, созданным на предыдущем шаге. Субъект-служба должен размещаться в том же клиенте Azure AD, что и Key Vault.

```powershell
$now = [System.DateTime]::Parse("2020-02-10")
# Set this to the expiration date of the certificate
$expirationDate = [System.DateTime]::Parse("2021-02-10")
# Point the script at the cer file you created $cerCertificateFilePath = 'c:\temp\batchcertificate.cer'
$cer = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$cer.Import($cerCertificateFilePath)
# Load the certificate into memory
$credValue = [System.Convert]::ToBase64String($cer.GetRawCertData())
# Create a new AAD application that uses this certificate
$newADApplication = New-AzureRmADApplication -DisplayName "Batch Key Vault Access" -HomePage "https://batch.mydomain.com" -IdentifierUris "https://batch.mydomain.com" -certValue $credValue -StartDate $now -EndDate $expirationDate
# Create new AAD service principal that uses this application
$newAzureAdPrincipal = New-AzureRmADServicePrincipal -ApplicationId $newADApplication.ApplicationId
```

URL-адреса для приложения не важны, так как они используются только для Key Vaultного доступа.

## <a name="grant-rights-to-key-vault"></a>Предоставление доступа к Key Vault

Субъекту-службе, созданному на предыдущем шаге, нужны разрешения на получение секретов из Key Vault. Разрешение можно предоставить либо с помощью [портал Azure](../key-vault/general/assign-access-policy-portal.md) , либо с помощью команды PowerShell ниже.

```powershell
Set-AzureRmKeyVaultAccessPolicy -VaultName 'BatchVault' -ServicePrincipalName '"https://batch.mydomain.com' -PermissionsToSecrets 'Get'
```

## <a name="assign-a-certificate-to-a-batch-account"></a>Назначение сертификата для учетной записи пакетной службы

Создайте пул пакетной службы, перейдите на вкладку сертификата в области со сведениями об этом пуле и назначьте ему только что созданный сертификат. Теперь этот сертификат будет размещен на всех узлах пакетной службы.

Затем назначьте сертификат учетной записи пакетной службы. Назначение сертификата учетной записи позволяет пакетно назначить его пулам, а затем узлам. Проще всего для этого перейти к учетной записи пакетной службы на портале и в разделе **Сертификаты** выбрать действие **Добавить**. Отправьте `.pfx` созданный ранее файл и укажите пароль. Когда этот процесс завершится, сертификат появится в списке и вы сможете проверить отпечаток.

Теперь при создании пула пакетной службы можно переходить к **сертификатам** в пуле и назначить сертификат, созданный для этого пула. При этом убедитесь, что выбрано расположение хранилища **LocalMachine**. Сертификат отправляется на все узлы пакетной службы в пуле.

## <a name="install-azure-powershell"></a>Установите Azure PowerShell

Если вы планируете обращаться к Key Vault через скрипты PowerShell на узлах, потребуется установить библиотеку Azure PowerShell. Если на узлах установлен Windows Management Framework (WMF) 5, для его загрузки можно использовать команду Install-Module. Если вы используете узлы, у которых нет WMF 5, самый простой способ установить его — это объединить `.msi` файл Azure PowerShell с пакетными файлами, а затем вызвать установщик в качестве первой части сценария запуска пакетной службы. Этот вариант подробно представлен в следующем примере:

```powershell
$psModuleCheck=Get-Module -ListAvailable -Name Azure -Refresh
if($psModuleCheck.count -eq 0) {
    $psInstallerPath = Join-Path $downloadPath "azure-powershell.3.4.0.msi" Start-Process msiexec.exe -ArgumentList /i, $psInstallerPath, /quiet -wait
}
```

## <a name="access-key-vault"></a>Получите доступ к Key Vault.

Теперь вы можете получить доступ к Key Vault в скриптах, выполняющихся на узлах пакетной службы. Чтобы обратиться к Key Vault из скрипта, нужно лишь пройти проверку подлинности в Azure AD с использованием сертификата. Для этого выполните приведенные ниже команды PowerShell. Укажите правильный GUID для параметра **отпечатка**, а также **идентификатор приложения** и **идентификатор клиента** (клиент, в котором размещается этот субъект-служба).

```powershell
Add-AzureRmAccount -ServicePrincipal -CertificateThumbprint -ApplicationId
```

После проверки подлинности обращайтесь к KeyVault обычным образом.

```powershell
$adminPassword=Get-AzureKeyVaultSecret -VaultName BatchVault -Name batchAdminPass
```

Это учетные данные для использования в скрипте.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [Azure Key Vault](../key-vault/general/overview.md).
- Ознакомьтесь с [базовым планом безопасности Azure для пакетной](security-baseline.md)службы.
- Сведения о пакетных функциях, таких как [Настройка доступа к вычисленным узлам](pool-endpoint-configuration.md) [с помощью вычислений на основе ресурсов Linux](batch-linux-nodes.md), а также [использование частных конечных точек](private-connectivity.md).