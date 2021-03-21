---
title: Проверка подлинности Azure Active Directory REST API
description: Используйте Azure Active Directory для проверки подлинности в конфигурации приложения Azure с помощью REST API
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: cbf05245768a663e324e9bb6e1ad422eeee3ab1a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96930523"
---
# <a name="azure-active-directory-authentication"></a>Проверка подлинности Azure Active Directory

Для проверки подлинности HTTP-запросов можно использовать `Bearer` схему проверки подлинности с маркером, полученным от Azure Active Directory (Azure AD). Эти запросы необходимо передавать по протоколу TLS.

## <a name="prerequisites"></a>Предварительные условия

Необходимо назначить участника, который будет использоваться для запроса маркера Azure AD, к одной из применимых [ролей конфигурации приложений Azure](./rest-api-authorization-azure-ad.md).

Предоставьте каждому запросу все заголовки HTTP, необходимые для проверки подлинности. Ниже приведены минимальные требования.

|  Заголовок запроса | Описание  |
| --------------- | ------------ |
| `Authorization` | Сведения о проверке подлинности, необходимые `Bearer` схеме. |

**Пример**.

```http
Host: {myconfig}.azconfig.io
Authorization: Bearer {{AadToken}}
```

## <a name="azure-ad-token-acquisition"></a>Получение маркера Azure AD

Перед получением маркера Azure AD необходимо определить, какой пользователь вы хотите проверить подлинность, какую аудиторию вы запрашиваете маркер и какую конечную точку Azure AD следует использовать.

### <a name="audience"></a>Аудитория

Запросите маркер Azure AD с соответствующей аудиторией. Для настройки приложения Azure используйте одну из следующих аудиторий. Аудитория также может называться *ресурсом* , для которого запрашивается маркер.

- {Конфигуратионсторенаме}. азконфиг. IO
- *. azconfig.io

> [!IMPORTANT]
> Когда запрошенная аудитория является `{configurationStoreName}.azconfig.io` , она должна точно соответствовать `Host` заголовку запроса (с учетом регистра), используемому для отправки запроса.

### <a name="azure-ad-authority"></a>Центр Azure AD

Центр Azure AD — это конечная точка, используемая для получения маркера Azure AD. Он имеет вид `https://login.microsoftonline.com/{tenantId}` . `{tenantId}`Сегмент относится к идентификатору клиента Azure AD, которому принадлежит пользователь или приложение, которое пытается пройти проверку подлинности.

### <a name="authentication-libraries"></a>Библиотеки проверки подлинности

Azure предоставляет набор библиотек, именуемых Azure Active Directory библиотеками проверки подлинности, чтобы упростить процесс получения маркера Azure AD. Azure создает эти библиотеки для нескольких языков. Дополнительные сведения см. в этой [документации](../active-directory/azuread-dev/active-directory-authentication-libraries.md).

## <a name="errors"></a>ошибки

Возможны следующие ошибки.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: HMAC-SHA256, Bearer
```

**Причина:** Вы не указали заголовок запроса авторизации с помощью `Bearer` схемы.

**Решение:** Укажите допустимый `Authorization` заголовок HTTP-запроса.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: HMAC-SHA256, Bearer error="invalid_token", error_description="Authorization token failed validation"
```

**Причина:** Маркер Azure AD недопустим.

**Решение:** Получите маркер Azure AD из центра Azure AD и убедитесь, что вы использовали нужную аудиторию.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: HMAC-SHA256, Bearer error="invalid_token", error_description="The access token is from the wrong issuer. It must match the AD tenant associated with the subscription to which the configuration store belongs. If you just transferred your subscription and see this error message, please try back later."
```

**Причина:** Маркер Azure AD недопустим.

**Решение:** Получение маркера Azure AD из центра Azure AD. Убедитесь, что клиент Azure AD связан с подпиской, к которой принадлежит хранилище конфигураций. Эта ошибка может возникнуть, если участник принадлежит нескольким клиентам Azure AD.