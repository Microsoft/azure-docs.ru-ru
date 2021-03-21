---
title: Проблема входа в локальное приложение с помощью прокси приложения Azure AD | Документация Майкрософт
description: Устранение типичных проблем, препятствующих выполнению входа в локальное приложение, интегрированное с Azure AD, с использованием прокси приложения Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/21/2018
ms.author: kenwith
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9a92ce03b4a8ad241a21b29bbff4e7367d656fab
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99260000"
---
# <a name="problems-signing-in-to-an-on-premises-application-using-the-azure-ad-application-proxy"></a>Проблемы при входе в локальное приложение с помощью прокси приложения Azure AD

Если при входе в локальное приложение возникают проблемы, попробуйте выполнить приведенные ниже действия, чтобы устранить их.

## <a name="i-can-load-my-application-but-something-on-the-page-looks-broken"></a>Я могу загрузить свое приложение, но что-то на странице отображается некорректно

Приведенные ниже документы помогут устранить некоторые из наиболее распространенных проблем в этой категории.

  * [Я имею доступ к приложению, но его страница отображается неправильно](application-proxy-page-appearance-broken-problem.md)
  * [Я имею доступ к приложению, но оно слишком долго загружается](application-proxy-page-load-speed-problem.md)
  * [Я имею доступ к приложению, но ссылки на странице приложения не работают](application-proxy-page-links-broken-problem.md)

## <a name="im-having-a-connectivity-problem-my-application"></a>Возникла проблема с подключением моего приложения
  Приведенные ниже документы помогут устранить некоторые из наиболее распространенных проблем в этой категории.
  * [Я не знаю, какие порты нужно открыть для моего приложения](application-proxy-add-on-premises-application.md)
  * [Возникла проблема из-за того, что в группе соединителей для моего приложения нет работающих соединителей](application-proxy-connectivity-no-working-connector.md)

## <a name="im-having-a-problem-configuring-the-azure-ad-application-proxy-in-the-admin-portal"></a>Возникла проблема при настройке прокси приложения Azure AD на портале администрирования
  Приведенные ниже документы помогут устранить некоторые из наиболее распространенных проблем в этой категории.
  * [Возникли сложности при настройке прокси приложения для приложения](application-proxy-config-how-to.md)
  * [Я не знаю, как настроить единый вход на прокси приложения для моего приложения](application-proxy-config-sso-how-to.md)
  * [Возникла проблема при создании моего приложения на портале администрирования](application-proxy-config-problem.md)

## <a name="im-having-a-problem-setting-up-back-end-authentication-to-my-application"></a>Возникла проблема при настройке внутренней аутентификации в моем приложении
  Приведенные ниже документы помогут устранить некоторые из наиболее распространенных проблем в этой категории.
  * [Я не знаю, как настроить ограниченное делегирование Kerberos](application-proxy-back-end-kerberos-constrained-delegation-how-to.md)
  * [Я не знаю, как настроить мое приложение для использования PingAccess](./application-proxy-ping-access-publishing-guide.md)

## <a name="im-having-a-problem-when-signing-in-to-my-application"></a>Возникла проблема при входе в мое приложение
  Приведенные ниже документы помогут устранить некоторые из наиболее распространенных проблем в этой категории.
  * [Появляется сообщение об ошибке "Can't Access this Corporate Application" (Нет доступа к этому корпоративному приложению)](application-proxy-sign-in-bad-gateway-timeout-error.md)

## <a name="im-having-a-problem-with-the-application-proxy-agent-connector"></a>Возникла проблема с соединителем агента прокси приложения
  Приведенные ниже документы помогут устранить некоторые из наиболее распространенных проблем в этой категории.
  * [Возникла проблема с установкой соединителя агента Application Proxy](application-proxy-connector-installation-problem.md)

## <a name="next-steps"></a>Дальнейшие действия
[Как обеспечить безопасный удаленный доступ к локальным приложениям](application-proxy.md)