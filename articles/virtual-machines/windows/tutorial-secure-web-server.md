---
title: Руководство по защите веб-сервера Windows с помощью TLS/SSL-сертификатов в Azure
description: В этом руководстве описано, как использовать Azure PowerShell для защиты виртуальной машины Windows, запущенной на веб-сервере IIS, с помощью TLS/SSL-сертификатов, хранимых в Azure Key Vault.
author: cynthn
ms.service: virtual-machines
ms.collection: windows
ms.subservice: security
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 02/09/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: a1009d24c1fb8712e88075097460ad179629f478
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102557409"
---
# <a name="tutorial-secure-a-web-server-on-a-windows-virtual-machine-in-azure-with-tlsssl-certificates-stored-in-key-vault"></a>Руководство по защите веб-сервера на виртуальной машине Windows в Azure с помощью TLS/SSL-сертификатов, хранимых в Key Vault

> [!NOTE]
> Сейчас это руководство подходит только для универсальных образов. Если попытаться выполнить инструкции, приведенные в этом руководстве, со специализированным диском, появится сообщение об ошибке. 

Чтобы защитить веб-серверы, для шифрования веб-трафика можно использовать протокол TLS, который ранее назывался протоколом SSL. TLS/SSL-сертификаты могут храниться в Azure Key Vault и разрешать безопасное развертывание сертификатов на виртуальных машинах Windows в Azure. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * создать Azure Key Vault;
> * создать или передать сертификат в Key Vault;
> * Создание виртуальной машины и установка веб-сервера IIS
> * внедрение сертификата в виртуальную машину и настройка IIS с помощью TLS-привязки.


## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.


## <a name="overview"></a>Обзор
Azure Key Vault защищает криптографические ключи и секреты, в том числе сертификаты или пароли. Key Vault помогает оптимизировать управление сертификатами и позволяет контролировать ключи, которые предоставляют доступ к этим сертификатам. Можно создать самозаверяющий сертификат в Key Vault или передать существующий доверенный сертификат.

Вместо того чтобы использовать образ виртуальной машины, включающий встроенные сертификаты, можно внедрить сертификаты в работающую виртуальную машину. Этот процесс гарантирует установку на веб-сервер самых последних сертификатов во время развертывания. Если вы обновляете или заменяете сертификат, вам не нужно создавать новый пользовательский образ виртуальной машины. Последние сертификаты внедряются автоматически при создании дополнительных виртуальных машин. При этом сертификат никогда не покидает платформу Azure и не отображается в скрипте, журнале командной строки или шаблоне.


## <a name="create-an-azure-key-vault"></a>создать Azure Key Vault;
Прежде чем создать хранилище ключей и сертификаты, выполните командлет [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup), чтобы создать группу ресурсов. В следующем примере создается имя группы ресурсов *myResourceGroupSecureWeb* в расположении *восточная часть США*:

```azurepowershell-interactive
$resourceGroup = "myResourceGroupSecureWeb"
$location = "East US"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

Далее, чтобы создать Key Vault, используйте командлет [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). У каждого Key Vault должно быть уникальное имя в нижнем регистре. Замените `mykeyvault` в следующем примере собственным уникальным именем Key Vault.

```azurepowershell-interactive
$keyvaultName="mykeyvault"
New-AzKeyVault -VaultName $keyvaultName `
    -ResourceGroup $resourceGroup `
    -Location $location `
    -EnabledForDeployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>Создание сертификата и его сохранение в Key Vault
Для использования в рабочей среде импортируйте действительный сертификат, подписанный доверенным поставщиком, выполнив командлет [Import-AzureKeyVaultCertificate](/powershell/module/az.keyvault/import-azkeyvaultcertificate). В следующем примере показано, как с помощью командлета [Add-AzKeyVaultCertificate](/powershell/module/az.keyvault/add-azkeyvaultcertificate) создать самозаверяющий сертификат, использующий политику сертификата по умолчанию из [New-AzreKeyVaultCertificatePolicy](/powershell/module/az.keyvault/new-azkeyvaultcertificatepolicy). 

```azurepowershell-interactive
$policy = New-AzKeyVaultCertificatePolicy `
    -SubjectName "CN=www.contoso.com" `
    -SecretContentType "application/x-pkcs12" `
    -IssuerName Self `
    -ValidityInMonths 12

Add-AzKeyVaultCertificate `
    -VaultName $keyvaultName `
    -Name "mycert" `
    -CertificatePolicy $policy 
```


## <a name="create-a-virtual-machine"></a>Создание виртуальной машины
Укажите имя и пароль администратора для виртуальной машины с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Теперь вы можете создать виртуальную машину с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm). В следующем примере создается виртуальная машина с именем *myVM* в расположении *EastUS*. Если они еще не существуют, создаются поддерживающие сетевые ресурсы. Чтобы разрешить безопасный веб-трафик, командлет также открывает порт *443*.

```azurepowershell-interactive
# Create a VM
New-AzVm `
    -ResourceGroupName $resourceGroup `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred `
    -OpenPorts 443

# Use the Custom Script Extension to install IIS
Set-AzVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server -IncludeManagementTools"}'
```

Создание виртуальной машины может занять несколько минут. На последнем шаге с помощью расширения пользовательских сценариев Azure устанавливается веб-сервер IIS. Для этого используется командлет [Set-AzVmExtension](/powershell/module/az.compute/set-azvmextension).


## <a name="add-a-certificate-to-vm-from-key-vault"></a>Добавление сертификата на виртуальную машину из Key Vault
Чтобы добавить сертификат из Key Vault на виртуальную машину, получите идентификатор сертификата с помощью командлета [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret). Добавьте сертификат на виртуальную машину с помощью командлета [Add-AzVMSecret](/powershell/module/az.compute/add-azvmsecret):

```azurepowershell-interactive
$certURL=(Get-AzKeyVaultSecret -VaultName $keyvaultName -Name "mycert").id

$vm=Get-AzVM -ResourceGroupName $resourceGroup -Name "myVM"
$vaultId=(Get-AzKeyVault -ResourceGroupName $resourceGroup -VaultName $keyVaultName).ResourceId
$vm = Add-AzVMSecret -VM $vm -SourceVaultId $vaultId -CertificateStore "My" -CertificateUrl $certURL

Update-AzVM -ResourceGroupName $resourceGroup -VM $vm
```


## <a name="configure-iis-to-use-the-certificate"></a>Настройка IIS для использования сертификата
Обновите конфигурацию IIS с помощью расширения пользовательских сценариев и командлета [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension). Это обновление применяет внедренный из Key Vault в IIS сертификат и настраивает веб-привязку:

```azurepowershell-interactive
$PublicSettings = '{
    "fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/secure-iis.ps1"],
    "commandToExecute":"powershell -ExecutionPolicy Unrestricted -File secure-iis.ps1"
}'

Set-AzVMExtension -ResourceGroupName $resourceGroup `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location $location `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -SettingString $publicSettings
```


### <a name="test-the-secure-web-app"></a>Тестирование защищенного веб-приложения
Получите общедоступный IP-адрес своей виртуальной машины с помощью командлета [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). Следующий пример позволяет получить IP-адрес для созданного ранее `myPublicIP`.

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName $resourceGroup -Name "myPublicIPAddress" | select "IpAddress"
```

Теперь можно открыть веб-браузер и ввести в адресной строке `https://<myPublicIP>`. Чтобы принять предупреждение системы безопасности, если используется самозаверяющий сертификат безопасности, выберите **Сведения**, а затем — **Перейти на веб-страницу**:

![Принятие предупреждения о безопасности веб-браузера](./media/tutorial-secure-web-server/browser-warning.png)

На экране отобразится защищенный веб-сайт IIS, как в показано следующем примере:

![Просмотр работающего защищенного сайта IIS](./media/tutorial-secure-web-server/secured-iis.png)


## <a name="next-steps"></a>Дальнейшие действия
Выполнив инструкции, приведенные в этом руководстве, вы защитили веб-сервер IIS TLS/SSL-сертификатом, который хранится в Azure Key Vault. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * создать Azure Key Vault;
> * создать или передать сертификат в Key Vault;
> * Создание виртуальной машины и установка веб-сервера IIS
> * внедрение сертификата в виртуальную машину и настройка IIS с помощью TLS-привязки.

Чтобы увидеть предварительно созданные примеры скриптов виртуальной машины, перейдите по ссылке ниже.

> [!div class="nextstepaction"]
> [Примеры скриптов для виртуальной машины Windows](https://github.com/Azure/azure-docs-powershell-samples/tree/master/virtual-machine)