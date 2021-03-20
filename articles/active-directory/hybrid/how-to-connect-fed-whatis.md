---
title: Azure AD Connect и федерация | Документация Майкрософт
description: На этой странице представлена вся документация об операциях служб федерации Active Directory с использованием Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: f9107cf5-0131-499a-9edf-616bf3afef4d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/09/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: a97142e0c512f4f95235ad08c94c852906d3efd8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92095862"
---
# <a name="azure-ad-connect-and-federation"></a>Azure AD Connect и федерация
Azure Active Directory (Azure AD) Connect позволяет настроить федерацию с локальными службами федерации Active Directory (AD FS) и Azure AD. С помощью федеративного входа ваши пользователи могут входить в службы Azure AD со своими локальными паролями, а в корпоративной сети им не нужно будет вводить пароли повторно. Благодаря возможности федерации с AD FS можно развернуть новую установку AD FS или указать существующую установку в ферме Windows Server 2012 R2.

В этой статье содержатся сведения о функциях службы Azure AD Connect, относящихся к федерации, и приводятся ссылки на все связанные статьи. Ссылки на Azure AD Connect см. в статье [интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).

## <a name="azure-ad-connect-federation-topics"></a>Статьи об Azure AD Connect и федерации
| Раздел | Содержание и целевая аудитория |
|:--- |:--- |
| **Параметры входа в Azure AD Connect** | |
| [Параметры входа в Azure AD Connect](plan-connect-user-signin.md) |Описание разных параметров входа пользователей и их влияние на работу пользователя в среде Azure. |
| **Установка AD FS с помощью Azure AD Connect** | |
| [Предварительные требования](how-to-connect-install-custom.md#ad-fs-configuration-prerequisites) |Предварительные требования для установки AD FS с помощью Azure AD Connect. |
| [Настройка федерации с AD FS](how-to-connect-install-custom.md#configuring-federation-with-ad-fs) |Установка новой фермы AD FS с помощью Azure AD Connect. |
| [Федерацию с Azure AD с помощью альтернативного имени пользователя](how-to-connect-fed-management.md#alternateid) | Настройка федерации с использованием альтернативного имени пользователя.  |
| **Изменение конфигурации AD FS** | |
| [Восстановление доверия](how-to-connect-fed-management.md#repairthetrust) |Исправьте текущее отношение доверия между локальными AD FS и Microsoft 365/Azure. |
| [Добавление сервера AD FS](how-to-connect-fed-management.md#addadfsserver) |Добавление к ферме AD FS дополнительных серверов AD FS после начальной установки. |
| [Добавление прокси-сервера веб-приложения AD FS](how-to-connect-fed-management.md#addwapserver) |Добавление к ферме AD FS дополнительного сервера прокси-службы веб-приложения (WAP) после начальной установки. |
| [Добавление нового федеративного домена](how-to-connect-fed-management.md#addfeddomain) |Добавление нового домена в федерацию с Azure AD. |
| [Обновление сертификата TLS/SSL](how-to-connect-fed-ssl-update.md)| Обновите сертификат TLS/SSL для AD FS фермы. |
| [Продлите сертификаты Федерации для Microsoft 365 и Azure AD](how-to-connect-fed-o365-certs.md)|Обновление сертификата Office 365 в Azure AD.|
| **Другая конфигурация федерации** | |
| [Федерация нескольких экземпляров Azure AD с одним экземпляром AD FS](how-to-connect-fed-single-adfs-multitenant-federation.md) | Федерация нескольких экземпляров Azure AD с одной фермой AD FS.| 
| [Добавление настраиваемого логотипа компании или иллюстрации](how-to-connect-fed-management.md#customlogo) |Персонализация входа в систему путем настройки логотипа, который будет отображаться на странице входа в AD FS. |
| [Добавление описания входа в систему](how-to-connect-fed-management.md#addsignindescription) |Изменение текста для описания входа на странице входа в AD FS. |
| [Изменение правил утверждений AD FS](how-to-connect-fed-management.md#modclaims) |Изменение или добавление правил утверждений в AD FS в соответствии с настройками синхронизации Azure AD Connect. |


## <a name="additional-resources"></a>Дополнительные ресурсы
* [Федерация нескольких экземпляров Azure AD с одним экземпляром AD FS](how-to-connect-fed-single-adfs-multitenant-federation.md)
* [Развертывание AD FS в Azure](/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs)
* [Развертывание AD FS высокого уровня доступности в нескольких регионах Azure с помощью диспетчера трафика Azure](/windows-server/identity/ad-fs/deployment/active-directory-adfs-in-azure-with-azure-traffic-manager)