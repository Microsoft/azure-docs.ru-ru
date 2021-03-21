---
title: Управление агентом Log Analytics Azure
description: В этой статье описываются различные задачи управления, которые обычно выполняются в течение жизненного цикла Log Analytics агента Windows или Linux, развернутого на компьютере.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/14/2019
ms.openlocfilehash: ca36c35d859e651c0d949f4b7fbb28137d01af90
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101734967"
---
# <a name="managing-and-maintaining-the-log-analytics-agent-for-windows-and-linux"></a>Обслуживание агента Log Analytics для Windows и Linux и управление им

После первоначального развертывания Log Analytics агента Windows или Linux в Azure Monitor может потребоваться перенастроить агент, обновить его или удалить с компьютера, если он достиг этапа прекращения использования в его жизненном цикле. Вы можете легко управлять этими задачами обслуживания вручную или автоматизировать этот процесс, сократив при этом эксплуатационные расходы и количество ошибок.

## <a name="upgrading-agent"></a>Обновление агента

Агент Log Analytics для Windows и Linux можно обновить до последнего выпуска вручную или автоматически в зависимости от сценария развертывания и среды, в которой выполняется виртуальная машина. Для обновления агента можно использовать следующие методы.

| Среда | Метод установки | Метод перехода |
|--------|----------|-------------|
| Azure | Расширение виртуальной машины агента Log Analytics для Windows и Linux | Агент автоматически обновляется по умолчанию, если не настроен шаблон Azure Resource Manager для отказаться от установки свойства *autoUpgradeMinorVersion* в **значение false**. |
| Пользовательские образы виртуальных машин Azure | Ручная установка агента Log Analytics для Windows и Linux | Обновление виртуальных машин до последней версии агента необходимо выполнить из командной строки, в которой выполняется пакет установщика Windows или самораспаковывающийся и устанавливаемый комплект сценариев оболочки для Linux.|
| Виртуальные машины, не относящиеся к Azure | Ручная установка агента Log Analytics для Windows и Linux | Обновление виртуальных машин до последней версии агента необходимо выполнить из командной строки, в которой выполняется пакет установщика Windows или самораспаковывающийся и устанавливаемый комплект сценариев оболочки для Linux. |

### <a name="upgrade-windows-agent"></a>Обновление агента Windows 

Чтобы обновить агент на виртуальной машине Windows до последней версии, не установленной с помощью расширения Log Analytics VM, запустите из командной строки, скрипта или другого решения по автоматизации или с помощью \<platform\> мастера установки ммасетуп-. msi.  

Последнюю версию агента Windows можно скачать из рабочей области Log Analytics, выполнив следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).

2. На портале Azure щелкните **Все службы**. В списке ресурсов введите **Log Analytics**. Как только вы начнете вводить символы, список отфильтруется соответствующим образом. Выберите **Рабочие области Log Analytics**.

3. В списке рабочих областей Log Analytics выберите рабочую область.

4. В рабочей области Log Analytics выберите **Дополнительные параметры**, затем выберите **подключенные источники** и наконец **серверы Windows**.

5. На странице **серверы Windows** выберите подходящую версию **агента Windows** для скачивания в зависимости от архитектуры процессора операционной системы Windows.

>[!NOTE]
>Во время обновления агента Log Analytics для Windows он не поддерживает настройку или перенастройку рабочей области для передачи отчетов в. Чтобы настроить агент, необходимо выполнить один из поддерживаемых методов, перечисленных в разделе [Добавление или удаление рабочей области](#adding-or-removing-a-workspace).
>

#### <a name="to-upgrade-using-the-setup-wizard"></a>Обновление с помощью мастера установки

1. Войдите в систему компьютера, используя учетную запись с правами администратора.

2. Выполните **ммасетуп- \<platform\> . exe** , чтобы запустить мастер установки.

3. На первой странице мастера установки нажмите кнопку **Далее**.

4. В диалоговом окне **установки Microsoft Monitoring Agent** нажмите кнопку **я принимаю** , чтобы принять условия лицензионного соглашения.

5. В диалоговом окне **Установка Microsoft Monitoring Agent** нажмите кнопку **Обновить**. На странице состояния отображается ход обновления.

6. При **успешном завершении настройки Microsoft Monitoring Agent.** появится страница, нажмите кнопку **Готово**.

#### <a name="to-upgrade-from-the-command-line"></a>Обновление с помощью командной строки

1. Войдите в систему компьютера, используя учетную запись с правами администратора.

2. Чтобы извлечь файлы установки агента из командной строки с повышенными привилегиями, запустите файл `MMASetup-<platform>.exe /c`. После этого вы сможете выбрать путь для извлечения файлов. Кроме того, путь можно указать, передав аргументы `MMASetup-<platform>.exe /c /t:<Full Path>`.

3. Выполните следующую команду (D:\ является местонахождением файла журнала обновления).

    ```dos
    setup.exe /qn /l*v D:\logs\AgentUpgrade.log AcceptEndUserLicenseAgreement=1
    ```

### <a name="upgrade-linux-agent"></a>Обновление агента Linux 

Поддерживается обновление с предыдущих версий (>1.0.0-47). При установке с помощью команды `--upgrade` все компоненты агента будут обновлены до последней версии.

Выполните следующую команду, чтобы обновить агент.

`sudo sh ./omsagent-*.universal.x64.sh --upgrade`

## <a name="adding-or-removing-a-workspace"></a>Добавление или удаление рабочей области

### <a name="windows-agent"></a>Агент Windows
Действия, описанные в этом разделе, необходимы, если требуется не только перенастроить агент Windows для передачи отчетов в другую рабочую область или удалить рабочую область из ее конфигурации, но и при необходимости настроить агент для передачи отчетов в несколько рабочих областей (обычно это называется многоадресной адресацией). Настройка агента Windows для создания отчетов в нескольких рабочих областях может быть выполнена только после начальной настройки агента и с помощью описанных ниже методов.    

#### <a name="update-settings-from-control-panel"></a>Обновление параметров на панели управления

1. Войдите в систему компьютера, используя учетную запись с правами администратора.

2. Откройте **Панель управления**.

3. Выберите **Microsoft Monitoring Agent** и перейдите на вкладку **Azure Log Analytics**.

4. Чтобы удалить рабочую область, выберите нужную и нажмите кнопку **Удалить**. Повторите этот шаг для всех рабочих областей, которые этот агент больше не должен отслеживать.

5. Чтобы добавить рабочую область, нажмите кнопку **Добавить**, а затем в диалоговом окне **Add a Log Analytics Workspace** (Добавление рабочей области Log Analytics) введите идентификатор и ключ (первичный) рабочей области. Если компьютер должен передавать данные в рабочую область Log Analytics в облаке Azure для государственных организаций, выберите "Azure для US Gov организаций" из раскрывающегося списка "Облако Azure".

6. Нажмите кнопку **ОК** , чтобы сохранить внесенные изменения.

#### <a name="remove-a-workspace-using-powershell"></a>Удаление рабочей области с помощью PowerShell

```powershell
$workspaceId = "<Your workspace Id>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.RemoveCloudWorkspace($workspaceId)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-commercial-using-powershell"></a>Добавление рабочей области в Azure для коммерческих организаций с помощью PowerShell

```powershell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey)
$mma.ReloadConfiguration()
```

#### <a name="add-a-workspace-in-azure-for-us-government-using-powershell"></a>Добавление рабочей области в Azure для US Gov организаций с помощью PowerShell

```powershell
$workspaceId = "<Your workspace Id>"
$workspaceKey = "<Your workspace Key>"
$mma = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'
$mma.AddCloudWorkspace($workspaceId, $workspaceKey, 1)
$mma.ReloadConfiguration()
```

>[!NOTE]
>Если ранее вы устанавливали или настраивали агент с помощью командной строки или сценария, теперь вместо `EnableAzureOperationalInsights` используется `AddCloudWorkspace` и `RemoveCloudWorkspace`.
>

### <a name="linux-agent"></a>Агент Linux
В следующих шагах показано, как перенастроить агент Linux, если вы решите зарегистрировать его в другой рабочей области или удалить рабочую область из ее конфигурации.

1. Чтобы убедиться, что он зарегистрирован в рабочей области, выполните следующую команду.

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l`

    Результат выполнения команды должен иметь следующий вид.

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

    Важно, чтобы по состоянию было видно, что агент выполняется. В противном случае приведенные ниже шаги по перенастройке агент выполнить не удастся.

2. Если он уже зарегистрирован в рабочей области, удалите ее, выполнив следующую команду. Если агент еще не зарегистрирован, перейдите к следующему шагу.

    `/opt/microsoft/omsagent/bin/omsadmin.sh -X`

3. Чтобы зарегистрировать агент в другой рабочей области, выполните следующую команду:

    `/opt/microsoft/omsagent/bin/omsadmin.sh -w <workspace id> -s <shared key> [-d <top level domain>]`
    
4. Чтобы убедиться, что изменения вступили в действие, выполните следующую команду:

    `/opt/microsoft/omsagent/bin/omsadmin.sh -l`

    Результат выполнения команды должен иметь следующий вид.

    `Primary Workspace: <workspaceId>   Status: Onboarded(OMSAgent Running)`

Службу агента не нужно перезапускать, чтобы изменения вступили в силу.

## <a name="update-proxy-settings"></a>Обновление параметров прокси-сервера
Чтобы после развертывания настроить взаимодействие агента со службой через прокси-сервер или [шлюз Log Analytics](./gateway.md), используйте один из приведенных ниже способов.

### <a name="windows-agent"></a>Агент Windows

#### <a name="update-settings-using-control-panel"></a>Обновление параметров с помощью панели управления

1. Войдите в систему компьютера, используя учетную запись с правами администратора.

2. Откройте **Панель управления**.

3. Выберите **Microsoft Monitoring Agent** и перейдите на вкладку **Параметры прокси-сервера** .

4. Щелкните **Использовать прокси-сервер** и укажите URL-адрес и номер порта прокси-сервера или шлюза. Если для доступа к прокси-серверу или шлюзу Log Analytics требуется аутентификация, введите имя пользователя и пароль, а затем нажмите кнопку **ОК**.

#### <a name="update-settings-using-powershell"></a>Обновление параметров с помощью PowerShell

Скопируйте следующий образец кода PowerShell, измените в нем сведения, относящиеся к используемой среде, и сохраните его в файле с расширением PS1. Запустите сценарий на каждом компьютере, который подключается непосредственно к рабочей области Log Analytics в Azure Monitor.

```powershell
param($ProxyDomainName="https://proxy.contoso.com:30443", $cred=(Get-Credential))

# First we get the Health Service configuration object. We need to determine if we
#have the right update rollup with the API we need. If not, no need to run the rest of the script.
$healthServiceSettings = New-Object -ComObject 'AgentConfigManager.MgmtSvcCfg'

$proxyMethod = $healthServiceSettings | Get-Member -Name 'SetProxyInfo'

if (!$proxyMethod)
{
    Write-Output 'Health Service proxy API not present, will not update settings.'
    return
}

Write-Output "Clearing proxy settings."
$healthServiceSettings.SetProxyInfo('', '', '')

$ProxyUserName = $cred.username

Write-Output "Setting proxy to $ProxyDomainName with proxy username $ProxyUserName."
$healthServiceSettings.SetProxyInfo($ProxyDomainName, $ProxyUserName, $cred.GetNetworkCredential().password)
```

### <a name="linux-agent"></a>Агент Linux
Если компьютерам Linux нужно взаимодействовать через прокси-сервер или шлюз Log Analytics, выполните приведенные ниже действия. Значение конфигурации прокси-сервера имеет следующий синтаксис: `[protocol://][user:password@]proxyhost[:port]`. Свойство *proxyhost* принимает полное доменное имя или IP-адрес прокси-сервера.

1. Измените файл `/etc/opt/microsoft/omsagent/proxy.conf`, выполнив следующие команды и указав собственные значения параметров.

    ```
    proxyconf="https://proxyuser:proxypassword@proxyserver01:30443"
    sudo echo $proxyconf >>/etc/opt/microsoft/omsagent/proxy.conf
    sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/proxy.conf
    ```

2. Перезапустите агент, выполнив следующую команду.

    ```
    sudo /opt/microsoft/omsagent/bin/service_control restart [<workspace id>]
    ```

## <a name="uninstall-agent"></a>Удаление агента
Используйте одну из следующих процедур для удаления агента Windows или Linux с помощью командной строки или мастера установки.

### <a name="windows-agent"></a>Агент Windows

#### <a name="uninstall-from-control-panel"></a>Удаление из панели управления
1. Войдите в систему компьютера, используя учетную запись с правами администратора.

2. На **панели управления** щелкните элемент **Программы и компоненты**.

3. В окне **Программы и компоненты** выберите **Microsoft Monitoring Agent**, щелкните **Удалить**, а затем нажмите кнопку **Да**.

>[!NOTE]
>Мастер настройки агента можно также запустить, дважды щелкнув файл **MMASetup-\<platform\>.exe**, который можно скачать из рабочей области на портале Azure.

#### <a name="uninstall-from-the-command-line"></a>Удаление из командной строки
Скачанный файл агента — это автономный пакет установки, созданный IExpress. В нем содержатся программа установки агента и вспомогательные файлы. Чтобы удалить агент с помощью командной строки, содержимое этого пакета нужно извлечь. Ниже описано, как это сделать.

1. Войдите в систему компьютера, используя учетную запись с правами администратора.

2. Чтобы извлечь файлы установки агента из командной строки с повышенными привилегиями, запустите файл `extract MMASetup-<platform>.exe`. После этого вы сможете выбрать путь для извлечения файлов. Кроме того, путь можно указать, передав аргументы `extract MMASetup-<platform>.exe /c:<Path> /t:<Path>`. Дополнительные сведения о параметрах командной строки, поддерживаемых IExpress, см. в [этой статье](https://www.betaarchive.com/wiki/index.php?title=Microsoft_KB_Archive/197147). Затем обновите пример в соответствии со своими потребностями.

3. В командной строке введите следующую команду: `%WinDir%\System32\msiexec.exe /x <Path>:\MOMAgent.msi /qb`.

### <a name="linux-agent"></a>Агент Linux
Чтобы удалить агент, выполните следующую команду на компьютере Linux. Аргумент *--purge* полностью удаляет агент и его конфигурацию.

   `wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh --purge`

## <a name="configure-agent-to-report-to-an-operations-manager-management-group"></a>Настройка передачи агентом отчетов в группу управления Operations Manager

### <a name="windows-agent"></a>Агент Windows
Чтобы в агенте Log Analytics для Windows настроить передачу отчетов в группу управления System Center Operations Manager, выполните приведенные ниже действия.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

1. Войдите в систему компьютера, используя учетную запись с правами администратора.

2. Откройте **Панель управления**.

3. Щелкните **Microsoft Monitoring Agent** и перейдите на вкладку **Operations Manager**.

4. Если серверы Operations Manager интегрированы с Active Directory, установите флажок **Автоматически обновлять назначения групп управления из AD DS**.

5. Нажмите кнопку **Добавить**, чтобы открыть диалоговое окно **Добавление группы управления**.

6. В поле **Имя группы управления** введите имя группы управления.

7. В поле **Основной сервер управления** введите имя компьютера основного сервера управления.

8. В поле **Порт сервера управления** введите номер порта TCP.

9. На странице **Учетная запись действия агента** выберите учетную запись Local System или учетную запись локального домена.

10. Нажмите кнопку **ОК**, чтобы закрыть диалоговое окно **Добавление группы управления**, и еще раз нажмите **ОК**, чтобы закрыть диалоговое окно **Свойства Microsoft Monitoring Agent**.

### <a name="linux-agent"></a>Агент Linux
Чтобы в агенте Log Analytics для Linux настроить передачу отчетов в группу управления System Center Operations Manager, выполните приведенные ниже действия.

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

1. Измените файл `/etc/opt/omi/conf/omiserver.conf`

2. Укажите для строки `httpsport=` номер порта 1270. Например: `httpsport=1270`.

3. Перезапустите сервер OMI: `sudo /opt/omi/bin/service_control restart`.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь [с разрешениями по устранению неполадок агента Linux в](agent-linux-troubleshoot.md) случае возникновения проблем при установке агента Linux или управлении им.

- Ознакомьтесь [с разрешениями по устранению неполадок агента Windows в](agent-windows-troubleshoot.md) случае возникновения проблем при установке агента Windows или управлении им.