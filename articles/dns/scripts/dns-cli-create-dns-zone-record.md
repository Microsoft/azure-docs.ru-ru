---
title: Создание зоны и записи DNS для доменного имени — Azure CLI — Azure DNS
description: В этом примере скрипта Azure CLI показано, как создать зону и запись DNS для имени домена
services: dns
author: rohinkoul
ms.service: dns
ms.topic: sample
ms.date: 09/20/2019
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5692c7a81d34ec9005c1c4675c71d63e697c5f47
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783707"
---
# <a name="azure-cli-script-example-create-a-dns-zone-and-record"></a>Пример скрипта Azure CLI: создание зоны и записи DNS

С помощью этого примера скрипта Azure CLI создается зона и запись DNS для имени домена. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Пример скрипта

[!code-azurecli-interactive[main](../../../cli_scripts/dns/create-dns-zone-and-record/create-dns-zone-and-record.sh "Create DNS zone and record")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

Выполните следующую команду, чтобы удалить группу ресурсов, зону DNS и все связанные с ними ресурсы.

```azurecli
az group delete -n myResourceGroup
```

## <a name="script-explanation"></a>Описание скрипта

Для создания группы ресурсов, виртуальной машины, группы доступности, балансировщика нагрузки и всех связанных ресурсов этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [az network dns zone create](/cli/azure/network/dns/zone#az_network_dns_zone_create) | Создает зону Azure DNS. |
| [az network dns record-set a add-record](/cli/azure/network/dns/record-set) | Добавляет запись *A* в зону DNS. |
| [az network dns record-set list](/cli/azure/network/dns/record-set) | Выводит список всех наборов записей *A* в зоне DNS. |
| [az group delete](/cli/azure/vm/extension#az_vm_extension_set) | Удаляет группу ресурсов со всеми вложенными ресурсами. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure).