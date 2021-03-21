---
title: Протокол SAML единого выхода Azure
description: В этой статье описывается протокол единого выхода SAML в Azure Active Directory
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 07/19/2017
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 2064ab7e759798d8934facb8d293e8ac60ec6c82
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97703421"
---
# <a name="single-sign-out-saml-protocol"></a>Протокол единого выхода SAML

Azure Active Directory (Azure AD) поддерживает профиль SAML 2.0 для единого выхода с использованием веб-браузера. Чтобы единый выход работал правильно, во время регистрации приложения необходимо явно зарегистрировать в Azure AD его **LogoutURL** (URL-адрес выхода). Azure AD использует этот URL-адрес для перенаправления пользователей после их выхода.

Azure AD поддерживает привязку перенаправления (HTTP GET), а не HTTP POST.

Эта схема демонстрирует рабочий процесс единого выхода Azure AD.

![Рабочий процесс единого выхода в Azure AD](./media/single-sign-out-saml-protocol/active-directory-saml-single-sign-out-workflow.png)

## <a name="logoutrequest"></a>LogoutRequest
Облачная служба отправляет сообщение `LogoutRequest` в Azure AD, чтобы сообщить о завершении сеанса. Ниже приведен пример элемента `LogoutRequest` .

```
<samlp:LogoutRequest xmlns="urn:oasis:names:tc:SAML:2.0:metadata" ID="idaa6ebe6839094fe4abc4ebd5281ec780" Version="2.0" IssueInstant="2013-03-28T07:10:49.6004822Z" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.workaad.com</Issuer>
  <NameID xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
</samlp:LogoutRequest>
```

### <a name="logoutrequest"></a>LogoutRequest
Элемент `LogoutRequest` , передаваемый в Azure AD, должен иметь следующие атрибуты.

* `ID` — признак запроса на выход. Значение `ID` не должно начинаться с цифры. Обычно здесь указывается строковое представление идентификатора GUID с добавлением перед ним строки **id** .
* `Version` — установите для этого элемента значение **2.0**. Это значение обязательно.
* `IssueInstant` — это строка `DateTime` со значением в формате всемирного времени (UTC) и [форматом кругового пути ("o")](/dotnet/standard/base-types/standard-date-and-time-format-strings). Azure AD ожидает значение такого типа, но не требует его.

### <a name="issuer"></a>Издатель
Элемент `Issuer` в `LogoutRequest` должен точно соответствовать одному из имен из списка **ServicePrincipalNames** в облачной службе в Azure AD. Обычно здесь передается **URI идентификатора приложения** , указанный во время регистрации приложения.

### <a name="nameid"></a>NameID
Значение элемента `NameID` должно точно совпадать с параметром `NameID` для пользователя, выполняющего выход.

## <a name="logoutresponse"></a>LogoutResponse
Azure AD отправляет `LogoutResponse` в ответ на элемент `LogoutRequest`. Ниже приведен фрагмент кода с элементом `LogoutResponse`.

```
<samlp:LogoutResponse ID="_f0961a83-d071-4be5-a18c-9ae7b22987a4" Version="2.0" IssueInstant="2013-03-18T08:49:24.405Z" InResponseTo="iddce91f96e56747b5ace6d2e2aa9d4f8c" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
  </samlp:Status>
</samlp:LogoutResponse>
```

### <a name="logoutresponse"></a>LogoutResponse
Azure AD устанавливает значения `ID`, `Version` и `IssueInstant` для элемента `LogoutResponse`. Он также задает для элемента `InResponseTo` значение, взятое из атрибута `ID` для `LogoutRequest`, который запрашивал ответ.

### <a name="issuer"></a>Издатель
Azure AD устанавливает для него значение `https://login.microsoftonline.com/<TenantIdGUID>/`, где \<TenantIdGUID> — это идентификатор клиента Azure AD.

Чтобы оценить значение элемента `Issuer` , используйте значение **URI идентификатора приложения** , указанное при регистрации приложения.

### <a name="status"></a>Состояние
Azure AD использует `StatusCode` элемент в элементе, `Status` чтобы указать успешность или неудачу выхода. При неудачной попытке выхода `StatusCode` элемент может также содержать настраиваемые сообщения об ошибках.
