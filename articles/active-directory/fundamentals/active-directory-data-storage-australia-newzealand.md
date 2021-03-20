---
title: Хранилище данных клиентов для клиентов в Австралии и новых Зеландия — Azure AD
description: Узнайте, где Azure Active Directory хранит данные, связанные с клиентами, для клиентов Австралии и Новой Зеландии.
services: active-directory
author: ajburnle
manager: daveba
ms.author: ajburnle
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 7/21/2020
ms.custom: it-pro, seodec18, references_regions
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4d7c37e64e4f1b339ae66fe3d9135b1874476eb3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94836977"
---
# <a name="customer-data-storage-for-australian-and-new-zealand-customers-in-azure-active-directory"></a>Хранилище данных клиентов для клиентов в Австралии и новых Зеландия в Azure Active Directory 

Azure Active Directory (Azure AD) хранит свои данные клиента в географическом расположении в зависимости от страны, которую вы указали при регистрации в Microsoft Online Service. Службы Microsoft Online Services включают Microsoft 365 и Azure. 

Сведения о расположении данных Azure AD и других служб Майкрософт см. в разделе где находятся [ваши данные?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) центра управления безопасностью Майкрософт.

С 26 февраля 2020 г. Корпорация Майкрософт приступила к хранению данных клиента Azure AD для новых клиентов с помощью адреса выставления счетов в Австралии или Новой Зеландии в пределах австралийских центров обработки данных. С 1 мая 2020 до 31 марта 2021 Корпорация Майкрософт перенесет существующие клиенты, у которых в австралийском центре обработки данных есть Австралийский или новый адрес для выставления счетов, без каких бы то ни было каких либо действий клиента. Процесс миграции не затрагивает время простоя клиентов и не влияет на функциональность клиента во время миграции.

Кроме того, некоторые функции Azure AD пока не поддерживают хранение данных клиентов в Австралии. Сведения о конкретных функциях см. в [карте данных Azure AD](https://msit.powerbi.com/view?r=eyJrIjoiYzEyZTc5OTgtNTdlZS00ZTVkLWExN2ItOTM0OWU4NjljOGVjIiwidCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsImMiOjV9). Например, Microsoft Azure AD многофакторная идентификация хранит данные клиентов в США и обрабатывает их глобально. См. [местонахождение данных и данные клиента для многофакторной идентификации Azure AD](../authentication/concept-mfa-data-residency.md).

> [!NOTE]
> Продукты, службы и сторонние приложения Майкрософт, которые интегрируются с Azure AD, имеют доступ к данным клиента. Оцените каждый продукт, службу и приложение, используемые для определения того, как данные клиента обрабатываются конкретным продуктом, службой и приложением, и должны ли они соответствовать требованиям к хранению данных вашей компании. Дополнительные сведения о местонахождении данных служб Майкрософт см. в разделе [Где находятся ваши данные?](https://www.microsoft.com/trustcenter/privacy/where-your-data-is-located) Центра управления безопасностью Майкрософт.