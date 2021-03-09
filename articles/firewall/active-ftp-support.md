---
title: Поддержка Active FTP в брандмауэре Azure
description: По умолчанию активный FTP отключен в брандмауэре Azure. Его можно включить с помощью шаблонов PowerShell, CLI и ARM.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 03/05/2021
ms.author: victorh
ms.openlocfilehash: adbc2a9eb6cd3b054df84911604143ddb711ad20
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102499141"
---
# <a name="azure-firewall-active-ftp-support"></a>Поддержка Active FTP в брандмауэре Azure

При использовании Active FTP сервер FTP инициирует подключение данных к назначенному порту данных FTP-клиента. Брандмауэры в сети на стороне клиента обычно блокируют внешний запрос на подключение к внутреннему порту клиента. Дополнительные сведения см. [в разделе Active FTP и пассивный FTP, подробное объяснение](https://slacksite.com/other/ftp.html).

По умолчанию поддержка Active FTP в брандмауэре Azure отключена для защиты от атак с использованием FTP-сервера с помощью `PORT` команды FTP. Однако можно включить Active FTP при развертывании с помощью Azure PowerShell, Azure CLI или шаблона Azure ARM.


## <a name="azure-powershell"></a>Azure PowerShell

Чтобы выполнить развертывание с помощью Azure PowerShell, используйте `AllowActiveFTP` параметр. Дополнительные сведения см. в разделе [Создание брандмауэра с разрешающим активным FTP](/powershell/module/az.network/new-azfirewall#16---create-a-firewall-with-allow-active-ftp-).

## <a name="azure-cli"></a>Azure CLI

Чтобы выполнить развертывание с помощью Azure CLI, используйте `--allow-active-ftp` параметр. Дополнительные сведения см. в статье [AZ Network Firewall Create](/cli/azure/ext/azure-firewall/network/firewall#ext_azure_firewall_az_network_firewall_create-optional-parameters). 

## <a name="azure-resource-manager-arm-template"></a>Шаблон Azure Resource Manager (ARM)

Чтобы выполнить развертывание с помощью шаблона ARM, используйте `AdditionalProperties` поле:

```json
"additionalProperties": {
            "Network.FTP.AllowActiveFTP": "True"
        },
```
Дополнительные сведения см. в разделе [Microsoft. Network азурефиреваллс](/azure/templates/microsoft.network/azurefirewalls).

## <a name="next-steps"></a>Дальнейшие действия

Сведения о развертывании брандмауэра Azure см. в статье [развертывание и настройка брандмауэра Azure с помощью Azure PowerShell](deploy-ps.md).