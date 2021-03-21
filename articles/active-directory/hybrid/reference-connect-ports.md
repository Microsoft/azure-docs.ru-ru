---
title: Порты и протоколы, необходимые для гибридной идентификации в Azure | Документация Майкрософт
description: Эта страница является страницей технического справочника по портам, которые должны быть открыты для Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: de97b225-ae06-4afc-b2ef-a72a3643255b
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 03/04/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 681d92ed162feff0510067556d7eeaa47b12d988
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520036"
---
# <a name="hybrid-identity-required-ports-and-protocols"></a>Порты и протоколы, необходимые для гибридной идентификации
Следующий документ представляет собой технический справочник по портам и протоколам, которые необходимы для реализации решения для гибридной идентификации. Используйте приведенный ниже рисунок и соответствующую таблицу.

![Что такое Azure AD Connect?](./media/reference-connect-ports/required3.png)

## <a name="table-1---azure-ad-connect-and-on-premises-ad"></a>Таблица 1. Azure AD Connect и локальная служба AD
В этой таблице описываются порты и протоколы, необходимые для взаимодействия между сервером Azure AD Connect и локальной службой AD.

| Протокол | порты; | Описание |
| --- | --- | --- |
| DNS |53 (TCP или UDP) |Поиски DNS в лесу назначения. |
| Kerberos |88 (TCP или UDP) |Проверка подлинности Kerberos для леса AD. |
| MS-RPC |135 (TCP) |Используется во время начальной настройки мастера Azure AD Connect, когда он выполняет привязку к лесу AD, а также при синхронизации паролей. |
| LDAP |389 (TCP или UDP) |Используется для импорта данных из AD. Данные шифруются с помощью функции подписи и запечатывания Kerberos. |
| SMB | 445 (TCP) |Используется функцией простого единого входа для создания учетной записи компьютера в лесу AD. |
| LDAP или SSL |636 (TCP или UDP) |Используется для импорта данных из AD. Передача данных подписывается и шифруется. Используется только при использовании TLS. |
| RPC |49152-65535 (случайный старший порт RPC) (TCP) |Используется во время начальной настройки мастера Azure AD Connect, когда он выполняет привязку к лесам AD, а также во время синхронизации паролей. Если динамический порт был изменен, необходимо открыть этот порт. Дополнительную информацию см. в статьях базы знаний [KB929851](https://support.microsoft.com/kb/929851), [KB832017](https://support.microsoft.com/kb/832017) и [KB224196](https://support.microsoft.com/kb/224196). |
|WinRM  | 5985 (TCP) |Используется только при установке AD FS с помощью мастера Azure AD Connect gMSA|
|Веб-службы AD DS | 9389 (TCP) |Используется только при установке AD FS с помощью мастера Azure AD Connect gMSA |

## <a name="table-2---azure-ad-connect-and-azure-ad"></a>Таблица 2. Azure AD Connect и Azure AD
В этой таблице описываются порты и протоколы, которые необходимы для взаимодействия между сервером Azure AD Connect и Azure AD.

| Протокол | порты; | Описание |
| --- | --- | --- |
| HTTP |80 (TCP) |Используется для загрузки списков отзыва сертификатов (CRL) для проверки сертификатов TLS/SSL. |
| HTTPS |443 (TCP) |Используется для синхронизации с Azure AD. |

Список адресов URL и IP-адресов, которые необходимо открыть в брандмауэре, см. в разделе [URL-адреса и диапазоны IP-адресов Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) и [устранение неполадок Azure AD Connect подключения](tshoot-connect-connectivity.md#troubleshoot-connectivity-issues-in-the-installation-wizard).

## <a name="table-3---azure-ad-connect-and-ad-fs-federation-serverswap"></a>Таблица 3. Azure AD Connect и серверы федерации AD FS и WAP
В этой таблице описываются порты и протоколы, необходимые для взаимодействия между сервером Azure AD Connect и серверами федерации AD FS и WAP.  

| Протокол | порты; | Описание |
| --- | --- | --- |
| HTTP |80 (TCP) |Используется для загрузки списков отзыва сертификатов (CRL) для проверки сертификатов TLS/SSL. |
| HTTPS |443 (TCP) |Используется для синхронизации с Azure AD. |
| WinRM |5985 |Прослушиватель WinRM |

## <a name="table-4---wap-and-federation-servers"></a>Таблица 4. Серверы WAP и серверы федерации
В этой таблице описываются порты и протоколы, необходимые для взаимодействия между серверами федерации и серверами WAP.

| Протокол | порты; | Описание |
| --- | --- | --- |
| HTTPS |443 (TCP) |Используется для проверки подлинности. |

## <a name="table-5---wap-and-users"></a>Таблица 5. WAP и пользователи
В этой таблице описываются порты и протоколы, необходимые для взаимодействия между пользователями и серверами WAP.

| Протокол | порты; | Описание |
| --- | --- | --- |
| HTTPS |443 (TCP) |Используется для проверки подлинности устройств. |
| TCP |49443 (TCP) |Используется для проверки подлинности с помощью сертификата. |

## <a name="table-6a--6b---pass-through-authentication-with-single-sign-on-sso-and-password-hash-sync-with-single-sign-on-sso"></a>Таблицы 6a и 6b. Сквозная аутентификация с помощью единого входа (SSO) и синхронизация хэша паролей с помощью единого входа (SSO)
В следующих таблицах описываются порты и протоколы, которые необходимы для взаимодействия между Azure AD Connect и Azure AD.

### <a name="table-6a---pass-through-authentication-with-sso"></a>Таблица 6a. Сквозная аутентификация с помощью SSO
|Протокол|Номер порта|Описание
| --- | --- | ---
|HTTP|80|Разрешение исходящего трафика HTTP для проверки безопасности, например SSL. Оно также требуется для правильной работы автоматического обновления соединителя.
|HTTPS|443| Исходящий трафик HTTPS необходимо включить для выполнения таких операций, как включение и отключение функций, регистрация соединителей, скачивание обновлений соединителя и обработка всех запросов пользователя на вход.

Кроме того, Azure AD Connect требуется возможность устанавливать прямые IP-подключения к [диапазонам IP-адресов центра обработки данных Azure](https://www.microsoft.com/download/details.aspx?id=41653).

### <a name="table-6b---password-hash-sync-with-sso"></a>Таблица 6b. Синхронизация хэша паролей с помощью SSO

|Протокол|Номер порта|Описание
| --- | --- | ---
|HTTPS|443| Включение регистрации SSO (требуется только для процесса регистрации SSO).

Кроме того, Azure AD Connect требуется возможность устанавливать прямые IP-подключения к [диапазонам IP-адресов центра обработки данных Azure](https://www.microsoft.com/download/details.aspx?id=41653). Опять же, это необходимо только для процесса регистрации единого входа.

## <a name="table-7a--7b---azure-ad-connect-health-agent-for-ad-fssync-and-azure-ad"></a>Таблицы 7а и 7б. Агент Azure AD Connect Health для AD FS и синхронизации и Azure AD
В следующих таблицах описываются конечные точки, порты и протоколы, необходимые для взаимодействия между агентами Azure AD Connect Health и Azure AD.

### <a name="table-7a---ports-and-protocols-for-azure-ad-connect-health-agent-for-ad-fssync-and-azure-ad"></a>Таблица 7а. Порты и протоколы для агента Azure AD Connect Health для AD FS и синхронизации и Azure AD
В этой таблице описываются следующие исходящие порты и протоколы, необходимые для взаимодействия между агентами Azure AD Connect Health и Azure AD.  

| Протокол | порты; | Описание |
| --- | --- | --- |
| HTTPS |443 (TCP) |Исходящий трафик |
| Azure Service Bus |5671 (TCP) |Исходящий трафик |

Порт 5671 служебной шины Azure больше не требуется для последней версии агента. Последняя версия агента Azure AD Connect Health только требуемый порт 443.

### <a name="7b---endpoints-for-azure-ad-connect-health-agent-for-ad-fssync-and-azure-ad"></a>Таблица 7б. Конечные точки для агента Azure AD Connect Health для AD FS и синхронизации и Azure AD
Список конечных точек см. в [разделе "Требования" для агента Azure AD Connect Health](how-to-connect-health-agent-install.md#requirements).

