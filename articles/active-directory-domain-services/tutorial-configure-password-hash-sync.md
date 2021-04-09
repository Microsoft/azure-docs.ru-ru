---
title: Включение синхронизации хэша пароля для Доменных служб Azure AD | Документация Майкрософт
description: В этом руководстве описано, как включить синхронизацию хэша паролей с помощью Azure AD Connect с управляемым доменом Доменных служб Azure Active Directory.
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: tutorial
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: e83d8941d1be7fd36f53a881a21716252ad01954
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96618354"
---
# <a name="tutorial-enable-password-synchronization-in-azure-active-directory-domain-services-for-hybrid-environments"></a>Руководство по Включению синхронизации паролей в Доменных службах Azure Active Directory для гибридных сред

В гибридных средах клиента Azure Active Directory (Azure AD) можно настроить для синхронизации с локальной средой Доменных служб Active Directory (AD DS) с помощью Azure AD Connect. По умолчанию Azure AD Connect не синхронизирует устаревшие хэши паролей NT LAN Manager (NTLM) и Kerberos, необходимые для Доменных служб Azure Active Directory (Azure AD DS).

Чтобы использовать Azure AD DS с учетными записями, синхронизированными из локальной среды AD DS, необходимо настроить Azure AD Connect для синхронизации таких хэшей паролей, которые необходимы для аутентификации NTLM и Kerberos. После настройки Azure AD Connect событие создания локальной учетной записи или изменения пароля также синхронизирует устаревшие хэши паролей с Azure AD.

Если вы используете только облачные учетные записи без локальной среды AD DS или если вы используете *лес ресурсов*, вам не нужно выполнять эти действия. Для управляемых доменов, которые используют лес ресурсов, локальные хэши паролей не синхронизируются. При аутентификации локальных учетных записей используется доверие леса, связанное с вашими контроллерами домена AD DS.

Из этого руководства вы узнаете:

> [!div class="checklist"]
> * Зачем нужны устаревшие хэши паролей NTLM и Kerberos
> * Сведения о настройке синхронизации хэша устаревшего пароля для Azure AD Connectt

Если у вас еще нет подписки Azure, [создайте учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником требуются следующие ресурсы:

* Активная подписка Azure.
    * Если у вас еще нет подписки Azure, [создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Связанный с вашей подпиской клиент Azure Active Directory, синхронизированный с локальным каталогом с помощью Azure AD Connect.
    * Если потребуется, [создайте клиент Azure Active Directory][create-azure-ad-tenant] или [свяжите подписку Azure со своей учетной записью][associate-azure-ad-tenant].
    * Если потребуется, [включите Azure AD Connect для синхронизации хэша пароля][enable-azure-ad-connect].
* Управляемый домен доменных служб Azure Active Directory, включенный и настроенный в клиенте Azure AD.
    * При необходимости [создайте и настройте управляемый домен доменных служб Azure Active Directory][create-azure-ad-ds-instance].

## <a name="password-hash-synchronization-using-azure-ad-connect"></a>Синхронизация хэша пароля с помощью Azure AD Connect

Azure AD Connect используется для синхронизации объектов, например учетных записей пользователей и групп из локальной среды AD DS, в клиент Azure AD. В рамках этого процесса синхронизация хэшей паролей позволяет учетным записям использовать один и тот же пароль в локальной среде AD DS и Azure AD.

В целях аутентификации пользователей в управляемом домене, Azure AD DS нужны хэши паролей в формате, подходящем для аутентификации NTLM и Kerberos. Пока вы не включите Azure AD DS для клиента, Azure AD не будет хранить хэши паролей в формате, требуемом для аутентификации NTLM или Kerberos. Из соображений безопасности Azure AD никогда не хранит пароли в виде открытого текста. Поэтому Azure AD не может автоматически создать хэши паролей в формате NTLM или Kerberos по уже существующим учетным данным пользователей.

Azure AD Connect можно настроить для синхронизации необходимых хэшей паролей NTLM или Kerberos для Azure AD DS. Убедитесь, что вы выполнили шаги, чтобы [включить Azure AD Connect для синхронизации хэша пароля][enable-azure-ad-connect]. Если у вас уже есть экземпляр Azure AD Connect, [загрузите и обновите его до последней версии][azure-ad-connect-download], чтобы убедиться, что вы можете синхронизировать устаревшие хэши паролей для NTLM и Kerberos. В более ранних выпусках Azure AD Connect и в устаревшей версии средства DirSync эта возможность недоступна. Требуется Azure AD Connect *1.1.614.0* или более поздней версии.

> [!IMPORTANT]
> Azure AD Connect следует устанавливать и настраивать только для синхронизации с локальными средами AD DS. Установка Azure AD Connect в управляемом домене Azure AD DS для синхронизации объектов обратно в Azure AD не поддерживается.

## <a name="enable-synchronization-of-password-hashes"></a>Включение синхронизации хэшей паролей

После установки и настройки Azure AD Connect для синхронизации с Azure AD настройте синхронизацию устаревшего хэша пароля для NTLM и Kerberos. Скрипт PowerShell используется для настройки обязательных параметров, а затем для запуска полной синхронизации пароля с Azure AD. После завершения процесса синхронизации хэша паролей Azure AD Connect, пользователи могут входить в приложения с помощью Azure AD DS, использующий устаревшие хэши паролей NTLM или Kerberos.

1. На компьютере с установленным Azure AD Connect в меню "Пуск" откройте **Azure AD Connect > Служба синхронизации**.
1. Щелкните вкладку **Соединители**. Здесь перечислены сведения о соединении, используемые для установления синхронизации между локальной средой AD DS и Azure AD.

    **Тип** указывает *Windows Azure Active Directory (Microsoft)* для соединителя Azure AD, или *Доменные службы Active Directory* для локального соединителя AD DS. Запишите имена соединителей, чтобы использовать их в скрипте PowerShell на следующем шаге.

    ![Перечисление имен соединителей в Диспетчере службы синхронизации](media/tutorial-configure-password-hash-sync/service-sync-manager.png)

    На снимке экрана в этом примере используются следующие соединители:

    * соединитель Azure AD называется *contoso.onmicrosoft.com - AAD*;
    * локальный соединитель AD DS называется *onprem.contoso.com*;

1. Скопируйте и вставьте следующий скрипт PowerShell на компьютер с установленным Azure AD Connect. Скрипт запускает полную синхронизацию паролей, включающую устаревшие хэши паролей. Обновите переменные `$azureadConnector` и `$adConnector`, указав имена соединителей из предыдущего шага.

    Запустите этот скрипт в каждом лесу AD, чтобы синхронизировать локальную учетную запись NTLM и хэши пароля Kerberos с Azure AD.

    ```powershell
    # Define the Azure AD Connect connector names and import the required PowerShell module
    $azureadConnector = "<CASE SENSITIVE AZURE AD CONNECTOR NAME>"
    $adConnector = "<CASE SENSITIVE AD DS CONNECTOR NAME>"
    
    Import-Module "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync\ADSync.psd1"
    Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1"

    # Create a new ForceFullPasswordSync configuration parameter object then
    # update the existing connector with this new configuration
    $c = Get-ADSyncConnector -Name $adConnector
    $p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null
    $p.Value = 1
    $c.GlobalParameters.Remove($p.Name)
    $c.GlobalParameters.Add($p)
    $c = Add-ADSyncConnector -Connector $c

    # Disable and re-enable Azure AD Connect to force a full password synchronization
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $false
    Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $true
    ```

    В зависимости от размера каталога с точки зрения количества учетных записей и групп, синхронизация устаревших хэшей паролей с Azure AD может занять некоторое время. После синхронизации с Azure AD пароли синхронизируются с управляемым доменом.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали следующее:

> [!div class="checklist"]
> * Зачем нужны устаревшие хэши паролей NTLM и Kerberos
> * Сведения о настройке синхронизации хэша устаревшего пароля для Azure AD Connectt

> [!div class="nextstepaction"]
> [Сведения о том, как работает синхронизация в управляемом домене доменных служб Azure AD](synchronization.md)

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[enable-azure-ad-connect]: ../active-directory/hybrid/how-to-connect-install-express.md

<!-- EXTERNAL LINKS -->
[azure-ad-connect-download]: https://www.microsoft.com/download/details.aspx?id=47594
