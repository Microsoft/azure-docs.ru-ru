---
title: Настройка гибридного присоединения к Azure Active Directory для федеративных доменов | Документация Майкрософт
description: Сведения о том, как настроить гибридное присоединение к Azure Active Directory для федеративных доменов.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 05/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 221b7bdbb8ab5d0121e9c8032be8f18d8ae60d1e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578062"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-join-for-federated-domains"></a>Руководство по Настройка гибридного присоединения к Azure Active Directory для федеративных доменов

Подобно пользователю в вашей организации, устройство — это еще одно основное удостоверение, которое необходимо защитить. Удостоверение устройства можно использовать для защиты ресурсов в любое время и из любого расположения. Для этого разместите удостоверения своих устройств в Azure AD и управляйте ими с помощью одного из следующих методов:

- присоединение к Azure AD;
- гибридное присоединение к Azure AD;
- регистрация в Azure AD.

Размещение устройств в Azure AD позволяет увеличить эффективность работы пользователей благодаря выполнению единого входа для облачных и локальных ресурсов. В то же время вы можете защитить доступ к облачным и локальным ресурсам с помощью [условного доступа](../conditional-access/howto-conditional-access-policy-compliant-device.md).

В федеративной среде должен быть поставщик удостоверений, поддерживающий следующие требования. Это не относится к федеративной среде, в которой используются службы федерации Active Directory (AD FS).

- **Утверждение WIAORMULTIAUTHN:** это утверждение требуется для гибридного присоединения к Azure AD устройств Windows нижнего уровня.
- **Протокол WS-Trust:** этот протокол требуется для аутентификации в Azure AD устройств Windows с текущим гибридным присоединением к Azure AD.
  При использовании AD FS нужно включить следующие конечные точки WS-Trust: `/adfs/services/trust/2005/windowstransport`
   `/adfs/services/trust/13/windowstransport`
   `/adfs/services/trust/2005/usernamemixed`
   `/adfs/services/trust/13/usernamemixed`
   `/adfs/services/trust/2005/certificatemixed`
   `/adfs/services/trust/13/certificatemixed` 

> [!WARNING] 
> Также нужно включить **adfs/services/trust/2005/windowstransport** и **adfs/services/trust/13/windowstransport**, но только в качестве конечных точек с подключением к интрасети. Их НЕЛЬЗЯ предоставлять как конечные точки с подключением к экстрасети через прокси-сервер веб-приложения. Дополнительные сведения см. в статье об [отключении конечных точек WS-Trust в Windows на прокси-сервере](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs#disable-ws-trust-windows-endpoints-on-the-proxy-ie-from-extranet). В разделе **Служба** > **Конечные точки** вы можете увидеть, какие конечные точки активированы в консоли управления AD FS.

В этом руководстве описано, как настроить гибридное присоединение к Azure AD для компьютеров, подключенных к домену Active Directory в федеративной среде, с помощью AD FS.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Настройка гибридного присоединения к Azure AD.
> * Включение устройств Windows нижнего уровня
> * Проверка регистрации
> * Диагностика

## <a name="prerequisites"></a>Предварительные требования

В данном руководстве предполагается, что вы ознакомлены со следующими статьями:

- [Что такое удостоверение устройства?](overview.md)
- [How to plan your hybrid Azure AD join implementation](hybrid-azuread-join-plan.md) (Как планировать реализацию гибридного присоединения к Azure AD)
- [Как выполнить контролируемую проверку гибридного присоединения к Azure AD](hybrid-azuread-join-control.md)

Для настройки сценария в этом руководстве вам понадобится следующее.

- Windows Server 2012 R2 с AD FS.
- [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) версии 1.1.819.0 или более поздней.

Начиная с версии 1.1.819.0, Azure AD Connect предоставляет мастер для настройки гибридного присоединения к Azure AD. Этот мастер значительно упрощает настройку. Связанный мастер:

- настраивает точки подключения службы (SCP) для регистрации устройств;
- создает резервную копию существующего отношения доверия с проверяющей стороной Azure AD;
- обновляет правила утверждений в вашем отношении доверия Azure AD.

Действия по настройке в этой статье основаны на использовании мастера Azure AD Connect. Если у вас установлена более ранняя версия Azure AD Connect, для использования мастера ее нужно обновить до версии 1.1.819 или более поздней версии. Если по каким-либо причинам вы не можете установить последнюю версию Azure AD Connect, узнайте, [как настроить гибридное присоединение к Azure AD вручную](hybrid-azuread-join-manual.md).

Для гибридного присоединения к Azure AD требуется, чтобы у устройств был доступ к следующим ресурсам Майкрософт из сети организации:  

- `https://enterpriseregistration.windows.net`
- `https://login.microsoftonline.com`
- `https://device.login.microsoftonline.com`
- Служба токенов безопасности вашей организации (STS) (для федеративных доменов).
- `https://autologon.microsoftazuread-sso.com` (если вы используете или планируете использовать простой единый вход).

> [!WARNING]
> Если ваша организация использует прокси-серверы, которые перехватывают трафик SSL для таких сценариев, как защита от потери данных или ограничения арендатора Azure AD, убедитесь, что трафик к https://device.login.microsoftonline.com исключается из процесса приостановки и изучения TLS-трафика. Если адрес https://device.login.microsoftonline.com не будет исключен, это может вызвать ошибки при аутентификации с помощью сертификата клиента, что приведет к проблемам с регистрацией устройств и условным доступом на основе устройств.

Начиная с Windows 10 версии 1803, если происходит сбой мгновенного гибридного присоединения к Azure AD для федеративной среды с помощью AD FS, мы применяем Azure AD Connect для синхронизации объекта-компьютера в Azure AD. Затем с его помощью завершается регистрация устройств для такого гибридного присоединения. Проверьте, синхронизированы ли в Azure AD Connect объекты-компьютеры гибридных устройств, которые нужно присоединить к Azure AD. Если объекты-компьютеры принадлежат конкретным подразделениям, эти подразделения также нужно настроить для синхронизации в Azure AD Connect. Дополнительные сведения о том, как синхронизировать объекты-компьютеры с помощью Azure AD Connect, см. в статье о [настройке фильтрации с помощью Azure AD Connect](../hybrid/how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering).

> [!NOTE]
> Чтобы синхронизировать регистрацию устройства, при настройке регистрации устройства не исключайте атрибуты устройств по умолчанию из конфигурации синхронизации Azure AD Connect. Дополнительные сведения об атрибутах устройств по умолчанию, синхронизированных с AAD, см. в разделе [Windows 10](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-sync-attributes-synchronized#windows-10).

Если вашей организации требуется доступ к Интернету через исходящий прокси-сервер, корпорация Майкрософт рекомендует [реализовать автоматическое обнаружение веб-прокси (WPAD)](/previous-versions/tn-archive/cc995261(v%3dtechnet.10)), чтобы обеспечить регистрацию устройств в Azure AD на компьютерах Windows 10. Если возникли проблемы при настройке и управлении WPAD, обратитесь к статье об [устранении неполадок с автоматическим обнаружением](/previous-versions/tn-archive/cc302643(v=technet.10)). 

Если вы не используете WPAD и хотите настроить параметры прокси-сервера на компьютере, это можно сделать, начиная с Windows 10 версии 1709. Дополнительные сведения см. в записи блога о [настройке параметров WinHTTP с использованием объекта групповой политики](/archive/blogs/netgeeks/winhttp-proxy-settings-deployed-by-gpo).

> [!NOTE]
> Если вы настроите параметры прокси-сервера на компьютере с помощью параметров WinHTTP, любые компьютеры, которым не удается подключиться к настроенному прокси-серверу, не смогут подключиться к Интернету.

Если вашей организации требуется доступ в Интернет через аутентифицированный исходящий прокси-сервер, вы должны убедиться, что ваши компьютеры с Windows 10 могут успешно пройти аутентификацию на этом прокси-сервере. Так как компьютеры с Windows 10 выполняют регистрацию устройства с помощью контекста компьютера, необходимо настроить аутентификацию исходящего прокси-сервера с использованием контекста компьютера. Обратитесь к поставщику исходящего прокси-сервера, чтобы узнать требования к конфигурации.

Проверить, имеет ли устройство доступ к указанным выше ресурсам Майкрософт под системной учетной записью, можно с помощью [скрипта для проверки подключения при регистрации устройств](/samples/azure-samples/testdeviceregconnectivity/testdeviceregconnectivity/).

## <a name="configure-hybrid-azure-ad-join"></a>Настройка гибридного присоединения к Azure AD.

Чтобы настроить гибридное присоединение к Azure AD с помощью Azure AD Connect, вам потребуется следующее:

- учетные данные глобального администратора для клиента Azure AD;  
- учетные данные администратора предприятия для каждого леса;
- учетные данные администратора AD FS.

**Чтобы настроить гибридное присоединение к Azure AD с помощью Azure AD Connect, сделайте следующее.** :

1. Запустите Azure AD Connect и выберите **Настройка**.

   ![Экран приветствия](./media/hybrid-azuread-join-federated-domains/11.png)

1. На странице **Дополнительные задачи** выберите **Настройка параметров устройств**, а затем **Далее**.

   ![Дополнительные задачи](./media/hybrid-azuread-join-federated-domains/12.png)

1. На странице **обзора** нажмите кнопку **Далее**.

   ![Обзор](./media/hybrid-azuread-join-federated-domains/13.png)

1. На странице **Подключение к Azure AD** укажите учетные данные глобального администратора для клиента Azure AD и нажмите кнопку **Далее**.

   ![Подключение к Azure AD](./media/hybrid-azuread-join-federated-domains/14.png)

1. На странице **Параметры устройств** выберите **Настроить гибридное присоединение к Azure AD** и выберите **Далее**.

   ![Параметры устройств](./media/hybrid-azuread-join-federated-domains/15.png)

1. На странице **SCP** выполните приведенные ниже действия, а затем выберите **Далее**:

   ![SCP](./media/hybrid-azuread-join-federated-domains/16.png)

   1. Выберите лес.
   1. Выберите службу аутентификации. Необходимо выбрать сервер **AD FS**, если только ваша организация не использует исключительно клиенты Windows 10 либо простой единый вход и вы не настроили синхронизацию с компьютером или устройством.
   1. Выберите **Добавить**, чтобы ввести учетные данные администратора предприятия.

1. На странице **Операционные системы устройств** выберите операционные системы, используемые устройствами в вашей среде Active Directory, и нажмите кнопку **Далее**.

   ![Операционная система устройства](./media/hybrid-azuread-join-federated-domains/17.png)

1. На странице **Настройка службы федерации** введите учетные данные администратора AD FS и нажмите кнопку **Далее**.

   ![Настройка службы федерации](./media/hybrid-azuread-join-federated-domains/18.png)

1. На странице **Готово к настройке** выберите **Настроить**.

   ![Готово к настройке](./media/hybrid-azuread-join-federated-domains/19.png)

1. На странице **Конфигурация завершена** выберите **Выйти**.

   ![Завершение настройки](./media/hybrid-azuread-join-federated-domains/20.png)

## <a name="enable-windows-downlevel-devices"></a>Включение устройств Windows нижнего уровня

Если некоторые из присоединенных к домену устройств являются устройствами Windows нижнего уровня, выполните указанные ниже действия:

- Настройка параметров локальной интрасети для регистрации устройств.
- Установка Microsoft Workplace Join для компьютеров Windows нижнего уровня.

> [!NOTE]
> Поддержка Windows 7 закончилась 14 января 2020 г. Дополнительные сведения см. в статье о [прекращении поддержки Windows 7](https://support.microsoft.com/en-us/help/4057281/windows-7-support-ended-on-january-14-2020).

### <a name="configure-the-local-intranet-settings-for-device-registration"></a>Настройка параметров локальной интрасети для регистрации устройств.

Чтобы успешно выполнить гибридное присоединение устройств Windows нижнего уровня к Azure AD и избежать запросов на предоставление сертификатов при аутентификации этих устройств в Azure AD, можно отправить на присоединенные к домену устройства политику для добавления следующих URL-адресов в зону локальной интрасети в Internet Explorer:

- `https://device.login.microsoftonline.com`
- служба токенов безопасности вашей организации (для федеративных доменов);
- `https://autologon.microsoftazuread-sso.com` (для простого единого входа).

Необходимо также включить параметр **Разрешить обновление строки состояния в сценарии** в локальной зоне интрасети пользователя.

### <a name="install-microsoft-workplace-join-for-windows-downlevel-computers"></a>Установка Microsoft Workplace Join для компьютеров Windows нижнего уровня.

Чтобы зарегистрировать устройства Windows нижнего уровня, необходимо установить в организации [Microsoft Workplace Join для компьютеров, на которых не используется Windows 10](https://www.microsoft.com/download/details.aspx?id=53554). Microsoft Workplace Join для компьютеров не на базе Windows 10 доступна в центре загрузки Майкрософт.

Развернуть пакет можно с помощью системы распространения программного обеспечения, например  [Microsoft Endpoint Configuration Manager](/configmgr/). Этот пакет поддерживает параметры стандартной автоматической установки с использованием параметра `quiet`. В текущей ветви System Center Configuration Manager доступны дополнительные преимущества предыдущих версий, такие как возможность отслеживать ход регистрации.

Установщик создает в системе запланированную задачу, которая выполняется в контексте пользователя. Задача запускается в момент входа пользователя в систему Windows. Эта задача автоматически присоединяет устройство к Azure AD, используя учетные данные пользователя, после аутентификации с помощью AD.

## <a name="verify-the-registration"></a>Проверка регистрации

Ниже приведены три способа определения и проверки состояния устройства.

### <a name="locally-on-the-device"></a>Локально на устройстве

1. Откройте Windows PowerShell.
2. Введите `dsregcmd /status`.
3. Убедитесь, что для **AzureAdJoined** и **DomainJoined** установлено значение **YES** (Да).
4. Вы можете использовать **DeviceId** и сравнить состояние службы с помощью портала Azure или PowerShell.

### <a name="using-the-azure-portal"></a>Использование портала Azure

1. Перейдите на страницу устройства по [прямой ссылке](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices).
2. См. сведения о том, как найти устройство, в статье [Управление удостоверениями устройств с помощью портала Azure](./device-management-azure-portal.md).
3. Если для столбца **Зарегистрировано** отображается состояние **Ожидается**, это значит, что гибридное присоединение к Azure AD не завершено. В федеративных средах это может произойти, только если не удалось выполнить регистрацию, а для AAD Connect настроена синхронизация устройств.
4. Если для столбца **Зарегистрировано** отображается **дата и время**, это значит, что гибридное присоединение к Azure AD завершено.

### <a name="using-powershell"></a>Использование PowerShell

Проверьте состояние регистрации устройства в клиенте Azure с помощью **[Get-MsolDevice](/powershell/module/msonline/get-msoldevice)** . Этот командлет применяется в [модуле PowerShell для Azure Active Directory](/powershell/azure/active-directory/install-msonlinev1).

Когда вы используете командлет **Get-MSolDevice**, чтобы получить сведения о службе:

- Должен существовать объект с **идентификатором устройства**, который совпадает с идентификатором в клиенте Windows.
- У параметра **DeviceTrustType** должно быть значение **Domain Joined**. Этот параметр является эквивалентным состоянию **Гибридные устройства, присоединенные к Azure AD** на странице **Устройства** на портале Azure AD.
- Для устройств, которые используются при условном доступе, у параметра **Enabled** должно быть значение **True**, а у **DeviceTrustLevel** — **Managed**.

1. Откройте Windows PowerShell от имени администратора.
2. Введите `Connect-MsolService`, чтобы подключится к своему клиенту Azure.

#### <a name="count-all-hybrid-azure-ad-joined-devices-excluding-pending-state"></a>Число всех устройств с гибридным присоединением к Azure AD, кроме тех, для кого отображается состояние **Ожидается**

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="count-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Число всех устройств с гибридным присоединением к Azure AD с отображаемым состоянием **Ожидается**

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="list-all-hybrid-azure-ad-joined-devices"></a>Перечисление всех устройств с гибридным присоединением к Azure AD

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Перечисление всех гибридных устройств с гибридным присоединением к Azure AD с отображаемым состоянием **Ожидается**

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-details-of-a-single-device"></a>Вывод сведений об одном устройстве.

1. Введите `get-msoldevice -deviceId <deviceId>` (это идентификатор **DeviceId**, полученный локально на устройстве).
2. Убедитесь, что параметр **Включено** имеет значение **True**.

## <a name="troubleshoot-your-implementation"></a>Устранение неполадок реализации

Если возникают проблемы с настройкой гибридного присоединения к Azure AD для устройств Windows, присоединенных к домену, ознакомьтесь со следующими статьями:

- [Устранение неполадок с устройствами с помощью команды dsregcmd](./troubleshoot-device-dsregcmd.md)
- [Устранение неполадок на устройствах под управлением Windows 10 и Windows Server 2016 с гибридным присоединением к Azure Active Directory](troubleshoot-hybrid-join-windows-current.md)
- [Устранение неполадок на устройствах нижнего уровня с гибридным присоединением к Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [управлять удостоверениями устройств с помощью портала Azure](device-management-azure-portal.md).

<!--Image references-->
[1]: ./media/active-directory-conditional-access-automatic-device-registration-setup/12.png
