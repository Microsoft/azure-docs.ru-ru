---
title: включить файл
description: включить файл
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 06/19/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 37571a82ca73342b8edfc4702686ccd9091887c4
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771514"
---
Вы можете отключить шифрование с помощью Azure PowerShell, Azure CLI или шаблона Resource Manager. 

>[!IMPORTANT]
>Отключение шифрования, выполняемого с помощью службы шифрования Azure, на виртуальных машинах Linux поддерживается только для томов данных. Если том операционной системы зашифрован, то отключение не поддерживается для томов данных и томов ОС.  

- **Отключение шифрования дисков с помощью Azure PowerShell:** Для отключения шифрования используйте командлет [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). 
     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM' [-VolumeType DATA]
     ```

- **Отключение шифрования дисков с помощью Azure CLI:** чтобы отключить шифрование, используйте команду [az vm encryption disable](/cli/azure/vm/encryption#az_vm_encryption_disable). 
     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type DATA
     ```
- **Отключение шифрования с помощью шаблона Resource Manager:** используйте шаблон [отключения шифрования на работающей виртуальной машине Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm-without-aad) для отключения шифрования.
     1. Нажмите кнопку **Развернуть в Azure**.
     2. Выберите подписку, группу ресурсов, расположение, виртуальную машину, условия использования и соглашение.
