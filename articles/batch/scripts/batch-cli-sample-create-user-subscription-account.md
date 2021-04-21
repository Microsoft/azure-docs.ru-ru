---
title: Пример скрипта Azure CLI для создания учетной записи пакетной службы в режиме подписки пользователя
description: Этот скрипт создает учетную запись пакетной службы Azure в режиме подписки пользователя. Эта учетная запись выделяет вычислительные узлы в подписке.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9bd7b7ac3dbb52ebafa00499e64ec3cff0969a13
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768365"
---
# <a name="cli-example-create-a-batch-account-in-user-subscription-mode"></a>Пример использования CLI: создание учетной записи пакетной службы в режиме подписки пользователя

Этот скрипт создает учетную запись пакетной службы Azure в режиме подписки пользователя. Учетная запись, которая выделяет вычислительные узлы для подписки, должна пройти проверку подлинности с помощью маркера Azure Active Directory. Число выделенных вычислительных узлов учитывается в квоте выделенных виртуальных процессоров (ядер) для вашей подписки. 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этим учебником требуется Azure CLI версии 2.0.20 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.  

## <a name="example-script"></a>Пример сценария

[!code-azurecli-interactive[main](../../../cli_scripts/batch/create-account/create-account-user-subscription.sh "Create Account using user subscription")]

## <a name="clean-up-deployment"></a>Очистка развертывания

Выполните следующую команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az role assignment create](/cli/azure/role); | Создайте новое назначение роли для пользователя, группы или субъекта-службы. |
| [az group create](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az keyvault create](/cli/azure/keyvault#az_keyvault_create) | Создает хранилище ключей. |
| [az keyvault set-policy](/cli/azure/keyvault#az_keyvault_set_policy) | Обновляет политику безопасности указанного хранилища ключей. |
| [az batch account create](/cli/azure/batch/account#az_batch_account_create) | Создает учетную запись пакетной службы.  |
| [az batch account login](/cli/azure/batch/account#az_batch_account_login) | Выполняет проверку подлинности с помощью указанной учетной записи пакетной службы для дальнейшего взаимодействия с интерфейсом командной строки.  |
| [az group delete](/cli/azure/group#az_group_delete) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
