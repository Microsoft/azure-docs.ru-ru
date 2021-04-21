---
title: Пример сценария Azure CLI. Выполнение пакетного задания
description: Этот сценарий создает задание пакетной службы и добавляет в него ряд задач. Он также демонстрирует, как отслеживать задание и его задачи.
ms.topic: sample
ms.date: 12/12/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8e51fa96370b46c9a5a5d5ed6fd7593a06e45430
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768174"
---
# <a name="cli-example-run-a-job-and-tasks-with-azure-batch"></a>Пример CLI. Выполнение задания и задач с использованием пакетной службы Azure

Этот сценарий создает задание пакетной службы и добавляет в него ряд задач. Он также демонстрирует, как отслеживать задание и его задачи. 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим учебником требуется Azure CLI версии 2.0.20 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="example-script"></a>Пример сценария

[!code-azurecli-interactive[main](../../../cli_scripts/batch/run-job/run-job.sh "Run Job")]

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
| [az batch account login](/cli/azure/batch/account#az_batch_account_login) | Выполняет проверку подлинности с помощью указанной учетной записи пакетной службы для дальнейшего взаимодействия с интерфейсом командной строки.  |
| [az batch pool create](/cli/azure/batch/pool#az_batch_pool_create) | Создает пул вычислительных узлов.  |
| [az batch job create](/cli/azure/batch/job#az_batch_job_create) | Создает задание пакетной службы.  |
| [az batch task create](/cli/azure/batch/task#az_batch_task_create) | Добавляет задачу в указанное задание пакетной службы.  |
| [az batch job set](/cli/azure/batch/job#az_batch_job_set) | Обновляет свойства задания пакетной службы.  |
| [az batch job show](/cli/azure/batch/job#az_batch_job_show) | Получает сведения об указанном задании пакетной службы.  |
| [az batch task show](/cli/azure/batch/task#az_batch_task_show) | Получает сведения о задаче из указанного задания пакетной службы.  |
| [az group delete](/cli/azure/group#az_group_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
