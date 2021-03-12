---
title: Установите агенты работоспособности Connect в Azure Active Directory
description: В этой статье Azure AD Connect Health описывается установка агента для службы федерации Active Directory (AD FS) (AD FS) и для синхронизации.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 1cc8ae90-607d-4925-9c30-6770a4bd1b4e
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/20/2020
ms.topic: how-to
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a3aff7b99dad910a9691eef2004df856ca883789
ms.sourcegitcommit: 94c3c1be6bc17403adbb2bab6bbaf4a717a66009
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103224390"
---
# <a name="azure-ad-connect-health-agent-installation"></a>Установка агента Azure AD Connect Health

В этой статье вы узнаете, как установить и настроить агенты Azure Connect Health для Azure Active Directory. Сведения о загрузке агентов см. в [этих инструкциях](how-to-connect-install-roadmap.md#download-and-install-azure-ad-connect-health-agent).

## <a name="requirements"></a>Требования

В следующей таблице перечислены требования для использования Azure AD Connect Health.

| Требование | Описание |
| --- | --- |
| Существует Azure AD Premium (P1 или P2) подписке.  |Azure AD Connect Health — это функция Azure AD Premium (P1 или P2). Дополнительные сведения см. в статье [Регистрация в Azure AD Premium](../fundamentals/active-directory-get-started-premium.md). <br /><br />Чтобы запустить бесплатную 30-дневную пробную версию, см. статью [Запуск пробной версии](https://azure.microsoft.com/trial/get-started-active-directory/). |
| Вы являетесь глобальным администратором в Azure AD. |По умолчанию только глобальные администраторы могут устанавливать и настраивать агенты работоспособности, получать доступ к порталу и выполнять любые операции в Azure AD Connect Health. Дополнительные сведения см. в статье [Администрирование каталога Azure AD](../fundamentals/active-directory-whatis.md). <br /><br /> С помощью управления доступом на основе ролей Azure (Azure RBAC) можно разрешить другим пользователям в организации получать доступ к Azure AD Connect Health. Дополнительные сведения см. в разделе [Azure RBAC for Azure AD Connect Health](how-to-connect-health-operations.md#manage-access-with-azure-rbac). <br /><br />**Важно**. для установки агентов используйте рабочую или учебную учетную запись. Нельзя использовать учетная запись Майкрософт. Дополнительные сведения см. в статье [Регистрация в Azure в качестве организации](../fundamentals/sign-up-organization.md). |
| Агент Azure AD Connect Health установлен на каждом целевом сервере. | Агенты работоспособности должны быть установлены и настроены на целевых серверах, чтобы они могли получать данные и предоставлять возможности мониторинга и аналитики. <br /><br />Например, чтобы получить данные из инфраструктуры службы федерации Active Directory (AD FS) (AD FS), необходимо установить агент на AD FS сервере и прокси-сервере веб – приложения. Аналогично, чтобы получить данные из локальной инфраструктуры доменных служб Azure AD (Azure AD DS), необходимо установить агент на контроллерах домена.  |
| Конечные точки службы Azure имеют исходящее подключение. | Во время установки и выполнения агенту требуется подключение к конечным точкам службы Azure AD Connect Health. Если брандмауэры блокируют исходящие подключения, добавьте [конечные точки исходящего подключения](how-to-connect-health-agent-install.md#outbound-connectivity-to-the-azure-service-endpoints) в список разрешений. |
|Исходящее подключение основано на IP-адресах. | Сведения о фильтрации брандмауэра по IP-адресам см. в разделе [диапазоны IP-адресов Azure](https://www.microsoft.com/download/details.aspx?id=56519).|
| Проверка TLS для исходящего трафика фильтруется или отключается. | Выполнение этапа регистрации агента или операции передачи данных может завершиться ошибкой, если проверка TLS или завершение исходящего трафика на сетевом уровне. Дополнительные сведения см. в разделе [Настройка проверки TLS](/previous-versions/tn-archive/ee796230(v=technet.10)). |
| Порты брандмауэра на сервере запускаются агентом. |Для работы агента необходимо открыть следующие порты брандмауэра, чтобы он мог взаимодействовать с конечными точками службы Azure AD Connect Health: <br /><li>TCP-порт 443</li><li>TCP-порт 5671</li> <br />Для последней версии агента не требуется порт 5671. Обновите до последней версии, чтобы требовалось только порт 443. Дополнительные сведения см. в статье [необходимые порты и протоколы для гибридной идентификации](./reference-connect-ports.md). |
| Если включена усиленная безопасность Internet Explorer, разрешите указанные веб-сайты.  |Если включена усиленная безопасность Internet Explorer, разрешите следующие веб-сайты на сервере, где устанавливается агент:<br /><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com</li><li>https:\//login.windows.net</li><li>HTTPS: \/ /aadcdn.msftauth.NET</li><li>Сервер федерации для вашей организации, которому доверяет Azure AD (например, HTTPS: \/ /STS.contoso.com);</li> <br />Дополнительные сведения см. [в разделе Настройка Internet Explorer](https://support.microsoft.com/help/815141/internet-explorer-enhanced-security-configuration-changes-the-browsing). Если у вас есть прокси-сервер в сети, см. Примечание, которое отображается в конце этой таблицы.|
| Установлена версия PowerShell 4,0 или более поздняя. | Windows Server 2012 включает PowerShell версии 3,0. Эта *версия недостаточно для* агента.</br></br> Windows Server 2012 R2 и более поздних версий включает в себя достаточно последнюю версию PowerShell.|
|FIPS (Федеральный стандарт обработки информации) отключен.|Агенты Azure AD Connect Health не поддерживают FIPS.|

> [!IMPORTANT]
> Windows Server Core не поддерживает установку агента Azure AD Connect Health.


> [!NOTE]
> При наличии сильно заблокированной и ограниченной среды необходимо добавить больше URL-адресов, чем список таблиц для усиленной безопасности Internet Explorer. Также добавьте URL-адреса, перечисленные в таблице в следующем разделе.  


### <a name="outbound-connectivity-to-the-azure-service-endpoints"></a>Исходящие подключения к конечным точкам службы Azure

Во время установки и выполнения агенту требуется подключение к конечным точкам службы Azure AD Connect Health. Если брандмауэры блокируют исходящие подключения, убедитесь, что URL-адреса в следующей таблице по умолчанию не заблокированы. 

Не отключайте мониторинг безопасности или проверку этих URL-адресов. Вместо этого разрешите им как можно более другой Интернет. 

Эти URL-адреса позволяют обмениваться данными с конечными точками службы Azure AD Connect Health. Далее в этой статье вы узнаете, как [проверить исходящие подключения](#test-connectivity-to-azure-ad-connect-health-service) с помощью `Test-AzureADConnectHealthConnectivity` .

| Доменная среда | Требуемые конечные точки службы Azure |
| --- | --- |
| Общие открытые | <li>&#42;.blob.core.windows.net </li><li>&#42;.aadconnecthealth.azure.com </li><li>&#42;. servicebus.windows.net-Port: 5671 (Эта конечная точка не является обязательной в последней версии агента).</li><li>&#42;.adhybridhealth.azure.com/;</li><li>https:\//management.azure.com </li><li>https:\//policykeyservice.dc.ad.msft.net/</li><li>https:\//login.windows.net</li><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>HTTPS: \/ /www.Office.com (Эта конечная точка используется только для целей обнаружения во время регистрации.)</li> <li>https://aadcdn.msftauth.net</li><li>https://aadcdn.msauth.net</li> |
| Azure для Германии | <li>*.blob.core.cloudapi.de; </li><li>*.servicebus.cloudapi.de; </li> <li>*.aadconnecthealth.microsoftazure.de; </li><li>https:\//management.microsoftazure.de </li><li>https:\//policykeyservice.aadcdi.microsoftazure.de </li><li>https:\//login.microsoftonline.de </li><li>https:\//secure.aadcdn.microsoftonline-p.de </li><li>HTTPS: \/ /www.Office.de (Эта конечная точка используется только для целей обнаружения во время регистрации.)</li> <li>https://aadcdn.msftauth.net</li><li>https://aadcdn.msauth.net</li> |
| Azure для государственных организаций | <li>&#42;.blob.core.usgovcloudapi.net </li> <li>&#42;.servicebus.usgovcloudapi.net </li> <li>&#42;.aadconnecthealth.microsoftazure.us </li> <li>https:\//management.usgovcloudapi.net </li><li>https:\//policykeyservice.aadcdi.azure.us </li><li>https:\//login.microsoftonline.us </li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>HTTPS: \/ /www.Office.com (Эта конечная точка используется только для целей обнаружения во время регистрации.)</li> <li>https://aadcdn.msftauth.net</li><li>https://aadcdn.msauth.net</li> |


## <a name="install-the-agent"></a>Установка агента.

Чтобы скачать и установить агент Azure AD Connect Health, выполните следующие действия. 

* Убедитесь, что соблюдены [требования](how-to-connect-health-agent-install.md#requirements) для Azure AD Connect Health.
* Приступая к работе с Azure AD Connect Health для AD FS:
    * [Скачайте агент Azure AD Connect Health для AD FS](https://go.microsoft.com/fwlink/?LinkID=518973).
    * См. [инструкции по установке](#install-the-agent-for-ad-fs).
* Начало работы с Azure AD Connect Health для синхронизации:
    * [Скачайте и установите последнюю версию Azure AD Connect.](https://go.microsoft.com/fwlink/?linkid=615771) Агент работоспособности для синхронизации устанавливается в составе Azure AD Connect установки (версия 1.0.9125.0 или более поздняя).
* Приступая к работе с Azure AD Connect Health для Azure AD DS:
    * [Скачайте агент Azure AD Connect Health для Azure AD DS](https://go.microsoft.com/fwlink/?LinkID=820540).
    * См. [инструкции по установке](#install-the-agent-for-azure-ad-ds).

## <a name="install-the-agent-for-ad-fs"></a>Установка агента для AD FS

> [!NOTE]
> Сервер AD FS должен отличаться от сервера синхронизации. Не устанавливайте агент AD FS на сервере синхронизации.
>

Перед установкой агента убедитесь, что имя узла сервера AD FS уникально и отсутствует в службе AD FS.
Чтобы запустить установку агента, дважды щелкните загруженный *exe* -файл. В первом окне выберите Install ( **установить**).

![Снимок экрана, показывающий окно установки агента Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install1.png)

После завершения установки нажмите кнопку **настроить сейчас**.

![Снимок экрана, показывающий сообщение с подтверждением для установки агента Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install2.png)

Откроется окно PowerShell для запуска процесса регистрации агента. При появлении запроса войдите в систему, используя учетную запись Azure AD с разрешениями на регистрацию агента. По умолчанию учетная запись глобального администратора имеет разрешения.

![Снимок экрана, показывающий окно входа для Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install3.png)

После входа в систему PowerShell продолжится. По завершении вы можете закрыть PowerShell. Настройка завершена.

На этом этапе службы агента должны запускаться автоматически, чтобы агент был безопасно загружать необходимые данные в облачную службу.

Если вы не удовлетворены всеми необходимыми компонентами, в окне PowerShell появятся предупреждения. Обязательно выполните [требования](how-to-connect-health-agent-install.md#requirements) перед установкой агента. На следующем снимке экрана показан пример этих предупреждений.

![Снимок экрана, показывающий Azure AD Connect Health AD FS настроить скрипт.](./media/how-to-connect-health-agent-install/install4.png)

Чтобы проверить, установлен ли агент, найдите на сервере следующие службы. Если вы выполнили настройку, эти службы должны работать. В противном случае они останавливаются до завершения настройки.

* Служба диагностики AD FS Azure AD Connect Health
* Служба AD FS получения ценной информации из данных о работоспособности Azure AD Connect
* Служба наблюдения AD FS Azure AD Connect Health

![Снимок экрана, показывающий Azure AD Connect Health AD FS Services.](./media/how-to-connect-health-agent-install/install5.png)


### <a name="enable-auditing-for-ad-fs"></a>Включение аудита для AD FS

> [!NOTE]
> Этот раздел относится только к AD FSным серверам. Вам не нужно выполнять эти действия на прокси-серверах приложений.
>

Функция анализа использования должна собирать и анализировать данные. Поэтому агенту Azure AD Connect Health требуются сведения в журналах аудита AD FS. Эти журналы не включены по умолчанию. Используйте следующие процедуры для включения аудита AD FS и выявления AD FS журналов аудита на серверах AD FS.

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2012-r2"></a>Включение аудита для AD FS на Windows Server 2012 R2

1. На начальном экране откройте **Диспетчер сервера**, а затем откройте **локальную политику безопасности**. На панели задач откройте **Диспетчер сервера**, а затем выберите **Сервис/локальная политика безопасности**.
2. Перейдите к папке *Security локальные Policies\User rightsing* . Затем дважды щелкните **создать аудит безопасности**.
3. На вкладке **Параметр локальной безопасности** убедитесь, что в списке указана учетная запись службы AD FS. Если он отсутствует в списке, выберите **Добавить пользователя или группу** и добавьте его в список. Затем нажмите кнопку **OK**.
4. Чтобы включить аудит, откройте окно командной строки с повышенными привилегиями. Затем выполните следующую команду. 
    
    `auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable`
1. Закройте вкладку **Параметры локальной безопасности**.
    >[!Important]
    >Следующие шаги необходимы только для основных серверов AD FS. 
1. Откройте оснастку **Управление AD FS**. (В **Диспетчер сервера** выберите **инструменты**  >  . **Управление AD FS**.)
1. На панели **действия** выберите **изменить служба федерации свойства**.
1. В диалоговом окне **свойства служба федерации** выберите вкладку **события** .
1. Установите флажки **успешный аудит и аудит сбоев** , а затем нажмите кнопку **ОК**.
1. Чтобы включить подробное ведение журнала с помощью PowerShell, используйте следующую команду: 

    `Set-AdfsProperties -LOGLevel Verbose`

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2016"></a>Включение аудита для AD FS на Windows Server 2016

1. На начальном экране откройте **Диспетчер сервера**, а затем откройте **локальную политику безопасности**. На панели задач откройте **Диспетчер сервера**, а затем выберите **Сервис/локальная политика безопасности**.
2. Перейдите в папку *Security локальные Policies\User rightsing* , а затем дважды щелкните **создать Аудиты безопасности**.
3. На вкладке **Параметр локальной безопасности** убедитесь, что в списке указана учетная запись службы AD FS. Если он отсутствует в списке, выберите **Добавить пользователя или группу** и добавьте учетную запись службы AD FS в список. Затем нажмите кнопку **OK**.
4. Чтобы включить аудит, откройте окно командной строки с повышенными привилегиями. Затем выполните следующую команду. 

    `auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable`
1. Закройте вкладку **Параметры локальной безопасности**.
    >[!Important]
    >Следующие шаги необходимы только для основных серверов AD FS.
1. Откройте оснастку **Управление AD FS**. (В **Диспетчер сервера** выберите **инструменты**  >  . **Управление AD FS**.)
1. На панели **действия** выберите **изменить служба федерации свойства**.
1. В диалоговом окне **свойства служба федерации** выберите вкладку **события** .
1. Установите флажки **успешный аудит и аудит сбоев** , а затем нажмите кнопку **ОК**. Аудит успехов и аудит отказов должны быть включены по умолчанию.
1. Откройте окно PowerShell и выполните следующую команду: 

    `Set-AdfsProperties -AuditLevel Verbose`

Уровень аудита "базовый" включен по умолчанию. Дополнительные сведения см. [в разделе AD FS аудит улучшений в Windows Server 2016](/windows-server/identity/ad-fs/technical-reference/auditing-enhancements-to-ad-fs-in-windows-server).


#### <a name="to-locate-the-ad-fs-audit-logs"></a>Поиск журналов аудита AD FS

1. Откройте средство **просмотра событий**.
2. Перейдите в раздел **журналы Windows** и выберите **Безопасность**.
3. Справа выберите **фильтровать текущие журналы**.
4. В качестве **источника событий** выберите **AD FS аудит**.

    Дополнительные сведения о журналах аудита см. в разделе [Operations](reference-connect-health-faq.md#operations-questions)informations.

    ![Снимок экрана, показывающий окно фильтра текущий журнал. В поле "источники событий" выбран параметр "аудит AD FS".](./media/how-to-connect-health-agent-install/adfsaudit.png)

> [!WARNING]
> Групповая политика может отключить аудит AD FS. Если AD FS аудит отключен, аналитика использования сведений о действиях входа недоступна. Убедитесь, что у вас нет групповой политики, которая отключает аудит AD FS.
>


## <a name="install-the-agent-for-sync"></a>Установка агента для синхронизации

Агент Azure AD Connect Health для синхронизации устанавливается автоматически в последней версии Azure AD Connect. Чтобы использовать Azure AD Connect для синхронизации, [скачайте последнюю версию Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) и установите ее.

Чтобы проверить, установлен ли агент, найдите на сервере приведенные ниже службы. Если вы завершили настройку, службы уже должны быть запущены. В противном случае службы останавливаются до завершения настройки.

* Служба Sync Insights Azure AD Connect Health
* Служба Sync Monitoring Azure AD Connect Health

![Снимок экрана, на котором показана работающая Azure AD Connect Health для служб Sync Services на сервере.](./media/how-to-connect-health-agent-install/services.png)

> [!NOTE]
> Помните, что для использования Azure AD Connect Health необходимо иметь Azure AD Premium (P1 или P2). Если у вас нет Azure AD Premium, вы не сможете завершить настройку в портал Azure. Дополнительные сведения см. в разделе [требования](how-to-connect-health-agent-install.md#requirements).
>
>

## <a name="manually-register-azure-ad-connect-health-for-sync"></a>Регистрация Azure AD Connect Health для синхронизации вручную

Если Azure AD Connect Health для регистрации агента синхронизации не удается выполнить после успешной установки Azure AD Connect, можно зарегистрировать агент вручную с помощью команды PowerShell.

> [!IMPORTANT]
> Используйте эту команду PowerShell только в случае сбоя регистрации агента после установки Azure AD Connect.
>
>

Вручную Зарегистрируйте агент Azure AD Connect Health для синхронизации с помощью следующей команды PowerShell. Службы Azure AD Connect Health запустятся после того, как агент будет успешно зарегистрирован.

`Register-AzureADConnectHealthSyncAgent -AttributeFiltering $false -StagingMode $false`

Команда принимает следующие параметры:

* **Аттрибутефилтеринг**: `$true` (по умолчанию), если Azure AD Connect не синхронизируется набор атрибутов по умолчанию и был настроен для использования отфильтрованного набора атрибутов. В противном случае используйте коллекцию `$false`.
* **Стагингмоде**: `$false` (по умолчанию), если Azure AD Connect сервер *не* находится в промежуточном режиме. Если сервер настроен для работы в промежуточном режиме, используйте `$true` .

При появлении запроса на проверку подлинности используйте ту же учетную запись глобального администратора (например admin@domain.onmicrosoft.com ,), которая использовалась для настройки Azure AD Connect.

## <a name="install-the-agent-for-azure-ad-ds"></a>Установка агента для Azure AD DS

Чтобы запустить установку агента, дважды щелкните загруженный *exe* -файл. В первом окне выберите Install ( **установить**).

![Снимок экрана, показывающий Azure AD Connect Health агент для AD DS установки.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install1.png)

После завершения установки нажмите кнопку **настроить сейчас**.

![Снимок экрана, показывающий окно, которое завершает установку агента Azure AD Connect Health для AD DS Azure.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install2.png)

Откроется окно командной строки. Выполняется PowerShell `Register-AzureADConnectHealthADDSAgent` . При появлении запроса войдите в Azure.

![Снимок экрана, показывающий окно входа для агента Azure AD Connect Health для AD DS Azure.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install3.png)

После входа в систему PowerShell продолжится. По завершении вы можете закрыть PowerShell. Настройка завершена.

На этом этапе службы должны запускаться автоматически, позволяя агенту отслеживать и собирать данные. Если вы не удовлетворены всеми предварительными требованиями, описанными в предыдущих разделах, в окне PowerShell появятся предупреждения. Обязательно выполните [требования](how-to-connect-health-agent-install.md#requirements) перед установкой агента. На следующем снимке экрана показан пример этих предупреждений.

![Снимок экрана с предупреждением агента Azure AD Connect Health для AD DS конфигурации Azure.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install4.png)

Чтобы проверить, установлен ли агент, найдите следующие службы на контроллере домена:

* служба AD DS для Azure AD Connect Health;
* служба наблюдения AD DS Azure AD Connect Health.

Если вы выполнили настройку, эти службы должны работать. В противном случае они останавливаются до завершения настройки.

![Снимок экрана, показывающий выполняющиеся службы на контроллере домена.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install5.png)

### <a name="quickly-install-the-agent-on-multiple-servers"></a>Быстрая установка агента на нескольких серверах

1. Создайте учетную запись пользователя в Azure AD. Защитите его с помощью пароля.
2. Назначьте роль **владельца** для этой локальной учетной записи Azure AD в Azure AD Connect Health с помощью портала. Выполните [следующие действия](how-to-connect-health-operations.md#manage-access-with-azure-rbac). Назначьте роль всем экземплярам службы. 
3. Скачайте *exe* MSI файл на локальном контроллере домена для установки.
4. Выполните следующий сценарий. Замените параметры новой учетной записью пользователя и паролем. 

    ```powershell
    AdHealthAddsAgentSetup.exe /quiet
    Start-Sleep 30
    $userName = "NEWUSER@DOMAIN"
    $secpasswd = ConvertTo-SecureString "PASSWORD" -AsPlainText -Force
    $myCreds = New-Object System.Management.Automation.PSCredential ($userName, $secpasswd)
    import-module "C:\Program Files\Azure Ad Connect Health Adds Agent\PowerShell\AdHealthAdds"
     
    Register-AzureADConnectHealthADDSAgent -Credential $myCreds
    
    ```

По завершении можно удалить доступ для локальной учетной записи, выполнив одну или несколько следующих задач. 
* Удалите назначение ролей для локальной учетной записи для Azure AD Connect Health.
* Смена пароля для локальной учетной записи. 
* Отключите локальную учетную запись Azure AD.
* Удалите локальную учетную запись Azure AD.  

## <a name="register-the-agent-by-using-powershell"></a>Регистрация агента с помощью PowerShell

После установки соответствующего файла *setup.exe* агента можно зарегистрировать агент с помощью следующих команд PowerShell в зависимости от роли. Откройте окно PowerShell и выполните соответствующую команду:

```powershell
    Register-AzureADConnectHealthADFSAgent
    Register-AzureADConnectHealthADDSAgent
    Register-AzureADConnectHealthSyncAgent

```

> [!NOTE]
> Чтобы зарегистрироваться в независимых облаках, используйте следующие командные строки:
>
> ```powershell
> Register-AzureADConnectHealthADFSAgent -UserPrincipalName upn-of-the-user
> Register-AzureADConnectHealthADDSAgent -UserPrincipalName upn-of-the-user
> Register-AzureADConnectHealthSyncAgent -UserPrincipalName upn-of-the-user
> ```
>


Эти команды принимаются в `Credential` качестве параметра для завершения регистрации в неинтерактивном режиме или для завершения регистрации на компьютере, на котором выполняется Server Core. Помните о следующем:
* Вы можете записать `Credential` переменную PowerShell, которая передается в качестве параметра.
* Вы можете указать любое удостоверение Azure AD, имеющее разрешения на регистрацию агентов и *не* поддерживающих многофакторную проверку подлинности.
* По умолчанию глобальные администраторы имеют разрешения на регистрацию агентов. Для этого шага можно также разрешить удостоверения с меньшими привилегиями. Дополнительные сведения см. в разделе [Azure RBAC](how-to-connect-health-operations.md#manage-access-with-azure-rbac).

```powershell
    $cred = Get-Credential
    Register-AzureADConnectHealthADFSAgent -Credential $cred

```

## <a name="configure-azure-ad-connect-health-agents-to-use-http-proxy"></a>Настройка агентов Azure AD Connect Health для использования прокси-сервера HTTP

Можно настроить агенты Azure AD Connect Health для работы с прокси-сервером HTTP.

> [!NOTE]
> * Тип `Netsh WinHttp set ProxyServerAddress` не поддерживается. Для выполнения веб-запросов агент использует System.Net вместо служб HTTP Windows.
> * Настроенный адрес прокси-сервера HTTP используется для передачи зашифрованных сообщений HTTPS.
> * Прокси-серверы с проверкой подлинности (с помощью HTTPBasic) не поддерживаются.
>
>

### <a name="change-the-agent-proxy-configuration"></a>Изменение конфигурации прокси-сервера агента

Чтобы настроить агент Azure AD Connect Health для использования прокси-сервера HTTP, можно:
* Импорт существующих параметров прокси-сервера.
* Укажите адреса прокси-сервера вручную.
* Очистите существующую конфигурацию прокси-сервера.

> [!NOTE]
> Чтобы обновить параметры прокси-сервера, необходимо перезапустить все службы Azure AD Connect Health Agent. Выполните следующую команду:
>
> `Restart-Service AzureADConnectHealth*`

#### <a name="import-existing-proxy-settings"></a>Импорт существующих параметров прокси-сервера

Можно импортировать параметры прокси-сервера HTTP Internet Explorer, чтобы агенты Azure AD Connect Health могли использовать эти параметры. На каждом из серверов, на которых выполняется агент работоспособности, выполните следующую команду PowerShell:

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromInternetSettings
```

Вы можете импортировать параметры прокси-сервера WinHTTP, чтобы агенты Azure AD Connect Health могли их использовать. На каждом из серверов, на которых выполняется агент работоспособности, выполните следующую команду PowerShell:

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromWinHttp
```

#### <a name="specify-proxy-addresses-manually"></a>Указание адресов прокси-сервера вручную

Вы можете вручную указать прокси-сервер. На каждом из серверов, на которых выполняется агент работоспособности, выполните следующую команду PowerShell:

```powershell
Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress address:port
```

Ниже приведен пример: 

`Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress myproxyserver: 443`

В данном примере:
* `address`Параметр может быть разрешимым именем сервера DNS или IPv4-адресом.
* Можно опустить `port` . В этом случае по умолчанию используется порт 443.

#### <a name="clear-the-existing-proxy-configuration"></a>Очистка существующей конфигурации прокси-сервера

Чтобы очистить текущую конфигурацию прокси-сервера, выполните следующую команду:

```powershell
Set-AzureAdConnectHealthProxySettings -NoProxy
```

### <a name="read-current-proxy-settings"></a>Считывание текущих параметров прокси-сервера

Вы можете прочитать текущие параметры прокси-сервера, выполнив следующую команду:

```powershell
Get-AzureAdConnectHealthProxySettings
```

## <a name="test-connectivity-to-azure-ad-connect-health-service"></a>Проверка подключения к службе Azure AD Connect Health

Иногда агент Azure AD Connect Health может потерять связь со службой Azure AD Connect Health. Причиной такой потери может быть проблема с сетью, проблемы с разрешениями и другие проблемы.

Если агент не может отправлять данные в службу Azure AD Connect Health дольше двух часов, на портале появляется следующее предупреждение: "служба работоспособности данные не являются актуальными". 

Вы можете узнать, может ли затронутый агент Azure AD Connect Health отправлять данные в службу Azure AD Connect Health, выполнив следующую команду PowerShell:

```powershell
Test-AzureADConnectHealthConnectivity -Role ADFS
```

Параметр role в настоящее время принимает следующие значения:

* ADFS
* Синхронизация
* ADDS

> [!NOTE]
> Чтобы использовать средство подключения, необходимо сначала зарегистрировать агент. Если вы не можете завершить регистрацию агента, убедитесь, что выполнены все [требования](how-to-connect-health-agent-install.md#requirements) к Azure AD Connect Health. Подключение проверяется по умолчанию во время регистрации агента.
>
>

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь со следующими статьями:

* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Azure AD Connect Health операции](how-to-connect-health-operations.md)
* [Использование Azure AD Connect Health с AD FS](how-to-connect-health-adfs.md)
* [Использование Azure AD Connect Health для синхронизации](how-to-connect-health-sync.md)
* [Использование Azure AD Connect Health с Azure AD DS](how-to-connect-health-adds.md)
* [Часто задаваемые вопросы об Azure AD Connect Health](reference-connect-health-faq.md)
* [Журнал версий Azure AD Connect Health](reference-connect-health-version-history.md)
