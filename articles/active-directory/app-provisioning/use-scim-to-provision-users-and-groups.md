---
title: Руководство. Разработка конечной точки SCIM для подготовки пользователей к работе с приложениями из Azure AD
description: System for Cross-domain Identity Management (SCIM) стандартизирует автоматическую подготовку пользователей. В этом руководстве показано, как разработать конечную точку SCIM, интегрировать API SCIM с Azure Active Directory и автоматизировать подготовку пользователей и групп в облачных приложениях.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.custom: contperf-fy21q2
ms.openlocfilehash: 8d517aaa6121120399e09bfef8aa6dd36e745563
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105022948"
---
# <a name="tutorial-develop-and-plan-provisioning-for-a-scim-endpoint"></a>Руководство по разработке и подготовке плана для конечной точки SCIM

С помощью API управления пользователями System for Cross-domain Identity Management (SCIM) разработчик приложения может включить автоматическую подготовку пользователей и групп между своим приложением и Azure AD (AAD). В этой статье объясняется, как создать конечную точку SCIM и объединить ее со службой подготовки AAD. Спецификация SCIM предоставляет общую схему пользователя для подготовки. При использовании в сочетании со стандартами федерации, такими как SAML или OpenID Connect, SCIM предоставляет администраторам комплексное решение для управления доступом на основе стандартов.

![Подготовка из Azure AD для приложения с SCIM](media/use-scim-to-provision-users-and-groups/scim-provisioning-overview.png)

SCIM — это стандартизированное определение двух конечных точек: конечной точки `/Users` и конечной точки `/Groups`. Система SCIM использует общие команды REST для создания, обновления и удаления объектов, а также предварительно определенную схему для общих атрибутов, таких как имя группы, имя пользователя, имя, фамилия и адрес электронной почты. Приложения, которые предоставляют REST API SCIM 2.0, могут сократить или устранить трудности при работе с API управления пользователями. Например, любой совместимый клиент SCIM знает, как выполнить команду HTTP POST объекта JSON в конечную точку `/Users` для создания новой записи пользователя. Вместо того, чтобы использовать несколько других API для одних и тех же основных действий, приложения, которые соответствуют стандарту SCIM, могут мгновенно воспользоваться преимуществами существующих клиентов, средств и кода. 

Стандартная схема объектов пользователя и API-интерфейсы для управления, определенные в SCIM 2.0 (RFC [7642](https://tools.ietf.org/html/rfc7642), [7643](https://tools.ietf.org/html/rfc7643), [7644](https://tools.ietf.org/html/rfc7644)), упрощают интеграцию поставщиков удостоверений с приложениями. Разработчики приложений, которые создают конечную точку SCIM, могут интегрироваться с любым клиентом, совместимым с SCIM, без необходимости выполнять роботу по настройке.

Чтобы автоматизировать подготовку приложения, потребуется создать конечную точку SCIM и интегрировать ее с клиентом SCIM Azure AD. Воспользуйтесь приведенными ниже инструкциями, чтобы начать подготовку пользователей и групп в приложении. 
    
1. Проектирование схемы пользователей и групп

   Определите объекты и атрибуты приложения, а также то, как они будут сопоставляется со схемой пользователя и группы, поддерживаемой реализацией SCIM для AAD.

1. Понимание особенностей реализации SCIM для AAD.

   Составьте представление о том, как реализуется клиент SCIM AAD для моделирования обработки запросов и ответов при использовании протокола SCIM.

1. Создание конечной точки SCIM

   Для интеграции со службой подготовки AAD конечная точка должна быть совместима с SCIM 2.0. Дополнительно для создания конечной точки можно использовать библиотеки Microsoft Common Language Infrastructure (CLI) и примеры кода. Эти примеры предназначены только для справки и тестирования. Мы не рекомендуем применять их в качестве зависимостей для приложения, которое используется в рабочей среде.

1. Интеграция конечной точки SCIM с клиентом SCIM AAD. 

   Если ваша организация использует стороннее приложение для реализации профиля SCIM 2.0, поддерживаемого AAD, вы можете быстро автоматизировать подготовку, а также отмену подготовки пользователей и групп.

1. Публикация приложения в коллекции приложений AAD. 

   Сделайте так, чтобы клиенты могли легко обнаружить ваше приложение и легко настраивайте подготовку. 

![Действия по интеграции конечной точки SCIM с Azure AD](media/use-scim-to-provision-users-and-groups/process.png)

## <a name="design-your-user-and-group-schema"></a>Проектирование схемы пользователей и групп

Для создания пользователя или группы в каждом приложении требуются разные атрибуты. Начните интеграцию, определив необходимые объекты (пользователи, группы) и атрибуты (имя, менеджер, должность и т. д.), которые требуются вашему приложению. 

Стандарт SCIM определяет схему для управления пользователями и группами. 

Для **основной** схемы пользователя требуется только три атрибута (все остальные являются необязательными):

- `id` — определенный поставщиком службы идентификатор;
- `externalId` — определенный клиентом идентификатор;
- `meta` — метаданные *только для чтения*, поддерживаемые поставщиком службы.

Помимо **основной** схемы пользователя, стандарт SCIM определяет расширение пользователя **предприятия** с моделью для расширения схемы пользователя в соответствии с потребностями вашего приложения. 

Если, например, приложению требуется адрес электронной почты пользователя и сведения о его менеджере, используйте **основную** схему, чтобы получить адрес электронной почты пользователя, и схему пользователя **предприятия**, чтобы получить сведения о менеджере пользователя.

Чтобы спроектировать схему, выполните следующие действия:

1. Перечислите требующиеся приложению атрибуты и распределите их по следующим категориям: атрибуты, необходимые для аутентификации (например, имя для входа и адрес электронной почты), атрибуты, необходимые для управления жизненным циклом пользователя (например, состояние или активность), и все другие атрибуты, необходимые для работы приложения (например, менеджер, тег).

1. Проверьте, определены ли уже эти атрибуты в **основной** схеме пользователя или схеме пользователя **предприятия**. Если нет, необходимо определить для схемы пользователя расширение, которое охватывает отсутствующие атрибуты. В приведенном ниже примере можно увидеть расширение для пользователя, разрешающее подготовку пользователя `tag`.

1. Сопоставьте атрибуты SCIM с атрибутами пользователя в Azure AD. Если один из атрибутов, определенных в конечной точке SCIM, не имеет точного аналога в схеме пользователя Azure AD, попросите администратора клиента расширить свою схему или использовать атрибут расширения, как показано ниже для свойства `tags`.

|Обязательный атрибут приложения|Сопоставляемый атрибут SCIM|Сопоставляемый атрибут Azure AD|
|--|--|--|
|loginName|userName|userPrincipalName|
|firstName|name.givenName|givenName|
|lastName|name.familyName|surName|
|workMail|emails[type eq “work”].value|Mail|
|manager|manager|manager|
|tag|urn:ietf:params:scim:schemas:extension:2.0:CustomExtension:tag|extensionAttribute1|
|status|active|isSoftDeleted (вычисленное значение не хранится на пользователе)|

**Пример списка обязательных атрибутов**

```json
{
     "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User",
      "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User",
      "urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:User"],
     "userName":"bjensen@testuser.com",
     "id": "48af03ac28ad4fb88478",
     "externalId":"bjensen",
     "name":{
       "familyName":"Jensen",
       "givenName":"Barbara"
     },
     "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": {
     "Manager": "123456"
   },
     "urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:CustomAttribute:User": {
     "tag": "701984",
   },
   "meta": {
     "resourceType": "User",
     "created": "2010-01-23T04:56:22Z",
     "lastModified": "2011-05-13T04:42:34Z",
     "version": "W\/\"3694e05e9dff591\"",
     "location":
 "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646"
   }
}   
```
**Пример схемы, определяемой полезными данными JSON**

> [!NOTE]
> В дополнение к атрибутам, необходимым для приложения, представление JSON содержит обязательные атрибуты `id`, `externalId` и `meta`.

Полезно распределить атрибуты по категориям `/User` и `/Group`, чтобы сопоставлять все атрибуты пользователя по умолчанию в Azure AD с атрибутами RFC SCIM. См. ведения о том, [как настроить сопоставление атрибутов между Azure AD и вашей конечной точкой SCIM](customize-application-attributes.md).


| Пользователь Azure Active Directory | "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User" |
| --- | --- |
| IsSoftDeleted |active |
|department|urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|
| displayName |displayName |
|employeeId|urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|
| Facsimile-TelephoneNumber |phoneNumbers[type eq "fax"].value |
| givenName |name.givenName |
| jobTitle |title |
| mail |emails[type eq "work"].value |
| mailNickname |externalId |
| manager |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager |
| mobile |phoneNumbers[type eq "mobile"].value |
| postalCode |addresses[type eq "work"].postalCode |
| proxy-Addresses |emails[type eq "other"].Value |
| physical-Delivery-OfficeName |addresses[type eq "other"].Formatted |
| streetAddress |addresses[type eq "work"].streetAddress |
| surname |name.familyName |
| telephone-Number |phoneNumbers[type eq "work"].value |
| user-PrincipalName |userName |

**Пример списка атрибутов пользователей и групп**

| Группа Azure Active Directory | urn:ietf:params:scim:schemas:core:2.0:Group |
| --- | --- |
| displayName |displayName |
| mail |emails[type eq "work"].value |
| mailNickname |displayName |
| members |members |
| objectId |externalId |
| proxyAddresses |emails[type eq "other"].Value |

**Пример списка атрибутов групп**

> [!NOTE]
> Вам не требуется поддерживать и пользователей, и группы или все показанные здесь атрибуты. Это только справочные сведения о том, как атрибуты в Azure AD зачастую сопоставляются со свойствами в протоколе SCIM. 

В RFC SCIM определено несколько конечных точек. Вы можете начать с конечной точки `/User` и двигаться от нее дальше. 

|Конечная точка|Описание|
|--|--|
|/User|Выполнение операций CRUD с объектом пользователя.|
|/Group|Выполнение операций CRUD с объектом группы.|
|/ServiceProviderConfig|Содержит сведения о поддерживаемых функциях стандарта SCIM, например о поддерживаемых ресурсах и о методе проверки подлинности.|
|/ResourceTypes|Указывает метаданные каждого ресурса|
|/Schemas|Набор атрибутов, поддерживаемых каждым клиентом и поставщиком служб, может различаться. Один поставщик услуг может включать `name`, `title` и `emails`, а другой — `name`, `title` и `phoneNumbers`. Конечная точка schemas позволяет обнаруживать поддерживаемые атрибуты.|
|/Bulk|Групповые операции позволяют выполнять операции над большой коллекцией объектов ресурсов за одну операцию (например, обновление членства для большой группы).|

**Пример списка конечных точек**

> [!NOTE]
> Используйте конечную точку `/Schemas` для поддержки настраиваемых атрибутов или в случае частого изменения схемы, так как эта конечная точка позволяет клиенту автоматически получать самую актуальную схему. Для поддержки групп используйте конечную точку `/Bulk`.

## <a name="understand-the-aad-scim-implementation"></a>Понимание особенностей реализации SCIM для AAD

Этот раздел посвящен поддержке API управления пользователями SCIM 2.0. В частности, здесь описана реализация клиента AAD SCIM, а также моделирование обработки запросов и ответов при использовании протокола SCIM.

> [!IMPORTANT]
> Поведение реализации SCIM Azure AD в последний раз обновлялось 18 декабря 2018 г. Дополнительные сведения об изменениях см. в разделе, [посвященном соответствию требованиям протокола SCIM 2.0 службы подготовки пользователей Azure AD](application-provisioning-config-problem-scim-compatibility.md).

В рамках [спецификации протокола SCIM 2.0](http://www.simplecloud.info/#Specification) приложения должны соответствовать следующим требованиям:

|Требование|Справочные примечания (протокол SCIM)|
|-|-|
|Создание пользователей и (при необходимости) групп|[Раздел 3.3](https://tools.ietf.org/html/rfc7644#section-3.3)|
|Изменение пользователей или групп с использованием запросов PATCH|[Раздел 3.5.2](https://tools.ietf.org/html/rfc7644#section-3.5.2). Поддержка гарантирует, что группы и пользователи подготавливаются к работе эффективно.|
|Получение известного ресурса для созданного ранее пользователя или группы|[Раздел 3.4.1](https://tools.ietf.org/html/rfc7644#section-3.4.1)|
|Запрашивание объектов пользователей или групп|[Раздел 3.4.2](https://tools.ietf.org/html/rfc7644#section-3.4.2).  По умолчанию пользователи извлекаются по `id` и запрашиваются по `username` и `externalId`, а группы запрашиваются по `displayName`.|
|Запрашивание объекта пользователя по идентификатору и по менеджеру|Раздел 3.4.2|
|Запрашивание групп по идентификатору и по членам|Раздел 3.4.2|
|Применение фильтра [excludedAttributes=members](#get-group) при запрашивании ресурса группы|Раздел 3.4.2.5|
|Прием одного маркера носителя для аутентификации и авторизации AAD в приложении.||
|Обратимое удаление пользователя `active=false` и восстановление пользователя `active=true`|Объект пользователя должен возвращаться в запросе независимо от активности пользователя. Пользователь не должен возвращаться, только если он необратимо удален из приложения.|
|Поддержка конечной точки /Schemas|[Раздел 7.](https://tools.ietf.org/html/rfc7643#page-30) Для обнаружения дополнительных атрибутов будет использоваться конечная точка обнаружения схем.|

При реализации конечной точки SCIM для обеспечения совместимости с AAD воспользуйтесь приведенными ниже общими рекомендациями:

* `id` является обязательным свойством для всех ресурсов. Каждый ответ, возвращающий ресурс, должен гарантировать, что каждый ресурс имеет это свойство, за исключением `ListResponse` с нулевым числом членов.
* Ответ на запрос или запрос фильтра всегда должен быть `ListResponse`.
* Группы являются необязательными, но они поддерживаются только в том случае, если реализация SCIM поддерживает запросы **PATCH**.
* Не обязательно включать весь ресурс в ответ **PATCH**.
* В Microsoft AAD используются только следующие операторы: `eq`, `and`
* Не требуйте совпадения с учетом регистра для структурных элементов в SCIM, в частности для значений операций `op` **PATCH**, как определено в [разделе 3.5.2](https://tools.ietf.org/html/rfc7644#section-3.5.2). AAD выводит для `op` значения **Add**, **Replace** и **Remove**.
* Microsoft AAD выполняет запросы на выбор случайного пользователя и группы, чтобы убедиться, что конечная точка и учетные данные действительны. Кроме того, это делается в рамках потока **проверки подключения** на [портале Azure](https://portal.azure.com). 
* Атрибут, по которому могут запрашиваться ресурсы, следует задать в приложении на [портале Azure](https://portal.azure.com) в качестве атрибута сопоставления. См. статью о [настройке сопоставлений атрибутов подготовки пользователей](customize-application-attributes.md).
* Поддержка HTTPS в конечной точке SCIM
* [Обнаружение схем:](#schema-discovery)
  * Сейчас пользовательское приложение не поддерживает обнаружение схем, но оно используется в определенных приложениях коллекции. В дальнейшем обнаружение схем будет использоваться в качестве основного метода для добавления дополнительных атрибутов к соединителю. 
  * Если значение отсутствует, не отправляйте значения NULL.
  * Значения свойств нужно задавать в "верблюжьем" стиле (например, readWrite).
  * В ответе должен возвращаться список.
  
### <a name="user-provisioning-and-deprovisioning"></a>Подготовка и отмена подготовки пользователей

На приведенном ниже изображении показаны сообщения, которые AAD отправляет в службу SCIM для управления жизненным циклом пользователя в хранилище удостоверений вашего приложения.  

![Показывает последовательность действий при подготовке и отмене подготовки пользователя](media/use-scim-to-provision-users-and-groups/scim-figure-4.png)<br/>
*Последовательность действий при подготовке и отмене подготовки пользователя*

### <a name="group-provisioning-and-deprovisioning"></a>Подготовка и отмена подготовки группы

Подготовка и отмена подготовки групп являются необязательными. На приведенном ниже изображении показаны сообщения, которые AAD отправляет в службу SCIM для управления жизненным циклом группы в хранилище удостоверений вашего приложения при включении и реализации. Эти сообщения отличаются от сообщений о пользователях по двум причинам.

* Запросы на получение групп указывают, что атрибут членов должен быть исключен из любого ресурса, предоставленного в ответ на запрос.  
* Запросы, которые должны определить, имеет ли ссылочный атрибут указанное значение, применяются в отношении атрибута members.  

![Показывает последовательность действий при подготовке и отмене подготовки группы](media/use-scim-to-provision-users-and-groups/scim-figure-5.png)<br/>
*Последовательность действий при подготовке и отмене подготовки группы*

### <a name="scim-protocol-requests-and-responses"></a>Запросы и ответы протокола SCIM
В этом разделе приведены примеры запросов SCIM, созданных клиентом SCIM AAD, и ожидаемых ответов. Чтобы получить наилучшие результаты, следует закодировать приложение, чтобы обрабатывать эти запросы в этом формате и выдавать ожидаемые ответы.

> [!IMPORTANT]
> Сведения о том, как и когда служба подготовки пользователей AAD выдает операции, описанные ниже, см. в разделе [Циклы подготовки: начальный и добавочный](how-provisioning-works.md#provisioning-cycles-initial-and-incremental) статьи [Описание процесса подготовки](how-provisioning-works.md).

[Операции пользователя](#user-operations)
  - [Создание пользователя](#create-user) ([Запрос](#request) / [Ответ](#response))
  - [Получение пользователя](#get-user) ([Запрос](#request-1) / [Ответ](#response-1))
  - [Получение пользователя по запросу](#get-user-by-query) ([Запрос](#request-2) / [Ответ](#response-2))
  - [Получение пользователя по запросу — нет результатов](#get-user-by-query---zero-results) ([Запрос](#request-3) / [Ответ](#response-3))
  - [Обновление пользователя [Многозначные свойства]](#update-user-multi-valued-properties) ([Запрос](#request-4) / [Ответ](#response-4))
  - [Обновление пользователя [Однозначные свойства]](#update-user-single-valued-properties) ([Запрос](#request-5) / [Ответ](#response-5)) 
  - [Отключение пользователя](#disable-user) ([Запрос](#request-14) / [Ответ](#response-14))
  - [Удаление пользователя](#delete-user) ([Запрос](#request-6) / [Ответ](#response-6))

[Операции с группой](#group-operations)
  - [Создание группы](#create-group) ([Запрос](#request-7) / [Ответ](#response-7))
  - [Получение группы](#get-group) ([Запрос](#request-8) / [Ответ](#response-8))
  - [Получение группы по displayName](#get-group-by-displayname) ([Запрос](#request-9) / [Ответ](#response-9))
  - [Обновление группы [атрибуты, не относящиеся к элементу]](#update-group-non-member-attributes) ([Запрос](#request-10)  / [Ответ](#response-10))
  - [Обновление группы [добавить элементы]](#update-group-add-members) ([Запрос](#request-11)  / [Ответ](#response-11))
  - [Обновление группы [удалить элементы]](#update-group-remove-members) ([Запрос](#request-12)  / [Ответ](#response-12))
  - [Удаление группы](#delete-group) ([Запрос](#request-13) / [Ответ](#response-13))

[Обнаружение схем:](#schema-discovery)
  - [обнаружение схемы](#discover-schema) ([запрос](#request-15) / [ответ](#response-15)).

### <a name="user-operations"></a>Операции пользователя

* Пользователей можно запрашивать с помощью атрибутов `userName` или `email[type eq "work"]`.  

#### <a name="create-user"></a>Создать пользователя

##### <a name="request"></a>Запрос

*POST /Users*
```json
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"],
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "active": true,
    "emails": [{
        "primary": true,
        "type": "work",
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com"
    }],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName"
    },
    "roles": []
}
```

##### <a name="response"></a>Ответ

*HTTP/1.1 201 Created*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "48af03ac28ad4fb88478",
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="get-user"></a>Получить пользователя

###### <a name="request"></a><a name="request-1"></a>Запрос
*GET /Users/5d48a0a8e9f04aa38008* 

###### <a name="response-user-found"></a><a name="response-1"></a>Ответ (найден пользователь)
*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5d48a0a8e9f04aa38008",
    "externalId": "58342554-38d6-4ec8-948c-50044d0a33fd",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_feed3ace-693c-4e5a-82e2-694be1b39934",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_22370c1a-9012-42b2-bf64-86099c2a1c22@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

###### <a name="request"></a>Запрос
*GET /Users/5171a35d82074e068ce2* 

###### <a name="response-user-not-found-note-that-the-detail-is-not-required-only-status"></a>Ответ (Пользователь не найден. Обратите внимание, что детали не являются обязательными, только статус.)

```json
{
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:Error"
    ],
    "status": "404",
    "detail": "Resource 23B51B0E5D7AE9110A49411D@7cca31655d49f3640a494224 not found"
}
```

#### <a name="get-user-by-query"></a>Получение пользователя по запросу

##### <a name="request"></a><a name="request-2"></a>Запрос

*GET /Users?filter=userName eq "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081"*

##### <a name="response"></a><a name="response-2"></a>Ответ

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "id": "2441309d85324e7793ae",
        "externalId": "7fce0092-d52e-4f76-b727-3955bd72c939",
        "meta": {
            "resourceType": "User",
            "created": "2018-03-27T19:59:26.000Z",
            "lastModified": "2018-03-27T19:59:26.000Z"
            
        },
        "userName": "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081",
        "name": {
            "familyName": "familyName",
            "givenName": "givenName"
        },
        "active": true,
        "emails": [{
            "value": "Test_User_91b67701-697b-46de-b864-bd0bbe4f99c1@testuser.com",
            "type": "work",
            "primary": true
        }]
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}
```

#### <a name="get-user-by-query---zero-results"></a>Получение пользователя по запросу — 0 результатов

##### <a name="request"></a><a name="request-3"></a>Запрос

*GET /Users?filter=userName eq "non-existent user"*

##### <a name="response"></a><a name="response-3"></a>Ответ

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 0,
    "Resources": [],
    "startIndex": 1,
    "itemsPerPage": 20
}
```

#### <a name="update-user-multi-valued-properties"></a>Обновление пользователя [Многозначные свойства]

##### <a name="request"></a><a name="request-4"></a>Запрос

*PATCH /Users/6764549bef60420686bc HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
            {
            "op": "Replace",
            "path": "emails[type eq \"work\"].value",
            "value": "updatedEmail@microsoft.com"
            },
            {
            "op": "Replace",
            "path": "name.familyName",
            "value": "updatedFamilyName"
            }
    ]
}
```

##### <a name="response"></a><a name="response-4"></a>Ответ

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "6764549bef60420686bc",
    "externalId": "6c75de36-30fa-4d2d-a196-6bdcdb6b6539",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_fbb9dda4-fcde-4f98-a68b-6c5599e17c27",
    "name": {
        "formatted": "givenName updatedFamilyName",
        "familyName": "updatedFamilyName",
        "givenName": "givenName"
    },
    "active": true,
    "emails": [{
        "value": "updatedEmail@microsoft.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="update-user-single-valued-properties"></a>Обновить пользователя [Свойства с одним значением]

##### <a name="request"></a><a name="request-5"></a>Запрос

*PATCH /Users/5171a35d82074e068ce2 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "userName",
        "value": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com"
    }]
}
```

##### <a name="response"></a><a name="response-5"></a>Ответ

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5171a35d82074e068ce2",
    "externalId": "aa1eca08-7179-4eeb-a0be-a519f7e5cd1a",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "userName": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_49dc1090-aada-4657-8434-4995c25a00f7@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

### <a name="disable-user"></a>Отключение пользователя

##### <a name="request"></a><a name="request-14"></a>Запрос

*PATCH /Users/5171a35d82074e068ce2 HTTP/1.1*
```json
{
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": false
        }
    ],
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ]
}
```

##### <a name="response"></a><a name="response-14"></a>Ответ

```json
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User"
    ],
    "id": "CEC50F275D83C4530A495FCF@834d0e1e5d8235f90a495fda",
    "userName": "deanruiz@testuser.com",
    "name": {
        "familyName": "Harris",
        "givenName": "Larry"
    },
    "active": false,
    "emails": [
        {
            "value": "gloversuzanne@testuser.com",
            "type": "work",
            "primary": true
        }
    ],
    "addresses": [
        {
            "country": "ML",
            "type": "work",
            "primary": true
        }
    ],
    "meta": {
        "resourceType": "Users",
        "location": "/scim/5171a35d82074e068ce2/Users/CEC50F265D83B4530B495FCF@5171a35d82074e068ce2"
    }
}
```
#### <a name="delete-user"></a>Удаление пользователя

##### <a name="request"></a><a name="request-6"></a>Запрос

*DELETE /Users/5171a35d82074e068ce2 HTTP/1.1*

##### <a name="response"></a><a name="response-6"></a>Ответ

*HTTP/1.1 204 No Content*

### <a name="group-operations"></a>Операции с группой

* Группы всегда следует создавать с пустым списком членов.
* Группы можно запрашивать с помощью атрибута `displayName`.
* Обновление запроса группы исправления в ответ должно приостановить *HTTP 204 No Content*. Не рекомендуется возвращать текст со списком всех членов.
* Поддержка возвращения всех членов группы не нужна.

#### <a name="create-group"></a>Создание группы

##### <a name="request"></a><a name="request-7"></a>Запрос

*POST /Groups HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group", "http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/2.0/Group"],
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "displayName": "displayName",
    "meta": {
        "resourceType": "Group"
    }
}
```

##### <a name="response"></a><a name="response-7"></a>Ответ

*HTTP/1.1 201 Created*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "927fa2c08dcb4a7fae9e",
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "displayName": "displayName",
    "members": []
}
```

#### <a name="get-group"></a>Получить группу

##### <a name="request"></a><a name="request-8"></a>Запрос

*GET /Groups/40734ae655284ad3abcc?excludedAttributes=members HTTP/1.1*

##### <a name="response"></a><a name="response-8"></a>Ответ
*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "40734ae655284ad3abcc",
    "externalId": "60f1bb27-2e1e-402d-bcc4-ec999564a194",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "displayName": "displayName",
}
```

#### <a name="get-group-by-displayname"></a>Получить группу по displayName

##### <a name="request"></a><a name="request-9"></a>Запрос
*GET /Groups?excludedAttributes=members&filter=displayName eq "displayName" HTTP/1.1*

##### <a name="response"></a><a name="response-9"></a>Ответ

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "id": "8c601452cc934a9ebef9",
        "externalId": "0db508eb-91e2-46e4-809c-30dcbda0c685",
        "meta": {
            "resourceType": "Group",
            "created": "2018-03-27T22:02:32.000Z",
            "lastModified": "2018-03-27T22:02:32.000Z",
            
        },
        "displayName": "displayName",
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}
```

#### <a name="update-group-non-member-attributes"></a>Обновление группы [Нет атрибутов пользователей]

##### <a name="request"></a><a name="request-10"></a>Запрос

*PATCH /Groups/fa2ce26709934589afc5 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "displayName",
        "value": "1879db59-3bdf-4490-ad68-ab880a269474updatedDisplayName"
    }]
}
```

##### <a name="response"></a><a name="response-10"></a>Ответ

*HTTP/1.1 204 No Content*

### <a name="update-group-add-members"></a>Обновить группу [Добавить элементы]

##### <a name="request"></a><a name="request-11"></a>Запрос

*PATCH /Groups/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Add",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a><a name="response-11"></a>Ответ

*HTTP/1.1 204 No Content*

#### <a name="update-group-remove-members"></a>Обновление группы [Удаление элементов]

##### <a name="request"></a><a name="request-12"></a>Запрос

*PATCH /Groups/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Remove",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a><a name="response-12"></a>Ответ

*HTTP/1.1 204 No Content*

#### <a name="delete-group"></a>Удалить группу

##### <a name="request"></a><a name="request-13"></a>Запрос

*DELETE /Groups/cdb1ce18f65944079d37 HTTP/1.1*

##### <a name="response"></a><a name="response-13"></a>Ответ

*HTTP/1.1 204 No Content*

### <a name="schema-discovery"></a>Обнаружение схем
#### <a name="discover-schema"></a>Обнаружение схемы

##### <a name="request"></a><a name="request-15"></a>Запрос
*GET /Schemas* 
##### <a name="response"></a><a name="response-15"></a>Ответ
*HTTP/1.1 200 OK*
```json
{
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
    ],
    "itemsPerPage": 50,
    "startIndex": 1,
    "totalResults": 3,
    "Resources": [
  {
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Schema"],
    "id" : "urn:ietf:params:scim:schemas:core:2.0:User",
    "name" : "User",
    "description" : "User Account",
    "attributes" : [
      {
        "name" : "userName",
        "type" : "string",
        "multiValued" : false,
        "description" : "Unique identifier for the User, typically
used by the user to directly authenticate to the service provider.
Each User MUST include a non-empty userName value.  This identifier
MUST be unique across the service provider's entire set of Users.
REQUIRED.",
        "required" : true,
        "caseExact" : false,
        "mutability" : "readWrite",
        "returned" : "default",
        "uniqueness" : "server"
      },                
    ],
    "meta" : {
      "resourceType" : "Schema",
      "location" :
        "/v2/Schemas/urn:ietf:params:scim:schemas:core:2.0:User"
    }
  },
  {
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Schema"],
    "id" : "urn:ietf:params:scim:schemas:core:2.0:Group",
    "name" : "Group",
    "description" : "Group",
    "attributes" : [
      {
        "name" : "displayName",
        "type" : "string",
        "multiValued" : false,
        "description" : "A human-readable name for the Group.
REQUIRED.",
        "required" : false,
        "caseExact" : false,
        "mutability" : "readWrite",
        "returned" : "default",
        "uniqueness" : "none"
      },
    ],
    "meta" : {
      "resourceType" : "Schema",
      "location" :
        "/v2/Schemas/urn:ietf:params:scim:schemas:core:2.0:Group"
    }
  },
  {
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Schema"],
    "id" : "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User",
    "name" : "EnterpriseUser",
    "description" : "Enterprise User",
    "attributes" : [
      {
        "name" : "employeeNumber",
        "type" : "string",
        "multiValued" : false,
        "description" : "Numeric or alphanumeric identifier assigned
to a person, typically based on order of hire or association with an
organization.",
        "required" : false,
        "caseExact" : false,
        "mutability" : "readWrite",
        "returned" : "default",
        "uniqueness" : "none"
      },
    ],
    "meta" : {
      "resourceType" : "Schema",
      "location" :
"/v2/Schemas/urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
    }
  }
]
}
```

### <a name="security-requirements"></a>Требования безопасности
**Версии протокола TLS**

Единственные допустимые версии протокола TLS — TLS 1.2 и TLS 1.3. Другие версии TLS не разрешены. Запрещены все версии SSL. 
- Ключи RSA должны составлять не менее 2 048 бит.
- Ключи ECC должны иметь по крайней мере 256 бит, созданных с помощью допустимой эллиптической кривой

**Длина ключей**

Все службы должны использовать сертификаты X. 509, созданные с использованием криптографических ключей достаточной длины, то есть:

**Комплекты шифров**

Все службы нужно настроить так, чтобы они использовали следующие комплекты шифров в указанном ниже порядке. Обратите внимание, что если у вас есть только сертификат RSA, установленные наборы шифров ECDSA не оказывают никакого влияния. </br>

Минимальная строка наборов шифров TLS 1.2:

- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

### <a name="ip-ranges"></a>Диапазоны IP-адресов
Служба подготовки Azure AD в настоящее время работает с диапазонами IP-адресов для AzureActiveDirectory, как указано [здесь](https://www.microsoft.com/download/details.aspx?id=56519&WT.mc_id=rss_alldownloads_all). Вы можете добавить диапазоны IP-адресов, перечисленные в теге AzureActiveDirectory, чтобы разрешить трафик из службы подготовки Azure AD к приложению. Обратите внимание, что список диапазонов IP-адресов нужно тщательно проверить на предмет вычисляемых адресов. Например, адрес 40.126.25.32 может быть представлен в списке диапазонов IP-адресов как 40.126.0.0/18. Список диапазонов IP-адресов можно также получить программно, используя приведенный ниже [API](/rest/api/virtualnetwork/servicetags/list).

## <a name="build-a-scim-endpoint"></a>Создание конечной точки SCIM

Теперь, когда вы разработали схему и поняли как реализовать SCIM Azure AD, можно приступить к разработке конечной точки SCIM. Вместо того чтобы начинать с нуля и самостоятельно создавать реализацию, вы можете использовать ряд открытых библиотек SCIM, опубликованных сообществом SCIM.

Сведения о создании конечной точки SCIM, а также примеры см. в руководстве [Разработка примера конечной точки SCIM](use-scim-to-build-users-and-groups-endpoints.md).

[Пример кода ссылки](https://aka.ms/SCIMReferenceCode) .NET Core с открытым кодом, опубликованный командой подготовки Azure AD, — это один из ресурсов, с помощью которого можно начать разработку. После создания конечной точки SCIM, ее нужно будет протестировать. Вы можете использовать коллекцию [тестов postman](https://github.com/AzureAD/SCIMReferenceCode/wiki/Test-Your-SCIM-Endpoint), предоставленную в виде части кода ссылки, или запустить проверку с помощью примеров запросов/ответов, предоставленных [выше](#user-operations).  

   > [!Note]
   > Код ссылки предназначен для того, чтобы помочь приступить к созданию конечной точки SCIM и предоставлен "как есть". Вклады сообщества могут помочь в сборке и поддержании кода.

Решение состоит из двух проектов: _Microsoft. SCIM_ и _Microsoft.SCIM.WebHostSample_.

Проект _Microsoft. SCIM_ — это библиотека, которая определяет компоненты веб-службы, которые соответствуют характеристике SCIM. Он объявляет интерфейс _Microsoft. SCIM.IProvider_, а запросы преобразуются в вызовы за методами поставщика, запрограммированными для работы с хранилищем удостоверений.

![Разбивка: Запрос, преобразованный в вызовы методов поставщика:](media/use-scim-to-provision-users-and-groups/scim-figure-3.png)

Проект _Microsoft.SCIM.WebHostSample_ — это веб-приложение Visual Studio ASP.NET Core, основанное на _пустом_ шаблоне. Это позволяет развертывать пример кода изолированно, размещать его в контейнерах или в службах IIS. Он также реализует интерфейс _Microsoft. SCIM.IProvider_, хранящий классы в памяти в качестве примера хранилища удостоверений.

```csharp
    public class Startup
    {
        ...
        public IMonitor MonitoringBehavior { get; set; }
        public IProvider ProviderBehavior { get; set; }

        public Startup(IWebHostEnvironment env, IConfiguration configuration)
        {
            ...
            this.MonitoringBehavior = new ConsoleMonitor();
            this.ProviderBehavior = new InMemoryProvider();
        }
        ...
```

### <a name="building-a-custom-scim-endpoint"></a>Создание пользовательской конечной точки SCIM

Служба SCIM должна иметь HTTP-адрес и сертификат проверки подлинности сервера одного из следующих корневых центров сертификации:

* CNNIC;
* Comodo;
* CyberTrust;
* DigiCert;
* GeoTrust;
* GlobalSign;
* Go Daddy;
* VeriSign;
* WoSign.
* Корневой ЦС X3 DST

NET Core SDK включает сертификат разработки HTTPS, который можно использовать во время создания, сертификат устанавливается во время первого запуска. В зависимости от способа запуска веб-приложения ASP.NET Core, оно будет прослушивать другой порт:

* Microsoft.SCIM.WebHostSample: https://localhost:5001
* IIS Express: https://localhost:44359/

Дополнительные сведения об использовании HTTPS в ASP.NET Core см. в статье: [Принудительное применение HTTPS в ASP.NET Core](/aspnet/core/security/enforcing-ssl)

### <a name="handling-endpoint-authentication"></a>Обработка аутентификации на конечной точке

Запросы от Azure Active Directory содержат токен носителя OAuth 2.0. Любая веб-служба, которая получает запрос, должна аутентифицировать издателя как Azure Active Directory для ожидаемого клиента Azure Active Directory.

В токене издатель идентифицируется с помощью утверждения iss, например `"iss":"https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/"`. В этом примере базовый адрес утверждения со значением`https://sts.windows.net` подтверждает, что издателем является Azure Active Directory. Сегмент относительного адреса _cbb1a5ac-f33b-45fa-9bf5-f37db0fed422_ — это уникальный идентификатор клиента Azure Active Directory, для которого выдан токен.

Аудиторией для токена будет идентификатор шаблона приложения для приложения в коллекции. Каждое приложение, зарегистрированное в одном клиенте, может получать одно и то же утверждение `iss` с запросами SCIM. Идентификатор шаблона приложения для всех пользовательских приложений — _8adf8e6e-67b2-4cf2-A259-e3dc5476c621_. Токен, созданный службой подготовки Azure AD, следует использовать только для тестирования. Его нельзя использовать в рабочих средах.

В примере кода, запросы проходят аутентификацию с помощью пакета Microsoft.AspNetCore.Authentication.JwtBearer. Следующий код обеспечивает проверку подлинности запросов к любой из конечных точек службы с помощью маркера носителя, выданного Azure Active Directory для указанного клиента:

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            if (_env.IsDevelopment())
            {
                ...
            }
            else
            {
                services.AddAuthentication(options =>
                {
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                })
                    .AddJwtBearer(options =>
                    {
                        options.Authority = " https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/";
                        options.Audience = "8adf8e6e-67b2-4cf2-a259-e3dc5476c621";
                        ...
                    });
            }
            ...
        }

        public void Configure(IApplicationBuilder app)
        {
            ...
            app.UseAuthentication();
            app.UseAuthorization();
            ...
       }
```

Маркер носителя также необходим для использования предоставленных [тестов postman](https://github.com/AzureAD/SCIMReferenceCode/wiki/Test-Your-SCIM-Endpoint) и выполнения локальной отладки с помощью localhost. Пример кода использует среды ASP.NET Core для изменения параметров проверки подлинности на этапе разработки и включения использования самозаверяющего токена.

Дополнительные сведения см. в статье [Использование нескольких сред в ASP.NET Core](/aspnet/core/fundamentals/environments).

Следующий код обеспечивает проверку подлинности запросов к любой из конечных точек службы с помощью маркера носителя, подписанного ключом пользователя:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    if (_env.IsDevelopment())
    {
        services.AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        })
        .AddJwtBearer(options =>
        {
            options.TokenValidationParameters =
                new TokenValidationParameters
                {
                    ValidateIssuer = false,
                    ValidateAudience = false,
                    ValidateLifetime = false,
                    ValidateIssuerSigningKey = false,
                    ValidIssuer = "Microsoft.Security.Bearer",
                    ValidAudience = "Microsoft.Security.Bearer",
                    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("A1B2C3D4E5F6A1B2C3D4E5F6"))
                };
        });
    }
...
```

Отправьте запрос GET на контроллер токена, чтобы получить действительный маркер носителя. Метод _GenerateJSONWebToken_ отвечает за создание маркера, соответствующего параметрам, настроенным для разработки:

```csharp
private string GenerateJSONWebToken()
{
    // Create token key
    SymmetricSecurityKey securityKey =
        new SymmetricSecurityKey(Encoding.UTF8.GetBytes("A1B2C3D4E5F6A1B2C3D4E5F6"));
    SigningCredentials credentials =
        new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

    // Set token expiration
    DateTime startTime = DateTime.UtcNow;
    DateTime expiryTime = startTime.AddMinutes(120);

    // Generate the token
    JwtSecurityToken token =
        new JwtSecurityToken(
            "Microsoft.Security.Bearer",
            "Microsoft.Security.Bearer",
            null,
            notBefore: startTime,
            expires: expiryTime,
            signingCredentials: credentials);

    string result = new JwtSecurityTokenHandler().WriteToken(token);
    return result;
}
```

### <a name="handling-provisioning-and-deprovisioning-of-users"></a>Обработка подготовки и отмены подготовки пользователей

***Пример 1. Запрос к службе для соответствующего пользователя***

Azure Active Directory (AAD) запрашивает службу для пользователя, у которого значение атрибута `externalId` совпадает со значением атрибута mailNickname пользователя AAD. Запрос имеет вид HTTP-запроса, пример которого приведен ниже. Значение jyoung в этом запросе обозначает пример mailNickname пользователя Azure Active Directory.

>[!NOTE]
> Это только пример. Не все пользователи будут иметь атрибут mailNickname, а значение пользователя может не быть уникальным в каталоге. Кроме того, атрибут, используемый для сопоставления (в нашем примере это `externalId`), можно настроить в [сопоставлениях атрибутов AAD](customize-application-attributes.md).

```
GET https://.../scim/Users?filter=externalId eq jyoung HTTP/1.1
 Authorization: Bearer ...
```

В примере кода запрос преобразуется в вызов метода QueryAsync поставщика службы. Подпись этого метода будет выглядеть так: 

```csharp
// System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
// Microsoft.SCIM.IRequest is defined in 
// Microsoft.SCIM.Service.  
// Microsoft.SCIM.Resource is defined in 
// Microsoft.SCIM.Schemas.  
// Microsoft.SCIM.IQueryParameters is defined in 
// Microsoft.SCIM.Protocol.  

Task<Resource[]> QueryAsync(IRequest<IQueryParameters> request);
```

В примере запроса для пользователя с заданным значением для атрибута `externalId`, значения аргументов, переданных в метод QueryAsync, будут следующими:

* parameters.AlternateFilters.Count: 1
* parameters.AlternateFilters.ElementAt(0).AttributePath: "externalId"
* parameters.AlternateFilters.ElementAt(0).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(0).ComparisonValue: "jyoung"

***Пример 2. Подготовка пользователя***

Если ответ на запрос к веб-службе для пользователя со значением атрибута `externalId`, совпадающим со значением атрибута mailNickname пользователя, не возвращает никаких пользователей, AAD отправит запрос к этой службе на подготовку пользователя, соответствующего пользователю в AAD.  Ниже приведен пример такого запроса. 

```
POST https://.../scim/Users HTTP/1.1
Authorization: Bearer ...
Content-type: application/scim+json
{
   "schemas":
   [
     "urn:ietf:params:scim:schemas:core:2.0:User",
     "urn:ietf:params:scim:schemas:extension:enterprise:2.0User"],
   "externalId":"jyoung",
   "userName":"jyoung@testuser.com",
   "active":true,
   "addresses":null,
   "displayName":"Joy Young",
   "emails": [
     {
       "type":"work",
       "value":"jyoung@Contoso.com",
       "primary":true}],
   "meta": {
     "resourceType":"User"},
    "name":{
     "familyName":"Young",
     "givenName":"Joy"},
   "phoneNumbers":null,
   "preferredLanguage":null,
   "title":null,
   "department":null,
   "manager":null}
```

В примере кода запрос преобразуется в вызов метода CreateAsync поставщика службы. Подпись этого метода будет выглядеть так: 

```csharp
// System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
// Microsoft.SCIM.IRequest is defined in 
// Microsoft.SCIM.Service.  
// Microsoft.SCIM.Resource is defined in 
// Microsoft.SCIM.Schemas.  

Task<Resource> CreateAsync(IRequest<Resource> request);
```

В запросе на подготовку пользователя, значение аргумента ресурса является экземпляром класса Microsoft.SCIM.Core2EnterpriseUser, определенного в библиотеке Microsoft.SCIM.Schemas.  Если запрос на подготовку пользователя завершился удачно, реализация метода должна вернуть экземпляр класса Microsoft.SCIM.Core2EnterpriseUser со значением идентификатора свойства, равным уникальному идентификатору только подготовленного пользователя.  

***Пример 3. Запрос на текущее состояние пользователя*** 

Для обновления пользователя, уже существующего в хранилище удостоверений с интерфейсом SCIM, Azure Active Directory запросит у службы текущее состояние этого пользователя, отправив запрос следующего вида. 

```
GET ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
Authorization: Bearer ...
```

В примере кода запрос преобразуется в вызов метода RetrieveAsync поставщика службы. Подпись этого метода будет выглядеть так: 

```csharp
// System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
// Microsoft.SCIM.IRequest is defined in 
// Microsoft.SCIM.Service.  
// Microsoft.SCIM.Resource and 
// Microsoft.SCIM.IResourceRetrievalParameters 
// are defined in Microsoft.SCIM.Schemas 

Task<Resource> RetrieveAsync(IRequest<IResourceRetrievalParameters> request);
```

В приведенном выше примере запроса на получение текущего состояния пользователя в качестве значения аргумента parameters будет передан объект со следующими значениями свойств. 
  
* Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

***Пример 4 Запрос на значение атрибута ссылки для обновления*** 

Если нужно обновить атрибут ссылки, Azure Active Directory передает службе запрос с требованием проверить, совпадает ли текущее значение атрибута ссылки в хранилище удостоверений с текущим значением этого атрибута в Azure Active Directory. Для пользователей таким способом запрашивается только значение атрибута manager. Ниже приведен пример запроса на проверку значения атрибута manager для пользователя. В примере кода запрос преобразуется в вызов метода QueryAsync поставщика службы. В нем передается аргумент parameters, значением которого будет объект со следующими свойствами. 
  
* parameters.AlternateFilters.Count: 2
* parameters.AlternateFilters.ElementAt(x).AttributePath: ID
* parameters.AlternateFilters.ElementAt(x).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(x).ComparisonValue: "54D382A4-2050-4C03-94D1-E769F1D15682"
* parameters.AlternateFilters.ElementAt(y).AttributePath: "manager"
* parameters.AlternateFilters.ElementAt(y).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(y).ComparisonValue: "2819c223-7f76-453a-919d-413861904646"
* parameters.RequestedAttributePaths.ElementAt(0): ID
* parameters.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

Здесь значение индекса "x" будет равно 0, а значение индекса "y" — 1 или наоборот, в зависимости от порядка выражений в параметре filter запроса.   

***Пример 5. Запрос из Azure AD в службу SCIM для обновления пользователя*** 

Ниже приведен пример запроса от Azure Active Directory к службе SCIM для обновления пользователя. 

```
PATCH ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
Authorization: Bearer ...
Content-type: application/scim+json
{
    "schemas": 
    [
      "urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations":
    [
      {
        "op":"Add",
        "path":"manager",
        "value":
          [
            {
              "$ref":"http://.../scim/Users/2819c223-7f76-453a-919d-413861904646",
              "value":"2819c223-7f76-453a-919d-413861904646"}]}]}
```

В примере кода запрос преобразуется в вызов метода UpdateAsync поставщика службы. Подпись этого метода будет выглядеть так: 

```csharp
// System.Threading.Tasks.Tasks and 
// System.Collections.Generic.IReadOnlyCollection<T>  // are defined in mscorlib.dll.  
// Microsoft.SCIM.IRequest is defined in
// Microsoft.SCIM.Service.
// Microsoft.SCIM.IPatch, 
// is defined in Microsoft.SCIM.Protocol. 

Task UpdateAsync(IRequest<IPatch> request);
```

В примере запроса на обновление пользователя в качестве значения аргумента patch передается объект со следующими значениями свойств. 

|Аргумент|Значение|
|-|-|
|ResourceIdentifier.Identifier|"54D382A4-2050-4C03-94D1-E769F1D15682"|
|ResourceIdentifier.SchemaIdentifier|"urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"|
|(PatchRequest as PatchRequest2).Operations.Count|1|
|(PatchRequest as PatchRequest2).Operations.ElementAt(0).OperationName|OperationName.Add|
|(PatchRequest as PatchRequest2).Operations.ElementAt(0).Path.AttributePath|"manager"|
|(PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.Count|1|
|(PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Reference|http://.../scim/Users/2819c223-7f76-453a-919d-413861904646|
|(PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Value| 2819c223-7f76-453a-919d-413861904646|

***Пример 6. Отмена подготовки пользователя***

Для отзыва пользователя из хранилища удостоверений, интерфейсом для которого является служба SCIM, AAD отправляет запрос следующего вида:

```
DELETE ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
Authorization: Bearer ...
```

В примере кода запрос преобразуется в вызов метода DeleteAsync поставщика службы. Подпись этого метода будет выглядеть так: 

```csharp
// System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
// Microsoft.SCIM.IRequest is defined in 
// Microsoft.SCIM.Service.  
// Microsoft.SCIM.IResourceIdentifier, 
// is defined in Microsoft.SCIM.Protocol. 

Task DeleteAsync(IRequest<IResourceIdentifier> request);
```

В примере запроса на отзыв пользователя в качестве значения аргумента resourceIdentifier передается объект со следующими значениями свойств. 

* ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

## <a name="integrate-your-scim-endpoint-with-the-aad-scim-client"></a>Интеграция конечной точки SCIM с клиентом SCIM AAD

Службу Azure AD можно настроить так, чтобы она автоматически подготавливала назначенных пользователей и групп к приложениям, которые реализуют конкретный профиль[протокола SCIM 2.0](https://tools.ietf.org/html/rfc7644). Особенности профиля описаны в разделе [Общие сведения о реализации Azure AD SCIM](#understand-the-aad-scim-implementation).

Рекомендуется связаться с поставщиком приложения или ознакомиться с документацией поставщика для получения подтверждения соответствия этим требованиям.

> [!IMPORTANT]
> Реализация Azure AD SCIM создана на основе службы подготовки пользователей Azure AD, предназначенной для обеспечения постоянной синхронизации пользователей между Azure AD и целевым приложением, и реализует очень конкретный набор стандартных операций. Изучение этих поведений важно для понимания поведения клиента Azure AD SCIM. Дополнительные сведения см. в разделе [Циклы подготовки: начальный и добавочный](how-provisioning-works.md#provisioning-cycles-initial-and-incremental) статьи [Описание процесса подготовки](how-provisioning-works.md).

### <a name="getting-started"></a>Начало работы

Описанные ранее в этой статье приложения, поддерживающие профиль SCIM, можно подключать к Azure Active Directory с помощью функции "Приложение не из коллекции", доступной в коллекции приложений Azure AD. После подключения Azure AD запускает процесс синхронизации каждые 40 минут, запрашивая конечную точку SCIM приложения для получения назначенных пользователей и групп, и создает или изменяет их согласно сведениям о назначении.

**Подключение приложения, поддерживающего SCIM:**

1. Войдите на [портал AAD](https://aad.portal.azure.com). Обратите внимание, что вы можете получить доступ к бесплатной пробной версии Azure Active Directory с лицензиями P2, зарегистрировавшись в [программе для разработчиков](https://developer.microsoft.com/office/dev-program)
1. В области слева выберите **Корпоративные приложения**. Отобразится список всех настроенных приложений, включая приложения, добавленные из коллекции.
1. Выберите **Новое приложение** > **Создайте собственное приложение**.
1. Введите имя приложения, выберите параметр "*Интеграция с любыми другими приложениями, которых нет в коллекции*", а затем — **Добавить**, чтобы создать объект приложения. Новое приложение добавлено в список корпоративных приложений и открывается на экране управления приложениями.

   ![Снимок экрана: коллекция приложения Azure AD](media/use-scim-to-provision-users-and-groups/scim-figure-2b-1.png)
   *Коллекция приложения Azure AD*

   > [!NOTE]
   > Если вы используете старый интерфейс коллекции приложения, следуйте руководству на экране ниже.
   
   ![Снимок экрана: старый интерфейс коллекции приложения Azure AD](media/use-scim-to-provision-users-and-groups/scim-figure-2a.png)
   *Старый интерфейс коллекции приложения Azure AD*

1. На экране управления приложениями выберите **Подготовка** на панели слева.
1. В меню **Режим подготовки к работе** выберите **Автоматический**.

   ![Пример. Страница подготовки приложения на портале Azure](media/use-scim-to-provision-users-and-groups/scim-figure-2b.png)<br/>
   *Настройка подготовки на портале Azure*

1. В поле **URL-адрес клиента** введите URL-адрес конечной точки SCIM приложения. Например, `https://api.contoso.com/scim/`.
1. Если конечной точке SCIM требуется токен носителя OAuth от издателя, отличного от Azure AD, скопируйте необходимый токен носителя OAuth в необязательное для заполнения поле **Секретный токен**. Если оставить это поле пустым, то Azure AD будет добавлять в каждый запрос маркер носителя OAuth, выданный Azure AD. Приложения, использующие Azure AD в качестве поставщика удостоверений, могут проверить этот выданный Azure AD токен. 
   > [!NOTE]
   > ***Не*** рекомендуется оставлять это поле пустым и рассчитывать на токен, созданный Azure AD. Этот вариант в основном доступен для целей тестирования.
1. Выберите **Проверить подключение**. Azure Active Directory попробует подключиться к конечной точке SCIM. Если подключиться не удастся, отобразятся сведения об ошибке.  

    > [!NOTE]
    > **Проверка подключения** запрашивает конечную точку SCIM для пользователя, который не существует, используя случайный идентификатор GUID в качестве свойства сопоставления, выбранного в конфигурации Azure AD. Ожидаемый правильный ответ — HTTP 200 OK с пустым сообщением ListResponse SCIM.

1. Если подключиться к приложению удалось, выберите **Сохранить**, чтобы сохранить учетные данные администратора.
1. В разделе **Сопоставления** доступны на выбор два набора [сопоставлений атрибутов](customize-application-attributes.md): для объектов пользователей и для объектов групп. Выберите каждый из них, чтобы просмотреть атрибуты, которые синхронизируются из Azure Active Directory с вашим приложением. Атрибуты, выбранные как свойства **Matching**, используются для сопоставления пользователей и групп в вашем приложении для операций обновления. Чтобы зафиксировать изменения, щелкните **Сохранить**.

    > [!NOTE]
    > При необходимости можно отключить синхронизацию объектов групп, отключив сопоставление групп.

1. Поле **Область** в разделе **Параметры** определяет, какие пользователи и группы синхронизируются. Если выбрать параметр **Синхронизация только назначенных пользователей и групп** (рекомендуется), то будут синхронизированы только пользователи и группы, назначенные на вкладке **Пользователи и группы**.
1. После завершения настройки установите для параметра **Состояние подготовки** значение **Включено**.
1. Выберите **Сохранить**, чтобы запустить службу подготовки Azure AD.
1. Если используется синхронизация только назначенных пользователей и групп (рекомендуется), не забудьте перейти на вкладку **Пользователи и группы** и назначить пользователей или группы, которые нужно синхронизировать.

После запуска начальной синхронизации для отслеживания хода выполнения можно использовать вкладку **Журналы подготовки**. На ней отображаются все действия, выполняемые службой подготовки с приложением. Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](check-status-user-account-provisioning.md).

> [!NOTE]
> Начальная операция синхронизации занимает больше времени, чем последующие. Последующие операции синхронизации выполняются примерно каждые 40 минут при условии, что служба запущена.

## <a name="publish-your-application-to-the-aad-application-gallery"></a>Публикация приложения в коллекции приложений AAD

Если вы создаете приложение, которое будет использоваться более чем одним клиентом, его можно сделать доступным в коллекции приложений Azure AD. Это упростит организациям поиск приложения и настройку подготовки. Публикация вашего приложения в коллекции Azure AD и обеспечение доступности подготовки для других пользователей — это просто. Узнайте [здесь](../develop/v2-howto-app-gallery-listing.md), как это сделать. Корпорация Майкрософт будет работать с вами, чтобы интегрировать приложение в нашу галерею, тестировать конечную точку и выпустить [документацию](../saas-apps/tutorial-list.md) по подключению для использования клиентами.

### <a name="gallery-onboarding-checklist"></a>Контрольный список для подключения коллекции
Контрольный список поможет подключить приложение и предоставить клиентам возможность беспрепятственного развертывания. Сведения будут собраны при подключении к коллекции. 
> [!div class="checklist"]
> * Поддержка конечной точки [SCIM 2.0](#understand-the-aad-scim-implementation) пользователя и группы (требуется только одна, но рекомендуются обе).
> * Поддержка по меньшей мере 25 запросов в секунду на каждого клиента, чтобы обеспечить подготовку и отмену подготовки без задержки для пользователей и групп (обязательно).
> * Создание контактов по проектированию и поддержке для направления клиентов при подключению коллекции (обязательно)
> * 3 тестовых учетных данных без истечения срока действия для приложения (обязательно)
> * Поддержка предоставления кода авторизации OAuth или токена с длительным временем существования, как описано ниже (обязательно)
> * Создание точки по проектированию и поддержке для поддержки клиентов после подключения коллекции (обязательно)
> * Поддержка обновления нескольких членств в группах с использованием одного запроса PATCH
> * Общедоступное документирование конечной точки SCIM
> * [Поддержка обнаружения схемы](https://tools.ietf.org/html/rfc7643#section-6)

### <a name="authorization-to-provisioning-connectors-in-the-application-gallery"></a>Авторизация для использования соединителей подготовки в коллекции приложений
Спецификация SCIM не определяет схему для аутентификации и авторизации, характерную для SCIM, и основывается на использовании существующих отраслевых стандартов.

|Метод авторизации|Плюсы|Минусы|Поддержка|
|--|--|--|--|
|Имя пользователя и пароль (не рекомендуется или не поддерживается в Azure AD)|Простота реализации|Небезопасный. [Этот пароль не подходит](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/your-pa-word-doesn-t-matter/ba-p/731984)|Поддерживается в каждом конкретном случае для приложений коллекции. Не поддерживается для приложений, не относящихся к коллекции.|
|Маркер носителя с длительным сроком действия|Для маркеров с длительным сроком действия не требуется наличие пользователя. Администраторы могут легко использовать их при настройке подготовки.|Обмен маркерами с длительным сроком действия с администратором может быть трудным, если только вы не используете незащищенные методы, такие как электронная почта. |Поддерживается для приложений как из коллекции, так и не из коллекции. |
|Предоставление кода авторизации OAuth|Срок действия маркеров доступа намного короче, чем у паролей, поэтому они имеют механизм автоматического обновления, который не имеет долгосрочных маркеров носителя.  Реальный пользователь должен присутствовать во время начальной авторизации для добавления уровня ответственности. |Требуется наличие пользователя. Если пользователь покидает организацию, маркер является недействительным, и авторизацию следует выполнять заново.|Поддерживается только для приложений из коллекции. Но для краткосрочных целей тестирования можно предоставить в пользовательском интерфейсе маркер доступа в качестве секретного маркера. Поддержка предоставления кода OAuth для приложений не из коллекции уже включена в список ожидаемых работ, как и поддержка настраиваемых URL-адресов для маркеров и проверки подлинности в приложениях из коллекции.|
|Предоставление учетных данных клиента OAuth|Срок действия маркеров доступа намного короче, чем у паролей, поэтому они имеют механизм автоматического обновления, который не имеет долгосрочных маркеров носителя. Предоставление кода авторизации и предоставление учетных данных клиента создают одинаковый тип маркера доступа, поэтому перемещение между этими методами прозрачно для API.  Подготовку можно полностью автоматизировать, а новые токены могут запрашиваться без вмешательства пользователя. ||Не поддерживается для приложений как из коллекции, так и не из коллекции. Поддержка находится в невыполненной работе.|

> [!NOTE]
> Поле маркера в пользовательском интерфейсе настраиваемого приложения конфигурации подготовки AAD не рекомендуется оставлять пустым. Создаваемый маркер в основном доступен для целей тестирования.

### <a name="oauth-code-grant-flow"></a>Поток предоставления кода OAuth

Служба подготовки поддерживает [предоставление кода авторизации](https://tools.ietf.org/html/rfc6749#page-24). После отправки запроса на публикацию вашего приложения в коллекции, наша команда обратится к вам, чтобы собрать следующую информацию:

- **URL-адрес авторизации.** URL-адрес клиента для получения авторизации от владельца ресурса через перенаправление агента пользователя. Пользователь переходит по этому URL-адресу для авторизации доступа. 

- **URL-адрес для обмена маркерами.** URL-адрес клиента для обмена данными авторизации для маркера доступа, обычно с аутентификацией клиента.

- **Идентификатор клиента.** Сервер авторизации выдает зарегистрированному клиенту идентификатор клиента, который представляет собой уникальную строку с предоставленными клиентом сведениями о регистрации.  Идентификатор клиента не является секретом. Он предоставляется владельцу ресурса, и **не должен** использоваться отдельно для проверки подлинности клиента.  

- **Секрет клиента.** Секрет, созданный сервером авторизации. Это должно быть уникальное значение, известное только серверу авторизации. 

> [!NOTE]
> **URL-адрес авторизации** и **URL-адрес обмена маркерами** сейчас не настраиваются для каждого клиента.

> [!NOTE]
> OAuth версии 1 не поддерживается из-за уязвимости секрета клиента. Поддерживается протокол OAuth версии 2.  

Рекомендации (рекомендуется, но не обязательно):
* Поддержка нескольких URL-адресов перенаправления. Администраторы могут настроить подготовку как с "portal.azure.com", так и с "aad.portal.azure.com". Поддержка нескольких URL-адресов перенаправления гарантирует, что пользователи смогут авторизовать доступ с любого портала.
* Поддержка нескольких секретов для упрощения продления секрета без простоев. 

#### <a name="how-to-setup-oauth-code-grant-flow"></a>Как настроить поток предоставления кода OAuth

1. Войдите на портал Azure, перейдите к разделу **Корпоративные приложения** > **Приложение** > **Подготовка** и выберите элемент **Авторизовать**.

   1. Портал Azure перенаправляет пользователя по URL-адресу авторизации (страница входа стороннего приложения).

   1. Администратор предоставляет стороннему приложению данные для входа. 

   1. Стороннее приложение возвращает пользователя на портал Azure и передает код предоставления разрешения. 

   1. Службы подготовки Azure AD вызывают URL-адрес маркера и передают код предоставления разрешения. Стороннее приложение в ответ передает маркер доступа, маркер обновления и дату истечения срока действия маркера.

1. Когда начинается очередной цикл подготовки, служба проверяет допустимость текущего маркера и при необходимости обменивает его на новый. Маркер доступа передается в каждом запросе к приложению, и перед каждым из них проверяется допустимость запроса.

> [!NOTE]
> Пока нельзя настроить OAuth в приложении, не входящем в коллекцию, но вы можете вручную создать на сервере авторизации маркер доступа и ввести его как секретный маркер для приложения не из коллекции. Это позволяет проверить совместимость сервера SCIM с клиентом SCIM AAD, прежде чем подключать его к коллекции приложений, которая поддерживает предоставление кода OAuth.  

**Маркеры носителя OAuth с длительным сроком действия.** Если приложение не поддерживает поток предоставления кода авторизации OAuth, вместо этого создайте маркер носителя OAuth с длительным сроком действия, чтобы администратор мог с помощью этого маркера настроить интеграцию подготовки. Маркер должен быть бессрочным, иначе задание подготовки будет [помещено в карантин](application-provisioning-quarantine-status.md) по истечении срока действия маркера.

Чтобы получить дополнительные методы проверки подлинности и авторизации, сообщите нам на [UserVoice](https://aka.ms/appprovisioningfeaturerequest).

### <a name="gallery-go-to-market-launch-check-list"></a>Контрольный список для запуска вывода коллекции на рынок
Чтобы повысить осведомленность и спрос на нашу совместную интеграцию, мы рекомендуем вам обновить существующую документацию и усилить эффективность интеграции в ваших маркетинговых каналах.  Ниже приведен набор действий контрольного списка, который рекомендуется выполнить для поддержки запуска

> [!div class="checklist"]
> * Убедитесь, что команды по продажам и поддержке клиентов осведомлены о возможностях интеграции, надлежащим образом подготовлены и могут рассказать о таких возможностях. Кратко опишите все своим командам, предоставьте им ответы на часто задаваемые вопросы и материалы по продажам. 
> * Создание записи блога или пресс-релиза, описывающих совместную интеграцию, преимущества и способы начала работы. [Пример. Пресс-релиз Imprivata и Azure Active Directory](https://www.imprivata.com/company/press/imprivata-introduces-iam-cloud-platform-healthcare-supported-microsoft) 
> * Используйте социальные сети, такие как Twitter, Facebook или LinkedIn, чтобы повысить уровень интеграции с клиентами. Не забудьте включить @AzureAD, чтобы можно было ретвитнуть запись. [Пример. Публикация Imprivata в Twitter](https://twitter.com/azuread/status/1123964502909779968)
> * Создайте или обновите маркетинговые страницы либо веб-сайт (например, страницу интеграции, страницу партнера, страницу с ценами и т. д.), чтобы включить доступность объединенной интеграции. [Пример. Страница интеграции Pingboard](https://pingboard.com/org-chart-for), [страница интеграции с Smartsheet](https://www.smartsheet.com/marketplace/apps/microsoft-azure-ad), [страница цен Monday.com](https://monday.com/pricing/) 
> * Создайте статью центра справки или техническую документацию о том, как клиенты могут приступить к работе. [Пример. Интеграция Envoy с Microsoft Azure Active Directory.](https://envoy.help/en/articles/3453335-microsoft-azure-active-directory-integration/
) 
> * Оповещать клиентов о новой интеграции через взаимодействие с клиентами (ежемесячные информационные бюллетени, кампании по электронной почте, заметки о выпусках продуктов). 

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Разработка примера конечной точки SCIM](use-scim-to-build-users-and-groups-endpoints.md)
> [Автоматизация подготовки пользователей и ее отмены в приложениях SaaS](user-provisioning.md)
> [Настройка сопоставлений атрибутов для подготовки пользователей](customize-application-attributes.md)
> [Написание выражений для сопоставления атрибутов](functions-for-customizing-application-data.md)
> [Фильтры области для подготовки пользователей](define-conditional-rules-for-provisioning-user-accounts.md)
> [Уведомления, касающиеся подготовки учетной записи](user-provisioning.md)
> [Список руководств по интеграции приложений SaaS](../saas-apps/tutorial-list.md)
