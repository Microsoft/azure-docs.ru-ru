---
title: Изменение текстовых последовательностей runbook в службе автоматизации Azure
description: В этой статье рассказывается, как использовать текстовый редактор службы автоматизации Azure для работы с последовательностями runbook PowerShell и рабочих процессов PowerShell.
services: automation
ms.service: automation
ms.subservice: process-automation
author: mgoedtel
ms.author: magoedte
ms.date: 08/01/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 8d6b786ffaf309e147de27e8cd8be314a3d8a5fb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98896992"
---
# <a name="edit-textual-runbooks-in-azure-automation"></a>Изменение текстовых последовательностей runbook в службе автоматизации Azure

Текстовый редактор в службе автоматизации Azure можно использовать для изменения [последовательностей runbook PowerShell](automation-runbook-types.md#powershell-runbooks) и [последовательностей runbook рабочих процессов PowerShell](automation-runbook-types.md#powershell-workflow-runbooks). Этот редактор имеет стандартные функции редакторов кода, таких как IntelliSense. В нем также используется цветовая маркировка и дополнительные специальные функции, предназначенные для упрощения доступа к стандартным ресурсам runbook. 

Текстовый редактор включает функцию, позволяющую вставлять в модуль runbook код для командлетов, ресурсов и дочерних модулей. Чтобы не вводить этот код самостоятельно, выберите его из списка доступных ресурсов, и редактор вставит соответствующий код в runbook.

У каждого модуля Runbook в службе автоматизации Azure есть черновая и опубликованная версия. Вы можете изменить черновую версию модуля Runbook, а затем опубликовать ее для последующего использования. Опубликованную версию изменить нельзя. Дополнительные сведения см. в разделе [Публикация последовательности runbook](manage-runbooks.md#publish-a-runbook).

В этой статье подробно описано, как использовать этот редактор для выполнения разнообразных задач. Инструкции для [графических последовательностей runbook](automation-runbook-types.md#graphical-runbooks) будут отличаться. Сведения о работе с такими последовательностями runbook приведены в статье [Графическая разработка в службе автоматизации Azure](automation-graphical-authoring-intro.md).

## <a name="edit-a-runbook-with-the-azure-portal"></a>Изменение последовательности runbook с помощью портала Azure

1. На портале Azure выберите свою учетную запись службы автоматизации.
2. В разделе **Автоматизация процессов** выберите **Модули Runbook**, чтобы открыть список модулей.
3. Выберите последовательность runbook, которую нужно изменить, и нажмите кнопку **Изменить**.
4. Внесите изменения в модуль runbook.
5. Закончив изменения, нажмите кнопку **Сохранить** .
6. Чтобы опубликовать последнюю черновую версию последовательности runbook, нажмите кнопку **Опубликовать**.

### <a name="insert-a-cmdlet-into-a-runbook"></a>Вставка командлета в runbook

1. На холсте текстового редактора установите курсор в то место, куда нужно вставить командлет.
2. Разверните узел **Командлеты** в элементе управления «Библиотека».
3. Откройте модуль, в котором находится нужный командлет.
4. Щелкните правой кнопкой мыши имя командлета, который нужно вставить, и выберите **Добавить на холст**. Если у командлета больше одного набора параметров, редактор добавит набор параметров по умолчанию. Вы также можете развернуть командлет и выбрать другой набор параметров.
5. Обратите внимание, что код для командлета вставляется с полным списком параметров.
6. Для всех необходимых параметров укажите нужное значение вместо значения, заключенного в фигурные скобки <>. Удалите ненужные параметры.

### <a name="insert-code-for-a-child-runbook-into-a-runbook"></a>Вставка кода для дочерней последовательности runbook в последовательность runbook

1. На холсте текстового редактора установите курсор в то место, куда нужно вставить код для [дочерней последовательности runbook](automation-child-runbooks.md).
2. Разверните узел **Модули Runbook** в элементе управления «Библиотека».
3. Щелкните правой кнопкой мыши модуль runbook, который нужно вставить, и выберите **Добавить на холст**.
4. Код для дочернего модуля Runbook вставляется со всеми заполнителями для всех параметров модуля Runbook.
5. Замените заполнители на соответствующие значения для каждого параметра.

### <a name="insert-an-asset-into-a-runbook"></a>Вставка ресурса в runbook

1. На холсте текстового редактора установите курсор в то место, куда нужно вставить код для дочерней последовательности runbook.
2. Разверните узел **Ресурс-контейнеры** в элементе управления «Библиотека».
3. Разверните узел для нужного типа ресурса.
4. Щелкните правой кнопкой мыши имя ресурса, который нужно вставить, и выберите **Добавить на холст**. Для [ресурсов-контейнеров переменных](./shared-resources/variables.md) выберите **Добавить "Получить переменную" на холст** или **Добавить "Установить переменную" на холст** в зависимости от того, нужно ли вам получить или задать переменную.
5. Обратите внимание, что код для ресурса вставляется в последовательность runbook.

## <a name="edit-an-azure-automation-runbook-using-windows-powershell"></a>Изменение последовательности runbook в службе автоматизации Azure с помощью Windows PowerShell

Чтобы изменить последовательность runbook с помощью Windows PowerShell, используйте любой редактор и сохраните ее в виде файла **PS1**. Для получения содержимого последовательности runbook можно использовать командлет [Export-AzAutomationRunbook](/powershell/module/Az.Automation/Export-AzAutomationRunbook). Вы можете использовать командлет [Import-AzAutomationRunbook](/powershell/module/Az.Automation/import-azautomationrunbook), чтобы заменить существующую последовательность runbook на измененную.

### <a name="retrieve-the-contents-of-a-runbook-using-windows-powershell"></a>Извлечение содержимого последовательности runbook с помощью Windows PowerShell

Команды в приведенном ниже примере показывают, как извлечь сценарий для модуля и сохранить его в файл сценария. В этом примере извлекается черновая версия. Извлечь можно также опубликованную версию последовательности runbook, но изменить ее нельзя.

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Export-AzAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -OutputFolder $scriptFolder -Slot Draft
```

### <a name="change-the-contents-of-a-runbook-using-windows-powershell"></a>Изменение содержимого последовательности runbook с помощью Windows PowerShell

Команды в приведенном ниже примере показывают, как заменить существующее содержимое модуля Runbook на содержимое файла сценария. 

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Import-AzAutomationRunbook -Path "$scriptfolder\Hello-World.ps1" -Name $runbookName -Type PowerShell -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -Force
Publish-AzAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

* [Управление последовательностями runbook в службе автоматизации Azure.](manage-runbooks.md)
* [Изучение рабочего процесса PowerShell.](automation-powershell-workflow.md)
* [Графическая разработка в службе автоматизации Azure.](automation-graphical-authoring-intro.md)
* [Сертификаты.](./shared-resources/certificates.md)
* [Соединения](automation-connections.md).
* [Учетные данные](./shared-resources/credentials.md).
* [Расписания](./shared-resources/schedules.md).
* [Переменные](./shared-resources/variables.md).
* [Справочник по командлетам PowerShell.](/powershell/module/az.automation)
