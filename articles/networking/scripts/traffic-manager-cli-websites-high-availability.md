---
title: Пример скрипта Azure CLI. Маршрутизация трафика для обеспечения высокой доступности приложений | Документация Майкрософт
description: Пример скрипта Azure CLI. Маршрутизация трафика для обеспечения высокой доступности приложений.
services: traffic-manager
documentationcenter: traffic-manager
author: asudbring
manager: KumudD
ms.service: traffic-manager
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: traffic-manager
ms.date: 06/26/2018
ms.author: allensu
ms.openlocfilehash: 7103b11b7ee268acbddd8b402e1be1d44074f54d
ms.sourcegitcommit: c7153bb48ce003a158e83a1174e1ee7e4b1a5461
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/15/2021
ms.locfileid: "98234075"
---
# <a name="route-traffic-for-high-availability-of-applications---azure-cli"></a>Маршрутизация трафика для обеспечения высокой доступности приложений — Azure CLI

Этот скрипт создает группу ресурсов, два плана службы приложений, два веб-приложения, профиль и две конечные точки диспетчера трафика. Диспетчер трафика направляет трафик в приложение в основном регионе. Если оно недоступно, трафик направляется в дополнительный регион. Перед выполнением этого скрипта необходимо задать уникальные в Azure значения MyWebApp, MyWebAppL1 и MyWebAppL2. После выполнения вы сможете подключиться к приложению в основном регионе, используя URL-адрес mywebapp.trafficmanager.net.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/traffic-manager/direct-traffic-for-increased-application-availability/direct-traffic-for-increased-application-availability.sh "Route traffic for high availability")]


## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполнив пример скрипта, вы можете удалить группу ресурсов, приложение службы приложений и все связанные ресурсы с помощью следующей команды.

```azurecli
az group delete --name myResourceGroup1 --yes
az group delete --name myResourceGroup2 --yes
```

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, веб-приложения, профиля диспетчера трафика и всех связанных ресурсов этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az appservice plan create](/cli/azure/appservice/plan) | Создает план службы приложений. Это как ферма сервера для веб-приложения Azure. |
| [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) | Создает веб-приложение Azure в плане службы приложений. |
| [az network traffic-manager profile create](/cli/azure/network/traffic-manager/profile) | Создает профиль диспетчера трафика Azure. |
| [az network traffic-manager endpoint create](/cli/azure/network/traffic-manager/endpoint) | Добавление конечной точки в профиль диспетчера трафика Azure. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры скриптов Azure CLI для службы приложений см. в [документации по сетям Azure](../cli-samples.md).