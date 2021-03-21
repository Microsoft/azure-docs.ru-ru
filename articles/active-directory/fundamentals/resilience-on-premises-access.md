---
title: Устойчивость к сборке в доступе к приложениям с помощью прокси приложения
description: Рекомендации для архитекторов и ИТ-администраторов по использованию прокси приложения для устойчивого доступа к локальным приложениям
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 11/30/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8fed78d7d2250d749ced7fe343689df76329b60d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98724666"
---
# <a name="build-resilience-in-application-access-with-application-proxy"></a>Устойчивость к сборке в доступе к приложениям с помощью прокси приложения

Azure Active Directory Application Proxy позволяет пользователям получать доступ с удаленного клиента к приложениям в локальной среде. Прокси приложения содержит как службу прокси приложения в облаке, так и соединители прокси приложения, которые работают на локальном сервере. 

Пользователи обращаются к локальным ресурсам через URL-адрес, опубликованный через прокси приложения. Они перенаправляются на страницу входа в Azure AD. Затем служба прокси приложения в Azure AD отправляет маркер в соединитель прокси приложения в корпоративной сети, который передает маркер в локальную Active Directory прошедший проверку подлинности пользователь может получить доступ к локальному ресурсу. На приведенной ниже схеме [соединители](../manage-apps/application-proxy-connectors.md) отображаются в [группе соединителей](../manage-apps/application-proxy-connector-groups.md).

> [!IMPORTANT]
> При публикации приложений через прокси приложения необходимо реализовать [планирование ресурсов и обеспечить соответствующую избыточность соединителей прокси приложения](../manage-apps/application-proxy-connectors.md#capacity-planning).

![Схема архитектуры приложения y](./media/resilience-on-prem-access/admin-resilience-app-proxy.png))

## <a name="how-do-i-implement-application-proxy"></a>Разделы справки реализовать прокси приложения?

Сведения о реализации удаленного доступа с помощью Azure AD Application Proxy см. в следующих ресурсах.

* [Планирование развертывания прокси приложения](../manage-apps/application-proxy-deployment-plan.md)

* [Рекомендации по обеспечению высокого уровня доступности и балансировки нагрузки](../manage-apps/application-proxy-high-availability-load-balancing.md)

* [Настройка прокси-серверов](../manage-apps/application-proxy-configure-connectors-with-proxy-servers.md)

* [Проектирование отказоустойчивой стратегии управления доступом](../authentication/concept-resilient-controls.md)

## <a name="next-steps"></a>Дальнейшие действия
Ресурсы по устойчивости для администраторов и архитекторов
 
* [Устойчивость к сборке с помощью управления учетными данными](resilience-in-credentials.md)

* [Устойчивость сборки к состояниям устройств](resilience-with-device-states.md)

* [Устойчивость сборки с помощью оценки непрерывного доступа (автоматизированного конструирования)](resilience-with-continuous-access-evaluation.md)

* [Устойчивость к сборке при аутентификации внешних пользователей](resilience-b2b-authentication.md)

* [Устойчивость к сборке в гибридной проверке подлинности](resilience-in-hybrid.md)

Обеспечение устойчивости ресурсов для разработчиков

* [Создание устойчивости IAM в приложениях](resilience-app-development-overview.md)

* [Устойчивость сборок в CIAM Systems](resilience-b2c.md)