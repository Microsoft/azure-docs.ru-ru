---
title: Пример скрипта Azure CLI. Запуск виртуальной машины в лаборатории | Документация Майкрософт
description: Этот скрипт Azure CLI предназначен для запуска виртуальной машины в лаборатории в Azure DevTest Labs.
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: f82d43fc212da034d978ee3d65236d806daeb008
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764987"
---
# <a name="use-azure-cli-to-start-a-virtual-machine-in-a-lab-in-azure-devtest-labs"></a>Запуск виртуальной машины в лаборатории в Azure DevTest Labs с помощью Azure CLI

Этот скрипт Azure CLI предназначен для запуска виртуальной машины (VM) в лаборатории. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/devtest-lab/start-connect-virtual-machine-in-lab/start-connect-virtual-machine-in-lab.sh "Start a VM")]


## <a name="script-explanation"></a>Описание скрипта

Этот сценарий использует следующие команды:

| Get-Help | Примечания |
|---|---|
| [az lab vm start](/cli/azure/lab/vm#az_lab_vm_start) | Запуск виртуальной машины в лаборатории. Эта операция может занять некоторое время. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

См. [дополнительные примеры скриптов CLI для Служб лабораторий Azure](../samples-cli.md).
