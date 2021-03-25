---
title: Краткое руководство. Создание Azure Key Vault с помощью Azure PowerShell
description: Краткое руководство по созданию Azure Key Vault с помощью Azure PowerShell.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: quickstart
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: e77493bc73bc2d6f590d9bdcf891171fbd71f74e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99070228"
---
# <a name="quickstart-create-a-key-vault-using-powershell"></a>Краткое руководство. Создание хранилища ключей с помощью PowerShell

Azure Key Vault — это облачная служба, которая предоставляет защищенное хранилище для [ключей](../keys/index.yml), [секретов](../secrets/index.yml) и [сертификатов](../certificates/index.yml). Дополнительные сведения о Key Vault см. в [этой статье](overview.md), а сведения о том, что можно хранить в хранилище ключей, см. [здесь](about-keys-secrets-certificates.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Из этого краткого руководства вы узнаете, как создать хранилище ключей с помощью [Azure PowerShell](/powershell/azure/). Чтобы установить и использовать PowerShell локально для работы с этим руководством, понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Чтобы узнать версию, введите `$PSVersionTable.PSVersion`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Login-AzAccount`, чтобы создать подключение к Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create a resource group](../../../includes/key-vault-powershell-rg-creation.md)]

## <a name="create-a-key-vault"></a>Создание хранилища ключей

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-kv-creation.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-delete-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали хранилище ключей с помощью Azure PowerShell. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](overview.md)
- Справочник по [командлетам Azure PowerShell для Azure Key Vault](/powershell/module/az.keyvault/)
- Статья [Обзор системы безопасности Azure Key Vault](security-overview.md)

