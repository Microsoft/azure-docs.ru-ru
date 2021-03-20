---
title: Обновление с DirSync и Azure AD Sync | Документация Майкрософт
description: Узнайте, как обновить DirSync и Azure AD Sync до Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: bd68fb88-110b-4d76-978a-233e15590803
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: 53d9e2ea10aefc96c2d4d72ad0020704ba48709b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99539422"
---
# <a name="upgrade-windows-azure-active-directory-sync-and-azure-active-directory-sync"></a>Обновление Windows Azure Active Directory Sync и Azure Active Directory Sync
Azure AD Connect — лучший способ подключения локального каталога к Azure AD и Microsoft 365. Это отличное время для обновления до Azure AD Connect с Windows Azure Active Directory Sync (DirSync) или Azure AD Sync (AADSync), так как эти средства теперь устарели и больше не поддерживаются по состоянию на 13 апреля 2017 г.

Одно из устаревших средств проверки подлинности удостоверений было предназначено работы с одним лесом (DirSync), а другое — для работы с несколькими лесами и для продвинутых пользователей (Azure AD Sync). На смену этим средствам пришло единое решение, подходящее для любой ситуации: Azure AD Connect. Оно включает новые возможности, усовершенствования функций и поддержку новых сценариев. Чтобы сохранить возможность синхронизации локальных данных удостоверений с Azure AD и Microsoft 365, настоятельно рекомендуется выполнить обновление до Azure AD Connect. Корпорация Майкрософт не гарантирует, что эти устаревшие версии будут работать после 31 декабря 2017 года.

Последний выпуск DirSync вышел в июле 2014 года, а последний выпуск Azure AD Sync — в мае 2015 года.

## <a name="what-is-azure-ad-connect"></a>Что такое Azure AD Connect?
Azure AD Connect является преемником DirSync и Azure AD Sync. В нем сочетаются все сценарии, поддерживаемые этими двумя вариантами. Дополнительные сведения см. в статье [Интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).

## <a name="deprecation-schedule"></a>График устаревания
| Дата | Комментировать |
| --- | --- |
| 13 апреля 2016 г. |Windows Azure Active Directory Sync (DirSync) и Microsoft Azure Active Directory Sync (Azure AD Sync) объявлены устаревшими. |
| 13 апреля 2017 г. |Прекращение поддержки. Пользователи не смогут обратиться в службу поддержки, пока не перейдут на Azure AD Connect. |
|31 декабря 2017 г.|Azure AD больше не будет принимать подключения из Windows Azure Active Directory Sync (DirSync) и Microsoft Azure Active Directory Sync (Azure AD Sync).
|1 апреля, 2021| Синхронизация Windows Azure Active Directory ("DirSync") и синхронизация Microsoft Azure Active Directory ("Azure AD Sync") больше не будут работать |

## <a name="how-to-transition-to-azure-ad-connect"></a>Переход на Azure AD Connect
Если вы используете DirSync, то переход можно выполнить одним из двух способов: это может быть обновление на месте и параллельное развертывание. Обновление на месте рекомендуется для большинства клиентов при наличии последней версии операционной системы и менее чем 50 000 объектов. В других случаях рекомендуется выполнять параллельное развертывание, при котором конфигурация DirSync переносится на новый сервер под управлением Azure AD Connect.

| Решение | Сценарий |
| --- | --- |
| [Обновление из DirSync](how-to-dirsync-upgrade-get-started.md) |<li>Если есть уже работающий сервер DirSync.</li> |
| [Обновление из Azure AD Sync](how-to-upgrade-previous-version.md) |<li>При переходе с Azure AD Sync.</li> |

Обновление на месте для перехода с DirSync на Azure AD Connect демонстрируется в следующем видеоролике Channel 9:

> [!VIDEO https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Azure-Active-Directory-Connect-in-place-upgrade-from-legacy-tools/player]
>
>

## <a name="faq"></a>Вопросы и ответы
**Вопрос. я получил уведомление по электронной почте от команды Azure и (или) сообщения из центра сообщений Microsoft 365, но я использую Connect.**  
Уведомление было также отправлено клиентам, использующим Azure AD Connect с номером сборки 1.0.\*.0 (версии, предшествующие выпуску 1.1). Корпорация Майкрософт рекомендует пользователям обновлять Azure AD Connect до последних выпусков. Функция [автоматического обновления](how-to-connect-install-automatic-upgrade.md), представленная в версии 1.1, существенно упрощает эту задачу: благодаря ей у вас всегда будет установлена последняя версия Azure AD Connect.

**Вопрос. Перестанет ли DirSync или Azure AD Sync работать 13 апреля 2017 года?**  
Служба DirSync (Azure AD Sync) продолжит работу после 13 апреля 2017 года.  Но после 31 декабря 2017 г. служба Azure AD больше не будет принимать подключения из DirSync и Azure AD Sync. DirSync и Azure AD Sync больше не будут работать после 1 апреля 2021

**Вопрос. Какие версии DirSync можно обновить?**  
 Поддерживается обновление любой версии DirSync, которая используется в настоящее время. 

**Вопрос. Как насчет соединителя Azure AD для FIM/MIM?**  
Соединитель Azure AD для FIM или MIM устаревшим **не** объявлен. Проект **заморожен**, т. е. новые функции не добавляются, а ошибки не исправляются. Корпорация Майкрософт рекомендует использующим его клиентам перейти на Azure AD Connect. Настоятельно рекомендуем не запускать с его помощью никакие новые развертывания. В будущем соединитель будет объявлен устаревшим.

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md)
