---
title: Многофакторная идентификация Azure AD для вашей организации — Azure Active Directory
description: Сведения о доступных функциях многофакторной идентификации Azure AD для вашей организации в зависимости от модели лицензии
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 03/18/2020
ms.author: daveba
author: daveba
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 85e7f02f8cbda6218396bf4a9a4654a113b7817c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94836748"
---
# <a name="overview-of-azure-ad-multi-factor-authentication-for-your-organization"></a>Обзор многофакторной идентификации Azure AD для вашей организации

Существует несколько способов включить многофакторную идентификацию Azure AD для пользователей Azure Active Directory (AD) на основе лицензий, которыми владеет ваша организация. 

![Исследование сигналов и принудительное применение MFA при необходимости](./media/concept-fundamentals-mfa-get-started/verify-signals-and-perform-mfa-if-required.png)

Исходя из наших исследований, многофакторная идентификация (MFA) позволяет повысить безопасность вашей учетной запись более чем на 99,9 %.

Как включить MFA для организации (даже бесплатно), чтобы исключить организацию из статистики жертв злоумышленников?

## <a name="free-option"></a>Бесплатный вариант

Клиенты, которые используют бесплатные преимущества Azure AD, могут использовать [параметры безопасности по умолчанию](../fundamentals/concept-fundamentals-security-defaults.md), чтобы включить многофакторную проверку подлинности в своей среде.

## <a name="microsoft-365-business-e3-or-e5"></a>Microsoft 365 бизнес, E3 или E5

Для клиентов с Microsoft 365 есть два варианта:

* Многофакторная идентификация Azure AD включена или отключена для всех пользователей для всех событий входа. Невозможно включить многофакторную проверку подлинности только для подмножества пользователей или только в определенных сценариях. Управление осуществляется через портал Office 365. 
* Для повышения удобства работы пользователей выполните обновление до Azure AD Premium P1 или P2 и используйте условный доступ. Дополнительные сведения см. в статье Защита ресурсов Microsoft 365 с помощью многофакторной проверки подлинности.

## <a name="azure-ad-premium-p1"></a>Azure AD Premium P1

Для клиентов с Azure AD Premium P1 или аналогичными лицензиями, которые включают такие функции, как Enterprise Mobility + Security E3, Microsoft 365 F1 или Microsoft 365 E3: 

Используйте [условный доступ Azure AD](../authentication/tutorial-enable-azure-mfa.md), чтобы запрашивать у пользователей многофакторную проверку подлинности в определенных сценариях или событиях в соответствии с требованиями бизнеса.

## <a name="azure-ad-premium-p2"></a>Azure AD Premium P2

Для клиентов с Azure AD Premium P2 или аналогичными лицензиями, которые включают такие функции, как Enterprise Mobility + Security E5 или Microsoft 365 E5: 

Обеспечивает наивысший уровень безопасности и улучшает взаимодействие с пользователем. Добавляет [Условный доступ на основе рисков](../conditional-access/howto-conditional-access-policy-risk.md) к функциям Azure AD Premium P1, которые адаптируются к шаблонам пользователей и сокращают запросы многофакторной проверки подлинности.

## <a name="authentication-methods"></a>Методы проверки подлинности

| Метод | Параметры безопасности по умолчанию | Все остальные методы |
| --- | --- | --- |
| Уведомление в мобильном приложении | X | X |
| Код проверки из мобильного приложения или токен оборудования |   | X |
| SMS на телефон |   | X |
| Звонок на телефон |   | X |

## <a name="next-steps"></a>Дальнейшие действия

Чтобы приступить к работе, см. Руководство по [защите событий входа пользователей с помощью многофакторной идентификации Azure AD](../authentication/tutorial-enable-azure-mfa.md).

Дополнительные сведения о лицензировании см. в статье [функции и лицензии для многофакторной идентификации Azure AD](../authentication/concept-mfa-licensing.md).
