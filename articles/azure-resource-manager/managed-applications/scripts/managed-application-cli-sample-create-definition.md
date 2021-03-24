---
title: Создание определения управляемого приложения с помощью Azure CLI
description: Здесь представлен пример скрипта Azure CLI, с помощью которого можно создать определение управляемого приложения в подписке.
author: tfitzmac
ms.devlang: azurecli
ms.topic: sample
ms.date: 10/25/2017
ms.author: tomfitz
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7feb00b581732cdc1956c4ac23af571180ff09e0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87497840"
---
# <a name="create-a-managed-application-definition-with-azure-cli"></a>Создание определения управляемого приложения с помощью Azure CLI

С помощью этого скрипта определение управляемого приложения публикуется в каталоге службы. 


[!INCLUDE [sample-cli-install](../../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli[main](../../../../cli_scripts/managed-applications/create-definition/create-definition.sh "Create definition")]


## <a name="script-explanation"></a>Описание скрипта

В этом скрипте используется приведенная ниже команда для создания определения управляемого приложения. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az managedapp definition create](/cli/azure/managedapp/definition#az-managedapp-definition-create) | Позволяет создать определение управляемого приложения. Укажите пакет с необходимыми файлами. |


## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения об управляемых приложениях Azure см. в [этой статье](../overview.md).
* Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).
