---
title: Диагностика ошибок с помощью подключенной службы Azure AD (Visual Studio)
description: Подключенная служба Active Directory обнаружила несовместимый тип аутентификации
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: aaddev, vs-azure
ms.openlocfilehash: 7b437e3117540719c8c0adc5701ac1a5e934340b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88114478"
---
# <a name="diagnosing-errors-with-the-azure-active-directory-connected-service"></a>Диагностика ошибок с помощью подключенной службы Azure Active Directory

При попытке определить предыдущий код проверки подлинности Azure Active Directory обнаружил несовместимый тип проверки подлинности.

Чтобы правильно определить существующий в проекте код проверки подлинности, необходимо перестроить проект. Если возникла эта ошибка и в вашем проекте нет предыдущего кода проверки подлинности, еще перестройте проект и повторите попытку.

## <a name="project-types"></a>Типы проектов

Подключенная служба проверяет тип проекта, над которым вы работаете, чтобы внедрить в него правильную логику аутентификации. Если в проекте есть какой-либо контроллер, являющийся производным от `ApiController`, то этот проект рассматривается как проект веб-API. Если в проекте есть какие-либо контроллеры, являющиеся производными от `MVC.Controller`, то этот проект рассматривается как проект MVC. Подключенная служба не поддерживает другие типы проектов.

## <a name="compatible-authentication-code"></a>Совместимый код аутентификации

Также подключенная служба проверяет параметры аутентификации, настроенные ранее или совместимые с этой службой. Если присутствуют все необходимые параметры, то проект рассматривается как подключаемый повторно, и подключенная служба отображает эти параметры при открытии.  Если присутствуют только некоторые параметры, это рассматривается как случай ошибки.

В проекте MVC подключенная служба проверяет все перечисленные ниже параметры, которые могли быть созданы при предыдущем применении этой службы.

```xml
<add key="ida:ClientId" value="" />
<add key="ida:Tenant" value="" />
<add key="ida:AADInstance" value="" />
<add key="ida:PostLogoutRedirectUri" value="" />
```

Кроме того, подключенная служба проверяет в проекте веб-API все перечисленные ниже параметры, которые могли быть созданы при предыдущем применении этой службы:

```xml
<add key="ida:ClientId" value="" />
<add key="ida:Tenant" value="" />
<add key="ida:Audience" value="" />
```

## <a name="incompatible-authentication-code"></a>Несовместимый код аутентификации

И в завершение подключенная служба пытается обнаружить версии кода аутентификации, настроенные для предыдущих версий Visual Studio. Эта ошибка означает, что проект содержит несовместимый тип проверки подлинности. Подключенная служба обнаруживает следующие типы аутентификации от предыдущих версий Visual Studio:

* Проверка подлинности Windows
* Учетные записи индивидуальных пользователей
* Учетные записи организации

Чтобы обнаружить аутентификацию Windows в проекте MVC, подключенная служба ищет элемент `authentication` в файле `web.config`.

```xml
<configuration>
    <system.web>
        <authentication mode="Windows" />
    </system.web>
</configuration>
```

Чтобы обнаружить аутентификацию Windows в проекте Web API, подключенная служба ищет элемент `IISExpressWindowsAuthentication` в файле `.csproj`.

```xml
<Project>
    <PropertyGroup>
        <IISExpressWindowsAuthentication>enabled</IISExpressWindowsAuthentication>
    </PropertyGroup>
</Project>
```

Чтобы обнаружить аутентификацию по учетным записям отдельных пользователей, подключенная служба ищет элемент пакета в файле `packages.config`.

```xml
<packages>
    <package id="Microsoft.AspNet.Identity.EntityFramework" version="2.1.0" targetFramework="net45" />
</packages>
```

Чтобы обнаружить старый тип аутентификации по корпоративной учетной записи, подключенная служба ищет в файле `web.config` следующий элемент:

```xml
<configuration>
    <appSettings>
        <add key="ida:Realm" value="***" />
    </appSettings>
</configuration>
```

Чтобы изменить тип аутентификации, удалите несовместимый тип аутентификации и повторно добавьте подключенную службу.

Дополнительные сведения см. в статье [Сценарии аутентификации в Azure Active Directory](./authentication-vs-authorization.md).