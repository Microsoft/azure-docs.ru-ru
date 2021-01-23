---
title: Создание и настройка хранилища ключей для службы "Шифрование дисков Azure"
description: В этой статье описаны действия по созданию и настройке хранилища ключей для использования с шифрованием дисков Azure на виртуальной машине Linux.
ms.service: virtual-machines-linux
ms.topic: conceptual
author: msmbaldwin
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 03536bbfedc7f5ecf2fe8d8bb6bd3035f27b72c7
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "98737886"
---
# <a name="creating-and-configuring-a-key-vault-for-azure-disk-encryption"></a>Создание и настройка хранилища ключей для службы "Шифрование дисков Azure"

Служба "Шифрование дисков Azure" использует Azure Key Vault, чтобы управлять секретами и ключами шифрования дисков и контролировать их использование.  Дополнительные сведения о хранилищах ключей см. в статье [Приступая к работе с Azure Key Vault](../../key-vault/general/overview.md) и [Защита хранилища ключей](../../key-vault/general/secure-your-key-vault.md). 

> [!WARNING]
> - Если вы уже использовали службу "Шифрование дисков Azure" c Azure AD для шифрования виртуальной машины, применяйте этот способ шифрования виртуальной машины и далее. Дополнительные сведения см. в статье [Создание и настройка хранилища ключей для службы "Шифрование дисков Azure" c Azure AD (предыдущий выпуск)](disk-encryption-key-vault-aad.md).

Создание и настройка хранилища ключей для использования со службой "Шифрование дисков Azure" состоит из трех этапов.

1. Создание группы ресурсов при необходимости.
2. Создание хранилища ключей. 
3. Установка политик расширенного доступа к хранилищу ключей.

Эти этапы продемонстрированы в следующих кратких руководствах.

- [Создание и шифрование виртуальной машины Linux с помощью Azure CLI](disk-encryption-cli-quickstart.md)
- [Создание и шифрование виртуальной машины Linux с помощью Azure PowerShell](disk-encryption-powershell-quickstart.md)

При необходимости можно создать или импортировать ключ шифрования ключа (KEK).

> [!Note]
> Описанные в этой статье процессы автоматизируются с помощью скриптов подготовки необходимых компонентов для службы "Шифрование дисков Azure". Скрипты доступны для [интерфейса командной строки](https://github.com/ejarvi/ade-cli-getting-started) и [PowerShell](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts).

## <a name="install-tools-and-connect-to-azure"></a>Установка средств и подключение к Azure

Действия, описанные в этой статье, можно выполнить с помощью [Azure CLI](/cli/azure/), [модуля Azure PowerShell Az](/powershell/azure/) или [портала Azure](https://portal.azure.com). 

Доступ к порталу возможен через браузер, тогда как Azure CLI и Azure PowerShell нужно установить локально, как описано в статье [об установке средств для службы "Шифрование дисков Azure" для Linux ](disk-encryption-linux.md#install-tools-and-connect-to-azure).

### <a name="connect-to-your-azure-account"></a>Подключение к учетной записи Azure

Перед использованием Azure CLI или Azure PowerShell нужно установить подключение к подписке Azure. Для этого войдите в систему [с помощью Azure CLI](/cli/azure/authenticate-azure-cli), [Azure PowerShell](/powershell/azure/authenticate-azureps) или укажите учетные данные на портале Azure при появлении соответствующего запроса.

```azurecli-interactive
az login
```

```azurepowershell-interactive
Connect-AzAccount
```

[!INCLUDE [disk-encryption-key-vault](../../../includes/disk-encryption-key-vault.md)]
 
 
## <a name="next-steps"></a>Дальнейшие действия

- [Скрипт CLI для подготовки необходимых компонентов для службы "Шифрование дисков Azure"](https://github.com/ejarvi/ade-cli-getting-started)
- [Скрипт PowerShell для подготовки необходимых компонентов для службы "Шифрование дисков Azure"](https://github.com/Azure/azure-powershell/tree/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts)
- Ознакомьтесь со [сценариями шифрования дисков Azure для виртуальных машин Linux](disk-encryption-linux.md)
- См. сведения об [устранении неполадок со службой "Шифрование дисков Azure"](disk-encryption-troubleshooting.md)
- Изучите [примеры скриптов для службы "Шифрование дисков Azure"](disk-encryption-sample-scripts.md)
