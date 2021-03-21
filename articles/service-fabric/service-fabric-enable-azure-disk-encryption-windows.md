---
title: Включение шифрования дисков для кластеров Windows
description: В этой статье описывается, как включить шифрование дисков для узлов кластера Azure Service Fabric с помощью Azure Key Vault в Azure Resource Manager.
ms.topic: article
ms.date: 03/22/2019
ms.openlocfilehash: b08cdb63aa6f334c5a6f7c230b1624d232206c3b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78251817"
---
# <a name="enable-disk-encryption-for-azure-service-fabric-cluster-nodes-in-windows"></a>Включение шифрования дисков для узлов кластера Azure Service Fabric в Windows 
> [!div class="op_single_selector"]
> * [Шифрование дисков для Windows](service-fabric-enable-azure-disk-encryption-windows.md)
> * [Шифрование дисков для Linux](service-fabric-enable-azure-disk-encryption-linux.md)
>
>

В этом руководстве вы узнаете, как включить шифрование дисков на узлах кластера Service Fabric в Windows. Для каждого из типов узлов и масштабируемых наборов виртуальных машин необходимо выполнить следующие действия. Для шифрования узлов мы будем использовать функцию шифрования дисков Azure в масштабируемых наборах виртуальных машин.

В руководстве рассматриваются следующие темы:

* Основные понятия, которые следует учитывать при включении шифрования дисков в масштабируемых наборах виртуальных машин кластера в Windows Service Fabric.
* Действия, которые следует выполнить перед включением шифрования дисков на узлах кластера Service Fabric в Windows.
* Действия, которые следует выполнить, чтобы включить шифрование дисков на узлах кластера Service Fabric в Windows.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

**Самостоятельная регистрация** 

Для предварительного просмотра шифрования дисков в масштабируемом наборе виртуальных машин требуется самостоятельная регистрация. Выполните указанные ниже действия. 

1. Сначала выполните следующую команду:
    ```powershell
    Register-AzProviderFeature -ProviderNamespace Microsoft.Compute -FeatureName "UnifiedDiskEncryption"
    ```
2. Подождите около 10 минут, пока не будет *зарегистрировано* состояние READS. Состояние можно проверить, выполнив следующую команду: 
    ```powershell
    Get-AzProviderFeature -ProviderNamespace "Microsoft.Compute" -FeatureName "UnifiedDiskEncryption"
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    ```
**Azure Key Vault**; 

1. Создайте хранилище ключей в той же подписке и регионе, что и масштабируемый набор, а затем выберите политику доступа **EnabledForDiskEncryption** в хранилище ключей, используя командлет PowerShell. Политику также можно задать с помощью пользовательского интерфейса Key Vault в портал Azure с помощью следующей команды:
    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -EnabledForDiskEncryption
    ```
2. Установите последнюю версию [Azure CLI](/cli/azure/install-azure-cli), которая содержит новые команды шифрования.
3. Установите последнюю версию [пакета SDK для Azure из Azure PowerShell](https://github.com/Azure/azure-powershell/releases) выпуска. Ниже приведены командлеты для шифрования дисков Azure масштабируемого набора виртуальных машин, позволяющие включить ([настроить](/powershell/module/az.compute/set-azvmssdiskencryptionextension)) шифрование, получить ([получить](/powershell/module/az.compute/get-azvmssvmdiskencryption)) состояние шифрования и удалить ([Отключить](/powershell/module/az.compute/disable-azvmssdiskencryption)) шифрование в экземпляре масштабируемого набора.

| Get-Help | Версия |  Источник  |
| ------------- |-------------| ------------|
| Get-AzVmssDiskEncryptionStatus   | 1.0.0 или более поздняя версия | Az.Compute |
| Get-AzVmssVMDiskEncryptionStatus   | 1.0.0 или более поздняя версия | Az.Compute |
| Disable-AzVmssDiskEncryption   | 1.0.0 или более поздняя версия | Az.Compute |
| Get-AzVmssDiskEncryption   | 1.0.0 или более поздняя версия | Az.Compute |
| Get-AzVmssVMDiskEncryption   | 1.0.0 или более поздняя версия | Az.Compute |
| Set-AzVmssDiskEncryptionExtension   | 1.0.0 или более поздняя версия | Az.Compute |


## <a name="supported-scenarios-for-disk-encryption"></a>Поддерживаемые сценарии для шифрования дисков
* Шифрование масштабируемых наборов виртуальных машин поддерживается только для масштабируемых наборов, созданных с помощью управляемых дисков. Оно не поддерживается для масштабируемых наборов на основе собственных (или неуправляемых) дисков.
* Шифрование поддерживается для томов ОС и данных в масштабируемых наборах виртуальных машин в Windows. Отключение шифрования также поддерживается для томов ОС и данных для масштабируемых наборов виртуальных машин в Windows.
* Операции пересоздания образа и обновления виртуальных машин для масштабируемых наборов виртуальных машин не поддерживаются в текущей предварительной версии.


## <a name="create-a-new-cluster-and-enable-disk-encryption"></a>Создание нового кластера и включение шифрования дисков

Используйте следующие команды для создания кластера и включения шифрования дисков с использованием шаблона Azure Resource Manager и самозаверяющего сертификата.

### <a name="sign-in-to-azure"></a>Вход в Azure 
Выполните вход с помощью следующих команд:
```powershell
Login-AzAccount
Set-AzContext -SubscriptionId <guid>

```

```azurecli

azure login
az account set --subscription $subscriptionId

```

### <a name="use-the-custom-template-that-you-already-have"></a>Использование собственного пользовательского шаблона 

Если вам нужно создать настраиваемый шаблон в соответствии с вашими потребностями, мы настоятельно рекомендуем начать с одного из шаблонов, доступных на странице [примеров шаблона создания кластера Azure Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master) . Сведения о [настройке раздела шаблона кластера][customize-your-cluster-template] см. в следующих руководствах.

Если у вас уже есть пользовательский шаблон, убедитесь, что все три параметра, связанные с сертификатами в шаблоне, и файл параметров имеют следующие имена, и эти значения равны NULL следующим образом:

```Json
   "certificateThumbprint": {
      "value": ""
    },
    "sourceVaultValue": {
      "value": ""
    },
    "certificateUrlValue": {
      "value": ""
    },
```


```powershell
$resourceGroupLocation="westus"
$resourceGroupName="mycluster"
$CertSubjectName="mycluster.westus.cloudapp.azure.com"
$certPassword="Password!1" | ConvertTo-SecureString -AsPlainText -Force 
$certOutputFolder="c:\certificates"

$parameterFilePath="c:\templates\templateparam.json"
$templateFilePath="c:\templates\template.json"


New-AzServiceFabricCluster -ResourceGroupName $resourceGroupName -CertificateOutputFolder $certOutputFolder -CertificatePassword $certpassword -CertificateSubjectName $CertSubjectName -TemplateFile $templateFilePath -ParameterFile $parameterFilePath 

```


```azurecli

declare certPassword=""
declare resourceGroupLocation="westus"
declare resourceGroupName="mylinux"
declare certSubjectName="mylinuxsecure.westus.cloudapp.azure.com"
declare parameterFilePath="c:\mytemplates\linuxtemplateparm.json"
declare templateFilePath="c:\mytemplates\linuxtemplate.json"
declare certOutputFolder="c:\certificates"


az sf cluster create --resource-group $resourceGroupName --location $resourceGroupLocation  \
    --certificate-output-folder $certOutputFolder --certificate-password $certPassword  \
    --certificate-subject-name $certSubjectName \
    --template-file $templateFilePath --parameter-file $parametersFilePath

```

### <a name="deploy-an-application-to-a-service-fabric-cluster-in-windows"></a>Развертывание приложения в кластере Service Fabric в Windows
Чтобы развернуть приложение в кластере, следуйте инструкциям в статье [развертывание и удаление приложений с помощью PowerShell](service-fabric-deploy-remove-applications.md).


### <a name="enable-disk-encryption-for-the-virtual-machine-scale-sets-created-previously"></a>Включение шифрования дисков для масштабируемых наборов виртуальных машин, созданных ранее

Чтобы включить шифрование дисков для масштабируемых наборов виртуальных машин, созданных с помощью предыдущих шагов, выполните следующие команды:
 
```powershell

$VmssName = "nt1vm"
$vaultName = "mykeyvault"
$resourceGroupName = "mycluster"
$KeyVault = Get-AzKeyVault -VaultName $vaultName -ResourceGroupName $rgName
$DiskEncryptionKeyVaultUrl = $KeyVault.VaultUri
$KeyVaultResourceId = $KeyVault.ResourceId

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName -DiskEncryptionKeyVaultUrl $DiskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All

```

```azurecli

az vmss encryption enable -g <resourceGroupName> -n <VMSS name> --disk-encryption-keyvault <KeyVaultResourceId>

```


### <a name="validate-if-disk-encryption-is-enabled-for-a-virtual-machine-scale-set-in-windows"></a>Проверка включения шифрования дисков для масштабируемого набора виртуальных машин в Windows
Получите состояние всего масштабируемого набора виртуальных машин или любого экземпляра в масштабируемом наборе, выполнив следующие команды.

```powershell

$VmssName = "nt1vm"
$resourceGroupName = "mycluster"
Get-AzVmssDiskEncryption -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName

Get-AzVmssVMDiskEncryption -ResourceGroupName $resourceGroupName -VMScaleSetName $VmssName -InstanceId "0"

```

```azurecli

az vmss encryption show -g <resourceGroupName> -n <VMSS name>

```


Кроме того, можно войти в масштабируемый набор виртуальных машин и убедиться, что диски зашифрованы.

### <a name="disable-disk-encryption-for-a-virtual-machine-scale-set-in-a-service-fabric-cluster"></a>Отключение шифрования дисков для масштабируемого набора виртуальных машин в кластере Service Fabric 
Отключите шифрование дисков для масштабируемого набора виртуальных машин, выполнив следующие команды. Обратите внимание, что отключение шифрования дисков применяется ко всему масштабируемому набору виртуальных машин, а не к отдельному экземпляру.

```powershell

$VmssName = "nt1vm"
$resourceGroupName = "mycluster"
Disable-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $VmssName

```

```azurecli

az vmss encryption disable -g <resourceGroupName> -n <VMSS name>

```


## <a name="next-steps"></a>Дальнейшие действия
На этом этапе у вас должен быть защищенный кластер, и вы узнаете, как включить и отключить шифрование дисков для Service Fabric узлов кластера и масштабируемых наборов виртуальных машин. Аналогичные рекомендации по Service Fabric узлов кластера в Linux см. в статье [Шифрование дисков для Linux](service-fabric-enable-azure-disk-encryption-linux.md).

[customize-your-cluster-template]: https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure#creating-a-custom-arm-template
