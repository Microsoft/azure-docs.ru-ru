---
title: Настройка службы "Аттестация Azure" с помощью Azure PowerShell
description: Сведения о том, как установить и настроить поставщик аттестации с помощью Azure PowerShell.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: cbc415411e05d6fdecee1acf2fbc02b3c170b9d6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102501130"
---
# <a name="quickstart-set-up-azure-attestation-with-azure-powershell"></a>Краткое руководство. Настройка службы "Аттестация Azure" с помощью Azure PowerShell

Выполнив описанные ниже шаги, вы установите и настроите поставщик аттестации с помощью Azure PowerShell. Сведения о том, как установить и запустить Azure PowerShell, см. в статье [Общие сведения об Azure PowerShell](/powershell/azure/).

Обратите особое внимание, что для коллекции PowerShell не рекомендуется использовать протокол TLS (Transport Layer Security) версий 1.0 и 1.1. Используйте TLS версии 1.2 или более поздней. По этой причине могут появляться следующие сообщения об ошибках.

- ПРЕДУПРЕЖДЕНИЕ. Не удалось разрешить источник пакетов "https://www.powershellgallery.com/api/v2"
- PackageManagement\Install-Package: Для указанных условий поиска и имен модулей не найдено никаких совпадений. 

Чтобы продолжить взаимодействие с коллекцией PowerShell Gallery, выполните следующие команды перед командами Install-Module.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
```

## <a name="install-azattestation-powershell-module"></a>Установка модуля PowerShell Az.Attestation

На компьютере с установленным Azure PowerShell установите модуль PowerShell Az.Attestation, который содержит командлеты для службы "Аттестация Azure".  

### <a name="initial-installation"></a>Начальная установка

Закройте все существующие окна PowerShell.

Чтобы выполнить установку для текущего пользователя, откройте окно PowerShell без прав администратора и выполните в нем следующее:

```powershell
Install-Module -Name Az.Attestation -AllowClobber -Scope CurrentUser
```

Чтобы выполнить установку для всех пользователей, откройте окно PowerShell с правами администратора и выполните в нем следующее:

```powershell
Install-Module -Name Az.Attestation -AllowClobber -Scope AllUsers
```

Закройте консоль PowerShell с правами администратора.

### <a name="update-the-installation"></a>Обновление установки

Закройте все существующие окна PowerShell.

Чтобы выполнить обновление для текущего пользователя, откройте окно PowerShell без прав администратора и выполните в нем следующее:

```powershell
Update-Module -Name Az.Attestation
```

Чтобы выполнить обновление для всех пользователей, откройте окно PowerShell с правами администратора и выполните в нем следующее:

```powershell
Update-Module -Name Az.Attestation
```

Закройте консоль PowerShell с правами администратора.

### <a name="get-installed-modules"></a>Получение установленных модулей

Минимально необходимые версии модулей Az для поддержки операций с аттестацией:
- Az 4.5.0;
- Az.Accounts 1.9.2;
- Az.Attestation 0.1.8.

Выполните приведенную ниже команду, чтобы проверить версии всех установленных модулей Az. 

```powershell
Get-InstalledModule
```
Если какие-то версии не соответствуют минимальным требованиям, выполните команды Update-Module.

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в Azure из консоли PowerShell (без повышенных прав доступа).

```powershell
Connect-AzAccount
```

Если нужно, перейдите в ту подписку, в которой будет использоваться служба "Аттестация Azure".

```powershell
Set-AzContext -Subscription <subscription id>  
```

## <a name="register-microsoftattestation-resource-provider"></a>Регистрация поставщика ресурсов Microsoft.Attestation

Зарегистрируйте в подписке поставщик ресурсов Microsoft.Attestation. Дополнительные сведения о поставщиках ресурсов Azure, о настройке этих поставщиков и управлении ими см. в статье [Поставщики и типы ресурсов Azure](../azure-resource-manager/management/resource-providers-and-types.md). Обратите внимание, что регистрацию поставщика ресурсов для подписки нужно выполнять только один раз.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Attestation
```
## <a name="regional-availability-of-azure-attestation"></a>Доступность службы "Аттестация Azure" по регионам

```powershell
(Get-AzResourceProvider -ProviderNamespace Microsoft.Attestation)[0].Locations
```

## <a name="create-an-azure-resource-group"></a>создание группы ресурсов Azure;

Создайте группу ресурсов для поставщика аттестации. Обратите внимание, что в эту же группу ресурсов можно поместить и другие ресурсы Azure (в том числе виртуальные машины с экземплярами клиентского приложения).

```powershell
$location = "uksouth" 
$attestationResourceGroup = "<attestation provider resource group name>"
New-AzResourceGroup -Name $attestationResourceGroup -Location $location 
```

## <a name="create-and-manage-an-attestation-provider"></a>Создание поставщика аттестации и управление им

Командлет New-AzAttestation создает поставщик аттестации.

```powershell
$attestationProvider = "<attestation provider name>" 
New-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Location $location
```

PolicySignerCertificateFile позволяет определить файл для набора доверенных ключей для подписи. Если в параметре PolicySignerCertificateFile указано имя файла, для поставщика аттестации удастся настроить политики только в формате подписанного маркера JWT. В остальных случаях для политик можно использовать форматы простого текста или неподписанного маркера JWT.

```powershell
New-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Location $location -PolicySignersCertificateFile "C:\test\policySignersCertificates.pem"
```

Пример использования параметра PolicySignersCertificateFile можно найти в [примерах сертификатов для подписывания политик](policy-signer-examples.md).

Командлет Get-AzAttestation извлекает такие свойства поставщика аттестации, как текущее состояние и AttestURI. Запишите значение AttestURI, которое вам потребуется позднее.

```azurepowershell
Get-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup  
```

Приведенная выше команда должна возвращать примерно такой результат: 

```
Id:/subscriptions/MySubscriptionID/resourceGroups/MyResourceGroup/providers/Microsoft.Attestation/attestationProviders/MyAttestationProvider
Location: MyLocation
ResourceGroupName: MyResourceGroup
Name: MyAttestationProvider
Status: Ready
TrustModel: AAD
AttestUri: https://MyAttestationProvider.us.attest.azure.net 
Tags: 
TagsTable: 
```

С помощью командлета Remove-AzAttestation можно удалить поставщик аттестации.  

```powershell
Remove-AzAttestation -Name $attestationProvider -ResourceGroupName $attestationResourceGroup
```

## <a name="policy-management"></a>Управление политикой

Для управления политиками пользователю Azure AD требуются следующие разрешения для действий:
- Microsoft.Attestation/attestationProviders/attestation/read
- Microsoft.Attestation/attestationProviders/attestation/write
- Microsoft.Attestation/attestationProviders/attestation/delete

Эти разрешения могут назначаться пользователю AD с помощью роли "Владелец" (разрешения с подстановочными знаками), "Участник" (разрешения с подстановочными знаками) или "Участник аттестации" (только определенные разрешения для Аттестации Azure).  

Для чтения политик пользователю Azure AD требуются следующие разрешения для действий:
- Microsoft.Attestation/attestationProviders/attestation/read

Это разрешение может назначаться пользователю AD с помощью роли "Читатель" (разрешения с подстановочными знаками) или "Читатель аттестации" (только определенные разрешения для Аттестации Azure).

Приведенные ниже командлеты PowerShell позволяют управлять политиками для поставщика аттестации (одна среда TEE за раз).

Командлет Get-AzAttestationPolicy возвращает текущую политику для указанной среды TEE. Командлет отображает политику в текстовом формате и формате JWT политики.

```powershell
$teeType = "<tee Type>"
Get-AzAttestationPolicy   -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType 
```

Для TEE поддерживаются типы SgxEnclave, OpenEnclave и VbsEnclave.

Командлет Set-AttestationPolicy задает новую политику для указанного TEE. Этот командлет принимает политику в формате простого текста или маркера JWT, а его поведением управляет параметр PolicyFormat. По умолчанию PolicyFormat имеет значение Text. 

```powershell
$policyFormat = "<policy format>"
$policy=Get-Content -path "C:\test\policy.txt" -Raw
Set-AzAttestationPolicy   -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType -Policy $policy -PolicyFormat $policyFormat 
```

Если при создании поставщика аттестации указать параметр PolicySignerCertificateFile, политики можно будет настраивать только в формате маркера JWT. В остальных случаях для политик можно использовать форматы простого текста или неподписанного маркера JWT.

Политика аттестации в формате JWT должна содержать утверждение с именем AttestationPolicy. Если используется подписанная политика, для подписывания JWT нужно применить закрытый ключ, соответствующий любому из существующих сертификатов для подписывания политик.

Примеры политики см. в [этом разделе](policy-examples.md).

Командлет Reset-AzAttestationPolicy сбрасывает значения политики до значений по умолчанию для указанного TEE.

```powershell
Reset-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Tee $teeType 
```

## <a name="policy-signer-certificates-management"></a>Управление сертификатами для подписывания политик

Приведенные ниже командлеты PowerShell позволяют управлять сертификатами для подписывания политик для поставщика аттестации:

```powershell
Get-AzAttestationPolicySigners -Name $attestationProvider -ResourceGroupName $attestationResourceGroup

Add-AzAttestationPolicySigner -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Signer <signer>

Remove-AzAttestationPolicySigner -Name $attestationProvider -ResourceGroupName $attestationResourceGroup -Signer <signer>
```

Сертификат для подписывания политик представляет собой подписанный маркер JWT, в котором есть утверждение с именем maa-policyCertificate. Значением этого утверждения является объект JWK, который содержит добавляемый доверенный ключ для подписывания. Для подписывания JWT нужно применить закрытый ключ, соответствующий любому из существующих сертификатов для подписывания политик.

Обратите внимание, что все семантические операции с сертификатом для подписывания политик следует выполнять за пределами PowerShell. Для среды PowerShell он является простой строкой.

Пример использования сертификата для подписывания политик см. [здесь](policy-signer-examples.md).

Дополнительные сведения о командлетах и их параметрах см. в статье [Командлеты PowerShell для службы "Аттестация Azure"](/powershell/module/az.attestation/#attestation). 

## <a name="next-steps"></a>Дальнейшие действия

- [Как создать и подписать файл политики аттестации](author-sign-policy.md)
- [Аттестация анклава SGX с помощью примеров кода](/samples/browse/?expanded=azure&terms=attestation)
