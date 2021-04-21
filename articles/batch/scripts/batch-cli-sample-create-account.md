---
title: Пример скрипта Azure CLI для создания учетной записи пакетной службы
description: С помощью этого скрипта создается учетная запись пакетной службы Azure в режиме пакетной службы. Кроме того, в скрипте показано, как запрашивать и обновлять различные свойства учетной записи.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3d978cf56e92abf9824c116e51f94d53bbe14bbb
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768339"
---
# <a name="cli-example-create-a-batch-account-in-batch-service-mode"></a>Пример скрипта CLI: создание учетной записи пакетной службы в режиме пакетной службы

С помощью этого скрипта создается учетная запись пакетной службы Azure в режиме пакетной службы. Кроме того, в скрипте показано, как запрашивать и обновлять различные свойства учетной записи. Когда вы создаете учетную запись пакетной службы в стандартном режиме пакетной службы, ее вычислительные узлы назначаются внутри пакетной службы. К выделенным вычислительным узлам будет применяться отдельная квота на виртуальные процессоры (ядра), и учетная запись сможет проходить проверку подлинности с помощью учетных данных общего ключа или маркера Azure Active Directory.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим учебником требуется Azure CLI версии 2.0.20 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 

## <a name="example-script"></a>Пример сценария

[!code-azurecli-interactive[main](../../../cli_scripts/batch/create-account/create-account.sh "Create Account")]

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
| [az storage account create](/cli/azure/storage/account#az_storage_account_create) | Создание учетной записи хранения. |
| [az batch account set](/cli/azure/batch/account#az_batch_account_set) | Обновляет свойства учетной записи пакетной службы.  |
| [az batch account show](/cli/azure/batch/account#az_batch_account_show) | Получает сведения об указанной учетной записи пакетной службы.  |
| [az batch account keys list](/cli/azure/batch/account/keys#az_batch_account_keys_list) | Получает ключи доступа указанной учетной записи пакетной службы.  |
| [az batch account login](/cli/azure/batch/account#az_batch_account_login) | Выполняет проверку подлинности с помощью указанной учетной записи пакетной службы для дальнейшего взаимодействия с интерфейсом командной строки.  |
| [az group delete](/cli/azure/group#az_group_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
