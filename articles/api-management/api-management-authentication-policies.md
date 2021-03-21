---
title: Политики аутентификации в службе управления API Azure | Документация Майкрософт
description: Сведения о политиках аутентификации, доступных для использования в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 061702a7-3a78-472b-a54a-f3b1e332490d
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/27/2021
ms.author: apimpm
ms.openlocfilehash: 22d2960801cac2222f868c384a55b4bf436bc75b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99492609"
---
# <a name="api-management-authentication-policies"></a>Политики аутентификации в службе управления API
В этой статье рассматриваются приведенные ниже политики управления API. Дополнительные сведения о добавлении и настройке политик см. в статье о [политиках в управлении API](./api-management-policies.md).

##  <a name="authentication-policies"></a><a name="AuthenticationPolicies"></a> Политики проверки подлинности

-   [Обычная проверка подлинности](api-management-authentication-policies.md#Basic) – обычная проверка подлинности внутренней службы.

-   [Аутентификация с помощью сертификата клиента](api-management-authentication-policies.md#ClientCertificate) – аутентификация внутренней службы с помощью сертификатов клиентов.

-   Проверка подлинности [с помощью управляемого удостоверения](api-management-authentication-policies.md#ManagedIdentity) — проверка подлинности с помощью [управляемого удостоверения](../active-directory/managed-identities-azure-resources/overview.md) для службы управления API.

##  <a name="authenticate-with-basic"></a><a name="Basic"></a> Обычная проверка подлинности
 Используйте политику `authentication-basic` для обычной проверки подлинности внутренней службы. Эта политика задает для заголовка авторизации HTTP значение, соответствующее учетным данным, предоставленным в политике.

### <a name="policy-statement"></a>Правило политики

```xml
<authentication-basic username="username" password="password" />
```

### <a name="example"></a>Пример

```xml
<authentication-basic username="testuser" password="testpassword" />
```

### <a name="elements"></a>Элементы

|Название|Описание|Обязательно|
|----------|-----------------|--------------|
|authentication-basic|Корневой элемент.|Да|

### <a name="attributes"></a>Атрибуты

|Имя|Описание|Обязательно|По умолчанию|
|----------|-----------------|--------------|-------------|
|username|Задает имя пользователя для обычных учетных данных.|Да|Н/Д|
|password|Задает пароль для обычных учетных данных.|Да|Н/Д|

### <a name="usage"></a>Использование
 Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).

-   **Разделы политики:** inbound.

-   **Области политики:** все области.

##  <a name="authenticate-with-client-certificate"></a><a name="ClientCertificate"></a> Аутентификация с помощью сертификата клиента
 Используйте `authentication-certificate` политику для аутентификации в серверной службе с помощью сертификата клиента. Сначала необходимо [установить сертификат в Управление API](./api-management-howto-mutual-certificates.md) и определить его отпечаток или идентификатор сертификата (имя ресурса). 

> [!CAUTION]
> Если сертификат ссылается на сертификат, хранящийся в Azure Key Vault, найдите его с помощью идентификатора сертификата. При вращении сертификата хранилища ключей его отпечаток в оснастке управления API изменится, и политика не будет разрешать новый сертификат, если он определен по отпечатку.

### <a name="policy-statement"></a>Правило политики

```xml
<authentication-certificate thumbprint="thumbprint" certificate-id="resource name"/>
```

### <a name="examples"></a>Примеры

В этом примере сертификат клиента идентифицируется по ИДЕНТИФИКАТОРу сертификата:

```xml  
<authentication-certificate certificate-id="544fe9ddf3b8f30fb490d90f" />  
``` 

В этом примере сертификат клиента определяется по его отпечатку:

```xml
<authentication-certificate thumbprint="CA06F56B258B7A0D4F2B05470939478651151984" />
```
В этом примере сертификат клиента задается в политике, а не извлекается из встроенного хранилища сертификатов.

```xml
<authentication-certificate body="@(context.Variables.GetValueOrDefault<byte[]>("byteCertificate"))" password="optional-certificate-password" />
```

### <a name="elements"></a>Элементы  
  
|Название|Описание|Обязательно|  
|----------|-----------------|--------------|  
|authentication-certificate|Корневой элемент.|Да|  
  
### <a name="attributes"></a>Атрибуты  
  
|Имя|Описание|Обязательно|По умолчанию|  
|----------|-----------------|--------------|-------------|  
|thumbprint|Отпечаток для сертификата клиента.|`thumbprint` `certificate-id` Должен присутствовать либо.|Н/Д|
|ИД сертификата|Имя ресурса сертификата.|`thumbprint` `certificate-id` Должен присутствовать либо.|Н/Д|
|текст|Сертификат клиента как массив байтов.|Нет|Н/Д|
|password|Пароль для сертификата клиента.|Используется, если сертификат, указанный в `body` , защищен паролем.|Н/Д|
  
### <a name="usage"></a>Использование  
 Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).  
  
-   **Разделы политики:** inbound.  
  
-   **Области политики:** все области.  

##  <a name="authenticate-with-managed-identity"></a><a name="ManagedIdentity"></a> Проверка подлинности с помощью управляемого удостоверения  
 Используйте `authentication-managed-identity` политику для аутентификации в серверной службе с помощью управляемого удостоверения. Эта политика по сути использует управляемое удостоверение для получения маркера доступа из Azure Active Directory для доступа к указанному ресурсу. После успешного получения маркера политика задаст значение маркера в `Authorization` заголовке с помощью `Bearer` схемы.

Для запроса маркера может использоваться удостоверение, назначенное системой, и любое из нескольких назначенных пользователю удостоверений. Если `client-id` значение не указано, предполагается, что назначено системой удостоверение. Если `client-id` переменная предоставлена, для этого назначенного пользователю удостоверения запрашивается маркер Azure Active Directory
  
### <a name="policy-statement"></a>Правило политики  
  
```xml  
<authentication-managed-identity resource="resource" client-id="clientid of user-assigned identity" output-token-variable-name="token-variable" ignore-error="true|false"/>  
```  
  
### <a name="example"></a>Пример  
#### <a name="use-managed-identity-to-authenticate-with-a-backend-service"></a>Использование управляемого удостоверения для проверки подлинности в серверной службе
```xml  
<authentication-managed-identity resource="https://graph.microsoft.com"/> 
```
```xml  
<authentication-managed-identity resource="https://management.azure.com/"/> <!--Azure Resource Manager-->
```
```xml  
<authentication-managed-identity resource="https://vault.azure.net"/> <!--Azure Key Vault-->
```
```xml  
<authentication-managed-identity resource="https://servicebus.azure.net/"/> <!--Azure Service Bus-->
```
```xml  
<authentication-managed-identity resource="https://storage.azure.com/"/> <!--Azure Blob Storage-->
```
```xml  
<authentication-managed-identity resource="https://database.windows.net/"/> <!--Azure SQL-->
```

```xml
<authentication-managed-identity resource="api://Client_id_of_Backend"/> <!--Your own Azure AD Application-->
```

#### <a name="use-managed-identity-and-set-header-manually"></a>Использование управляемого удостоверения и заголовка Set вручную

```xml
<authentication-managed-identity resource="api://Client_id_of_Backend"
   output-token-variable-name="msi-access-token" ignore-error="false" /> <!--Your own Azure AD Application-->
<set-header name="Authorization" exists-action="override">
   <value>@("Bearer " + (string)context.Variables["msi-access-token"])</value>
</set-header>
```

#### <a name="use-managed-identity-in-send-request-policy"></a>Использование управляемого удостоверения в политике отправки запросов
```xml  
<send-request mode="new" timeout="20" ignore-error="false">
    <set-url>https://example.com/</set-url>
    <set-method>GET</set-method>
    <authentication-managed-identity resource="ResourceID"/>
</send-request>
```

### <a name="elements"></a>Элементы  
  
|Название|Описание|Обязательно|  
|----------|-----------------|--------------|  
|Проверка подлинности — управляемое удостоверение |Корневой элемент.|Да|  
  
### <a name="attributes"></a>Атрибуты  
  
|Имя|Описание|Обязательно|По умолчанию|  
|----------|-----------------|--------------|-------------|  
|ресурс|Строка. Идентификатор приложения целевого веб-API (защищенный ресурс) в Azure Active Directory.|Да|Н/Д|
|client-id|Строка. Идентификатор приложения, которому назначено пользовательское удостоверение, в Azure Active Directory.|Нет|удостоверение, назначенное системой|
|Output-Token-переменная-имя|Строка. Имя переменной контекста, которая будет принимать значение токена в качестве типа объекта `string` . |Нет|Н/Д|  
|ignore-error|Логическое. Если задано значение `true` , конвейер политики продолжит выполняться, даже если маркер доступа не получен.|Нет|false|  
  
### <a name="usage"></a>Использование  
 Эта политика может использоваться в следующих [разделах](./api-management-howto-policies.md#sections) и [областях](./api-management-howto-policies.md#scopes).  
  
-   **Разделы политики:** inbound.  
  
-   **Области политики:** все области.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о работе с политиками см. в следующих статьях:

+ [Политики в управлении API](api-management-howto-policies.md)
+ [Преобразование API-интерфейсов](transform-api.md).
+ Полный перечень операторов политик и их параметров см. в [справочнике по политикам](./api-management-policies.md).
+ [Примеры политик](./policy-reference.md).