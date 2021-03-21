---
title: Регистрация веб-приложения, вызывающего веб-API | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как зарегистрировать веб-приложение, вызывающее веб-API
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
ms.openlocfilehash: bb9a1ca6c2c81e3b0d5dbeff06f4de012446cf79
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98756325"
---
# <a name="a-web-app-that-calls-web-apis-app-registration"></a>Веб-приложение, вызывающее веб-API: регистрация приложения

Веб-приложение, вызывающее веб-API, имеет ту же регистрацию, что и веб-приложение, которое подписывает пользователей. Поэтому следуйте инструкциям в [веб-приложении, которое входит в систему пользователей: регистрация приложения](scenario-web-app-sign-user-app-registration.md).

Однако поскольку веб-приложение теперь также вызывает веб-API, оно становится конфиденциальным клиентским приложением. Вот почему требуется еще одна регистрация. Приложение должно совместно использовать учетные данные клиента или *секреты* с платформой Microsoft Identity.

[!INCLUDE [Registration of client secrets](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>Разрешения API

Веб-приложения вызывают API от имени пользователя, выполнившего вход. Для этого они должны запросить *делегированные разрешения*. Дополнительные сведения см. [в разделе Добавление разрешений для доступа к веб-API](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api).

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии — [Конфигурация кода](scenario-web-app-call-api-app-configuration.md).
