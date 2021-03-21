---
title: Включение удаленного доступа к SharePoint AD Application Proxy Azure
description: Рассматриваются основные сведения об интеграции локального сервера SharePoint с прокси приложения Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/02/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5b5895d016e2d9d9b471218bc083ea7585254b45
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99258687"
---
# <a name="enable-remote-access-to-sharepoint-with-azure-ad-application-proxy"></a>Настройка удаленного доступа к SharePoint с помощью прокси приложения Azure AD

В этом пошаговом руководство объясняется, как интегрировать локальную ферму SharePoint с прокси приложения Azure Active Directory (Azure AD).

## <a name="prerequisites"></a>Предварительные условия

Для выполнения настройки необходимы следующие ресурсы:
- Ферма SharePoint 2013 или более поздней версии.
- Клиент Azure AD с планом, который включает прокси приложения. Узнайте больше о [планах и ценах Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).
- [Пользовательский проверенный домен](../fundamentals/add-custom-domain.md) в клиенте Azure AD.
- Локальные Active Directory синхронизируются с Azure AD Connect, с помощью которых пользователи могут [входить в Azure](../hybrid/plan-connect-user-signin.md).
- Соединитель прокси приложения установлен и работает на компьютере в корпоративном домене.

Для настройки SharePoint с помощью прокси приложения требуется два URL-адреса:
- Внешний URL-адрес, видимый для конечных пользователей и определенный в Azure AD. Этот URL-адрес может использовать личный домен. Дополнительные сведения о [работе с пользовательскими доменами в Azure AD application proxy](application-proxy-configure-custom-domain.md).
- Внутренний URL-адрес, известный только в корпоративном домене и не использующийся напрямую.

> [!IMPORTANT]
> Чтобы убедиться, что ссылки сопоставлены правильно, следуйте следующим рекомендациям по внутреннему URL-адресу:
> - Использовать HTTPS.
> - Не используйте настраиваемые порты.
> - В корпоративной системе доменных имен (DNS) создайте узел (A), указывающий на WFE SharePoint (или подсистему балансировки нагрузки), а не псевдоним (CName).

В этой статье используются следующие значения:
- Внутренний URL-адрес: `https://sharepoint`
- Внешний URL-адрес: `https://spsites-demo1984.msappproxy.net/`
- Учетная запись пула приложений для веб-приложения SharePoint: `Contoso\spapppool`

## <a name="step-1-configure-an-application-in-azure-ad-that-uses-application-proxy"></a>Шаг 1. Настройка приложения в Azure AD, использующего прокси приложения

На этом шаге вы создадите приложение в клиенте Azure Active Directory, использующем прокси приложения. Задайте внешний URL-адрес и укажите внутренний URL-адрес, который будет использоваться позже в SharePoint.

1. Создайте приложение, как описано в следующих параметрах. Пошаговые инструкции см. в статье [Публикация приложений с помощью прокси приложения Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).
   * **Внутренний URL-адрес**: внутренний URL-адрес SharePoint, который будет задан позже в SharePoint, например `https://sharepoint` .
   * **Предварительная проверка подлинности**: Azure Active Directory
   * **Перевод URL-адресов в заголовки**: нет
   * **Перевод URL-адресов в тексте приложения**: нет

   ![Публикация SharePoint в качестве приложения](./media/application-proxy-integrate-with-sharepoint-server/publish-app.png)

1. После публикации приложения выполните следующие действия, чтобы настроить параметры единого входа.

   1. На странице приложения на портале выберите **Единый вход**.
   1. Для **единого входа** выберите режим **Встроенная проверка подлинности Windows**.
   1. Задайте для **внутреннего субъекта-службы приложения** значение, заданное ранее. В этом примере значение равно `HTTP/sharepoint` .
   1. В разделе **делегированное удостоверение входа** выберите наиболее подходящий вариант для конфигурации леса Active Directory. Например, если в лесу имеется один Active Directory домен, выберите **Локальное имя учетной записи SAM** (как показано на следующем снимке экрана). Но если пользователи не находятся в том же домене, что и серверы соединителя для прокси-сервера приложений, выберите **Локальное имя участника-пользователя** (не показано на снимке экрана).

   ![Настройка встроенной проверки подлинности Windows для единого входа](./media/application-proxy-integrate-with-sharepoint-server/configure-iwa.png)

1. Чтобы завершить настройку приложения, перейдите к разделу **Пользователи и группы** и назначьте доступ к этому приложению необходимым пользователям. 

## <a name="step-2-configure-the-sharepoint-web-application"></a>Шаг 2. Настройка веб-приложения SharePoint

Веб-приложение SharePoint должно быть настроено с использованием протокола Kerberos и соответствующих сопоставлений альтернативного доступа для правильной работы с Azure AD Application Proxy. Существует два возможных варианта.

- Создайте новое веб-приложение и используйте только зону по умолчанию. Это предпочтительный вариант, так как он обеспечивает оптимальную работу с SharePoint (например, ссылки в оповещениях по электронной почте, созданные SharePoint, всегда указывают на зону по умолчанию).
- Расширение существующего веб-приложения для настройки Kerberos в зоне, отличной от зоны по умолчанию.

> [!IMPORTANT]
> Независимо от используемой зоны, учетная запись пула приложений веб-приложения SharePoint должна быть учетной записью домена для правильной работы Kerberos.

### <a name="provision-the-sharepoint-web-application"></a>Подготавливает веб-приложение SharePoint

- При создании нового веб-приложения и использовании только зоны по умолчанию (предпочтительный вариант):

    1. Запустите **консоль управления SharePoint** и выполните следующий скрипт:

       ```powershell
       # This script creates a web application and configures the Default zone with the internal/external URL needed to work with Azure AD Application Proxy
       # Edit variables below to fit your environment. Note that the managed account must exist and it must be a domain account
       $internalUrl = "https://sharepoint"
       $externalUrl = "https://spsites-demo1984.msappproxy.net/"
       $applicationPoolManagedAccount = "Contoso\spapppool"
            
       $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$false
       $wa = New-SPWebApplication -Name "SharePoint - AAD Proxy" -Port 443 -SecureSocketsLayer -URL $externalUrl -ApplicationPool "SharePoint - AAD Proxy" -ApplicationPoolAccount (Get-SPManagedAccount $applicationPoolManagedAccount) -AuthenticationProvider $winAp
       New-SPAlternateURL -Url $internalUrl -WebApplication $wa -Zone Default -Internal
       ```

    2. Откройте страницу **центра администрирования SharePoint**.
    1. В разделе **Параметры системы** выберите **Настройка сопоставлений альтернативного доступа**. Откроется окно **коллекция альтернативного сопоставления доступа** .
    1. Отфильтруйте экран с помощью нового веб-приложения и убедитесь, что вы видите нечто вроде этого:

       ![Сопоставления альтернативного доступа веб-приложения](./media/application-proxy-integrate-with-sharepoint-server/new-webapp-aam.png)

- При расширении существующего веб-приложения в новую зону (если вы не можете использовать зону по умолчанию):

    1. Запустите консоль управления SharePoint и выполните следующий скрипт:

       ```powershell
       # This script extends an existing web application to Internet zone with the internal/external URL needed to work with Azure AD Application Proxy
       # Edit variables below to fit your environment
       $webAppUrl = "http://spsites/"
       $internalUrl = "https://sharepoint"
       $externalUrl = "https://spsites-demo1984.msappproxy.net/"
       
       $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$false
       $wa = Get-SPWebApplication $webAppUrl
       New-SPWebApplicationExtension -Name "SharePoint - AAD Proxy" -Identity $wa -SecureSocketsLayer -Zone Extranet -Url $externalUrl -AuthenticationProvider $winAp
       New-SPAlternateURL -Url $internalUrl -WebApplication $wa -Zone Extranet -Internal
       ```

    2. Откройте страницу **центра администрирования SharePoint**.
    1. В разделе **Параметры системы** выберите **Настройка сопоставлений альтернативного доступа**. Откроется окно **коллекция альтернативного сопоставления доступа** .
    1. Отфильтруйте отображение веб-приложения, которое было расширено, и убедитесь, что вы видите нечто вроде этого:

        ![Сопоставления альтернативного доступа расширенного приложения](./media/application-proxy-integrate-with-sharepoint-server/extend-webapp-aam.png)

### <a name="make-sure-the-sharepoint-web-application-is-running-under-a-domain-account"></a>Убедитесь, что веб-приложение SharePoint работает под учетной записью домена.

Чтобы узнать учетную запись, выполняющую пул приложений веб-приложения SharePoint, и убедиться, что это учетная запись домена, выполните следующие действия.

1. Откройте страницу **центра администрирования SharePoint**.
1. Щелкните **Безопасность** и выберите **Настройка учетных записей служб**.
1. Выберите **пул веб-приложений — йоурвебаппликатионнаме**.

   ![Параметры для настройки учетной записи службы](./media/application-proxy-integrate-with-sharepoint-server/service-web-application.png)

1. Убедитесь, что **параметр выбрать учетную запись для этого компонента** Возвращает учетную запись домена, и помните, так как он потребуется на следующем шаге.

### <a name="make-sure-that-an-https-certificate-is-configured-for-the-iis-site-of-the-extranet-zone"></a>Убедитесь, что HTTPS-сертификат настроен для сайта IIS зоны экстрасети.

Так как внутренний URL-адрес использует протокол HTTPS ( `https://SharePoint/` ), на сайте службы IIS (IIS) должен быть установлен сертификат.

1. Откройте консоль Windows PowerShell.
1. Выполните следующий сценарий, чтобы создать самозаверяющий сертификат и добавить его в мое хранилище компьютера:

   ```powershell
   # Replace "SharePoint" with the actual hostname of the Internal URL of your Azure AD proxy application
   New-SelfSignedCertificate -DnsName "SharePoint" -CertStoreLocation "cert:\LocalMachine\My"
   ```

   > [!IMPORTANT]
   > Самозаверяющие сертификаты допустимо использовать только для целей тестирования. В рабочих средах настоятельно рекомендуется использовать сертификаты, выданные центром сертификации.

1. Откройте консоль службы IIS Manager.
1. Разверните сервер в представлении в виде дерева, разверните узел **сайты**, выберите сайт **SharePoint-прокси-сервер AAD** и щелкните **привязки**.
1. Выберите **Привязка HTTPS** и нажмите кнопку **изменить**.
1. В поле сертификат TLS/SSL выберите сертификат **SharePoint** и нажмите кнопку **ОК**.

Теперь вы можете получить доступ к сайту SharePoint извне с помощью Azure AD Application Proxy.

## <a name="step-3-configure-kerberos-constrained-delegation"></a>Шаг 3. Настройка ограниченного делегирования Kerberos

Пользователи будут сначала проходить проверку подлинности в Azure AD, а затем к SharePoint с помощью протокола Kerberos через соединитель прокси Azure AD. Чтобы разрешить соединителю получать маркер Kerberos от имени пользователя Azure AD, необходимо настроить ограниченное делегирование Kerberos (KCD) с использованием смены протокола. Дополнительные сведения о KCD см. в разделе [Общие сведения о ограниченном делегировании Kerberos](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj553400(v=ws.11)).

### <a name="set-the-spn-for-the-sharepoint-service-account"></a>Задание имени субъекта-службы для учетной записи службы SharePoint

В этой статье внутренний URL-адрес имеет значение `https://sharepoint` , и поэтому имя участника-службы (SPN) — `HTTP/sharepoint` . Эти значения необходимо заменить значениями, соответствующими вашей среде.
Чтобы зарегистрировать имя участника-службы `HTTP/sharepoint` для учетной записи пула приложений SharePoint `Contoso\spapppool` , выполните следующую команду в командной строке в качестве администратора домена:

`setspn -S HTTP/sharepoint Contoso\spapppool`

`Setspn`Команда выполняет поиск имени субъекта-службы перед его добавлением. Если имя субъекта-службы уже существует, вы увидите **повторяющееся значение имени SPN** . В этом случае рекомендуется удалить существующее имя субъекта-службы, если оно не задано в правильной учетной записи пула приложений. Чтобы убедиться, что имя субъекта-службы успешно добавлено, выполнив `Setspn` команду с параметром – L. Дополнительные сведения о команде setspn см. в [этой статье](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc731241(v=ws.11)).

### <a name="make-sure-the-connector-is-trusted-for-delegation-to-the-spn-that-was-added-to-the-sharepoint-application-pool-account"></a>Убедитесь, что соединитель является доверенным для делегирования имени субъекта-службы, которое было добавлено в учетную запись пула приложений SharePoint.

Настройте ограниченное делегирование Kerberos, чтобы служба прокси приложения Azure AD могла делегировать удостоверения пользователей для учетной записи пула приложений SharePoint. Для этого настройте для соединителя прокси приложения возможность получать билеты Kerberos для пользователей, которые прошли проверку подлинности в Azure AD. Затем этот сервер передает контекст целевому приложению (в данном случае SharePoint).

Чтобы настроить KCD, выполните следующие действия для каждого компьютера соединителя:

1. Войдите на контроллер домена в качестве администратора домена, а затем откройте Active Directory пользователи и компьютеры.
1. Найдите компьютер, на котором работает соединитель прокси-сервера Azure AD. В этом примере это сам сервер SharePoint.
1. Дважды щелкните имя компьютера, а затем перейдите на вкладку **Делегирование**.
1. Убедитесь, что для параметров делегирования задано значение **доверять компьютеру делегирование указанных служб**. Затем выберите **использовать любой протокол проверки подлинности**.
1. Нажмите кнопку **Добавить** , выберите **Пользователи или компьютеры** и укажите учетную запись пула приложений SharePoint. Например, `Contoso\spapppool`.
1. В списке имен участников-служб выберите того, который вы создали ранее для учетной записи службы.
1. Нажмите кнопку **ОК** , а затем еще раз нажмите кнопку **ОК** , чтобы сохранить изменения.
  
   ![Параметры делегирования](./media/application-proxy-integrate-with-sharepoint-server/delegation-box2.png)

Теперь все готово для входа в SharePoint с использованием внешнего URL-адреса и проверки подлинности в Azure.

## <a name="troubleshoot-sign-in-errors"></a>Устранение ошибок входа

Если вход на сайт не работает, можно получить дополнительные сведения о данной ошибке в журналах соединителя: с компьютера, на котором работает соединитель, открыть средство просмотра событий, перейти к разделу **журналы приложений и служб**  >  **Microsoft**  >  **AadApplicationProxy**  >  **Connector** и проверить журнал **администратора** .

## <a name="next-steps"></a>Дальнейшие действия

* [Работа с пользовательскими доменами в прокси приложения Azure AD](application-proxy-configure-custom-domain.md)
* [Сведения о соединителях прокси приложения Azure AD](application-proxy-connectors.md)