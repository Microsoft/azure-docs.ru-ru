---
title: Версии JavaScript и макета страницы
titleSuffix: Azure AD B2C
description: Узнайте, как включить JavaScript и использовать версии макета страницы в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/22/2021
ms.custom: project-no-code, devx-track-js
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: d234e57f7c11b0d9f2cd212bde93a8b8e478ef41
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104781370"
---
# <a name="javascript-and-page-layout-versions-in-azure-active-directory-b2c"></a>Версии JavaScript и макета страницы в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

Azure AD B2C предоставляет набор упакованного содержимого, содержащего HTML, CSS и JavaScript для элементов пользовательского интерфейса в потоках пользователя и пользовательских политик. Чтобы включить JavaScript для приложений, выполните следующие действия.

::: zone pivot="b2c-user-flow"

* Выберите [Макет страницы](page-layout.md)
* Включите его в потоке пользователей с помощью портал Azure
* Использование [b2clogin.com](b2clogin.md) в запросах

::: zone-end

::: zone pivot="b2c-custom-policy"

* Выберите [Макет страницы](page-layout.md)
* Добавление элемента в [настраиваемую политику](custom-policy-overview.md)
* Использование [b2clogin.com](b2clogin.md) в запросах

::: zone-end

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]


## <a name="select-a-page-layout-version"></a>Выберите версию макета страницы

Если вы планируете включить код на стороне клиента JavaScript, элементы, которые основаны на коде JavaScript, должны быть неизменными. Если они не являются неизменяемыми, то любые изменения могут привести к непредвиденному поведению пользовательских страниц. Чтобы избежать этих проблем, принудительно Используйте макет страницы и укажите версию макета страницы, чтобы определения содержимого, основанные на JavaScript, были неизменяемыми. Даже если вы не планируете включать JavaScript, можно указать версию макета страницы для страниц.

::: zone pivot="b2c-user-flow"

Чтобы указать версию макета страницы для страниц потока пользователя, сделайте следующее: 

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Откройте политику (например B2C_1_SignupSignin), щелкнув ее.
1. Выберите **макеты страниц**. Выберите **имя макета**, а затем выберите **версию макета страницы (Предварительная версия)**.

Сведения о различных версиях макета страницы см. в [журнале изменений версии макета страницы](page-layout.md).

![Параметры макета страницы на портале, в котором отображается раскрывающийся список версий макета страницы](media/javascript-and-page-layout/page-layout-version.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

Выберите [Макет страницы](contentdefinitions.md#select-a-page-layout) для элементов пользовательского интерфейса приложения.

Определите [версию макета страницы](contentdefinitions.md#migrating-to-page-layout) с версией страницы `contract` для *всех* определений содержимого в пользовательской политике. Формат значения должен содержать слово `contract` : _urn: com: Microsoft: AAD: B2C: Elements:**контракт**:p Age-Name: версия_. Сведения о [переходе на макет страницы](contentdefinitions.md#migrating-to-page-layout) с использованием версии страницы.

В следующем примере показаны идентификаторы определения содержимого и соответствующие **DataUri** с контрактом страницы: 

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:globalexception:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:multifactor:1.2.0</DataUri>
  </ContentDefinition>
</ContentDefinitions>
```
::: zone-end

## <a name="enable-javascript"></a>Включение JavaScript

::: zone pivot="b2c-user-flow"

В **свойствах** потока пользователя можно включить JavaScript. Включение JavaScript также обеспечивает принудительное использование макета страницы. Затем можно задать версию макета страницы для пользовательского потока, как описано в следующем разделе.

![Страница свойств потока пользователей с выделенным параметром "включить JavaScript"](media/javascript-and-page-layout/javascript-settings.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

Включите выполнение сценариев. Для этого добавьте элемент **ScriptExecution** в элемент [RelyingParty](relyingparty.md).

1. Откройте файл настраиваемой политики. Например, *SignUpOrSignin.xml*.
1. Добавьте элемент **скриптексекутион** в элемент **релингпарти** :

    ```xml
    <RelyingParty>
      <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
      <UserJourneyBehaviors>
        <ScriptExecution>Allow</ScriptExecution>
      </UserJourneyBehaviors>
      ...
    </RelyingParty>
    ```
1. Сохраните и отправьте этот файл.

::: zone-end

## <a name="guidelines-for-using-javascript"></a>Рекомендации по использованию JavaScript

При настройке интерфейса приложения с помощью JavaScript следуйте приведенным ниже рекомендациям.

- Не привязывайте событие click к элементам HTML `<a>`.
- Не принимайте зависимость от кода Azure AD B2C или комментариев.
- Не изменяйте порядок или иерархию элементов HTML в Azure AD B2C. Управляйте порядком элементов пользовательского интерфейса с помощью политики Azure AD B2C.
- С учетом приведенных ниже рекомендаций можно вызвать любую службу RESTful:
    - Возможно, потребуется установить CORS службы RESTful, чтобы разрешить вызовы HTTP на стороне клиента.
    - Убедитесь, что служба RESTful безопасна и использует только протокол HTTPS.
    - Не используйте JavaScript непосредственно для вызова конечных точек Azure AD B2C.
- Вы можете внедрить JavaScript или привязать внешние файлы JavaScript. Если вы привязываете внешний файл JavaScript, обязательно используйте абсолютный URL-адрес, а не относительный URL-адрес.
- Платформы JavaScript:
    - Azure AD B2C использует [определенную версию jQuery](page-layout.md#jquery-version). Не включайте другие версии jQuery. Использование более одной версии на одной и той же странице вызывает проблемы.
    - RequireJS не поддерживается.
    - Azure AD B2C не поддерживает большинство платформ JavaScript.
- Параметры Azure AD B2C можно считывать с помощью вызова объектов `window.SETTINGS`, `window.CONTENT`, таких как текущий язык пользовательского интерфейса. Не изменяйте значения этих объектов.
- Чтобы настроить сообщение об ошибке Azure AD B2C, используйте локализацию в политике.
- Если что-то можно реализовать с помощью политики, рекомендуем использовать именно этот способ.

## <a name="javascript-samples"></a>Примеры для JavaScript

### <a name="show-or-hide-a-password"></a>Отображение или скрытие пароля

Распространенный способ помочь клиентам при регистрации — позволить им просмотреть пароль, который они ввели. Благодаря этой функции при необходимости они могут легко вносить изменения в пароль. В каждом поле ввода пароля есть флажок с подписью **Показать пароль**.  Если установить его, пароль отобразится в виде обычного текста. Включите этот фрагмент кода в шаблон для страницы подтверждения регистрации или входа в систему:

```Javascript
function makePwdToggler(pwd){
  // Create show-password checkbox
  var checkbox = document.createElement('input');
  checkbox.setAttribute('type', 'checkbox');
  var id = pwd.id + 'toggler';
  checkbox.setAttribute('id', id);

  var label = document.createElement('label');
  label.setAttribute('for', id);
  label.appendChild(document.createTextNode('show password'));

  var div = document.createElement('div');
  div.appendChild(checkbox);
  div.appendChild(label);

  // Add show-password checkbox under password input
  pwd.insertAdjacentElement('afterend', div);

  // Add toggle password callback
  function toggle(){
    if(pwd.type === 'password'){
      pwd.type = 'text';
    } else {
      pwd.type = 'password';
    }
  }
  checkbox.onclick = toggle;
  // For non-mouse usage
  checkbox.onkeydown = toggle;
}

function setupPwdTogglers(){
  var pwdInputs = document.querySelectorAll('input[type=password]');
  for (var i = 0; i < pwdInputs.length; i++) {
    makePwdToggler(pwdInputs[i]);
  }
}

setupPwdTogglers();
```

### <a name="add-terms-of-use"></a>Добавление условий использования

Включите приведенный ниже код в страницу, где требуется отобразить флажок **Условия использования**. Этот флажок обычно необходим на страницах регистрации локальной учетной записи и на страницах регистрации учетной записи социальных сетей.

```Javascript
function addTermsOfUseLink() {
    // find the terms of use label element
    var termsOfUseLabel = document.querySelector('#api label[for="termsOfUse"]');
    if (!termsOfUseLabel) {
        return;
    }

    // get the label text
    var termsLabelText = termsOfUseLabel.innerHTML;

    // create a new <a> element with the same inner text
    var termsOfUseUrl = 'https://docs.microsoft.com/legal/termsofuse';
    var termsOfUseLink = document.createElement('a');
    termsOfUseLink.setAttribute('href', termsOfUseUrl);
    termsOfUseLink.setAttribute('target', '_blank');
    termsOfUseLink.appendChild(document.createTextNode(termsLabelText));

    // replace the label text with the new element
    termsOfUseLabel.replaceChild(termsOfUseLink, termsOfUseLabel.firstChild);
}
```

Замените `termsOfUseUrl` ссылкой на ваше соглашение об условиях использования. Для каталога создайте новый атрибут пользователя с именем **термсофусе** , а затем включите **термсофусе** в качестве атрибута пользователя.

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения о настройке пользовательского интерфейса приложения в [Azure Active Directory B2C](customize-ui-with-html.md)см. в этой статье.
