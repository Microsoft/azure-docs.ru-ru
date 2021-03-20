---
title: Защищенный веб-API — обзор
titleSuffix: Microsoft identity platform
description: Узнайте, как создать защищенный веб-API (обзор).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 434c255f0b9634e0d17bad1627ddd92af9498bfe
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99582440"
---
# <a name="scenario-protected-web-api"></a>Сценарий: защищенный веб-API

В этом сценарии вы узнаете, как предоставить веб-API. Вы также узнаете, как защитить веб-API таким образом, чтобы доступ к нему могли получить только пользователи, прошедшие проверку подлинности.

Чтобы использовать веб-API, можно включить пользователей, прошедших проверку подлинности, с помощью рабочих и учебных учетных записей или включить личные учетные записи Майкрософт.

## <a name="specifics"></a>Особенности

Ниже приведены конкретные сведения, которые необходимо знать для защиты веб-API.

- Регистрация приложения должна предоставлять по крайней мере одну *область* или одну *роль приложения*.
  - Области предоставляются веб-API, которые вызываются от имени пользователя.
  - Роли приложений предоставляются веб-API, которые вызываются приложениями управляющей программы (которые вызывают ваш веб-API от своего имени).
- Если вы создаете новую регистрацию приложения веб-API, выберите [версию маркера доступа](reference-app-manifest.md#accesstokenacceptedversion-attribute) , которая принимает ваш веб-API `2` . Для устаревших веб-API допустимой версией токена может быть `null` , но это значение запрещает доступ только для организаций, а личные учетные записи Майкрософт (MSA) не поддерживаются.
- Конфигурация кода для веб-API должна проверять токен, используемый при вызове веб-API.
- Код в действиях контроллера должен проверять роли или области в маркере.

## <a name="recommended-reading"></a>Рекомендуемые материалы

[!INCLUDE [recommended-topics](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии — [Регистрация приложения](scenario-protected-web-api-app-registration.md).
