---
title: Пакет SDK для .NET и ресурсы API таблиц для базы данных Azure Cosmos DB
description: Узнайте обо всех Azure Cosmos DB API таблиц для .NET, включая даты выпуска, даты выбытия и изменения, внесенные в каждую версию.
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/17/2018
ms.custom: devx-track-dotnet
ms.openlocfilehash: 5af77f34f8e82edf187c1a68b61995ec5c82a8d9
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105566835"
---
# <a name="azure-cosmos-db-table-net-api-download-and-release-notes"></a>API таблицы .NET для базы данных Azure Cosmos DB. Скачивание и заметки о выпуске
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

> [!div class="op_single_selector"]
> * [.NET](table-sdk-dotnet.md)
> * [.NET Standard](table-sdk-dotnet-standard.md)
> * [Java](table-sdk-java.md)
> * [Node.js](table-sdk-nodejs.md)
> * [Python](table-sdk-python.md)

|   | Ссылки|
|---|---|
|**Скачивание пакета SDK**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table)|
|**Краткое руководство**|[Azure Cosmos DB: Build a .NET application using the Table API](create-table-dotnet.md) (Создание приложения .NET с помощью API таблицы Azure Cosmos DB)|
|**Руководство**|[Разработка с помощью API таблицы базы данных Azure Cosmos DB на языке .NET](tutorial-develop-table-dotnet.md)|
|**Текущая поддерживаемая платформа**|[Microsoft .NET Framework 4.5.1](https://www.microsoft.com/en-us/download/details.aspx?id=40779)|

> [!IMPORTANT]
> Библиотека [Microsoft.Azure.CosmosDB.Table](https://www.nuget.org/packages/Microsoft.Azure.CosmosDB.Table) пакета SDK для .NET Framework находится в режиме обслуживания. Ее поддержка скоро будет прекращена. Выполните обновление до новой библиотеки .NET Standard [Microsoft.Azure.Cosmos.Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), чтобы и дальше получать последние функции, которые поддерживает API таблиц.

> Если вы создали учетную запись API таблиц на этапе работы с предварительной версией, для работы с общедоступными пакетами SDK для API таблиц создайте [новую учетную запись](create-table-dotnet.md#create-a-database-account).
>

## <a name="release-notes"></a>Заметки о выпуске

### <a name="212"></a><a name="2.1.2"></a>2.1.2

* Исправленные ошибки

### <a name="210"></a><a name="2.1.0"></a>2.1.0

* Исправленные ошибки

### <a name="200"></a><a name="2.0.0"></a>2.0.0

* Добавлена поддержка для операций записи в нескольких регионах
* Исправлены основные зависимости пакетов NuGet в Microsoft.Azure.DocumentDB, Microsoft.OData.Core, Microsoft.OData.Edm, Microsoft.Spatial.

### <a name="113"></a><a name="1.1.3"></a>1.1.3

* Исправлены зависимости пакетов NuGet от Microsoft.Azure.Storage.Common и Microsoft.Azure.DocumentDB.
* Исправление ошибок в сериализации таблиц при настройке JsonConvert.DefaultSettings.

### <a name="111"></a><a name="1.1.1"></a>1.1.1

* Добавлена проверка тегов сущности неправильного формата в режиме прямого подключения.
* Исправлена ошибка, связанная с запросом LINQ, в режиме шлюза.
* Синхронные API теперь запускаются в пуле потоков с помощью SynchronizationContext.

### <a name="110"></a><a name="1.1.0"></a>1.1.0

* Добавлены TableRequestOptions TableQueryMaxItemCount, TableQueryEnableScan, TableQueryMaxDegreeOfParallelism и TableQueryContinuationTokenLimitInKb
* Исправления ошибок

### <a name="100"></a><a name="1.0.0"></a>1.0.0

* Выпуск общедоступной версии

### <a name="090-preview"></a><a name="0.1.0-preview"></a>0.9.0 (предварительная версия)

* Первоначальный выпуск предварительной версии

## <a name="release-and-retirement-dates"></a>Даты выпуска и вывода из эксплуатации

Корпорация Майкрософт отправляет уведомление минимум за **12 месяцев** до вывода пакета SDK из эксплуатации, чтобы обеспечить более плавный переход на новую или поддерживаемую версию.

`Microsoft.Azure.CosmosDB.Table`Библиотека в настоящее время доступна только для платформа .NET Framework и находится в режиме обслуживания и скоро будет считаться устаревшей. Новые функции и возможности и оптимизации добавляются только в библиотеку .NET Standard [Microsoft. Azure. Cosmos. Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table), поэтому рекомендуется выполнить обновление до [Microsoft. Azure. Cosmos. Table](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table).

Пакет предварительной версии [WindowsAzure. Storage-PremiumTable](https://www.nuget.org/packages/WindowsAzure.Storage-PremiumTable/0.1.0-preview) является устаревшим. Поддержка пакета SDK WindowsAzure.Storage PremiumTable будет прекращена 15 ноября 2018 г. С этого времени запросы к устаревшему пакету SDK будут запрещены.

| Версия | Дата выпуска | Дата вывода |
| --- | --- | --- |
| [2.1.2](#2.1.2) |16 сентября 2019 г.| |
| [2.1.0](#2.1.0) |22 января 2019 г.|01 апреля 2020 г. |
| [2.0.0](#2.0.0) |26 сентября 2018 г.|01 марта 2020 г. |
| [1.1.3](#1.1.3) |17 июля 2018 г.|01 декабря 2019 г. |
| [1.1.1](#1.1.1) |26 марта 2018 г.|01 декабря 2019 г. |
| [1.1.0](#1.1.0) |21 февраля 2018 г.|01 декабря 2019 г. |
| [1.0.0](#1.0.0) |15 ноября 2017 г.|15 ноября 2019 г. |
| 0.9.0 (предварительная версия) |11 ноября 2017 г. |11 ноября 2019 г. |

## <a name="troubleshooting"></a>Устранение неполадок

Если возникает ошибка 

```
Unable to resolve dependency 'Microsoft.Azure.Storage.Common'. Source(s) used: 'nuget.org', 
'CliFallbackFolder', 'Microsoft Visual Studio Offline Packages', 'Microsoft Azure Service Fabric SDK'`
```

при попытке использовать NuGet-пакет Microsoft.Azure.CosmosDB.Table, ее можно устранить двумя способами.

* Установите пакет Microsoft.Azure.CosmosDB.Table и его зависимости с помощью консоли диспетчера пакетов. Для этого введите в консоли диспетчера пакетов такую команду: 

    ```powershell
    Install-Package Microsoft.Azure.CosmosDB.Table -IncludePrerelease
    ```

    
* Используя предпочитаемый инструмент управления пакетами NuGet, сначала установите пакет Microsoft.Azure.Storage.Common, а потом — Microsoft.Azure.CosmosDB.Table.

## <a name="faq"></a>ВОПРОСЫ И ОТВЕТЫ

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>См. также раздел

Дополнительные сведения об API таблицы для базы данных Azure Cosmos DB см. в [этой статье](table-introduction.md). 
