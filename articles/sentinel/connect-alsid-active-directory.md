---
title: Подключение Алсид для Active Directory к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель Алсид для Active Directory, чтобы извлечь журналы Алсид в Azure Sentinel. Просмотр данных Алсид в книгах, создание оповещений и улучшение расследования.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/03/2021
ms.author: yelevin
ms.openlocfilehash: 654ecb65068e4321b85594d96e8ca7a7f73cde7e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99566798"
---
# <a name="connect-your-alsid-for-active-directory-ad-to-azure-sentinel"></a>Подключение Алсид для Active Directory (AD) к Azure Sentinel

> [!IMPORTANT]
> Соединитель Алсид для Active Directory в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить решение Алсид для AD к Azure Sentinel. Соединитель данных Алсид for Active Directory позволяет легко подключать Алсид для журналов AD с помощью Sentinel Azure, чтобы можно было просматривать данные в книгах, запрашивать их для создания пользовательских оповещений и внедрять их для улучшения расследования. При интеграции между Алсид для AD и Sentinel-метками Azure используется сервер syslog с установленным агентом Log Analytics. В нем также используется настраиваемое средство синтаксического анализа журналов, основанное на функции Kusto.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должно быть разрешение на запись в рабочей области "Sentinel" Azure.

- Решение Алсид для AD должно быть настроено для экспорта журналов через системный журнал.

## <a name="send-alsid-for-ad-logs-to-azure-sentinel-via-the-syslog-agent"></a>Отправка журналов Алсид для AD в метку Azure с помощью агента системного журнала

Настройте Алсид для AD, чтобы пересылать сообщения syslog в рабочую область Sentinel Azure с помощью агента системного журнала.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите соединитель **алсид для Active Directory (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Следуйте инструкциям на странице **алсид for Active Directory** Connector:

    1. Настройка сервера Syslog

        1. Если у вас его еще нет, создайте сервер системного журнала Linux для Алсид для AD, чтобы отправить журналы. Azure Sentinel поддерживает управляющие программы **rsyslog** и **syslog-ng** . 

        1. Настройте сервер системного журнала для вывода Алсид для журналов AD в отдельном файле.

    1. Настройка Алсид для AD для отправки журналов на сервер syslog

        1. На портале **алсид для AD** перейдите в раздел *System*, *Configuration* , а затем *syslog*. Здесь можно создать новое оповещение syslog для сервера Syslog. Для удаленного сервера используйте IP-адрес компьютера Linux, на котором установлен агент Linux.

        1. Убедитесь, что журналы правильно собраны на сервере в отдельном файле (для этого можно воспользоваться кнопкой " **проверить конфигурацию** " в конфигурации оповещений *syslog* в алсид for AD).

    1. Установка и подключение агента Log Analytics для Linux

        - Выберите виртуальную машину Linux в Azure или компьютер, не относящийся к Azure Linux (физический или виртуальный). Перейдите по ссылкам и инструкциям на экране. Дополнительные сведения см. [в статье Настройка компьютера или устройства Linux](connect-syslog.md#configure-your-linux-machine-or-appliance) .

    1. Настройка журналов, собираемых агентом Log Analytics

        - Выберите средства и серьезность в конфигурации дополнительных параметров рабочей области.

            1. Щелкните ссылку **открыть рабочую область Конфигурация дополнительных параметров >** на странице соединителя.

            1. На экране **Дополнительные параметры** выберите **данные** , а затем — **пользовательские журналы**.

            1. Установите флажок **Применить ниже конфигурацию к моим компьютерам Linux** и нажмите кнопку **Добавить**.

            1. Щелкните **выбрать файл** , чтобы отправить пример алсид для файла журнала AD с компьютера Linux, на котором работает сервер syslog, и нажмите кнопку **Далее**.

            1. Убедитесь, что для **параметра задать разделитель записей** задано значение **Новая строка** , и нажмите кнопку **Далее**.

            1. Выберите **Linux** и введите путь к файлу системного журнала, **+** а затем нажмите и **Далее**.

            1. В поле Имя введите *алсидфорадлог* перед суффиксом _CL, а затем нажмите кнопку **Готово**.
    
После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе **пользовательские журналы** в таблице *AlsidForADLog_CL* .

Этот соединитель данных зависит от средства синтаксического анализа, основанного на функции Kusto, чтобы работать должным образом. Чтобы настроить функцию **afad_parser** Kusto для использования в запросах и книгах, выполните следующие действия.

1. В меню навигации меток Azure выберите **журналы**.

1. Скопируйте следующий запрос и вставьте его в окно запроса.
    ```kusto
    let CodenameTable=datatable(Codename: string, Explanation: string) [
    "test-checker-codename", "This is a test checker",
    "", "Not an alert",
    "C-ADM-ACC-USAGE", "Recent use of the default administrator account",
    "C-UNCONST-DELEG", "Dangerous delegation",
    "C-PASSWORD-DONT-EXPIRE", "Accounts with never expiring passwords",
    "C-USERS-CAN-JOIN-COMPUTERS", "Users allowed to join computers to the domain",
    "C-CLEARTEXT-PASSWORD", "Potential clear-text password",
    "C-PROTECTED-USERS-GROUP-UNUSED", "Protected Users group not used",
    "C-PASSWORD-POLICY", "Weak password policies are applied on users",
    "C-GPO-HARDENING", "Domain without computer-hardening GPOs",
    "C-LAPS-UNSECURE-CONFIG", "Local administrative account management",
    "C-AAD-CONNECT", "Verify permissions related to AAD Connect accounts",
    "C-AAD-SSO-PASSWORD", "Verify AAD SSO account password last change",
    "C-GPO-SD-CONSISTENCY", "Verify sensitive GPO objects and files permissions",
    "C-DSHEURISTICS", "Domain using a dangerous backward-compatibility configuration",
    "C-DOMAIN-FUNCTIONAL-LEVEL", "Domains have an outdated functional level",
    "C-DISABLED-ACCOUNTS-PRIV-GROUPS", "Disabled accounts in privileged groups",
    "C-DCSHADOW", "Rogue domain controllers",
    "C-DC-ACCESS-CONSISTENCY", "Domain controllers managed by illegitimate users",
    "C-DANGEROUS-TRUST-RELATIONSHIP", "Dangerous trust relationship",
    "C-DANGEROUS-SENSITIVE-PRIVILEGES", "Dangerous sensitive privileges",
    "C-DANG-PRIMGROUPID", "User Primary Group ID",
    "C-BAD-PASSWORD-COUNT", "Brute-force attack detection",
    "C-ADMINCOUNT-ACCOUNT-PROPS", "AdminCount attribute set on standard users",
    "C-ACCOUNTS-DANG-SID-HISTORY", "Accounts having a dangerous SID History attribute",
    "C-ABNORMAL-ENTRIES-IN-SCHEMA", "Dangerous rights in AD's schema",
    "C-GPOLICY-DISABLED-UNLINKED", "Unlinked, disabled or orphan GPO",
    "C-KERBEROS-CONFIG-ACCOUNT", "Kerberos configuration on user account",
    "C-KRBTGT-PASSWORD", "KDC password last change",
    "C-LAPS-UNSECURE-CONFIG", "Local administrative account management",
    "C-NATIVE-ADM-GROUP-MEMBERS", "Native administrative group members",
    "C-NETLOGON-SECURITY", "Unsecured configuration of Netlogon protocol",
    "C-OBSOLETE-SYSTEMS", "Computers running an obsolete OS",
    "C-PASSWORD-NOT-REQUIRED", "Account that might have an empty password",
    "C-PKI-WEAK-CRYPTO", "Use of weak cryptography algorithms into Active Directory PKI",
    "C-PRE-WIN2000-ACCESS-MEMBERS", "Accounts using a pre-Windows 2000 compatible access control",
    "C-PRIV-ACCOUNTS-SPN", "Privileged accounts running Kerberos services",
    "C-REVER-PWD-GPO", "Reversible passwords in GPO",
    "C-ROOTOBJECTS-SD-CONSISTENCY", "Root objects permissions allowing DCSync-like attacks",
    "C-SDPROP-CONSISTENCY", "Ensure SDProp consistency",
    "C-SENSITIVE-CERTIFICATES-ON-USER", "Ensure SDProp consistency",
    "C-SLEEPING-ACCOUNTS", "Sleeping accounts",
    "C-USER-PASSWORD", "User account using old password",
    "C-USERS-REVER-PWDS", "Reversible passwords"
    ];
    let Common = AlsidForADLog_CL
    | parse RawData with
                         Time:datetime  " "
                         Host:string  " "
                         Product:string "["
                         PID:int "]: \""
                         MessageType:int "\" \""
                         AlertID:int "\" \""
                         Forest:string "\" \""
                         Domain:string "\" "
                         DistinctPart:string;
    let Deviances = Common
    | where MessageType == 0 | parse DistinctPart with "\""
                         Codename:string "\" \""
                         Severity:string "\" \""
                         ADObject:string "\" \""
                         DevianceID:string "\" \""
                         ProfileID:string "\" \""
                         ReasonCodename:string "\" \""
                         EventID:string "\""
                         Attributes:string "\r\n";
    let Changes = Common
    | where MessageType == 1
    | parse kind=regex DistinctPart with "\""
                         ADObject:string "\" \""
                         EventID:string "\" \""
                         EventType:string "\" "
                         Attributes:string "\r?\n";
    union Changes, Deviances
    | project-away DistinctPart, Product, _ResourceId, _SubscriptionId
    | lookup kind=leftouter CodenameTable on Codename;
    ```

1. Щелкните раскрывающийся список **сохранить** и нажмите кнопку **сохранить**. На панели " **сохранить** "

    1. В поле **имя** введите **afad_parser**.

    1. В разделе **Сохранить как** выберите **функция**.

    1. В разделе **псевдоним функции** введите **afad_parser**.

    1. В разделе **Категория** введите **функции**.

    1. Выберите команду **Сохранить**.

    Активация приложений функций обычно занимает от 10 до 15 минут.

Теперь вы готовы запросить Алсид для данных AD, введя `afad_parser` в верхней строке окна запроса.

Дополнительные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Алсид для AD к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
