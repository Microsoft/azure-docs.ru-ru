---
title: Ограничения службы Azure AD B2C и ограничения
titleSuffix: Azure AD B2C
description: Справочник по ограничениям службы и ограничениям для службы Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 02/02/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 190d88e62069a34b61017a0079f75696d67f6c82
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99979918"
---
# <a name="azure-active-directory-b2c-service-limits-and-restrictions"></a>Ограничения службы Azure Active Directory B2C и ограничения

В этой статье содержатся ограничения на использование и другие ограничения службы для службы Azure Active Directory B2C (Azure AD B2C).

## <a name="end-userconsumption-related-limits"></a>Ограничения, связанные с конечным пользователем и потреблением

Следующие ограничения службы, связанные с конечным пользователем, применяются ко всем протоколам проверки подлинности и авторизации, поддерживаемым Azure AD B2C, включая SAML, Open ID Connect, OAuth2 и РОПК.

|Категория |Ограничение    |
|---------|---------|
|Число запросов на каждый IP-адрес для каждого Azure AD B2C клиента       |6000/5 мин          |
|Общее число запросов на Azure AD B2C клиент     |12000/мин          |

Количество запросов может изменяться в зависимости от числа операций чтения и записи каталога, происходящих во время Azure AD B2C пути взаимодействия пользователя. Например, простой путь входа, считывающий данные из каталога, состоит из одного запроса. Если при входе необходимо также обновить каталог, эта операция учитывается как дополнительный запрос.

## <a name="azure-ad-b2c-configuration-limits"></a>Ограничения конфигурации Azure AD B2C

В следующей таблице перечислены ограничения административной конфигурации в службе Azure AD B2C.

|Категория  |Ограничение  |
|---------|---------|
|Число областей на приложение        |1000          |
|Число [настраиваемых атрибутов](user-profile-attributes.md#extension-attributes)   на пользователя <sup>1</sup>       |100         |
|Число URL-адресов перенаправления для каждого приложения       |100         |
|Число URL-адресов выхода для каждого приложения        |1          |
|Ограничение строки на атрибут      |250 знаков          |
|Число клиентов B2C на подписку      |20         |
|Уровни [наследования](custom-policy-overview.md#inheritance-model) в пользовательских политиках     |10         |
|Число политик на Azure AD B2C клиента      |200          |
|Максимальный размер файла политики      |400 КБ          |

<sup>1</sup> см. также [ограничения службы Azure AD](../active-directory/enterprise-users/directory-service-limits-restrictions.md).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте о [руководстве по регулированию Microsoft Graph](/graph/throttling) 
- Сведения о [различиях в проверке для Azure AD B2C приложений](../active-directory/develop/supported-accounts-validation.md)













