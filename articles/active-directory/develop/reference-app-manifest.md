---
title: Основные сведения о манифесте приложения Azure Active Directory
titleSuffix: Microsoft identity platform
description: Подробное описание манифеста приложения Azure Active Directory, который представляет конфигурацию удостоверений приложения в клиенте Azure AD и используется для упрощения авторизации OAuth, предоставления согласия и многого другого.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: reference
ms.workload: identity
ms.date: 02/02/2021
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: sureshja
ms.openlocfilehash: 199d85d2805b92e1421330be8f299c769637b649
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578470"
---
# <a name="azure-active-directory-app-manifest"></a>Манифест приложения Azure Active Directory

Манифест приложения содержит определения всех атрибутов для объекта приложения на платформе удостоверений Майкрософт. Также он служит механизмом для обновления объекта приложения. Дополнительные сведения см. в разделе, посвященном сущности "Приложение" и ее схеме (в [документации по сущности "Приложение API Graph"](/graph/api/resources/application)).

Вы можете настроить атрибуты приложения через портал Azure или программным способом с помощью [REST API](/graph/api/resources/application) или [PowerShell](/powershell/module/azuread#applications). Но в некоторых сценариях манифест приложения придется редактировать напрямую, чтобы настроить атрибут приложения. Ниже приведены соответствующие сценарии.

* Если вы зарегистрировали приложение c мультитенантной учетной записью AAD и личной учетной записью Майкрософт, вы не сможете изменить в пользовательском интерфейсе поддерживаемые учетные записи Майкрософт. Вместо этого для изменения типа поддерживаемых учетных записей придется использовать редактор манифеста приложения.
* Чтобы определить разрешения и роли, поддерживаемые приложением, необходимо изменить манифест приложения.

## <a name="configure-the-app-manifest"></a>Настройка манифеста приложения

Чтобы настроить манифест приложения, выполните следующее.

1. Перейдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>. Найдите и выберите службу **Azure Active Directory**.
1. Щелкните **Регистрация приложений**.
1. Выберите приложение, которое вам нужно настроить.
1. На странице **Обзор** выберите раздел **Манифест**. Откроется веб-редактор манифеста, который позволяет изменить манифест, не покидая портал. Также здесь можно выбрать команду **Скачать**, чтобы изменить манифест локально и применить к приложению обновленную версию с помощью команды **Отправить**.

## <a name="manifest-reference"></a>Справка по манифесту

В этом разделе описываются атрибуты в манифесте приложения.

### <a name="id-attribute"></a>Атрибут идентификатора

| Клавиши | Тип значения |
| :--- | :--- |
| идентификатор | Строка |

Уникальный идентификатор для приложения в каталоге. Это не тот идентификатор, который используется для идентификации приложения в транзакциях протоколов. Он используется для ссылки на объект в запросах к каталогу.

Пример

```json
    "id": "f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd",
```

### <a name="accesstokenacceptedversion-attribute"></a>Атрибут accessTokenAcceptedVersion

| Клавиши | Тип значения |
| :--- | :--- |
| accessTokenAcceptedVersion | Int32, допускающий значение NULL |

Указывает версию маркера доступа, ожидаемую ресурсом. Этот параметр меняет версию и формат JWT, созданного независимо от конечной точки или клиента, используемых для запроса маркера доступа.

Используемая конечная точка версии 1.0 или 2.0 выбирается клиентом и влияет только на версию id_tokens. В ресурсах необходимо явным образом настраивать `accesstokenAcceptedVersion` для указания поддерживаемого формата маркера доступа.

Возможные значения для `accesstokenAcceptedVersion`: 1, 2 или null. Если значение равно NULL, по умолчанию используется значение 1, соответствующее конечной точке версии 1.0.

Если `signInAudience` — `AzureADandPersonalMicrosoftAccount`, значением должно быть `2`.

Пример

```json
    "accessTokenAcceptedVersion": 2,
```

### <a name="addins-attribute"></a>Атрибут addIns

| Клавиши | Тип значения |
| :--- | :--- |
| addIns | Коллекция |

Определяет пользовательское поведение, которое может использоваться службой для вызова приложения в конкретных контекстах. Например, приложения, которые могут визуализировать файловые потоки, могут задавать свойство `addIns` для функции "FileHandler". Этот параметр позволяет службам, таким как Microsoft 365 вызывать приложение в контексте документа, над которым работает пользователь.

Пример

```json
    "addIns": [
       {
        "id": "968A844F-7A47-430C-9163-07AE7C31D407",
        "type":" FileHandler",
        "properties": [
           {
              "key": "version",
              "value": "2"
           }
        ]
       }
    ],
```

### <a name="allowpublicclient-attribute"></a>Атрибут allowPublicClient

| Клавиши | Тип значения |
| :--- | :--- |
| allowPublicClient | Логическое |

Указывает тип возврата приложения. Azure AD по умолчанию задает тип приложения из replyUrlsWithType. В некоторых сценариях Azure AD не удается определить тип клиентского приложения. Например, одним из таких сценариев является поток [ROPC](https://tools.ietf.org/html/rfc6749#section-4.3), где HTTP-запрос происходит без перенаправления URL-адреса. В таких случаях Azure AD будет интерпретировать тип приложения на основе значения этого свойства. Если для этого значения установлено значение true, тип резервного приложения устанавливается как общедоступный клиент, например, установленное приложение, работающее на мобильном устройстве. Значение по умолчанию — false. Это значит, что тип возврата приложения является закрытым клиентом, таким как веб-приложение.

Пример

```json
    "allowPublicClient": false,
```

### <a name="appid-attribute"></a>Атрибут appId

| Клавиши | Тип значения |
| :--- | :--- |
| appId | Строка |

Указывает уникальный идентификатор приложения, который назначается приложению в Azure AD.

Пример

```json
    "appId": "601790de-b632-4f57-9523-ee7cb6ceba95",
```

### <a name="approles-attribute"></a>Атрибут appRoles

| Клавиши | Тип значения |
| :--- | :--- |
| appRoles | Коллекция |

Указывает коллекцию ролей, которые может объявить приложение. Эти роли могут назначаться пользователям, группам или субъектам-службам. Ознакомьтесь с дополнительными сведениями и примерами в статье [Добавление ролей приложения в приложение и их получение в токене](howto-add-app-roles-in-azure-ad-apps.md).

Пример

```json
    "appRoles": [
        {
           "allowedMemberTypes": [
               "User"
           ],
           "description": "Read-only access to device information",
           "displayName": "Read Only",
           "id": "601790de-b632-4f57-9523-ee7cb6ceba95",
           "isEnabled": true,
           "value": "ReadOnly"
        }
    ],
```

### <a name="errorurl-attribute"></a>Атрибут errorUrl

| Клавиши | Тип значения |
| :--- | :--- |
| errorUrl | Строка |

Не поддерживается.

### <a name="groupmembershipclaims-attribute"></a>Атрибут groupMembershipClaims

| Клавиши | Тип значения |
| :--- | :--- |
|groupMembershipClaims | Строка |

Настраивает утверждение `groups`, выданное в маркере пользователя или маркере доступа OAuth 2.0, которого ожидает приложение. Чтобы задать этот атрибут, используйте одно из следующих допустимых строковых значений.

- `"None"`
- `"SecurityGroup"` (группы безопасности и роли Azure AD)
- `"ApplicationGroup"` (этот параметр включает только группы, назначенные приложению)
- `"DirectoryRole"` (Получает роли каталога Azure AD, в которые входит пользователь)
- `"All"` (будут получены все группы безопасности, группы рассылки и роли каталога Azure AD, в которые входит вошедший пользователь.

Пример

```json
    "groupMembershipClaims": "SecurityGroup",
```

### <a name="optionalclaims-attribute"></a>Атрибут optionalClaims

| Клавиши | Тип значения |
| :--- | :--- |
| optionalClaims | Строка |

Необязательные утверждения, возвращаемые в маркере службой токенов безопасности для этого конкретного приложения.

На данный момент в приложениях, в которых поддерживаются как личные учетные записи, так и Azure AD (зарегистрированный с помощью портала регистрации приложений), невозможно использовать необязательные утверждения. Тем не менее приложения, которые были зарегистрированы, только чтобы использовать Azure AD и конечную точку версии 2.0, могут получить необязательные утверждения, запрашиваемые в манифесте. Дополнительные сведения см. в статье [Необязательные утверждения](active-directory-optional-claims.md).

Пример

```json
    "optionalClaims": null,
```


### <a name="identifieruris-attribute"></a>Атрибут identifierUris

| Клавиши | Тип значения |
| :--- | :--- |
| identifierUris | Массив строк |

Определяемые пользователем URI, которые уникальным образом идентифицируют веб-приложение в клиенте Azure AD или в проверенном личном домене, если приложение является мультитенантным.

Пример

```json
    "identifierUris": "https://MyRegisteredApp",
```

### <a name="informationalurls-attribute"></a>Атрибут informationalUrls

| Клавиши | Тип значения |
| :--- | :--- |
| informationalUrls | Строка |

Указывает ссылки на условия предоставления услуг и заявление о конфиденциальности приложения. Условия обслуживания и заявление о конфиденциальности отображаются в окне запроса согласия пользователя. Дополнительные сведения см. в разделе [Практическое руководство. Добавление условий обслуживания и заявления о конфиденциальности для зарегистрированных приложений Azure AD](howto-add-terms-of-service-privacy-statement.md).

Пример

```json
    "informationalUrls": {
        "termsOfService": "https://MyRegisteredApp/termsofservice",
        "support": "https://MyRegisteredApp/support",
        "privacy": "https://MyRegisteredApp/privacystatement",
        "marketing": "https://MyRegisteredApp/marketing"
    },
```

### <a name="keycredentials-attribute"></a>Атрибут keyCredentials

| Клавиши | Тип значения |
| :--- | :--- |
| keyCredentials | Коллекция |

Содержит ссылки на учетные данные, назначенные приложению, строковые общие секреты и сертификаты X.509. Эти учетные данные используются при запросе маркеров доступа (когда приложение работает в качестве клиента, а не ресурса).

Пример

```json
    "keyCredentials": [
        {
           "customKeyIdentifier":null,
           "endDate":"2018-09-13T00:00:00Z",
           "keyId":"<guid>",
           "startDate":"2017-09-12T00:00:00Z",
           "type":"AsymmetricX509Cert",
           "usage":"Verify",
           "value":null
        }
    ],
```

### <a name="knownclientapplications-attribute"></a>Атрибут knownClientApplications

| Клавиши | Тип значения |
| :--- | :--- |
| knownClientApplications | Массив строк |

Используется для объединения согласия, если у вас есть решение, которое содержит две части: клиентское приложение и пользовательское приложение веб-API. Если в это значение ввести appID клиентского приложения, пользователю нужно будет только единожды предоставить согласие на использование клиентского приложения. Azure AD будет знать, что согласие на клиент означает неявное согласие на веб-API. Он автоматически подготовит субъекты-службы для клиента и веб-API одновременно. И клиентское приложение, и веб-API должны быть зарегистрированы в одном и том же клиенте.

Пример

```json
    "knownClientApplications": ["f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd"],
```

### <a name="logourl-attribute"></a>Атрибут logoUrl

| Клавиши | Тип значения |
| :--- | :--- |
| logoUrl | Строка |

Значение "Только для чтения", указывает на URL-адрес CDN логотипа, который был загружен на портале.

Пример

```json
    "logoUrl": "https://MyRegisteredAppLogo",
```

### <a name="logouturl-attribute"></a>Атрибут logoutUrl

| Клавиши | Тип значения |
| :--- | :--- |
| logoutUrl | Строка |

URL-адрес для выхода из приложения.

Пример

```json
    "logoutUrl": "https://MyRegisteredAppLogout",
```

### <a name="name-attribute"></a>Атрибут name

| Клавиши | Тип значения |
| :--- | :--- |
| name | Строка |

Отображаемое имя приложения.

Пример

```json
    "name": "MyRegisteredApp",
```

### <a name="oauth2allowimplicitflow-attribute"></a>Атрибут oauth2AllowImplicitFlow

| Клавиши | Тип значения |
| :--- | :--- |
| oauth2AllowImplicitFlow | Логическое |

Определяет, может ли это веб-приложение запрашивать маркеры доступа неявного потока OAuth 2.0. Значение по умолчанию — false. Этот флаг используется для браузерных приложений, таких как одностраничные приложения JavaScript. Чтобы получить дополнительные сведения, введите `OAuth 2.0 implicit grant flow` в поле над содержанием и просмотрите статьи о неявном потоке.

Пример

```json
    "oauth2AllowImplicitFlow": false,
```

### <a name="oauth2allowidtokenimplicitflow-attribute"></a>Атрибут oauth2AllowIdTokenImplicitFlow

| Клавиши | Тип значения |
| :--- | :--- |
| oauth2AllowIdTokenImplicitFlow | Логическое |

Определяет, может ли это веб-приложение запрашивать маркеры идентификатора неявного потока OAuth 2.0. Значение по умолчанию — false. Этот флаг используется для браузерных приложений, таких как одностраничные приложения JavaScript.

Пример

```json
    "oauth2AllowIdTokenImplicitFlow": false,
```

### <a name="oauth2permissions-attribute"></a>Атрибут oauth2Permissions

| Клавиши | Тип значения |
| :--- | :--- |
| oauth2Permissions | Коллекция |

Указывает коллекцию областей разрешений доступа OAuth 2.0, которые веб-API (ресурс) предоставляет клиентским приложениям. Эти области действия разрешений могут быть назначены клиентским приложениям во время предоставления согласия.

Пример

```json
    "oauth2Permissions": [
       {
          "adminConsentDescription": "Allow the app to access resources on behalf of the signed-in user.",
          "adminConsentDisplayName": "Access resource1",
          "id": "<guid>",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Allow the app to access resource1 on your behalf.",
          "userConsentDisplayName": "Access resources",
          "value": "user_impersonation"
        }
    ],
```

### <a name="oauth2requiredpostresponse-attribute"></a>Атрибут oauth2RequiredPostResponse

| Клавиши | Тип значения |
| :--- | :--- |
| oauth2RequiredPostResponse | Логическое |

Указывает, будет ли Azure AD в рамках запросов маркеров OAuth 2.0 разрешать запросы POST в отличие от запросов GET. Значение по умолчанию — false. В таком случае будут разрешены только запросы GET.

Пример

```json
    "oauth2RequirePostResponse": false,
```

### <a name="parentalcontrolsettings-attribute"></a>Атрибут parentalControlSettings

| Клавиши | Тип значения |
| :--- | :--- |
| parentalControlSettings | Строка |

- `countriesBlockedForMinors` указывает страны или регионы, в которых приложение заблокировано для несовершеннолетних.
- `legalAgeGroupRule` определяет правило возрастной группы, применяемое к пользователям приложения. Можно установить `Allow`, `RequireConsentForPrivacyServices`, `RequireConsentForMinors`, `RequireConsentForKids` или `BlockMinors`.

Пример

```json
    "parentalControlSettings": {
        "countriesBlockedForMinors": [],
        "legalAgeGroupRule": "Allow"
    },
```

### <a name="passwordcredentials-attribute"></a>Атрибут passwordCredentials

| Клавиши | Тип значения |
| :--- | :--- |
| passwordCredentials | Коллекция |

См. описание свойства `keyCredentials`.

Пример

```json
    "passwordCredentials": [
      {
        "customKeyIdentifier": null,
        "endDate": "2018-10-19T17:59:59.6521653Z",
        "keyId": "<guid>",
        "startDate":"2016-10-19T17:59:59.6521653Z",
        "value":null
      }
    ],
```

### <a name="preauthorizedapplications-attribute"></a>Атрибут preAuthorizedApplications

| Клавиши | Тип значения |
| :--- | :--- |
| preAuthorizedApplications | Коллекция |

Список приложений и запрашиваемых разрешений для косвенного согласия. Требуется, чтобы администратор предоставил согласие для приложения. preAuthorizedApplications не требуют согласия пользователя на запрашиваемые разрешения. Разрешения, перечисленные в preAuthorizedApplications, не требуют согласия пользователя. Тем не менее любые дополнительные запрашиваемые разрешения, не указанные в preAuthorizedApplications, требуют согласия пользователя.

Пример

```json
    "preAuthorizedApplications": [
       {
          "appId": "abcdefg2-000a-1111-a0e5-812ed8dd72e8",
          "permissionIds": [
             "8748f7db-21fe-4c83-8ab5-53033933c8f1"
            ]
        }
    ],
```

### <a name="publisherdomain-attribute"></a>Атрибут publisherDomain

| Клавиши | Тип значения |
| :--- | :--- |
| publisherDomain | Строка |

Домен проверенного издателя для приложения. Только для чтения.

Пример

```json
    "publisherDomain": "{tenant}.onmicrosoft.com",
```

### <a name="replyurlswithtype-attribute"></a>Атрибут replyUrlsWithType

| Клавиши | Тип значения |
| :--- | :--- |
| replyUrlsWithType | Коллекция |

Это свойство с несколькими значениями хранит список зарегистрированных значений redirect_uri, которые Azure AD будет принимать в качестве назначений при возвращении маркеров. Каждое значение URI должно содержать значение типа связанного приложения. Поддерживаемые значения:

- `Web`
- `InstalledClient`
- `Spa`

Дополнительные сведения см. в разделе [Ограничения replyUrl](./reply-url.md).

Пример

```json
    "replyUrlsWithType": [
       {
          "url": "https://localhost:4400/services/office365/redirectTarget.html",
          "type": "InstalledClient"
       }
    ],
```

### <a name="requiredresourceaccess-attribute"></a>Атрибут requiredResourceAccess

| Клавиши | Тип значения |
| :--- | :--- |
| requiredResourceAccess | Коллекция |

При динамическом согласии `requiredResourceAccess` обеспечивает процедуру использования согласия администратора и согласия пользователя для пользователей, которые используют статическое согласие. Однако этот параметр не предоставляет процедуру использования согласия пользователя для общего случая.

- `resourceAppId` — уникальный идентификатор для ресурса, доступ к которому нужен приложению. Это значение должно соответствовать appId, который был объявлен в целевом приложении ресурса.
- `resourceAccess` — массив, в котором перечислены области разрешений OAuth 2.0 и роли приложений, которые требуются приложению из указанного ресурса. Содержит значения `id` и `type` указанных ресурсов.

Пример

```json
    "requiredResourceAccess": [
        {
            "resourceAppId": "00000002-0000-0000-c000-000000000000",
            "resourceAccess": [
                {
                    "id": "311a71cc-e848-46a1-bdf8-97ff7156d8e6",
                    "type": "Scope"
                }
            ]
        }
    ],
```

### <a name="samlmetadataurl-attribute"></a>Атрибут samlMetadataUrl

| Клавиши | Тип значения |
| :--- | :--- |
| samlMetadataUrl | Строка |

URL-адрес метаданных SAML для приложения.

Пример

```json
    "samlMetadataUrl": "https://MyRegisteredAppSAMLMetadata",
```

### <a name="signinurl-attribute"></a>Атрибут signInUrl

| Клавиши | Тип значения |
| :--- | :--- |
| signInUrl | Строка |

Указывает URL-адрес домашней страницы приложения.

Пример

```json
    "signInUrl": "https://MyRegisteredApp",
```

### <a name="signinaudience-attribute"></a>Атрибут signInAudience

| Клавиши | Тип значения |
| :--- | :--- |
| signInAudience | Строка |

Указывает, какие учетные записи Майкрософт поддерживаются для текущего приложения. Поддерживаются значения:
- `AzureADMyOrg`. Пользователи с учетной записью Майкрософт или учебной учетной записью в клиенте организации Azure AD (т. е. единственный клиент).
- `AzureADMultipleOrgs`. Пользователи с учетной записью Майкрософт или учебной учетной записью в клиенте любой организации Azure AD (т. е. множество клиентов).
- `AzureADandPersonalMicrosoftAccount`. Пользователи с личной учетной записью Майкрософт, рабочей или учебной учетной записью в клиенте Azure AD любой организации.
- `PersonalMicrosoftAccount`. Личные учетные записи, которые используются для входа в службы, такие как Xbox и Skype.

Пример

```json
    "signInAudience": "AzureADandPersonalMicrosoftAccount",
```

### <a name="tags-attribute"></a>Атрибут tags

| Клавиши | Тип значения |
| :--- | :--- |
| tags | Массив строк  |

Пользовательские строки, которые могут использоваться для классификации и идентификации приложения.

Пример

```json
    "tags": [
       "ProductionApp"
    ],
```

## <a name="common-issues"></a>Распространенные проблемы

### <a name="manifest-limits"></a>Ограничения манифеста

Манифест приложения имеет несколько атрибутов, которые называются коллекциями; например, appRoles, keyCredentials, knownClientApplications, identifierUris, redirectUris, requiredResourceAccess и oauth2Permissions. В полном манифесте приложения для любого приложения общее число записей во всех коллекциях вместе не может составлять более 1200. Если ранее в манифесте приложения вы указали 100 URI перенаправления, то у вас остается только 1100 записей для использования во всех остальных коллекциях, составляющих манифест.

> [!NOTE]
> Если вы попытаетесь добавить более 1200 записей в манифест приложения, может появиться сообщение об ошибке **"Не удалось обновить приложение XXXXXX. Сведения об ошибке: Размер манифеста превысил установленный предел. Сократите количество значений и повторите запрос".**

### <a name="unsupported-attributes"></a>Неподдерживаемые атрибуты

Манифест приложения представляет схему базовой модели приложения в Azure AD. По мере развития базовой схемы редактор манифеста будет обновляться в соответствии с новой схемой. В результате вы заметите новые атрибуты в манифесте приложения. В редких случаях вы можете заметить синтаксические или семантические изменения существующих атрибутов или атрибут, который существовал ранее, но больше не поддерживается. Например, вы увидите новые атрибуты в интерфейсе [Регистрация приложений](https://go.microsoft.com/fwlink/?linkid=2083908), которые назывались иначе в интерфейсе "Регистрация приложений (прежних версий)".

| Регистрация приложений (прежних версий)| Регистрация приложений           |
|---------------------------|-----------------------------|
| `availableToOtherTenants` | `signInAudience`            |
| `displayName`             | `name`                      |
| `errorUrl`                | -                           |
| `homepage`                | `signInUrl`                 |
| `objectId`                | `Id`                        |
| `publicClient`            | `allowPublicClient`         |
| `replyUrls`               | `replyUrlsWithType`         |

Описание этих атрибутов см. в [справочнике по манифесту](#manifest-reference).

При попытке передать ранее скачанный манифест может появиться одна из следующих ошибок. Эта ошибка может быть вызвана тем, что редактор манифестов теперь поддерживает более новую версию схемы, которая не соответствует той, которую вы пытаетесь передать.

* "Не удалось обновить приложение xxxxxx. Сведения об ошибке: Недопустимый идентификатор объекта undefined. []."
* "Не удалось обновить приложение xxxxxx. Сведения об ошибке: Одно или несколько указанных значений свойств недопустимы. []."
* "Не удалось обновить приложение xxxxxx. Сведения об ошибке: Не разрешено задавать availableToOtherTenants в этой версии API для обновления. []."
* "Не удалось обновить приложение xxxxxx. Сведения об ошибке: Обновление свойства replyUrls не разрешено для этого приложения. Используйте вместо этого свойство replyUrlsWithType. []."
* "Не удалось обновить приложение xxxxxx. Сведения об ошибке: Обнаружено значение без имени типа, а ожидаемый тип недоступен. Если указана модель, то для каждого значения в полезных данных необходим тип, который может указываться в полезных данных, явно задаваться вызывающим объектом или неявно выводиться из родительского значения. []"

Если появляется одна из этих ошибок, мы рекомендуем выполнить следующие действия.

1. Измените атрибуты по отдельности в редакторе манифеста вместо отправки ранее скачанного манифеста. Используйте таблицу [справочника по манифестам](#manifest-reference), чтобы понять синтаксис и семантику старых и новых атрибутов и изменить нужные атрибуты.
1. Если рабочий процесс требует сохранить манифесты в исходном репозитории для последующего использования, мы рекомендуем заменить сохраненные манифесты в репозитории на те, которые отображаются в интерфейсе **Регистрация приложений**.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о связи между объектами приложения и субъекта-службы см. в статье [Объекты приложения и субъекта-службы в Azure Active Directory (Azure AD)](app-objects-and-service-principals.md).
* Определения основных концепций разработчика платформы идентификации Майкрософт см. в разделе [Глоссарий разработчика платформы удостоверений Майкрософт](developer-glossary.md).

Оставляйте свои замечания и пожелания в разделе ниже. Ваши отзывы помогают нам улучшать содержимое веб-сайта.

<!--article references -->
[AAD-APP-OBJECTS]:app-objects-and-service-principals.md
[AAD-DEVELOPER-GLOSSARY]:developer-glossary.md
[AAD-GROUPS-FOR-AUTHORIZATION]: http://www.dushyantgill.com/blog/2014/12/10/authorization-cloud-applications-using-ad-groups/
[ADD-UPD-RMV-APP]:quickstart-v1-integrate-apps-with-azure-ad.md
[AZURE-PORTAL]: https://portal.azure.com
[DEV-GUIDE-TO-AUTH-WITH-ARM]: http://www.dushyantgill.com/blog/2015/05/23/developers-guide-to-auth-with-azure-resource-manager-api/
[GRAPH-API]: /graph/migrate-azure-ad-graph-planning-checklist
[IMPLICIT-GRANT]:v1-oauth2-implicit-grant-flow.md
[INTEGRATING-APPLICATIONS-AAD]: ./quickstart-register-app.md
[O365-PERM-DETAILS]: /graph/permissions-reference
[RBAC-CLOUD-APPS-AZUREAD]: http://www.dushyantgill.com/blog/2014/12/10/roles-based-access-control-in-cloud-applications-using-azure-ad/