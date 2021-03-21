---
title: Набор средств тестирования шаблонов ARM
description: Описывается, как запустить набор средств тестирования шаблона Azure Resource Manager (шаблон ARM) в шаблоне. Набор средств позволяет увидеть, реализованы ли рекомендации.
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: e5ad0b6dca7718166517b52148fbc6dd49f38869
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97674018"
---
# <a name="use-arm-template-test-toolkit"></a>Использование набора средств для тестирования шаблонов ARM

[Набор средств тестирования шаблона Azure Resource Manager (шаблон ARM)](https://aka.ms/arm-ttk) проверяет, использует ли шаблон рекомендованные методы. Если шаблон не соответствует рекомендуемым методикам, он возвращает список предупреждений с предлагаемыми изменениями. С помощью набора средств тестирования можно узнать, как избежать распространенных проблем при разработке шаблонов.

Набор тестов набора средств тестирования предоставляет ряд [проверок по умолчанию](test-cases.md). Эти тесты являются рекомендациями, но не требованиями. Вы можете решить, какие тесты соответствуют вашим целям, и настроить, какие тесты выполняются.

В этой статье описывается, как запустить набор средств для тестирования и как добавить или удалить тесты. Описание тестов по умолчанию см. в разделе [тестовые случаи набора инструментов](test-cases.md).

Набор средств — это набор сценариев PowerShell, которые можно запускать из команды в PowerShell или CLI.

## <a name="install-on-windows"></a>Установка в Windows

1. Если у вас еще нет PowerShell, [установите PowerShell в Windows](/powershell/scripting/install/installing-powershell-core-on-windows).

1. [Скачайте последнюю ZIP-файл](https://aka.ms/arm-ttk-latest) для набора средств тестирования и извлеките его.

1. Запустите PowerShell.

1. Перейдите к папке, в которую был извлечен набор средств для тестирования. В этой папке перейдите в папку **ARM-ТТК** .

1. Если [Политика выполнения](/powershell/module/microsoft.powershell.core/about/about_execution_policies) блокирует сценарии из Интернета, необходимо разблокировать файлы скриптов. Убедитесь, что вы находитесь в папке **ARM-ТТК** .

   ```powershell
   Get-ChildItem *.ps1, *.psd1, *.ps1xml, *.psm1 -Recurse | Unblock-File
   ```

1. Импортируйте модуль.

   ```powershell
   Import-Module .\arm-ttk.psd1
   ```

1. Чтобы выполнить тесты, используйте следующую команду:

   ```powershell
   Test-AzTemplate -TemplatePath \path\to\template
   ```

## <a name="install-on-linux"></a>Установка в Linux

1. Если у вас еще нет PowerShell, [установите PowerShell в Linux](/powershell/scripting/install/installing-powershell-core-on-linux).

1. [Скачайте последнюю ZIP-файл](https://aka.ms/arm-ttk-latest) для набора средств тестирования и извлеките его.

1. Запустите PowerShell.

   ```bash
   pwsh
   ```

1. Перейдите к папке, в которую был извлечен набор средств для тестирования. В этой папке перейдите в папку **ARM-ТТК** .

1. Если [Политика выполнения](/powershell/module/microsoft.powershell.core/about/about_execution_policies) блокирует сценарии из Интернета, необходимо разблокировать файлы скриптов. Убедитесь, что вы находитесь в папке **ARM-ТТК** .

   ```powershell
   Get-ChildItem *.ps1, *.psd1, *.ps1xml, *.psm1 -Recurse | Unblock-File
   ```

1. Импортируйте модуль.

   ```powershell
   Import-Module ./arm-ttk.psd1
   ```

1. Чтобы выполнить тесты, используйте следующую команду:

   ```powershell
   Test-AzTemplate -TemplatePath /path/to/template
   ```

## <a name="install-on-macos"></a>Установка в macOS

1. Если у вас еще нет PowerShell, [установите PowerShell на macOS](/powershell/scripting/install/installing-powershell-core-on-macos).

1. Установите `coreutils`:

   ```bash
   brew install coreutils
   ```

1. [Скачайте последнюю ZIP-файл](https://aka.ms/arm-ttk-latest) для набора средств тестирования и извлеките его.

1. Запустите PowerShell.

   ```bash
   pwsh
   ```

1. Перейдите к папке, в которую был извлечен набор средств для тестирования. В этой папке перейдите в папку **ARM-ТТК** .

1. Если [Политика выполнения](/powershell/module/microsoft.powershell.core/about/about_execution_policies) блокирует сценарии из Интернета, необходимо разблокировать файлы скриптов. Убедитесь, что вы находитесь в папке **ARM-ТТК** .

   ```powershell
   Get-ChildItem *.ps1, *.psd1, *.ps1xml, *.psm1 -Recurse | Unblock-File
   ```

1. Импортируйте модуль.

   ```powershell
   Import-Module ./arm-ttk.psd1
   ```

1. Чтобы выполнить тесты, используйте следующую команду:

   ```powershell
   Test-AzTemplate -TemplatePath /path/to/template
   ```

## <a name="result-format"></a>Формат результата

Пройденные тесты отображаются **зеленым цветом** и начинаются с **[+]**.

Тесты, которые не пройдены, отображаются **красным цветом** и начинаются с **[-]**.

:::image type="content" source="./media/template-test-toolkit/view-results.png" alt-text="Просмотр результатов теста":::

Результаты будут:

```powershell
[+] adminUsername Should Not Be A Literal (24 ms)
[+] apiVersions Should Be Recent (18 ms)
[+] artifacts parameter (16 ms)
[+] DeploymentTemplate Schema Is Correct (17 ms)
[+] IDs Should Be Derived From ResourceIDs (15 ms)
[-] Location Should Not Be Hardcoded (41 ms)
     azuredeploy.json must use the location parameter, not resourceGroup().location (except when used as a default value in the main template)
```

## <a name="test-parameters"></a>Параметры тестирования

Когда вы предоставляете параметр **-TemplatePath** , набор средств ищет в этой папке шаблон с именем azuredeploy.json или maintemplate.json. Сначала он проверяет этот шаблон, а затем проверяет все остальные шаблоны в папке и ее вложенных папках. Другие шаблоны тестируются как связанные шаблоны. Если путь содержит файл с именем [CreateUiDefinition.js](../managed-applications/create-uidefinition-overview.md), он выполняет тесты, относящиеся к определению пользовательского интерфейса.

```powershell
Test-AzTemplate -TemplatePath $TemplateFolder
```

Чтобы проверить один файл в этой папке, добавьте параметр **-File** . Однако папка по-прежнему должна иметь основной шаблон с именем azuredeploy.json или maintemplate.js.

```powershell
Test-AzTemplate -TemplatePath $TemplateFolder -File cdn.json
```

По умолчанию выполняются все тесты. Чтобы указать отдельные тесты для выполнения, используйте параметр **-Test** . Укажите имя теста. Имена см. в разделе [тестовые случаи для набора инструментов](test-cases.md).

```powershell
Test-AzTemplate -TemplatePath $TemplateFolder -Test "Resources Should Have Location"
```

## <a name="customize-tests"></a>Настройка тестов

Для шаблонов ARM набор средств выполняет все тесты в папке **\арм-ттк\тесткасес\деплойменттемплате**. Если вы хотите удалить тест без возможности восстановления, удалите этот файл из папки.

Для файлов [CreateUiDefinition](../managed-applications/create-uidefinition-overview.md) все тесты выполняются в папке **\арм-ттк\тесткасес\креатеуидефинитион**.

Чтобы добавить собственный тест, создайте файл с соглашением об именовании: **Your-Custom-Test-Name.test.ps1**.

Тест может получить шаблон в виде параметра объекта или строкового параметра. Как правило, используется один или другой, но можно использовать и то, и другое.

Используйте параметр Object, если необходимо получить раздел шаблона и выполнить итерацию по его свойствам. Тест, использующий параметр объекта, имеет следующий формат:

```powershell
param(
    [Parameter(Mandatory=$true,Position=0)]
    [PSObject]
    $TemplateObject
)

# Implement test logic that evaluates parts of the template.
# Output error with: Write-Error -Message
```

Объект шаблона имеет следующие свойства:

* $schema
* contentVersion
* parameters
* variables
* ресурсов
* outputs

Например, можно получить коллекцию параметров с помощью `$TemplateObject.parameters` .

Используйте параметр String, если необходимо выполнить операцию со строкой для всего шаблона. Тест, использующий строковый параметр, имеет следующий формат:

```powershell
param(
    [Parameter(Mandatory)]
    [string]
    $TemplateText
)

# Implement test logic that performs string operations.
# Output error with: Write-Error -Message
```

Например, можно выполнить регулярное выражение строкового параметра, чтобы узнать, используется ли конкретный синтаксис.

Чтобы узнать больше о реализации теста, просмотрите другие тесты в этой папке.

## <a name="integrate-with-azure-pipelines"></a>Интеграция с Azure Pipelines

Вы можете добавить набор средств тестирования в конвейер Azure. С помощью конвейера тест можно выполнять каждый раз при обновлении шаблона или запускать его в рамках процесса развертывания.

Самый простой способ добавить набор средств тестирования в конвейер — с расширениями сторонних разработчиков. Доступны следующие два расширения:

* [Выполнение тестов ARM ТТК](https://marketplace.visualstudio.com/items?itemName=Sam-Cogan.ARMTTKExtension)
* [Тестер шаблонов ARM](https://marketplace.visualstudio.com/items?itemName=maikvandergaag.maikvandergaag-arm-ttk)

Также можно реализовать собственные задачи. В следующем примере показано, как скачать набор средств тестирования.

```json
{
    "environment": {},
    "enabled": true,
    "continueOnError": false,
    "alwaysRun": false,
    "displayName": "Download TTK",
    "timeoutInMinutes": 0,
    "condition": "succeeded()",
    "task": {
        "id": "e213ff0f-5d5c-4791-802d-52ea3e7be1f1",
        "versionSpec": "2.*",
        "definitionType": "task"
    },
    "inputs": {
        "targetType": "inline",
        "filePath": "",
        "arguments": "",
        "script": "New-Item '$(ttk.folder)' -ItemType Directory\nInvoke-WebRequest -uri '$(ttk.uri)' -OutFile \"$(ttk.folder)/$(ttk.asset.filename)\" -Verbose\nGet-ChildItem '$(ttk.folder)' -Recurse\n\nWrite-Host \"Expanding files...\"\nExpand-Archive -Path '$(ttk.folder)/*.zip' -DestinationPath '$(ttk.folder)' -Verbose\n\nWrite-Host \"Expanded files found:\"\nGet-ChildItem '$(ttk.folder)' -Recurse",
        "errorActionPreference": "stop",
        "failOnStderr": "false",
        "ignoreLASTEXITCODE": "false",
        "pwsh": "true",
        "workingDirectory": ""
    }
}
```

В следующем примере показано, как выполнять тесты.

```json
{
    "environment": {},
    "enabled": true,
    "continueOnError": true,
    "alwaysRun": false,
    "displayName": "Run Best Practices Tests",
    "timeoutInMinutes": 0,
    "condition": "succeeded()",
    "task": {
        "id": "e213ff0f-5d5c-4791-802d-52ea3e7be1f1",
        "versionSpec": "2.*",
        "definitionType": "task"
    },
    "inputs": {
        "targetType": "inline",
        "filePath": "",
        "arguments": "",
        "script": "Import-Module $(ttk.folder)/arm-ttk/arm-ttk.psd1 -Verbose\n$testOutput = @(Test-AzTemplate -TemplatePath \"$(sample.folder)\")\n$testOutput\n\nif ($testOutput | ? {$_.Errors }) {\n   exit 1 \n} else {\n    Write-Host \"##vso[task.setvariable variable=result.best.practice]$true\"\n    exit 0\n} \n",
        "errorActionPreference": "continue",
        "failOnStderr": "true",
        "ignoreLASTEXITCODE": "false",
        "pwsh": "true",
        "workingDirectory": ""
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о тестах по умолчанию см. в разделе [тестовые случаи по умолчанию для набора средств тестирования шаблонов ARM](test-cases.md).
- Сведения о модуле Microsoft Learn, посвященном использованию набора тестов, см. в разделе [Предварительный просмотр изменений и проверка ресурсов Azure с помощью набора средств для тестирования шаблонов ARM](/learn/modules/arm-template-test/).
