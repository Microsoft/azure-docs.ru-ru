---
title: Выполнение runbook службы автоматизации Azure в гибридной рабочей роли Runbook
description: В этой статье описывается, как запускать модули Runbook на компьютерах в локальном центре обработки данных или другом поставщике облачных служб с помощью гибридной рабочей роли Runbook.
services: automation
ms.subservice: process-automation
ms.date: 03/10/2021
ms.topic: conceptual
ms.openlocfilehash: 6d1f504458aed440464015a34479d75992fe5c45
ms.sourcegitcommit: 6776f0a27e2000fb1acb34a8dddc67af01ac14ac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103149381"
---
# <a name="run-runbooks-on-a-hybrid-runbook-worker"></a>Запуск модулей Runbook в гибридной рабочей роли Runbook

Runbook, которые выполняются в [гибридной рабочей роли Runbook](automation-hybrid-runbook-worker.md), обычно используются для управления ресурсами на локальном компьютере или в локальной среде, где развернута эта рабочая роль. Модули Runbook в службе автоматизации Azure обычно управляют ресурсами в облаке Azure. Несмотря на различия в применении, runbook в службе автоматизации Azure и в гибридной рабочей роли Runbook имеют идентичную структуру.

Когда вы создаете последовательности runbook для выполнения в гибридной рабочей роли Runbook, их нужно редактировать и тестировать на том компьютере, где размещена эта рабочая роль. Этот компьютер уже имеет все модули PowerShell и доступ к сети, необходимые для управления локальными ресурсами. Завершив тестирование runbook на компьютере с гибридной рабочей ролью Runbook, вы можете передать его в среду службы автоматизации Azure, где он будет выполняться в рабочей роли.

## <a name="plan-runbook-job-behavior"></a>Планирование поведения для задания runbook

Служба автоматизации Azure обрабатывает задания в гибридных рабочих ролях Runbook по-разному из заданий, выполняемых в изолированных средах Azure. Если ваш runbook будет выполняться долго, обеспечьте ему устойчивость к возможному перезапуску. Дополнительные сведения о поведении задания см. в разделе [Задания гибридной рабочей роли Runbook](automation-hybrid-runbook-worker.md#hybrid-runbook-worker-jobs).

Задания для гибридных рабочих ролей Runbook выполняются под локальной **системной** учетной записью Windows или учетной записью **нксаутоматион** в Linux. Для Linux убедитесь, что учетная запись **нксаутоматион** имеет доступ к расположению, в котором хранятся модули Runbook. При использовании командлета [Install-Module](/powershell/module/powershellget/install-module) обязательно укажите значение AllUsers для параметра `Scope`, чтобы предоставить необходимый доступ для учетной записи **nxautomation**. Дополнительные сведения об использовании PowerShell в среде Linux см. в разделе [Известные проблемы с PowerShell на платформах, отличных от Windows](/powershell/scripting/whats-new/what-s-new-in-powershell-70).

## <a name="configure-runbook-permissions"></a>Настройка разрешений Runbook

Разрешения для работы runbook в гибридной рабочей роли Runbook можно определить следующими способами:

* Настройте runbook так, чтобы он сам выполнял аутентификацию для доступа к локальным ресурсам.
* Настройте аутентификацию через [управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-arm.md#grant-your-vm-access-to-a-resource-group-in-resource-manager).
* Укажите учетную запись запуска от имени, чтобы предоставить контекст пользователя для всех последовательностей runbook.

### <a name="use-runbook-authentication-to-local-resources"></a>Использование аутентификации runbook для доступа к локальным ресурсам

Для подготовки runbook, который самостоятельно выполняет аутентификацию для доступа к ресурсам, включите в этот runbook [учетные данные](./shared-resources/credentials.md) и [сертификат](./shared-resources/certificates.md) для нужных ресурсов. Существует несколько командлетов, которые поддерживают работу с учетными данными, что позволяет выполнять аутентификацию для доступа к разным ресурсам. В следующем примере показана часть модуля Runbook, предназначенная для перезапуска компьютера. Он извлекает учетные данные из ресурса учетных данных, а также получает имя компьютера из ресурса переменной, после чего использует эти значения в командлете `Restart-Computer`.

```powershell
$Cred = Get-AutomationPSCredential -Name "MyCredential"
$Computer = Get-AutomationVariable -Name "ComputerName"

Restart-Computer -ComputerName $Computer -Credential $Cred
```

Можно также использовать действие [InlineScript](automation-powershell-workflow.md#use-inlinescript). `InlineScript` позволяет запускать блоки кода на другом компьютере с использованием учетных данных.

### <a name="use-runbook-authentication-with-managed-identities"></a><a name="runbook-auth-managed-identities"></a>Аутентификация runbook на основе управляемых удостоверений

Гибридные рабочие роли Runbook на виртуальных машинах Azure могут использовать управляемые удостоверения при аутентификации для доступа к ресурсам Azure. Использование управляемых удостоверений вместо учетных записей запуска от имени дает некоторые преимущества, а точнее избавляет от выполнения следующих операций:

* экспорт сертификата запуска от имени с последующим импортом в гибридную рабочую роль Runbook;
* обновление сертификата учетной записи запуска от имени;
* обработка объекта подключения для запуска от имени в коде runbook.

Выполните следующие действия, чтобы использовать управляемое удостоверение для ресурсов Azure в гибридной рабочей роли Runbook:

1. Создайте виртуальную машину Azure.
1. Настройте управляемые удостоверения для ресурсов Azure на этой виртуальной машине. Подробнее см. в разделе [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине с помощью портала Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md#enable-system-assigned-managed-identity-on-an-existing-vm).
1. Предоставьте виртуальной машине доступ к группе ресурсов в Resource Manager. См. раздел [Использование назначаемого системой управляемого удостоверения на виртуальной машине Windows для доступа к Resource Manager](../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-arm.md#grant-your-vm-access-to-a-resource-group-in-resource-manager).
1. Установите на виртуальной машине гибридную рабочую роль Runbook. Подробнее см. в статьях [Развертывание гибридной рабочей роли Runbook для Windows](automation-windows-hrw-install.md) и [Развертывание гибридной рабочей роли Runbook для Linux](automation-linux-hrw-install.md).
1. Обновите runbook, чтобы он использовал командлет [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) с параметром `Identity` для аутентификации при доступе к ресурсам Azure. Такая конфигурация сокращает потребность использовать учетную запись запуска от имени и управлять сертификатом для нее.

    ```powershell
    # Connect to Azure using the managed identities for Azure resources identity configured on the Azure VM that is hosting the hybrid runbook worker
    Connect-AzAccount -Identity

    # Get all VM names from the subscription
    Get-AzVM | Select Name
    ```

    > [!NOTE]
    > `Connect-AzAccount -Identity` применяется для гибридной рабочей роли Runbook с использованием двух удостоверений: назначаемого системой и назначаемого пользователем. Если для гибридной рабочей роли Runbook используется несколько назначаемых пользователем удостоверений, в runbook нужно указать параметр `AccountId` для `Connect-AzAccount`, чтобы выбрать одно из них.

### <a name="use-runbook-authentication-with-run-as-account"></a>Аутентификация runbook с помощью учетной записи запуска от имени

Вы можете не настраивать в runbook собственную аутентификацию для доступа к локальным ресурсам, а указать для группы гибридных рабочих ролей Runbook учетную запись запуска от имени. Чтобы указать учетную запись запуска от имени, необходимо определить [ресурс учетных данных](./shared-resources/credentials.md) , который имеет доступ к локальным ресурсам. К этим ресурсам относятся хранилища сертификатов и все runbook, которые выполняются с этими учетными данными в гибридных рабочих ролях Runbook, входящих в эту группу.

- Имя пользователя для учетных данных должно быть представлено в одном из следующих форматов:

   * домен\имя пользователя;
   * username@domain
   * имя пользователя (для локальных учетных записей на локальном компьютере).

- Чтобы использовать Runbook PowerShell **Export-RunAsCertificateToHybridWorker**, необходимо установить модули AZ для службы автоматизации Azure на локальном компьютере.

#### <a name="use-a-credential-asset-to-specify-a-run-as-account"></a>Использование ресурса учетных данных для указания учетной записи запуска от имени

Чтобы указать учетную запись запуска от имени для гибридной рабочей роли Runbook, выполните следующую процедуру:

1. Создайте [ресурс учетных данных](./shared-resources/credentials.md) с доступом к локальным ресурсам.
1. На портале Azure откройте учетную запись службы автоматизации.
1. Щелкните **Группы гибридных рабочих ролей** и выберите нужную группу.
1. Щелкните **Все параметры**, а затем — **Настройки группы гибридных рабочих ролей**.
1. Измените для параметра **Запуск от имени** значение **По умолчанию** на **Пользовательский**.
1. Выберите учетные данные и нажмите кнопку **Сохранить**.

## <a name="install-run-as-account-certificate"></a><a name="runas-script"></a>Установка сертификата учетной записи запуска от имени

В ходе автоматизированного процесса сборки для развертывания ресурсов в Azure вам может потребоваться доступ к локальным системам для выполнения задачи или набора действий, входящих в последовательность развертывания. Чтобы обеспечить поддержку аутентификации в Azure с использованием учетной записи запуска от имени, необходимо установить соответствующий сертификат.

>[!NOTE]
>Сейчас Runbook PowerShell не выполняется на компьютерах Linux. Она выполняется только на компьютерах Windows.
>

Следующий runbook PowerShell с именем **Export-RunAsCertificateToHybridWorker** экспортирует сертификат запуска от имени из учетной записи службы автоматизации Azure. Также этот runbook скачивает сертификат и импортирует его в хранилище сертификатов локального компьютера, относящегося к гибридной рабочей роли Runbook, которая подключена к той же учетной записи. После этого runbook проверяет, может ли рабочая роль успешно выполнять аутентификацию в Azure с использованием учетной записи запуска от имени.

>[!NOTE]
>Этот модуль Runbook PowerShell не предназначен или предназначен для запуска вне учетной записи службы автоматизации в качестве скрипта на целевом компьютере.
>

```azurepowershell-interactive
<#PSScriptInfo
.VERSION 1.0
.GUID 3a796b9a-623d-499d-86c8-c249f10a6986
.AUTHOR Azure Automation Team
.COMPANYNAME Microsoft
.COPYRIGHT
.TAGS Azure Automation
.LICENSEURI
.PROJECTURI
.ICONURI
.EXTERNALMODULEDEPENDENCIES
.REQUIREDSCRIPTS
.EXTERNALSCRIPTDEPENDENCIES
.RELEASENOTES
#>

<#
.SYNOPSIS
Exports the Run As certificate from an Azure Automation account to a hybrid worker in that account.

.DESCRIPTION
This runbook exports the Run As certificate from an Azure Automation account to a hybrid worker in that account. Run this runbook on the hybrid worker where you want the certificate installed. This allows the use of the AzureRunAsConnection to authenticate to Azure and manage Azure resources from runbooks running on the hybrid worker.

.EXAMPLE
.\Export-RunAsCertificateToHybridWorker

.NOTES
LASTEDIT: 2016.10.13
#>

# Generate the password used for this certificate
Add-Type -AssemblyName System.Web -ErrorAction SilentlyContinue | Out-Null
$Password = [System.Web.Security.Membership]::GeneratePassword(25, 10)

# Stop on errors
$ErrorActionPreference = 'stop'

# Get the management certificate that will be used to make calls into Azure Service Management resources
$RunAsCert = Get-AutomationCertificate -Name "AzureRunAsCertificate"

# location to store temporary certificate in the Automation service host
$CertPath = Join-Path $env:temp  "AzureRunAsCertificate.pfx"

# Save the certificate
$Cert = $RunAsCert.Export("pfx",$Password)
Set-Content -Value $Cert -Path $CertPath -Force -Encoding Byte | Write-Verbose

Write-Output ("Importing certificate into $env:computername local machine root store from " + $CertPath)
$SecurePassword = ConvertTo-SecureString $Password -AsPlainText -Force
Import-PfxCertificate -FilePath $CertPath -CertStoreLocation Cert:\LocalMachine\My -Password $SecurePassword -Exportable | Write-Verbose

# Test to see if authentication to Azure Resource Manager is working
$RunAsConnection = Get-AutomationConnection -Name "AzureRunAsConnection"

Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $RunAsConnection.TenantId `
    -ApplicationId $RunAsConnection.ApplicationId `
    -CertificateThumbprint $RunAsConnection.CertificateThumbprint | Write-Verbose

Set-AzContext -Subscription $RunAsConnection.SubscriptionID | Write-Verbose

# List automation accounts to confirm that Azure Resource Manager calls are working
Get-AzAutomationAccount | Select-Object AutomationAccountName
```

>[!NOTE]
>Для модулей runbook PowerShell `Add-AzAccount` и `Add-AzureRMAccount` являются псевдонимами для `Connect-AzAccount`. Если при поиске элементов библиотеки вы не видите `Connect-AzAccount`, можно использовать вместо него `Add-AzAccount` или обновить модули в учетной записи службы автоматизации.

Чтобы завершить подготовку учетной записи запуска от имени, сделайте следующее:

1. Сохраните runbook с именем **Export-RunAsCertificateToHybridWorker** на локальном компьютере, присвоив ему расширение **.ps1**.
1. Импортируйте его в учетную запись службы автоматизации.
1. Откройте runbook в редакторе и замените значение переменной `Password` своим паролем.
1. Опубликуйте runbook.
1. Запустите runbook в той группе гибридных рабочих ролей Runbook, в которой настроена учетная запись запуска от имени для запуска и аутентификации runbook. 
1. Изучите поток задания и убедитесь, что в нем есть сообщение о попытке импортировать сертификат из хранилища на локальном компьютере, и еще несколько строк. Это поведение зависит от количества учетных записей службы автоматизации, определенных в вашей подписке, и успешного прохождения аутентификации.

## <a name="work-with-signed-runbooks-on-a-windows-hybrid-runbook-worker"></a>Работа с подписанными runbook в гибридной рабочей роли Runbook для Windows

Вы можете настроить гибридную рабочую роль Runbook для Windows так, чтобы она выполняла только подписанные runbook.

> [!IMPORTANT]
> После того как вы настроите выполнение только подписанных runbook в гибридной рабочей роли Runbook, последовательности runbook без подписи не смогут в ней выполняться.

### <a name="create-signing-certificate"></a>Создание сертификата для подписи

В следующем примере создается самозаверяющийся сертификат, который можно использовать для подписывания модулей runbook. Этот код создает сертификат и экспортирует его, чтобы гибридная рабочая роль Runbook могла позднее его импортировать. Также возвращается отпечаток для последующего использования в ссылке на сертификат.

```powershell
# Create a self-signed certificate that can be used for code signing
$SigningCert = New-SelfSignedCertificate -CertStoreLocation cert:\LocalMachine\my `
    -Subject "CN=contoso.com" `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -Provider "Microsoft Enhanced RSA and AES Cryptographic Provider" `
    -KeyExportPolicy Exportable `
    -KeyUsage DigitalSignature `
    -Type CodeSigningCert

# Export the certificate so that it can be imported to the hybrid workers
Export-Certificate -Cert $SigningCert -FilePath .\hybridworkersigningcertificate.cer

# Import the certificate into the trusted root store so the certificate chain can be validated
Import-Certificate -FilePath .\hybridworkersigningcertificate.cer -CertStoreLocation Cert:\LocalMachine\Root

# Retrieve the thumbprint for later use
$SigningCert.Thumbprint
```

### <a name="import-certificate-and-configure-workers-for-signature-validation"></a>Импорт сертификата и настройка рабочих ролей для проверки подписей

Скопируйте созданный сертификат в каждую гибридную рабочую роль Runbook в группе. Выполните следующий скрипт, чтобы импортировать сертификат и настроить в рабочих ролях использование проверки подписи для runbook.

```powershell
# Install the certificate into a location that will be used for validation.
New-Item -Path Cert:\LocalMachine\AutomationHybridStore
Import-Certificate -FilePath .\hybridworkersigningcertificate.cer -CertStoreLocation Cert:\LocalMachine\AutomationHybridStore

# Import the certificate into the trusted root store so the certificate chain can be validated
Import-Certificate -FilePath .\hybridworkersigningcertificate.cer -CertStoreLocation Cert:\LocalMachine\Root

# Configure the hybrid worker to use signature validation on runbooks.
Set-HybridRunbookWorkerSignatureValidation -Enable $true -TrustedCertStoreLocation "Cert:\LocalMachine\AutomationHybridStore"
```

### <a name="sign-your-runbooks-using-the-certificate"></a>Подписывание runbook с помощью сертификата

Если для гибридных рабочих ролей Runbook настроено использование только подписанных runbook, вам необходимо подписать все runbook, которые будут использоваться в этих гибридных рабочих ролях Runbook. Для подписывания runbook используйте следующий пример кода PowerShell.

```powershell
$SigningCert = ( Get-ChildItem -Path cert:\LocalMachine\My\<CertificateThumbprint>)
Set-AuthenticodeSignature .\TestRunbook.ps1 -Certificate $SigningCert
```

Подписанные runbook нужно импортировать в учетную запись службы автоматизации Azure и опубликовать вместе с блоком подписей. Чтобы узнать, как импортировать последовательности runbook, ознакомьтесь с разделом [Импорт runbook](manage-runbooks.md#import-a-runbook).

## <a name="work-with-signed-runbooks-on-a-linux-hybrid-runbook-worker"></a>Работа с подписанными runbook в гибридной рабочей роли Runbook для Linux

Чтобы поддерживать работу со подписанными runbook, гибридная рабочая роль Runbook для Linux должна иметь исполняемый файл [GPG](https://gnupg.org/index.html) на локальном компьютере.

> [!IMPORTANT]
> После того как вы настроите выполнение только подписанных runbook в гибридной рабочей роли Runbook, последовательности runbook без подписи не смогут в ней выполняться.

Чтобы завершить эту настройку, выполните следующие действия.

* Создание набора и пары ключей GPG
* Предоставление доступа к набору ключей для гибридной рабочей роли Runbook
* Проверка включения проверки подписи
* Подпись модуля runbook

### <a name="create-a-gpg-keyring-and-keypair"></a>Создание набора и пары ключей GPG

Чтобы создать набор ключей и пару ключей, используйте учетную запись [nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) в гибридной рабочей роли Runbook.

1. С помощью приложения sudo выполните вход с учетной записью **nxautomation**.

    ```bash
    sudo su – nxautomation
    ```

1. Войдя в сеанс с учетной записью **nxautomation**, создайте пару ключей GPG. Процесс GPG будет подсказывать вам следующие шаги. Вам нужно будет указать имя, адрес электронной почты, срок действия и парольную фразу. Затем подождите, пока компьютер достигнет достаточного уровня энтропии, чтобы создать ключ.

    ```bash
    sudo gpg --generate-key
    ```

1. Так как каталог GPG был создан с использованием программы sudo, необходимо изменить его владельца на **nxautomation**, выполнив следующую команду.

    ```bash
    sudo chown -R nxautomation ~/.gnupg
    ```

### <a name="make-the-keyring-available-to-the-hybrid-runbook-worker"></a>Предоставление доступа к набору ключей для гибридной рабочей роли Runbook

Создав набор ключей, сделайте его доступным для гибридной рабочей роли Runbook. Измените файл Settings **Home/нксаутоматион/штат/Worker. conf** , чтобы включить следующий пример кода в раздел File `[worker-optional]` .

```bash
gpg_public_keyring_path = /home/nxautomation/run/.gnupg/pubring.kbx
```

### <a name="verify-that-signature-validation-is-on"></a>Проверка включения проверки подписи

Если на компьютере отключена проверка подписи, вам необходимо включить ее с помощью следующей команды sudo. Замените `<LogAnalyticsworkspaceId>` идентификатором рабочей области.

```bash
sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/require_runbook_signature.py --true <LogAnalyticsworkspaceId>
```

### <a name="sign-a-runbook"></a>Подпись модуля runbook

Настроив проверку подписи,выполните следующую команду GPG для подписывания runbook.

```bash
gpg –-clear-sign <runbook name>
```

Подписанный runbook будет иметь имя **<runbook name>.asc**.

Теперь подписанный runbook можно отправить в службу автоматизации Azure и выполнять как обычную последовательность runbook.

## <a name="start-a-runbook-on-a-hybrid-runbook-worker"></a>Запуск runbook в гибридной рабочей роли Runbook

В статье [Запуск runbook в службе автоматизации Azure](start-runbooks.md) описываются разные методы запуска для runbook. При запуске модуля Runbook в гибридной рабочей роли Runbook используется параметр **выполнить в** , который позволяет указать имя группы гибридных рабочих ролей Runbook. Если группа указана, одна из рабочих ролей в этой группе извлекает и выполняет runbook. Если для runbook не указан этот параметр, служба автоматизации Azure выполняет его обычным образом.

При запуске runbook на портале Azure вы увидите вариант действия **Выполнить в**, для которого можно выбрать значения **Azure** или **Гибридная рабочая роль**. При выборе значения **Гибридная рабочая роль** можно выбрать группу ролей из раскрывающегося списка.

При запуске runbook с помощью PowerShell используйте параметр `RunOn` с командлетом [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook). Следующий пример применяет Windows PowerShell для запуска runbook с именем **Test-Runbook** в группе гибридных рабочих ролей Runbook с именем MyHybridGroup.

```azurepowershell-interactive
Start-AzAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" -RunOn "MyHybridGroup"
```

## <a name="logging"></a>Ведение журнала

Чтобы помочь в устранении проблем с модулями Runbook, работающими в гибридной рабочей роли Runbook, журналы хранятся локально в следующем расположении:

* В Windows в `C:\ProgramData\Microsoft\System Center\Orchestrator\<version>\SMA\Sandboxes` для подробного ведения журнала процесса среды выполнения задания. События высокого уровня задания модуля Runbook записываются в журнал событий **приложений и служб логс\микрософт-аутоматион\оператионс** .

* В Linux журналы гибридных рабочих ролей пользователей можно найти по адресу `/home/nxautomation/run/worker.log` , а журналы рабочих ролей Runbook System можно найти по адресу `/var/opt/microsoft/omsagent/run/automationworker/worker.log` .

## <a name="next-steps"></a>Дальнейшие действия

* Если последовательности runbook выполняются с ошибками, см. [руководство по устранению неполадок при выполнении runbook](troubleshoot/hybrid-runbook-worker.md#runbook-execution-fails).
* Дополнительные сведения о PowerShell, включая Справочник по языку и обучающие модули, см. в документации по [PowerShell](/powershell/scripting/overview).
* Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
