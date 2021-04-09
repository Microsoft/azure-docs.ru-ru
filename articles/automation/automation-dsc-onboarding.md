---
title: Подключение службы State Configuration службы автоматизации Azure
description: В этой статье описана настройка компьютеров для управления с помощью службы State Configuration службы автоматизации Azure.
services: automation
ms.service: automation
ms.subservice: dsc
author: mgoedtel
ms.author: magoedte
ms.topic: conceptual
ms.date: 12/10/2019
manager: carmonm
ms.openlocfilehash: c0dc68bd7dacf0cd7f4be9732d45831e2dbb712c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98897009"
---
# <a name="enable-azure-automation-state-configuration"></a>Подключение службы State Configuration службы автоматизации Azure

В этой теме описана настройка компьютеров для управления с помощью службы State Configuration службы автоматизации Azure. Дополнительные сведения см. в [обзоре службы State Configuration службы автоматизации Azure](automation-dsc-overview.md).

## <a name="enable-azure-vms"></a>Включение виртуальных машин Azure

Служба State Configuration службы автоматизации Azure позволяет легко подключать виртуальные машины Azure для управления конфигурацией с помощью портала Azure, шаблонов Azure Resource Manager или PowerShell. В процессе работы расширение DSC регистрирует виртуальную машину в службе State Configuration службы автоматизации Azure, исключая необходимость выполнения администратором удаленного входа на виртуальную машину. Так как расширение Azure выполняется асинхронно, действия по отслеживанию хода выполнения описаны в разделе [Проверка состояния виртуальной машины](#check-status-of-vm-setup).

> [!NOTE]
>При развертывании DSC на узле Linux используется папка **/tmp**. Модули, такие как `nxautomation`, временно загружаются для проверки перед установкой в соответствующих расположениях. Чтобы обеспечить правильную установку модулей, агенту Log Analytics для Linux требуются разрешения на чтение и запись в папке **/tmp**.<br><br>
>Агент Log Analytics для Linux запускается от имени пользователя `omsagent`. Чтобы предоставить разрешение на запись пользователю `omsagent`, выполните команду `setfacl -m u:omsagent:rwx /tmp`.

### <a name="enable-a-vm-using-azure-portal"></a>Подключение виртуальной машины с помощью портала Azure

Чтобы подключить виртуальную машину Azure к State Configuration с помощью [портала Azure](https://portal.azure.com/), выполните следующие действия:

1. Перейдите к учетной записи службы автоматизации Azure, в которой нужно подключить виртуальные машины. 

2. На странице State Configuration на вкладке **Узлы** щелкните **Добавить**.

3. Выберите виртуальную машину.

4. Если на виртуальной машине не установлено требуемое расширение состояний PowerShell и она запущена, щелкните **Подключить**.

5. В разделе **Регистрация** введите [значения Local Configuration Manager для расширения PowerShell DSC](/powershell/scripting/dsc/managing-nodes/metaConfig), необходимые для вашего варианта использования. При необходимости можно ввести конфигурацию узла для назначения виртуальной машине.

![подключение виртуальной машины](./media/automation-dsc-onboarding/DSC_Onboarding_6.png)

### <a name="enable-a-vm-using-azure-resource-manager-templates"></a>Подключение виртуальной машины с использованием шаблонов Azure Resource Manager

Вы можете установить виртуальную машину и подключить ее к State Configuration с помощью шаблонов Azure Resource Manager. Пример шаблона для подключения существующей виртуальной машины к State Configuration см. в статье [Сервер под управлением службы Desired State Configuration](https://azure.microsoft.com/resources/templates/101-automation-configuration/). Если вы управляете масштабируемым набором виртуальных машин, см. пример шаблона в статье [Конфигурация масштабируемого набора виртуальных машин под управлением службы автоматизации Azure](https://azure.microsoft.com/resources/templates/201-vmss-automation-dsc/).

### <a name="enable-machines-using-powershell"></a>Подключение компьютеров с помощью PowerShell

Чтобы подключить виртуальные машины к State Configuration, также можно использовать командлет [Register-AzAutomationDscNode](/powershell/module/az.automation/register-azautomationdscnode) в PowerShell. 

> [!NOTE]
>В настоящее время командлет `Register-AzAutomationDscNode` реализован только для компьютеров под управлением Windows, так как он запускает только расширение Windows.

### <a name="register-vms-across-azure-subscriptions"></a>Регистрация виртуальных машин в подписках Azure

Лучший способ регистрации виртуальных машин из других подписок Azure — использовать расширение DSC в шаблоне развертывания для Azure Resource Manager. Примеры приведены в статье [Расширение Desired State Configuration (DSC) с использованием шаблонов Azure Resource Manager](../virtual-machines/extensions/dsc-template.md).

Сведения о расположении ключа и URL-адреса регистрации для использования в качестве параметров в шаблоне см. в разделе [Безопасное подключение компьютеров с помощью регистрации](#enable-machines-securely-using-registration).

## <a name="enable-physicalvirtual-windows-machines"></a>Подключение физических или виртуальных компьютеров Windows

Вы можете подключить серверы Windows, работающие локально или в других облачных средах (включая экземпляры AWS EC2), к службе State Configuration службы автоматизации Azure. У серверов должен быть [исходящий доступ к Azure](automation-dsc-overview.md#network-planning).

1. Убедитесь, что на компьютерах установлена последняя версия [WMF 5](https://aka.ms/wmf5latest), чтобы их можно было подключить к State Configuration. На компьютере, используемом для подключения, также должна быть установлена платформа WMF 5.
1. Создайте папку, содержащую необходимые метаконфигурации DSC, выполнив инструкции из раздела [Создание метаконфигураций DSC](#generate-dsc-metaconfigurations). 
1. Используйте следующий командлет, чтобы удаленно применить метаконфигурации PowerShell DSC к компьютерам, которые нужно подключить к State Configuration. 

   ```powershell
   Set-DscLocalConfigurationManager -Path C:\Users\joe\Desktop\DscMetaConfigs -ComputerName MyServer1, MyServer2
   ```

1. Если вы не можете применить метаконфигурации PowerShell DSC удаленно, скопируйте папку **metaconfigurations** на компьютеры, которые нужно подключить к State Configuration. Затем добавьте код для вызова [Set-DscLocalConfigurationManager](/powershell/module/psdesiredstateconfiguration/set-dsclocalconfigurationmanager) локально на компьютерах.
1. С помощью портала Azure или командлетов убедитесь, что компьютеры отображаются как узлы State Configuration, зарегистрированные в вашей учетной записи службы автоматизации Azure.

## <a name="enable-physicalvirtual-linux-machines"></a>Подключение компьютеров или виртуальных машин Linux

Вы можете подключить к State Configuration серверы Linux, работающие локально или в других облачных средах. У серверов должен быть [исходящий доступ к Azure](automation-dsc-overview.md#network-planning).

1. Убедитесь, что на компьютерах, которые будут подключены к State Configuration, установлена последняя версия расширения [PowerShell Desired State Configuration для Linux](https://github.com/Microsoft/PowerShell-DSC-for-Linux).
2. Если [значения Local Configuration Manager для расширения PowerShell DSC по умолчанию](/powershell/scripting/dsc/managing-nodes/metaConfig4) соответствуют требуемым, а подключаемые компьютеры должны извлекать данные из службы State Configuration и передавать их в нее, сделайте следующее:

   - На каждом компьютере Linux, который нужно подключить, используйте `Register.py`, чтобы выполнить подключение с использованием значений Local Configuration Manager для расширения PowerShell DSC по умолчанию.

     `/opt/microsoft/dsc/Scripts/Register.py <Automation account registration key> <Automation account registration URL>`

   - Сведения о расположении ключа и URL-адреса регистрации для учетной записи службы автоматизации см. в разделе [Безопасное подключение компьютеров с помощью регистрации](#enable-machines-securely-using-registration).

3. Если значения Local Configuration Manager для расширения PowerShell DSC по умолчанию не соответствуют вашему варианту использования или вы хотите подключить компьютеры, которые передают данные только в службу State Configuration службы автоматизации Azure, выполните шаги 4–7. В противном случае сразу перейдите к шагу 7.

4. Создайте папку, содержащую необходимые метаконфигурации DSC, выполнив инструкции из раздела [Создание метаконфигураций DSC](#generate-dsc-metaconfigurations).

5. Убедитесь, что на компьютере, который используется для подключения к State Configuration, установлена последняя версия [WMF 5](https://aka.ms/wmf5latest).

6. Используйте следующий код, чтобы удаленно применить метаконфигурации PowerShell DSC к компьютерам, которые нужно подключить к State Configuration.

    ```powershell
    $SecurePass = ConvertTo-SecureString -String '<root password>' -AsPlainText -Force
    $Cred = New-Object System.Management.Automation.PSCredential 'root', $SecurePass
    $Opt = New-CimSessionOption -UseSsl -SkipCACheck -SkipCNCheck -SkipRevocationCheck

    # need a CimSession for each Linux machine to onboard
    $Session = New-CimSession -Credential $Cred -ComputerName <your Linux machine> -Port 5986 -Authentication basic -SessionOption $Opt

    Set-DscLocalConfigurationManager -CimSession $Session -Path C:\Users\joe\Desktop\DscMetaConfigs
    ```

7. Если вы не можете применить метаконфигурации PowerShell DSC удаленно, скопируйте на компьютеры Linux метаконфигурации, соответствующие удаленным компьютерам, из папки, которая описана в шаге 4.

8. Добавьте код для вызова `Set-DscLocalConfigurationManager.py` локально на каждом компьютере Linux, чтобы подключить его к State Configuration.

   `/opt/microsoft/dsc/Scripts/SetDscLocalConfigurationManager.py -configurationmof <path to metaconfiguration file>`

9. С помощью портала Azure или командлетов убедитесь, что все компьютеры, которые нужно подключить, теперь отображаются как узлы DSC, зарегистрированные в вашей учетной записи службы автоматизации Azure.

## <a name="generate-dsc-metaconfigurations"></a>Создание метаконфигураций DSC

Чтобы подключить любой компьютер к State Configuration, можно создать [метаконфигурацию DSC](/powershell/scripting/dsc/managing-nodes/metaConfig). Эта конфигурация сообщает агенту DSC о необходимости извлечения данных из службы State Configuration службы автоматизации Azure и (или) передачи данных в нее. Чтобы создать метаконфигурации DSC для службы State Configuration службы автоматизации Azure, вы можете использовать либо конфигурацию PowerShell DSC, либо командлеты PowerShell в службе автоматизации Azure.

> [!NOTE]
> Метаконфигурации DSC содержат секреты, необходимые при подключении компьютера к учетной записи службы автоматизации для управления. Обеспечьте должную защиту создаваемых метаконфигураций или удаляйте их сразу после использования.

Поддержка прокси-сервера для метаконфигураций контролируется [Local Configuration Manager](/powershell/scripting/dsc/managing-nodes/metaconfig) (подсистемой платформы PowerShell DSC для Windows). LCM работает на всех целевых узлах и отвечает за вызов ресурсов конфигураций, включенных в скрипт метаконфигурации DSC. Вы можете включить поддержку прокси-сервера в метаконфигурации, включив в блоках `ConfigurationRepositoryWeb`, `ResourceRepositoryWeb`и `ReportServerWeb` необходимые определения свойств `ProxyURL` и `ProxyCredential`. Пример параметра URL-адреса — `ProxyURL = "http://172.16.3.6:3128";`. Объекту `PSCredential` присвоено свойство `ProxyCredential`, как описано в статье об [управлении учетными данными в службе автоматизации Azure](shared-resources/credentials.md). 

### <a name="generate-dsc-metaconfigurations-using-a-dsc-configuration"></a>Создание метаконфигураций DSC с помощью конфигурации DSC

1. Запустите на компьютере, входящем в локальную среду, VSCode (или другой предпочитаемый редактор) от имени администратора. На этом компьютере должна быть установлена последняя версия [WMF 5](https://aka.ms/wmf5latest) .
1. Локально выполните следующий скрипт. Этот сценарий содержит конфигурацию PowerShell DSC для создания метаконфигураций и команду, запускающую этот процесс.

    > [!NOTE]
    > Имена конфигурации узла State Configuration чувствительны к регистру на портале Azure. Если регистр не соответствует, узел не будет отображаться на вкладке **Узлы**.

   ```powershell
   # The DSC configuration that will generate metaconfigurations
   [DscLocalConfigurationManager()]
   Configuration DscMetaConfigs
   {
        param
        (
            [Parameter(Mandatory=$True)]
            [String]$RegistrationUrl,

            [Parameter(Mandatory=$True)]
            [String]$RegistrationKey,

            [Parameter(Mandatory=$True)]
            [String[]]$ComputerName,

            [Int]$RefreshFrequencyMins = 30,

            [Int]$ConfigurationModeFrequencyMins = 15,

            [String]$ConfigurationMode = 'ApplyAndMonitor',

            [String]$NodeConfigurationName,

            [Boolean]$RebootNodeIfNeeded= $False,

            [String]$ActionAfterReboot = 'ContinueConfiguration',

            [Boolean]$AllowModuleOverwrite = $False,

            [Boolean]$ReportOnly
        )

        if(!$NodeConfigurationName -or $NodeConfigurationName -eq '')
        {
            $ConfigurationNames = $null
        }
        else
        {
            $ConfigurationNames = @($NodeConfigurationName)
        }

        if($ReportOnly)
        {
            $RefreshMode = 'PUSH'
        }
        else
        {
            $RefreshMode = 'PULL'
        }

        Node $ComputerName
        {
            Settings
            {
                RefreshFrequencyMins           = $RefreshFrequencyMins
                RefreshMode                    = $RefreshMode
                ConfigurationMode              = $ConfigurationMode
                AllowModuleOverwrite           = $AllowModuleOverwrite
                RebootNodeIfNeeded             = $RebootNodeIfNeeded
                ActionAfterReboot              = $ActionAfterReboot
                ConfigurationModeFrequencyMins = $ConfigurationModeFrequencyMins
            }

            if(!$ReportOnly)
            {
            ConfigurationRepositoryWeb AzureAutomationStateConfiguration
                {
                    ServerUrl          = $RegistrationUrl
                    RegistrationKey    = $RegistrationKey
                    ConfigurationNames = $ConfigurationNames
                }

                ResourceRepositoryWeb AzureAutomationStateConfiguration
                {
                    ServerUrl       = $RegistrationUrl
                    RegistrationKey = $RegistrationKey
                }
            }

            ReportServerWeb AzureAutomationStateConfiguration
            {
                ServerUrl       = $RegistrationUrl
                RegistrationKey = $RegistrationKey
            }
        }
   }

    # Create the metaconfigurations
    # NOTE: DSC Node Configuration names are case sensitive in the portal.
    # TODO: edit the below as needed for your use case
   $Params = @{
        RegistrationUrl = '<fill me in>';
        RegistrationKey = '<fill me in>';
        ComputerName = @('<some VM to onboard>', '<some other VM to onboard>');
        NodeConfigurationName = 'SimpleConfig.webserver';
        RefreshFrequencyMins = 30;
        ConfigurationModeFrequencyMins = 15;
        RebootNodeIfNeeded = $False;
        AllowModuleOverwrite = $False;
        ConfigurationMode = 'ApplyAndMonitor';
        ActionAfterReboot = 'ContinueConfiguration';
        ReportOnly = $False;  # Set to $True to have machines only report to AA DSC but not pull from it
   }

   # Use PowerShell splatting to pass parameters to the DSC configuration being invoked
   # For more info about splatting, run: Get-Help -Name about_Splatting
   DscMetaConfigs @Params
   ```

1. Введите ключ и URL-адрес регистрации для учетной записи службы автоматизации, а также имена виртуальных машин, которые необходимо подключить. Все остальные параметры являются необязательными. Сведения о расположении ключа и URL-адреса регистрации для учетной записи службы автоматизации см. в разделе [Безопасное подключение компьютеров с помощью регистрации](#enable-machines-securely-using-registration).

1. Если вы хотите, чтобы компьютеры передавали сведения о состоянии DSC в службу State Configuration службы автоматизации Azure, не извлекая конфигурацию или модули PowerShell, присвойте параметру `ReportOnly` значение Тrue.

1. Если параметр `ReportOnly` не задан, компьютеры передают сведения о состоянии DSC в службу State Configuration службы автоматизации Azure и извлекают конфигурацию или модули PowerShell. Задайте соответствующие параметры в блоках `ConfigurationRepositoryWeb`, `ResourceRepositoryWeb` и `ReportServerWeb`.

1. Выполните скрипт. В рабочем каталоге появится папка **DscMetaConfigs**, содержащая метаконфигурации PowerShell DSC для подключаемых компьютеров (с правами администратора).

    ```powershell
    Set-DscLocalConfigurationManager -Path ./DscMetaConfigs
    ```

### <a name="generate-dsc-metaconfigurations-using-azure-automation-cmdlets"></a>Создание метаконфигураций DSC с помощью командлетов службы автоматизации Azure

Если параметры по умолчанию диспетчера LCM для PowerShell DSC соответствуют вашему варианту использования и вы хотите разрешить компьютерам извлекать данные из службы State Configuration службы автоматизации Azure и передавать их в нее, вы можете воспользоваться более простым способом создания необходимых метаконфигураций DSC — с помощью командлетов службы автоматизации Azure.

1. Запустите на компьютере, входящем в локальную среду, консоль PowerShell или VSCode с правами администратора.
2. Подключитесь к Azure Resource Manager с помощью командлета [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount).
3. Из учетной записи службы автоматизации, в которой вы настраиваете узлы, загрузите метаконфигурации PowerShell DSC для подключаемых компьютеров.

   ```powershell
   # Define the parameters for Get-AzAutomationDscOnboardingMetaconfig using PowerShell Splatting
   $Params = @{
       ResourceGroupName = 'ContosoResources'; # The name of the Resource Group that contains your Azure Automation account
       AutomationAccountName = 'ContosoAutomation'; # The name of the Azure Automation account where you want a node on-boarded to
       ComputerName = @('web01', 'web02', 'sql01'); # The names of the computers that the metaconfiguration will be generated for
       OutputFolder = "$env:UserProfile\Desktop\";
   }
   # Use PowerShell splatting to pass parameters to the Azure Automation cmdlet being invoked
   # For more info about splatting, run: Get-Help -Name about_Splatting
   Get-AzAutomationDscOnboardingMetaconfig @Params
   ```

1. В рабочем каталоге появится папка **DscMetaConfigs**, содержащая метаконфигурации PowerShell DSC для подключаемых компьютеров (с правами администратора).

    ```powershell
    Set-DscLocalConfigurationManager -Path $env:UserProfile\Desktop\DscMetaConfigs
    ```

## <a name="enable-machines-securely-using-registration"></a>Безопасное подключение компьютеров с помощью регистрации

Вы можете безопасно подключать компьютеры к учетной записи службы автоматизации Azure с помощью протокола регистрации DSC платформы WMF 5. Этот протокол позволяет узлу DSC проходить аутентификацию на опрашиваемом или опрашивающем сервере PowerShell DSC, включая службу State Configuration службы автоматизации Azure. Узел регистрируется на сервере по URL-адресу регистрации и проходит аутентификацию с помощью ключа регистрации. Во время регистрации узел DSC и опрашиваемый или опрашивающий сервер DSC создают для этого узла уникальный сертификат, который будет использоваться для аутентификации после регистрации на сервере. Этот процесс исключает возможность подмены подключенных узлов друг другом, например, если один из узлов взломан и находится под контролем злоумышленника. После регистрации ключ регистрации не используется повторно для аутентификации и удаляется из узла.

Данные, необходимые для протокола регистрации State Configuration, можно найти в подразделе **Ключи** раздела **Параметры учетной записи** на портале Azure. 

![URL-адрес и ключи службы автоматизации Azure](./media/automation-dsc-onboarding/DSC_Onboarding_4.png)

- URL-адрес регистрации соответствует значению поля "URL-адрес" на странице "Ключи".
- Ключ регистрации — это значение поля **Первичный ключ доступа** или **Вторичный ключ доступа** на странице "Ключи". Можно использовать любой из этих ключей.

Для улучшения безопасности на странице "Ключи" можно в любое время повторно создать первичный и вторичный ключи доступа для учетной записи службы автоматизации. Повторное создание ключа предотвращает использование предыдущих ключей для регистрации последующих узлов.

## <a name="re-register-a-node"></a>Повторная регистрация узла

Возможно, после регистрации компьютера в качестве узла DSC в службе State Configuration службы автоматизации Azure этот узел нужно будет зарегистрировать повторно. Такая необходимость может возникнуть по ряду причин.

- **Обновление сертификата.** Для версий Windows Server до Windows Server 2019 каждый узел автоматически согласовывает уникальный сертификат для аутентификации по истечении одного года. Если срок действия сертификата истекает без продления действия, узел не сможет связаться со службой автоматизации Azure и помечается как `Unresponsive`. В настоящее время протокол регистрации PowerShell DSC не может автоматически обновлять сертификаты при приближении даты окончания срока их действия, поэтому по истечении года необходимо повторно регистрировать узлы. Перед повторной регистрацией убедитесь, что на каждом узле выполняется WMF 5 RTM. 

    Повторная регистрация, выполненная не позднее чем через 90 дней с момента истечения срока действия сертификата или в любой момент после истечения срока действия сертификата, приводит к созданию и использованию нового сертификата. Эта проблема решена в Windows Server 2019 и более поздних версиях.

- **Изменения в значениях LCM для DSC.** Возможно, потребуется изменить значения [LCM для PowerShell DSC](/powershell/scripting/dsc/managing-nodes/metaConfig4), заданные во время начальной регистрации узла, например `ConfigurationMode`. Сейчас эти значения агента DSC можно изменять только с помощью повторной регистрации. Единственным исключением является значение Node Configuration (Конфигурация узла), назначенное узлу. Эту настройку можно изменить в DSC службы автоматизации Azure напрямую.

Вы можете повторно зарегистрировать узел точно таким же образом, как вы регистрировали узел изначально, используя любой из описанных в этом документе методов. Перед повторной регистрацией отменять регистрацию узла в службе State Configuration службы автоматизации Azure не нужно.

## <a name="check-status-of-vm-setup"></a>Проверка состояния виртуальной машины

State Configuration позволяет легко подключать виртуальные машины Azure для управления конфигурацией. Расширение DSC для виртуальных машин Azure используется для регистрации виртуальных машин в службе "Настройка состояния службы автоматизации Azure". Так как с виртуальными машинами Azure расширение Desired State Configuration работает асинхронно, важно отслеживать ход выполнения и устранять неполадки.

> [!NOTE]
> При любом способе подключения виртуальных машин Azure Windows к State Configuration, в котором используется расширение Desired State Configuration виртуальной машины Azure, может потребоваться до часа, пока служба автоматизации Azure не покажет виртуальные машины как зарегистрированные. Эта задержка возникает из-за установки WMF 5 на виртуальной машине с помощью расширения Desired State Configuration виртуальной машины Azure, которое требуется для подключения виртуальных машин к State Configuration.

Чтобы просмотреть состояние расширения Desired State Configuration виртуальной машины Azure:

1. На портале Azure перейдите к подключаемой виртуальной машине.
2. Щелкните **Расширения** в разделе **Параметры**. 
3. Затем выберите **DSC** или **DSCForLinux** в зависимости от операционной системы. 
4. Для получения дополнительных сведений щелкните **Просмотреть подробные сведения о состоянии**.

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы приступить к работе со службой State Configuration службы автоматизации Azure, см. сведения в [этой статье](automation-dsc-getting-started.md).
- Сведения о компилировании конфигураций DSC, которые затем можно назначить целевым узлам, см. в статье [Компилирование конфигураций DSC в службе State Configuration службы автоматизации Azure](automation-dsc-compile.md).
- Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
- Сведения о ценах см. на странице [с расценками для службы State Configuration службы автоматизации Azure](https://azure.microsoft.com/pricing/details/automation/).
- Пример использования службы State Configuration в службе автоматизации Azure в конвейере непрерывного развертывания см. в статье [Настройка непрерывного развертывания с помощью Chocolatey](automation-dsc-cd-chocolatey.md).
- Сведения об устранении неполадок службы State Configuration службы автоматизации Azure см. в [этой статье](./troubleshoot/desired-state-configuration.md).
