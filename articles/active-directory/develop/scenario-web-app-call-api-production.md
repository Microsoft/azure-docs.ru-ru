---
title: Переход к рабочей среде веб-приложения, которое вызывает веб-API | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как перейти к рабочей среде веб-приложения, которое вызывает веб-API.
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
ms.openlocfilehash: cf32274a49cb1b790e9d872efe36f2e1cb188d1d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104675946"
---
# <a name="a-web-app-that-calls-web-apis-move-to-production"></a>Веб-приложение, вызывающее веб-API: переместить в рабочую среду

Теперь, когда вы узнали, как получить маркер для вызова веб-API, ниже приведены некоторые моменты, которые следует учитывать при переносе приложения в рабочую среду.

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше, выполнив полное последовательное руководство по ASP.NET Core веб-приложениям. Из этого руководства вы узнаете:

- Показывает, как подписывать пользователей в нескольких аудиториях или в национальных облаках, а также с помощью удостоверений социальных сетей.
- Вызывает Microsoft Graph.
- Вызывает несколько API-интерфейсов Майкрософт.
- Обрабатывает последовательное согласие.
- Вызывает собственный веб-API.

[Руководство. Веб-приложение ASP.NET Core](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial#scope-of-this-tutorial)

<!--- Removing this diagram as it's already shown from the next step linked tutorial

![Tutorial overview](media/scenarios/aspnetcore-webapp-tutorial.svg)

--->
