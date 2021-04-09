---
title: Примеры Azure PowerShell для Azure Cosmos DB с API Gremlin
description: Получение примеров Azure PowerShell для выполнения типичных задач в API Gremlin для Azure Cosmos DB
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: sample
ms.date: 01/20/2021
ms.author: mjbrown
ms.openlocfilehash: a0bd1c732a23ca123c391521f5895eb4dcb8c879
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102503408"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db-gremlin-api"></a>Примеры Azure PowerShell для Azure Cosmos DB с API Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

В приведенной ниже таблице содержатся ссылки на популярные скрипты Azure PowerShell для Azure Cosmos DB. Щелкните ссылки справа, чтобы перейти к примерам API. Общие примеры одинаковы для всех API. Страницы справки по всем командлетам PowerShell Azure Cosmos DB доступны в [справочнике по Azure PowerShell](/powershell/module/az.cosmosdb). Модуль `Az.CosmosDB` теперь является частью модуля `Az`. [Скачайте и установите](/powershell/azure/install-az-ps) последнюю версию модуля Az, чтобы получить командлеты Azure Cosmos DB. Вы также можете получить последнюю версию из [коллекции PowerShell](https://www.powershellgallery.com/packages/Az/5.4.0). Если вы хотите создать вилки этих примеров PowerShell для Cosmos DB, посетите [страницу репозитория с примерами PowerShell для Cosmos DB на сайте GitHub](https://github.com/Azure/azure-docs-powershell-samples/tree/master/cosmosdb).

## <a name="common-samples"></a>Распространенные примеры

|Задача | Описание |
|---|---|
|[Обновление учетной записи](scripts/powershell/common/account-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Обновление уровня согласованности по умолчанию для учетной записи Cosmos DB. |
|[Изменение регионов учетной записи](scripts/powershell/common/update-region.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Изменение регионов учетной записи Cosmos DB. |
|[Изменение приоритета отработки отказа или запуск отработки отказа](scripts/powershell/common/failover-priority-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Изменяет приоритет региональной отработки отказа для учетной записи Azure Cosmos или позволяет вручную запустить отработку отказа. |
|[Ключи или строки подключения учетной записи](scripts/powershell/common/keys-connection-strings.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Получение первичного и вторичного ключей, строк подключения или воссоздание ключа учетной записи Azure Cosmos DB. |
|[Создание учетной записи Cosmos с правилами брандмауэра для IP-адресов](scripts/powershell/common/firewall-create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создание учетной записи Azure Cosmos DB с активными правилами брандмауэра для IP-адресов. |
|||

## <a name="gremlin-api-samples"></a>Примеры API Gremlin

|Задача | Описание |
|---|---|
|[Создание учетной записи, базы данных и графа](scripts/powershell/gremlin/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создание учетной записи Azure Cosmos, базы данных и графа. |
|[Создание учетной записи, базы данных и графа с автомасштабированием](scripts/powershell/gremlin/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создание учетной записи Azure Cosmos, базы данных и графа с автомасштабированием. |
|[Получение списка либо возврат баз данных или графов](scripts/powershell/gremlin/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Получение списка либо возврат баз данных или графов. |
|[Операции с пропускной способностью](scripts/powershell/gremlin/throughput.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Операции c пропускной способностью для базы данных или графа, включая получение, обновление и миграцию между автомасштабируемой и стандартной пропускной способностью. |
|[Блокировка ресурсов от удаления](scripts/powershell/gremlin/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Запрет на удаление ресурсов с помощью блокировок ресурсов. |
|||
