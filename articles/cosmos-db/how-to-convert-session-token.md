---
title: Как преобразовать форматы токенов сеанса в пакет SDK для .NET — Azure Cosmos DB
description: Узнайте, как преобразовать форматы токенов сеанса для обеспечения совместимости различных версий пакета SDK для .NET
author: vinhms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/30/2020
ms.author: vitrinh
ms.custom: devx-track-csharp
ms.openlocfilehash: 7ed06eeb7330989b4a251dc77e516eb8ac578bff
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93342051"
---
# <a name="convert-session-token-formats-in-net-sdk"></a>Преобразование форматов токенов сеанса в пакет SDK для .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье объясняется, как выполнить преобразование между различными форматами маркеров сеанса для обеспечения совместимости между версиями пакета SDK.

> [!NOTE]
> По умолчанию пакет SDK отслеживает маркер сеанса автоматически и будет использовать последний маркер сеанса.  Дополнительные сведения см. на странице [Использование маркеров сеанса](how-to-manage-consistency.md#utilize-session-tokens). Инструкции в этой статье применимы только со следующими условиями.
> * Ваша учетная запись Azure Cosmos DB использует согласованность сеанса.
> * Вы управляете токенами сеанса вручную.
> * Одновременно используется несколько версий пакета SDK.

## <a name="session-token-formats"></a>Форматы токенов сеанса

Существует два формата маркеров сеанса: **простой** и **Векторный**.  Эти два формата не взаимозаменяемы, поэтому формат следует преобразовать при передаче клиентскому приложению с разными версиями.
- Формат **простого** маркера сеанса используется пакетом SDK для .NET V1 (Microsoft.Azure.Docументдб-Version 1. x).
- Формат токена **векторного** сеанса используется пакетом SDK .NET версии 2 (Microsoft.Azure.Docументдб-Version 2. x).

### <a name="simple-session-token"></a>Простой токен сеанса

Простой маркер сеанса имеет следующий формат: `{pkrangeid}:{globalLSN}`

### <a name="vector-session-token"></a>Маркер векторного сеанса

Маркер векторного сеанса имеет следующий формат: `{pkrangeid}:{Version}#{GlobalLSN}#{RegionId1}={LocalLsn1}#{RegionId2}={LocalLsn2}....#{RegionIdN}={LocalLsnN}`

## <a name="convert-to-simple-session-token"></a>Преобразовать в простой токен сеанса

Чтобы передать маркер сеанса клиенту с помощью пакета SDK для .NET v1, используйте **простой** формат маркера сеанса.  Например, используйте следующий пример кода, чтобы преобразовать его.

```csharp
private static readonly char[] SegmentSeparator = (new[] { '#' });
private static readonly char[] PkRangeSeparator = (new[] { ':' });

// sessionTokenToConvert = session token from previous response
string[] items = sessionTokenToConvert.Split(PkRangeSeparator, StringSplitOptions.RemoveEmptyEntries);
string[] sessionTokenSegments = items[1].Split(SessionTokenHelpers.SegmentSeparator, StringSplitOptions.RemoveEmptyEntries);

string sessionTokenInSimpleFormat;

if (sessionTokenSegments.Length == 1)
{
    // returning the same token since it already has the correct format
    sessionTokenInSimpleFormat = sessionTokenToConvert;
}
else
{
    long version = 0;
    long globalLSN = 0;

    if (!long.TryParse(sessionTokenSegments[0], out version)
        || !long.TryParse(sessionTokenSegments[1], out globalLSN))
    {
        throw new ArgumentException("Invalid session token format", sessionTokenToConvert);
    }

    sessionTokenInSimpleFormat = string.Format("{0}:{1}", items[0], globalLSN);
}
```

## <a name="convert-to-vector-session-token"></a>Преобразовать в маркер сеанса вектора

Чтобы передать маркер сеанса клиенту с помощью пакета SDK для .NET версии 2, используйте формат маркера **векторного** сеанса.  Например, используйте следующий пример кода, чтобы преобразовать его.

```csharp

private static readonly char[] SegmentSeparator = (new[] { '#' });
private static readonly char[] PkRangeSeparator = (new[] { ':' });

// sessionTokenToConvert = session token from previous response
string[] items = sessionTokenToConvert.Split(PkRangeSeparator, StringSplitOptions.RemoveEmptyEntries);
string[] sessionTokenSegments = items[1].Split(SegmentSeparator, StringSplitOptions.RemoveEmptyEntries);

string sessionTokenInVectorFormat;

if (sessionTokenSegments.Length == 1)
{
    long globalLSN = 0;
    if (long.TryParse(sessionTokenSegments[0], out globalLSN))
    {
        sessionTokenInVectorFormat = string.Format("{0}:-2#{1}", items[0], globalLSN);
    }
    else
    {
        throw new ArgumentException("Invalid session token format", sessionTokenToConvert);
    }
}
else
{
    // returning the same token since it already has the correct format
    sessionTokenInVectorFormat = sessionTokenToConvert;
}
```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь со следующими статьями:

* [Использование маркеров сеанса](how-to-manage-consistency.md#utilize-session-tokens)
* [Choose the right consistency level](./consistency-levels.md) (Выбор правильного уровня согласованности)
* [Недостатки согласованности, доступности и производительности в Azure Cosmos DB](./consistency-levels.md)
* [Достижение компромисса между доступностью и быстродействием для разных уровней согласованности](./consistency-levels.md)