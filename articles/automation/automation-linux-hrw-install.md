---
title: Развертывание гибридной рабочей роли Runbook в службе автоматизации Linux
description: В этой статье рассказывается, как установить гибридную рабочую роль Runbook службы автоматизации Azure для запуска модулей Runbook на компьютерах под управлением Linux в локальном центре обработки данных или в облачной среде.
services: automation
ms.subservice: process-automation
ms.date: 02/26/2021
ms.topic: conceptual
ms.openlocfilehash: d4d9bcd16e36e76808f19f7fbd43dd0d3e7550c3
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102182338"
---
# <a name="deploy-a-linux-hybrid-runbook-worker"></a>Развертывание гибридной рабочей роли Runbook для Linux

Вы можете использовать функцию гибридной рабочей роли Runbook пользователя в службе автоматизации Azure для запуска модулей Runbook непосредственно на компьютере с Azure или без Azure, включая серверы, зарегистрированные на [серверах с поддержкой Arc Azure](../azure-arc/servers/overview.md). С компьютера или сервера, на котором размещается роль, можно запускать модули Runbook напрямую и для ресурсов в среде, чтобы управлять этими локальными ресурсами.

Гибридная рабочая роль Runbook Linux запускает модули runbook от имени особого пользователя, для которого можно повысить разрешения, чтобы выполнить команды, требующие повышения разрешений. Служба автоматизации Azure хранит и управляет модулями Runbook, а затем доставляет их на один или несколько определенных компьютеров. В этой статье описывается, как установить гибридную рабочую роль Runbook на компьютере Linux, как удалить отдельную рабочую роль и их группу.

После успешного развертывания рабочей роли Runbook ознакомьтесь с [запуском модулей runbook в гибридной рабочей роли Runbook](automation-hrw-run-runbooks.md), чтобы узнать, как настроить модули runbook для автоматизации процессов в локальном центре обработки данных или другой облачной среде.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем начать, убедитесь, что у вас есть следующее.

### <a name="a-log-analytics-workspace"></a>Рабочая область Log Analytics .

Гибридная Рабочая роль Runbook зависит от Azure Monitor Log Analytics рабочей области для установки и настройки роли. Его можно создать с помощью [Azure Resource Manager](../azure-monitor/logs/resource-manager-workspace.md#create-a-log-analytics-workspace), с помощью [PowerShell](../azure-monitor/logs/powershell-sample-create-workspace.md?toc=/powershell/module/toc.json)или в [портал Azure](../azure-monitor/logs/quick-create-workspace.md).

Если у вас нет рабочей области Log Analytics Azure Monitor, ознакомьтесь с [руководством по проектированию журналов Azure Monitor](../azure-monitor/logs/design-logs-deployment.md) перед созданием рабочей области.

### <a name="log-analytics-agent"></a>Агент Log Analytics

Для работы гибридной рабочей роли Runbook требуется [агент log Analytics](../azure-monitor/agents/log-analytics-agent.md) для поддерживаемой операционной системы Linux. Для серверов или компьютеров, размещенных за пределами Azure, можно установить агент Log Analytics с помощью [серверов с поддержкой Arc Azure](../azure-arc/servers/overview.md).

>[!NOTE]
>После установки агента Log Analytics для Linux не следует изменять разрешения `sudoers.d` папки или ее владельца. Для учетной записи **нксаутоматион** требуется разрешение sudo, то есть пользовательский контекст, в котором выполняется Гибридная Рабочая роль Runbook. Не следует удалять разрешения. Это может привести к критическим изменениям в определенных папках или командах.
>

### <a name="supported-linux-operating-systems"></a>Поддерживаемые операционные системы Linux

Гибридная функция Runbook Worker поддерживает следующие дистрибутивы. Предполагается, что все операционные системы имеют версию x64. Архитектура x86 не поддерживается для любой операционной системы.

* Amazon Linux 2012,09 – 2015,09
* CentOS Linux 5, 6, 7 и 8
* Oracle Linux 5, 6 и 7;
* Red Hat Enterprise Linux Server 5, 6, 7 и 8
* Debian GNU/Linux 6, 7 и 8;
* Ubuntu 12,04 LTS, 14,04 LTS, 16,04 LTS и 18,04 LTS
* SUSE Linux Enterprise Server 12 и 15 (SUSE не выпускает версии 13 или 14)

> [!IMPORTANT]
> Перед включением функции Управление обновлениями, которая зависит от роли Гибридная Рабочая роль Runbook системы, подтвердите дистрибутивы, которые [она поддерживает.](update-management/overview.md#supported-operating-systems)

### <a name="minimum-requirements"></a>Минимальные требования

Ниже приведены минимальные требования к системе гибридной рабочей роли Runbook системы и пользователя Linux.

* два ядра;
* 4 ГБ ОЗУ;
* Порт 443 (исходящий).

| **Требуемый пакет** | **Описание** | **Минимальная версия**|
|--------------------- | --------------------- | -------------------|
|Glibc |Библиотека C GNU| 2.5-12 |
|Openssl| Библиотеки OpenSSL | 1.0 (поддерживается TLS 1.1 и TLS 1.2)|
|Curl | Веб-клиент cURL | 7.15.5|
|Python-ctypes | Требуется Python 2. x или Python 3. x |
|PAM | Подключаемые модули аутентификации|
| **Дополнительный пакет** | **Описание** | **Минимальная версия**|
| PowerShell Core | Для запуска модулей Runbook PowerShell необходимо установить PowerShell Core. Подробные сведения об установке см. в статье [Установка PowerShell Core в Linux](/powershell/scripting/install/installing-powershell-core-on-linux). | 6.0.0 |

### <a name="adding-a-machine-to-a-hybrid-runbook-worker-group"></a>Добавление компьютера в группу гибридных рабочих ролей Runbook

Рабочий компьютер можно добавить в группу гибридных рабочих ролей Runbook в одной из учетных записей службы автоматизации. Для компьютеров, на которых размещена система гибридной рабочей роли Runbook, управляемая Управление обновлениями, их можно добавить в группу гибридных рабочих ролей Runbook. Но вы должны использовать одну и ту же учетную запись службы автоматизации как для Управление обновлениями, так и для членства в группе гибридной рабочей роли Runbook.

>[!NOTE]
>Служба автоматизации Azure [Управление обновлениями](./update-management/overview.md) автоматически устанавливает гибридную рабочую роль Runbook системы на компьютере Azure или не на Azure, для которого включена Управление обновлениями. Однако этот рабочий процесс не зарегистрирован ни в одной из гибридных групп рабочих ролей Runbook в учетной записи службы автоматизации. Чтобы запустить модули Runbook на этих компьютерах, необходимо добавить их в группу гибридных рабочих ролей Runbook. Выполните шаг 4 в разделе [Установка гибридной рабочей роли Runbook Linux](#install-a-linux-hybrid-runbook-worker) , чтобы добавить ее в группу.

## <a name="supported-linux-hardening"></a>Поддерживаемые усиление защиты Linux

Следующие еще не поддерживаются:

* СОСТОЯНИИ

## <a name="supported-runbook-types"></a>Поддерживаемые типы runbook

Гибридные рабочие роли Runbook Linux поддерживают ограниченный набор типов Runbook в службе автоматизации Azure, и они описаны в следующей таблице.

|Тип Runbook | Поддерживается |
|-------------|-----------|
|Python 3 (Предварительная версия)|Да, требуется только для этих дистрибутивов: SUSE LES 15, RHEL 8 и CentOS 8|
|Python 2; |Да, для любых дистрибутив, не требующих Python 3<sup>1</sup> |
|PowerShell |Да<sup>2</sup> |
|Рабочий процесс PowerShell |Нет |
|Графический |Нет |
|графический модуль рабочего процесса PowerShell. |Нет |

<sup>1</sup> См. раздел [Поддерживаемые операционные системы Linux](#supported-linux-operating-systems).

<sup>2</sup> Для модулей Runbook PowerShell необходимо установить PowerShell Core на компьютере Linux. Подробные сведения об установке см. в статье [Установка PowerShell Core в Linux](/powershell/scripting/install/installing-powershell-core-on-linux).

### <a name="network-configuration"></a>Сетевая конфигурация

Требования к сети для гибридной рабочей роли Runbook см. в разделе [Настройка сети](automation-hybrid-runbook-worker.md#network-planning).

## <a name="install-a-linux-hybrid-runbook-worker"></a>Установка гибридной рабочей роли Runbook для Linux

Чтобы установить и настроить гибридную рабочую роль Runbook Linux, выполните следующие действия.

1. Включите решение службы автоматизации Azure в рабочей области Log Analytics, выполнив следующую команду в командной строке PowerShell с повышенными привилегиями или в Cloud Shell в [портал Azure](https://portal.azure.com):

    ```powershell
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <resourceGroupName> -WorkspaceName <workspaceName> -IntelligencePackName "AzureAutomation" -Enabled $true
    ```

2. Разверните агент Log Analytics на целевом компьютере.

    * Для виртуальных машин Azure установите агент Log Analytics для Linux с помощью [расширения виртуальной машины для Linux](../virtual-machines/extensions/oms-linux.md). Это расширение устанавливает агент Log Analytics на виртуальных машинах Azure и регистрирует виртуальные машины в существующей рабочей области Log Analytics. Вы можете использовать шаблон Azure Resource Manager, Azure CLI или политику Azure, чтобы назначить встроенную политику [развертывания агента log Analytics для *Linux* или виртуальных машин *Windows*](../governance/policy/samples/built-in-policies.md#monitoring) . После установки агента можно добавить компьютер в группу гибридных рабочих ролей Runbook в учетной записи службы автоматизации.

    * Для компьютеров, не относящихся к Azure, можно установить агент Log Analytics с помощью [серверов с поддержкой Arc Azure](../azure-arc/servers/overview.md). Серверы с поддержкой Arc поддерживают развертывание агента Log Analytics с помощью следующих методов.

        - Использование платформы расширений виртуальных машин.

            Эта функция на серверах с поддержкой Arc Azure позволяет развернуть расширение виртуальной машины агента Log Analytics на сервере Windows и (или) Linux, отличном от Azure. Для управления расширениями виртуальной машины можно использовать следующие методы на гибридных компьютерах или серверах, управляемых с помощью серверов с поддержкой ARC:

            - [портал Azure](../azure-arc/servers/manage-vm-extensions-portal.md).
            - [Интерфейс командной строки Azure](../azure-arc/servers/manage-vm-extensions-cli.md)
            - [Azure PowerShell](../azure-arc/servers/manage-vm-extensions-powershell.md)
            - [Шаблоны диспетчер ресурсов](../azure-arc/servers/manage-vm-extensions-template.md) Azure

        - Использование политики Azure.

            При таком подходе вы используете политику Azure [Deploy log Analytics Agent на компьютерах Linux или Windows Azure Arc](../governance/policy/samples/built-in-policies.md#monitoring) , чтобы выполнять аудит, если на сервере с поддержкой Arc установлен агент log Analytics. Если агент не установлен, он автоматически развертывает его с помощью задачи исправления. Кроме того, если планируется отслеживать компьютеры с Azure Monitor для виртуальных машин, используйте инициативу [Enable Azure Monitor для виртуальных машин](../governance/policy/samples/built-in-initiatives.md#monitoring) для установки и настройки агента log Analytics.

        Мы рекомендуем установить агент Log Analytics для Windows или Linux с помощью политики Azure.

    > [!NOTE]
    > Для управления конфигурацией компьютеров, поддерживающих гибридную рабочую роль Runbook с настройкой требуемого состояния (DSC), необходимо добавить компьютеры в качестве узлов DSC.

    > [!NOTE]
    > Во время установки гибридной рабочей роли Linux должна присутствовать учетная запись [nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) с соответствующими разрешениями для работы команды sudo. Если вы попытаетесь установить рабочую роль, а учетная запись отсутствует или не имеет соответствующих разрешений, установка завершится сбоем.

3. Убедитесь, что агент сообщает о рабочей области.

    Агент Log Analytics для Linux подключает компьютеры к рабочей области Log Analytics Azure Monitor. При установке агента на компьютере и подключении его к рабочей области автоматически загружаются компоненты, необходимые для работы гибридного рабочего процесса Runbook.

    Подключение агента к рабочей области Log Analytics занимает несколько минут, и после этого вы можете выполнить следующий запрос и убедиться, что он отправляет данные пульса в рабочую область.

    ```kusto
    Heartbeat
    | where Category == "Direct Agent"
    | where TimeGenerated > ago(30m)
    ```

    В результатах поиска должны отобразиться записи пульса для компьютера, указывающие, что он подключен и сообщает службе. По умолчанию каждый агент перенаправляет запись пульса в назначенную ему рабочую область.

4. Выполните следующую команду, чтобы добавить компьютер в группу гибридной рабочей роли Runbook, указав значения параметров `-w` ,, `-k` `-g` и `-e` .

    Сведения, необходимые для параметров, можно получить `-k` на `-e` странице **ключи** в учетной записи службы автоматизации. Выберите **ключи** в разделе **Параметры учетной записи** в левой части страницы.

    ![Страница "Управление ключами"](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

    * Для `-e` параметра скопируйте значение для **URL-адреса**.

    * Для `-k` параметра скопируйте значение для **первичного ключа доступа**.

    * В качестве `-g` параметра укажите имя группы гибридных рабочих ролей Runbook, к которой будет присоединена Новая Гибридная Рабочая роль Runbook Linux. Если эта группа уже существует в учетной записи службы автоматизации, в нее добавляется текущий компьютер. Если эта группа не существует, она будет создана с таким именем.

    * В качестве `-w` параметра укажите идентификатор рабочей области log Analytics.

   ```bash
   sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/onboarding.py --register -w <logAnalyticsworkspaceId> -k <automationSharedKey> -g <hybridGroupName> -e <automationEndpoint>
   ```

5. Проверьте развертывание после завершения скрипта. На странице **гибридные группы рабочих ролей Runbook** в учетной записи службы автоматизации на вкладке **Гибридная группа рабочих ролей Runbook пользователя** отображается новая или существующая группа и число членов. Если указать имеющуюся группу, количество элементов в ней увеличивается. Вы можете выбрать группу из списка на странице, в меню слева выберите **гибридные рабочие роли**. На странице **гибридные рабочие роли** можно увидеть каждый член группы в списке.

    > [!NOTE]
    > Если вы используете Log Analytics расширение виртуальной машины для Linux для виртуальной машины Azure, мы рекомендуем установить `autoUpgradeMinorVersion` в `false` качестве версий автоматического обновления, которые могут вызвать проблемы с гибридной рабочей ролью Runbook. Сведения об обновлении расширения вручную см. в разделе [Развертывание с помощью Azure CLI](../virtual-machines/extensions/oms-linux.md#azure-cli-deployment).

## <a name="turn-off-signature-validation"></a>Отключение проверки подписи

По умолчанию для гибридных рабочих ролей Runbook Linux требуется проверка подписи. При запуске неподписанной последовательности runbook для рабочей роли отображается ошибка `Signature validation failed`. Чтобы отключить проверку подписи, выполните следующую команду. Замените второй параметр идентификатором вашей рабочей области Log Analytics.

 ```bash
 sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/require_runbook_signature.py --false <logAnalyticsworkspaceId>
 ```

## <a name="remove-the-hybrid-runbook-worker"></a><a name="remove-linux-hybrid-runbook-worker"></a>Удаление гибридной рабочей роли Runbook

Вы можете использовать команду `ls /var/opt/microsoft/omsagent` в гибридной рабочей роли Runbook, чтобы получить идентификатор рабочей области. При этом создается папка с именем, соответствующим идентификатору рабочей области.

```bash
sudo python onboarding.py --deregister --endpoint="<URL>" --key="<PrimaryAccessKey>" --groupname="Example" --workspaceid="<workspaceId>"
```

> [!NOTE]
> Этот сценарий не удаляет агент Log Analytics для Linux с компьютера. Он удаляет только функциональные возможности и конфигурацию гибридной рабочей роли Runbook.

## <a name="remove-a-hybrid-worker-group"></a>Удаление группы гибридных рабочих ролей

Чтобы удалить группу гибридных рабочих ролей Runbook на компьютерах Linux, используйте те же действия, что и для группы гибридных рабочих ролей Windows. См. раздел [Удаление группы гибридных рабочих ролей](automation-windows-hrw-install.md#remove-a-hybrid-worker-group).

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы узнать, как настроить модули runbook для автоматизации процессов в локальном центре обработки данных или другой облачной среде, см. статью [Запуск модулей runbook в гибридной рабочей роли Runbook](automation-hrw-run-runbooks.md).

* Дополнительные сведения см. в разделе "Linux" статьи [Устранение неполадок с гибридными рабочими ролями Runbook](troubleshoot/hybrid-runbook-worker.md#linux).