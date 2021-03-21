---
title: REST API конфигурации приложений Azure — проверка подлинности
description: Справочные страницы по проверке подлинности с помощью REST API конфигурации приложений Azure
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 58fba309f4cf7e4fc4364d075500c02e0ed45259
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96932699"
---
# <a name="authentication"></a>Аутентификация

Все HTTP-запросы должны пройти проверку подлинности. Поддерживаются следующие схемы проверки подлинности.

## <a name="hmac"></a>Код проверки подлинности сообщений с помощью хэш-функций

При [проверке подлинности HMAC](./rest-api-authentication-hmac.md) для подписания полезных данных запроса используется созданный случайным образом секрет. Сведения о том, как разрешены запросы, использующие этот метод проверки подлинности, можно найти в разделе [авторизации HMAC](./rest-api-authorization-hmac.md) .

## <a name="azure-active-directory"></a>Azure Active Directory

[Проверка подлинности Azure Active Directory (Azure AD)](../active-directory/authentication/overview-authentication.md) использует токен носителя, полученный от Azure Active Directory для проверки подлинности запросов. Сведения о том, как разрешены запросы, использующие этот метод проверки подлинности, можно найти в разделе [авторизация Azure AD](./rest-api-authorization-azure-ad.md) .