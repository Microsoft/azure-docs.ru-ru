---
title: Компилирование конфигураций DSC в службе настройки состояния службы автоматизации Azure
description: В этой статье описывается, как компилировать конфигурации службы настройки требуемого состояния (DSC) для службы автоматизации Azure.
services: automation
ms.subservice: dsc
ms.date: 04/06/2020
ms.topic: conceptual
ms.openlocfilehash: b45aad71e04418c7c7dda4fc3f0c84a5fe99ecdf
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98896348"
---
# <a name="compile-dsc-configurations-in-azure-automation-state-configuration"></a>Компилирование конфигураций DSC в службе настройки состояния службы автоматизации Azure

Вы можете компилировать конфигурации службы настройки требуемого состояния (DSC) следующими способами с помощью службы настройки состояния службы автоматизации Azure.

- С помощью службы компиляции Azure State Configuration
  - Самый простой способ с интерактивным пользовательским интерфейсом
   - Легко отслеживаемое состояние задания.

- Windows PowerShell
  - Вызов из Windows PowerShell на локальной рабочей станции или в службе сборки
  - Интеграция с конвейером тестирования разработки
  - Предоставление значений сложных параметров
  - Работа с данными узла и с данными, отличными от данных узла, в большом масштабе
  - Значительное повышение производительности

Вы также можете использовать шаблоны Azure Resource Manager с расширением DSC Azure для принудительной отправки конфигураций на виртуальные машины Azure. Расширение DSC Azure использует платформу агента Azure, чтобы доставлять и применять конфигурации DSC виртуальных машин Azure, а также сообщать об этих конфигурациях. Сведения о компиляции с помощью шаблонов Azure Resource Manager см. в разделе [Расширение Desired State Configuration (DSC) с использованием шаблонов Azure Resource Manager](../virtual-machines/extensions/dsc-template.md#details). 

## <a name="compile-a-dsc-configuration-in-azure-state-configuration"></a>Компиляция конфигурации DSC в Azure State Configuration

### <a name="portal"></a>Портал

1. В учетной записи службы автоматизации нажмите кнопку **Настройка состояния (DSC)** .
1. Нажмите кнопку **Конфигурации**, а затем щелкните имя конфигурации для компилирования.
1. Нажмите кнопку **Компилировать**.
1. Если нет параметров конфигурации, нужно подтвердить ее компиляцию. Если параметры конфигурации есть, отобразится колонка **Компиляция конфигурации**, в которой можно указать значения параметров.
1. Откроется страница "Задание компиляции", на которой можно будет отследить состояние задания компиляции. Эту страницу также можно использовать для отслеживания конфигураций узлов (документы конфигурации MOF), размещенных на опрашиваемом сервере службы настройки состояния службы автоматизации Azure.

### <a name="azure-powershell"></a>Azure PowerShell

Чтобы начать компиляцию с помощью Windows PowerShell, можно использовать [Start-AzAutomationDscCompilationJob](/powershell/module/az.automation/start-azautomationdsccompilationjob). В следующем примере кода начинается компилирование конфигурации DSC под именем **SampleConfig**.

```powershell
Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'
```

`Start-AzAutomationDscCompilationJob` возвращает объект задания компилирования, с помощью которого вы можете отслеживать состояние задания. Затем можно использовать этот объект задания компиляции с [Get-AzAutomationDscCompilationJob](/powershell/module/az.automation/get-azautomationdsccompilationjob), чтобы определить состояние задания компиляции, и [Get-AzAutomationDscCompilationJobOutput](/powershell/module/az.automation/get-azautomationdscconfiguration), чтобы просмотреть его потоки (выходные данные). В следующем примере запускается компилирование конфигурации SampleConfig, ожидается, пока оно завершится, а затем отображаются его потоки.

```powershell
$CompilationJob = Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'

while($null -eq $CompilationJob.EndTime -and $null -eq $CompilationJob.Exception)
{
    $CompilationJob = $CompilationJob | Get-AzAutomationDscCompilationJob
    Start-Sleep -Seconds 3
}

$CompilationJob | Get-AzAutomationDscCompilationJobOutput –Stream Any
```

### <a name="declare-basic-parameters"></a>Объявление базовых параметров

Объявление параметров, в том числе типов и свойств параметров, в конфигурациях DSC выполняется так же, как и в модулях Runbook службы автоматизации Azure. Дополнительные сведения о параметрах модуля Runbook см. в статье [Запуск модуля Runbook в службе автоматизации Azure](./start-runbooks.md).

Чтобы определить значения свойств в конфигурации узла **ParametersExample.sample**, созданной во время компилирования, в следующем примере используются параметры `FeatureName` и `IsPresent`.

```powershell
Configuration ParametersExample
{
    param(
        [Parameter(Mandatory=$true)]
        [string] $FeatureName,

        [Parameter(Mandatory=$true)]
        [boolean] $IsPresent
    )

    $EnsureString = 'Present'
    if($IsPresent -eq $false)
    {
        $EnsureString = 'Absent'
    }

    Node 'sample'
    {
        WindowsFeature ($FeatureName + 'Feature')
        {
            Ensure = $EnsureString
            Name   = $FeatureName
        }
    }
}
```

Вы можете компилировать конфигурации DSC, которые используют базовые параметры, на портале службы настройки состояния службы автоматизации Azure или с помощью Azure PowerShell.

#### <a name="portal"></a>Портал

Чтобы ввести значения параметров на портале, нажмите кнопку **Компилировать**.

![Параметры компиляции конфигурации](./media/automation-dsc-compile/DSC_compiling_1.png)

#### <a name="azure-powershell"></a>Azure PowerShell

Для модуля PowerShell нужны параметры в таблице [hashtable](/powershell/module/microsoft.powershell.core/about/about_hash_tables), в которой раздел соответствует имени параметра, а значение — значению параметра.

```powershell
$Parameters = @{
    'FeatureName' = 'Web-Server'
    'IsPresent' = $False
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ParametersExample' -Parameters $Parameters
```

Сведения о передаче объектов `PSCredential` в качестве параметров см. в разделе [Активы учетных данных](#credential-assets).

### <a name="compile-configurations-containing-composite-resources-in-azure-automation"></a>Компиляция конфигураций, содержащих составные ресурсы, в службе автоматизации Azure

Функция **Составные ресурсы** позволяют использовать конфигурации DSC в качестве вложенных ресурсов в рамках конфигурации. Эта функция обеспечивает применение нескольких конфигураций к одному ресурсу. См. статью [Composite resources: использование конфигурации DSC как ресурса](/powershell/scripting/dsc/resources/authoringresourcecomposite), чтобы узнать подробнее о составных ресурсах.

> [!NOTE]
> Для правильной компиляции конфигураций, содержащих составные ресурсы, необходимо сначала импортировать в службу автоматизации Azure все ресурсы DSC, от которых зависят такие составные ресурсы. Добавление составного ресурса DSC не отличается от добавления модуля PowerShell в службу автоматизации Azure. Этот процесс описан в статье [Администрирование модулей в службе автоматизации Azure](./shared-resources/modules.md).

### <a name="manage-configurationdata-when-compiling-configurations-in-azure-automation"></a>Управление ConfigurationData при компиляции конфигураций в службе автоматизации Azure

Встроенный параметр DSC `ConfigurationData` позволяет при использовании PowerShell DSC отделить конфигурацию структуры от любой конфигурации среды. Дополнительные сведения о см. в публикации блога [Разница между "что" и "где" в DSC PowerShell](https://devblogs.microsoft.com/powershell/separating-what-from-where-in-powershell-dsc/).

> [!NOTE]
> При компилировании в службе настройки состояния службы автоматизации Azure можно использовать `ConfigurationData` в Azure PowerShell, однако не следует использовать этот параметр на портале Azure.

В следующем примере конфигурации DSC используется `ConfigurationData` посредством ключевых слов `$ConfigurationData` и `$AllNodes`. Для этого примера также нужен [модуль xWebAdministration](https://www.powershellgallery.com/packages/xWebAdministration/).

```powershell
Configuration ConfigurationDataSample
{
    Import-DscResource -ModuleName xWebAdministration -Name MSFT_xWebsite

    Write-Verbose $ConfigurationData.NonNodeData.SomeMessage

    Node $AllNodes.Where{$_.Role -eq 'WebServer'}.NodeName
    {
        xWebsite Site
        {
            Name         = $Node.SiteName
            PhysicalPath = $Node.SiteContents
            Ensure       = 'Present'
        }
    }
}
```

Вы можете компилировать конфигурацию DSC, показанную выше, с помощью Windows PowerShell. С помощью следующего сценария в опрашиваемую службу настройки состояния службы автоматизации Azure добавляются две конфигурации узла: **ConfigurationDataSample.MyVM1** и **ConfigurationDataSample.MyVM3**.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = 'MyVM1'
            Role = 'WebServer'
        },
        @{
            NodeName = 'MyVM2'
            Role = 'SQLServer'
        },
        @{
            NodeName = 'MyVM3'
            Role = 'WebServer'
        }
    )

    NonNodeData = @{
        SomeMessage = 'I love Azure Automation State Configuration and DSC!'
    }
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ConfigurationDataSample' -ConfigurationData $ConfigData
```

### <a name="work-with-assets-in-azure-automation-during-compilation"></a>Работа с активами в службе автоматизации Azure во время компиляции

Ссылки на ресурсы одинаковы в конфигурациях и модулях Runbook службы настройки состояния службы автоматизации Azure. Дополнительные сведения см. в следующих разделах:

- [Сертификаты](./shared-resources/certificates.md)
- [Соединения](automation-connections.md)
- [Учетные данные](./shared-resources/credentials.md)
- [Переменные](./shared-resources/variables.md)

#### <a name="credential-assets"></a>Активы учетных данных

Конфигурации DSC в службе автоматизации Azure могут ссылаться на ресурсы учетных данных службы автоматизации с помощью командлета `Get-AutomationPSCredential`. Если конфигурация включает параметр, задающий объект `PSCredential`, используйте `Get-AutomationPSCredential`, передав имя строки ресурса учетных данных службы автоматизации Azure в командлет для получения учетных данных. Затем используйте этот объект для параметра, которому требуется объект `PSCredential`. Названный этим именем актив учетных данных, которые используются для службы автоматизации Azure, будет в фоновом режиме извлечен и передан в конфигурацию. Этот сценарий показан в приведенном ниже примере.

Чтобы обеспечить безопасность учетных данных в конфигурациях узла (документы конфигурации MOF), учетные данные нужно зашифровать в MOF-файле конфигурации узла. В настоящее время необходимо предоставить DSC PowerShell разрешение на вывод учетных данных в виде обычного текста во время создания MOF конфигурации узла. DSC PowerShell не известно, что служба автоматизации Azure шифрует весь MOF-файл после его создания с помощью задания компиляции.

Вы можете подтвердить в модуле DSC PowerShell, что учетные данные можно отобразить в формате обычного текста в MOF-файлах конфигурации узлов, использующих ConfigurationData. `PSDscAllowPlainTextPassword = $true` следует передать через `ConfigurationData` для каждого имени блока узла, которое отображается в конфигурации DSC и для которого нужны учетные данные.

В следующем примере показана конфигурация DSC, использующая актив учетных данных автоматизации.

```powershell
Configuration CredentialSample
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    $Cred = Get-AutomationPSCredential 'SomeCredentialAsset'

    Node $AllNodes.NodeName
    {
        File ExampleFile
        {
            SourcePath      = '\\Server\share\path\file.ext'
            DestinationPath = 'C:\destinationPath'
            Credential      = $Cred
        }
    }
}
```

Вы можете компилировать конфигурацию DSC, показанную выше, с помощью PowerShell. С помощь следующего кода PowerShell на опрашиваемый сервер настройки состояния службы автоматизации Azure добавляются две конфигурации узла: **CredentialSample.MyVM1** и **CredentialSample.MyVM2**.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = '*'
            PSDscAllowPlainTextPassword = $True
        },
        @{
            NodeName = 'MyVM1'
        },
        @{
            NodeName = 'MyVM2'
        }
    )
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'CredentialSample' -ConfigurationData $ConfigData
```

> [!NOTE]
> После завершения компиляции может появиться сообщение об ошибке `The 'Microsoft.PowerShell.Management' module was not imported because the 'Microsoft.PowerShell.Management' snap-in was already imported.`. Это сообщение можно проигнорировать.

## <a name="compile-your-dsc-configuration-in-windows-powershell"></a>Компилирование конфигурации DSC с помощью Windows PowerShell

Процесс компиляции конфигураций DSC в Windows PowerShell включен в документацию по DSC PowerShell [Запись, компиляция и применение конфигурации](/powershell/scripting/dsc/configurations/write-compile-apply-configuration#compile-the-configuration).
Этот процесс можно выполнить на рабочей станции разработчика или в службе сборки, например в [Azure DevOps](https://dev.azure.com). Затем можно импортировать MOF-файлы, созданные путем компиляции конфигурации, в службу настройки состояния Azure.

Компиляция в Windows PowerShell также предоставляет возможность подписывать содержимое конфигурации. Агент DSC проверяет конфигурацию подписанного узла локально на управляемом узле. Проверка гарантирует, что конфигурация, примененная к узлу, получена из полномочного источника.

Вы также можете импортировать конфигурации узла (MOF-файлы), скомпилированные вне Azure. Импорт включает компиляцию на рабочей станции разработчика или в службе, например в [Azure DevOps](https://dev.azure.com). Такой подход имеет несколько преимуществ, включая производительность и надежность.

> [!NOTE]
> Размер файла конфигурации узла не должен превышать 1 МБ, чтобы его можно было импортировать в службу автоматизации Azure.

Дополнительные сведения о том, как подписывать конфигурации узла, см. в разделе [Улучшения в WMF 5.1: подпись конфигурации и модуля](/powershell/scripting/wmf/whats-new/dsc-improvements#dsc-module-and-configuration-signing-validations).

### <a name="import-a-node-configuration-in-the-azure-portal"></a>Импорт конфигурации узла на портале Azure

1. В учетной записи службы автоматизации в разделе **Управление конфигурацией** щелкните **Настройка состояния (DSC)** .
1. На странице "Настройка состояния (DSC)" выберите вкладку **Конфигурации**, а затем щелкните **Добавить**.
1. На странице "Импорт" щелкните значок папки рядом с текстовым полем **Файл конфигурации узла**, чтобы найти файл конфигурации узла (MOF) на локальном компьютере.

   ![Поиск локального файла](./media/automation-dsc-compile/import-browse.png)

1. В текстовом поле **Имя конфигурации** введите имя. Это имя должно совпадать с именем конфигурации, из которой была скомпилирована данная конфигурация узла.
1. Нажмите кнопку **ОК**.

### <a name="import-a-node-configuration-with-azure-powershell"></a>Импорт конфигурации узла с помощью Azure PowerShell

Для импорта конфигурации узла в учетную запись службы автоматизации можно использовать командлет [Import-AzAutomationDscNodeConfiguration](/powershell/module/az.automation/import-azautomationdscnodeconfiguration).

```powershell
Import-AzAutomationDscNodeConfiguration -AutomationAccountName 'MyAutomationAccount' -ResourceGroupName 'MyResourceGroup' -ConfigurationName 'MyNodeConfiguration' -Path 'C:\MyConfigurations\TestVM1.mof'
```

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы приступить к работе со службой State Configuration службы автоматизации Azure, см. сведения в [этой статье](automation-dsc-getting-started.md).
- Сведения о компилировании конфигураций DSC, которые затем можно назначить целевым узлам, см. в статье [Компилирование конфигураций DSC в службе State Configuration службы автоматизации Azure](automation-dsc-compile.md).
- Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
- Сведения о ценах см. на странице [с расценками для службы настройки состояния службы автоматизации Azure](https://azure.microsoft.com/pricing/details/automation/).
- Пример использования службы настройки состояния в конвейере непрерывного развертывания см. в разделе [Настройка непрерывного развертывания с помощью Chocolatey](automation-dsc-cd-chocolatey.md).
