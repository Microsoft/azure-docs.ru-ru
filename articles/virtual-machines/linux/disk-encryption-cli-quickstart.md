---
title: Создание и шифрование виртуальной машины Linux с помощью Azure CLI
description: В этом кратком руководстве содержатся сведения о создании и шифровании виртуальной машины Linux с помощью Azure CLI.
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines
ms.collection: linux
ms.subservice: disks
ms.topic: quickstart
ms.date: 05/17/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5b98fde5e15a3c57b56ecc8aea60023ffb8c22a8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774311"
---
# <a name="quickstart-create-and-encrypt-a-linux-vm-with-the-azure-cli"></a>Краткое руководство. Создание и шифрование виртуальной машины Linux с помощью Azure CLI

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. В этом кратком руководстве показано, как с помощью Azure CLI создать и зашифровать виртуальную машину Linux.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Если вы решили установить и использовать Azure CLI на локальном компьютере, для выполнения инструкций из этого руководства вам потребуется Azure CLI 2.0.30 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name "myResourceGroup" --location "eastus"
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm#az_vm_create). В следующем примере создается виртуальная машина с именем *myVM*.

```azurecli-interactive
az vm create \
    --resource-group "myResourceGroup" \
    --name "myVM" \
    --image "Canonical:UbuntuServer:16.04-LTS:latest" \
    --size "Standard_D2S_V3"\
    --generate-ssh-keys
```

Создание виртуальной машины и вспомогательных ресурсов занимает несколько минут. В следующем примере выходных данных показано, что виртуальная машина успешно создана.

```
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="create-a-key-vault-configured-for-encryption-keys"></a>Создание Key Vault, настроенного для ключей шифрования

Шифрование дисков Azure хранит ключи шифрования в Azure Key Vault. Создайте Key Vault, используя команду [az keyvault create](/cli/azure/keyvault#az_keyvault_create). Чтобы включить в Key Vault возможность хранения ключей шифрования, используйте параметр --enabled-for-disk-encryption.

> [!Important]
> Каждое хранилище ключей должно иметь уникальное имя в Azure. В приведенных ниже примерах замените переменную <your-unique-keyvault-name> выбранным именем.

```azurecli-interactive
az keyvault create --name "<your-unique-keyvault-name>" --resource-group "myResourceGroup" --location "eastus" --enabled-for-disk-encryption
```

## <a name="encrypt-the-virtual-machine"></a>Шифрование виртуальной машины

Зашифруйте виртуальную машину, выполнив команду [az vm encryption](/cli/azure/vm/encryption) и предоставив уникальное имя хранилища ключей в параметр --disk-encryption-keyvault.

```azurecli-interactive
az vm encryption enable -g "MyResourceGroup" --name "myVM" --disk-encryption-keyvault "<your-unique-keyvault-name>"
```

Через некоторое время процесс вернет сообщение: The encryption request was accepted. Please use 'show' command to monitor the progress (Запрос на шифрование принят. Выполните команду для просмотра, чтобы отследить прогресс.). Командой для просмотра является команда [az vm show](/cli/azure/vm/encryption#az_vm_encryption_show).

```azurecli-interactive
az vm encryption show --name "myVM" -g "MyResourceGroup"
```

Если шифрование включено, в полученных выходных данных вы увидите следующий код:

```
"EncryptionOperation": "EnableEncryption"
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда группа ресурсов, виртуальная машина и Key Vault станут ненужными, их можно удалить с помощью команды [az group delete](/cli/azure/group). 

```azurecli-interactive
az group delete --name "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы создали виртуальную машину, включенный для ключей шифрования Key Vault и зашифровали виртуальную машину.  Перейдите к следующей статье, чтобы узнать больше о шифровании дисков Azure для виртуальных машин Linux.

> [!div class="nextstepaction"]
> [Общие сведения о шифровании дисков Azure](disk-encryption-overview.md)
