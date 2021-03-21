---
title: Автоматическое обновление службы Mobility Service в Azure Site Recovery
description: Общие сведения об автоматическом обновлении службы Mobility Service при репликации виртуальных машин Azure с помощью Azure Site Recovery.
services: site-recovery
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 04/02/2020
ms.author: sideeksh
ms.openlocfilehash: 42d3c74229ab7eeec0ac716073a9e631775fd002
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96187346"
---
# <a name="automatic-update-of-the-mobility-service-in-azure-to-azure-replication"></a>Автоматическое обновление службы Mobility Service в репликации Azure в Azure

Azure Site Recovery использует ежемесячную частоту выпуска для устранения любых проблем и улучшения существующих функций или добавления новых. Для поддержания актуальности службы необходимо планировать развертывание исправлений каждый месяц. Чтобы избежать издержек, связанных с каждым обновлением, можно разрешить Site Recovery управлять обновлениями компонентов.

Как упоминалось в [архитектуре аварийного восстановления Azure в Azure](azure-to-azure-architecture.md), служба Mobility Service устанавливается на всех виртуальных машинах Azure, для которых включена репликация из одного региона Azure в другой. При использовании автоматического обновления каждый новый выпуск обновляет расширение службы Mobility Service.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="how-automatic-updates-work"></a>Как работают автоматические обновления

При использовании Site Recovery для управления обновлениями он развертывает глобальный модуль Runbook (используемый службами Azure) через учетную запись службы автоматизации, созданную в той же подписке, что и хранилище. В каждом хранилище используется одна учетная запись службы автоматизации. Для каждой виртуальной машины в хранилище модуль Runbook проверяет наличие активных АвтоОбновлений. Если доступна более новая версия расширения службы Mobility Service, то обновление устанавливается.

Расписание Runbook по умолчанию выполняется ежедневно в 12:00 часового пояса в регионе реплицированной виртуальной машины. Вы также можете изменить расписание Runbook с помощью учетной записи службы автоматизации.

> [!NOTE]
> Начиная с [накопительного пакета обновления 35](site-recovery-whats-new.md#updates-march-2019), можно выбрать существующую учетную запись службы автоматизации для использования при обновлениях. Перед обновлением накопительного пакета обновления 35 по умолчанию Site Recovery создала учетная запись службы автоматизации. Этот параметр можно выбрать только при включении репликации для виртуальной машины. Она недоступна для виртуальной машины, для которой уже включена репликация. Выбранный параметр применяется ко всем виртуальным машинам Azure, защищенным в одном хранилище.

Включение автоматического обновления не требует перезапуска виртуальных машин Azure или повлияет на текущую репликацию.

Выставление счетов за задание в учетной записи службы автоматизации зависит от количества минут выполнения заданий, использованных в месяц. Каждый день выполнение задания занимает несколько секунд и рассматривается как свободные единицы. По умолчанию 500 минут включается в качестве свободных единиц для учетной записи службы автоматизации, как показано в следующей таблице.

| Количество свободных единиц (каждый месяц) | Цена |
|---|---|
| Время выполнения задания 500 минут | ₹ 0.14/мин

## <a name="enable-automatic-updates"></a>Включение автоматических обновлений

Существует несколько способов управления обновлениями расширений Site Recovery.

- [Управление в рамках шага включения репликации](#manage-as-part-of-the-enable-replication-step)
- [Переключение параметров обновления для расширения внутри хранилища](#toggle-the-extension-update-settings-inside-the-vault)
- [Управление обновлениями вручную](#manage-updates-manually)

### <a name="manage-as-part-of-the-enable-replication-step"></a>Управление в рамках шага включения репликации

При включении репликации для виртуальной машины либо из [представления виртуальной машины](azure-to-azure-quickstart.md) , либо [из хранилища служб восстановления](azure-to-azure-how-to-enable-replication.md)можно либо разрешить Site Recovery управлять обновлениями для расширения Site Recovery, либо управлять им вручную.

:::image type="content" source="./media/azure-to-azure-autoupdate/enable-rep.png" alt-text="Параметры расширения":::

### <a name="toggle-the-extension-update-settings-inside-the-vault"></a>Переключение параметров обновления для расширения внутри хранилища

1. В хранилище служб восстановления перейдите к разделу **Управление**  >  **инфраструктурой Site Recovery**.
1. В разделе **для**  >  **параметров обновления расширения** виртуальных машин Azure  >  **Разрешить Site Recovery управлять**, выберите **вкл**.

   Чтобы вручную управлять расширением, установите флажок **выкл**.

1. Щелкните **Сохранить**.

:::image type="content" source="./media/azure-to-azure-autoupdate/vault-toggle.png" alt-text="Параметры обновления расширения":::

> [!IMPORTANT]
> При выборе параметра **разрешить Site Recovery для управления** этот параметр применяется ко всем виртуальным машинам в хранилище.

> [!NOTE]
> Любой параметр уведомляет вас об учетной записи службы автоматизации, используемой для управления обновлениями. Если вы используете эту функцию в хранилище впервые, по умолчанию создается новая учетная запись службы автоматизации. Кроме того, можно настроить параметр и выбрать существующую учетную запись службы автоматизации. После определения все последующие действия по включению репликации в том же хранилище будут использовать выбранную учетную запись службы автоматизации. В настоящее время в раскрывающемся меню будут перечислены учетные записи службы автоматизации, которые находятся в той же группе ресурсов, что и хранилище.

> [!IMPORTANT]
> Следующий скрипт необходимо запустить в контексте учетной записи службы автоматизации.
Для пользовательской учетной записи службы автоматизации используйте следующий скрипт:

```azurepowershell
param(
    [Parameter(Mandatory=$true)]
    [String] $VaultResourceId,
    [Parameter(Mandatory=$true)]
    [ValidateSet("Enabled",'Disabled')]
    [Alias("Enabled or Disabled")]
    [String] $AutoUpdateAction,
    [Parameter(Mandatory=$false)]
    [String] $AutomationAccountArmId
)
$SiteRecoveryRunbookName = "Modify-AutoUpdateForVaultForPatner"
$TaskId = [guid]::NewGuid().ToString()
$SubscriptionId = "00000000-0000-0000-0000-000000000000"
$AsrApiVersion = "2018-01-10"
$RunAsConnectionName = "AzureRunAsConnection"
$ArmEndPoint = "https://management.azure.com"
$AadAuthority = "https://login.windows.net/"
$AadAudience = "https://management.core.windows.net/"
$AzureEnvironment = "AzureCloud"
$Timeout = "160"
function Throw-TerminatingErrorMessage
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [String]
        $Message
        )
    throw ("Message: {0}, TaskId: {1}.") -f $Message, $TaskId
}
function Write-Tracing
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [ValidateSet("Informational", "Warning", "ErrorLevel", "Succeeded", IgnoreCase = $true)]
                [String]
        $Level,
        [Parameter(Mandatory=$true)]
        [String]
        $Message,
            [Switch]
        $DisplayMessageToUser
        )
    Write-Output $Message
}
function Write-InformationTracing
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [String]
        $Message
        )
    Write-Tracing -Message $Message -Level Informational -DisplayMessageToUser
}
function ValidateInput()
{
    try
    {
        if(!$VaultResourceId.StartsWith("/subscriptions", [System.StringComparison]::OrdinalIgnoreCase))
        {
            $ErrorMessage = "The vault resource id should start with /subscriptions."
            throw $ErrorMessage
        }
        $Tokens = $VaultResourceId.SubString(1).Split("/")
        if(!($Tokens.Count % 2 -eq 0))
        {
            $ErrorMessage = ("Odd Number of tokens: {0}." -f $Tokens.Count)
            throw $ErrorMessage
        }
        if(!($Tokens.Count/2 -eq 4))
        {
            $ErrorMessage = ("Invalid number of resource in vault ARM id expected:4, actual:{0}." -f ($Tokens.Count/2))
            throw $ErrorMessage
        }
        if($AutoUpdateAction -ieq "Enabled" -and [string]::IsNullOrEmpty($AutomationAccountArmId))
        {
            $ErrorMessage = ("The automation account ARM id should not be null or empty when AutoUpdateAction is enabled.")
            throw $ErrorMessage
        }
    }
    catch
    {
        $ErrorMessage = ("ValidateInput failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
function Initialize-SubscriptionId()
{
    try
    {
        $Tokens = $VaultResourceId.SubString(1).Split("/")
        $Count = 0
                $ArmResources = @{}
        while($Count -lt $Tokens.Count)
        {
            $ArmResources[$Tokens[$Count]] = $Tokens[$Count+1]
            $Count = $Count + 2
        }
                return $ArmResources["subscriptions"]
    }
    catch
    {
        Write-Tracing -Level ErrorLevel -Message ("Initialize-SubscriptionId: failed with [Exception: {0}]." -f $_.Exception) -DisplayMessageToUser
        throw
    }
}
function Invoke-InternalRestMethod($Uri, $Headers, [ref]$Result)
{
    $RetryCount = 0
    $MaxRetry = 3
    do
    {
        try
        {
            $ResultObject = Invoke-RestMethod -Uri $Uri -Headers $Headers
            ($Result.Value) += ($ResultObject)
            break
        }
        catch
        {
            Write-InformationTracing ("Retry Count: {0}, Exception: {1}." -f $RetryCount, $_.Exception)
            $RetryCount++
            if(!($RetryCount -le $MaxRetry))
            {
                throw
            }
            Start-Sleep -Milliseconds 2000
        }
    }while($true)
}
function Invoke-InternalWebRequest($Uri, $Headers, $Method, $Body, $ContentType, [ref]$Result)
{
    $RetryCount = 0
    $MaxRetry = 3
    do
    {
        try
        {
            $ResultObject = Invoke-WebRequest -Uri $UpdateUrl -Headers $Header -Method 'PATCH' `
                -Body $InputJson  -ContentType "application/json" -UseBasicParsing
            ($Result.Value) += ($ResultObject)
            break
        }
        catch
        {
            Write-InformationTracing ("Retry Count: {0}, Exception: {1}." -f $RetryCount, $_.Exception)
            $RetryCount++
            if(!($RetryCount -le $MaxRetry))
            {
                throw
            }
            Start-Sleep -Milliseconds 2000
        }
    }while($true)
}
function Get-Header([ref]$Header, $AadAudience, $AadAuthority, $RunAsConnectionName){
    try
    {
        $RunAsConnection = Get-AutomationConnection -Name $RunAsConnectionName
        $TenantId = $RunAsConnection.TenantId
        $ApplicationId = $RunAsConnection.ApplicationId
        $CertificateThumbprint = $RunAsConnection.CertificateThumbprint
        $Path = "cert:\CurrentUser\My\{0}" -f $CertificateThumbprint
        $Secret = Get-ChildItem -Path $Path
        $ClientCredential = New-Object Microsoft.IdentityModel.Clients.ActiveDirectory.ClientAssertionCertificate(
                $ApplicationId,
                $Secret)
        # Trim the forward slash from the AadAuthority if it exist.
        $AadAuthority = $AadAuthority.TrimEnd("/")
        $AuthContext = New-Object Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext(
            "{0}/{1}" -f $AadAuthority, $TenantId )
        $AuthenticationResult = $authContext.AcquireToken($AadAudience, $Clientcredential)
        $Header.Value['Content-Type'] = 'application\json'
        $Header.Value['Authorization'] = $AuthenticationResult.CreateAuthorizationHeader()
        $Header.Value["x-ms-client-request-id"] = $TaskId + "/" + (New-Guid).ToString() + "-" + (Get-Date).ToString("u")
    }
    catch
    {
        $ErrorMessage = ("Get-BearerToken: failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
function Get-ProtectionContainerToBeModified([ref] $ContainerMappingList)
{
    try
    {
        Write-InformationTracing ("Get protection container mappings : {0}." -f $VaultResourceId)
        $ContainerMappingListUrl = $ArmEndPoint + $VaultResourceId + "/replicationProtectionContainerMappings" + "?api-version=" + $AsrApiVersion
        Write-InformationTracing ("Getting the bearer token and the header.")
        Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
        $Result = @()
        Invoke-InternalRestMethod -Uri $ContainerMappingListUrl -Headers $header -Result ([ref]$Result)
        $ContainerMappings = $Result[0]
        Write-InformationTracing ("Total retrieved container mappings: {0}." -f $ContainerMappings.Value.Count)
        foreach($Mapping in $ContainerMappings.Value)
        {
            if(($Mapping.properties.providerSpecificDetails -eq $null) -or ($Mapping.properties.providerSpecificDetails.instanceType -ine "A2A"))
            {
                Write-InformationTracing ("Mapping properties: {0}." -f ($Mapping.properties))
                Write-InformationTracing ("Ignoring container mapping: {0} as the provider does not match." -f ($Mapping.Id))
                continue;
            }
            if($Mapping.Properties.State -ine "Paired")
            {
                Write-InformationTracing ("Ignoring container mapping: {0} as the state is not paired." -f ($Mapping.Id))
                continue;
            }
            Write-InformationTracing ("Provider specific details {0}." -f ($Mapping.properties.providerSpecificDetails))
            $MappingAutoUpdateStatus = $Mapping.properties.providerSpecificDetails.agentAutoUpdateStatus
            $MappingAutomationAccountArmId = $Mapping.properties.providerSpecificDetails.automationAccountArmId
            $MappingHealthErrorCount = $Mapping.properties.HealthErrorDetails.Count
            if($AutoUpdateAction -ieq "Enabled" -and
                ($MappingAutoUpdateStatus -ieq "Enabled") -and
                ($MappingAutomationAccountArmId -ieq $AutomationAccountArmId) -and
                ($MappingHealthErrorCount -eq 0))
            {
                Write-InformationTracing ("Provider specific details {0}." -f ($Mapping.properties))
                Write-InformationTracing ("Ignoring container mapping: {0} as the auto update is already enabled and is healthy." -f ($Mapping.Id))
                continue;
            }
            ($ContainerMappingList.Value).Add($Mapping.id)
        }
    }
    catch
    {
        $ErrorMessage = ("Get-ProtectionContainerToBeModified: failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
$OperationStartTime = Get-Date
$ContainerMappingList = New-Object System.Collections.Generic.List[System.String]
$JobsInProgressList = @()
$JobsCompletedSuccessList = @()
$JobsCompletedFailedList = @()
$JobsFailedToStart = 0
$JobsTimedOut = 0
$Header = @{}
$AzureRMProfile = Get-Module -ListAvailable -Name AzureRM.Profile | Select Name, Version, Path
$AzureRmProfileModulePath = Split-Path -Parent $AzureRMProfile.Path
Add-Type -Path (Join-Path $AzureRmProfileModulePath "Microsoft.IdentityModel.Clients.ActiveDirectory.dll")
$Inputs = ("Tracing inputs VaultResourceId: {0}, Timeout: {1}, AutoUpdateAction: {2}, AutomationAccountArmId: {3}." -f $VaultResourceId, $Timeout, $AutoUpdateAction, $AutomationAccountArmId)
Write-Tracing -Message $Inputs -Level Informational -DisplayMessageToUser
$CloudConfig = ("Tracing cloud configuration ArmEndPoint: {0}, AadAuthority: {1}, AadAudience: {2}." -f $ArmEndPoint, $AadAuthority, $AadAudience)
Write-Tracing -Message $CloudConfig -Level Informational -DisplayMessageToUser
$AutomationConfig = ("Tracing automation configuration RunAsConnectionName: {0}." -f $RunAsConnectionName)
Write-Tracing -Message $AutomationConfig -Level Informational -DisplayMessageToUser
ValidateInput
$SubscriptionId = Initialize-SubscriptionId
Get-ProtectionContainerToBeModified ([ref]$ContainerMappingList)
$Input = @{
  "properties"= @{
    "providerSpecificInput"= @{
        "instanceType" = "A2A"
        "agentAutoUpdateStatus" = $AutoUpdateAction
        "automationAccountArmId" = $AutomationAccountArmId
    }
  }
}
$InputJson = $Input |  ConvertTo-Json
if ($ContainerMappingList.Count -eq 0)
{
    Write-Tracing -Level Succeeded -Message ("Exiting as there are no container mappings to be modified.") -DisplayMessageToUser
    exit
}
Write-InformationTracing ("Container mappings to be updated has been retrieved with count: {0}." -f $ContainerMappingList.Count)
try
{
    Write-InformationTracing ("Start the modify container mapping jobs.")
    ForEach($Mapping in $ContainerMappingList)
    {
    try {
            $UpdateUrl = $ArmEndPoint + $Mapping + "?api-version=" + $AsrApiVersion
            Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
            $Result = @()
            Invoke-InternalWebRequest -Uri $UpdateUrl -Headers $Header -Method 'PATCH' `
                -Body $InputJson  -ContentType "application/json" -Result ([ref]$Result)
            $Result = $Result[0]
            $JobAsyncUrl = $Result.Headers['Azure-AsyncOperation']
            Write-InformationTracing ("The modify container mapping job invoked with async url: {0}." -f $JobAsyncUrl)
            $JobsInProgressList += $JobAsyncUrl;
            # Rate controlling the set calls to maximum 60 calls per minute.
            # ASR throttling for set calls is 200 in 1 minute.
            Start-Sleep -Milliseconds 1000
        }
        catch{
            Write-InformationTracing ("The modify container mappings job creation failed for: {0}." -f $Ru)
            Write-InformationTracing $_
            $JobsFailedToStart++
        }
    }
    Write-InformationTracing ("Total modify container mappings has been initiated: {0}." -f $JobsInProgressList.Count)
}
catch
{
    $ErrorMessage = ("Modify container mapping jobs failed with [Exception: {0}]." -f $_.Exception)
    Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
try
{
    while($JobsInProgressList.Count -ne 0)
    {
        Sleep -Seconds 30
        $JobsInProgressListInternal = @()
        ForEach($JobAsyncUrl in $JobsInProgressList)
        {
            try
            {
                Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
                $Result = Invoke-RestMethod -Uri $JobAsyncUrl -Headers $header
                $JobState = $Result.Status
                if($JobState -ieq "InProgress")
                {
                    $JobsInProgressListInternal += $JobAsyncUrl
                }
                elseif($JobState -ieq "Succeeded" -or `
                    $JobState -ieq "PartiallySucceeded" -or `
                    $JobState -ieq "CompletedWithInformation")
                {
                    Write-InformationTracing ("Jobs succeeded with state: {0}." -f $JobState)
                    $JobsCompletedSuccessList += $JobAsyncUrl
                }
                else
                {
                    Write-InformationTracing ("Jobs failed with state: {0}." -f $JobState)
                    $JobsCompletedFailedList += $JobAsyncUrl
                }
            }
            catch
            {
                Write-InformationTracing ("The get job failed with: {0}. Ignoring the exception and retrying the next job." -f $_.Exception)
                # The job on which the tracking failed, will be considered in progress and tried again later.
                $JobsInProgressListInternal += $JobAsyncUrl
            }
            # Rate controlling the get calls to maximum 120 calls each minute.
            # ASR throttling for get calls is 10000 in 60 minutes.
            Start-Sleep -Milliseconds 500
        }
        Write-InformationTracing ("Jobs remaining {0}." -f $JobsInProgressListInternal.Count)
        $CurrentTime = Get-Date
        if($CurrentTime -gt $OperationStartTime.AddMinutes($Timeout))
        {
            Write-InformationTracing ("Tracing modify cloud pairing jobs has timed out.")
            $JobsTimedOut = $JobsInProgressListInternal.Count
            $JobsInProgressListInternal = @()
        }
        $JobsInProgressList = $JobsInProgressListInternal
    }
}
catch
{
    $ErrorMessage = ("Tracking modify cloud pairing jobs failed with [Exception: {0}]." -f $_.Exception)
    Write-Tracing -Level ErrorLevel -Message $ErrorMessage  -DisplayMessageToUser
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
Write-InformationTracing ("Tracking modify cloud pairing jobs completed.")
Write-InformationTracing ("Modify cloud pairing jobs success: {0}." -f $JobsCompletedSuccessList.Count)
Write-InformationTracing ("Modify cloud pairing jobs failed: {0}." -f $JobsCompletedFailedList.Count)
Write-InformationTracing ("Modify cloud pairing jobs failed to start: {0}." -f $JobsFailedToStart)
Write-InformationTracing ("Modify cloud pairing jobs timedout: {0}." -f $JobsTimedOut)
if($JobsTimedOut -gt  0)
{
    $ErrorMessage = "One or more modify cloud pairing jobs has timedout."
    Write-Tracing -Level ErrorLevel -Message ($ErrorMessage)
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
elseif($JobsCompletedSuccessList.Count -ne $ContainerMappingList.Count)
{
    $ErrorMessage = "One or more modify cloud pairing jobs failed."
    Write-Tracing -Level ErrorLevel -Message ($ErrorMessage)
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
Write-Tracing -Level Succeeded -Message ("Modify cloud pairing completed.") -DisplayMessageToUser
```

### <a name="manage-updates-manually"></a>Управление обновлениями вручную

1. Если на виртуальных машинах установлены новые обновления для службы Mobility Service, вы увидите следующее уведомление: **новое Site Recovery доступно обновление агента репликации. Щелкните, чтобы установить.**

   :::image type="content" source="./media/vmware-azure-install-mobility-service/replicated-item-notif.png" alt-text="Окно &quot;Реплицированные элементы&quot;":::

1. Выберите уведомление, чтобы открыть страницу выбора виртуальной машины.
1. Выберите виртуальные машины, которые требуется обновить, и нажмите кнопку **ОК**. Служба обновления Mobility Service будет запущена для каждой выбранной виртуальной машины.

   :::image type="content" source="./media/vmware-azure-install-mobility-service/update-okpng.png" alt-text="Список виртуальных машин &quot;Реплицированные элементы&quot;":::

## <a name="common-issues-and-troubleshooting"></a>Распространенные проблемы и устранение неполадок

Если возникла проблема с автоматическими обновлениями, вы увидите уведомление об ошибке в разделе **проблемы конфигурации** на панели мониторинга хранилища.

Если вы не можете включить автоматическое обновление, ознакомьтесь со следующими распространенными ошибками и рекомендуемыми действиями.

- **Ошибка**: у вас нет разрешений на создание учетной записи запуска от имени Azure (субъекта-службы) и предоставление субъекту-службе роли участника.

  **Рекомендуемое действие**: Убедитесь, что учетная запись для входа назначена как участник, и повторите попытку. Дополнительные сведения о назначении разрешений см. в разделе "необходимые разрешения" статьи [как использовать портал для создания приложения Azure AD и субъекта-службы, которые имеют доступ к ресурсам](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

  Чтобы устранить большинство проблем после включения автоматического обновления, выберите **восстановить**. Если кнопка восстановить недоступна, см. сообщение об ошибке, отображаемое в области Параметры обновления расширений.

  :::image type="content" source="./media/azure-to-azure-autoupdate/repair.png" alt-text="Кнопка восстановления службы Site Recovery в параметрах обновления расширения":::

- **Ошибка**: учетная запись запуска от имени не имеет разрешения на доступ к ресурсу служб восстановления.

  **Рекомендуемое действие**: удалите и [повторно создайте учетную запись запуска от имени](../automation/manage-runas-account.md). Или убедитесь, что Azure Active Directory приложение учетной записи запуска от имени службы автоматизации может получить доступ к ресурсу служб восстановления.

- **Ошибка**: учетная запись запуска от имени не найдена. Не был создан или был удален один из следующих объектов: приложение Azure Active Directory, субъект-служба, роль, ресурс сертификата службы автоматизации, ресурс подключения службы автоматизации. Либо не совпадает отпечаток у сертификата и подключения.

  **Рекомендуемое действие**: удалите и [повторно создайте учетную запись запуска от имени](../automation/manage-runas-account.md).

- **Ошибка**: истекает срок действия сертификата запуска от имени Azure, используемого учетной записью службы автоматизации.

  Срок действия самозаверяющего сертификата, созданного для учетной записи запуска от имени, истекает один год с даты создания. Его можно обновить в любое время до истечения его срока действия. Если вы подписались на уведомления по электронной почте, вы также получаете сообщения электронной почты, когда на стороне вас потребуются какие-либо действия. Эта ошибка будет отображаться на два месяца раньше даты истечения срока действия и изменится на критическую ошибку, если срок действия сертификата истек. После истечения срока действия сертификата автоматическое обновление не будет работать до тех пор, пока вы не обновите его.

  **Рекомендуемое действие**: чтобы устранить эту проблему, выберите **восстановить** , а затем **продлить сертификат**.

  :::image type="content" source="./media/azure-to-azure-autoupdate/automation-account-renew-runas-certificate.PNG" alt-text="продление сертификата":::

  > [!NOTE]
  > После продления сертификата обновите страницу, чтобы отобразить текущее состояние.
