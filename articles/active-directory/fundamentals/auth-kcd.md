---
title: Ограниченное делегирование Kerberos с Azure Active Directory
description: Рекомендации по архитектуре для достижения ограниченного делегирования Kerberos с Azure Active Directory.
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
ms.openlocfilehash: a1c7aa4d2300a6dee44da067b122fc7af97f7aa9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96172862"
---
# <a name="windows-authentication---kerberos-constrained-delegation-with-azure-active-directory"></a>Проверка подлинности Windows — ограниченное делегирование Kerberos с Azure Active Directory

Ограниченное делегирование Kerberos (KCD) обеспечивает ограниченное делегирование между ресурсами и основывается на именах участника-службы. Для этого требуется, чтобы администраторы домена создали делегирование и были ограничены одним доменом. KCD на основе ресурсов часто используется в качестве способа предоставления проверки подлинности Kerberos для веб-приложения, в котором пользователи находятся в нескольких доменах в Active Directory лесу.

Azure Active Directory Application Proxy может предоставлять единый вход и удаленный доступ к приложениям на основе KCD, которым требуется билет Kerberos для доступа и ограниченное делегирование Kerberos (KCD).

Вы включаете единый вход для локальных приложений KCD, использующих встроенную проверку подлинности Windows (IWA), предоставляя соединителям прокси приложения разрешение на олицетворение пользователей в Active Directory. Соединитель прокси приложения использует это разрешение для отправки и получения маркеров от имени пользователя.

## <a name="use-when"></a>Используется, если

Необходимо предоставить удаленный доступ, защитить с помощью предварительной проверки подлинности и обеспечить единый вход для локальных приложений IWA.

![Схема архитектуры](./media/authentication-patterns/kcd-auth.png)

## <a name="components-of-system"></a>Компоненты системы

* **Пользователь**: обращается к устаревшему приложению, обслуживаемому прокси приложения.

* **Веб-браузер**. компонент, с которым взаимодействует пользователь, чтобы получить доступ к внешнему URL-адресу приложения.

* **Azure AD**: проверяет подлинность пользователя. 

* **Служба прокси приложения**. действует как обратный прокси-сервер для отправки запроса от пользователя к локальному приложению. Он располагается в Azure AD. Прокси приложения также может применять любые политики условного доступа.

* **Соединитель прокси приложения**. установлен локально на серверах Windows для обеспечения возможности подключения к приложению. Возвращает ответ на Azure AD. Выполняет согласование KCD с Active Directory, олицетворяя пользователя, чтобы получить маркер Kerberos для приложения.

* **Active Directory**: отправляет маркер Kerberos для приложения в соединитель прокси приложения.

* **Устаревшие приложения**: приложения, которые получают запросы пользователей от прокси приложения. Устаревшие приложения возвращают ответ соединителю прокси приложения.

## <a name="implement-windows-authentication-kcd-with-azure-ad"></a>Реализация проверки подлинности Windows (KCD) с помощью Azure AD

* [Ограниченное делегирование Kerberos для поддержки единого входа в приложения с помощью прокси приложения](../manage-apps/application-proxy-configure-single-sign-on-with-kcd.md) 

* [Добавление локального приложения для удаленного доступа через прокси приложения в Azure Active Directory](../manage-apps/application-proxy-add-on-premises-application.md)

