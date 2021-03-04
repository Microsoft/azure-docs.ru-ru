---
title: Создание веб-API, вызывающего веб-API | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать веб-API, который вызывает нисходящие веб-API (обзор).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/03/2021
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 376c61f6a5ba94492cac26950465c61e3d8fe4ed
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102038566"
---
# <a name="scenario-a-web-api-that-calls-web-apis"></a>Сценарий: веб-API, который вызывает веб-API

Узнайте, что нужно знать для создания веб-API, который вызывает веб-API.

## <a name="prerequisites"></a>Предварительные требования

Этот сценарий, в котором защищенный веб-API вызывает другие веб-API, строится на [сценарии: защищенный веб-API](scenario-protected-web-api-overview.md).

## <a name="overview"></a>Обзор

- Клиентское, настольное, мобильное или одностраничное приложение (не представленное в сопровождающей схеме) вызывает защищенный веб-API и предоставляет маркер носителя JSON Web Token (JWT) в своем HTTP-заголовке "Authorization".
- Защищенный веб-API проверяет маркер и использует метод библиотеки проверки подлинности Майкрософт (MSAL) `AcquireTokenOnBehalfOf` для запроса другого маркера из Azure Active Directory (Azure AD), чтобы защищенный веб-API мог вызвать второй веб-API или нисходящий веб-API от имени пользователя. `AcquireTokenOnBehalfOf` обновляет токен при необходимости.
![Схема веб-API, вызывающего веб-API](media/scenarios/web-api.svg)

## <a name="specifics"></a>Особенности

Часть регистрации приложения, связанная с разрешениями API, является классической. Конфигурация приложения включает использование потока OAuth 2,0 от имени пользователя для обмена токеном носителя JWT с маркером для подчиненного API. Этот маркер добавляется в кэш маркеров, где он доступен на контроллерах веб-API, и затем может получить маркер без уведомления для вызова интерфейсов API нисходящей связи.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии — [Регистрация приложения](scenario-web-api-call-api-app-registration.md).
