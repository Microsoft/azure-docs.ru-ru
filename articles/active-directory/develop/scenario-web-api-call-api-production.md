---
title: Перемещение веб-API, вызывающего веб-API, в рабочую среду | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как переместить веб-API, который вызывает веб-API, в рабочую среду.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 0ab5baef925b7c8589dd7852b6ff8058d67ba745
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104675878"
---
# <a name="a-web-api-that-calls-web-apis-move-to-production"></a>Веб-API, который вызывает веб-API: переместить в рабочую среду

После получения маркера для вызова веб-API ниже приведены некоторые моменты, которые следует учитывать при переносе приложения в рабочую среду.

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знакомы с основами вызова веб-API из собственного веб-API, вы можете заинтересовать следующий учебник, в котором описывается код, используемый для создания защищенного веб-API, который вызывает веб-API.

| Образец | Платформа | Описание |
|--------|----------|-------------|
| [Active-Directory-aspnetcore-webapi-Tutorial-v2](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/tree/master/2.%20Web%20API%20now%20calls%20Microsoft%20Graph) глава 1 | ASP.NET Core веб-API, Desktop (WPF) | ASP.NET Core вызовы веб-API Microsoft Graph, которые вызываются из приложения WPF с помощью платформы Microsoft Identity. |
