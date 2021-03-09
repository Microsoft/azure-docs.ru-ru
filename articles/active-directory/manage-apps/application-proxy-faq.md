---
title: Azure Active Directory Application Proxy часто задаваемые вопросы
description: Ознакомьтесь с ответами на часто задаваемые вопросы об использовании AD Application Proxy Azure для публикации внутренних локальных приложений для удаленных пользователей.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: reference
ms.date: 07/23/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: contperf-fy21q2
ms.openlocfilehash: edd2ec633bd78ce1a596782deab57105e9d7f1c3
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102487752"
---
# <a name="active-directory-azure-ad-application-proxy-frequently-asked-questions"></a>Часто задаваемые вопросы о прокси приложения Active Directory (Azure AD)

На этой странице приведены ответы на часто задаваемые вопросы о Azure Active Directory прокси приложения (Azure AD).

## <a name="enabling-azure-ad-application-proxy"></a>Включение прокси приложения Azure AD

### <a name="what-license-is-required-to-use-azure-ad-application-proxy"></a>Какая лицензия необходима для использования AD Application Proxy Azure?

Чтобы использовать Azure AD Application Proxy, необходимо иметь лицензию Azure AD Premium P1 или P2. Дополнительные сведения о лицензировании см. в разделе [цены на Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/)

### <a name="what-happens-to-azure-ad-application-proxy-in-my-tenant-if-my-license-expires"></a>Что происходит с Azure AD Application Proxy в моем клиенте, если срок действия моей лицензии истек?
Если срок действия лицензии истекает, прокси приложения будет автоматически отключен. Сведения о приложении будут сохранены в течение одного года.

### <a name="why-is-the-enable-application-proxy-button-grayed-out"></a>Почему кнопка "включить прокси приложения" неактивна?

Убедитесь, что у вас есть по крайней мере лицензия Azure AD Premium P1 или P2 и установлен соединитель AD Application Proxy Azure. После успешной установки первого соединителя служба AD Application Proxy Azure будет включена автоматически.

## <a name="connector-configuration"></a>Конфигурация соединителя

### <a name="why-is-my-connector-still-using-an-older-version-and-not-auto-upgraded-to-latest-version"></a>Почему мой соединитель по-прежнему использует старую версию без автоматического обновления до последней версии?

Это может быть вызвано неправильной работой службы обновления или отсутствием новых обновлений, которые может установить служба.

Служба обновления находится в работоспособном состоянии, если она запущена и в журнале событий не записаны ошибки (журналы приложений и служб — > Microsoft-> AadApplicationProxy-> обновления > администратора). 

> [!IMPORTANT]
> Для автоматического обновления выпущены только основные версии. Рекомендуем обновить соединитель вручную по регулярному расписанию. Дополнительные сведения о новых выпусках, типе выпуска (Загрузка, автоматическое обновление), исправлении ошибок и новых возможностях см. в статье [Журнал выпусков версий Azure AD application proxy: версия](application-proxy-release-version-history.md).

Чтобы вручную обновить соединитель, выполните следующие действия.

-  Скачайте последнюю версию соединителя. (Его можно найти в разделе прокси приложения на портале Azure. Ссылку можно также найти в [журнале выпусков Azure AD application proxy: версия](application-proxy-release-version-history.md).
-   Установщик перезапускает службы соединителя Azure AD Application Proxy. В некоторых случаях может потребоваться перезагрузка сервера, если установщик не может заменить все файлы. Поэтому перед началом обновления рекомендуется закрыть все приложения (т. е. Просмотр событий).
-   Запустите установщик. Процесс обновления выполняется быстро и не требует указания учетных данных, и соединитель не будет повторно зарегистрирован.

### <a name="can-application-proxy-connector-services-run-in-a-different-user-context-than-the-default"></a>Могут ли службы соединителя прокси приложения запускаться в контексте пользователя, отличном от контекста по умолчанию?

Нет, этот сценарий не поддерживается. По умолчанию используются следующие параметры:

- Соединитель прокси приложения Microsoft AAD — Вапксвк — сетевая служба
- Средство обновления соединителей Microsoft AAD Application Proxy-Вапкупдатерсвк-NT Authority\System

### <a name="can-a-guest-user-with-the-global-administrator-or-the-application-administrator-role-register-the-connector-for-the-guest-tenant"></a>Может ли гостевой пользователь с ролью глобального администратора или администратора приложения регистрировать соединитель для клиента (гостевого)?

Нет, в настоящее время это невозможно. Попытка регистрации всегда выполняется в домашнем клиенте пользователя.

### <a name="my-back-end-application-is-hosted-on-multiple-web-servers-and-requires-user-session-persistence-stickiness-how-can-i-achieve-session-persistence"></a>Мое серверное приложение размещается на нескольких веб-серверах и требует сохранения пользовательского сеанса («закрепления»). Как можно добиться сохранения сеанса? 

Рекомендации см. в статье [Высокая доступность и балансировка нагрузки соединителей прокси приложения и приложений](application-proxy-high-availability-load-balancing.md).

### <a name="is-tls-termination-tlshttps-inspection-or-acceleration-on-traffic-from-the-connector-servers-to-azure-supported"></a>Поддерживается ли прерывание TLS (проверка или ускорение TLS/HTTPS) на трафике с серверов соединителей в Azure?

Соединитель прокси приложения выполняет проверку подлинности на основе сертификата в Azure. Прерывание TLS (проверка или ускорение TLS/HTTPS) прерывает этот метод проверки подлинности и не поддерживается. Трафик из соединителя в Azure должен обходить любые устройства, выполняющие завершение TLS.  

### <a name="is-tls-12-required-for-all-connections"></a>Требуется ли TLS 1,2 для всех подключений?
Да. Для обеспечения лучшего в своем классе шифрования прокси-служба приложения поддерживает доступ только по протоколам TLS 1.2. Эти изменения внедрялись поэтапно и окончательно вступили в силу 31 августа 2019 г. Убедитесь, что все ваши комбинации "клиент-сервер" и "браузер-сервер" обновлены для использования протокола TLS 1.2 и смогут обеспечить подключение к прокси-службе приложения. Это также касается клиентов, с помощью которых ваши пользователи получают доступ к приложениям, опубликованным через прокси приложения. Полезные ссылки и ресурсы см. в статье [Подготовка к использованию TLS 1.2 в Office 365](/microsoft-365/compliance/prepare-tls-1.2-in-office-365).

### <a name="can-i-place-a-forward-proxy-device-between-the-connector-servers-and-the-back-end-application-server"></a>Можно ли разместить устройство с прямым прокси-сервером между серверами соединителей и сервером серверных приложений?
Да, этот сценарий поддерживается начиная с соединителя версии 1.5.1526.0. См. раздел [Работа с существующими локальными прокси-серверами](application-proxy-configure-connectors-with-proxy-servers.md).

### <a name="should-i-create-a-dedicated-account-to-register-the-connector-with-azure-ad-application-proxy"></a>Следует ли создать выделенную учетную запись для регистрации соединителя в Azure AD Application Proxy?

Нет причин. Будет работать любая учетная запись глобального администратора или администратора приложения. Учетные данные, указанные во время установки, не используются после процесса регистрации. Вместо этого к соединителю выдается сертификат, который используется для проверки подлинности с этого момента.

### <a name="how-can-i-monitor-the-performance-of-the-azure-ad-application-proxy-connector"></a>Как можно отслеживать производительность соединителя Azure AD Application Proxy?

Существуют счетчики системного монитора, которые устанавливаются вместе с соединителем. Чтобы просмотреть эти элементы:  

1. Нажмите кнопку **Пуск**, введите "PerfMon" и нажмите клавишу ВВОД.
2. Выберите **системный монитор** и щелкните зеленый **+** значок.
3. Добавьте счетчики **соединителя прокси приложения Microsoft AAD** , которые необходимо отслеживать.

### <a name="does-the-azure-ad-application-proxy-connector-have-to-be-on-the-same-subnet-as-the-resource"></a>Должен ли соединитель Azure AD Application Proxy находиться в той же подсети, что и ресурс?

Соединитель не обязательно должен находиться в той же подсети. Однако ему требуется разрешение имен (DNS, файл Hosts) для ресурса и необходимое сетевое подключение (маршрутизация к ресурсу, порты, открытые на ресурсе и т. д.). Рекомендации см. в разделе [рекомендации по топологии сети при использовании Azure Active Directory Application proxy](application-proxy-network-topology.md).

### <a name="what-versions-of-windows-server-can-i-install-a-connector-on"></a>В каких версиях Windows Server можно установить соединитель?

Для прокси приложения требуется Windows Server 2012 R2 или более поздней версии. В настоящее время существует ограничение на HTTP2 для Windows Server 2019. Для успешного использования соединителя в Windows Server 2019 необходимо добавить следующий раздел реестра и перезапустить сервер:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp\EnableDefaultHttp2 (DWORD) Value: 0 
```

## <a name="application-configuration"></a>Конфигурация приложений

### <a name="i-am-receiving-an-error-about-an-invalid-certificate-or-possible-wrong-password"></a>Я получаю сообщение об ошибке: Недопустимый сертификат или возможный неверный пароль

После передачи SSL-сертификата вы получите сообщение "Недопустимый сертификат, возможно, неверный пароль" на портале.

Ниже приведены некоторые советы по устранению этой ошибки.
- Проверьте наличие проблем с сертификатом. Установите его на локальном компьютере. Если проблемы не возникают, сертификат является хорошим.
- Убедитесь, что пароль не содержит специальных символов. Для тестирования пароль должен содержать только символы 0-9, A – Z и A – z.
- Если сертификат был создан с помощью поставщика хранилища ключей программного обеспечения Майкрософт, необходимо использовать алгоритм RSA.

### <a name="what-is-the-length-of-the-default-and-long-back-end-timeout-can-the-timeout-be-extended"></a>Какова длина значения по умолчанию и времени ожидания серверной части "Long"? Можно ли увеличить время ожидания?

Длина по умолчанию составляет 85 секунд. Значение параметра "Long" составляет 180 секунд. Ограничение времени ожидания не может быть расширено.

### <a name="can-a-service-principal-manage-application-proxy-using-powershell-or-microsoft-graph-apis"></a>Может ли субъект-служба управлять прокси приложения с помощью PowerShell или Microsoft Graph API?

Нет, в настоящее время такая возможность не поддерживается.

### <a name="what-happens-if-i-delete-cwap_authsecret-the-client-secret-in-the-app-registration"></a>Что произойдет, если удалить CWAP_AuthSecret (секрет клиента) в регистрации приложения?

Секрет клиента, также называемый *CWAP_AuthSecret*, автоматически добавляется в объект приложения (регистрация приложения) при создании приложения AD application proxy Azure.

Секрет клиента действителен в течение одного года. Новый секрет клиента в течение одного года создается автоматически до истечения срока действия текущего действительного секрета клиента. Три CWAP_AuthSecret клиентских секретов хранятся в объекте приложения постоянно. 

> [!IMPORTANT]
> Удаление CWAP_AuthSecret прерывает предварительную проверку подлинности для AD Application Proxy Azure. Не удаляйте CWAP_AuthSecret.

### <a name="how-do-i-change-the-landing-page-my-application-loads"></a>Разделы справки изменить целевую страницу, с которой загружается приложение?

На странице регистрации приложений можно изменить URL-адрес домашней страницы на требуемый внешний URL-адрес целевой странице. Указанная страница будет загружаться при запуске приложения из папки "Мои приложения" или с портала Office 365. Инструкции по настройке см. в разделе [Настройка пользовательской домашней страницы для опубликованных приложений с помощью Azure AD application proxy](./application-proxy-configure-custom-home-page.md)

### <a name="can-only-iis-based-applications-be-published-what-about-web-applications-running-on-non-windows-web-servers-does-the-connector-have-to-be-installed-on-a-server-with-iis-installed"></a>Можно ли публиковать только приложения на основе IIS? Как насчет веб-приложений, работающих на веб-серверах, отличных от Windows? Нужно ли устанавливать соединитель на сервере с установленными службами IIS?

Нет, для опубликованных приложений не требуется IIS. Можно публиковать веб-приложения, работающие на серверах, отличных от Windows Server. Однако использование предварительной проверки подлинности на сервере, отличном от Windows Server, может оказаться невозможным в зависимости от того, поддерживает ли веб-сервер согласование (проверка подлинности Kerberos). Службы IIS не требуются на сервере, на котором установлен соединитель.

### <a name="can-i-configure-application-proxy-to-add-the-hsts-header"></a>Можно ли настроить прокси приложения для добавления заголовка HSTS?
Прокси приложения не добавляет в ответы HTTPS заголовок HTTP с ограниченным транспортом, но он будет поддерживать заголовок, если он находится в исходном ответе, отправленном опубликованным приложением. Чтобы включить эту функцию, следует проделать этот параметр в соответствии с планом. Если вы заинтересованы в предварительной версии, которая позволяет добавить это к ответам, обратитесь к aadapfeedback@microsoft.com для получения дополнительных сведений.

## <a name="integrated-windows-authentication"></a>Встроенная проверка подлинности Windows

### <a name="when-should-i-use-the-principalsallowedtodelegatetoaccount-method-when-setting-up-kerberos-constrained-delegation-kcd"></a>Когда следует использовать метод PrincipalsAllowedToDelegateToAccount при настройке ограниченного делегирования Kerberos (KCD)?

Метод PrincipalsAllowedToDelegateToAccount используется, когда серверы соединителей находятся в разных доменах из учетной записи службы веб-приложений. Он требует использования ограниченного делегирования на основе ресурсов.
Если серверы соединителей и учетная запись службы веб-приложения находятся в одном домене, можно использовать Active Directory пользователи и компьютеры для настройки параметров делегирования на каждой учетной записи компьютера соединителя, что позволяет им делегировать целевому имени участника-службы.

Если серверы соединителей и учетная запись службы веб-приложения находятся в разных доменах, используется делегирование на основе ресурсов. Разрешения на делегирование настраиваются на целевом веб-сервере и в учетной записи службы веб-приложений. Этот метод ограниченного делегирования является относительно новым. Этот метод появился в Windows Server 2012, который поддерживает делегирование между доменами, позволяя владельцу ресурса (веб-службе) управлять тем, какие компьютеры и учетные записи служб могут быть делегированы. Нет пользовательского интерфейса для помощи с этой конфигурацией, поэтому вам потребуется использовать PowerShell.
Дополнительные сведения см. в техническом документе [об ограниченном делегировании Kerberos с помощью прокси приложения](https://aka.ms/kcdpaper).

### <a name="does-ntlm-authentication-work-with-azure-ad-application-proxy"></a>Работает ли проверка подлинности NTLM с Azure AD Application Proxy?

Проверку подлинности NTLM нельзя использовать в качестве метода предварительной проверки подлинности или единого входа. Проверка подлинности NTLM может использоваться только в том случае, если ее можно согласовать непосредственно между клиентом и опубликованным веб-приложением. Использование проверки подлинности NTLM обычно приводит к отображению запроса на вход в браузере.

### <a name="can-i-use-the-logon-identity-on-premises-user-principal-name-or-on-premises-sam-account-name-in-a-b2b-iwa-single-sign-on-scenario"></a>Можно ли использовать удостоверение входа "локальный участник пользователя" или "локальное имя учетной записи SAM" в сценарии с единым входом B2B IWA?

Нет, это не сработает, так как гостевой пользователь в Azure AD не имеет атрибута, необходимого для любого из указанных выше удостоверений входа.

В этом случае будет использоваться резервное имя участника-пользователя. Дополнительные сведения о сценарии B2B см. в статье [предоставление пользователям B2B в Azure AD доступа к локальным приложениям](../external-identities/hybrid-cloud-to-on-premises.md).

## <a name="pass-through-authentication"></a>Сквозная аутентификация

### <a name="can-i-use-conditional-access-policies-for-applications-published-with-pass-through-authentication"></a>Можно ли использовать политики условного доступа для приложений, опубликованных с помощью сквозной проверки подлинности?

Политики условного доступа применяются только для пользователей, которые прошли предварительную проверку подлинности в Azure AD. Сквозная проверка подлинности не активирует аутентификацию Azure AD, поэтому применение политик условного доступа невозможно. При использовании сквозной проверки подлинности политики MFA должны быть реализованы на локальном сервере, если это возможно, или путем включения предварительной проверки подлинности в Azure AD Application Proxy.

### <a name="can-i-publish-a-web-application-with-client-certificate-authentication-requirement"></a>Можно ли опубликовать веб-приложение с требованием проверки подлинности сертификата клиента?

Нет, этот сценарий не поддерживается, так как прокси приложения будет завершать трафик TLS.  

## <a name="remote-desktop-gateway-publishing"></a>Публикация шлюза удаленный рабочий стол

### <a name="how-can-i-publish-remote-desktop-gateway-over-azure-ad-application-proxy"></a>Как опубликовать удаленный рабочий стол шлюза в Azure AD Application Proxy?

См. статью [публикация удаленный рабочий стол с помощью AD application proxy Azure](application-proxy-integrate-with-remote-desktop-services.md).

### <a name="can-i-use-kerberos-constrained-delegation-single-sign-on---windows-integrated-authentication-in-the-remote-desktop-gateway-publishing-scenario"></a>Можно ли использовать ограниченное делегирование Kerberos (единый Sign-On — встроенная проверка подлинности Windows) в сценарии публикации шлюза удаленный рабочий стол?

Нет, этот сценарий не поддерживается.  

### <a name="my-users-dont-use-internet-explorer-11-and-the-pre-authentication-scenario-doesnt-work-for-them-is-this-expected"></a>Пользователи не используют Internet Explorer 11, и сценарий предварительной проверки подлинности не подходит для них. Так и должно быть?

Да, это ожидаемое значение. Для сценария предварительной проверки подлинности требуется элемент управления ActiveX, который не поддерживается сторонними браузерами.

### <a name="is-the-remote-desktop-web-client-html5-supported"></a>Поддерживается ли удаленный рабочий стол веб-клиентом (HTML5)?

Да, этот сценарий сейчас находится в общедоступной предварительной версии. См. статью [публикация удаленный рабочий стол с помощью AD application proxy Azure](application-proxy-integrate-with-remote-desktop-services.md).

### <a name="after-i-configured-the-pre-authentication-scenario-i-realized-that-the-user-has-to-authenticate-twice-first-on-the-azure-ad-sign-in-form-and-then-on-the-rdweb-sign-in-form-is-this-expected-how-can-i-reduce-this-to-one-sign-in"></a>После настройки сценария предварительной проверки подлинности я понял, что пользователь должен пройти проверку подлинности дважды: сначала на форме входа в Azure AD, а затем на форме входа RDWeb. Так и должно быть? Как можно сократить это до одного входа?

Да, это ожидаемое значение. Если компьютер пользователя подключен к Azure AD, он автоматически входит в Azure AD. Пользователь должен предоставить свои учетные данные только в форме входа RDWeb.

## <a name="sharepoint-publishing"></a>Публикация SharePoint

### <a name="how-can-i-publish-sharepoint-over-azure-ad-application-proxy"></a>Как опубликовать SharePoint через Azure AD Application Proxy?

См. статью [Включение удаленного доступа к SharePoint с помощью AD application proxy Azure](application-proxy-integrate-with-sharepoint-server.md).

### <a name="can-i-use-the-sharepoint-mobile-app-ios-android-to-access-a-published-sharepoint-server"></a>Можно ли использовать мобильное приложение SharePoint (iOS и Android) для доступа к опубликованному серверу SharePoint?

[Мобильное приложение SharePoint](/sharepoint/administration/supporting-the-sharepoint-mobile-apps-online-and-on-premises) не поддерживает Azure Active Directory предварительную проверку подлинности в настоящее время.

## <a name="active-directory-federation-services-ad-fs-publishing"></a>Публикация службы федерации Active Directory (AD FS) (AD FS) 

### <a name="can-i-use-azure-ad-application-proxy-as-ad-fs-proxy-like-web-application-proxy"></a>Можно ли использовать AD Application Proxy Azure в качестве прокси-сервера AD FS (например, прокси)?

Нет. Azure AD Application Proxy предназначен для работы с Azure AD и не выполняет требования, выступающие в роли прокси-сервера AD FS.

## <a name="websocket"></a>WebSocket

### <a name="does-websocket-support-work-for-applications-other-than-qliksense-and-remote-desktop-web-client-html5"></a>Поддерживает ли WebSocket работу для приложений, отличных от QlikSense и удаленный рабочий стол веб-клиента (HTML5)?

В настоящее время поддержка протокола WebSocket по-прежнему доступна в общедоступной предварительной версии, и она может не работать для других приложений. Некоторые клиенты успешно выполнили смешанный успешный протокол WebSocket с другими приложениями. При тестировании таких сценариев мы будем рады услышать результаты. Отправьте нам свой отзыв по адресу aadapfeedback@microsoft.com .

Функции (журналы событий, PowerShell и службы удаленных рабочих столов) в центре администрирования Windows (ВАК) не работают в Azure AD Application Proxy в настоящее время.

## <a name="link-translation"></a>Преобразование ссылок

### <a name="does-using-link-translation-affect-performance"></a>Влияет ли преобразование ссылок на производительность?

Да. Преобразование ссылок влияет на производительность. Служба прокси приложения просматривает приложение на наличие жестко закодированных ссылок и заменяет их соответствующими опубликованными внешними URL-адресами перед их представлением пользователю. 

Для лучшей производительности рекомендуется использовать идентичные внутренние и внешние URL-адреса, настроив [Личные домены](./application-proxy-configure-custom-domain.md). Если использование пользовательских доменов невозможно, можно улучшить производительность преобразования ссылок с помощью расширения для безопасного входа в мои приложения или браузера Microsoft ребра на мобильных устройствах. См. раздел [Перенаправление жестко закодированных ссылок для приложений, опубликованных с помощью Azure AD application proxy](application-proxy-configure-hard-coded-link-translation.md).

## <a name="wildcards"></a>Знаки подстановки

### <a name="how-do-i-use-wildcards-to-publish-two-applications-with-the-same-custom-domain-name-but-with-different-protocols-one-for-http-and-one-for-https"></a>Разделы справки использовать подстановочные знаки для публикации двух приложений с одним и тем же именем личного домена, но с разными протоколами, одним для HTTP и одним для HTTPS?

Этот сценарий не поддерживается напрямую. Параметры для этого сценария:

1. Опубликуйте URL-адреса HTTP и HTTPS как отдельные приложения с подстановочным знаком, но присвойте каждому из них другой личный домен. Эта конфигурация будет работать, так как у них разные внешние URL-адреса.

2. Опубликуйте URL-адрес HTTPS с помощью приложения с подстановочными знаками. Опубликуйте приложения HTTP отдельно с помощью этих командлетов PowerShell для прокси приложения:
   - [Управление приложениями прокси приложения](/powershell/module/azuread/#application_proxy_application_management&preserve-view=true)
   - [Управление соединителем прокси приложения](/powershell/module/azuread/#application_proxy_connector_management&preserve-view=true)
