---
title: Подключение гибридных компьютеров к Azure с помощью PowerShell
description: Из этой статьи вы узнаете, как установить агент и подключить компьютер к Azure с помощью серверов, поддерживающих дугу Azure. Для этого мы воспользуемся PowerShell.
ms.date: 10/28/2020
ms.topic: conceptual
ms.openlocfilehash: 07a00de9077378ce3e3f7a7578b66e93d1b04f2b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100584935"
---
# <a name="connect-hybrid-machines-to-azure-by-using-powershell"></a>Подключение гибридных компьютеров к Azure с помощью PowerShell

Для серверов, включенных с помощью дуги Azure, можно выполнить действия вручную, чтобы включить их для одной или нескольких компьютеров Windows или Linux в вашей среде. Кроме того, можно использовать командлет PowerShell [Connect-азконнектедмачине](/powershell/module/az.connectedmachine/remove-azconnectedmachine) , чтобы скачать агент подключенного компьютера, установить агент и зарегистрировать машину в службе "Дуга Azure". Командлет загружает пакет агента Windows (установщик Windows) из центра загрузки Майкрософт и пакета агента Linux из репозитория пакетов Майкрософт.

Этот метод требует наличия разрешений администратора на компьютере для установки и настройки агента. В Linux, используя корневую учетную запись, и в Windows вы являетесь членом группы локальных администраторов. Этот процесс можно выполнить в интерактивном или удаленном режиме на сервере Windows Server с помощью [удаленного взаимодействия PowerShell](/powershell/scripting/learn/ps101/08-powershell-remoting).

Прежде чем начать, ознакомьтесь с [предварительными](agent-overview.md#prerequisites) требованиями и убедитесь, что подписка и ресурсы соответствуют требованиям. Сведения о поддерживаемых регионах и других связанных вопросах см. в разделе [Поддерживаемые регионы Azure](overview.md#supported-regions).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

- Компьютер с Azure PowerShell. Инструкции см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/).

PowerShell используется для управления расширениями виртуальной машины на гибридных серверах, управляемых с помощью серверов с поддержкой дуги Azure. Перед использованием PowerShell установите `Az.ConnectedMachine` модуль. Выполните следующую команду на сервере с поддержкой дуги Azure.

```powershell
Install-Module -Name Az.ConnectedMachine
```

После завершения установки появится следующее сообщение:

`The installed extension ``Az.ConnectedMachine`` is experimental and not covered by customer support. Please use with discretion.`

## <a name="install-the-agent-and-connect-to-azure"></a>Установка агента и подключение к Azure

1. Откройте консоль PowerShell с повышенными привилегиями.

2. Войдите в Azure, выполнив команду `Connect-AzAccount` .

3. Чтобы установить агент подключенного компьютера, используйте параметр `Connect-AzConnectedMachine` с `-Name` `-ResourceGroupName` параметрами, и `-Location` . Используйте `-SubscriptionId` параметр, чтобы переопределить подписку по умолчанию в результате создания контекста Azure после входа. Выполните одну из следующих команд:

    * Чтобы установить агент подключенного компьютера на целевом компьютере, который может напрямую взаимодействовать с Azure, выполните:

        ```azurepowershell
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Name myMachineName -Location <region>
        ```
    
    * Чтобы установить агент подключенного компьютера на целевом компьютере, который обменивается данными через прокси-сервер, выполните:
        
        ```azurepowershell
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Name myMachineName -Location <region> -Proxy http://<proxyURL>:<proxyport>
        ```

Если не удается запустить агент после завершения установки, просмотрите подробные сведения об ошибке в журналах. В Windows проверьте этот файл: *%програмдата%\азуреконнектедмачинеажент\лог\химдс.лог*. В Linux Проверьте этот файл: */Вар/ОПТ/азкмажент/лог/химдс.лог*.

## <a name="install-and-connect-by-using-powershell-remoting"></a>Установка и подключение с помощью удаленного взаимодействия PowerShell

Вот как можно настроить один или несколько серверов Windows с включенными серверами в службе "Дуга Azure". Необходимо включить удаленное взаимодействие PowerShell на удаленном компьютере. Для этого используйте командлет `Enable-PSRemoting`.

1. Откройте консоль PowerShell с правами администратора.

2. Войдите в Azure, выполнив команду `Connect-AzAccount` .

3. Чтобы установить агент подключенного компьютера, используйте параметр `Connect-AzConnectedMachine` с `-ResourceGroupName` параметрами и `-Location` . Имена ресурсов Azure будут автоматически использовать имя узла каждого сервера. Используйте `-SubscriptionId` параметр, чтобы переопределить подписку по умолчанию в результате создания контекста Azure после входа.

    * Чтобы установить агент подключенного компьютера на целевом компьютере, который может напрямую взаимодействовать с Azure, выполните следующую команду:
    
        ```azurepowershell
        $sessions = New-PSSession -ComputerName myMachineName
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Location <region> -PSSession $sessions
        ```
    
    * Чтобы установить агент подключенного компьютера одновременно на нескольких удаленных компьютерах, добавьте список имен удаленных компьютеров, разделенных запятыми.

        ```azurepowershell
        $sessions = New-PSSession -ComputerName myMachineName1, myMachineName2, myMachineName3
        Connect-AzConnectedMachine -ResourceGroupName myResourceGroup -Location <region> -PSSession $sessions
        ```

    В следующем примере показаны результаты выполнения команды, предназначенной для одного компьютера:
    
    ```azurepowershell
    time="2020-08-07T13:13:25-07:00" level=info msg="Onboarding Machine. It usually takes a few minutes to complete. Sometimes it may take longer depending on network and server load status."
    time="2020-08-07T13:13:25-07:00" level=info msg="Check network connectivity to all endpoints..."
    time="2020-08-07T13:13:29-07:00" level=info msg="All endpoints are available... continue onboarding"
    time="2020-08-07T13:13:50-07:00" level=info msg="Successfully Onboarded Resource to Azure" VM Id=f65bffc7-4734-483e-b3ca-3164bfa42941
    
    Name           Location OSName   Status     ProvisioningState
    ----           -------- ------   ------     -----------------
    myMachineName  eastus   windows  Connected  Succeeded
    ```

## <a name="verify-the-connection-with-azure-arc"></a>Проверка подключения с помощью Azure Arc

После установки и настройки агента для регистрации на серверах с поддержкой Arc Azure перейдите к портал Azure, чтобы убедиться, что сервер успешно подключен. Просмотрите свои компьютеры на [портале Azure](https://portal.azure.com).

![Снимок экрана панели мониторинга "серверы" с успешным подключением к серверу.](./media/onboard-portal/arc-for-servers-successful-onboard.png)

## <a name="next-steps"></a>Дальнейшие действия

* При необходимости ознакомьтесь с [руководством по устранению неполадок подключенного компьютера](troubleshoot-agent-onboard.md).

* Узнайте, как управлять компьютером с помощью [политики Azure](../../governance/policy/overview.md). Вы можете использовать [гостевую конфигурацию](../../governance/policy/concepts/guest-configuration.md)виртуальной машины, убедиться, что компьютер сообщает о предполагаемой log Analytics рабочей области, и включить мониторинг с помощью [Azure Monitor с виртуальными машинами](../../azure-monitor/vm/vminsights-enable-policy.md).

* Узнайте больше об [агенте Log Analytics](../../azure-monitor/agents/log-analytics-agent.md). Агент Log Analytics для Windows и Linux необходим, если требуется получить данные мониторинга операционной системы и рабочей нагрузки, а также управлять ими с помощью модулей Runbook или таких функций службы автоматизации Azure, как Управление обновлениями. Этот агент также необходим для использования других служб Azure, таких как [Центр безопасности Azure](../../security-center/security-center-introduction.md).
