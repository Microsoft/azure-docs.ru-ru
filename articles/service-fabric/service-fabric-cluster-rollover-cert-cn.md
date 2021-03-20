---
title: Переносите сертификат кластера Azure Service Fabric
description: Узнайте, как выполнить переработку Service Fabric сертификата кластера, определяемого общим именем сертификата.
ms.topic: conceptual
ms.date: 09/06/2019
ms.openlocfilehash: 65ea4f463073c472ac6a31e62dcfdfd11cb28cc5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88853352"
---
# <a name="manually-roll-over-a-service-fabric-cluster-certificate"></a>Смена сертификата для кластера Service Fabric вручную
Если срок действия для сертификата кластера Service Fabric подходит к концу, необходимо обновить сертификат.  Сменить сертификат не сложно, если кластер [настроен для использования сертификатов на основе общего имени](service-fabric-cluster-change-cert-thumbprint-to-cn.md) (вместо отпечатка).  Получите новый сертификат из центра сертификации с новой датой окончания срока действия.  Самозаверяющие сертификаты не поддерживаются производственными кластерами Service Fabric, включая сертификаты, созданные при выполнении рабочего процесса создания кластера на портале Azure. Новый сертификат должен иметь то же общее имя, что и старый. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Когда на узле установлено несколько сертификатов проверки, кластер Service Fabric будет автоматически использовать объявленный сертификат с дальнейшей датой истечения срока действия. Для подготовки ресурсов Azure мы рекомендуем использовать шаблон Resource Manager. Для непроизводственной среды приведенный ниже сценарий можно использовать, чтобы отправить новый сертификат в хранилище ключей. Затем он устанавливается в масштабируемом наборе виртуальных машин. 

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

$SubscriptionId  =  <subscription ID>

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $SubscriptionId

$region = "southcentralus"
$KeyVaultResourceGroupName  = "keyvaultgroup"
$VaultName = "cntestvault2"
$certFilename = "C:\users\sfuser\sftutorialcluster20180419110824.pfx"
$certname = "cntestcert"
$Password  = "!P@ssw0rd321"
$VmssResourceGroupName     = "sfclustertutorialgroup"
$VmssName                  = "prnninnxj"

# Create new Resource Group 
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Get the key vault.  The key vault must be enabled for deployment.
$keyVault = Get-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName 
$resourceId = $keyVault.ResourceId  

# Add the certificate to the key vault.
$PasswordSec = ConvertTo-SecureString -String $Password -AsPlainText -Force
$KVSecret = Import-AzKeyVaultCertificate -VaultName $vaultName -Name $certName  -FilePath $certFilename -Password $PasswordSec

$CertificateThumbprint = $KVSecret.Thumbprint
$CertificateURL = $KVSecret.SecretId
$SourceVault = $resourceId
$CommName    = $KVSecret.Certificate.SubjectName.Name

Write-Host "CertificateThumbprint    :"  $CertificateThumbprint
Write-Host "CertificateURL           :"  $CertificateURL
Write-Host "SourceVault              :"  $SourceVault
Write-Host "Common Name              :"  $CommName    

Set-StrictMode -Version 3
$ErrorActionPreference = "Stop"

$certConfig = New-AzVmssVaultCertificateConfig -CertificateUrl $CertificateURL -CertificateStore "My"

# Get current VM scale set 
$vmss = Get-AzVmss -ResourceGroupName $VmssResourceGroupName -VMScaleSetName $VmssName

# Add new secret to the VM scale set.
$vmss.VirtualMachineProfile.OsProfile.Secrets[0].VaultCertificates.Add($certConfig)

# Update the VM scale set 
Update-AzVmss -ResourceGroupName $VmssResourceGroupName -Name $VmssName -VirtualMachineScaleSet $vmss  -Verbose
```

>[!NOTE]
> Так как каждый секрет является уникальным ресурсом с контролем версий, секреты масштабируемого набора виртуальных машин в Вычислительных средах не поддерживают один идентификатор ресурса для двух отдельных секретов. 

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о [безопасности кластера](service-fabric-cluster-security.md).
* [Обновление сертификатов кластера и управление ими](service-fabric-cluster-security-update-certs-azure.md)
