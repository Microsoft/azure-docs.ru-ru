---
title: Развертывание Облачной службы (расширенная поддержка). PowerShell
description: Развертывание Облачной службы (расширенная поддержка) с помощью PowerShell
ms.topic: tutorial
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: bcf6b2f6b964a056b9d90f08c0586fcbdec5b260
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106167283"
---
# <a name="deploy-a-cloud-service-extended-support-using-azure-powershell"></a>Развертывание Облачной службы (расширенная поддержка) с помощью PowerShell

В этой статье описано, как использовать модуль PowerShell `Az.CloudService` для развертывания Облачных служб (расширенная поддержка) в Azure с несколькими ролями (WebRole и WorkerRole) и расширением удаленного рабочего стола. 

## <a name="before-you-begin"></a>Подготовка к работе

Ознакомьтесь с [предварительными требованиями развертывания](deploy-prerequisite.md) для Облачных служб (расширенная поддержка) и создайте связанные ресурсы. 

## <a name="deploy-a-cloud-services-extended-support"></a>Развертывание Облачных служб (расширенная поддержка)
1. Установите модуль Az.CloudService PowerShell  

    ```powershell
    Install-Module -Name Az.CloudService 
    ```

2. Создание группы ресурсов Этот шаг можно пропустить, если вы используете имеющуюся группу ресурсов.   

    ```powershell
    New-AzResourceGroup -ResourceGroupName “ContosOrg” -Location “East US” 
    ```

3. Создайте учетную запись хранения и контейнер, которые будут использоваться для хранения CSCFG-файлов пакета Облачной службы и CSDEF-файлов конфигурации службы. Используйте для учетной записи хранения уникальное имя. 

    ```powershell
    $storageAccount = New-AzStorageAccount -ResourceGroupName “ContosOrg” -Name “contosostorageaccount” -Location “East US” -SkuName “Standard_RAGRS” -Kind “StorageV2” 
    $container = New-AzStorageContainer -Name “contosocontainer” -Context $storageAccount.Context -Permission Blob 
    ```

4. Отправьте пакет Облачной службы (CSPKG-файл) в учетную запись хранения.

    ```powershell
    $tokenStartTime = Get-Date 
    $tokenEndTime = $tokenStartTime.AddYears(1) 
    $cspkgBlob = Set-AzStorageBlobContent -File “./ContosoApp/ContosoApp.cspkg” -Container “contosocontainer” -Blob “ContosoApp.cspkg” -Context $storageAccount.Context 
    $cspkgToken = New-AzStorageBlobSASToken -Container “contosocontainer” -Blob $cspkgBlob.Name -Permission rwd -StartTime $tokenStartTime -ExpiryTime $tokenEndTime -Context $storageAccount.Context 
    $cspkgUrl = $cspkgBlob.ICloudBlob.Uri.AbsoluteUri + $cspkgToken 
    ```
 

5.  Отправьте конфигурацию облачной службы (CSCFG-файл) в учетную запись хранения. 

    ```powershell
    $cscfgBlob = Set-AzStorageBlobContent -File “./ContosoApp/ContosoApp.cscfg” -Container contosocontainer -Blob “ContosoApp.cscfg” -Context $storageAccount.Context 
    $cscfgToken = New-AzStorageBlobSASToken -Container “contosocontainer” -Blob $cscfgBlob.Name -Permission rwd -StartTime $tokenStartTime -ExpiryTime $tokenEndTime -Context $storageAccount.Context 
    $cscfgUrl = $cscfgBlob.ICloudBlob.Uri.AbsoluteUri + $cscfgToken 
    ```

6. Создайте виртуальную сеть и подсеть. Этот шаг можно пропустить, если используется имеющаяся сеть и подсеть. В этом примере используются одна виртуальная сеть и подсеть для обеих ролей облачной службы (WebRole и WorkerRole). 

    ```powershell
    $subnet = New-AzVirtualNetworkSubnetConfig -Name "ContosoWebTier1" -AddressPrefix "10.0.0.0/24" -WarningAction SilentlyContinue 
    $virtualNetwork = New-AzVirtualNetwork -Name “ContosoVNet” -Location “East US” -ResourceGroupName “ContosOrg” -AddressPrefix "10.0.0.0/24" -Subnet $subnet 
    ```
 
7. Создайте общедоступный IP-адрес и задайте свойство метки DNS для этого адреса. Облачные службы (расширенная поддержка) поддерживают общедоступные IP-адреса с номерами SKU только ценовой категории "Базовый" (https://docs.microsoft.com/azure/virtual-network/public-ip-addresses#basic) ). Общедоступные IP-адреса с номерами SKU категории "Стандартный" не работают с облачными службами.
Если вы используете статический IP-адрес, он должен быть указан как зарезервированный IP-адрес в CSCFG-файле конфигурации службы. 

    ```powershell
    $publicIp = New-AzPublicIpAddress -Name “ContosIp” -ResourceGroupName “ContosOrg” -Location “East US” -AllocationMethod Dynamic -IpAddressVersion IPv4 -DomainNameLabel “contosoappdns” -Sku Basic 
    ```

8. Создайте объект сетевого профиля и свяжите общедоступный IP-адрес с внешним интерфейсом подсистемы балансировки нагрузки. Платформа Azure автоматически создает подсистему балансировки нагрузки с номером SKU уровня "Классический" в той же подписке, что и ресурс облачной службы. Ресурс подсистемы балансировки нагрузки представляет собой ресурс только для чтения в ARM. Все обновления ресурса поддерживаются только через файлы развертывания облачной службы (.cscfg и .csdef).

    ```powershell
    $publicIP = Get-AzPublicIpAddress -ResourceGroupName ContosOrg -Name ContosIp  
    $feIpConfig = New-AzCloudServiceLoadBalancerFrontendIPConfigurationObject -Name 'ContosoFe' -PublicIPAddressId $publicIP.Id 
    $loadBalancerConfig = New-AzCloudServiceLoadBalancerConfigurationObject -Name 'ContosoLB' -FrontendIPConfiguration $feIpConfig 
    $networkProfile = @{loadBalancerConfiguration = $loadBalancerConfig} 
    ```
 
9. создание Key Vault; Этот Key Vault будет использоваться для хранения сертификатов, связанных с ролями Облачной службы (расширенная поддержка). Key Vault должен находиться в тех же регионе и подписке, что и облачная служба, и иметь уникальное имя. Дополнительные сведения см. в статье [Использование сертификатов с Облачными службами Azure (расширенная поддержка)](certificates-and-key-vault.md).

    ```powershell
    New-AzKeyVault -Name "ContosKeyVault” -ResourceGroupName “ContosOrg” -Location “East US” 
    ```

10. Обновите политику доступа Key Vault и предоставьте учетной записи пользователя разрешения на доступ к сертификату. 

    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName 'ContosKeyVault' -ResourceGroupName 'ContosOrg' -EnabledForDeployment
    Set-AzKeyVaultAccessPolicy -VaultName 'ContosKeyVault' -ResourceGroupName 'ContosOrg' -UserPrincipalName 'user@domain.com' -PermissionsToCertificates create,get,list,delete 
    ```

    Кроме того, можно задать политику доступа через ObjectId (его можно получить, запустив командлет `Get-AzADUser`) 
    
    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName 'ContosKeyVault' -ResourceGroupName 'ContosOrg' -ObjectId 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' -PermissionsToCertificates create,get,list,delete 
    ```
 

11. В этом примере мы добавим в Key Vault самозаверяющий сертификат. Отпечаток этого сертификата необходимо добавить в CSCFG-файл конфигурации Облачной службы для развертывания в ролях облачной службы. 

    ```powershell
    $Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "Self" -ValidityInMonths 6 -ReuseKeyOnRenewal 
    Add-AzKeyVaultCertificate -VaultName "ContosKeyVault" -Name "ContosCert" -CertificatePolicy $Policy 
    ```
 
12. Создайте объект профиля ОС в памяти. Профиль ОС определяет сертификаты, связанные с ролями облачной службы. Этим сертификатом будет сертификат, созданный на предыдущем шаге. 

    ```powershell
    $keyVault = Get-AzKeyVault -ResourceGroupName ContosOrg -VaultName ContosKeyVault 
    $certificate = Get-AzKeyVaultCertificate -VaultName ContosKeyVault -Name ContosCert 
    $secretGroup = New-AzCloudServiceVaultSecretGroupObject -Id $keyVault.ResourceId -CertificateUrl $certificate.SecretId 
    $osProfile = @{secret = @($secretGroup)} 
    ```

13. Создайте объект профиля роли в памяти. Профиль роли определяет свойства конкретной ценовой категории роли, такие как имя, емкость и уровень. В этом примере определены две роли: frontendRole и backendRole. Сведения о профиле роли должны соответствовать конфигурации роли, определенной в CSCFG-файле конфигурации и CSDEF-файле определения службы. 

    ```powershell
    $frontendRole = New-AzCloudServiceRoleProfilePropertiesObject -Name 'ContosoFrontend' -SkuName 'Standard_D1_v2' -SkuTier 'Standard' -SkuCapacity 2 
    $backendRole = New-AzCloudServiceRoleProfilePropertiesObject -Name 'ContosoBackend' -SkuName 'Standard_D1_v2' -SkuTier 'Standard' -SkuCapacity 2 
    $roleProfile = @{role = @($frontendRole, $backendRole)} 
    ```

14. (Необязательно) Создайте объект профиля расширения в памяти, который требуется добавить в облачную службу. В этом примере мы добавим расширение RDP. 

    ```powershell
    $credential = Get-Credential 
    $expiration = (Get-Date).AddYears(1) 
    $rdpExtension = New-AzCloudServiceRemoteDesktopExtensionObject -Name 'RDPExtension' -Credential $credential -Expiration $expiration -TypeHandlerVersion '1.2.1' 

    $storageAccountKey = Get-AzStorageAccountKey -ResourceGroupName "ContosOrg" -Name "contosostorageaccount"
    $configFile = "<WAD public configuration file path>"
    $wadExtension = New-AzCloudServiceDiagnosticsExtension -Name "WADExtension" -ResourceGroupName "ContosOrg" -CloudServiceName "ContosCS" -StorageAccountName "contosostorageaccount" -StorageAccountKey $storageAccountKey[0].Value -DiagnosticsConfigurationPath $configFile -TypeHandlerVersion "1.5" -AutoUpgradeMinorVersion $true 
    $extensionProfile = @{extension = @($rdpExtension, $wadExtension)} 
    ```
    Обратите внимание, что configFile должен иметь только теги PublicConfig и содержать следующее пространство имен:
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        ...............
    </PublicConfig>
    ```
15. (Необязательно) Определите теги как хэш-таблицу PowerShell, которую требуется добавить в облачную службу. 

    ```powershell
    $tag=@{"Owner" = "Contoso"} 
    ```

17. Создайте развертывание Облачной службы, используя объекты профиля и URL-адреса SAS.

    ```powershell
    $cloudService = New-AzCloudService ` 
    -Name “ContosoCS” ` 
    -ResourceGroupName “ContosOrg” ` 
    -Location “East US” ` 
    -PackageUrl $cspkgUrl ` 
    -ConfigurationUrl $cscfgUrl ` 
    -UpgradeMode 'Auto' ` 
    -RoleProfile $roleProfile ` 
    -NetworkProfile $networkProfile  ` 
    -ExtensionProfile $extensionProfile ` 
    -OSProfile $osProfile `
    -Tag $tag 
    ```

## <a name="next-steps"></a>Дальнейшие действия 
- Ознакомьтесь с [часто задаваемыми вопросами об Облачных службах (расширенная поддержка)](faq.md).
- Разверните Облачную службу (расширенная поддержка) с помощью [портала Azure](deploy-portal.md), [PowerShell](deploy-powershell.md), [шаблона](deploy-template.md) или [Visual Studio](deploy-visual-studio.md).
- Перейдите в [репозиторий примеров для Облачных служб (расширенная поддержка) ](https://github.com/Azure-Samples/cloud-services-extended-support)
