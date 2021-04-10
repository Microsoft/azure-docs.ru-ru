---
title: Примеры Azure PowerShell для API Azure Cosmos DB для MongoDB
description: Получение примеров Azure PowerShell для выполнения типичных задач в API Azure Cosmos DB для MongoDB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 01/20/2021
ms.author: mjbrown
ms.openlocfilehash: 2f852b1ec359033c8bc700c8a382705b4479d6de
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102503357"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db-api-for-mongodb"></a>Примеры Azure PowerShell для API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

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

## <a name="mongo-db-api-samples"></a>Примеры API Mongo DB

|Задача | Описание |
|---|---|
|[Создание учетной записи, базы данных и коллекции](scripts/powershell/mongodb/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создание учетной записи Azure Cosmos, базы данных и коллекции. |
|[Создание учетной записи, базы данных и коллекции с автомасштабированием](scripts/powershell/mongodb/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Создание учетной записи Azure Cosmos, базы данных и коллекции с автомасштабированием. |
|[Получение списка либо возврат баз данных или коллекций](scripts/powershell/mongodb/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Получение списка либо возврат баз данных или коллекций. |
|[Операции с пропускной способностью](scripts/powershell/mongodb/throughput.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Операции c пропускной способностью для базы данных или коллекции, включая получение, обновление и миграцию между автомасштабируемой и стандартной пропускной способностью. |
|[Блокировка ресурсов от удаления](scripts/powershell/mongodb/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Запрет на удаление ресурсов с помощью блокировок ресурсов. |
|||
