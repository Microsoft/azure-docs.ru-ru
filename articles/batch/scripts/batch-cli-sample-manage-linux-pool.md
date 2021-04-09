---
title: Пример скрипта Azure CLI для создания пула Linux в пакетной службе
description: В этом скрипте показаны некоторые из доступных команд Azure CLI для создания пулов вычислительных узлов Linux в пакетной службе Azure и управления ими.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: b2e0fbf44be5718cf5577f6bc9aea436968e2fc3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93073557"
---
# <a name="cli-example-create-and-manage-a-linux-pool-in-azure-batch"></a>Пример использования CLI: создание пула Linux и управление им в пакетной службе Azure

В этом скрипте показаны некоторые из доступных команд Azure CLI для создания пулов вычислительных узлов Linux в пакетной службе Azure и управления ими.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим учебником требуется Azure CLI версии 2.0.20 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="example-script"></a>Пример сценария

[!code-azurecli-interactive[main](../../../cli_scripts/batch/manage-pool/manage-pool-linux.sh "Manage Linux Virtual Machine Pool")]

## <a name="clean-up-deployment"></a>Очистка развертывания

Выполните следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az batch account create](/cli/azure/batch/account#az-batch-account-create) | Создает учетную запись пакетной службы. |
| [az batch account login](/cli/azure/batch/account#az-batch-account-login) | Выполняет проверку подлинности с помощью указанной учетной записи пакетной службы для дальнейшего взаимодействия с интерфейсом командной строки.  |
| [az batch pool node-agent-skus list](../batch-linux-nodes.md#list-of-virtual-machine-images) | Отображает список доступных номеров SKU агента узла и сведения об образе.  |
| [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create) | Создает пул вычислительных узлов.  |
| [az batch pool resize](/cli/azure/batch/pool#az-batch-pool-resize) | Изменяет количество работающих виртуальных машин в указанном пуле.  |
| [az batch pool show](/cli/azure/batch/pool#az-batch-pool-show) | Отображает свойства пула.  |
| [az batch node list](/cli/azure/batch/node#az-batch-node-list) | Отображает список всех вычислительных узлов в указанном пуле.  |
| [az batch node reboot](/cli/azure/batch/node#az-batch-node-reboot) | Перезапускает указанный вычислительный узел.  |
| [az batch node delete](/cli/azure/batch/node#az-batch-node-delete) | Удаляет указанные узлы из выбранного пула.  |
| [az group delete](/cli/azure/group#az-group-delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
