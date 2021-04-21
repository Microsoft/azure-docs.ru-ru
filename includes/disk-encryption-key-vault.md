---
title: включить файл
description: включить файл
services: virtual-machines
author: msmbaldwin
ms.service: virtual-machines
ms.topic: include
ms.date: 10/06/2019
ms.author: mbaldwin
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: 98922829e83f84078c3d8cadae15844dba194c93
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800175"
---
## <a name="create-a-resource-group"></a>Создание группы ресурсов

*Если у вас уже есть группа ресурсов, можно перейти к [созданию хранилища ключей](#create-a-key-vault).*

Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. 

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create) в Azure CLI, команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) в Azure PowerShell или на [портале Azure](https://portal.azure.com).

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az group create --name "myResourceGroup" --location eastus
```
### <a name="azure-powershell"></a>Azure PowerShell
```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-a-key-vault"></a>Создайте хранилище ключей.

*Если у вас уже есть хранилище ключей, можно перейти к [заданию политик расширенного доступа к хранилищу ключей](#set-key-vault-advanced-access-policies).*

Создайте хранилище ключей с помощью команды [az keyvault create](/cli/azure/keyvault#az_keyvault_create) в Azure CLI, команды [New-AzKeyvault](/powershell/module/az.keyvault/new-azkeyvault) в Azure PowerShell, на [портале Azure](https://portal.azure.com) или с помощью [шаблона Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create).

>[!WARNING]
> Хранилище ключей и виртуальные машины должны находиться в одной подписке. Кроме того, чтобы убедиться, что секреты шифрования не пересекают региональные границы, службе "Шифрование дисков Azure" требуется, чтобы хранилище ключей и виртуальные машины были расположены в одном регионе. Создайте и используйте хранилище ключей в тех же подписке и регионе, в которых находится виртуальная машина, которую нужно зашифровать. 

Каждый Key Vault должен иметь уникальное имя. В следующих примерах замените <your-unique-keyvault-name> именем своего хранилища ключей.

### <a name="azure-cli"></a>Azure CLI

При создании хранилища ключей с помощью Azure CLI добавьте флаг --enabled-for-disk-encryption.

```azurecli-interactive
az keyvault create --name "<your-unique-keyvault-name>" --resource-group "myResourceGroup" --location "eastus" --enabled-for-disk-encryption
```

### <a name="azure-powershell"></a>Azure PowerShell

При создании хранилища ключей с помощью Azure PowerShell добавьте флаг -EnabledForDiskEncryption.

```azurepowershell-interactive
New-AzKeyvault -name "<your-unique-keyvault-name>" -ResourceGroupName "myResourceGroup" -Location "eastus" -EnabledForDiskEncryption
```
### <a name="resource-manager-template"></a>Шаблон Resource Manager

Вы также можете создать хранилище ключей с помощью [шаблона Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create).

1. На странице шаблона быстрого запуска Azure нажмите кнопку **Deploy to Azure** (Развернуть в Azure).
2. Выберите подписку, группу ресурсов, расположение группы ресурсов, имя хранилища ключей, идентификатор объекта, юридические условия и соглашение, а затем щелкните **Приобрести**. 


##  <a name="set-key-vault-advanced-access-policies"></a>установить политики расширенного доступа к хранилищу ключей.

Платформа Azure должна иметь доступ к ключам шифрования или секретам, расположенным в вашем хранилище ключей, чтобы предоставлять их виртуальной машине при ее загрузке для расшифровки томов. 

Если вы не включили хранилище ключей для шифрования дисков, развертывания или развертывания шаблона во время создания (как показано на предыдущем шаге), необходимо обновить соответствующие политики расширенного доступа.  

### <a name="azure-cli"></a>Azure CLI

Включите шифрование дисков для хранилища ключей с помощью команды [az keyvault update](/cli/azure/keyvault#az_keyvault_update). 

 - **Включение хранилища ключей для шифрования диска:** требуется использовать параметр enabled-for-disk-encryption. 

     ```azurecli-interactive
     az keyvault update --name "<your-unique-keyvault-name>" --resource-group "MyResourceGroup" --enabled-for-disk-encryption "true"
     ```  

 - **Включение хранилища ключей для развертывания (если необходимо):** позволяет поставщику ресурсов Microsoft.Compute извлекать секреты из этого хранилища ключей в случае использования ссылки на него при создании ресурса, например, виртуальной машины.

     ```azurecli-interactive
     az keyvault update --name "<your-unique-keyvault-name>" --resource-group "MyResourceGroup" --enabled-for-deployment "true"
     ``` 

 - **Включение хранилища ключей для развертывания шаблона (если необходимо):** позволяет Resource Manager получать секреты из хранилища.
     ```azurecli-interactive  
     az keyvault update --name "<your-unique-keyvault-name>" --resource-group "MyResourceGroup" --enabled-for-template-deployment "true"
     ```

###  <a name="azure-powershell"></a>Azure PowerShell
 Используйте командлет [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) в PowerShell, чтобы включить шифрование диска для хранилища ключей.

  - **Включение хранилища ключей для шифрования диска:** для шифрования дисков Azure требуется использовать параметр EnabledForDiskEncryption.
      
     ```azurepowershell-interactive 
     Set-AzKeyVaultAccessPolicy -VaultName "<your-unique-keyvault-name>" -ResourceGroupName "MyResourceGroup" -EnabledForDiskEncryption
     ```

  - **Включение хранилища ключей для развертывания (если необходимо):** позволяет поставщику ресурсов Microsoft.Compute извлекать секреты из этого хранилища ключей в случае использования ссылки на него при создании ресурса, например, виртуальной машины.

     ```azurepowershell-interactive
      Set-AzKeyVaultAccessPolicy -VaultName "<your-unique-keyvault-name>" -ResourceGroupName "MyResourceGroup" -EnabledForDeployment
     ```

  - **Включение хранилища ключей для развертывания шаблона (если необходимо):** позволяет Azure Resource Manager получать секреты из этого хранилища ключей в случае использования ссылки на него при развертывании шаблона.

     ```azurepowershell-interactive             
     Set-AzKeyVaultAccessPolicy -VaultName "<your-unique-keyvault-name>" -ResourceGroupName "MyResourceGroup" -EnabledForTemplateDeployment
     ```

### <a name="azure-portal"></a>Портал Azure

1. Выберите хранилище ключей, перейдите на вкладку **Политики доступа** и выберите **Щелкните, чтобы показать политики расширенного доступа**.
2. Установите флажок **Включить доступ к шифрованию дисков Azure для шифрования томов**.
3. Выберите **Включить доступ к виртуальным машинам Azure для развертывания** или **Включить доступ к Azure Resource Manager для развертывания шаблонов**, если это необходимо. 
4. Нажмите кнопку **Сохранить**.

    ![Установка политики расширенного доступа к хранилищу Azure Key Vault](../articles/virtual-machines/media/disk-encryption/keyvault-portal-fig4.png)


## <a name="set-up-a-key-encryption-key-kek"></a>Настройка ключа шифрования ключа (KEK)

Если вы хотите использовать ключ шифрования ключей (KEK), чтобы добавить уровень безопасности ключей шифрования, поместите KEK в хранилище ключей. Когда ключ шифрования ключей указан, шифрование дисков Azure использует его для упаковки секретов шифрования перед записью в Key Vault.

Вы можете создать новый KEK с помощью команды [az keyvault key create](/cli/azure/keyvault/key#az_keyvault_key_create) в Azure CLI, команды [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) в Azure PowerShell или на [портале Azure](https://portal.azure.com/). Вам нужно создать тип ключа RSA. Служба "Шифрование дисков Azure" пока не поддерживает использование ключей на основе эллиптической кривой.

Вместо этого KEK можно импортировать из своего локального устройства HSM управления ключами. Дополнительные сведения см. в документации по [Key Vault](../articles/key-vault/keys/hsm-protected-keys.md).

Для URL-адресов KEK хранилища ключей необходимо включить управление версиями. Это требование Azure. Ниже приведены примеры действительных URL-адресов секрета и ключа шифрования ключей.

* Пример допустимого URL-адреса секрета: *https://contosovault.vault.azure.net/secrets/EncryptionSecretWithKek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx* .
* Пример допустимого URL-адреса KEK: *https://contosovault.vault.azure.net/keys/diskencryptionkek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx* .

Шифрование дисков Azure не поддерживает указание номеров портов в URL-адресах секрета и ключа шифрования ключей для хранилища ключей. Ниже приведены примеры недопустимых и допустимых URL-адресов хранилища ключей.

  * Допустимый URL-адрес хранилища ключей: *https://contosovault.vault.azure.net/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx* .
  * Недопустимый URL-адрес хранилища ключей: *https://contosovault.vault.azure.net:443/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx* .

### <a name="azure-cli"></a>Azure CLI

Используйте команду [az keyvault key create](/cli/azure/keyvault/key#az_keyvault_key_create) в Azure CLI, чтобы создать новый KEK и сохранить его в хранилище ключей.

```azurecli-interactive
az keyvault key create --name "myKEK" --vault-name "<your-unique-keyvault-name>" --kty RSA
```

Вместо этого можно импортировать закрытый ключ с помощью команды [az keyvault key import](/cli/azure/keyvault/key#az_keyvault_key_import) в Azure CLI.

В любом случае вы укажете имя KEK в параметре --key-encryption-key команды [az vm encryption enable](/cli/azure/vm/encryption#az_vm_encryption_enable) в Azure CLI. 

```azurecli-interactive
az vm encryption enable -g "MyResourceGroup" --name "myVM" --disk-encryption-keyvault "<your-unique-keyvault-name>" --key-encryption-key "myKEK"
```

###  <a name="azure-powershell"></a>Azure PowerShell 

Используйте командлет [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) в Azure PowerShell, чтобы создать новый KEK и сохранить его в хранилище ключей.

 ```powershell-interactive
Add-AzKeyVaultKey -Name "myKEK" -VaultName "<your-unique-keyvault-name>" -Destination "HSM"
```

Вместо этого можно импортировать закрытый ключ с помощью команды [az keyvault key import](/cli/azure/keyvault/key#az_keyvault_key_import) в Azure PowerShell.

В любом случае вы укажете идентификатор хранилища ключей KEK и URL-адрес KEK в параметрах -KeyEncryptionKeyVaultId и -KeyEncryptionKeyUrl команды [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) в Azure PowerShell. Обратите внимание, что в этом примере предполагается, что вы используете одно хранилище ключей как для ключа шифрования диска, так и для KEK.

 ```powershell-interactive
$KeyVault = Get-AzKeyVault -VaultName "<your-unique-keyvault-name>" -ResourceGroupName "myResourceGroup"
$KEK = Get-AzKeyVaultKey -VaultName "<your-unique-keyvault-name>" -Name "myKEK"

Set-AzVMDiskEncryptionExtension -ResourceGroupName MyResourceGroup -VMName "MyVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -KeyEncryptionKeyVaultId $KeyVault.ResourceId -KeyEncryptionKeyUrl $KEK.Id -SkipVmBackup -VolumeType All
 ```