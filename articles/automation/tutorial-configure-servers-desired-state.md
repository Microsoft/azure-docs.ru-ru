---
title: Настройка требуемого состояния компьютеров в службе автоматизации Azure
description: В этой статье объясняется, как настроить требуемое состояние для компьютеров с помощью службы State Configuration в составе службы автоматизации Azure.
services: automation
ms.subservice: dsc
ms.topic: conceptual
ms.date: 08/08/2018
ms.openlocfilehash: f16db3f55ebd0f09e4d7b75750fa319daf03977e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99053573"
---
# <a name="configure-machines-to-a-desired-state"></a>Настройка требуемого состояния компьютеров

Служба "Настройка состояния службы автоматизации Azure" позволяет задавать конфигурации для серверов и гарантировать, что спустя время они будут находиться в указанном состоянии.

> [!div class="checklist"]
> - Подключение виртуальной машины для управления с помощью DSC службы автоматизации Azure.
> - Передача конфигурации в службу автоматизации Azure.
> - Компиляция конфигурации в конфигурации узла
> - Назначение конфигурации узла управляемому узлу.
> - Проверка состояния соответствия управляемого узла

В этом руководстве используется простая [конфигурация DSC](/powershell/scripting/dsc/configurations/configurations), которая гарантирует, что на виртуальной машине установлены службы IIS.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись службы автоматизации Azure. Дополнительные сведения о учетной записи службы автоматизации и ее требованиях см. в разделе [Обзор проверки подлинности учетной записи службы автоматизации](./automation-security-overview.md).
- Виртуальная машина Azure Resource Manager (не классическая) под управлением Windows Server 2008 R2 или более поздней версии. Для ее создания воспользуйтесь статьей [Краткое руководство. Создание виртуальной машины Windows на портале Azure](../virtual-machines/windows/quick-create-portal.md).
- Azure PowerShell 3.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/azurerm/install-azurerm-ps).
- Знакомство с платформой Desired State Configuration (DSC) Дополнительную информацию о DSC см. в статье [Общие сведения о службе настройки требуемого состояния Windows PowerShell](/powershell/scripting/dsc/overview/overview).

## <a name="support-for-partial-configurations"></a>Поддержка частичных конфигураций

State Configuration в службе автоматизации Azure поддерживает использование [частичных конфигураций](/powershell/scripting/dsc/pull-server/partialconfigs). В этом случае DSC настраивается для управления несколькими конфигурациями независимо друг от друга, где каждая конфигурация извлекается из службы автоматизации Azure. Однако для каждого узла можно назначить только одну конфигурацию от одной учетной записи службы автоматизации. Это означает, что для использования двух конфигураций на одном узле вам потребуются две учетные записи службы автоматизации.

Дополнительные сведения о регистрации частичной конфигурации от опрашивающей службы см. в [документации по частичным конфигурациям](/powershell/scripting/dsc/pull-server/partialconfigs#partial-configurations-in-pull-mode).

Дополнительную информацию о совместном управлении серверами с применением конфигурации как кода см. в статье [Общие сведения о роли DSC в конвейере CI/CD](/powershell/scripting/dsc/overview/authoringadvanced).

## <a name="log-in-to-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью командлета [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount) и следуйте инструкциям на экране.

```powershell
Connect-AzAccount
```

## <a name="create-and-upload-a-configuration-to-azure-automation"></a>Создание и передача конфигурации в службе автоматизации Azure

В текстовом редакторе введите следующий код и сохраните его локально в виде файла **TestConfig.ps1**.

```powershell
configuration TestConfig {
   Node WebServer {
      WindowsFeature IIS {
         Ensure               = 'Present'
         Name                 = 'Web-Server'
         IncludeAllSubFeature = $true
      }
   }
}
```

> [!NOTE]
> В более сложных сценариях, где необходимо импортировать несколько модулей, предоставляющих ресурсы DSC, создайте в конфигурации для каждого модуля уникальную строку `Import-DscResource`.

Для отправки конфигурации в учетную запись службы автоматизации выполните командлет [Import-AzAutomationDscConfiguration](/powershell/module/Az.Automation/Import-AzAutomationDscConfiguration).

```powershell
 Import-AzAutomationDscConfiguration -SourcePath 'C:\DscConfigs\TestConfig.ps1' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Published
```

## <a name="compile-a-configuration-into-a-node-configuration"></a>Компиляция конфигурации в конфигурации узла

Прежде чем назначать конфигурацию DSC узлу, ее нужно скомпилировать в конфигурации узла. См. статью [Конфигурации DSC](/powershell/scripting/dsc/configurations/configurations).

Для компиляции конфигурации `TestConfig` в конфигурацию узла `TestConfig.WebServer`, размещенную в учетной записи службы автоматизации, выполните командлет [Start-AzAutomationDscCompilationJob](/powershell/module/Az.Automation/Start-AzAutomationDscCompilationJob).

```powershell
Start-AzAutomationDscCompilationJob -ConfigurationName 'TestConfig' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount'
```

## <a name="register-a-vm-to-be-managed-by-state-configuration"></a>Регистрация виртуальной машины, управляемой с помощью настройки состояния

С помощью службы "Настройка состояния службы автоматизации Azure" можно управлять виртуальными машинами Azure (классическими или Resource Manager), локальными виртуальными машинами, компьютерами Linux, виртуальными машинами AWS и локальными физическими компьютерами. В этом разделе рассматривается регистрация только виртуальных машин Azure Resource Manager. Дополнительные сведения о регистрации компьютеров других типов см. в статье [Подключение компьютеров для управления с помощью Azure Automation DSC](automation-dsc-onboarding.md).

Чтобы зарегистрировать виртуальную машину как управляемый узел в службе State Configuration службы автоматизации Azure, выполните командлет [Register-AzAutomationDscNode](/powershell/module/Az.Automation/Register-AzAutomationDscNode). 

```powershell
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm'
```

### <a name="specify-configuration-mode-settings"></a>Указание параметров режима конфигурации

Чтобы зарегистрировать виртуальную машину как управляемый узел и указать параметры конфигурации, выполните командлет [Register-AzAutomationDscNode](/powershell/module/azurerm.automation/register-azurermautomationdscnode). Например, вы можете указать, что состояние виртуальной машины должно применяться только один раз, назначив значение `ApplyOnly` свойству `ConfigurationMode`. Тогда State Configuration не будет повторно применять конфигурацию после первоначальной проверки.

```powershell
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationMode 'ApplyOnly'
```

Вы также можете указать, как часто DSC нужно проверять состояние конфигурации, с помощью свойства `ConfigurationModeFrequencyMins`. Дополнительные сведения о настройках конфигурации DSC см. в статье [Настройка локального диспетчера конфигураций](/powershell/scripting/dsc/managing-nodes/metaConfig).

```powershell
# Run a DSC check every 60 minutes
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationModeFrequencyMins 60
```

## <a name="assign-a-node-configuration-to-a-managed-node"></a>Назначение конфигурации узла управляемому узлу.

Теперь можно назначить конфигурации скомпилированного узла для виртуальной машины, которую необходимо настроить.

```powershell
# Get the ID of the DSC node
$node = Get-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Assign the node configuration to the DSC node
Set-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -NodeConfigurationName 'TestConfig.WebServer' -NodeId $node.Id
```

При этом конфигурация узла с именем `TestConfig.WebServer` присваивается зарегистрированному узлу DSC `DscVm`. По умолчанию узел DSC проверяется на соответствие конфигурации узла каждые 30 минут. Сведения о том, как изменить интервал проверки соответствия, см. в статье [Настройка локального диспетчера конфигураций](/powershell/scripting/dsc/managing-nodes/metaConfig).

## <a name="check-the-compliance-status-of-a-managed-node"></a>Проверка состояния соответствия управляемого узла

Отчеты о состоянии соответствия управляемого узла можно получить с помощью командлета [Get-AzAutomationDscNodeReport](/powershell/module/Az.Automation/Get-AzAutomationDscNodeReport).

```powershell
# Get the ID of the DSC node
$node = Get-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Get an array of status reports for the DSC node
$reports = Get-AzAutomationDscNodeReport -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -NodeId $node.Id

# Display the most recent report
$reports[0]
```

## <a name="remove-nodes-from-service"></a>Удаление узлов из службы

При добавлении узла в службу State Configuration службы автоматизации Azure в локальном экземпляре Configuration Manager задаются параметры для регистрации в службе и получения конфигураций и требуемых модулей для настройки компьютера.
Если вы решите удалить узел из службы, это можно сделать с помощью портала Azure или командлетов Az.

> [!NOTE]
> Отмена регистрации узла в службе просто изменяет параметры локального экземпляра Configuration Manager так, чтобы узел больше не подключался к службе.
> Это действие никак не влияет на конфигурацию, которая сейчас применена к этому узлу.
> Чтобы удалить текущую конфигурацию, используйте [PowerShell](/powershell/module/psdesiredstateconfiguration/remove-dscconfigurationdocument) или удалите локальный файл конфигурации (для узлов Linux можно использовать только этот способ).

### <a name="azure-portal"></a>Портал Azure

В службе автоматизации Azure в оглавлении щелкните пункт **State configuration (DSC)** .
Затем щелкните **Узлы**, чтобы просмотреть список зарегистрированных в службе узлов.
Щелкните имя узла, который вы хотите удалить.
В открывшемся представлении узла щелкните **Отменить регистрацию**.

### <a name="powershell"></a>PowerShell

Чтобы отменить регистрацию узла в службе State Configuration службы автоматизации Azure с помощью PowerShell, см. документацию по командлету [Unregister-AzAutomationDscNode](/powershell/module/az.automation/unregister-azautomationdscnode).

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы приступить к работе, см. статью [Начало работы со службой State Configuration службы автоматизации Azure](automation-dsc-getting-started.md).
- Сведения о том, как включить узлы, см. в статье [Включение службы State Configuration службы автоматизации Azure](automation-dsc-onboarding.md).
- Сведения о компилировании конфигураций DSC, которые затем можно назначить целевым узлам, см. в статье [Компилирование конфигураций DSC в службе State Configuration службы автоматизации Azure](automation-dsc-compile.md).
- Пример использования службы State Configuration в службе автоматизации Azure в конвейере непрерывного развертывания см. в статье [Настройка непрерывного развертывания с помощью Chocolatey](automation-dsc-cd-chocolatey.md).
- Сведения о ценах см. на странице [с ценами на использование State Configuration службы автоматизации Azure](https://azure.microsoft.com/pricing/details/automation/).
- Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
