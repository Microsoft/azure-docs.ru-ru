---
title: Настройка пользовательского интерфейса
titleSuffix: Azure AD B2C
description: Узнайте, как настроить пользовательский интерфейс для приложений, использующих Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 01/28/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: ecece3a00a788b67f6c831804bf5b00372fef93d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99056118"
---
# <a name="customize-the-user-interface-in-azure-active-directory-b2c"></a>Настройка пользовательского интерфейса в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Фирменная символика и настройка пользовательского интерфейса, который Azure Active Directory B2C (Azure AD B2C) для клиентов, помогает обеспечить удобство работы пользователей в приложении. Эти возможности включают регистрацию, вход, изменение профиля и сброс пароля. В этой статье вы настраиваете страницы Azure AD B2C с помощью шаблона страницы и фирменной символики компании. 

> [!TIP]
> Сведения о настройке других аспектов пользовательских страниц за пределами шаблона страницы, логотипа баннера, фонового изображения или цвета фона см. в разделе [Настройка пользовательского интерфейса с помощью шаблона HTML](customize-ui-with-html.md).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="overview"></a>Обзор

Azure AD B2C предоставить несколько встроенных шаблонов, которые можно использовать для предоставления пользователям страниц профессионального вида. Эти шаблоны страниц также могут использоваться в качестве отправной точки для собственной настройки с помощью функции [фирменной символики компании](#company-branding) .

### <a name="ocean-blue"></a>Синий морской

Пример шаблона "голубой", отображаемый на странице входа в систему:

![Снимок экрана шаблона с голубым океаном](media/customize-ui/template-ocean-blue.png)

### <a name="slate-gray"></a>Серый планшет

Пример шаблона серого элемента, отображаемого на странице входа в систему:

![Снимок экрана шаблона серого содержания](media/customize-ui/template-slate-gray.png)

### <a name="classic"></a>Классический

Пример классического шаблона, отображаемого на странице входа в систему:

![Снимок экрана классического шаблона](media/customize-ui/template-classic.png)

### <a name="company-branding"></a>Корпоративная фирменная символика

Вы можете настроить страницы Azure AD B2C с логотипом баннера, фоновым изображением и цветом фона с помощью Azure Active Directory [фирменной символики компании](../active-directory/fundamentals/customize-branding.md). Фирменная символика компании включает в себя регистрацию, вход, изменение профиля и сброс пароля. 

В следующем примере показана страница *регистрации и входа* с настраиваемым логотипом, фоновым изображением с использованием шаблона морской цвета:

![Страница регистрации или входа с фирменной символикой, обслуживаемая Azure AD B2C](media/customize-ui/template-ocean-blue-branded.png)


## <a name="select-a-page-template"></a>Выбор шаблона страницы

::: zone pivot="b2c-user-flow"

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите значок **Каталог и подписка** в верхней панели инструментов портала, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. Выберите **Потоки пользователей**.
1. Выберите пользовательский поток, который требуется настроить.
1. В разделе **Настройка** в меню слева выберите **макеты страниц** , а затем выберите **шаблон**.
    ![Раскрывающийся список выбора шаблона на странице "поток пользователей" портал Azure](./media/customize-ui/select-page-template.png)

При выборе шаблона выбранный шаблон применяется ко всем страницам в потоке пользователя. Универсальный код ресурса (URI) для каждой страницы отображается в поле **универсального кода ресурса (URI) настраиваемой страницы** .

::: zone-end

::: zone pivot="b2c-custom-policy"

Чтобы выбрать шаблон страницы, установите `LoadUri` элемент [определений содержимого](contentdefinitions.md). В следующем примере показаны идентификаторы определения содержимого и соответствующие `LoadUri` . 

Синий океан:

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/AzureBlue/exception.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/AzureBlue/unified.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/AzureBlue/multifactor-1.0.0.cshtml</LoadUri>
  </ContentDefinition>
</ContentDefinitions>
```

Серый планшет:

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/MSA/exception.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/MSA/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/MSA/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/MSA/unified.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <LoadUri>~/tenant/templates/MSA/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/MSA/multifactor-1.0.0.cshtml</LoadUri>
  </ContentDefinition>
</ContentDefinitions>
```

Классические 

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/default/exception.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/default/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/default/idpSelector.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/AzureBlue/unified.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <LoadUri>~/tenant/templates/default/selfAsserted.cshtml</LoadUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/default/multifactor-1.0.0.cshtml</LoadUri>
  </ContentDefinition>
</ContentDefinitions>
```
::: zone-end


## <a name="configure-company-branding"></a>Настройка фирменной символики компании

Чтобы настроить страницы потока пользователей, сначала настройте фирменную символику компании в Azure Active Directory, а затем включите ее в пользователях в Azure AD B2C.

Начните с настройки логотипа баннера, фонового изображения и цвета фона в рамках **фирменной символики компании**.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. В разделе **Управление** выберите **Корпоративная фирменная символика**.
1. Выполните действия, описанные в статье [Добавление фирменной символики на страницу входа Azure Active Directory вашей организации](../active-directory/fundamentals/customize-branding.md).

При настройке фирменной символики компании в Azure AD B2C необходимо учитывать следующие моменты.

* Фирменная символика компании в Azure AD B2C в настоящее время ограничена **фоновым изображением**, **логотипом баннера** и настройкой **цвета фона** . Другие свойства в области фирменной символики компании, например **Дополнительные параметры**, *не поддерживаются*.
* На страницах потока пользователя цвет фона отображается перед загрузкой фонового изображения. Мы рекомендуем выбрать цвет фона, который ближе всего соответствует цветам в фоновом изображении, чтобы обеспечить более гладкую загрузку.
* Эмблема баннера появляется в сообщениях электронной почты для проверки, отправляемых пользователям при инициировании потока регистрации пользователя.



## <a name="enable-company-branding-in-user-flow-pages"></a>Включение фирменной символики компании на страницах потока пользователя

::: zone pivot="b2c-user-flow"

После настройки фирменной символики компании включите ее в пользовательских потоках.

1. В меню слева портал Azure выберите **Azure AD B2C**.
1. В разделе **Политики** выберите **Пользовательские потоки (политики)** .
1. Выберите поток пользователя, для которого вы хотите включить фирменную символику компании. Фирменная символика компании **не поддерживается** для стандартных типов *входа* и *редактирования стандартного профиля* .
1. В разделе **Настройка** выберите **макеты страниц**, а затем выберите страницу, для которой вы хотите установить торговую марку. Например, выберите **Единая регистрация или страница входа**.
1. В качестве **версии макета страницы (Предварительная версия)** выберите версию **1.2.0** или более позднюю.
1. Щелкните **Сохранить**.

Если вы хотите настроить фирменную символику всех страниц в потоке пользователя, задайте версию макета страницы для каждого макета страницы в потоке пользователя.

![Выбор макета страницы в Azure AD B2C в портал Azure](media/customize-ui/portal-02-page-layout-select.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

После настройки фирменной символики компании включите ее в настраиваемой политике. Настройте [версию макета страницы](contentdefinitions.md#migrating-to-page-layout) с версией страницы `contract` для *всех* определений содержимого в пользовательской политике. Формат значения должен содержать слово `contract` : _urn: com: Microsoft: AAD: B2C: Elements:**контракт**:p Age-Name: версия_. Чтобы указать макет страницы в пользовательских политиках, использующих старое значение **DataUri** . Дополнительные сведения см. в статье [Переход к макету страницы](contentdefinitions.md#migrating-to-page-layout) с использованием версии страницы.

В следующем примере показаны определения содержимого с соответствующими контрактом страницы и шаблоном *голубого цвета* страницы: 

```xml
<ContentDefinitions>
  <ContentDefinition Id="api.error">
    <LoadUri>~/tenant/templates/AzureBlue/exception.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:globalexception:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.idpselections.signup">
    <LoadUri>~/tenant/templates/AzureBlue/idpSelector.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:providerselection:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.signuporsignin">
    <LoadUri>~/tenant/templates/AzureBlue/unified.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:unifiedssp:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted">
    <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.selfasserted.profileupdate">
     <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountsignup">
     <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
     <LoadUri>~/tenant/templates/AzureBlue/selfAsserted.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:1.2.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.phonefactor">
    <LoadUri>~/tenant/templates/AzureBlue/multifactor-1.0.0.cshtml</LoadUri>
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:multifactor:1.2.0</DataUri>
  </ContentDefinition>
</ContentDefinitions>
```

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о настройке пользовательского интерфейса приложения в [Azure Active Directory B2C](customize-ui-with-html.md)см. в этой статье.
