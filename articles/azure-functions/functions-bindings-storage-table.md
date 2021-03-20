---
title: Привязки хранилища таблиц Azure для службы "Функции Azure"
description: Узнайте, как использовать привязки службы хранилища таблиц Azure в службе "Функции Azure".
author: craigshoemaker
ms.topic: reference
ms.date: 09/03/2018
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 4f2b890dc60cd50b5fcaefabe8d418268b738c20
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92096731"
---
# <a name="azure-table-storage-bindings-for-azure-functions"></a>Привязки хранилища таблиц Azure для службы "Функции Azure"

Функции Azure интегрируются с [хранилищем Azure](../storage/index.yml) с помощью [триггеров и привязок](./functions-triggers-bindings.md). Интеграция с хранилищем таблиц позволяет создавать функции, считывающие и записывающие данные хранилища таблиц.

| Действие | Type |
|---------|---------|
| Чтение данных хранилища таблиц в функции | [Входная привязка](./functions-bindings-storage-table-input.md) |
| Разрешение функции записи данных хранилища таблиц |[Выходная привязка](./functions-bindings-storage-table-output.md) |

## <a name="packages---functions-2x-and-higher"></a>Packages — функции 2. x и более поздних версий

Привязки хранилища таблиц доступны в пакете NuGet [Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage) версии 3.х. Исходный код для пакета находится в репозитории GitHub [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Tables).

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

## <a name="packages---functions-1x"></a>Пакеты – Функции 1.x

Привязки службы "Хранилище таблиц" доступны в пакете NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) версии 2.х. Исходный код для пакета находится в репозитории GitHub [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk/tree/v2.x/src/Microsoft.Azure.WebJobs.Storage/Table).

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

[!INCLUDE [functions-storage-sdk-version](../../includes/functions-storage-sdk-version.md)]

## <a name="next-steps"></a>Дальнейшие действия

- [Чтение данных хранилища таблиц при выполнении функции](./functions-bindings-storage-table-input.md)
- [Запись данных хранилища таблиц из функции](./functions-bindings-storage-table-output.md)
