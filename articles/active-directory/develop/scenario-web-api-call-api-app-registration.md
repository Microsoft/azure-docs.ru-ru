---
title: Регистрация веб-API, вызывающего веб-API | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать веб-API, который вызывает нисходящие веб-API (регистрация приложений).
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
ms.openlocfilehash: af1047c5f890b1b88ae6d043a30704e84b8dc079
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99584320"
---
# <a name="a-web-api-that-calls-web-apis-app-registration"></a>Веб-API, который вызывает веб-API: регистрация приложения

Веб-API, который вызывает нисходящие веб-API, имеет ту же регистрацию, что и защищенный веб-API. Следуйте инструкциям в разделе [защищенный веб-API: регистрация приложения](scenario-protected-web-api-app-registration.md).

Так как веб-приложение теперь вызывает веб-API, оно становится конфиденциальным клиентским приложением. Вот почему требуются дополнительные сведения о регистрации: приложение должно обмениваться секретами (учетные данные клиента) с платформой Microsoft Identity.

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>Разрешения API

Веб-приложения вызывают интерфейсы API от имени пользователей, которым был получен маркер носителя. Веб-приложениям необходимо запросить делегированные разрешения. Дополнительные сведения см. [в статье Добавление разрешений для доступа к веб-API](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api).

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии: [Конфигурация кода приложения](scenario-web-api-call-api-app-configuration.md).
