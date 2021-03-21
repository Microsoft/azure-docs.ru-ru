---
title: ContentDefinitions
titleSuffix: Azure AD B2C
description: Сведения об указании элемента ContentDefinitions настраиваемой политики в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 02/15/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 62bae22b6a4bb06b1e97c18e52ad614fd2439902
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103489327"
---
# <a name="contentdefinitions"></a>ContentDefinitions

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Вы можете настроить внешний вид любого [самоподтвержденного технического профиля](self-asserted-technical-profile.md). Azure Active Directory B2C (Azure AD B2C) выполняет код в браузере клиента и использует современный подход, именуемый обменом ресурсами между источниками (CORS).

Для настройки пользовательского интерфейса необходимо указать URL-адрес в элементе **ContentDefinition** с настраиваемым содержимым HTML. В самоподтвержденном техническом профиле или **OrchestrationStep** нужно указать этот идентификатор определения содержимого. Определение содержимого может содержать элемент **LocalizedResourcesReferences**, задающий список локализованных ресурсов для загрузки. Azure AD B2C объединяет элементы пользовательского интерфейса с содержимым HTML, загруженным по указанному URL-адресу, и отображает страницу для пользователя.

Элемент **ContentDefinitions** содержит URL-адреса шаблонов HTML5, которые могут использоваться в пути взаимодействия пользователя. URI страницы HTML5 используется для указанного шага пользовательского интерфейса, например для входа или регистрации, сброса пароля или страниц ошибок. Внешний вид можно изменить с помощью переопределения LoadUri для файла HTML5. Можно создать новые определения содержимого в соответствии с потребностями. Этот элемент может содержать ссылку на локализованные ресурсы — идентификатор локализации, заданный в элементе [Localization](localization.md).

В следующем примере показаны идентификатор определения содержимого и определение локализованных ресурсов:

```xml
<ContentDefinition Id="api.localaccountsignup">
  <LoadUri>~/tenant/default/selfAsserted.cshtml</LoadUri>
  <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
  <DataUri>urn:com:microsoft:aad:b2c:elements:selfasserted:1.1.0</DataUri>
  <Metadata>
    <Item Key="DisplayName">Local account sign up page</Item>
  </Metadata>
  <LocalizedResourcesReferences MergeBehavior="Prepend">
    <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.localaccountsignup.en" />
    <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.localaccountsignup.es" />
    ...
```

Метаданные самоподтвержденного технического профиля **LocalAccountSignUpWithLogonEmail** содержат идентификатор определения содержимого **ContentDefinitionReferenceId**, для которого задано значение `api.localaccountsignup`

```xml
<TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
  <DisplayName>Email signup</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ContentDefinitionReferenceId">api.localaccountsignup</Item>
    ...
  </Metadata>
  ...
```

## <a name="contentdefinition"></a>ContentDefinition

Элемент **ContentDefinition** содержит следующий атрибут:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| Идентификатор | Да | Идентификатор определения содержимого. Значение указано в разделе **Идентификаторы определения содержимого** далее на этой странице. |

Элемент **ContentDefinition** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| LoadUri | 1:1 | Строка, содержащая URL-адрес страницы HTML5 для определения содержимого. |
| RecoveryUri | 1:1 | Строка, содержащая URL-адрес страницы HTML для отображения ошибки, связанной с определением содержимого. В настоящее время не используется, значение должно быть `~/common/default_page_error.html` . |
| DataUri | 1:1 | Строка, содержащая относительный URL-адрес HTML-файла, который предоставляет пользовательский интерфейс для вызова соответствующего шага. |
| Метаданные | 0:1 | Коллекция пар "ключ — значение", содержащая метаданные, используемые в определении содержимого. |
| LocalizedResourcesReferences | 0:1 | Коллекция ссылок на локализованные ресурсы. Этот элемент позволяет настроить локализацию пользовательского интерфейса и атрибута утверждений (claims). |

### <a name="datauri"></a>DataUri

Элемент **DataUri** используется для указания идентификатора страницы. В Azure AD B2C идентификатор страницы используется для загрузки и запуска элементов пользовательского интерфейса и JavaScript на стороне клиента. Формат значения — `urn:com:microsoft:aad:b2c:elements:page-name:version`. В следующей таблице перечислены идентификаторы страниц, которые можно использовать.

| Идентификатор страницы | Описание |
| ----- | ----------- |
| `globalexception` | Отображает страницу ошибки при обнаружении исключения или ошибки. |
| `providerselection`, `idpselection` | Отображает список поставщиков удостоверений, которые пользователи могут выбирать во время входа.  |
| `unifiedssp` | Отображает форму для входа в локальную учетную запись (на основе адреса электронной почты или имени пользователя). Это значение также предоставляет функцию "Оставаться в системе" и ссылку "Забыли пароль?" и создайте ее бесплатно. |
| `unifiedssd` | Отображает форму для входа в локальную учетную запись (на основе адреса электронной почты или имени пользователя). |
| `multifactor` | Проверяет номера телефонов с помощью SMS-сообщения или голосового вызова во время регистрации или входа. |
| `selfasserted` | Отображает форму для получения данных от пользователя. Например, позволяет пользователям создавать или обновлять свои профили. |

### <a name="select-a-page-layout"></a>Выберите макет страницы

[Код на стороне клиента JavaScript](javascript-and-page-layout.md) можно включить путем вставки `contract` между `elements` и типом страницы. Например, `urn:com:microsoft:aad:b2c:elements:contract:page-name:version`.

[!INCLUDE [active-directory-b2c-public-preview](../../includes/active-directory-b2c-public-preview.md)]

Часть [версии](page-layout.md) `DataUri` определяет пакет содержимого, содержащего HTML, CSS и JavaScript для элементов пользовательского интерфейса в политике. Если вы планируете включить код на стороне клиента JavaScript, элементы, которые основаны на коде JavaScript, должны быть неизменными. Если они не являются неизменяемыми, то любые изменения могут привести к непредвиденному поведению пользовательских страниц. Чтобы избежать этих проблем, принудительно примените макет страницы и укажите версию макета страницы. Это гарантирует, что все определения содержимого, основанные на JavaScript, будут неизменными. Даже если вы не планируете включать JavaScript, необходимо по-прежнему указать версию макета страницы для страниц.

В следующем примере показана **DataUri** `selfasserted` версии `1.2.0` :

```xml
<ContentDefinition Id="api.localaccountpasswordreset">
<LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
<RecoveryUri>~/common/default_page_error.html</RecoveryUri>
<DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
<Metadata>
    <Item Key="DisplayName">Local account change password page</Item>
</Metadata>
</ContentDefinition>
```

#### <a name="migrating-to-page-layout"></a>Переход на макет страницы

Формат значения должен содержать слово `contract` : _urn: com: Microsoft: AAD: B2C: Elements:**контракт**:p Age-Name: версия_. Чтобы указать макет страницы в пользовательских политиках, использующих старое значение **DataUri** , используйте следующую таблицу для перехода к новому формату.

| Старое значение DataUri | Новое значение DataUri |
| ----------------- | ----------------- |
| `urn:com:microsoft:aad:b2c:elements:globalexception:1.0.0` | `urn:com:microsoft:aad:b2c:elements:contract:globalexception:1.2.1` |
| `urn:com:microsoft:aad:b2c:elements:globalexception:1.1.0` | `urn:com:microsoft:aad:b2c:elements:contract:globalexception:1.2.1` |
| `urn:com:microsoft:aad:b2c:elements:idpselection:1.0.0` | `urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.1` |
| `urn:com:microsoft:aad:b2c:elements:selfasserted:1.0.0` | `urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.2` |
| `urn:com:microsoft:aad:b2c:elements:selfasserted:1.1.0` | `urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.2` |
| `urn:com:microsoft:aad:b2c:elements:unifiedssd:1.0.0` | `urn:com:microsoft:aad:b2c:elements:contract:unifiedssd:2.1.2` |
| `urn:com:microsoft:aad:b2c:elements:unifiedssp:1.0.0` | `urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:2.1.2` |
| `urn:com:microsoft:aad:b2c:elements:unifiedssp:1.1.0` | `urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:2.1.2` |
| `urn:com:microsoft:aad:b2c:elements:multifactor:1.0.0` | `urn:com:microsoft:aad:b2c:elements:contract:multifactor:1.2.0` |
| `urn:com:microsoft:aad:b2c:elements:multifactor:1.1.0` | `urn:com:microsoft:aad:b2c:elements:contract:multifactor:1.2.0` |

В следующем примере показаны идентификаторы определения содержимого и соответствующие **DataUri** с последней версией страницы: 

```xml
<!-- 
<BuildingBlocks> -->
  <ContentDefinitions>
    <ContentDefinition Id="api.error">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:globalexception:1.2.1</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.idpselections">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.1</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.idpselections.signup">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.1</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.signuporsignin">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:2.1.2</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.selfasserted">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.2</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.selfasserted.profileupdate">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.2</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.localaccountsignup">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.2</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.localaccountpasswordreset">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.2</DataUri>
    </ContentDefinition>
    <ContentDefinition Id="api.phonefactor">
      <DataUri>urn:com:microsoft:aad:b2c:elements:contract:multifactor:1.2.2</DataUri>
    </ContentDefinition>
  </ContentDefinitions>
<!-- 
</BuildingBlocks> -->
```

### <a name="metadata"></a>Метаданные

Элемент **Metadata** содержит следующие элементы.

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Item | 0:n | Метаданные, относящиеся к определению содержимого. |

Элемент **Item** из элемента **Metadata** содержит следующие атрибуты:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| Клавиши | Да | Ключ метаданных.  |

#### <a name="metadata-keys"></a>Ключи метаданных

Определение содержимого поддерживает следующие элементы метаданных:

| Ключ | Обязательно | Описание |
| --------- | -------- | ----------- |
| DisplayName | Нет | Строка, содержащая имя определения содержимого. |

### <a name="localizedresourcesreferences"></a>LocalizedResourcesReferences

Элемент **LocalizedResourcesReferences** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| LocalizedResourcesReference | 1:n | Список ссылок на локализованные ресурсы для определения содержимого. |

Элемент **локализедресаурцесреференце** содержит следующие атрибуты:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| Язык | Да | Строка, содержащая поддерживаемый язык для политики согласно стандарту RFC 5646 "Tags for Identifying Languages" (Теги для идентификации языков). |
| LocalizedResourcesReferenceId | Да | Идентификатор элемента **LocalizedResources**. |

Следующий пример показывает определение содержимого регистрации или входа в систему со ссылкой на локализацию для английского, французского и испанского языков:

```xml
<ContentDefinition Id="api.signuporsignin">
  <LoadUri>~/tenant/default/unified.cshtml</LoadUri>
  <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
  <DataUri>urn:com:microsoft:aad:b2c:elements:unifiedssp:1.0.0</DataUri>
  <Metadata>
    <Item Key="DisplayName">Signin and Signup</Item>
  </Metadata>
  <LocalizedResourcesReferences MergeBehavior="Prepend">
    <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.signuporsignin.en" />
    <LocalizedResourcesReference Language="fr" LocalizedResourcesReferenceId="api.signuporsignin.rf" />
    <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.signuporsignin.es" />
</LocalizedResourcesReferences>
</ContentDefinition>
```

Сведения о том, как добавить поддержку локализации в определения содержимого, см. в описании элемента [Localization](localization.md).

## <a name="content-definition-ids"></a>Идентификаторы определения содержимого

Атрибут ID (идентификатор) элемента **ContentDefinition** указывает тип страницы, относящейся к определению содержимого. Этот элемент определяет контекст, который будет применяться в пользовательском шаблоне HTML5 или CSS. В следующей таблице описан набор идентификаторов определений содержимого, распознаваемых Identity Experience Framework, и типы страниц, относящиеся к ним. Вы можете создать собственные определения содержимого с произвольным идентификатором.

| ID | Шаблон по умолчанию | Описание |
| -- | ---------------- | ----------- |
| **api.error** | [exception.cshtml](https://login.microsoftonline.com/static/tenant/default/exception.cshtml) | **Страница ошибки** — отображает страницу ошибки при обнаружении исключения или ошибки. |
| **api.idpselections** | [idpSelector.cshtml](https://login.microsoftonline.com/static/tenant/default/idpSelector.cshtml) | **Страница выбора поставщика удостоверений** — на этой странице содержится список поставщиков удостоверений, которые пользователи могут выбирать во время входа. Обычно это поставщики удостоверений организаций, социальных сетей, включая Facebook и Google+, или локальных учетных записей. |
| **api.idpselections.signup** | [idpSelector.cshtml](https://login.microsoftonline.com/static/tenant/default/idpSelector.cshtml) | **Выбор поставщика удостоверений для регистрации** — на этой странице содержится список поставщиков удостоверений, которые пользователи могут выбирать во время регистрации. Обычно это поставщики удостоверений организаций, социальных сетей, включая Facebook и Google+, или локальных учетных записей. |
| **api.localaccountpasswordreset** | [selfasserted. cshtml](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **Страница "Забыли пароль?"** — отображает форму, которую пользователи должны заполнить для сброса пароля. |
| **api.localaccountsignin** | [selfasserted. cshtml](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **Страница входа в локальную учетную запись** — отображает форму для входа в локальную учетную запись (на основе адреса электронной почты или имени пользователя). Форма может содержать текстовое поле ввода и окно ввода пароля. |
| **api.localaccountsignup** | [selfasserted. cshtml](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **Страница регистрации локальной учетной записи** — отображает форму для регистрации локальной учетной записи на основе адреса электронной почты или имени пользователя. Форма может содержать разные элементы управления для ввода текста, например: текстовое поле, поле ввода пароля, переключатель, раскрывающийся список с единственным выбором или флажки множественного выбора. |
| **api.phonefactor** | [multifactor-1.0.0.cshtml](https://login.microsoftonline.com/static/tenant/default/multifactor-1.0.0.cshtml) | **Страница многофакторной проверки подлинности** — проверяет номера телефонов с помощью SMS-сообщения или голосового вызова во время регистрации или входа. |
| **api.selfasserted** | [selfasserted. cshtml](https://login.microsoftonline.com/static/tenant/default/selfAsserted.cshtml) | **Страница регистрации учетной записи социальной сети** — отображает форму, которую нужно заполнить при регистрации с использованием существующей учетной записи от поставщика удостоверений в социальных сетях. Эта страница аналогична странице регистрации локальных учетных записей, за исключением полей для ввода пароля. |
| **api.selfasserted.profileupdate** | [упдатепрофиле. cshtml](https://login.microsoftonline.com/static/tenant/default/updateProfile.cshtml) | **Страница обновления профиля** — отображает форму для обновления профиля пользователя. Эта страница аналогична странице регистрации локальных учетных записей, за исключением полей для ввода пароля. |
| **api.signuporsignin** | [Unified. cshtml](https://login.microsoftonline.com/static/tenant/default/unified.cshtml) | **Унифицированная страница регистрации и входа** — обрабатывает процесс регистрации и входа пользователей. Пользователи могут использовать поставщики удостоверений организаций, социальных сетей, включая Facebook и Google+, или локальные учетные записи. |

## <a name="next-steps"></a>Следующие шаги

Пример настройки пользовательского интерфейса с помощью определений содержимого см. в следующих статьях:

[Настройка пользовательского интерфейса приложения с помощью настраиваемой политики](customize-ui-with-html.md)
