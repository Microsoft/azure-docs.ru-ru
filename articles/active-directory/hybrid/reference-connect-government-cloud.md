---
title: 'Azure AD Connect выполняет следующие функции: Рекомендации по гибридной идентификации для облака Azure для государственных организаций'
description: Специальные рекомендации по развертыванию Azure AD Connect в облаке Azure для государственных организаций.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/14/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: bbb4298d0d9a9d7589c4a2055b4d55a0b852f951
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97504361"
---
# <a name="hybrid-identity-considerations-for-the-azure-government-cloud"></a>Рекомендации по гибридной идентификации в облаке Azure для государственных организаций

В этой статье приведены рекомендации по интеграции гибридной среды с облаком Microsoft Azure для государственных организаций. Эти сведения предоставляются как справочные материалы для администраторов и архитекторов, работающих с облаком Azure для государственных организаций.

> [!NOTE]
> Чтобы интегрировать среду Microsoft Active Directory (локальную или размещенную в IaaS, которая является частью того же облачного экземпляра) с облаком Azure для государственных организаций, необходимо выполнить обновление до последней версии [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594).

Полный список оконечных точек в Министерстве обороны США см. в [документации](/office365/enterprise/office-365-u-s-government-dod-endpoints).

## <a name="azure-ad-pass-through-authentication"></a>Сквозная аутентификация Azure AD

Далее приведены сведения о реализации сквозной проверки подлинности и облака Azure для государственных организаций.

### <a name="allow-access-to-urls"></a>Разрешение доступа к URL-адресам

Перед развертыванием агента сквозной проверки подлинности проверьте, установлен ли брандмауэр между вашими серверами и Azure AD. Если брандмауэр или прокси-сервер разрешает использовать защищенные программы или программы, заблокированные службой доменных имен (DNS), добавьте следующие подключения.

> [!NOTE]
> Следующие рекомендации также применимы к установке [соединителя Azure AD Application Proxy](../manage-apps/what-is-application-proxy.md) для сред Azure для государственных организаций.

|URL-адрес |Как он используется|
|-----|-----|
|&#42;.msappproxy.us</br>&#42;.servicebus.usgovcloudapi.net|Агент использует эти URL-адреса для взаимодействия с облачной службой Azure AD. |
|`mscrl.microsoft.us:80` </br>`crl.microsoft.us:80` </br>`ocsp.msocsp.us:80` </br>`www.microsoft.us:80`| Агент использует эти URL-адреса для проверки сертификатов.|
|login.windows.us </br>secure.aadcdn.microsoftonline-p.com </br>&#42;.microsoftonline.us </br>&#42;.microsoftonline-p.us </br>&#42;.msauth.net </br>&#42;.msauthimages.net </br>&#42;.msecnd.net</br>&#42;.msftauth.net </br>&#42;.msftauthimages.net</br>&#42;.phonefactor.net </br>enterpriseregistration.windows.net</br>management.azure.com </br>policykeyservice.dc.ad.msft.net</br>ctldl.windowsupdate.us:80| Агент использует эти URL-адреса в процессе регистрации.

### <a name="install-the-agent-for-the-azure-government-cloud"></a>Установка агента для облака Azure для государственных организаций

Выполните следующие действия, чтобы установить агент для облака Azure для государственных организаций.

1. В терминале командной строки перейдите в папку, где хранится исполняемый файл, который устанавливает агент.
1. Выполните следующие команды, которые указывают, что установка выполняется для Azure для государственных организаций.

   Для сквозной проверки подлинности:

   ```
   AADConnectAuthAgentSetup.exe ENVIRONMENTNAME="AzureUSGovernment"
   ```

   Для прокси приложения:

   ```
   AADApplicationProxyConnectorInstaller.exe ENVIRONMENTNAME="AzureUSGovernment" 
   ```

## <a name="single-sign-on"></a>Единый вход

### <a name="set-up-your-azure-ad-connect-server"></a>Настройка сервера Azure AD Connect

Если в качестве метода входа в систему используется сквозная проверка подлинности, дополнительные проверки предварительных требований не требуются. Если в качестве метода входа в систему используется синхронизация хэша паролей и между Azure AD Connect и Azure AD установлен брандмауэр, необходимо проверить соответствие следующим условиям.

- Установлен Azure AD Connect версии 1.1.644.0 или более поздней.
- Если брандмауэр или прокси-сервер разрешает использовать защищенные программы или программы, заблокированные службой доменных имен (DNS), добавьте подключения к URL-адресам &#42;.msappproxy.us через порт 443.

  Если нет, разрешите доступ к диапазонам IP-адресов центра обработки данных Azure, список которых обновляется еженедельно. Это предварительное условие применяется только при включении функции. Его выполнение не обязательно для фактического входа пользователя в систему.

### <a name="roll-out-seamless-single-sign-on"></a>Тестирование простого единого входа

Можно выполнить постепенное развертывание простого единого входа Azure AD для пользователей, соблюдая следующие инструкции. Сначала нужно добавить URL-адрес Azure AD `https://autologon.microsoft.us` в параметры зоны интрасети всех или выбранных пользователей с помощью групповой политики Active Directory.

Кроме того, необходимо включить параметр политики зоны интрасети (с помощью групповой политики), который называется **Разрешить обновление строки состояния в сценарии**.

## <a name="browser-considerations"></a>Рекомендации для браузера

### <a name="mozilla-firefox-all-platforms"></a>Mozilla Firefox (все платформы)

Mozilla Firefox не выполняет аутентификацию Kerberos автоматически. Каждый пользователь должен вручную добавить URL-адрес Azure AD в параметры Firefox, выполнив следующие действия.

1. Запустите Firefox и введите  **about:config** в адресной строке. Проигнорируйте все отображаемые уведомления.
1. Найдите настройку  **network.negotiate-auth.trusted-uris** . Этот параметр выводит список доверенных сайтов в Firefox для проверки подлинности Kerberos.
1. Щелкните правой кнопкой мыши имя параметра и выберите  **Modify** (Изменить).
1. Введите `https://autologon.microsoft.us` в текстовое поле.
1. Нажмите кнопку  **OK**  и вновь откройте браузер.

### <a name="microsoft-edge-based-on-chromium-all-platforms"></a>Microsoft Edge на базе Chromium (все платформы)

Если вы переопределили параметры политики `AuthNegotiateDelegateAllowlist`  или `AuthServerAllowlist` в своей среде, убедитесь, что в них добавлен URL-адрес Azure AD `https://autologon.microsoft.us`.

### <a name="google-chrome-all-platforms"></a>Google Chrome (все платформы)

Если вы переопределили параметры политики `AuthNegotiateDelegateWhitelist`  или `AuthServerWhitelist` в своей среде, убедитесь, что в них добавлен URL-адрес Azure AD `https://autologon.microsoft.us`.

## <a name="next-steps"></a>Дальнейшие действия

- [Сквозная проверка подлинности](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites)
- [Единый вход](how-to-connect-sso-quick-start.md#step-1-check-the-prerequisites)
