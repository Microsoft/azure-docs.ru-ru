---
title: Создание подключений Active Directory для Azure NetApp Files и управление ими | Документация Майкрософт
description: В этой статье показано, как создавать подключения Active Directory для Azure NetApp Files и управлять ими.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/24/2021
ms.author: b-juche
ms.openlocfilehash: d238b566c1286b9b765fb574cd72ee68ccf4b4a7
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105048380"
---
# <a name="create-and-manage-active-directory-connections-for-azure-netapp-files"></a>Создание Active Directory подключений и управление ими для Azure NetApp Files

Для некоторых функций Azure NetApp Files требуется подключение Active Directory.  Например, необходимо установить подключение Active Directory, прежде чем можно будет создать [том SMB](azure-netapp-files-create-volumes-smb.md) или [том с двумя протоколами](create-volumes-dual-protocol.md).  В этой статье показано, как создавать подключения Active Directory для Azure NetApp Files и управлять ими.

## <a name="before-you-begin"></a>Перед началом  

Перед началом необходимо настроить пул емкости.   
[Настройка пула емкости](azure-netapp-files-set-up-capacity-pool.md)   
Подсеть должна быть делегирована службе Azure NetApp Files.  
[Делегирование подсети службе Azure NetApp Files](azure-netapp-files-delegate-subnet.md)

## <a name="requirements-for-active-directory-connections"></a>Требования к подключениям Active Directory

 Ниже перечислены требования к подключениям Active Directory. 

* Используемая учетная запись администратора должна иметь возможность создавать учетные записи компьютеров в указанном пути подразделения (OU).  

* На соответствующем сервере Windows Active Directory (AD) должны быть открыты правильные порты.  
    Ниже перечислены необходимые порты. 

    |     Служба           |     Порт     |     Протокол     |
    |-----------------------|--------------|------------------|
    |    Веб-службы AD    |    9389      |    TCP           |
    |    DNS                |    53        |    TCP           |
    |    DNS                |    53        |    UDP           |
    |    ICMPv4             |    Недоступно       |    Ответ на запрос проверки связи    |
    |    Kerberos           |    464       |    TCP           |
    |    Kerberos           |    464       |    UDP           |
    |    Kerberos           |    88        |    TCP           |
    |    Kerberos           |    88        |    UDP           |
    |    LDAP               |    389       |    TCP           |
    |    LDAP               |    389       |    UDP           |
    |    LDAP               |    3268      |    TCP           |
    |    NetBIOS-имя       |    138       |    UDP           |
    |    SAM/LSA            |    445       |    TCP           |
    |    SAM/LSA            |    445       |    UDP           |
    |    w32time            |    123       |    UDP           |

* Топология сайта для целевых доменных служб Active Directory должна соответствовать рекомендациям, в частности в виртуальной сети Azure, в которой развернута Azure NetApp Files.  

    Адресное пространство виртуальной сети, в которой развертывается Azure NetApp Files, должно быть добавлено на новый или существующий сайт Active Directory (где размещен контроллер домена, доступный Azure NetApp Files). 

* Указанные DNS-серверы должны быть доступны из [делегированной подсети](./azure-netapp-files-delegate-subnet.md) Azure NetApp Files.  

    См. [Рекомендации по планированию сети Azure NetApp Files](./azure-netapp-files-network-topologies.md), чтобы ознакомиться с поддерживаемыми сетевыми топологиями.

    Группы безопасности сети (NSG) и брандмауэры должны иметь соответствующим образом настроенные правила, чтобы разрешить запросы к Active Directory и запросы трафика DNS. 

* Делегированная подсеть Azure NetApp Files должна иметь доступ ко всем контроллерам домена доменных служб Active Directory служб (ADDS) в домене, включая все локальные и удаленные контроллеры домена. В противном случае работа службы может быть прервана.  

    Если у вас есть контроллеры домена, недоступные делегированной подсети Azure NetApp Files, можно указать сайт Active Directory во время создания подключения Active Directory.  Служба Azure NetApp Files должна взаимодействовать только с контроллерами домена на сайте, на которых находится адресное пространство делегированной подсети Azure NetApp Files.

    Сведения сайтах и службах Active Directory см. в разделе [Проектирование топологии сайта](/windows-server/identity/ad-ds/plan/designing-the-site-topology). 
    
* Вы можете включить шифрование AES для проверки подлинности AD, установив флажок **шифрование AES** в окне [Присоединение Active Directory](#create-an-active-directory-connection) . Azure NetApp Files поддерживает типы шифрования DES, Kerberos AES 128 и Kerberos AES 256 (от минимального уровня безопасности до наиболее безопасного). При включении шифрования AES учетные данные пользователя, используемые для приподключения Active Directory, должны иметь включенный параметр с наивысшей соответствующей учетной записью, соответствующий возможностям, включенным для Active Directory.    

    Например, если Active Directory имеет только возможность AES-128, необходимо включить параметр учетной записи AES-128 для учетных данных пользователя. Если Active Directory имеет возможность AES-256, необходимо включить параметр учетной записи AES-256 (который также поддерживает AES-128). Если у Active Directory нет функции шифрования Kerberos, Azure NetApp Files по умолчанию использует DES.  

    Параметры учетной записи можно включить в свойствах Active Directory "пользователи и компьютеры" консоли управления (MMC):   

    ![Active Directory MMC "пользователи и компьютеры"](../media/azure-netapp-files/ad-users-computers-mmc.png)

* Azure NetApp Files поддерживает [Подписывание LDAP](/troubleshoot/windows-server/identity/enable-ldap-signing-in-windows-server), что обеспечивает безопасную передачу трафика LDAP между службой Azure NetApp Files и целевыми [контроллерами домена Active Directory](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview). Если вы используете рекомендации Microsoft консультационных [ADV190023](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190023) для подписывания LDAP, то следует включить функцию подписывания ldap в Azure NetApp Files, установив флажок **Подписывание LDAP** в окне [присоединиться Active Directory](#create-an-active-directory-connection) . 

    Только конфигурация [привязки канала LDAP](https://support.microsoft.com/help/4034879/how-to-add-the-ldapenforcechannelbinding-registry-entry) не влияет на службу Azure NetApp Files. Однако если вы используете как привязку каналов LDAP, так и защищенный протокол LDAP (например, LDAPs или `start_tls` ), то создание тома SMB завершится ошибкой.

* Для службы DNS, не являющейся интегрированной с AD, следует добавить запись DNS A/PTR, чтобы Azure NetApp Files можно было использовать "понятное имя". 

## <a name="decide-which-domain-services-to-use"></a>Выбор доменных служб для использования 

Для подключений AD Azure NetApp Files поддерживает и [доменные службы Active Directory](/windows-server/identity/ad-ds/plan/understanding-active-directory-site-topology) (ADDS), и доменные службы Azure Active Directory (AADDS).  Перед созданием подключения к Active Directory необходимо решить, следует ли использовать ADDS или AADDS.  

Дополнительные сведения см. в статье [Сравнение самостоятельно управляемых доменных служб Active Directory, Azure Active Directory и управляемых доменных служб Azure Active Directory](../active-directory-domain-services/compare-identity-solutions.md). 

### <a name="active-directory-domain-services"></a>Доменные службы Active Directory

Для Azure NetApp Files можно использовать предпочтительную область [сайтов и служб Active Directory](/windows-server/identity/ad-ds/plan/understanding-active-directory-site-topology). Этот параметр позволяет выполнять операции чтения и записи в контроллеры домена доменных служб Active Directory Services (ADDS), которые [доступны службе Azure NetApp Files](azure-netapp-files-network-topologies.md). Это также предотвращает взаимодействие службы с контроллерами домена, которые не находятся на указанном сайте модуля "Active Directory — сайты и службы". 

Чтобы найти имя сайта при использовании ADDS, можно обратиться к администраторам организации, которые отвечают за работу доменных служб Active Directory. В приведенном ниже примере показан подключаемый модуль "Active Directory — сайты и службы", в котором отображается имя сайта. 

![Active Directory — сайты и службы](../media/azure-netapp-files/azure-netapp-files-active-directory-sites-services.png)

При настройке подключения AD для Azure NetApp Files необходимо указать имя сайта в области в поле **Имя сайта AD**.

### <a name="azure-active-directory-domain-services"></a>Доменные службы Azure Active Directory 

Сведения о конфигурации доменных служб Azure Active Directory (AADDS) и соответствующие инструкции см. в статье [Документация по доменным службам Azure AD](../active-directory-domain-services/index.yml).

Дополнительные моменты относительно AADDS, которые следует учитывать при использовании Azure NetApp Files: 

* Убедитесь, что виртуальная сеть или подсеть, в которой развернуты AADDS, находится в том же регионе Azure, что и развертывание Azure NetApp Files.
* Если вы используете другую виртуальную сеть в регионе, где развернуты Azure NetApp Files, следует создать пиринг между двумя виртуальными сетями.
* Azure NetApp Files поддерживает типы `user` и `resource forest`.
* Для типа синхронизации можно выбрать `All` или `Scoped`.   
    При выборе `Scoped` убедитесь, что выбрана правильная группа Azure AD для доступа к общим папкам SMB.  Если вы не уверены, можно использовать тип синхронизации `All`.
* Необходимо использовать номер SKU "Корпоративный" или "Премиум". Номер SKU "Стандартный" не поддерживается.

При создании подключения Active Directory обратите внимание на следующие особенности для AADDS.

* В меню AADDS представлены сведения об **основном DNS**, **дополнительном DNS** и **имени DNS-домена AD**.  
Для DNS-серверов будут использоваться два IP-адреса для настройки подключения Active Directory. 
* **Путь подразделения** — `OU=AADDC Computers`.  
Этот параметр настраивается в подразделе **Подключения Active Directory** раздела **Учетная запись NetApp**.

  ![Путь подразделения](../media/azure-netapp-files/azure-netapp-files-org-unit-path.png)

* В поле **Имя пользователя** можно указать любого пользователя, который является членом группы Azure AD **Администраторы Azure AD DC**.


## <a name="create-an-active-directory-connection"></a>Создание подключения Active Directory

1. В учетной записи NetApp нажмите **Присоединение каталога Active Directory**, а затем нажмите **Присоединиться**.  

    ![Подключения Active Directory](../media/azure-netapp-files/azure-netapp-files-active-directory-connections.png)

2. В окне "Присоединение каталога Active Directory" укажите следующие сведения в зависимости от доменных служб, которые вы хотите использовать.  

    Сведения, относящиеся к используемым доменным службам, см. в разделе [Выбор доменных служб для использования](#decide-which-domain-services-to-use). 

    * **Основной DNS**  
        Требуется для присоединения к домену Active Directory и прохождения проверки подлинности SMB. 
    * **Дополнительный DNS**   
        Это дополнительный DNS-сервер для обеспечения избыточности служб имен. 
    * **Имя DNS-домена AD**  
        Это доменное имя доменных служб Active Directory, которые необходимо присоединить.
    * **Имя сайта AD**  
        Это имя сайта, которое будет ограничено для обнаружения контроллера домена. Оно должно соответствовать имени сайта в Active Directory сайтах и службах.
    * **Префикс SMB-сервера (учетная запись компьютера)**  
        Это префикс именования для учетной записи компьютера в Active Directory, который Azure NetApp Files будет использовать для создания новых учетных записей.

        Например, если стандарт именования, используемый организацией для файловых серверов, — NAS-01, NAS-02..., NAS-045, то для префикса нужно ввести "NAS". 

        Служба будет создавать дополнительные учетные записи компьютеров в Active Directory по мере необходимости.

        > [!IMPORTANT] 
        > Переименование префикса сервера SMB после создания подключения Active Directory нарушается. После переименования префикса сервера SMB потребуется повторно подключить существующие общие ресурсы SMB.

    * **Путь подразделения**  
        Это путь LDAP к подразделению, в котором будут созданы учетные записи компьютера SMB-сервера (OU = SecondLevel, OU = FirstLevel). 

        При использовании Azure NetApp Files с доменными службами Azure Active Directory путь подразделения при настройке Active Directory для учетной записи NetApp будет `OU=AADDC Computers`.

        ![Присоединение каталога Active Directory](../media/azure-netapp-files/azure-netapp-files-join-active-directory.png)

    * **Шифрование AES**   
        Установите этот флажок, если требуется включить шифрование AES для тома SMB. Требования см. [в разделе Требования для Active Directory подключений](#requirements-for-active-directory-connections) . 

        ![Active Directory шифрование AES](../media/azure-netapp-files/active-directory-aes-encryption.png)

        Функция **шифрования AES** в настоящее время доступна в предварительной версии. Если вы впервые используете эту функцию, зарегистрируйте эту функцию, прежде чем использовать ее: 

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFAesEncryption
        ```

        Проверьте состояние регистрации компонента: 

        > [!NOTE]
        > **RegistrationState** может находиться в состоянии до `Registering` 60 минут до перехода на `Registered` . Дождитесь состояния, `Registered` прежде чем продолжить.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFAesEncryption
        ```
        
        Вы также можете использовать [Azure CLI команды](/cli/azure/feature) `az feature register` , `az feature show` чтобы зарегистрировать эту функцию и отобразить состояние регистрации. 

    * **Подписывание LDAP**   
        Установите этот флажок, чтобы включить подписывание LDAP. Эта функция обеспечивает безопасный поиск LDAP между службой Azure NetApp Files и указанными пользователем [контроллерами домена служб домен Active Directory Services](/windows/win32/ad/active-directory-domain-services). Дополнительные сведения см. в разделе [ADV190023 | Руководство Майкрософт по включению привязки каналов LDAP и подписывания LDAP](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190023).  

        ![Active Directory подписывания LDAP](../media/azure-netapp-files/active-directory-ldap-signing.png) 

        Функция **подписывания LDAP** сейчас доступна в предварительной версии. Если вы впервые используете эту функцию, зарегистрируйте эту функцию, прежде чем использовать ее: 

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapSigning
        ```

        Проверьте состояние регистрации компонента: 

        > [!NOTE]
        > **RegistrationState** может находиться в состоянии до `Registering` 60 минут до перехода на `Registered` . Дождитесь состояния, `Registered` прежде чем продолжить.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFLdapSigning
        ```
        
        Вы также можете использовать [Azure CLI команды](/cli/azure/feature) `az feature register` , `az feature show` чтобы зарегистрировать эту функцию и отобразить состояние регистрации. 

     * **Пользователи с правами доступа**   <!-- SMB CA share feature -->   
        Вы можете предоставить привилегии безопасности ( `SeSecurityPrivilege` ) пользователям, которым требуются повышенные привилегии для доступа к Azure NetApp Filesным томам. Указанные учетные записи пользователей могут выполнять определенные действия на Azure NetApp Files общих ресурсах SMB, которым по умолчанию требуются права безопасности, не назначенные пользователям домена.   

        Например, учетные записи пользователей, используемые для установки SQL Server в определенных сценариях, должны обладать повышенными привилегиями безопасности. Если для установки SQL Server используется учетная запись без прав администратора (домена), а учетная запись не имеет назначенного права доступа, следует добавить права доступа к учетной записи.  

        > [!IMPORTANT]
        > Учетная запись домена, используемая для установки SQL Server, должна уже существовать, прежде чем ее можно будет добавить в поле " **Пользователи привилегий безопасности** ". При добавлении учетной записи установщика SQL Server к **пользователям с привилегиями безопасности** служба Azure NetApp Files может проверить учетную запись, обратившись к контроллеру домена. Команда может завершиться ошибкой, если не сможет связаться с контроллером домена.  

        Дополнительные сведения о `SeSecurityPrivilege` и SQL Server см. [в статье сбой установки SQL Server, если учетная запись установки не имеет определенных прав пользователя](/troubleshoot/sql/install/installation-fails-if-remove-user-right).

        ![Снимок экрана, показывающий поле "Пользователи привилегий безопасности" в окне Active Directory "подключения".](../media/azure-netapp-files/security-privilege-users.png) 

     * **Пользователи политики резервного копирования**  
        Можно включить дополнительные учетные записи, требующие повышенных прав доступа к учетной записи компьютера, созданной для использования с Azure NetApp Files. Указанным учетным записям можно будет изменять разрешения NTFS на уровне файлов и папок. Например, можно указать непривилегированную учетную запись службы, используемую для переноса данных в общую папку SMB в Azure NetApp Files.  

        ![Пользователи политики резервного копирования Active Directory](../media/azure-netapp-files/active-directory-backup-policy-users.png)

        Функция " **Пользователи политики резервного копирования** " сейчас доступна в предварительной версии. Если вы впервые используете эту функцию, зарегистрируйте эту функцию, прежде чем использовать ее: 

        ```azurepowershell-interactive
        Register-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFBackupOperator
        ```

        Проверьте состояние регистрации компонента: 

        > [!NOTE]
        > **RegistrationState** может находиться в состоянии до `Registering` 60 минут до перехода на `Registered` . Дождитесь состояния, `Registered` прежде чем продолжить.

        ```azurepowershell-interactive
        Get-AzProviderFeature -ProviderNamespace Microsoft.NetApp -FeatureName ANFBackupOperator
        ```
        
        Вы также можете использовать [Azure CLI команды](/cli/azure/feature) `az feature register` , `az feature show` чтобы зарегистрировать эту функцию и отобразить состояние регистрации. 

    * Учетные данные, включая **имя пользователя** и **пароль**.

        ![Учетные данные Active Directory](../media/azure-netapp-files/active-directory-credentials.png)

3. Щелкните **Соединить**.  

    Откроется созданное подключение Active Directory.

    ![Создано Active Directory подключений](../media/azure-netapp-files/azure-netapp-files-active-directory-connections-created.png)

## <a name="next-steps"></a>Дальнейшие действия  

* [Создание тома SMB](azure-netapp-files-create-volumes-smb.md)
* [Создание тома с двумя протоколами](create-volumes-dual-protocol.md)
* [Настройка шифрования Kerberos для NFSv4.1](configure-kerberos-encryption.md)
* [Установка нового леса Active Directory с помощью Azure CLI](/windows-server/identity/ad-ds/deploy/virtual-dc/adds-on-azure-vm) 
