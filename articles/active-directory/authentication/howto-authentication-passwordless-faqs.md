---
title: Вопросы и ответы по развертыванию ключа безопасности гибридной FIDO2 — Azure Active Directory
description: Узнайте о некоторых часто задаваемых вопросах при входе в систему с использованием ключа безопасности гибридного FIDO2 с паролем с помощью Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 02/22/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: ca4943293f9474d4089267d05460d6d8766b79e6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101646390"
---
# <a name="deployment-frequently-asked-questions-faqs-for-hybrid-fido2-security-keys-in-azure-ad"></a>Часто задаваемые вопросы о развертывании для гибридных FIDO2 ключей безопасности в Azure AD 

В этой статье рассматриваются часто задаваемые вопросы по развертыванию для гибридных устройств, присоединенных к Azure AD, и входа без пароля в локальные ресурсы. Эта функция без пароля позволяет включить проверку подлинности Azure AD на устройствах Windows 10 для гибридных устройств, присоединенных к Azure AD, с помощью ключей безопасности FIDO2. Пользователи могут входить в Windows на своих устройствах с помощью современных учетных данных, таких как FIDO2 ключи, и получать доступ к ресурсам на базе традиционных домен Active Directory Services (AD DS) с помощью простого интерфейса единого входа (SSO) для своих локальных ресурсов.

Поддерживаются следующие сценарии для пользователей в гибридной среде:

* Войдите в гибридные устройства, присоединенные к Azure AD, с помощью ключей безопасности FIDO2 и получите единый доступ к локальным ресурсам.
* Войдите в устройства, присоединенные к Azure AD, с помощью ключей безопасности FIDO2 и получите единый доступ к локальным ресурсам.

Чтобы приступить к работе с ключами безопасности FIDO2 и гибридным доступом к локальным ресурсам, ознакомьтесь со следующими статьями:

* [Ключи безопасности FIDO2 без пароля](howto-authentication-passwordless-security-key.md)
* [Вход без пароля в Windows 10](howto-authentication-passwordless-security-key-windows.md)
* [Вход без пароля в локальной среде](howto-authentication-passwordless-security-key-on-premises.md)

## <a name="security-keys"></a>Ключи безопасности

* [Для доступа к ресурсам Моя организация должна использовать двухфакторную проверку подлинности. Что можно сделать для поддержки этого требования?](#my-organization-requires-multi-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement)
* [Где можно найти совместимые ключи безопасности FIDO2?](#where-can-i-find-compliant-fido2-security-keys)
* [Что делать, если я потеряли ключ безопасности?](#what-if-i-lose-my-security-key)
* [Как данные защищаются с помощью ключа безопасности FIDO2?](#how-is-the-data-protected-on-the-fido2-security-key)
* [Как работает регистрация ключей безопасности FIDO2?](#how-does-the-registering-of-fido2-security-keys-work)
* [Есть ли у администраторов способ самостоятельного предоставления ключей пользователям?](#is-there-a-way-for-admins-to-provision-the-keys-for-the-users-directly)

### <a name="my-organization-requires-multi-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement"></a>Моя организация требует многофакторной проверки подлинности для доступа к ресурсам. Что можно сделать для поддержки этого требования?

Ключи безопасности FIDO2 бывают разными конструктивными факторами. Обратитесь к изготовителю устройства, чтобы обсудить, как можно включить устройства с помощью ПИН-кода или биометрической метрики в качестве второго фактора. Список поддерживаемых поставщиков см. в разделе [FIDO2 Security Keys providers](concept-authentication-passwordless.md#fido2-security-key-providers).

### <a name="where-can-i-find-compliant-fido2-security-keys"></a>Где можно найти совместимые ключи безопасности FIDO2?

Список поддерживаемых поставщиков см. в разделе [FIDO2 Security Keys providers](concept-authentication-passwordless.md#fido2-security-key-providers).

### <a name="what-if-i-lose-my-security-key"></a>Что делать, если я потерял ключ безопасности?

Вы можете удалить ключи в портал Azure, перейдя на страницу **сведений о безопасности** и удалив ключ безопасности FIDO2.

### <a name="how-is-the-data-protected-on-the-fido2-security-key"></a>Как данные защищаются с помощью ключа безопасности FIDO2?

Ключи безопасности FIDO2 имеют безопасный енклавес, защищающий закрытые ключи, хранящиеся на них. Ключ безопасности FIDO2 также содержит встроенные в него свойства, такие как Windows Hello, в которых нельзя извлечь закрытый ключ.

### <a name="how-does-the-registering-of-fido2-security-keys-work"></a>Как работает регистрация ключей безопасности FIDO2?

Дополнительные сведения о регистрации и использовании ключей безопасности FIDO2 см. в разделе [Включение входа с помощью ключа безопасности без пароля](howto-authentication-passwordless-security-key.md).

### <a name="is-there-a-way-for-admins-to-provision-the-keys-for-the-users-directly"></a>Есть ли у администраторов способ самостоятельного предоставления ключей пользователям?

Нет, в настоящее время нет.

## <a name="prerequisites"></a>Предварительные условия

* [Работает ли эта функция при отсутствии подключения к Интернету?](#does-this-feature-work-if-theres-no-internet-connectivity)
* [Каковы конкретные конечные точки, которые должны быть открыты для Azure AD?](#what-are-the-specific-end-points-that-are-required-to-be-open-to-azure-ad)
* [Разделы справки укажите тип присоединения к домену (присоединенный к Azure AD или гибридную службу Azure AD) для устройства Windows 10?](#how-do-i-identify-the-domain-join-type-azure-ad-joined-or-hybrid-azure-ad-joined-for-my-windows-10-device)
* [Какова рекомендация по количеству контроллеров домена, которые следует исправить?](#whats-the-recommendation-on-the-number-of-dcs-that-should-be-patched)
* [Можно ли развернуть поставщик учетных данных FIDO2 на локальном устройстве?](#can-i-deploy-the-fido2-credential-provider-on-an-on-premises-only-device)
* [Вход с ключом безопасности FIDO2 не работает для администратора домена или других учетных записей с высоким уровнем привилегий. Важно?](#fido2-security-key-sign-in-isnt-working-for-my-domain-admin-or-other-high-privilege-accounts-why)

### <a name="does-this-feature-work-if-theres-no-internet-connectivity"></a>Работает ли эта функция при отсутствии подключения к Интернету?

Подключение к Интернету является обязательным условием для включения этой функции. При первом входе пользователя с помощью ключей безопасности FIDO2 они должны иметь подключение к Интернету. Для последующих событий входа в систему должен работать кэшированный вход, позволяющий пользователю пройти проверку подлинности без подключения к Интернету.

Для обеспечения единообразной работы убедитесь, что устройства имеют доступ к Интернету и линии, соответствующие контроллерам домена.

### <a name="what-are-the-specific-end-points-that-are-required-to-be-open-to-azure-ad"></a>Каковы конкретные конечные точки, которые должны быть открыты для Azure AD?

Для регистрации и проверки подлинности необходимы следующие конечные точки:

* **. microsoftonline.com*
* **. microsoftonline-p.com*
* **. msauth.net*
* **. msauthimages.net*
* **. msecnd.net*
* **. msftauth.net*
* **. msftauthimages.net*
* **. phonefactor.net*
* *enterpriseregistration.windows.net*
* *management.azure.com*
* *policykeyservice.dc.ad.msft.net*
* *secure.aadcdn.microsoftonline-p.com*

Полный список конечных точек, необходимых для использования продуктов Microsoft Online, см. в разделе [URL-адреса и диапазоны IP-адресов Office 365](/microsoft-365/enterprise/urls-and-ip-address-ranges).

### <a name="how-do-i-identify-the-domain-join-type-azure-ad-joined-or-hybrid-azure-ad-joined-for-my-windows-10-device"></a>Разделы справки укажите тип присоединения к домену (присоединенный к Azure AD или гибридную службу Azure AD) для устройства Windows 10?

Чтобы проверить, имеет ли клиентское устройство Windows 10 правильный тип присоединение к домену, используйте следующую команду:

```console
Dsregcmd/status
```

В следующем примере выходных данных показано, что устройство подключено к Azure AD, так как *AzureADJoined* имеет значение *Yes*:

```output
+---------------------+
| Device State        |
+---------------------+

AzureADJoined: YES
EnterpriseJoined: NO
DomainedJoined: NO
```

В следующем примере выходных данных показано, что устройство гибридно присоединено к Azure AD, так как *домаинеджоинед* также имеет значение *Yes*. Имя *имя_домена* также показано ниже.

```output
+---------------------+
| Device State        |
+---------------------+

AzureADJoined: YES
EnterpriseJoined: NO
DomainedJoined: YES
DomainName: CONTOSO
```

Убедитесь, что на контроллере домена Windows Server 2016 или 2019 установлены следующие исправления. При необходимости запустите Центр обновления Windows, чтобы установить их:

* Windows Server 2016 — [KB4534307](https://support.microsoft.com/help/4534307/windows-10-update-kb4534307)
* Windows Server 2019 — [KB4534321](https://support.microsoft.com/help/4534321/windows-10-update-kb4534321)

На клиентском устройстве выполните следующую команду, чтобы проверить подключение к подходящему контроллеру домена с установленными исправлениями:

```console
nltest /dsgetdc:<domain> /keylist /kdc
```

### <a name="whats-the-recommendation-on-the-number-of-dcs-that-should-be-patched"></a>Какова рекомендация по количеству контроллеров домена, которые следует исправить?

Мы рекомендуем установить исправление для большинства контроллеров домена Windows Server 2016 или 2019 с помощью исправления, чтобы убедиться, что они могут справиться с нагрузкой запроса проверки подлинности вашей организации.

Убедитесь, что на контроллере домена Windows Server 2016 или 2019 установлены следующие исправления. При необходимости запустите Центр обновления Windows, чтобы установить их:

* Windows Server 2016 — [KB4534307](https://support.microsoft.com/help/4534307/windows-10-update-kb4534307)
* Windows Server 2019 — [KB4534321](https://support.microsoft.com/help/4534321/windows-10-update-kb4534321)

### <a name="can-i-deploy-the-fido2-credential-provider-on-an-on-premises-only-device"></a>Можно ли развернуть поставщик учетных данных FIDO2 на локальном устройстве?

Нет, эта функция не поддерживается для локального устройства. Поставщик учетных данных FIDO2 не будет отображаться.

### <a name="fido2-security-key-sign-in-isnt-working-for-my-domain-admin-or-other-high-privilege-accounts-why"></a>Вход с ключом безопасности FIDO2 не работает для администратора домена или других учетных записей с высоким уровнем привилегий. Почему?

Политика безопасности по умолчанию не предоставляет службе Azure AD разрешение на подписание учетных записей с высоким уровнем привилегий на локальные ресурсы.

Чтобы разблокировать учетные записи, используйте **Active Directory пользователи и компьютеры** для изменения свойства *msDS-Неверревеалграуп* *объекта компьютера Azure AD Kerberos (CN = азуреадкерберос, OU = Controllers \<domain-DN> )*.

## <a name="under-the-hood"></a>Механизм

* [Как Azure AD Kerberos связывается с локальной средой домен Active Directory Services?](#how-is-azure-ad-kerberos-linked-to-my-on-premises-active-directory-domain-services-environment)
* [Где можно просмотреть эти объекты сервера Kerberos, созданные в AD и опубликованные в Azure AD?](#where-can-i-view-these-kerberos-server-objects-that-are-created-in-ad-ds-and-published-in-azure-ad)
* [Почему у нас нет открытого ключа, зарегистрированного в локальной AD DS поэтому нет никакой зависимости от Интернета?](#why-cant-we-have-the-public-key-registered-to-on-premises-ad-ds-so-there-is-no-dependency-on-the-internet)
* [Как поворачиваются ключи, повернутые на объект сервера Kerberos?](#how-are-the-keys-rotated-on-the-kerberos-server-object)
* [Зачем нужно Azure AD Connect? Записывает ли любая информация обратно в AD DS из Azure AD?](#why-do-we-need-azure-ad-connect-does-it-write-any-info-back-to-ad-ds-from-azure-ad)
* [Как выглядит HTTP-запрос или ответ при запросе PRT и частичного TGT?](#what-does-the-http-requestresponse-look-like-when-requesting-prt-partial-tgt)

### <a name="how-is-azure-ad-kerberos-linked-to-my-on-premises-active-directory-domain-services-environment"></a>Как Azure AD Kerberos связывается с локальной средой домен Active Directory Services?

Существует два компонента: локальная среда AD DS и клиент Azure AD.

**Доменные службы Active Directory (AD DS)**

Сервер Kerberos Azure AD представлен в локальной среде AD DS в качестве объекта контроллера домена (DC). Этот объект контроллера домена состоит из нескольких объектов:

* *CN = Азуреадкерберос, OU = контроллеры домена,\<domain-DN>*
    
    Объект- *компьютер* , представляющий Read-Only контроллер домена (RODC) в AD DS. Нет компьютера, связанного с этим объектом. Вместо этого это логическое представление контроллера домена.

* *CN = krbtgt_AzureAD, CN = Users,\<domain-DN>*

    Объект *пользователя* , представляющий ключ шифрования билета предоставления билетов Kerberos (TGT) RODC.

* *CN = 900274c4-b7d2-43c8-90ee-00a9f650e335, CN = AzureAD, CN = System,\<domain-DN>*

    Объект *ServiceConnectionPoint* , хранящий метаданные об объектах сервера Kerberos AD Azure. Средства администрирования используют этот объект для обнаружения и поиска объектов сервера Azure AD Kerberos.

**Azure Active Directory**

Сервер Kerberos Azure AD представлен в Azure AD как объект *керберосдомаин* . Каждая локальная среда AD DS представляется в виде одного объекта *керберосдомаин* в клиенте Azure AD.

Например, у вас может быть AD DS лес с двумя доменами, такими как *contoso.com* и *Fabrikam.com*. Если вы разрешите Azure AD выдавать билеты Kerberos (TGT) для всего леса, в Azure AD есть два объекта *керберосдомаин* : один объект для *contoso.com* и один для *Fabrikam.com*.

При наличии нескольких AD DS лесов у вас есть один объект *керберосдомаин* для каждого домена в каждом лесу.

### <a name="where-can-i-view-these-kerberos-server-objects-that-are-created-in-ad-ds-and-published-in-azure-ad"></a>Где можно просмотреть эти объекты сервера Kerberos, созданные в AD DS и опубликованные в Azure AD?

Чтобы просмотреть все объекты, используйте командлеты PowerShell для сервера Azure AD, входящие в последнюю версию Azure AD Connect.

Дополнительные сведения, в том числе инструкции по просмотру объектов, см. в разделе [CREATE Kerberos Server Objects](howto-authentication-passwordless-security-key-on-premises.md#create-kerberos-server-object).

### <a name="why-cant-we-have-the-public-key-registered-to-on-premises-ad-ds-so-there-is-no-dependency-on-the-internet"></a>Почему у нас нет открытого ключа, зарегистрированного в локальной AD DS поэтому нет никакой зависимости от Интернета?

Мы получили отзыв о сложности модели развертывания для Windows Hello для бизнеса, поэтому необходимо упростить модель развертывания без использования сертификатов и PKI (FIDO2 не использует сертификаты).

### <a name="how-are-the-keys-rotated-on-the-kerberos-server-object"></a>Как поворачиваются ключи, повернутые на объект сервера Kerberos?

Как и любой другой контроллер домена, ключи *KRBTGT* для шифрования сервера Azure AD необходимо поворачивать на регулярной основе. Рекомендуется следовать тому же расписанию, что и при использовании для смены всех остальных AD DS ключей *KRBTGT* .

> [!NOTE]
> Хотя есть и другие средства для смены ключей *KRBTGT* , необходимо [использовать командлеты PowerShell для смены ключей *KRBTGT*](howto-authentication-passwordless-security-key-on-premises.md#rotating-the-azure-ad-kerberos-server-key) сервера Kerberos Azure AD. Этот метод гарантирует, что ключи будут обновлены как в локальной среде AD DS, так и в Azure AD.

### <a name="why-do-we-need-azure-ad-connect-does-it-write-any-info-back-to-ad-ds-from-azure-ad"></a>Зачем нужно Azure AD Connect? Записывает ли любая информация обратно в AD DS из Azure AD?

Azure AD Connect не записывает сведения из Azure AD в AD DS. Программа включает модуль PowerShell для создания объекта сервера Kerberos в AD DS и его публикации в Azure AD.

### <a name="what-does-the-http-requestresponse-look-like-when-requesting-prt-partial-tgt"></a>Как выглядит HTTP-запрос или ответ при запросе PRT и частичного TGT?

HTTP-запрос является стандартным запросом основного маркера обновления (PRT). Этот запрос PRT включает утверждение, указывающее, что требуется билет на выдачу билета Kerberos (TGT).

| Утверждение | Значение | Описание                             |
|-------|-------|-----------------------------------------|
| TGT   | true  | Заявка указывает клиенту требуется TGT. |

Azure AD объединяет зашифрованный ключ клиента и буфер сообщений в ответ PRT в качестве дополнительных свойств. Полезные данные шифруются с помощью ключа сеанса устройства Azure AD.

| Поле              | Тип   | Описание  |
|--------------------|--------|--------------|
| tgt_client_key     | строка | Ключ клиента в кодировке Base64 (секрет). Этот ключ является секретом клиента, используемым для защиты TGT. В этом сценарии секрет клиента создается сервером как часть каждого запроса TGT, а затем возвращается клиенту в ответе. |
| tgt_key_type       | INT    | Локальный AD DS тип ключа, используемый как для ключа клиента, так и для ключа сеанса Kerberos, включенного в KERB_MESSAGE_BUFFER. |
| tgt_message_buffer | строка | KERB_MESSAGE_BUFFER в кодировке Base64. |

## <a name="next-steps"></a>Дальнейшие действия

Чтобы приступить к работе с ключами безопасности FIDO2 и гибридным доступом к локальным ресурсам, ознакомьтесь со следующими статьями:

* [Ключи безопасности FIDO2 без пароля](howto-authentication-passwordless-security-key.md)
* [Вход без пароля в Windows 10](howto-authentication-passwordless-security-key-windows.md)
* [Вход без пароля в локальной среде](howto-authentication-passwordless-security-key-on-premises.md)
