---
title: 'Azure AD Connect: устранение проблем с подключением к Azure AD | Документация Майкрософт'
description: Сведения об устранении неполадок подключения в Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 3aa41bb5-6fcb-49da-9747-e7a3bd780e64
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 04/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 56e9820c5e3a750a35b7271b86750df00eb4784e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92677057"
---
# <a name="troubleshoot-azure-ad-connectivity"></a>Устранение неполадок с подключением к Azure AD
В этой статье рассказывается, как работает подключение между Azure AD Connect и Azure AD и как устранять неполадки подключения. Как правило, проблемы возникают в среде с прокси-сервером.

## <a name="troubleshoot-connectivity-issues-in-the-installation-wizard"></a>Устранение неполадок подключения в мастере установки
Azure AD Connect использует для аутентификации современную проверку подлинности (с использованием библиотеки ADAL). Поскольку речь идет о двух приложениях .NET, мастер установки и модуль синхронизации требуют правильной настройки файла machine.config.

В этой статье показано, как Fabrikam подключается к Azure AD через прокси-сервер. Прокси-сервер имеет имя fabrikamproxy и использует порт 8080.

Сначала необходимо убедиться, что [**machine.config**](how-to-connect-install-prerequisites.md#connectivity) настроен правильно и после обновления файла machine.config **Microsoft Azure AD служба синхронизации** была перезапущена.
![На снимке экрана показана часть файла конфигурации Machine Dot.](./media/tshoot-connect-connectivity/machineconfig.png)

> [!NOTE]
> В некоторых сторонних блогах говорится, что вместо него изменения необходимо вносить в файл miiserver.exe.config. Однако при каждом обновлении этот файл перезаписывается, так что даже если при первичной установке все работает, то с первым же обновлением система работать перестанет. По этой причине рекомендуется обновлять файл machine.config.
>
>

На прокси-сервере должен быть также открыт необходимый URL-адрес. Официальный список см. в статье [URL-адреса и диапазоны IP-адресов Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2).

Для этих URL-адресов в приведенной ниже таблице показаны минимальные условия, которые должны соблюдаться для подключения к Azure AD. Этот список не включает дополнительные функции, такие как обратная запись паролей или Azure AD Connect Health. Он предназначен только для поиска и устранения неполадок, связанных с начальной конфигурацией.

| URL-адрес | Порт | Описание |
| --- | --- | --- |
| mscrl.microsoft.com |HTTP/80 |Используется для загрузки списков CRL. |
| \*.verisign.com |HTTP/80 |Используется для загрузки списков CRL. |
| \*.entrust.net |HTTP/80 |Используется для загрузки списков CRL для MFA. |
| \*.windows.net |HTTPS/443 |Используется для входа в Azure AD. |
| secure.aadcdn.microsoftonline-p.com |HTTPS/443 |Используется для многофакторной проверки подлинности (MFA). |
| \*.microsoftonline.com |HTTPS/443 |Используется для настройки каталога Azure AD, а также импорта и экспорта данных. |
| \*. crl3.digicert.com |HTTP/80 |Используется для проверки сертификатов. |
| \*. crl4.digicert.com |HTTP/80 |Используется для проверки сертификатов. |
| \*. ocsp.digicert.com |HTTP/80 |Используется для проверки сертификатов. |
| \*. www.d-trust.net |HTTP/80 |Используется для проверки сертификатов. |
| \*. root-c3-ca2-2009.ocsp.d-trust.net |HTTP/80 |Используется для проверки сертификатов. |
| \*. crl.microsoft.com |HTTP/80 |Используется для проверки сертификатов. |
| \*. oneocsp.microsoft.com |HTTP/80 |Используется для проверки сертификатов. |
| \*. ocsp.msocsp.com |HTTP/80 |Используется для проверки сертификатов. |

## <a name="errors-in-the-wizard"></a>Ошибки в мастере
Мастер установки использует два различных контекста безопасности. На странице **Подключение к Azure AD** он использует имя пользователя, выполнившего вход. На странице **Настройка** он переключается на [учетную запись, под которой работает служба модуля синхронизации](reference-connect-accounts-permissions.md#adsync-service-account). Если возникают проблемы, то обычно они проявляются уже на странице мастера **Подключение к Azure AD**, так как конфигурация прокси-сервера является глобальной.

Ниже приведены наиболее распространенные ошибки, которые встречаются в мастере установки.

### <a name="the-installation-wizard-has-not-been-correctly-configured"></a>Неправильно настроен мастер установки
Эта ошибка появляется в случае, если мастер не может связаться с прокси-сервером.
![На снимке экрана показана ошибка: не удалось проверить учетные данные.](./media/tshoot-connect-connectivity/nomachineconfig.png)

* Если возникает эта ошибка, проверьте конфигурацию в файле [machine.config](how-to-connect-install-prerequisites.md#connectivity).
* Если конфигурация выглядит нормально, выполните действия, описанные в разделе [Проверка подключения прокси-сервера](#verify-proxy-connectivity) , и убедитесь в том, что проблема возникает не только в мастере.

### <a name="a-microsoft-account-is-used"></a>Используется учетная запись Майкрософт
Если использовать **учетную запись Майкрософт** вместо **учебной или рабочей учетной записи**, то возникнет общая ошибка.
![Используется учетная запись Майкрософт](./media/tshoot-connect-connectivity/unknownerror.png)

### <a name="the-mfa-endpoint-cannot-be-reached"></a>Не удается связаться с конечной точкой многофакторной проверки подлинности
Эта ошибка возникает, если конечная точка **https://secure.aadcdn.microsoftonline-p.com** недоступна и для глобального администратора включена поддержка mfa.
![nomachineconfig](./media/tshoot-connect-connectivity/nomicrosoftonlinep.png)

* Если вы видите эту ошибку, то убедитесь, что конечная точка **secure.aadcdn.microsoftonline-p.com** добавлена на прокси-сервер.

### <a name="the-password-cannot-be-verified"></a>Невозможно проверить пароль
Если мастер установки успешно подключается к Azure AD, но не удается проверить пароль, появится сообщение об ошибке " ![ неправильный пароль".](./media/tshoot-connect-connectivity/badpassword.png)

* Проверьте, не используется ли временный пароль, который необходимо сменить. Проверьте, правильно ли указан пароль. Попробуйте войти в систему по адресу `https://login.microsoftonline.com` (на другом компьютере, отличном от сервера Azure AD Connect), и убедитесь, что учетная запись доступна.

### <a name="verify-proxy-connectivity"></a>Проверка подключения прокси-сервера
Для проверки возможности подключения сервера Azure AD Connect к прокси-серверу и Интернету можно использовать некоторые командлеты PowerShell, показывающие, пропускает ли прокси-сервер веб-запросы. В командной строке PowerShell выполните командлет `Invoke-WebRequest -Uri https://adminwebservice.microsoftonline.com/ProvisioningService.svc`. (С технической точки зрения первый вызов направляется по адресу `https://login.microsoftonline.com`, и этот URI также будет работать, хотя другие URI отвечают быстрее.)

Для связи с прокси-сервером PowerShell использует конфигурацию в файле machine.config. Параметры в winhttp/netsh не должны влиять на эти командлеты.

Если прокси-сервер настроен правильно, вы должны получить состояние успеха: ![ снимок экрана, показывающий состояние успеха при правильной настройке прокси-сервера.](./media/tshoot-connect-connectivity/invokewebrequest200.png)

Если вы получаете команду **не удается подключиться к удаленному серверу**, то PowerShell пытается выполнить прямой вызов без использования прокси-сервера или неправильно настроенной службы DNS. Убедитесь, что файл **machine.config** настроен правильно.
![unabletoconnect](./media/tshoot-connect-connectivity/invokewebrequestunable.png)

Если прокси-сервер настроен неправильно, то появляется ошибка: ![proxy200](./media/tshoot-connect-connectivity/invokewebrequest403.png)
![proxy407](./media/tshoot-connect-connectivity/invokewebrequest407.png).

| Ошибка | Текст сообщения об ошибке | Комментировать |
| --- | --- | --- |
| 403 |Запрещено |Прокси-сервер не был открыт для запрошенного URL-адреса. Проверьте конфигурацию прокси-сервера и убедитесь, что [URL-адреса](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) открыты. |
| 407 |Требуется проверка подлинности прокси-сервера |Для прокси-сервера требуется имя входа, которое не было указано. Если для прокси-сервера требуется проверка подлинности, убедитесь, что этот параметр настроен в machine.config. Также убедитесь, что вы используете учетные записи домена для пользователя, запустившего мастер, и для учетной записи службы. |

### <a name="proxy-idle-timeout-setting"></a>Параметр времени ожидания прокси-сервера в режиме простоя
Когда Azure AD Connect отправляет запрос на экспорт в службу Azure AD, службе Azure AD может потребоваться около 5 минут на обработку запроса и создание ответа. Это особенно актуально в случаях, когда один запрос на экспорт содержит несколько объектов групп с большим количеством участников. Убедитесь, что время ожидания прокси-сервера в режиме простоя превышает 5 минут. В противном случае на сервере Azure AD Connect могут наблюдаться перебои с подключением к Azure AD.

## <a name="the-communication-pattern-between-azure-ad-connect-and-azure-ad"></a>Шаблон взаимодействия между Azure AD Connect и Azure AD
Если вы выполнили все описанные выше действия, а подключение по-прежнему невозможно, то обратитесь к журналам сети. В этом разделе описан нормальный, работающий шаблон подключения, а также перечислены распространенные ложные сигналы, которые при чтении журналов сети можно игнорировать.

* Есть вызовы по адресу `https://dc.services.visualstudio.com`. Для успешной установки открытие этого URL-адреса в прокси-сервере не требуется, и эти вызовы можно игнорировать.
* Вы заметите, что в разрешении DNS указано, что фактически узлы находятся в пространстве DNS-имен nsatc.net и других, а не в microsoftonline.com. Фактические имена серверов в запросах к веб-службам не указываются, и добавлять эти URL-адреса в прокси-сервер не нужно.
* Конечные точки adminwebservice и provisioningapi представляют собой конечные точки обнаружения и используются для поиска фактически используемой конечной точки. Выбор этих конечных точек зависит от вашего региона.

### <a name="reference-proxy-logs"></a>Справочные журналы прокси-сервера
Приведем дамп из действительного журнала прокси-сервера и страницу мастера установки, с которой он был взят (дублирующиеся записи, относящиеся к одной и той же конечной точке, были удалены). Этот раздел можно использовать как образец для собственных журналов прокси-сервера и сети. В вашей среде конечные точки могут отличаться (особенно URL-адреса, выделенные *курсивом*).

**Подключение к Azure AD**

| Time | URL-адрес |
| --- | --- |
| 1/11/2016 8:31 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:31 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:32 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:32 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:33 |connect://provisioningapi.microsoftonline.com:443 |
| 1/11/2016 8:33 |connect://*bwsc02-relay*.microsoftonline.com:443 |

**Настройка**

| Time | URL-адрес |
| --- | --- |
| 1/11/2016 8:43 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:43 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:43 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://*bba900-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://*bba800-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:44 |connect://login.microsoftonline.com:443 |
| 1/11/2016 8:46 |connect://provisioningapi.microsoftonline.com:443 |
| 1/11/2016 8:46 |connect://*bwsc02-relay*.microsoftonline.com:443 |

**Начальная синхронизация**

| Time | URL-адрес |
| --- | --- |
| 1/11/2016 8:48 |connect://login.windows.net:443 |
| 1/11/2016 8:49 |connect://adminwebservice.microsoftonline.com:443 |
| 1/11/2016 8:49 |connect://*bba900-anchor*.microsoftonline.com:443 |
| 1/11/2016 8:49 |connect://*bba800-anchor*.microsoftonline.com:443 |

## <a name="authentication-errors"></a>Ошибки проверки подлинности
В этом разделе описываются ошибки, которые могут возвращаться ADAL (библиотекой аутентификации, используемой службой Azure AD Connect) и PowerShell. Объяснение ошибки поможет понять причины ее возникновения и определить дальнейшие действия.

### <a name="invalid-grant"></a>Недопустимый набор прав
Недопустимое имя пользователя или пароль. Дополнительные сведения см. в разделе [Невозможно проверить пароль](#the-password-cannot-be-verified).

### <a name="unknown-user-type"></a>Неизвестный тип пользователя
Невозможно найти или разрешить каталог Azure AD. Возможно, при попытке входа в систему вы используете имя пользователя в непроверенном домене?

### <a name="user-realm-discovery-failed"></a>Не удалось выполнить обнаружение области пользователя
Проблемы конфигурации сети или прокси-сервера. Сеть недоступна. Ознакомьтесь с разделом [Устранение неполадок подключения в мастере установки](#troubleshoot-connectivity-issues-in-the-installation-wizard).

### <a name="user-password-expired"></a>Срок действия пароля пользователя истек
Срок действия ваших учетных данных истек. Измените пароль.

### <a name="authorization-failure"></a>"Authorization Failure" (Сбой авторизации)
Не удалось авторизовать пользователя для выполнения действия в Azure AD.

### <a name="authentication-canceled"></a>Проверка подлинности отменена
Запрос многофакторной проверки подлинности (MFA) был отменен.

<div id="connect-msolservice-failed">
<!--
  Empty div just to act as an alias for the "Connect To MS Online Failed" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="connect-to-ms-online-failed"></a>"Connect To MS Online Failed" (Не удалось подключиться к Microsoft Online)
Проверка подлинности прошла успешно, но есть проблема с проверкой подлинности в Azure AD PowerShell.

<div id="get-msoluserrole-failed">
<!--
  Empty div just to act as an alias for the "Azure AD Global Admin Role Needed" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="azure-ad-global-admin-role-needed"></a>"Azure AD Global Admin Role Needed" (Требуется роль глобального администратора Azure AD)
Пользователь успешно прошел аутентификацию. Однако ему не назначена роль глобального администратора. Вот [как можно назначить роль глобального администратора](../roles/permissions-reference.md) пользователю.

<div id="privileged-identity-management">
<!--
  Empty div just to act as an alias for the "Privileged Identity Management Enabled" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="privileged-identity-management-enabled"></a>"Privileged Identity Management Enabled" (Включено управление привилегированными пользователями)
Проверка подлинности прошла успешно. Было включено управление привилегированными пользователями, и в настоящее время вы не являетесь глобальным администратором. Дополнительные сведения см. в статье [Приступая к работе с управлением привилегированными пользователями Azure AD](../privileged-identity-management/pim-getting-started.md).

<div id="get-msolcompanyinformation-failed">
<!--
  Empty div just to act as an alias for the "Company Information Unavailable" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="company-information-unavailable"></a>"Company Information Unavailable" (Сведения об организации недоступны)
Проверка подлинности прошла успешно. Не удалось получить от Azure AD сведения об организации.

<div id="get-msoldomain-failed">
<!--
  Empty div just to act as an alias for the "Domain Information Unavailable" header
  because we used the mentioned id in the code to jump to this section.
-->
</div>

### <a name="domain-information-unavailable"></a>"Domain Information Unavailable" (Сведения о домене недоступны)
Проверка подлинности прошла успешно. Не удалось получить от Azure AD сведения о домене.

### <a name="unspecified-authentication-failure"></a>"Unspecified Authentication Failure" (Неопределенная ошибка аутентификации)
Отображается как непредвиденная ошибка в мастере установки. Она может возникнуть при попытке использовать **учетную запись Майкрософт** вместо **учебной или рабочей учетной записи**.

## <a name="troubleshooting-steps-for-previous-releases"></a>Действия по устранению неполадок для предыдущих версий.
Начиная с версии с номером сборки 1.1.105.0 (выпущенной в феврале 2016 г.) помощник по входу более не используется. Этот раздел и конфигурация больше не требуются, но хранятся для ссылки.

Для работы помощника по единому входу необходимо настроить winhttp. Эту настройку можно сделать с помощью [**netsh**](how-to-connect-install-prerequisites.md#connectivity).
![На снимке экрана показано окно командной строки, в котором запускается средство Netsh для настройки прокси-сервера.](./media/tshoot-connect-connectivity/netsh.png)

### <a name="the-sign-in-assistant-has-not-been-correctly-configured"></a>Неправильно настроен помощник по входу
Эта ошибка появляется, если помощник по входу не может связаться с прокси-сервером или прокси-сервер не пропускает запрос.
![На снимке экрана показана ошибка: не удается проверить учетные данные, проверьте сетевое подключение и параметры брандмауэра или прокси-сервера.](./media/tshoot-connect-connectivity/nonetsh.png)

* Если возникает эта ошибка, проверьте конфигурацию прокси-сервера в [netsh](how-to-connect-install-prerequisites.md#connectivity) и убедитесь в правильности ее настройки.
  ![На снимке экрана показано окно командной строки, в котором запускается средство Netsh для отображения конфигурации прокси-сервера.](./media/tshoot-connect-connectivity/netshshow.png)
* Если конфигурация выглядит нормально, выполните действия, описанные в разделе [Проверка подключения прокси-сервера](#verify-proxy-connectivity) , и убедитесь в том, что проблема возникает не только в мастере.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
