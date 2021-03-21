---
title: 'Azure AD Connect выполняет следующие функции: Настройка разрешений учетной записи соединителя AD DS | Документация Майкрософт'
description: В этом документе описаны способы настройки учетной записи соединителя AD DS с помощью нового модуля ADSyncConfig PowerShell
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/18/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 62bfc528886767bc09159ca2a2696c8c9264b307
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96349945"
---
# <a name="azure-ad-connectconfigure-ad-ds-connector-account-permissions"></a>Azure AD Connect выполняет следующие функции: Настройка разрешений учетной записи соединителя AD DS 

Модуль PowerShell [ADSyncConfig.psm1](reference-connect-adsyncconfig.md) был представлен в сборке 1.1.880.0 (выпущенной в августе 2018 года). Эта сборка включает в себя коллекцию командлетов, которые помогут вам настроить правильные разрешения Active Directory для развертывания соединителя Azure AD. 

## <a name="overview"></a>Обзор 
Чтобы настроить разрешения Active Directory для учетной записи соединителя AD DS, можно использовать следующие командлеты PowerShell. Их можно использовать для каждой функции, какую вы включаете в Azure AD Connect. Чтобы избежать проблем, следует заранее подготовлять разрешения Active Directory каждый раз, когда вы хотите установить Azure AD Connect, используя личный домен для подключения к вашему лесу. Модуль ADSyncConfig также можно использовать для настройки разрешений после развертывания Azure AD Connect.

![Обзор учетной записи AD DS](media/how-to-connect-configure-ad-ds-connector-account/configure1.png)

Для экспресс установки Azure AD Connect в Active Directory автоматически создается учетная запись (с именем MSOL_nnnnnnnnnn) с необходимыми разрешениями, поэтому необходимости использовать модуль ADSyncConfig в этом случае нет. Кроме тех случаев, когда вы заблокировали наследование разрешений в подразделениях или в определенных объектах Active Directory, которые вы хотите синхронизировать с Azure AD. 
 
### <a name="permissions-summary"></a>Сводка разрешений 
Следующая таблица предоставляет сводку разрешений, необходимых для объектов AD: 

| Компонент | Разрешения |
| --- | --- |
| функция ms-DS-ConsistencyGuid |Разрешения на чтение и запись для атрибута ms-DS-ConsistencyGuid см. в статье [Azure AD Connect: принципы проектирования](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor). | 
| Синхронизация хэша паролей |<li>Репликация изменений каталога</li>  <li>Репликация всех изменений каталога |
| Гибридное развертывание Exchange |Разрешения на чтение и запись для атрибутов, описанных в статье [Гибридная обратная запись Exchange](reference-connect-sync-attributes-synchronized.md#exchange-hybrid-writeback) для пользователей, групп и контактов. |
| Общедоступная папка почты Exchange |Разрешения на чтение для атрибутов, описанных в статье [Службы синхронизации Azure AD Connect: атрибуты, синхронизируемые с Azure Active Directory](reference-connect-sync-attributes-synchronized.md#exchange-mail-public-folder), для общедоступных папок. | 
| Компонент обратной записи паролей |Разрешения на чтение и запись для атрибутов, описанных в статье [Приступая к работе с компонентами управления паролями](../authentication/tutorial-enable-sspr-writeback.md) для пользователей. |
| Обратная запись устройств |Разрешения на чтение и запись для объектов и контейнеров устройства, описанные в [обратной записи устройств](how-to-connect-device-writeback.md). |
| Обратная запись групп |Чтение, создание, обновление и удаление объектов групп для синхронизированных **групп Office 365**.|

## <a name="using-the-adsyncconfig-powershell-module"></a>Использование модуля ADSyncConfig PowerShell 
Модулю ADSyncConfig необходимы [средства удаленного администрирования сервера (RSAT) для AD DS](/windows-server/remote/remote-server-administration-tools), поскольку он зависит от модуля PowerShell для AD DS и средств. Чтобы установить средства удаленного администрирования сервера для AD DS, откройте окно Windows PowerShell в режиме "Запуск от имени администратора" и выполните: 

``` powershell
Install-WindowsFeature RSAT-AD-Tools 
```
![Configure](media/how-to-connect-configure-ad-ds-connector-account/configure2.png)

>[!NOTE]
>Вы также можете скопировать файл **C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\ADSyncConfig.psm1** в контроллер домена, в котором уже есть средства удаленного администрирования сервера для AD DS, и использовать модуль PowerShell оттуда.

Чтобы приступить к использованию ADSyncConfig, необходимо загрузить модуль в окно Windows PowerShell: 

``` powershell
Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1" 
```

Для проверки всех командлетов в этом модуле введите:  

``` powershell
Get-Command -Module AdSyncConfig  
```

![Проверить](media/how-to-connect-configure-ad-ds-connector-account/configure3.png)

Каждый командлет имеет те же параметры для ввода учетной записи соединителя AD DS и коммутатора AdminSDHolder. Чтобы указать учетную запись соединителя AD DS, можно предоставить имя учетной записи и домен или только различающееся имя (DN) учетной записи.

Например:

```powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountName <ADAccountName> -ADConnectorAccountDomain <ADDomainName>
```

Или;

```powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountDN <ADAccountDN>
```

Обязательно замените `<ADAccountName>`, `<ADDomainName>` и `<ADAccountDN>` соответствующими значениями для вашей среды.

Если вы не хотите изменить разрешения на контейнере AdminSDHolder, используйте параметр `-SkipAdminSdHolders`. 

По умолчанию все командлеты заданных разрешений будут пытаться установить разрешения AD DS в корне каждого домена в лесу. Это значит, что пользователю, выполняющему сеанс PowerShell, требуются права администратора домена для каждого домена в лесу.  Из-за этого требования рекомендуется использовать администратора предприятия из корня леса. Если в развертывании Azure AD Connect есть несколько соединителей AD DS, то потребуется выполнение одного и того же командлета в каждом лесу, имеющему соединитель AD DS. 

Можно также задать разрешения на определенном объекте подразделения или AD DS с помощью параметра `-ADobjectDN`, которому следует различающееся имя целевого объекта, где требуется задать разрешения. При использовании целевого объекта ADobjectDN, командлет установит права доступа только для этого объекта, а не для корневого домена или контейнера AdminSDHolder. Этот параметр может быть полезным, если у вас есть некоторые подразделения или объекты AD DS, в которых отключено наследование разрешений (см. в разделе "Размещение объектов AD DS с отключенным наследованием разрешений") 

Исключением к этим общим параметрам являются командлет `Set-ADSyncRestrictedPermissions`, который позволяет задать разрешения на самой учетной записи соединителя AD DS, и командлет `Set-ADSyncPasswordHashSyncPermissions`, поскольку разрешения, необходимые для синхронизации хэша паролей можно установить только в корневом домене, следовательно этот командлет не поддерживает параметр `-ObjectDN` или `-SkipAdminSdHolders`.

### <a name="determine-your-ad-ds-connector-account"></a>Определите свою учетную запись соединителя AD DS 
Если Azure AD Connect уже установлено и вы хотите проверить, какая учетная запись соединителя AD DS используется на данное время, можете выполнить командлет: 

``` powershell
Get-ADSyncADConnectorAccount 
```
### <a name="locate-ad-ds-objects-with-permission-inheritance-disabled"></a>Размещение объектов AD DS с отключенным наследованием разрешений 
Если вы хотите проверить наличие какого-либо объекта AD DS с отключенным наследованием разрешений, можете выполнить: 

``` powershell
Get-ADSyncObjectsWithInheritanceDisabled -SearchBase '<DistinguishedName>' 
```
По умолчанию этот командлет будет искать только подразделения с отключенным наследованием, но вы можете указать в параметре `-ObjectClass` другие классы объектов AD DS или использовать "*" для всех классов объектов следующим образом: 

``` powershell
Get-ADSyncObjectsWithInheritanceDisabled -SearchBase '<DistinguishedName>' -ObjectClass * 
```
 
### <a name="view-ad-ds-permissions-of-an-object"></a>Просмотр разрешений AD DS объекта 
Вы можете использовать указанный ниже командлет, чтобы просмотреть список разрешений, установленных в объекте Active Directory в настоящее время, предоставив его DistinguishedName: 

``` powershell
Show-ADSyncADObjectPermissions -ADobjectDN '<DistinguishedName>' 
```

## <a name="configure-ad-ds-connector-account-permissions"></a>Настройка разрешений учетной записи соединителя AD DS 
 
### <a name="configure-basic-read-only-permissions"></a>Настройка основных разрешения только для чтения 
Чтобы задать учетной записи соединителя AD DS основные разрешения только для чтения, не используя компонентов Azure AD Connect, выполните: 

``` powershell
Set-ADSyncBasicReadPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


или; 

``` powershell
Set-ADSyncBasicReadPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```


Этот командлет задаст следующие разрешения: 
 

|Тип |Имя |Доступ |Применяется к| 
|-----|-----|-----|-----|
|Allow |Учетная запись соединителя AD DS |Чтение всех свойств |Дочерние объекты устройств| 
|Allow |Учетная запись соединителя AD DS|Чтение всех свойств |Дочерние объекты InetOrgPerson| 
|Allow |Учетная запись соединителя AD DS |Чтение всех свойств |Дочерние объекты компьютера| 
|Allow |Учетная запись соединителя AD DS |Чтение всех свойств |Дочерние объекты foreignSecurityPrincipal| 
|Allow |Учетная запись соединителя AD DS |Чтение всех свойств |Дочерние объекты группы| 
|Allow |Учетная запись соединителя AD DS |Чтение всех свойств |Дочерние объекты пользователя| 
|Allow |Учетная запись соединителя AD DS |Чтение всех свойств |Дочерние объекты контакта| 

 
### <a name="configure-ms-ds-consistency-guid-permissions"></a>Настройка разрешений MS-DS-Consistency-Guid 
Чтобы задать разрешения для учетной записи соединителя AD DS при использовании атрибута ms-DS-parallelism-GUID в качестве привязки к источнику (также известного как "разрешить Azure управлять привязкой к источнику"), выполните: 

``` powershell
Set-ADSyncMsDsConsistencyGuidPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```

или; 

``` powershell
Set-ADSyncMsDsConsistencyGuidPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```

Этот командлет задаст следующие разрешения: 

|Тип |Имя |Доступ |Применяется к|
|-----|-----|-----|-----| 
|Allow|Учетная запись соединителя AD DS|Свойство чтения/записи|Дочерние объекты пользователя|

### <a name="permissions-for-password-hash-synchronization"></a>Разрешения для синхронизации хэшей пароля 
Чтобы задать разрешения для учетной записи соединителя AD DS при использовании синхронизации хэша паролей, запустите: 

``` powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [<CommonParameters>] 
```


или; 

``` powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountDN <String> [<CommonParameters>] 
```

Этот командлет задаст следующие разрешения: 

|Тип |Имя |Доступ |Применяется к|
|-----|-----|-----|-----| 
|Allow |Учетная запись соединителя AD DS |Репликация изменений каталога |Только в этом объекте (корневой домен)| 
|Allow |Учетная запись соединителя AD DS |Репликация всех изменений каталога |Только в этом объекте (корневой домен)| 
  
### <a name="permissions-for-password-writeback"></a>Разрешения для компонента обратной записи паролей 
Чтобы задать разрешения для учетной записи соединителя AD DS при использовании компонента обратной записи паролей, запустите: 

``` powershell
Set-ADSyncPasswordWritebackPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


или;

``` powershell
Set-ADSyncPasswordWritebackPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```
Этот командлет задаст следующие разрешения: 

|Тип |Имя |Доступ |Применяется к|
|-----|-----|-----|-----| 
|Allow |Учетная запись соединителя AD DS |Сброс пароля |Дочерние объекты пользователя| 
|Allow |Учетная запись соединителя AD DS |Запись свойства lockoutTime |Дочерние объекты пользователя| 
|Allow |Учетная запись соединителя AD DS |Запись свойства pwdLastSet |Дочерние объекты пользователя| 

### <a name="permissions-for-group-writeback"></a>Разрешения для обратной записи групп 
Чтобы задать разрешения для учетной записи соединителя AD DS при использовании записи групп, запустите: 

``` powershell
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```
или; 

``` powershell
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>]
```
 
Этот командлет задаст следующие разрешения: 

|Тип |Имя |Доступ |Применяется к|
|-----|-----|-----|-----| 
|Allow |Учетная запись соединителя AD DS |Универсальное чтение/запись |Все атрибуты группы типов объектов и дочерних объектов| 
|Allow |Учетная запись соединителя AD DS |Создать/удалить дочерний объект |Все атрибуты группы типов объектов и дочерних объектов| 
|Allow |Учетная запись соединителя AD DS |Удалить объекты дерева|Все атрибуты группы типов объектов и дочерних объектов|

### <a name="permissions-for-exchange-hybrid-deployment"></a>Разрешения для гибридного развертывания Exchange 
Чтобы задать разрешения для учетной записи соединителя AD DS при использовании гибридного развертывания Exchange, запустите: 

``` powershell
Set-ADSyncExchangeHybridPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


или; 

``` powershell
Set-ADSyncExchangeHybridPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```

Этот командлет задаст следующие разрешения:  
 

|Тип |Имя |Доступ |Применяется к|
|-----|-----|-----|-----| 
|Allow |Учетная запись соединителя AD DS |Чтение/запись всех свойств |Дочерние объекты пользователя| 
|Allow |Учетная запись соединителя AD DS |Чтение/запись всех свойств |Дочерние объекты InetOrgPerson| 
|Allow |Учетная запись соединителя AD DS |Чтение/запись всех свойств |Дочерние объекты группы| 
|Allow |Учетная запись соединителя AD DS |Чтение/запись всех свойств |Дочерние объекты контакта| 

### <a name="permissions-for-exchange-mail-public-folders-preview"></a>Разрешения для общедоступных папок почты Exchange (предварительная версия) 
Чтобы задать разрешения для учетной записи соединителя AD DS при использовании общедоступных папок почты Exchange, запустите: 

``` powershell
Set-ADSyncExchangeMailPublicFolderPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


или; 

``` powershell
Set-ADSyncExchangeMailPublicFolderPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```
Этот командлет задаст следующие разрешения: 

|Тип |Имя |Доступ |Применяется к|
|-----|-----|-----|-----| 
|Allow |Учетная запись соединителя AD DS |Чтение всех свойств |Дочерние объекты PublicFolder| 

### <a name="restrict-permissions-on-the-ad-ds-connector-account"></a>Ограничение разрешений учетной записи соединителя AD DS 
Этот сценарий PowerShell будет сужать разрешения для учетной записи соединителя AD, если он предоставлен в качестве параметра. Сужение разрешения включает следующие шаги: 

- Отключение наследования для указанного объекта. 
- Удаление всех элементов управления доступом в конкретном объекте, за исключением элементов управления доступом, характерных для самого объекта, поскольку при работе с самим объектом нам нужно сохранить разрешения по умолчанию без изменений. 
 
  Параметр -ADConnectorAccountDN — это учетная запись AD, для которой нужно ограничить разрешения. Обычно это учетная запись домена с именем MSOL_nnnnnnnnnnnn, настроенная в соединителе AD DS (см. в разделе "Определите свою учетную запись соединителя AD DS"). В параметре учетных данных необходимо указать учетную запись администратора, которая имеет необходимые правами доступа для ограничения разрешений Active Directory в целевом объекте AD. Обычно это администратор предприятия или домена.  

``` powershell
Set-ADSyncRestrictedPermissions [-ADConnectorAccountDN] <String> [-Credential] <PSCredential> [-DisableCredentialValidation] [-WhatIf] [-Confirm] [<CommonParameters>] 
```
 
Пример: 

``` powershell
$credential = Get-Credential 
Set-ADSyncRestrictedPermissions -ADConnectorAccountDN'CN=ADConnectorAccount,CN=Users,DC=Contoso,DC=com' -Credential $credential  
```

Этот командлет задаст следующие разрешения: 

|Тип |Имя |Доступ |Применяется к|
|-----|-----|-----|-----| 
|Allow |SYSTEM |Полный доступ |этому объекту 
|Allow |Администраторы предприятия |Полный доступ |этому объекту 
|Allow |Администраторы домена |Полный доступ |этому объекту 
|Allow |Администраторы |Полный доступ |этому объекту 
|Allow |Контроллеры домена предприятия |Вывод списка содержимого |этому объекту 
|Allow |Контроллеры домена предприятия |Чтение всех свойств |этому объекту 
|Allow |Контроллеры домена предприятия |Разрешения на чтение |этому объекту 
|Allow |Прошедшие проверку пользователи |Вывод списка содержимого |этому объекту 
|Allow |Прошедшие проверку пользователи |Чтение всех свойств |этому объекту 
|Allow |Прошедшие проверку пользователи |Разрешения на чтение |этому объекту 

## <a name="next-steps"></a>Next Steps
- [Azure AD Connect: учетные записи и разрешения](reference-connect-accounts-permissions.md)
- [Приступая к работе с Azure AD Connect с использованием стандартных параметров](how-to-connect-install-express.md)
- [Выборочная установка Azure AD Connect](how-to-connect-install-custom.md)
- [Справочник по ADSyncConfig](reference-connect-adsyncconfig.md)