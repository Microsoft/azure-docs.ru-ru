---
title: Пример для CLI. Настройка аудита и Расширенной защиты от угроз для Базы данных SQL Azure
description: Пример скрипта Azure CLI для настройки аудита и Расширенной защиты от угроз для Базы данных SQL Azure
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: security
ms.devlang: azurecli
ms.topic: sample
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 02/09/2021
ms.openlocfilehash: 7450911dabe04ea04ba16ec5e648f09ecf4a616f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100095993"
---
# <a name="use-cli-to-configure-sql-database-auditing-and-advanced-threat-protection"></a>Настройка аудита и Расширенной защиты от угроз для Базы данных SQL с помощью CLI

Этот пример скрипта Azure CLI настраивает аудит и Расширенную защита от угроз для Базы данных SQL.

Если вы решили установить и использовать интерфейс командной строки локально, для работы с этим руководством вам понадобится Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Пример скрипта

### <a name="sign-in-to-azure"></a>Вход в Azure

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

```azurecli-interactive
$subscription = "<subscriptionId>" # add subscription here

az account set -s $subscription # ...or use 'az login'
```

### <a name="run-the-script"></a>Выполнение скрипта

```azurecli-interactive
#!/bin/bash
location="East US"
randomIdentifier=random123

resource="resource-$randomIdentifier"
server="server-$randomIdentifier"
database="database-$randomIdentifier"
storage="storage$randomIdentifier"

notification="changeto@your.email;changeto@your.email"

login="sampleLogin"
password="samplePassword123!"

echo "Using resource group $resource with login: $login, password: $password..."

echo "Creating $resource..."
az group create --name $resource --location "$location"

echo "Creating $server in $location..."
az sql server create --name $server --resource-group $resource --location "$location" --admin-user $login --admin-password $password

echo "Creating $database on $server..."
az sql db create --name $database --resource-group $resource --server $server --service-objective S0

echo "Creating $storage..."
az storage account create --name $storage --resource-group $resource --location "$location" --sku Standard_LRS

echo "Setting access policy on $storage..."
az sql db audit-policy update --resource-group $resource --server $server --name $database --state Enabled --blob-storage-target-state Enabled --storage-account $storage

echo "Setting threat detection policy on $storage..."
az sql db threat-policy update --email-account-admins Disabled --email-addresses $notification --name $database --resource-group $resource --server $server --state Enabled --storage-account $storage
```

### <a name="clean-up-deployment"></a>Очистка развертывания

Используйте приведенную ниже команду, чтобы удалить группу ресурсов и все связанные с ней ресурсы.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Примеры

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Описание |
|---|---|
| [az sql db audit-policy](/cli/azure/sql/db/audit-policy) | Задает политику аудита для базы данных. |
| [az sql db threat-policy](/cli/azure/sql/db/threat-policy) | Задает политику Расширенной защиты от угроз для базы данных. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).

Дополнительные примеры сценариев интерфейса командной строки для Базы данных SQL Azure см. в [документации по Базе данных SQL](../../azure-sql/database/az-cli-script-samples-content-guide.md).
