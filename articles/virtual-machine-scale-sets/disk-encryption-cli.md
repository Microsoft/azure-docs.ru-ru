---
title: Шифрование дисков для масштабируемых наборов Azure с помощью Azure CLI
description: Узнайте, как использовать Azure CLI для шифрования экземпляров виртуальной машины и подключенных к ним дисков в масштабируемом наборе виртуальных машин Windows.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/15/2019
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: e6630cbb44157f25bd2cbfcff25ec3132c74c61c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105565577"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-the-azure-cli"></a>Шифрование диска ОС и подключенных дисков данных в масштабируемом наборе виртуальных машин с помощью Azure CLI

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. В этом кратком руководстве показано, как с помощью Azure CLI создать и зашифровать масштабируемый набор виртуальных машин. Дополнительные сведения см. в статье [Шифрование дисков Azure для масштабируемых наборов виртуальных машин](disk-encryption-overview.md).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.31 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-scale-set"></a>Создание масштабируемого набора

Прежде чем создать масштабируемый набор, выполните команду [az group create](/cli/azure/group) для создания группы ресурсов. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Создайте масштабируемый набор виртуальных машин с помощью команды [az vmss create](/cli/azure/vmss). В следующем примере создается масштабируемый набор с именем *myScaleSet*, который настроен на автоматическое обновление при применении изменений. Также создаются ключи SSH, если они не существуют в *~/.ssh/id_rsa*. К каждому экземпляру виртуальной машины подключается диск данных объемом 32 ГБ. Диски данных подготавливаются с помощью [расширения пользовательского скрипта](../virtual-machines/extensions/custom-script-linux.md) Azure. Для этого выполняется команда [az vmss extension set](/cli/azure/vmss/extension):

```azurecli-interactive
# Create a scale set with attached data disk
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 32

# Prepare the data disk for use with the Custom Script Extension
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.sh"],"commandToExecute":"./prepare_vm_disks.sh"}'
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>Создание хранилища ключей Azure с поддержкой шифрования дисков

В хранилище ключей Azure можно хранить ключи, секреты или пароли, что позволяет безопасно реализовать их в приложениях и службах. Криптографические ключи хранятся в хранилище ключей Azure с применением защиты программного обеспечения. В качестве альтернативы можно импортировать или создать ключи аппаратных модулей безопасности (HSM), сертифицированных по стандартам уровня 2 FIPS 140-2. Эти криптографические ключи используются для шифрования и расшифровки виртуальных дисков, подключенных к виртуальной машине. Вы сохраняете контроль над этими криптографическими ключами и можете проводить аудит их использования.

Определите уникальное значение *keyvault_name*. Затем создайте хранилище ключей с помощью команды [az keyvault create](/cli/azure/keyvault#ext-keyvault-preview-az-keyvault-create) в тех же подписке и регионе, что и масштабируемый набор, и настройте политику доступа *--enabled-for-disk-encryption*.

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault create --resource-group myResourceGroup --name $keyvault_name --enabled-for-disk-encryption
```

### <a name="use-an-existing-key-vault"></a>Использование существующего Key Vault

Этот шаг выполняется только в том случае, если у вас уже есть хранилище ключей и вы хотите применить его для шифрования дисков. Пропустите этот шаг, если в предыдущем разделе вы создали новый Key Vault.

Определите уникальное значение *keyvault_name*. Затем обновите хранилище ключей с помощью команды [az keyvault update](/cli/azure/keyvault#ext-keyvault-preview-az-keyvault-update) и настройте политику доступа *--enabled-for-disk-encryption*.

```azurecli-interactive
# Provide your own unique Key Vault name
keyvault_name=myuniquekeyvaultname

# Create Key Vault
az keyvault update --name $keyvault_name --enabled-for-disk-encryption
```

## <a name="enable-encryption"></a>Включение шифрования

Чтобы зашифровать экземпляры виртуальных машин в масштабируемом наборе, сначала нужно получить некоторые сведения об идентификаторе ресурса для Key Vault с помощью команды [az keyvault show](/cli/azure/keyvault#ext-keyvault-preview-az-keyvault-show). Эти переменные применяются при запуске процесса шифрования с помощью команды [az vmss encryption enable](/cli/azure/vmss/encryption#az-vmss-encryption-enable):

```azurecli-interactive
# Get the resource ID of the Key Vault
vaultResourceId=$(az keyvault show --resource-group myResourceGroup --name $keyvault_name --query id -o tsv)

# Enable encryption of the data disks in a scale set
az vmss encryption enable \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --disk-encryption-keyvault $vaultResourceId \
    --volume-type DATA
```

На запуск процесса шифрования могут потребоваться одна-две минуты.

Так как для набора масштабирования, созданного ранее, задана политика *автоматического* обновления, процесс шифрования для экземпляров виртуальных машин запускается автоматически. Если для масштабируемого набора задана политика обновления вручную, запустите шифрование для экземпляров виртуальных машин с помощью команды [az vmss update-instances](/cli/azure/vmss#az-vmss-update-instances).

### <a name="enable-encryption-using-kek-to-wrap-the-key"></a>Включение шифрования с использованием ключа шифрования ключа для переноса ключа

Ключ шифрования ключа (KEK) можно также использовать для повышения безопасности при шифровании масштабируемого набора виртуальных машин.

```azurecli-interactive
# Get the resource ID of the Key Vault
vaultResourceId=$(az keyvault show --resource-group myResourceGroup --name $keyvault_name --query id -o tsv)

# Enable encryption of the data disks in a scale set
az vmss encryption enable \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --disk-encryption-keyvault $vaultResourceId \
    --key-encryption-key myKEK \
    --key-encryption-keyvault $vaultResourceId \
    --volume-type DATA
```

> [!NOTE]
>  Синтаксис значения параметра disk-encryption-keyvault — это строка полного идентификатора:</br>
/subscriptions/[guid_подписки]/resourceGroups/[имя_группы_ресурсов]/providers/Microsoft.KeyVault/vaults/[имя_хранилища_ключей]</br></br>
> Синтаксис значения параметра key-encryption-key — это полный универсальный код ресурса (URI) для KEK:</br>
https://[имя_хранилища_ключей].vault.azure.net/keys/[имя_kek]/[уникальный_идентификатор_kek]

## <a name="check-encryption-progress"></a>Проверка хода выполнения шифрования

Чтобы проверить состояние шифрования диска, используйте команду [az vmss encryption show](/cli/azure/vmss/encryption#az-vmss-encryption-show):

```azurecli-interactive
az vmss encryption show --resource-group myResourceGroup --name myScaleSet
```

Если экземпляры виртуальных машин зашифрованы, код состояния возвращает *EncryptionState/encrypted*, как показано в следующем примере выходных данных:

```console
[
  {
    "disks": [
      {
        "encryptionSettings": null,
        "name": "myScaleSet_myScaleSet_0_disk2_3f39c2019b174218b98b3dfae3424e69",
        "statuses": [
          {
            "additionalProperties": {},
            "code": "EncryptionState/encrypted",
            "displayStatus": "Encryption is enabled on disk",
            "level": "Info",
            "message": null,
            "time": null
          }
        ]
      }
    ],
    "id": "/subscriptions/guid/resourceGroups/MYRESOURCEGROUP/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/virtualMachines/0",
    "resourceGroup": "MYRESOURCEGROUP"
  }
]
```

## <a name="disable-encryption"></a>Отключение шифрования

Если вы решите отказаться от шифрования дисков для экземпляров виртуальных машин, его можно отключить с помощью команды [az vmss encryption disable](/cli/azure/vmss/encryption#az-vmss-encryption-disable):

```azurecli-interactive
az vmss encryption disable --resource-group myResourceGroup --name myScaleSet
```

## <a name="next-steps"></a>Дальнейшие действия

- В этой статье вы зашифровали масштабируемый набор виртуальных машин с помощью Azure CLI. Вы также можете использовать [Azure PowerShell](disk-encryption-powershell.md) или [шаблоны Azure Resource Manager](disk-encryption-azure-resource-manager.md).
- Если вы хотите, чтобы шифрование дисков Azure применялось после подготовки другого расширения, можно использовать [последовательность расширения](virtual-machine-scale-sets-extension-sequencing.md). 
- Полный пример пакетного файла для шифрования дисков данных масштабируемого набора Linux можно найти [здесь](https://gist.githubusercontent.com/ejarvi/7766dad1475d5f7078544ffbb449f29b/raw/03e5d990b798f62cf188706221ba6c0c7c2efb3f/enable-linux-vmss.bat). Этот пример создает группу ресурсов и масштабируемый набор Linux, подключает диск данных размером 5 ГБ и шифрует этот масштабируемый набор виртуальных машин.
