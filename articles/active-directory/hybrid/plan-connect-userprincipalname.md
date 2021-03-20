---
title: Указание атрибута UserPrincipalName в Azure AD
description: В следующем документе описано, как указать атрибут UserPrincipalName.
author: billmath
ms.subservice: hybrid
ms.author: billmath
ms.date: 06/26/2018
ms.topic: conceptual
ms.workload: identity
ms.service: active-directory
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 58b4bbcac110398ee4ff132b76ce8c4868ee17f8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91317597"
---
# <a name="azure-ad-userprincipalname-population"></a>Указание атрибута UserPrincipalName в Azure AD

В этой статье описывается, как указать атрибут UserPrincipalName в Azure Active Directory (Azure AD).
Значение атрибута UserPrincipalName — это имя пользователя Azure AD для учетных записей пользователей.

## <a name="upn-terminology"></a>Терминология имени участника-пользователя
В этом руководстве используется следующая терминология:

|Термин|Описание|
|-----|-----|
|Исходный домен|Домен по умолчанию (onmicrosoft.com) в клиенте Azure AD. Например, contoso.onmicrosoft.com.|
|Адрес маршрутизации электронной почты Microsoft Online (MOERA)|В Azure AD MOERA определяется по атрибуту MailNickName Azure AD, а исходный домен Azure AD указывается в формате &lt;MailNickName&gt;@&lt;исходный домен&gt;.|
|атрибут mailNickName в локальной среде;|Атрибут в Active Directory, значение которого представляет собой псевдоним пользователя в организации Exchange.|
|Атрибут mail в локальной среде|Атрибут в Active Directory, значение которого представляет собой адрес электронной почты пользователя.|
|Основной SMTP-адрес|Основной адрес электронной почты объекта получателя Exchange. Например, SMTP:user\@contoso.com.|
|Альтернативное имя пользователя|Атрибут в локальной среде, который не является UserPrincipalName, например атрибут mail, используется для входа.|

## <a name="what-is-userprincipalname"></a>Что такое UserPrincipalName?
UserPrincipalName — это атрибут, который является именем пользователя для входа через Интернет на основе интернет-стандарта [RFC 822](https://www.ietf.org/rfc/rfc0822.txt). 

### <a name="upn-format"></a>Формат имени участника-пользователя
Имя участника-пользователя состоит из префикса (имя участника-пользователя) и суффикса (имя субъекта-пользователя). Префикс объединяется с суффиксом с помощью символа "\@". Например, "someone\@example.com". Имя участника-пользователя должно быть уникальным среди всех объектов субъекта безопасности в пределах леса каталога. 

## <a name="upn-in-azure-ad"></a>Имя участника-пользователя в Azure AD 
Имя участника-пользователя применяется в Azure AD, чтобы пользователи могли войти.  Имя участника-пользователя, доступное для применения, зависит от того, является ли домен проверенным.  Если домен проверен, тогда пользователь с этим суффиксом получит право на вход в Azure AD.  

Атрибуты синхронизируются службой Azure AD Connect.  Во время установки можно просматривать проверенные и непроверенные домены.

   ![Непроверенные домены](./media/plan-connect-userprincipalname/unverifieddomain.png) 

## <a name="alternate-login-id"></a>Альтернативное имя пользователя
В некоторых средах пользователи могут знать только свой адрес электронной почты, но не имя участника-пользователя.  Использование адреса электронной почты может объясняться требованиями корпоративной политики или зависимостью в локальном бизнес-приложении.

Альтернативное имя пользователя позволяет настроить процедуру входа таким образом, чтобы пользователи могли использовать для входа атрибут, отличный от имени участника-пользователя, например адрес электронной почты.

Чтобы включить альтернативное имя пользователя в Azure AD, не нужно дополнительно ничего настраивать, если включен Azure AD Connect. Альтернативное имя пользователя можно настроить непосредственно из мастера. Сведения о настройке входа в Azure AD для пользователей см. в разделе Синхронизация. В раскрывающемся списке **имя участника-пользователя** выберите атрибут для альтернативного идентификатора входа.

![Снимок экрана, в котором выделяется список имен участников-пользователей, в котором выбирается альтернативный атрибут идентификатора имени входа.](./media/plan-connect-userprincipalname/altloginid.png)  

Дополнительные сведения см. в статье [Configuring Alternate Login ID](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id) (Настройка альтернативного имени пользователя) и разделе [Конфигурация входа в Azure AD](how-to-connect-install-custom.md#azure-ad-sign-in-configuration).

## <a name="non-verified-upn-suffix"></a>Непроверенный суффикс имени участника-пользователя
Если атрибут UserPrincipalName или суффикс альтернативного имени пользователя для локальной среды не проверен клиентом Azure AD, тогда в качестве значения атрибута UserPrincipalName Azure AD будет задано MOERA. В Azure AD MOERA определяется по атрибуту MailNickName Azure AD, а исходный домен Azure AD указывается в формате &lt;MailNickName&gt;@&lt;исходный домен&gt;.

## <a name="verified-upn-suffix"></a>Проверенный суффикс имени участника-пользователя
Если атрибут UserPrincipalName или суффикс альтернативного имени пользователя для локальной среды подтвержден клиентом Azure AD, тогда значение атрибута UserPrincipalName Azure AD будет таким же, как и значение локального атрибута UserPrincipalName или альтернативного имени пользователя.

## <a name="azure-ad-mailnickname-attribute-value-calculation"></a>Расчет значения атрибута MailNickName Azure AD
Так как в качестве значения атрибута UserPrincipalName Azure AD можно задать MOERA, важно понять, как вычисляется значение атрибута MailNickName Azure AD, которое является префиксом MOERA.

Когда объект пользователя синхронизируется с клиентом Azure AD в первый раз, Azure AD проверяет следующие значения в определенном порядке и задает первое существующее значение для атрибута MailNickName:

- атрибут mailNickName в локальной среде;
- префикс основного SMTP-адреса;
- префикс атрибута mail в локальной среде;
- префикс локального атрибута userPrincipalName или альтернативного имени пользователя.
- префикс дополнительного SMTP-адреса.

Когда обновления пользовательского объекта синхронизируются с клиентом Azure AD, значение атрибута MailNickName обновляется, только если в локальной среде обновляется значение атрибута mailNickName.

>[!IMPORTANT]
>Azure AD повторно вычислит значение атрибута UserPrincipalName, только если обновление значения атрибута UserPrincipalName или альтернативного имени входа в локальной среде синхронизируется с клиентом Azure AD. 
>
>Каждый раз, когда Azure AD повторно вычисляет атрибут UserPrincipalName, также повторно вычисляется и значение MOERA. 

## <a name="upn-scenarios"></a>Сценарии имени участника-пользователя
Ниже приведены примеры сценариев, по которым вычисляется имя участника-пользователя.

### <a name="scenario-1-non-verified-upn-suffix--initial-synchronization"></a>Сценарий 1. Непроверенный суффикс имени участника-пользователя. Начальная синхронизация

![Сценарий 1](./media/plan-connect-userprincipalname/example1.png)

Объект пользователя в локальной среде:
- mailNickName      : &lt;не задан&gt;
- proxyAddresses        : {SMTP:us1@contoso.com}
- mail          : us2@contoso.com
- userPrincipalName : us3@contoso.com

Синхронизация объекта пользователя с клиентом Azure AD в первый раз
- Задайте для атрибута Azure AD MailNickName основной префикс SMTP-адреса.
- Задайте MOERA в формате &lt;MailNickName&gt;@&lt;исходный домен&gt;.
- Задайте для атрибута userPrincipalName в Azure AD значение MOERA.

Объект пользователя клиента Azure AD:
- MailNickName      : us1           
- UserPrincipalName : us1@contoso.onmicrosoft.com


### <a name="scenario-2-non-verified-upn-suffix--set-on-premises-mailnickname-attribute"></a>Сценарий 2. Непроверенный суффикс имени участника-пользователя. Настройка атрибута mailNickName в локальной среде

![Сценарий 2](./media/plan-connect-userprincipalname/example2.png)

Объект пользователя в локальной среде:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us1@contoso.com}
- mail          : us2@contoso.com
- userPrincipalName : us3@contoso.com

Синхронизация обновления атрибута mailNickName в локальной среде и клиента Azure AD
- Задайте атрибут mailNickName локальной среды в качестве значения атрибута MailNickName Azure AD.
- Так как атрибут userPrincipalName в локально среде не обновляется, атрибут UserPrincipalName Azure AD не нужно изменять.

Объект пользователя клиента Azure AD:
- MailNickName      : us4
- UserPrincipalName : us1@contoso.onmicrosoft.com

### <a name="scenario-3-non-verified-upn-suffix--update-on-premises-userprincipalname-attribute"></a>Сценарий 3. Непроверенный суффикс имени участника-пользователя. Обновление атрибута userPrincipalName в локальной среде

![Сценарий 3](./media/plan-connect-userprincipalname/example3.png)

Объект пользователя в локальной среде:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us1@contoso.com}
- mail          : us2@contoso.com
- userPrincipalName : us5@contoso.com

Синхронизация обновления атрибута userPrincipalName в локальной среде и клиента Azure AD
- Обновите атрибут userPrincipalName в локальной среде, что вызовет повторное вычисление MOERA, и атрибут UserPrincipalName Azure AD.
- Задайте MOERA в формате &lt;MailNickName&gt;@&lt;исходный домен&gt;.
- Задайте для атрибута userPrincipalName в Azure AD значение MOERA.

Объект пользователя клиента Azure AD:
- MailNickName      : us4
- UserPrincipalName : us4@contoso.onmicrosoft.com

### <a name="scenario-4-non-verified-upn-suffix--update-primary-smtp-address-and-on-premises-mail-attribute"></a>Сценарий 4. Непроверенный суффикс имени участника-пользователя. Обновление локального атрибута mail и основного SMTP-адреса

![Сценарий 4](./media/plan-connect-userprincipalname/example4.png)

Объект пользователя в локальной среде:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us6@contoso.com}
- mail          : us7@contoso.com
- userPrincipalName : us5@contoso.com

Синхронизация обновления атрибута mail в локальной среде и основного адреса SMTP с клиентом Azure AD
- После начальной синхронизации объекта пользователя обновление атрибута mail в локальной среде и основного адреса SMTP не затронут атрибуты Azure AD MailNickName и UserPrincipalName.

Объект пользователя клиента Azure AD:
- MailNickName      : us4
- UserPrincipalName : us4@contoso.onmicrosoft.com

### <a name="scenario-5-verified-upn-suffix--update-on-premises-userprincipalname-attribute-suffix"></a>Сценарий 5. Проверенный суффикс имени участника-пользователя. Обновление суффикса атрибута userPrincipalName в локальной среде

![Сценарий 5](./media/plan-connect-userprincipalname/example5.png)

Объект пользователя в локальной среде:
- mailNickName      : us4
- proxyAddresses        : {SMTP:us6@contoso.com}
- mail          : us7@contoso.com
- userPrincipalName : us5@verified.contoso.com

Синхронизация обновления атрибута userPrincipalName в локальной среде и клиента Azure AD
- Обновите атрибут userPrincipalName в локальной среде, что вызовет повторное вычисление атрибута UserPrincipalName Azure AD.
- Задайте атрибут userPrincipalName в локальной среде в качестве значения атрибута UserPrincipalName Azure AD, так как суффикс имени участника-пользователя проверяется с помощью клиента Azure AD.

Объект пользователя клиента Azure AD:
- MailNickName      : us4     
- UserPrincipalName : us5@verified.contoso.com

## <a name="next-steps"></a>Дальнейшие действия
- [Интеграция локальных каталогов с Azure Active Directory](whatis-hybrid-identity.md)
- [Выборочная установка Azure AD Connect](how-to-connect-install-custom.md)