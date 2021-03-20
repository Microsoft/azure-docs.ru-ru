---
title: Общие сведения о протоколе проверки подлинности Azure Active Directory и синхронизации
description: Руководство по архитектуре для интеграции Azure AD с устаревшими протоколами проверки подлинности и шаблонами синхронизации
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: ab63bc5bd2819a239741da525eebb2404a47bbf9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92441204"
---
# <a name="azure-active-directory-integrations-with-authentication-and-synchronization-protocols"></a>Интеграция Azure Active Directory с протоколами проверки подлинности и синхронизации

Microsoft Azure Active Directory (Azure AD) обеспечивает интеграцию с множеством протоколов проверки подлинности и синхронизации. Интеграция аутентификации позволяет использовать Azure AD, а также функции безопасности и управления с минимальными изменениями в приложениях, использующих устаревшие методы проверки подлинности. Интеграция синхронизации позволяет синхронизировать данные пользователей и групп с Azure AD, а затем выполнять функции управления пользователями Azure AD. Некоторые шаблоны синхронизации также обеспечивают автоматическую подготовку.

## <a name="legacy-authentication-protocols"></a>Устаревшие протоколы аутентификации

В следующей таблице представлена проверка подлинности интеграции Azure AD с устаревшими протоколами проверки подлинности и их возможностями. Выберите имя протокола проверки подлинности для просмотра

* Подробное описание

* Сценарии использования

* Схема архитектуры

* Объяснение компонентов системы

* Ссылки для реализации интеграции

 

| Протокол аутентификации| Аутентификация| Авторизация| Многофакторная идентификация| Условный доступ |
| - |- | - | - | - |
| [Проверка подлинности на основе заголовков](auth-header-based.md)|![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Проверка подлинности LDAP](auth-ldap.md)| ![флажок](./media/authentication-patterns/check.png)| | |  |
| [Проверка подлинности OAuth 2.0](auth-oauth2.md)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Проверка подлинности OIDC](auth-oidc.md)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Проверка подлинности единого входа на основе пароля](auth-password-based-sso.md )| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Проверка подлинности RADIUS]( auth-radius.md)| ![флажок](./media/authentication-patterns/check.png)| | ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Службы шлюза удаленный рабочий стол](auth-remote-desktop-gateway.md)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Secure Shell (SSH)](auth-ssh.md) |  ![флажок](./media/authentication-patterns/check.png)| | ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Проверка подлинности SAML](auth-saml.md).| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |
| [Проверка подлинности Windows — ограниченное делегирование Kerberos](auth-kcd.md)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |


 
## <a name="synchronization-patterns"></a>Шаблоны синхронизации

В следующей таблице представлена интеграция Azure AD с шаблонами синхронизации и их возможностями. Выберите имя шаблона для просмотра

* Подробное описание

* Сценарии использования

* Схема архитектуры

* Объяснение компонентов системы

* Ссылки для реализации интеграции



| Шаблон синхронизации| Синхронизация каталогов| Подготовка пользователей |
| - | - | - |
| [Синхронизация каталогов](sync-directory.md)| ![флажок](./media/authentication-patterns/check.png)|  |
| [Синхронизация LDAP](sync-ldap.md)| ![флажок](./media/authentication-patterns/check.png)|  |
| [Синхронизация SCIM](sync-scim.md)| ![флажок](./media/authentication-patterns/check.png)| ![флажок](./media/authentication-patterns/check.png) |

