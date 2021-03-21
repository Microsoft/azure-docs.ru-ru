---
title: Развертывание сервера пересылки журналов для подключения данных CEF к Azure Sentinel | Документация Майкрософт
description: Узнайте, как развернуть агент для подключения данных CEF к Azure Sentinel.
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
ms.date: 01/05/2021
ms.author: yelevin
ms.openlocfilehash: a4303f43dffa98f842bd3daf9e3a0cd5214932b1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100585364"
---
# <a name="step-1-deploy-the-log-forwarder"></a>Шаг 1. Развертывание сервера пересылки журналов


На этом шаге вы назначите и настроите компьютер Linux, который будет пересылать журналы из решения безопасности в рабочую область Azure Sentinel. Это может быть физический компьютер или виртуальная машина в локальной среде, ВИРТУАЛЬная машина Azure или ВИРТУАЛЬная машина в другом облаке. Используя указанную ссылку, вы запустите сценарий на указанном компьютере, который выполняет следующие задачи:

- Устанавливает агент Log Analytics для Linux (также известный как агент OMS) и настраивает его для следующих целей:
    - Прослушивание сообщений CEF из встроенной управляющей программы Linux syslog на TCP-порте 25226
    - Безопасная отправка сообщений по протоколу TLS в рабочую область "Sentinel" Azure, где они анализируются и дополнены

- Настраивает встроенную управляющую программу Linux syslog (rsyslog. d/syslog-ng) в следующих целях:
    - Прослушивание сообщений системного журнала в решениях безопасности через TCP-порт 514
    - Пересылка только тех сообщений, которые он идентифицирует как CEF агенту Log Analytics на localhost с помощью TCP-порта 25226
 
## <a name="prerequisites"></a>Предварительные условия

- Необходимо иметь повышенные разрешения (sudo) на назначенном компьютере Linux.

- На компьютере Linux должен быть установлен **python 2,7** или **3** .<br>Используйте `python -version` команду для проверки.

- Перед установкой агента Log Analytics компьютер Linux не должен быть подключен к рабочим областям Azure.

- На компьютере Linux должно быть не менее **4 ядер ЦП и 8 ГБ ОЗУ**.

    > [!NOTE]
    > - Один компьютер сервера пересылки журналов, использующий управляющую программу **rsyslog** , имеет поддерживаемую емкость **до 8500 событий в секунду (EPS)** .

- В этом процессе может потребоваться идентификатор рабочей области и первичный ключ рабочей области. Их можно найти в ресурсе рабочей области в разделе **Управление агентами**.

## <a name="run-the-deployment-script"></a>Выполнение скрипта развертывания
 
1. В меню навигации меток Azure щелкните **соединители данных**. В списке соединителей щелкните плитку **общий формат событий (CEF)** , а затем нажмите кнопку **Открыть соединительную страницу** в правом нижнем углу. 

1. В разделе **1,2 Установка СБОРЩИКА CEF на компьютере Linux** скопируйте ссылку, указанную в разделе **выполните следующий скрипт, чтобы установить и применить сборщик CEF**, или из приведенного ниже текста (применение идентификатора рабочей области и первичного ключа вместо заполнителей):

    ```bash
    sudo wget -O cef_installer.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/cef_installer.py&&sudo python cef_installer.py [WorkspaceID] [Workspace Primary Key]
    ```

1. Во время выполнения скрипта убедитесь, что не получены сообщения об ошибках и предупреждения.
    - Вы можете получить сообщение с запросом на выполнение команды для устранения проблемы с сопоставлением поля *компьютер* . Дополнительные сведения см. в описании [в скрипте развертывания](#mapping-command) .

1. Перейдите к [шагу 2. Настройка решения безопасности для пересылки сообщений CEF](connect-cef-solution-config.md).


> [!NOTE]
> **Использование одного компьютера для пересылки обычных системных syslog *и* сообщений CEF**
>
> Если вы планируете использовать этот компьютер сервера пересылки журналов для пересылки [сообщений Syslog](connect-syslog.md) , а также CEF, то во избежание дублирования событий в таблицы syslog и CommonSecurityLog:
>
> 1. На каждом исходном компьютере, который отправляет журналы серверу пересылки в формате CEF, необходимо изменить файл конфигурации syslog, чтобы удалить средства, которые используются для отправки сообщений CEF. Таким образом, средства, отправляемые в CEF, не будут отправляться в syslog. Подробные инструкции по выполнению этой задачи см. в статье [Настройка системного журнала в агенте Linux](../azure-monitor/agents/data-sources-syslog.md#configure-syslog-on-linux-agent) .
>
> 1. На этих компьютерах необходимо выполнить следующую команду, чтобы отключить синхронизацию агента с конфигурацией syslog в Azure Sentinel. Это гарантирует, что изменение конфигурации, сделанное на предыдущем шаге, не перезаписывается.<br>
> `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable'`

> [!NOTE]
> **Изменение источника поля TimeGenerated**
>
> - По умолчанию агент Log Analytics заполняет поле *timegenerated* в схеме временем получения агентом события из управляющей программы syslog. В результате время, когда событие было создано в исходной системе, не записывается в Azure Sentinel.
>
> - Тем не менее, можно выполнить следующую команду, которая загрузит и запустит `TimeGenerated.py` скрипт. Этот сценарий настраивает агент Log Analytics для заполнения поля *timegenerated* исходным временем события в исходной системе, а не временем получения агентом.
>
>    ```bash
>    wget -O TimeGenerated.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/TimeGenerated.py && python TimeGenerated.py {ws_id}
>    ```

## <a name="deployment-script-explained"></a>Описание скрипта развертывания

Ниже приведено описание действий сценария развертывания с помощью команды.

Выберите управляющую программу системного журнала, чтобы просмотреть соответствующее описание.

# <a name="rsyslog-daemon"></a>[Управляющая программа rsyslog](#tab/rsyslog)

1. **Загрузка и установка агента Log Analytics:**

    - Скачивает скрипт установки для агента Log Analytics (OMS) Linux.

        ```bash
        wget -O onboard_agent.sh https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/
            master/installer/scripts/onboard_agent.sh
        ```

    - Устанавливает агент Log Analytics.
    
        ```bash
        sh onboard_agent.sh -w [workspaceID] -s [Primary Key] -d opinsights.azure.com
        ```

1. **Настройка конфигурации агента Log Analytics для прослушивания порта 25226 и пересылка сообщений CEF в Azure Sentinel:**

    - Скачивает конфигурацию из репозитория GitHub агента Log Analytics.

        ```bash
        wget -O /etc/opt/microsoft/omsagent/[workspaceID]/conf/omsagent.d/security_events.conf
            https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/installer/conf/
            omsagent.d/security_events.conf
        ```

1. **Настройка управляющей программы syslog:**

    - Открывает порт 514 для связи TCP с помощью файла конфигурации системного журнала `/etc/rsyslog.conf` .

    - Настраивает управляющую программу на пересылку сообщений CEF агенту Log Analytics через TCP-порт 25226, вставляя специальный файл конфигурации `security-config-omsagent.conf` в каталог управляющей программы системного журнала `/etc/rsyslog.d/` .

        Содержимое `security-config-omsagent.conf` файла:

        ```bash
        if $rawmsg contains "CEF:" or $rawmsg contains "ASA-" then @@127.0.0.1:25226 
        ```

1. **Перезапуск управляющей программы системного журнала и агента Log Analytics:**

    - Перезапускает управляющую программу rsyslog.
    
        ```bash
        service rsyslog restart
        ```

    - Перезапускает агент Log Analytics.

        ```bash
        /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. **Проверка соответствия поля *компьютера* ожидаемым образом:**

    - Проверяет, что поле " *компьютер* " в источнике системного журнала правильно сопоставлено в агенте log Analytics, используя следующую команду: 

        ```bash
        grep -i "'Host' => record\['host'\]"  /opt/microsoft/omsagent/plugin/filter_syslog_security.rb
        ```
    - <a name="mapping-command"></a>При возникновении проблемы с сопоставлением сценарий выдаст сообщение об ошибке, в котором можно **выполнить следующую команду вручную** (вместо ЗАПОЛНИТЕЛЯ используется идентификатор рабочей области). Команда обеспечит правильное сопоставление и перезапустите агент.
    
        ```bash
        sed -i -e "/'Severity' => tags\[tags.size - 1\]/ a \ \t 'Host' => record['host']" -e "s/'Severity' => tags\[tags.size - 1\]/&,/" /opt/microsoft/omsagent/plugin/filter_syslog_security.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

# <a name="syslog-ng-daemon"></a>[Демон syslog-ng](#tab/syslogng)

1. **Загрузка и установка агента Log Analytics:**

    - Скачивает скрипт установки для агента Log Analytics (OMS) Linux.

        ```bash
        wget -O onboard_agent.sh https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/
            master/installer/scripts/onboard_agent.sh
        ```

    - Устанавливает агент Log Analytics.
    
        ```bash
        sh onboard_agent.sh -w [workspaceID] -s [Primary Key] -d opinsights.azure.com
        ```

1. **Настройка конфигурации агента Log Analytics для прослушивания порта 25226 и пересылка сообщений CEF в Azure Sentinel:**

    - Скачивает конфигурацию из репозитория GitHub агента Log Analytics.

        ```bash
        wget -O /etc/opt/microsoft/omsagent/[workspaceID]/conf/omsagent.d/security_events.conf
            https://raw.githubusercontent.com/microsoft/OMS-Agent-for-Linux/master/installer/conf/
            omsagent.d/security_events.conf
        ```

1. **Настройка управляющей программы syslog:**

    - Открывает порт 514 для связи TCP с помощью файла конфигурации системного журнала `/etc/syslog-ng/syslog-ng.conf` .

    - Настраивает управляющую программу на пересылку сообщений CEF агенту Log Analytics через TCP-порт 25226, вставляя специальный файл конфигурации `security-config-omsagent.conf` в каталог управляющей программы системного журнала `/etc/syslog-ng/conf.d/` .

        Содержимое `security-config-omsagent.conf` файла:

        ```bash
        filter f_oms_filter {match(\"CEF\|ASA\" ) ;};destination oms_destination {tcp(\"127.0.0.1\" port(25226));};
        log {source(s_src);filter(f_oms_filter);destination(oms_destination);};
        ```

1. **Перезапуск управляющей программы системного журнала и агента Log Analytics:**

    - Перезапускает управляющую программу syslog-ng.
    
        ```bash
        service syslog-ng restart
        ```

    - Перезапускает агент Log Analytics.

        ```bash
        /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```

1. **Проверка соответствия поля *компьютера* ожидаемым образом:**

    - Проверяет, что поле " *компьютер* " в источнике системного журнала правильно сопоставлено в агенте log Analytics, используя следующую команду: 

        ```bash
        grep -i "'Host' => record\['host'\]"  /opt/microsoft/omsagent/plugin/filter_syslog_security.rb
        ```
    - <a name="mapping-command"></a>При возникновении проблемы с сопоставлением сценарий выдаст сообщение об ошибке, в котором можно **выполнить следующую команду вручную** (вместо ЗАПОЛНИТЕЛЯ используется идентификатор рабочей области). Команда обеспечит правильное сопоставление и перезапустите агент.
    
        ```bash
        sed -i -e "/'Severity' => tags\[tags.size - 1\]/ a \ \t 'Host' => record['host']" -e "s/'Severity' => tags\[tags.size - 1\]/&,/" /opt/microsoft/omsagent/plugin/filter_syslog_security.rb && sudo /opt/microsoft/omsagent/bin/service_control restart [workspaceID]
        ```
---

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как развернуть агент Log Analytics, чтобы подключить устройства CEF к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](./tutorial-detect-threats-built-in.md).
