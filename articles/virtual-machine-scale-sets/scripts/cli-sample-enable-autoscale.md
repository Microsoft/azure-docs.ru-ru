---
title: Примеры для Azure CLI. Включение автомасштабирования на основе узла
description: С помощью этого скрипта создается масштабируемый набор виртуальных машин под управлением Ubuntu. При этом используются метрики на основе узла для автоматического масштабирования в соответствии с изменением нагрузки на ЦП.
author: ju-shim
tags: azure-resource-manager
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.devlang: azurecli
ms.topic: sample
ms.date: 03/27/2018
ms.author: jushiman
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 8e8118a32da296d090ba22122b5188f941419e94
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87502143"
---
# <a name="automatically-scale-a-virtual-machine-scale-set-with-the-azure-cli"></a>Автоматическое масштабирование масштабируемых наборов виртуальных машин с помощью Azure CLI
С помощью этого скрипта создается масштабируемый набор виртуальных машин под управлением Ubuntu. При этом используются метрики на основе узла для автоматического масштабирования в соответствии с изменением нагрузки на ЦП.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта
[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine-scale-sets/auto-scale-host-metrics/auto-scale-host-metrics.sh "Automatically scale a virtual machine scale set")]

## <a name="clean-up-deployment"></a>Очистка развертывания
Выполните следующую команду, чтобы удалить группу ресурсов, масштабируемый набор и все связанные с ними ресурсы.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Описание скрипта
Для создания группы ресурсов, масштабируемого набора виртуальных машин и всех связанных ресурсов в этом скрипте используются приведенные ниже команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/ad/group) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az vmss create](/cli/azure/vmss) | Создает масштабируемый набор виртуальных машин и подключает его к виртуальной сети, подсети и группе безопасности сети. Чтобы распределить трафик между несколькими экземплярами виртуальных машин, создается еще и подсистема балансировки нагрузки. Эта команда также указывает образ виртуальной машины, который будет использоваться, и учетные данные администратора.  |
| [az monitor autoscale-settings create](/cli/azure/monitor/autoscale-settings) | Создание и применение правила автоматического масштабирования для масштабируемого набора виртуальных машин. |
| [az group delete](/cli/azure/ad/group) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure/overview).
