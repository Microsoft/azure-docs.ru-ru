---
title: Настройка устройств с гибридным присоединением к Azure Active Directory вручную | Документация Майкрософт
description: Узнайте, как вручную настроить устройства с гибридным присоединением к Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 05/14/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 28cc8a858d1779e17c893d64eda5f907bb4c808e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104577994"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-joined-devices-manually"></a>Руководство по Настройка устройств с гибридным присоединением к Azure Active Directory

Благодаря управлению устройствами в Azure Active Directory (Azure AD) пользователи получают доступ к ресурсам с устройств, которые соответствуют стандартам безопасности и нормативным требованиям. Дополнительные сведения см. в статье [Общие сведения об управлении устройствами в Azure Active Directory](overview.md).

> [!TIP]
> Если вы можете использовать Azure AD Connect, ознакомьтесь со связанными руководствами об [управляемых](hybrid-azuread-join-managed-domains.md) или [федеративных](hybrid-azuread-join-federated-domains.md) доменах. С помощью Azure AD Connect можно значительно упростить настройку гибридного присоединения к Azure AD.

Если вы хотите присоединить входящие в состав домена устройства к Azure AD в имеющейся локальной среде Active Directory, можно настроить гибридные устройства, присоединенные к Azure AD. В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка гибридного присоединения к Azure AD вручную;
> * настройка точки подключения службы;
> * настройка выдачи утверждений;
> * Включение устройств Windows нижнего уровня.
> * Проверка присоединенных устройств.
> * Устранение неполадок реализации

## <a name="prerequisites"></a>Предварительные требования

В данном руководстве предполагается, что вы ознакомлены со следующими темами:

* [Общие сведения об управлении устройствами в Azure Active Directory](./overview.md)
* [Планирование реализации гибридного присоединения к Azure Active Directory](hybrid-azuread-join-plan.md).
* [Управление гибридным присоединением устройств к Azure Active Directory](hybrid-azuread-join-control.md)

Прежде чем настраивать в организации гибридные устройства, присоединенные к Azure AD, убедитесь в следующем:

* вы работаете с актуальной версией Azure AD Connect;
* в Azure AD Connect синхронизированы объекты-компьютеры гибридных устройств, которые нужно присоединить к Azure AD. Если объекты-компьютеры принадлежат конкретным подразделениям, эти подразделения также нужно настроить для синхронизации в Azure AD Connect.

Azure AD Connect выполняет следующие функции:

* отслеживает связи между учетными записями на компьютерах в локальном экземпляре Active Directory и объектами устройств в Azure AD;
* поддерживает другие функции, связанные с устройствами, например Windows Hello для бизнеса.

Убедитесь, что следующие URL-адреса доступны с компьютеров в сети организации для регистрации компьютеров в Azure AD:

* `https://enterpriseregistration.windows.net`
* `https://login.microsoftonline.com`
* `https://device.login.microsoftonline.com`
* Служба токенов безопасности вашей организации (для федеративных доменов) должна быть включена в пользовательских параметрах локальной интрасети.

> [!WARNING]
> Если ваша организация использует прокси-серверы, которые перехватывают трафик SSL для таких сценариев, как защита от потери данных или ограничения арендатора Azure AD, убедитесь, что трафик к https://device.login.microsoftonline.com исключается из процесса приостановки и изучения TLS-трафика. Если адрес https://device.login.microsoftonline.com не будет исключен, это может вызвать ошибки при аутентификации с помощью сертификата клиента, что приведет к проблемам с регистрацией устройств и условным доступом на основе устройств.

Если ваша организация планирует использовать простой единый вход, следующий URL-адрес должен быть доступен с компьютеров в вашей организации. Его необходимо добавить в зону локальной интрасети пользователя.

* `https://autologon.microsoftazuread-sso.com`

Кроме того, в зоне интрасети пользователя должен быть включен следующий параметр: "Разрешить обновление строки состояния в сценарии".

Если ваша организация использует управляемую (нефедеративную) конфигурацию с локальной службой Active Directory и не использует службы федерации Active Directory (AD FS) для образования федерации с Azure AD, то соединение посредством гибридной среды Azure AD в Windows 10 зависит от синхронизации объектов-компьютеров в Active Directory с Azure AD. Убедитесь в том, что для всех подразделений с компьютерами-объектами, которые должны быть соединены посредством гибридной среды Azure AD, обеспечивается синхронизация в конфигурации синхронизации Azure AD Connect.

Если вы используете устройства Windows 10 под управлением версии 1703 или более ранней и при этом вашей организации требуется доступ к Интернету через исходящий прокси-сервер, необходимо реализовать автоматическое обнаружение веб-прокси (WPAD), чтобы компьютеры Windows 10 можно было зарегистрировать в Azure AD.

Начиная с Windows 10 версии 1803, если попытка гибридного присоединения к Azure AD для федеративного домена (такого как AD FS) завершается ошибкой, а в Azure AD Connect настроена синхронизация объекта-устройства или компьютера с Azure AD, устройство попытается выполнить это присоединение с помощью синхронизированного компьютера или устройства.

> [!NOTE]
> Чтобы синхронизировать регистрацию устройства, при настройке регистрации устройства не исключайте атрибуты устройств по умолчанию из конфигурации синхронизации Azure AD Connect. Дополнительные сведения об атрибутах устройств по умолчанию, синхронизированных с Azure AD, см. в разделе [Windows 10](https://docs.microsoft.com/azure/active-directory/hybrid/reference-connect-sync-attributes-synchronized#windows-10).

Проверить, имеет ли устройство доступ к указанным выше ресурсам Майкрософт под системной учетной записью, можно с помощью [скрипта для проверки подключения при регистрации устройств](/samples/azure-samples/testdeviceregconnectivity/testdeviceregconnectivity/).

## <a name="verify-configuration-steps"></a>Проверка шагов конфигурации

Гибридные устройства, присоединенные к Azure AD, можно настроить для различных типов платформ устройств Windows. В этом разделе описаны обязательные действия для всех типичных сценариев настройки.  

Следующая таблица содержит обзор действий, необходимых для вашего сценария.  

| Шаги | Текущие устройства Windows и синхронизация хэша паролей | Текущие устройства Windows и федерация | Устройства Windows нижнего уровня |
| :--- | :---: | :---: | :---: |
| Настройка точки подключения службы | ![Проверить][1] | ![Проверить][1] | ![Проверить][1] |
| настройка выдачи утверждений; |     | ![Проверить][1] | ![Проверить][1] |
| Включение поддержки устройств с ОС, отличными от Windows 10 |       |        | ![Проверить][1] |
| Проверка присоединенных устройств. | ![Проверить][1] | ![Проверить][1] | ![Проверить][1] |

## <a name="configure-a-service-connection-point"></a>настройка точки подключения службы;

Объект точки подключения службы (SCP) используется устройствами во время регистрации для получения сведений о клиенте Azure AD. В локальном экземпляре Active Directory должен существовать объект SCP для гибридных устройств, присоединенных к Azure AD, определенный в разделе контекста именования конфигурации в лесу компьютеров. В каждом лесу существует только один контекст именования конфигурации. В конфигурации Active Directory с несколькими лесами точка подключения службы должна существовать во всех лесах с присоединенными к домену компьютерами.

Для получения контекста именования конфигурации для леса можно использовать командлет [**Get-ADRootDSE**](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee617246(v=technet.10)).  

Для леса, в котором домен Active Directory имеет имя *example.com*, контекст именования конфигурации будет таким:

`CN=Configuration,DC=fabrikam,DC=com`

В лесу объект SCP для автоматической регистрации присоединенных к домену устройств имеет следующее расположение:  

`CN=62a0ff2e-97b9-4513-943f-0d221bd30080,CN=Device Registration Configuration,CN=Services,[Your Configuration Naming Context]`

В зависимости от того, как выполнялось развертывание Azure AD Connect, объект SCP уже может быть настроен.
Чтобы проверить существование объекта и получить значения для обнаружения, используйте следующий сценарий Windows PowerShell:

   ```PowerShell
   $scp = New-Object System.DirectoryServices.DirectoryEntry;

   $scp.Path = "LDAP://CN=62a0ff2e-97b9-4513-943f-0d221bd30080,CN=Device Registration Configuration,CN=Services,CN=Configuration,DC=fabrikam,DC=com";

   $scp.Keywords;
   ```

В выходных данных **$scp.Keywords** указаны сведения о клиенте Azure AD. Ниже приведен пример:

   ```
   azureADName:microsoft.com
   azureADId:72f988bf-86f1-41af-91ab-2d7cd011db47
   ```

Если точка подключения службы не существует, создайте ее, выполнив командлет `Initialize-ADSyncDomainJoinedComputerSync` на сервере Azure AD Connect. Для выполнения этого командлета необходимы учетные данные администратора организации.  

Этот командлет:

* Создает точку подключения службы в лесу Active Directory, к которому подключена служба Azure AD Connect.
* Требует указать обязательный параметр `AdConnectorAccount`, который обозначает учетную запись, настроенную в Azure AD Connect в качестве учетной записи соединителя Active Directory.


Следующий скрипт демонстрирует использование этого командлета. В этом скрипте есть параметр `$aadAdminCred = Get-Credential`, который требует ввести имя пользователя. Имя пользователя следует вводить в формате имени участника-пользователя (`user@example.com`).

   ```PowerShell
   Import-Module -Name "C:\Program Files\Microsoft Azure Active Directory Connect\AdPrep\AdSyncPrep.psm1";

   $aadAdminCred = Get-Credential;

   Initialize-ADSyncDomainJoinedComputerSync –AdConnectorAccount [connector account name] -AzureADCredentials $aadAdminCred;
   ```

Командлет `Initialize-ADSyncDomainJoinedComputerSync`:

* Использует модуль PowerShell для Active Directory и средства доменных служб Azure Active Directory (Azure AD DS). Эти средства зависят от веб-служб Active Directory, работающих на контроллере домена. Поддержку веб-служб Active Directory выполняют контроллеры домена под управлением Windows Server 2008 R2 и более поздних версий.
* Он поддерживается только модулем MSOnline PowerShell версии 1.1.166.0. Чтобы скачать этот модуль, используйте эту [ссылку](https://www.powershellgallery.com/packages/MSOnline/1.1.166.0).
* Если не установить средства AD DS, выполнение командлета `Initialize-ADSyncDomainJoinedComputerSync` завершится ошибкой. Средства AD DS можно установить с помощью диспетчера сервера в разделе **Компоненты** > **Средства удаленного администрирования сервера** > **Средства администрирования ролей**.

Если вы используете контроллер домена под управлением Windows Server 2008 или более ранних версий, для создания точки подключения службы используйте приведенный ниже сценарий. В конфигурации с несколькими лесами следует использовать следующий сценарий, чтобы создать точку подключения службы в каждом лесу, где существуют компьютеры.

   ```PowerShell
   $verifiedDomain = "contoso.com" # Replace this with any of your verified domain names in Azure AD
   $tenantID = "72f988bf-86f1-41af-91ab-2d7cd011db47" # Replace this with you tenant ID
   $configNC = "CN=Configuration,DC=corp,DC=contoso,DC=com" # Replace this with your Active Directory configuration naming context

   $de = New-Object System.DirectoryServices.DirectoryEntry
   $de.Path = "LDAP://CN=Services," + $configNC
   $deDRC = $de.Children.Add("CN=Device Registration Configuration", "container")
   $deDRC.CommitChanges()

   $deSCP = $deDRC.Children.Add("CN=62a0ff2e-97b9-4513-943f-0d221bd30080", "serviceConnectionPoint")
   $deSCP.Properties["keywords"].Add("azureADName:" + $verifiedDomain)
   $deSCP.Properties["keywords"].Add("azureADId:" + $tenantID)

   $deSCP.CommitChanges()
   ```

В приведенном выше сценарии `$verifiedDomain = "contoso.com"` является заполнителем, который необходимо заменить одним из проверенных доменных имен в Azure AD. Необходимо будет стать владельцем этого домена, прежде чем использовать его.

Дополнительные сведения о проверенных доменных именах см. в статье [Краткое руководство. Добавление личного домена в Azure Active Directory](../fundamentals/add-custom-domain.md).

Чтобы получить список проверенных доменов компании, можно использовать командлет [Get-AzureADDomain](/powershell/module/Azuread/Get-AzureADDomain).

![Список доменов компании](./media/hybrid-azuread-join-manual/01.png)

## <a name="set-up-issuance-of-claims"></a>настройка выдачи утверждений;

В федеративной конфигурации Azure AD устройства проходят проверку подлинности в AD FS или с помощью службы федерации на локальном сервере партнера корпорации Майкрософт. При проверке подлинности устройства получают маркер доступа для регистрации в службе регистрации устройств Azure Active Directory (Azure DRS).

Текущие устройства Windows с помощью встроенной проверки подлинности Windows проходят проверку подлинности в активной конечной точке WS-Trust (версии 1.3 или 2005), размещенной в локальной службе федерации.

При использовании AD FS нужно включить следующие конечные точки WS-Trust.
- `/adfs/services/trust/2005/windowstransport`
- `/adfs/services/trust/13/windowstransport`
- `/adfs/services/trust/2005/usernamemixed`
- `/adfs/services/trust/13/usernamemixed`
- `/adfs/services/trust/2005/certificatemixed`
- `/adfs/services/trust/13/certificatemixed`

> [!WARNING]
> Также нужно включить **adfs/services/trust/2005/windowstransport** и **adfs/services/trust/13/windowstransport**, но только в качестве конечных точек с подключением к интрасети. Их НЕЛЬЗЯ предоставлять как конечные точки с подключением к экстрасети через прокси-сервер веб-приложения. Дополнительные сведения см. в статье об [отключении конечных точек WS-Trust в Windows на прокси-сервере](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs#disable-ws-trust-windows-endpoints-on-the-proxy-ie-from-extranet). В разделе **Служба** > **Конечные точки** вы можете увидеть, какие конечные точки активированы в консоли управления AD FS.

> [!NOTE]
>Если вы не используете AD FS в качестве локальной службы федерации, обратитесь к своему поставщику за инструкциями и убедитесь, что система поддерживает конечные точки WS-Trust 1.3 или 2005 и что эти конечные точки публикуются с использованием файла обмена метаданными (MEX-файла).

Чтобы регистрация устройства прошла успешно, в полученном с помощью DRS Azure маркере должны существовать следующие утверждения. Azure DRS создаст в Azure AD объект устройства, который содержит часть этой информации. Затем Azure AD Connect использует эти сведения для привязки созданного объекта устройства к учетной записи локального компьютера.

* `http://schemas.microsoft.com/ws/2012/01/accounttype`
* `http://schemas.microsoft.com/identity/claims/onpremobjectguid`
* `http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid`

Если у вас есть несколько подтвержденных доменных имен, для компьютеров нужно предоставить следующее утверждение:

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid`

Если вы уже выдаете утверждение ImmutableID (например, используя `mS-DS-ConsistencyGuid` или другой атрибут в качестве исходного значения для ImmutableID), предоставьте одно соответствующее утверждение для компьютеров.

* `http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`

В следующих разделах вы найдете следующую информацию:

* какие значения должны входить в каждое утверждение;
* как выглядит определение в AD FS.

Определение помогает проверить, существуют ли нужные значения, или их необходимо создать.

> [!NOTE]
> Если вы не используете AD FS в качестве локального сервера федерации, следуйте инструкциям поставщика, чтобы создать соответствующую конфигурацию для выдачи этих утверждений.

### <a name="issue-account-type-claim"></a>Выдача утверждений для типа учетной записи

Утверждение `http://schemas.microsoft.com/ws/2012/01/accounttype` должно содержать значение **DJ**, которое идентифицирует устройство как присоединенный к домену компьютер. В AD FS вы можете добавить правила преобразования выдачи, например такие:

   ```
   @RuleName = "Issue account type for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "DJ"
   );
   ```

### <a name="issue-objectguid-of-the-computer-account-on-premises"></a>Выдача идентификатора objectGUID для локальной учетной записи компьютера

Утверждение `http://schemas.microsoft.com/identity/claims/onpremobjectguid` должно содержать значение **objectGUID** для учетной записи локального компьютера. В AD FS вы можете добавить правила преобразования выдачи, например такие:

   ```
   @RuleName = "Issue object GUID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$", 
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/identity/claims/onpremobjectguid"),
      query = ";objectguid;{0}",
      param = c2.Value
   );
   ```

### <a name="issue-objectsid-of-the-computer-account-on-premises"></a>Выдача идентификатора objectSID для локальной учетной записи компьютера

Утверждение `http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid` должно содержать значение **objectSid** для учетной записи локального компьютера. В AD FS вы можете добавить правила преобразования выдачи, например такие:

   ```
   @RuleName = "Issue objectSID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(claim = c2);
   ```

### <a name="issue-issuerid-for-the-computer-when-multiple-verified-domain-names-are-in-azure-ad"></a>Выдача issuerID для компьютера при наличии нескольких проверенных доменных имен в Azure AD

Утверждение `http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid` должно содержать универсальный идентификатор ресурса (URI) для любого проверенного доменного имени, которое подключается с использованием локальной службы федерации (AD FS или стороннего поставщика), выдающей маркер. В AD FS вы можете добавить правила преобразования выдачи, похожие на представленные ниже, (именно в таком порядке) после указанных выше правил. Обратите внимание, что для явной выдачи правил пользователям необходимо одно правило. В следующих правилах добавляется первое правило, определяющее аутентификацию пользователя, а не компьютера.

   ```
   @RuleName = "Issue account type with the value User when its not a computer"
   NOT EXISTS(
   [
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "DJ"
   ]
   )
   => add(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "User"
   );

   @RuleName = "Capture UPN when AccountType is User and issue the IssuerID"
   c1:[
      Type == "http://schemas.xmlsoap.org/claims/UPN"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "User"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = regexreplace(
      c1.Value,
      ".+@(?<domain>.+)",
      "http://${domain}/adfs/services/trust/"
      )
   );

   @RuleName = "Issue issuerID for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = "http://<verified-domain-name>/adfs/services/trust/"
   );
   ```

В приведенном выше утверждении `<verified-domain-name>` является заполнителем, который необходимо заменить одним из проверенных доменных имен в Azure AD. Например, воспользуйтесь `Value = "http://contoso.com/adfs/services/trust/"`.

Дополнительные сведения о проверенных доменных именах см. в статье [Краткое руководство. Добавление личного домена в Azure Active Directory](../fundamentals/add-custom-domain.md).  

Чтобы получить список проверенных доменов компании, можно использовать командлет [Get-MsolDomain](/powershell/module/msonline/get-msoldomain).

![Список доменов компании](./media/hybrid-azuread-join-manual/01.png)

### <a name="issue-immutableid-for-the-computer-when-one-for-users-exists-for-example-using-ms-ds-consistencyguid-as-the-source-for-immutableid"></a>Выдача утверждения ImmutableID для компьютера, если оно существует (например, используя mS-DS-ConsistencyGuid в качестве источника для ImmutableID)

Утверждение `http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID` должно содержать допустимое значение для компьютеров. В AD FS можно создать такие правила преобразования выдачи:

   ```
   @RuleName = "Issue ImmutableID for computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"),
      query = ";objectguid;{0}",
      param = c2.Value
   );
   ```

### <a name="helper-script-to-create-the-ad-fs-issuance-transform-rules"></a>Вспомогательный скрипт для создания правила преобразования выдачи AD FS

Следующий скрипт поможет вам создавать описанные ранее правила преобразования выдачи.

   ```
   $multipleVerifiedDomainNames = $false
   $immutableIDAlreadyIssuedforUsers = $false
   $oneOfVerifiedDomainNames = 'example.com'   # Replace example.com with one of your verified domains

   $rule1 = '@RuleName = "Issue account type for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "DJ"
   );'

   $rule2 = '@RuleName = "Issue object GUID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/identity/claims/onpremobjectguid"),
      query = ";objectguid;{0}",
      param = c2.Value
   );'

   $rule3 = '@RuleName = "Issue objectSID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(claim = c2);'

   $rule4 = ''
   if ($multipleVerifiedDomainNames -eq $true) {
   $rule4 = '@RuleName = "Issue account type with the value User when it is not a computer"
   NOT EXISTS(
   [
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "DJ"
   ]
   )
   => add(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "User"
   );

   @RuleName = "Capture UPN when AccountType is User and issue the IssuerID"
   c1:[
      Type == "http://schemas.xmlsoap.org/claims/UPN"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "User"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = regexreplace(
      c1.Value,
      ".+@(?<domain>.+)",
      "http://${domain}/adfs/services/trust/"
      )
   );

   @RuleName = "Issue issuerID for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = "http://' + $oneOfVerifiedDomainNames + '/adfs/services/trust/"
   );'
   }

   $rule5 = ''
   if ($immutableIDAlreadyIssuedforUsers -eq $true) {
   $rule5 = '@RuleName = "Issue ImmutableID for computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"),
      query = ";objectguid;{0}",
      param = c2.Value
   );'
   }

   $existingRules = (Get-ADFSRelyingPartyTrust -Identifier urn:federation:MicrosoftOnline).IssuanceTransformRules 

   $updatedRules = $existingRules + $rule1 + $rule2 + $rule3 + $rule4 + $rule5

   $crSet = New-ADFSClaimRuleSet -ClaimRule $updatedRules

   Set-AdfsRelyingPartyTrust -TargetIdentifier urn:federation:MicrosoftOnline -IssuanceTransformRules $crSet.ClaimRulesString
   ```

#### <a name="remarks"></a>Remarks

* Этот скрипт добавляет правила к существующим правилам. Не выполняйте этот скрипт дважды, иначе набор правил будет также добавлен дважды. Перед повторным запуском скрипта убедитесь, что для этих утверждений не существует установленных правил (при соответствующих условиях).
* Если у вас есть несколько проверенных доменных имен (это можно проверить на портале Azure AD или с помощью командлета **Get-MsolDomains**), в скрипте для переменной **$multipleVerifiedDomainNames** задайте значение **$true**. Не забудьте также удалить все существующие утверждения **issuerid**, которые могли создаваться в Azure AD Connect или другими средствами. Вот пример для этого правила:

   ```
   c:[Type == "http://schemas.xmlsoap.org/claims/UPN"]
   => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)",  "http://${domain}/adfs/services/trust/")); 
   ```

Если утверждение **ImmutableID** для учетных записей пользователей уже выдано, задайте для **$immutableIDAlreadyIssuedforUsers** в сценарии значение **$true**.

## <a name="enable-windows-down-level-devices"></a>Включение устройств Windows нижнего уровня.

Если некоторые из присоединенных к домену устройств являются устройствами Windows нижнего уровня, выполните указанные ниже действия.

* Настройте в Azure AD политику, разрешающую пользователям регистрировать устройства.
* Настройте локальную службу федерации, чтобы она выдавала утверждения встроенной проверки подлинности Windows (IWA), необходимые для регистрации устройств.
* В локальные зоны интрасети добавьте конечную точку проверки подлинности устройств в Azure AD, чтобы избежать запросов сертификатов при проверке подлинности устройств.
* Управляйте устройствами Windows нижнего уровня.

### <a name="set-a-policy-in-azure-ad-to-enable-users-to-register-devices"></a>Настройте политику регистрации устройств в Azure AD.

Чтобы поддерживать регистрацию устройств Windows нижнего уровня, необходимо установить параметр, разрешающий пользователям регистрировать устройства в Azure AD. Чтобы найти этот параметр, на портале Azure выберите **Azure Active Directory** > **Пользователи и группы** > **Параметры устройства**.

Для следующей политики нужно установить значение **Все**: **Пользователи могут регистрировать устройства в Azure AD**.

![Кнопка "Все", которая позволяет пользователям регистрировать устройства](./media/hybrid-azuread-join-manual/23.png)

### <a name="configure-the-on-premises-federation-service"></a>Настройка локальной службы федерации

Локальная служба федерации должна поддерживать выдачу утверждений **authenticationmehod** и **wiaormultiauthn** при получении запросов на аутентификацию к проверяющей стороне Azure AD, содержащих параметр resource_params со следующим закодированным значением:

   ```
   eyJQcm9wZXJ0aWVzIjpbeyJLZXkiOiJhY3IiLCJWYWx1ZSI6IndpYW9ybXVsdGlhdXRobiJ9XX0

   which decoded is {"Properties":[{"Key":"acr","Value":"wiaormultiauthn"}]}
   ```

При поступлении такого запроса локальная служба федерации должна выполнить проверку подлинности пользователя, используя встроенную проверку подлинности Windows. При успешной проверке подлинности служба федерации должна выдать следующие два утверждения:

   `http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/windows` `http://schemas.microsoft.com/claims/wiaormultiauthn`

В AD FS следует добавить правило преобразования выдачи, пропускающее метод проверки подлинности. Чтобы добавить это правило, сделайте следующее:

1. В консоли управления служб федерации Active Directory выберите **AD FS** > **Отношения доверия** > **Отношения доверия проверяющей стороны**.
1. Щелкните правой кнопкой мыши объект отношений доверия с проверяющей стороной платформы удостоверений Microsoft Office 365 и выберите **Edit Claim Rules** (Изменить правила для утверждений).
1. На вкладке **Правила преобразования выдачи** выберите **Добавить правило**.
1. Из списка шаблонов **Claim rule** (Правило для утверждений) выберите **Отправка утверждений с помощью настраиваемого правила**.
1. Выберите **Далее**.
1. В поле **Claim rule name** (Имя правила утверждения) введите **Auth Method Claim Rule** (Правило для утверждений метода проверки подлинности).
1. В поле **Claim rule** (Правило для утверждения) введите такое правило:

   `c:[Type == "http://schemas.microsoft.com/claims/authnmethodsreferences"] => issue(claim = c);`

1. На сервере федерации введите следующую команду PowerShell. Замените **\<RPObjectName\>** именем объекта проверяющей стороны для объекта отношений доверия с проверяющей стороной Azure AD. Как правило, этот объект называется **платформой удостоверений Microsoft Office 365**.

   `Set-AdfsRelyingPartyTrust -TargetName <RPObjectName> -AllowedAuthenticationClassReferences wiaormultiauthn`

### <a name="add-the-azure-ad-device-authentication-endpoint-to-the-local-intranet-zones"></a>Добавление конечной точки для аутентификации устройств Azure AD в локальные зоны интрасети

Чтобы избежать запросов сертификатов при проверке подлинности в Azure AD пользователей с зарегистрированных устройств, можно отправить политику на присоединенные к домену устройства, добавив следующий URL-адрес в зону локальной интрасети в Internet Explorer:

`https://device.login.microsoftonline.com`

### <a name="control-windows-down-level-devices"></a>Управление устройствами Windows нижнего уровня

Чтобы регистрировать устройства Windows нижнего уровня, необходимо скачать из Центра загрузки Майкрософт и установить пакет установщика Windows (MSI-файл). Дополнительные сведения см. в разделе об [управлении устройствами Windows нижнего уровня](hybrid-azuread-join-control.md#controlled-validation-of-hybrid-azure-ad-join-on-windows-down-level-devices).

## <a name="verify-joined-devices"></a>Проверка присоединенных устройств.

Ниже приведены три способа определения и проверки состояния устройства.

### <a name="locally-on-the-device"></a>Локально на устройстве

1. Откройте Windows PowerShell.
2. Введите `dsregcmd /status`.
3. Убедитесь, что для **AzureAdJoined** и **DomainJoined** установлено значение **YES** (Да).
4. Вы можете использовать **DeviceId** и сравнить состояние службы с помощью портала Azure или PowerShell.

### <a name="using-the-azure-portal"></a>Использование портала Azure

1. Перейдите на страницу устройства по [прямой ссылке](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices).
2. См. сведения о том, как найти устройство, в статье [Управление удостоверениями устройств с помощью портала Azure](./device-management-azure-portal.md#manage-devices).
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

Если возникают проблемы с настройкой гибридного присоединения к Azure AD для устройств Windows, присоединенных к домену, ознакомьтесь со следующими статьями:

- [Устранение неполадок с устройствами с помощью команды dsregcmd](./troubleshoot-device-dsregcmd.md)
- [Устранение неполадок на устройствах с гибридным присоединением к Azure Active Directory](troubleshoot-hybrid-join-windows-current.md)
- [Устранение неполадок на устройствах нижнего уровня с гибридным присоединением к Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Общие сведения об управлении устройствами в Azure Active Directory](overview.md)

<!--Image references-->
[1]: ./media/hybrid-azuread-join-manual/12.png
