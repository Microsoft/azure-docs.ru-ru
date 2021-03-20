---
title: Общие политики условного доступа — Azure Active Directory
description: Часто используемые политики условного доступа для организаций
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 07/02/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: bdaaed72b85bc3c0196a8897be311efa5fb714e2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92366349"
---
# <a name="common-conditional-access-policies"></a>Распространенные политики условного доступа

[Параметры безопасности по умолчанию](../fundamentals/concept-fundamentals-security-defaults.md) прекрасно подходят для некоторых организаций, но многие организации требуют большей гибкости, чем они предлагают. Например, многим организациям требуется возможность исключить из политик условного доступа, требующих многофакторную проверку подлинности, определенные учетные записи, такие как учетные записи для аварийного доступа или администрирования с разделением. Для этих организаций распространенные политики, упоминаемые в этой статье, могут быть использованы.

![Политики условного доступа в портал Azure](./media/concept-conditional-access-policy-common/conditional-access-policies-azure-ad-listing.png)

## <a name="emergency-access-accounts"></a>Учетные записи для аварийного доступа

Дополнительные сведения об учетных записях для аварийного доступа и их важность можно найти в следующих статьях: 

* [Управление учетными записями для аварийного доступа в Azure AD](../roles/security-emergency-access.md)
* [Создание устойчивой стратегии управления доступом с помощью Azure Active Directory](../authentication/concept-resilient-controls.md)

## <a name="typical-policies-deployed-by-organizations"></a>Типичные политики, развернутые организациями

* [Блокировать устаревшую проверку подлинности](howto-conditional-access-policy-block-legacy.md)\*
* [Требовать MFA для администраторов](howto-conditional-access-policy-admin-mfa.md)\*
* [Требовать MFA для управления Azure](howto-conditional-access-policy-azure-management.md)\*
* [Требовать MFA для всех пользователей](howto-conditional-access-policy-all-users-mfa.md)\*

\* Эти четыре политики при совместной настройке, функции имитации, включенные [по умолчанию безопасности](../fundamentals/concept-fundamentals-security-defaults.md).

## <a name="additional-policies"></a>Дополнительные политики

* [Условный доступ на основе риска входа (требуется Azure AD Premium P2)](howto-conditional-access-policy-risk.md)
* [Условный доступ пользователя на основе рисков (требуется Azure AD Premium P2)](howto-conditional-access-policy-risk-user.md)
* [Запрос на доверенное расположение для регистрации MFA](howto-conditional-access-policy-registration.md)
* [Блокирование доступа по расположению](howto-conditional-access-policy-location.md)
* [Требование использовать соответствующее устройство](howto-conditional-access-policy-compliant-device.md)
* [Блокировать доступ, за исключением конкретных приложений](howto-conditional-access-policy-block-access.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Моделирование поведения входа с помощью средства What If условного доступа.](troubleshoot-conditional-access-what-if.md)

- [Используйте режим "только отчет" для условного доступа, чтобы определить влияние новых политик.](concept-conditional-access-report-only.md)
