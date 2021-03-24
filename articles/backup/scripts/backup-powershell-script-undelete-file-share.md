---
title: Сценарий PowerShell — отмена удаления общей папки
description: Узнайте, как использовать сценарий Azure PowerShell, чтобы отменить случайное удаление общей папки.
ms.topic: sample
ms.date: 02/02/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 99cd980e13b3b9b9a7a450520218c76ccf26fd69
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89079869"
---
# <a name="powershell-script-to-undelete-an-accidentally-deleted-file-share"></a>Сценарий PowerShell для отмены случайного удаления общей папки

Этот сценарий помогает отменить удаление общей папки, если она была удалена случайно. Функция безопасности обратимого удаления для общей папки предоставляет возможность отменить удаление общей папки в течение 14-дневного периода хранения, позволяя восстановить все содержимое общей папки, моментальные снимки и точки восстановления. Чтобы получить дополнительные сведения об обратимом удалении, перейдите по этой [ссылке](../soft-delete-azure-file-share.md).

## <a name="sample-script"></a>Пример скрипта

```powershell
#Import-Module Az.Storage -MinimumVersion 1.7.0 -Scope Local
Param(
        [Parameter(Mandatory=$True)][System.String] $ResourceGroupName,
        [Parameter(Mandatory=$True)][System.String] $StorageAccountName,
        [Parameter(Mandatory=$True)][System.String] $FileShareName,
        [Parameter(Mandatory=$True)][System.String] $SubscriptionId,
        [Parameter(Mandatory=$False)][System.Boolean] $ListOption,
        [Parameter(Mandatory=$False)][System.String] $DeletedShareVersion
    )

Function Restore-DeletedFileShare
{
    Param(
        [Parameter(Mandatory=$True)][Microsoft.WindowsAzure.Commands.Common.Storage.LazyAzureStorageContext] $Context,
        [Parameter(Mandatory=$True)][System.String] $FileShareName,
        [Parameter(Mandatory=$False)][System.String] $DeletedShareVersion
        )

    if ([string]::IsNullOrWhiteSpace($FileShareName))
    {
        Write-Error "Please specify the required input parameter: FileShareName" -ErrorAction Stop
    }

    $FileShareName = $FileShareName.ToLowerInvariant()

    Write-Verbose "Restoring a file share with the name: $FileShareName" -Verbose


    Write-Information -MessageData "Started: Creating SASToken to List File Shares" -InformationAction Continue

    $listToken = New-AzStorageAccountSASToken -Context $Context -Service File -ResourceType Service -Permission "l" -Protocol HttpsOrHttp -StartTime (Get-Date).AddHours(-1) -ExpiryTime (Get-Date).AddHours(1)

    Write-Information -MessageData "Completed: Creating SASToken to List File Shares" -InformationAction Continue

    Write-Information -MessageData "Started: Listing File Shares to find the deleted file share" -InformationAction Continue

    $listSharesUrl = [string]::Concat($Context.FileEndPoint, "?include=metadata,deleted&comp=list&api-version=2019-10-10&", $listToken.Substring(1))

    $listSharesResponse = Invoke-WebRequest $listSharesUrl -Method "GET" -Verbose

    if ($listSharesResponse.StatusCode -ne 200)
    {
        Write-Error "Request to list file shares failed." -ErrorAction Stop
    }

    Write-Verbose $listSharesResponse.RawContent -Verbose

    $listSharesResponseContent = $listSharesResponse.Content.Substring(3)

    Write-Information -MessageData "Completed: Listing File Shares to find the deleted file share" -InformationAction Continue

    Write-Information -MessageData "Started: Search for a deleted file share with the specified name" -InformationAction Continue

    $deletedFileShares = Select-Xml -Content $listSharesResponseContent -XPath "/EnumerationResults/Shares/Share[Deleted=""true"" and Name=""$FileShareName""]"

    $matchedCount = 0
    $deletedShareVersions = New-Object System.Collections.Generic.List[string]

    foreach($share in $deletedFileShares)
    {
        if($matchedCount -eq 0)
        {
          Write-Verbose $share.Node.InnerXml -Verbose

          Write-Information -MessageData "Completed: Search for a deleted file share with the specified name And Found versions" -InformationAction Continue
        }

        $shareVer = $share.Node.Item("Version").InnerText
        $shareDelTime = $share.Node.Item("Properties").Item("DeletedTime").InnerText
        $retDays = $share.Node.Item("Properties").Item("RemainingRetentionDays").InnerText

        $deletedShareVersions.Add($share.Node.Item("Version").InnerText)

        Write-Information -MessageData "DeletedVersion: $shareVer, DeletedTime: $shareDelTime, RemainingRetentionDays: $retDays"  -InformationAction Continue

        $matchedCount++
    }

    if($ListOption -eq $True)
    {
       return;
    }

    if ($matchedCount -eq 0)
    {
        Write-Error "Deleted file share with the specified name was not found." -ErrorAction Stop
    }
    elseif($matchedCount -eq 1 -and ([string]::IsNullOrWhiteSpace($DeletedShareVersion) -or $deletedShareVersions.Contains($DeletedShareVersion)))
    {
      $DeletedShareVersion = $deletedShareVersions
    }
    elseif ($matchedCount -gt 1)
    {
      if ([string]::IsNullOrWhiteSpace($DeletedShareVersion) -or !$deletedShareVersions.Contains($DeletedShareVersion))
      {
        Write-Error "More than one share with the specified name was found. Please specify a valid DeletedShareVersion parameter from above possible values." -ErrorAction Stop
      }
    }

    Write-Information -MessageData "Completed: Search for a deleted file share with the specified name And Found version: $DeletedShareVersion" -InformationAction Continue

    Write-Information -MessageData "Started: Creating SASToken to Restore File Share" -InformationAction Continue

    $restoreToken = New-AzStorageAccountSASToken -Context $Context -Service File -ResourceType Container -Permission "w" -Protocol HttpsOrHttp -StartTime (Get-Date).AddHours(-1) -ExpiryTime (Get-Date).AddHours(1)

    Write-Information -MessageData "Completed: Creating SASToken to Restore File Share" -InformationAction Continue

    Write-Information -MessageData "Started: Restore File Share" -InformationAction Continue

 $restoreShareUrl = [string]::Concat($Context.FileEndPoint, $FileShareName, "?restype=share&comp=undelete&api-version=2019-10-10&", $restoreToken.Substring(1))

    $restoreHeaders = @{"x-ms-deleted-share-name" = $FileShareName; "x-ms-deleted-share-version" = $DeletedShareVersion}

    $restoreResponse = Invoke-WebRequest $restoreShareUrl -Headers $restoreHeaders -Method "PUT" -Verbose

    if ($restoreResponse.StatusCode -ne 201)
    {
        Write-Error "Request to restore a file share failed." -ErrorAction Stop
    }

    Write-Verbose $restoreResponse.RawContent -Verbose

    Write-Information -MessageData "Completed: Restore File Share" -InformationAction Continue
}

Connect-AzAccount
Select-AzSubscription -Subscription $SubscriptionId
$sa = Get-AzStorageAccount -ResourceGroupName $ResourceGroupName -Name $StorageAccountName


Restore-DeletedFileShare $sa.Context $FileShareName $DeletedShareVersion
```

## <a name="how-to-use-the-script-in-different-scenarios"></a>Использование сценария в различных ситуациях

### <a name="prerequisites"></a>Предварительные требования

1. Перед запуском сценария установите последнюю версию модулей Azure PowerShell Az (перейдите по [этой ссылке](/powershell/azure/install-az-ps)).
2. Держите следующие сведения под рукой, так как их нужно будет указать в качестве значений параметров сценария:

    * **-SubscriptionId** — идентификатор подписки, к которой относится общая папка.
    * **-ResourceGroupName** — группа ресурсов учетной записи хранения, в которой размещена общая папка.
    * **-StorageAccountName** — имя учетной записи хранения, к которой относится общая папка.
    * **-FileShareName** — имя общей папки, удаление которой необходимо отменить.

### <a name="execution-steps"></a>Шаги выполнения

1. Сохраните приведенный выше сценарий на компьютере, назвав его на свое усмотрение. В этом примере мы сохранили его под именем *Undelete.ps1*
2. Запустите сценарий в соответствии с ситуацией, которая подходит под ваши требования.

#### <a name="scenario-1"></a>Сценарий 1

Существует единственная удаленная версия с тем же именем, что и у общей папки, удаление которой вы пытаетесь отменить.

В следующем примере отменяется удаление общей папки *share1* из учетной записи хранения *afsshare*.

```powershell
   .\UnDelete.ps1 -ResourceGroupName afsshare -StorageAccountName afsshare -SubscriptionId f75d8d8b-6735-4697-82e1-1a7a3ff0d5d4 -FileShareName share1
```

В результате должно отобразится сообщение `Completed:Restore File Share`

#### <a name="scenario-2"></a>Сценарий 2

Существует несколько удаленных версий с тем же именем, что и у общей папки, удаление которой вы пытаетесь отменить.

В следующем примере отменяется удаление версии общей папки *share1*

##### <a name="step-1"></a>Шаг 1

Выполните сценарий, указав имя общей папки.

```PowerShell
   .\UnDelete.ps1 -ResourceGroupName afsshare -StorageAccountName afsshare -SubscriptionId f75d8d8b-6735-4697-82e1-1a7a3ff0d5d4 -FileShareName share1
```

```Output
Completed: Search for a deleted file share with the specified name and Found versions
DeletedVersion: 01D5D7F77ACC7864, DeletedTime: Fri, 31 Jan 2020 05:30:33 GMT, RemainingRetentionDays: 14
DeletedVersion: 01D5D7F7A76CAF42, DeletedTime: Fri, 31 Jan 2020 05:31:25 GMT, RemainingRetentionDays: 14
Restore-DeletedFileShare : More than one share with the specified name was found. Please specify a valid DeletedShareVersion parameter from above possible values.
```

##### <a name="step-2"></a>Шаг 2

Выберите одну из версий, указанных в результате выполнения шага 1, удаление которой необходимо отменить, и укажите ее в качестве значения параметра **-DeletedShareVersion**.

В следующем примере отменяется удаление версии *01D5D7F77ACC7864* общей папки *share1*.

```powershell
   .\UnDelete.ps1 -ResourceGroupName afsshare-StorageAccountName afsshare -SubscriptionId f75d8d8b-6735-4697-82e1-1a7a3ff0d5d4 -FileShareName share1 -DeletedShareVersion 01D5D7F77ACC7864
```
