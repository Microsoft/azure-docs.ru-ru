---
title: Пример скрипта Azure CLI. Пул Windows в пакетной службе
description: Этот скрипт демонстрирует некоторые из доступных команд Azure CLI для создания пулов вычислительных узлов Windows в пакетной службе Azure и управления ими.
ms.topic: sample
ms.date: 12/12/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 773699dde9342a4b230a08471a289a56fca7e308
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768221"
---
# <a name="cli-example-create-and-manage-a-windows-pool-in-azure-batch"></a>Пример CLI: создание пула Windows и управление им в пакетной службе Azure

Этот скрипт демонстрирует некоторые из доступных команд Azure CLI для создания пулов вычислительных узлов Windows в пакетной службе Azure и управления ими. Пул Windows можно настроить двумя способами: с помощью настройки облачных служб или виртуальных машин. В этом примере показано создание пула Windows с помощью настройки облачных служб.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим учебником требуется Azure CLI версии 2.0.20 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="example-script"></a>Пример сценария

[!code-azurecli-interactive[main](../../../cli_scripts/batch/manage-pool/manage-pool-windows.sh "Manage Windows Cloud Services Pool")]

## <a name="clean-up-deployment"></a>Очистка развертывания

Выполните следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az batch account create](/cli/azure/batch/account#az_batch_account_create) | Создает учетную запись пакетной службы. |
| [az batch account login](/cli/azure/batch/account#az_batch_account_login) | Выполняет проверку подлинности с помощью указанной учетной записи пакетной службы для дальнейшего взаимодействия с интерфейсом командной строки. |
| [az batch pool create](/cli/azure/batch/pool#az_batch_pool_create) | Создает пул вычислительных узлов.  |
| [az batch pool set](/cli/azure/batch/pool#az_batch_pool_set) | Обновляет свойства пула.  |
| [az batch pool autoscale enable](/cli/azure/batch/pool/autoscale#az_batch_pool_autoscale_enable) | Включает автоматическое масштабирование в пуле и применяет формулу.  |
| [az batch pool show](/cli/azure/batch/pool#az_batch_pool_show) | Отображает свойства пула.  |
| [az batch pool autoscale disable](/cli/azure/batch/pool/autoscale#az_batch_pool_autoscale_disable) | Отключает автоматическое масштабирование в пуле. |
| [az group delete](/cli/azure/group#az_group_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
