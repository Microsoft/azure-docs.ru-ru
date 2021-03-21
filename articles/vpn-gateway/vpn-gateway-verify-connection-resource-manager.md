---
title: 'VPN-шлюз Azure: Проверка подключения шлюза'
description: В этой статье показано, как проверить подключение VPN-шлюза виртуальной сети.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/19/2020
ms.author: cherylmc
ms.custom: devx-track-azurecli
ms.openlocfilehash: b59294d07ef64875cb6fbd3e3a49dec61d8b8135
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94659409"
---
# <a name="verify-a-vpn-gateway-connection"></a>Проверка подключения VPN-шлюза

В этой статье вы узнаете, как проверить подключение VPN-шлюза в классической модели и в модели Resource Manager.

## <a name="azure-portal"></a>Портал Azure

[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="powershell"></a>PowerShell

Чтобы проверить подключение VPN-шлюза для модели развертывания Resource Manager с помощью PowerShell, установите последнюю версию [командлетов PowerShell для Azure Resource Manager](/powershell/azure/).

[!INCLUDE [PowerShell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="azure-cli"></a>Azure CLI

Чтобы проверить подключение VPN-шлюза для модели развертывания Resource Manager с помощью Azure CLI, установите последнюю версию [командлетов CLI](/cli/azure/install-azure-cli) (2.0 или более поздней версии).

[!INCLUDE [CLI](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]

## <a name="azure-portal-classic"></a>Портал Azure (классическая модель)

[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

## <a name="powershell-classic"></a>PowerShell (классическая модель)

Чтобы проверить подключение VPN-шлюза для классической модели развертывания с помощью PowerShell, установите последнюю версию командлетов Azure PowerShell. Не забудьте загрузить и установить модуль [управления службами](/powershell/azure/servicemanagement/install-azure-ps?#azure-service-management-cmdlets). Используйте командлет Add-AzureAccount, чтобы войти в классическую модель развертывания.

[!INCLUDE [Classic PowerShell](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

* В виртуальные сети можно добавлять виртуальные машины. Инструкции см. в статье о [создании виртуальной машины](../virtual-machines/windows/quick-create-portal.md).