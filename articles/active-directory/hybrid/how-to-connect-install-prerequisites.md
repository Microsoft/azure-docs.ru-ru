---
title: 'Azure AD Connect: оборудование и необходимые компоненты | Документация Майкрософт'
description: В этой статье описаны необходимые условия и требования к оборудованию для Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 91b88fda-bca6-49a8-898f-8d906a661f07
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/16/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e758933b80efbf36dc263b7bd7d2d3c45a59a9f8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102426796"
---
# <a name="prerequisites-for-azure-ad-connect"></a>Необходимые условия для Azure AD Connect
В этой статье описаны необходимые условия и требования к оборудованию для Azure Active Directory (Azure AD) Connect.

## <a name="before-you-install-azure-ad-connect"></a>Перед установкой Azure AD Connect
Прежде чем установить Azure AD Connect и обновить DirSync, вам потребуется ряд элементов.

### <a name="azure-ad"></a>Azure AD
* Необходим клиент Azure AD. Вы получите его с [бесплатной пробной версией Azure](https://azure.microsoft.com/pricing/free-trial/). Вы можете использовать один из следующих порталов для управления Azure AD Connect:
  * [Портал Azure](https://portal.azure.com).
  * [Портал Office](https://portal.office.com).
* [Добавьте и подтвердите домен](../fundamentals/add-custom-domain.md) , который вы планируете использовать в Azure AD. Например, если вы планируете использовать contoso.com для пользователей, убедитесь, что этот домен проверен и вы не используете только домен contoso.onmicrosoft.com по умолчанию.
* Клиент Azure AD предоставляет по умолчанию (50 000) объекты. При проверке домена ограничение увеличивается до 300 000 объектов. Если вам нужно еще больше объектов в Azure AD, откройте обращение в службу поддержки, чтобы увеличить это ограничение. Если требуется более 500 000 объектов, требуется лицензия, например Microsoft 365, Azure AD Premium или Enterprise Mobility + Security.

### <a name="prepare-your-on-premises-data"></a>Подготовка локальных данных
* Используйте [идфикс](https://support.office.com/article/Install-and-run-the-Office-365-IdFix-tool-f4bd2439-3e41-4169-99f6-3fabdfa326ac) для выявления ошибок, таких как дублирование и форматирование, в каталоге перед синхронизацией с Azure AD и Microsoft 365.
* Ознакомьтесь с [дополнительными функциями синхронизации, которые можно включить в Azure AD](how-to-connect-syncservice-features.md), и оцените, какие функции следует включить.

### <a name="on-premises-active-directory"></a>Локальная служба Active Directory
* Версия схемы Active Directory и функциональный уровень леса должны быть Windows Server 2003 или более поздней версии. Контроллеры домена могут запускать любую версию при условии соблюдения требований к версии схемы и уровня леса.
* Если вы планируете использовать компонент *обратной записи паролей*, контроллеры домена должны быть на Windows Server 2012 или более поздней версии.
* Контроллер домена, используемый Azure AD, должен быть доступен для записи. Использование контроллера домена только для чтения (RODC) *не поддерживается*, и Azure AD Connect не соответствует перенаправлениям записи.
* Использование локальных лесов или доменов с точками (имя содержит точку ".") NetBIOS-имена *не поддерживаются*.
* Рекомендуется [Включить корзину Active Directory](how-to-connect-sync-recycle-bin.md).

### <a name="powershell-execution-policy"></a>Политика выполнения PowerShell
Azure Active Directory Connect выполняет подписанные скрипты PowerShell в процессе установки. Убедитесь, что политика выполнения PowerShell разрешит выполнение скриптов.

Рекомендуемая политика выполнения во время установки — "RemoteSigned".

Дополнительные сведения о настройке политики выполнения PowerShell см. в разделе [Set-ExecutionPolicy](/powershell/module/microsoft.powershell.security/set-executionpolicy).


### <a name="azure-ad-connect-server"></a>Сервер Azure AD Connect
Сервер Azure AD Connect содержит важные данные удостоверения. Важно, чтобы административный доступ к этому серверу был надежно защищен. Следуйте рекомендациям по [защите привилегированного доступа](/windows-server/identity/securing-privileged-access/securing-privileged-access). 

Azure AD Connect сервер должен рассматриваться как компонент уровня 0, как описано в [модели административного уровня Active Directory](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material) . 

Дополнительные сведения о защите среды Active Directory см. в статье рекомендации [по обеспечению безопасности Active Directory](/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory).

#### <a name="installation-prerequisites"></a>Предварительные требования для установки

- Azure AD Connect должны быть установлены на присоединенном к домену сервере Windows Server 2012 или более поздней версии. 
- Azure AD Connect невозможно установить на Small Business Server или Windows Server Essentials до 2019 (поддерживается Windows Server Essentials 2019). Сервер должен использовать Windows Server Standard или более поздней версии. 
- На сервере Azure AD Connect должен быть установлен полный графический интерфейс пользователя. Установка Azure AD Connect в Windows Server Core не поддерживается. 
- Azure AD Connect сервер не должен включать групповая политика записи PowerShell, если для управления конфигурацией службы федерации Active Directory (AD FS) (AD FS) используется мастер Azure AD Connect. Вы можете включить транскрипцию PowerShell, если для управления конфигурацией синхронизации используется мастер Azure AD Connect. 
- Если развертывается AD FS: 
    - Серверы, на которых установлены AD FS или прокси-служба веб – приложения, должны иметь версию Windows Server 2012 R2 или более позднюю. удаленное управление Windows . 
    - Необходимо настроить сертификаты TLS/SSL. Дополнительные сведения см. в разделе [Управление протоколами SSL/TLS и комплектами шифров для AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs) и [управления SSL-сертификатами в AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-certificates-ad-fs-wap).
    - Необходимо настроить разрешение имен. 
- Если для глобальных администраторов включен MFA, URL-адрес https://secure.aadcdn.microsoftonline-p.com *должен* находиться в списке надежных сайтов. Вам будет предложено добавить этот сайт в список надежных сайтов при запросе на запрос MFA, который еще не был добавлен. Добавить его в список надежных сайтов можно в Internet Explorer.
- Если вы планируете использовать Azure AD Connect Health для синхронизации, убедитесь, что выполнены также необходимые условия для Azure AD Connect Health. Дополнительные сведения см. в разделе [Установка агента Azure AD Connect Health](how-to-connect-health-agent-install.md).

#### <a name="harden-your-azure-ad-connect-server"></a>Усиление защиты сервера Azure AD Connect 
Рекомендуется зафиксировать сервер Azure AD Connect для снижения уязвимой зоны безопасности для этого критического компонента ИТ. Следуйте этим рекомендациям, чтобы устранить некоторые угрозы безопасности в Организации.

- Рассматривайте Azure AD Connect так же, как и контроллер домена, и другие ресурсы уровня 0. Дополнительные сведения см. в разделе [модель административного уровня Active Directory](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material).
- Ограничьте административный доступ к серверу Azure AD Connect только администраторам домена или другим жестко управляемым группам безопасности.
- Создайте [выделенную учетную запись для всех сотрудников с привилегированным доступом](/windows-server/identity/securing-privileged-access/securing-privileged-access). Администраторы не должны просматривать веб-узел, проверять электронную почту и выполнять повседневные задачи по повышению производительности с помощью учетных записей с высоким уровнем привилегий.
- Следуйте инструкциям, приведенным в статье [Защита привилегированного доступа](/windows-server/identity/securing-privileged-access/securing-privileged-access). 
- Запретите использование проверки подлинности NTLM на сервере AADConnect. Вот несколько способов сделать это: [ограничения NTLM на сервере AADConnect](/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-outgoing-ntlm-traffic-to-remote-servers) и [ограничения NTLM в домене](/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain) .
- Убедитесь, что у каждого компьютера есть уникальный пароль локального администратора. Дополнительные сведения см. в статье [решение "пароль локального администратора" (Lap)](https://support.microsoft.com/help/3062591/microsoft-security-advisory-local-administrator-password-solution-laps) , позволяющее настроить уникальные случайные пароли на каждой рабочей станции и сервере для хранения их в Active Directory, защищенных ACL. Только допустимые, авторизованные пользователи могут читать или запрашивать сброс паролей учетной записи локального администратора. Вы можете получить Lap для использования на рабочих станциях и серверах из [центра загрузки Майкрософт](https://www.microsoft.com/download/details.aspx?id=46899). Дополнительные рекомендации по работе с рабочей средой с Lap и привилегированным доступом (лапы) можно найти в статье [эксплуатационные стандарты на основе принципа чистого источника](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material#operational-standards-based-on-clean-source-principle). 
- Реализуйте выделенные [рабочие станции привилегированного доступа](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/) для всех сотрудников, имеющих привилегированный доступ к информационным системам вашей организации. 
- Следуйте этим [дополнительным рекомендациям](/windows-server/identity/ad-ds/plan/security-best-practices/reducing-the-active-directory-attack-surface) , чтобы уменьшить уязвимую область Active Directoryной среды.


### <a name="sql-server-used-by-azure-ad-connect"></a>SQL Server, используемый Azure AD Connect
* Azure AD Connect требуется база данных SQL Server для хранения учетных данных. По умолчанию устанавливается SQL Server 2012 Express LocalDB (легкая версия SQL Server Express). Размер SQL Server Express имеет ограничение в 10 ГБ, что позволяет управлять приблизительно 100 000 объектами. Если необходимо управлять большим объемом объектов каталога, укажите другой экземпляр SQL Server в мастере установки. Тип установки SQL Server может повлиять на [производительность Azure AD Connect](./plan-connect-performance-factors.md#sql-database-factors).
* Если вы используете другую установку SQL Server, эти требования относятся к следующим требованиям.
  * Azure AD Connect поддерживает все версии SQL Server от 2012 (с последним пакетом обновления) до SQL Server 2019. База данных SQL Azure *не поддерживается* в качестве базы данных.
  * Необходимо использовать параметры сортировки SQL без учета регистра. Их можно определить по суффиксу \_CI_ в имени. Использование параметров сортировки с учетом регистра, идентифицируемых \_ CS_ в имени, *не поддерживается*.
  * В каждом экземпляре SQL Server может быть только один модуль синхронизации. Совместное использование экземпляра SQL с FIM/MIM Sync, DirSync или Azure AD Sync *не поддерживается*.

### <a name="accounts"></a>Учетные записи
* Необходимо иметь учетную запись глобального администратора Azure AD для клиента Azure AD, с которым вы хотите выполнить интеграцию. Эта учетная запись должна быть *учетной записью учебного заведения или организации* и не может быть *учетная запись Майкрософт*.
* Если вы используете [быстрые параметры](reference-connect-accounts-permissions.md#express-settings-installation) или обновляете DirSync, у вас должна быть учетная запись администратора предприятия для локального Active Directory.
* Если вы используете путь установки настраиваемых параметров, у вас есть дополнительные параметры. Дополнительные сведения см. в разделе [Параметры выборочной установки](reference-connect-accounts-permissions.md#custom-installation-settings).

### <a name="connectivity"></a>Возможность подключения
* Для серверов Azure AD Connect требуется разрешение DNS как для интрасети, так и для Интернета. DNS-сервер должен иметь возможность разрешения имен как для локальной службы Active Directory, так и для конечных точек Azure AD.
* Azure AD Connect требуется сетевое подключение ко всем настроенным доменам
* Если в интрасети есть брандмауэры и необходимо открыть порты между серверами Azure AD Connect и контроллерами домена, см. Дополнительные сведения в разделе [Azure AD Connect Ports](reference-connect-ports.md) .
* Если прокси-сервер или брандмауэр ограничивает доступ к URL-адресам, необходимо открыть URL-адреса, описанные в списке URL-адресов [и диапазонов IP-адреса Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2) . См. также [портал Azure URL-адреса списка надежных адресов в брандмауэре или прокси-сервере](../../azure-portal/azure-portal-safelist-urls.md?tabs=public-cloud).
  * Если вы используете Microsoft Cloud в Германии или в облаке Microsoft Azure для государственных организаций, см. статью [рекомендации по экземплярам службы синхронизации Azure AD Connect](reference-connect-instances.md) для URL-адресов.
* Azure AD Connect (версия 1.1.614.0 и более поздние версии) по умолчанию использует TLS 1.2 для шифрования связи между модулем синхронизации и Azure AD. Если TLS 1.2 не поддерживается для базовой операционной системы, Azure AD Connect последовательно пытается перейти на более старые протоколы (TLS 1.1 и TLS 1.0).
* До версии 1.1.614.0 Azure AD Connect по умолчанию использует TLS 1.0 для шифрования связи между модулем синхронизации и Azure AD. Чтобы перейти на использование TLS 1.2, нужно выполнить инструкции из раздела [Включение протокола TLS 1.2 для Azure AD Connect](#enable-tls-12-for-azure-ad-connect).
* Если для подключения к Интернету используется исходящий прокси-сервер, то для мастера установки необходимо добавить следующий параметр в файле **C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config** и Azure AD Connect Sync, чтобы иметь возможность подключения к Интернету и Azure AD. Этот текст необходимо добавить в конец указанного файла. В этом коде *&lt; PROXYADDRESS &gt;* представляет фактический IP-адрес или имя узла прокси-сервера.

    ```
        <system.net>
            <defaultProxy>
                <proxy
                usesystemdefault="true"
                proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
                bypassonlocal="true"
                />
            </defaultProxy>
        </system.net>
    ```

* Если для прокси-сервера требуется проверка подлинности, [учетная запись службы](reference-connect-accounts-permissions.md#adsync-service-account) должна находиться в домене. Используйте путь установки настраиваемых параметров, чтобы указать [пользовательскую учетную запись службы](how-to-connect-install-custom.md#install-required-components). Кроме того, для machine.config потребуется другое изменение. После этого изменения в machine.config мастер установки и модуль синхронизации отвечают на запросы проверки подлинности от прокси-сервера. На страницах мастера установки, исключая страницу **Настройка** , используются учетные данные пользователя, выполнившего вход. На странице **Настройка** в конце работы мастера установки контекст переключается на созданную [учетную запись службы](reference-connect-accounts-permissions.md#adsync-service-account) . Раздел machine.config должен выглядеть следующим образом:

    ```
        <system.net>
            <defaultProxy enabled="true" useDefaultCredentials="true">
                <proxy
                usesystemdefault="true"
                proxyaddress="http://<PROXYADDRESS>:<PROXYPORT>"
                bypassonlocal="true"
                />
            </defaultProxy>
        </system.net>
    ```

* Если конфигурация прокси-сервера выполняется в существующей программе установки, то для того, чтобы Azure AD Connect прочитать конфигурацию прокси-сервера и обновить поведение, необходимо перезапустить **службу Microsoft Azure AD Sync** . 
* Когда в процессе синхронизации каталогов Azure AD Connect отправляет веб-запрос в Azure AD, то для ответа Azure AD может потребоваться до 5 минут. Для прокси-серверов обычно используется конфигурация времени ожидания простоя подключения. Убедитесь, что для конфигурации задано не менее 6 минут.

Дополнительные сведения см. в разделе MSDN об [элементе прокси по умолчанию](/dotnet/framework/configure-apps/file-schema/network/defaultproxy-element-network-settings).
В случае проблем с подключением изучите статью [Устранение неполадок подключения в Azure AD Connect](tshoot-connect-connectivity.md).

### <a name="other"></a>Другое
Необязательно. Используйте тестовую учетную запись пользователя для проверки синхронизации.

## <a name="component-prerequisites"></a>Предварительные требования к компонентам
### <a name="powershell-and-net-framework"></a>PowerShell и платформа .NET Framework
Работа Azure AD Connect основана на Microsoft PowerShell и .NET Framework 4.5.1. На сервере должна быть установлена эта или более поздняя версия. В зависимости от версии Windows Server выполните следующие действия.

* Windows Server 2012 R2
  * Microsoft PowerShell устанавливается по умолчанию. Никаких действий не требуется.
  * Платформа .NET Framework 4.5.1 и более поздних версий распространяется через Центр обновления Windows. Убедитесь, что установлены последние обновления для Windows Server на панели управления.
* Windows Server 2012
  * Последняя версия Microsoft PowerShell доступна в Windows Management Framework 4,0, доступной в [центре загрузки Майкрософт](https://www.microsoft.com/downloads).
  * Платформа .NET Framework 4.5.1 и более поздние версии доступны в [центре загрузки Майкрософт](https://www.microsoft.com/downloads).


### <a name="enable-tls-12-for-azure-ad-connect"></a>Включение протокола TLS 1.2 для Azure AD Connect
До версии 1.1.614.0 Azure AD Connect по умолчанию использует TLS 1.0 для шифрования связи между сервером модуля синхронизации и Azure AD. Вы можете настроить приложения .NET на использование TLS 1,2 по умолчанию на сервере. Дополнительные сведения о TLS 1,2 см. в [статье рекомендации по безопасности в майкрософт 2960358](/security-updates/SecurityAdvisories/2015/2960358).

1. Убедитесь, что для вашей операционной системы установлено исправление .NET 4.5.1. Дополнительные сведения см. в [статье Microsoft Security совет 2960358](/security-updates/SecurityAdvisories/2015/2960358). Это исправление или его более поздняя версия может быть уже установлена на вашем сервере.

1. Для любой операционной системы задайте этот ключ реестра и перезагрузите сервер.
    ```
    HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319
    "SchUseStrongCrypto"=dword:00000001
    ```
1. Если вы также хотите включить TLS 1,2 между сервером модуля синхронизации и удаленным SQL Server, убедитесь, что установлены требуемые версии для [поддержки TLS 1,2 для Microsoft SQL Server](https://support.microsoft.com/kb/3135244).

### <a name="dcom-prerequisites-on-the-synchronization-server"></a>Необходимые компоненты DCOM на сервере синхронизации
Во время установки службы синхронизации Azure AD Connect проверяет наличие следующего раздела реестра:

- HKEY_LOCAL_MACHINE: Софтваре\микрософт\оле

В этом разделе реестра Azure AD Connect проверит наличие и отсутствие повреждений следующих значений: 

- [мачинеакцессрестриктион](/windows/win32/com/machineaccessrestriction)
- [мачинелаунчрестриктион](/windows/win32/com/machinelaunchrestriction)
- [дефаултлаунчпермиссион](/windows/win32/com/defaultlaunchpermission)

## <a name="prerequisites-for-federation-installation-and-configuration"></a>Предварительные требования для установки и настройки федерации
### <a name="windows-remote-management"></a>Удаленное управление Windows
При использовании Azure AD Connect для развертывания AD FS или прокси веб-приложения (WAP) Проверьте следующие требования:

* Если целевой сервер присоединен к домену, убедитесь, что удаленное управление Windows включено.
  * В окне командной строки PowerShell с повышенными привилегиями используйте команду `Enable-PSRemoting –force` .
* Если целевой сервер является компьютером WAP, не присоединенным к домену, существует ряд дополнительных требований:
  * На целевом компьютере (WAP):
    * Убедитесь, что служба служба удаленного управления Windows или WS-Management (WinRM) запущена с помощью оснастки "службы".
    * В окне командной строки PowerShell с повышенными привилегиями используйте команду `Enable-PSRemoting –force` .
  * На компьютере, на котором работает мастер (если целевой компьютер не присоединен к домену или является недоверенным доменом):
    * В окне командной строки PowerShell с повышенными привилегиями используйте команду `Set-Item.WSMan:\localhost\Client\TrustedHosts –Value <DMZServerFQDN> -Force –Concatenate` .
    * В диспетчере сервера выполните следующие действия.
      * Добавьте узел WAP DMZ в пул компьютеров. В диспетчере сервера выберите **Управление**  >  **Добавить серверы**, а затем используйте вкладку **DNS** .
      * На вкладке **Диспетчер сервера все серверы** щелкните правой кнопкой мыши сервер WAP и выберите пункт **Управление как**. Введите локальные учетные данные (не домен) для компьютера WAP.
      * Чтобы проверить удаленное подключение PowerShell, на вкладке **Диспетчер сервера все серверы** щелкните правой кнопкой мыши сервер WAP и выберите пункт **Windows PowerShell**. Должен открыться удаленный сеанс PowerShell, чтобы обеспечить возможность установки удаленных сеансов PowerShell.

### <a name="tlsssl-certificate-requirements"></a>Требования к TLS- и SSL-сертификатам
* Мы рекомендуем использовать один и тот же сертификат TLS/SSL для всех узлов фермы AD FS и всех прокси-серверов.
* Это должен быть сертификат X509.
* На серверах федерации в тестовой лабораторной среде можно использовать самозаверяющий сертификат. Для рабочей среды рекомендуется получить сертификат из общедоступного центра сертификации.
  * Если вы используете сертификат, который не является общедоступным, убедитесь, что сертификат, установленный на каждом прокси-сервере, является доверенным как на локальном сервере, так и на всех серверах федерации.
* Идентификатор сертификата должен совпадать с именем службы федерации (например sts.contoso.com).
  * Удостоверение представляет собой либо расширение "альтернативное имя субъекта" (SAN) типа dNSName, либо, если нет записей SAN, имя субъекта указано как общее имя.
  * В сертификате может присутствовать несколько записей SAN, если один из них соответствует имени службы федерации.
  * Если вы планируете использовать Workplace Join, требуется дополнительная сеть SAN со значением **enterpriseregistration.** , за которым следует суффикс имени участника-пользователя (UPN) вашей организации, например enterpriseregistration.contoso.com.
* Сертификаты, основанные на ключах следующего поколения (CNG) и поставщиках хранилища ключей (KSP), не поддерживаются. В результате необходимо использовать сертификат, основанный на поставщике служб шифрования (CSP), а не KSP.
* Поддерживаются групповые сертификаты.

### <a name="name-resolution-for-federation-servers"></a>Разрешение имен для серверов федерации
* Настройте записи DNS для имени AD FS (например, sts.contoso.com) как для интрасети (внутреннего DNS-сервера), так и для экстрасети (общедоступный DNS-сервер через регистратор домена). Для записи DNS интрасети обязательно используйте записи A, а не записи CNAME. Для правильной работы проверки подлинности Windows с компьютера, присоединенного к домену, требуется использовать записи A.
* Если вы развертываете несколько серверов AD FS или прокси-сервера, убедитесь, что вы настроили подсистему балансировки нагрузки и что записи DNS для имени AD FS (например, sts.contoso.com) указывают на подсистему балансировки нагрузки.
* Чтобы встроенная проверка подлинности Windows работала для браузерных приложений, использующих Internet Explorer в интрасети, убедитесь, что имя AD FS (например, sts.contoso.com) добавляется в зону интрасети в Internet Explorer. Это требование можно контролировать с помощью групповая политика и развертывать на всех компьютерах, присоединенных к домену.

## <a name="azure-ad-connect-supporting-components"></a>Вспомогательные компоненты Azure AD Connect
Azure AD Connect устанавливает следующие компоненты на сервере, где установлена Azure AD Connect. Этот список предназначен для базовой установки Express. Если вы решили использовать другую SQL Server на странице **Установка служб синхронизации** , то SQL Express LocalDB не устанавливается локально.

* Azure AD Connect Health
* Программы командной строки Microsoft SQL Server 2012
* Microsoft SQL Server 2012 Express LocalDB
* Microsoft SQL Server 2012 Native Client
* Распространяемый пакет Microsoft Visual C++ 2013

## <a name="hardware-requirements-for-azure-ad-connect"></a>Требования к оборудованию для Azure AD Connect
В следующей таблице приведены минимальные требования к компьютеру синхронизации Azure AD Connect.

| Количество объектов в Active Directory | ЦП | Память | Размер жесткого диска |
| --- | --- | --- | --- |
| Менее 10 000 |1,6 ГГц |4 ГБ |70 ГБ |
| 10 000–50 000 |1,6 ГГц |4 ГБ |70 ГБ |
| 50 000–100 000 |1,6 ГГц |16 Гб |100 ГБ |
| Для 100 000 или более объектов требуется полная версия SQL Server | | | |
| 100 000–300 000 |1,6 ГГц |32 ГБ |300 ГБ |
| 300 000–600 000 |1,6 ГГц |32 ГБ |450 ГБ |
| Более 600 000 |1,6 ГГц |32 ГБ |500 ГБ |

Ниже приведены минимальные требования к компьютерам, на которых выполняются AD FS или прокси-серверы.

* процессор: двухъядерный с тактовой частотой 1,6 ГГц или выше;
* Память: 2 ГБ или более
* виртуальная машина Azure: конфигурация A2 или выше.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
