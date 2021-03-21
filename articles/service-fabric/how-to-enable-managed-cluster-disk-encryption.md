---
title: Включение шифрования дисков для узлов управляемого кластера (Предварительная версия) Service Fabric
description: Узнайте, как включить шифрование дисков для управляемых узлов кластера Azure Service Fabric в Windows с помощью шаблона ARM.
ms.topic: how-to
ms.date: 02/15/2021
ms.openlocfilehash: b7e56ff8db9f94b8c6681a1a7d69a4751b3f43a5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100642508"
---
# <a name="enable-disk-encryption-for-service-fabric-managed-cluster-preview-nodes"></a>Включение шифрования дисков для узлов управляемого кластера (Предварительная версия) Service Fabric

В этом руководство вы узнаете, как включить шифрование дисков для Service Fabric управляемых узлов кластера в Windows с помощью функции [шифрования дисков Azure](../virtual-machines/windows/disk-encryption-overview.md) для [масштабируемых наборов виртуальных машин](../virtual-machine-scale-sets/disk-encryption-azure-resource-manager.md) с помощью шаблонов Azure Resource Manager (ARM).

> [!IMPORTANT]
> Предварительная версия шифрования диска масштабируемого набора виртуальных машин еще не поддерживает обновление образа или повторное создание образа. Не используйте, если вам потребуется обновить образ операционной системы.

## <a name="register-for-azure-disk-encryption"></a>Регистрация для шифрования дисков Azure

Для предварительного просмотра шифрования дисков в масштабируемом наборе виртуальных машин требуется самостоятельная регистрация. Выполните следующую команду.

```powershell
Register-AzProviderFeature -FeatureName "UnifiedDiskEncryption" -ProviderNamespace "Microsoft.Compute"
```

Проверьте состояние регистрации, выполнив команду.

```powershell
Get-AzProviderFeature -ProviderNamespace "Microsoft.Compute" -FeatureName "UnifiedDiskEncryption"
```

Когда состояние изменится на *зарегистрировано*, вы можете продолжать работу.

## <a name="provision-a-key-vault-for-disk-encryption"></a>Подготавливает Key Vault к шифрованию диска

Службе шифрования дисков Azure необходимо Azure Key Vault, чтобы управлять секретами и ключами шифрования дисков и контролировать их. Управляемый кластер Key Vault и Service Fabric должен находиться в одном регионе Azure и подписке. При условии соблюдения этих требований можно использовать новую или существующую Key Vault, включив ее для шифрования дисков.

### <a name="create-key-vault-with-disk-encryption-enabled"></a>Создание Key Vault с включенным шифрованием дисков

Выполните следующие команды, чтобы создать новый Key Vault для шифрования дисков. Убедитесь, что регион для Key Vault [поддерживается для Service Fabric управляемых кластеров](faq-managed-cluster.md#what-regions-are-supported-in-the-preview) и находится в том же регионе, что и кластер.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$resourceGroupName = "<rg-name>" 
$keyvaultName = "<kv-name>" 

New-AzResourceGroup -Name $resourceGroupName -Location eastus2 
New-AzKeyVault -ResourceGroupName $resourceGroupName -Name $keyvaultName -Location eastus2 -EnabledForDiskEncryption
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
$resourceGroupName = "<rg-name>" 
$keyvaultName = "<kv-name>" 

az keyvault create --resource-group $resourceGroupName --name $keyvaultName --enabled-for-disk-encryption
```

---

### <a name="update-existing-key-vault-to-enable-disk-encryption"></a>Обновление существующих Key Vault для включения шифрования дисков

Выполните следующие команды, чтобы включить шифрование дисков для существующей Key Vault.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
# ps 

Set-AzKeyVaultAccessPolicy -ResourceGroupName $resourceGroupName -VaultName $keyvaultName -EnabledForDiskEncryption
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az keyvault update --name keyvaultName --enabled-for-disk-encryption 
```

---

## <a name="update-the-template-and-parameters-files-for-disk-encryption"></a>Обновление файлов шаблонов и параметров для шифрования дисков

На следующем шаге будут рассмотрены необходимые изменения шаблона, чтобы включить шифрование дисков в [существующем управляемом кластере](tutorial-managed-cluster-deploy.md). Кроме того, можно развернуть новый управляемый кластер Service Fabric с включенным шифрованием дисков с помощью этого шаблона: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Standard-SKU-1-NT-DiskEncryption

1. Добавьте в шаблон следующие параметры, подставив собственную подписку, имя группы ресурсов и имя хранилища в разделе `keyVaultResourceId` :

```json
"parameters": { 
…
    "keyVaultResourceId": { 
        "type": "string", 
        "defaultValue": "/subscriptions/########-####-####-####-############/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<kv-name>", 
        "metadata": { 
            "description": "Full resource id of the Key Vault used for disk encryption." 
        } 
    },
    "volumeType": { 
        "type": "string", 
        "defaultValue": "All", 
        "metadata": { 
            "description": "Type of the volume OS or Data to perform encryption operation" 
        }
    }
}, 
```

2. Затем добавьте `AzureDiskEncryption` расширение виртуальной машины в типы управляемых узлов кластера в шаблоне:

```json
"properties": { 
    "vmExtensions": [ 
        { 
            "name": "AzureDiskEncryption", 
            "properties": { 
                "publisher": "Microsoft.Azure.Security", 
                "type": "AzureDiskEncryption", 
                "typeHandlerVersion": "2.1", 
                "autoUpgradeMinorVersion": true, 
                "settings": {                     
                    "EncryptionOperation": "EnableEncryption", 
                    "KeyVaultURL": "[reference(parameters('keyVaultResourceId'),'2016-10-01').vaultUri]", 
                    "KeyVaultResourceId": "[parameters('keyVaultResourceID')]",
                    "VolumeType": "[parameters('volumeType')]" 
                } 
            } 
        } 
    ] 
} 
```

3. Наконец, обновите файл параметров, заменив собственную подписку, группу ресурсов и имя хранилища ключей в *кэйваултресаурцеид*:

```json
"parameters": { 
    … 
    "keyVaultResourceId": { 
        "value": "/subscriptions/########-####-####-####-############/resourceGroups/<rg-name>/providers/Microsoft.KeyVault/vaults/<kv-name>" 
    },   
    "volumeType": { 
        "value": "All" 
    }    
} 
```

## <a name="deploy-and-verify-the-changes"></a>Развертывание и проверка изменений

Когда вы будете готовы, разверните изменения, чтобы включить шифрование дисков в управляемом кластере.

```powershell
$clusterName = "<clustername>" 

New-AzResourceGroupDeployment -Name $resourceGroupName -ResourceGroupName $resourceGroupName -TemplateFile .\template_diskEncryption.json -TemplateParameterFile \.parameters_diskEncryption.json -Debug -Verbose 
```

Состояние шифрования диска можно проверить в базовом масштабируемом наборе узлов с помощью `Get-AzVmssDiskEncryption` команды. Сначала необходимо найти имя поддерживающей группы ресурсов управляемого кластера (которая содержит базовую виртуальную сеть, подсистему балансировки нагрузки, общедоступный IP-адрес, NSG, масштабируемые наборы и учетные записи хранения). Не забудьте изменить `VmssName` имя типа узла кластера, который вы хотите проверить (как указано в шаблоне развертывания).

```powershell
$VmssName = "NT1"
$supportResourceGroupName = "SFC_" + (Get-AzServiceFabricManagedCluster -ResourceGroupName $resourceGroupName).ClusterId
Get-AzVmssDiskEncryption -ResourceGroupName $supportResourceGroupName -VMScaleSetName $VmssName
```

Результат должен выглядеть следующим образом:

```console
ResourceGroupName            : SFC_########-####-####-####-############
VmScaleSetName               : NT1
EncryptionEnabled            : True
EncryptionExtensionInstalled : True
```

## <a name="next-steps"></a>Дальнейшие действия

[Пример: Стандартный номер SKU Service Fabric управляемый кластер, тип 1 узла с включенным шифрованием дисков](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Standard-SKU-1-NT-DiskEncryption)

[Шифрование дисков Azure для виртуальных машин Windows](../virtual-machines/windows/disk-encryption-overview.md)

[Шифрование масштабируемых наборов виртуальных машин с помощью Azure Resource Manager](../virtual-machine-scale-sets/disk-encryption-azure-resource-manager.md)
