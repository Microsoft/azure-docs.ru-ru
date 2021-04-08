---
title: Поддержка Хранилища таблиц в Azure Cosmos DB
description: Узнайте об особенностях взаимодействия API таблиц Azure Cosmos DB и таблиц службы хранилища Azure при использовании одной и той же модели данных таблиц и операций
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 01/08/2021
author: sakash279
ms.author: akshanka
ms.reviewer: sngun
ms.openlocfilehash: c2b2a9a03a654e8ccba58e62a1d017cd11db5abc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98050773"
---
# <a name="developing-with-azure-cosmos-db-table-api-and-azure-table-storage"></a>Разработка с использованием API таблиц Azure Cosmos DB и хранилища таблиц Azure
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

API таблиц Azure Cosmos DB и хранилище таблиц Azure используют одну и ту же модель данных и предоставляют одинаковые операции создания, удаления, обновления и запроса в своих пакетах SDK.

> [!NOTE]
> В API таблиц Azure Cosmos DB теперь доступен [режим бессерверной емкости](serverless.md).

[!INCLUDE [storage-table-cosmos-comparison](../../includes/storage-table-cosmos-comparison.md)]

## <a name="developing-with-the-azure-cosmos-db-table-api"></a>Разработка с использованием API таблиц Azure Cosmos DB

В настоящее время [API таблиц в Azure Cosmos DB](table-introduction.md) содержит четыре пакета SDK для разработки: 

* [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table). Пакет SDK .NET. Эта библиотека предназначена для .NET Standard и включает в себя те же классы и подписи методов, что и общедоступный [пакет SDK WindowsAzure.Storage](https://www.nuget.org/packages/WindowsAzure.Storage). Кроме того, она может подключаться к учетным записям Azure Cosmos DB через API таблиц. Пользователям библиотеки .NET Framework [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table/) рекомендуется выполнить обновление до [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), так как библиотека сейчас находится в режиме обслуживания и ее поддержка скоро будет прекращена.

* [Пакет SDK для Python.](table-sdk-python.md) Новый пакет SDK для Python в Azure Cosmos DB — единственный пакет SDK, который поддерживает хранилище таблиц Azure в Python. Этот пакет SDK подключается к хранилищу таблиц Azure и API таблиц Azure Cosmos DB.

* [Пакет SDK для Java](table-sdk-java.md). Этот пакет SDK для службы хранилища Azure может подключаться к учетным записям Azure Cosmos DB с помощью API таблиц.

* [Пакет SDK для Node.js](table-sdk-nodejs.md). Этот пакет SDK для службы хранилища Azure может подключаться к учетным записям Azure Cosmos DB с помощью API таблиц.


Дополнительные сведения о работе с API таблиц доступны в статье [Часто задаваемые вопросы о различных API в службе Azure Cosmos DB](table-api-faq.md).

## <a name="developing-with-azure-table-storage"></a>Разработка с использованием хранилища таблиц Azure

Хранилище таблиц Azure содержит следующие пакеты SDK для разработки.

- Библиотеки [Microsoft.Azure.Storage.Blob](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/), [Microsoft.Azure.Storage.File](https://www.nuget.org/packages/Microsoft.Azure.Storage.File/), [Microsoft.Azure.Storage.Queue](https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/) и [Microsoft.Azure.Storage.Common](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/) позволяют взаимодействовать со службой Хранилища таблиц Azure. Если вы используете API таблиц в Azure Cosmos DB, можно использовать вместо них библиотеку [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table/).
- [Пакет SDK для Python](https://github.com/Azure/azure-cosmos-table-python). Пакет SDK таблиц Azure Cosmos DB для Python поддерживает Хранилище таблиц, так как Хранилище таблиц Azure и API таблиц Cosmos DB совместно используют одни и те же функции и компоненты. Поэтому, чтобы упростить разработку, мы рекомендуем использовать именно этот пакет SDK.
- [Пакет SDK службы хранилища Azure для Java](https://github.com/azure/azure-storage-java). Этот пакет SDK хранилища Azure предоставляет клиентскую библиотеку Java для использования хранилища таблиц Azure.
- [Пакет SDK для Node.js](https://github.com/Azure/azure-storage-node). Этот пакет SDK содержит пакет Node.js и совместимую клиентскую библиотеку JavaScript для использования службы хранилища таблиц.
- [Модуль AzureRmStorageTable PowerShell](https://www.powershellgallery.com/packages/AzureRmStorageTable). Этот модуль PowerShell содержит командлеты для работы с таблицами хранилища.
- [Клиентская библиотека службы хранилища Azure для C++](https://github.com/Azure/azure-storage-cpp/). Эта библиотека позволяет создавать приложения для службы хранилища Azure.
- [Клиентская библиотека таблиц хранилища Azure для Ruby](https://github.com/azure/azure-storage-ruby/tree/master/table). Этот проект содержит пакет Ruby, который упрощает доступ к службам таблиц хранилища Azure.
- [Клиентская библиотека PHP для таблиц хранилища Azure](https://github.com/Azure/azure-storage-php/tree/master/azure-storage-table). Этот проект содержит клиентскую библиотеку PHP, которая упрощает доступ к службам таблиц хранилища Azure.


   





