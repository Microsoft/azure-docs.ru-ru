---
title: Управление переменными в службе автоматизации Azure
description: В этой статье рассказывается, как работать с переменными в модулях runbook и конфигурациях DSC.
services: automation
ms.subservice: shared-capabilities
ms.date: 12/01/2020
ms.topic: conceptual
ms.openlocfilehash: 6db0c82c034aab97deee1be4aa8bdc54368521bc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98131531"
---
# <a name="manage-variables-in-azure-automation"></a>Управление переменными в службе автоматизации Azure

Ресурсы-контейнеры переменных — это значения, которые доступны для всех модулей runbook и конфигураций DSC в учетной записи службы автоматизации. Вы можете управлять ими на портале Azure, посредством PowerShell, в модуле runbook или в конфигурации DSC.

Переменные автоматизации полезны в следующих случаях:

- совместное использование значения несколькими модулями runbook или конфигурациями DSC;

- совместное использование значения несколькими заданиями из одного модуля runbook или конфигурации DSC;

- управление значением, используемым модулями runbook или конфигурациями DSC, с портала или из командной строки PowerShell. Примером может быть набор типичных элементов конфигурации, таких как список имен виртуальных машин, определенная группа ресурсов, доменное имя AD и т. д.  

Служба автоматизации Azure сохраняет переменные, чтобы они были доступны даже после сбоя модуля runbook или конфигурации DSC. Это также позволяет модулю runbook или конфигурации DSC задать значение, которое может использоваться другим модулем или этим же модулем либо конфигурацией DSC при следующем запуске.

Служба автоматизации Azure хранит каждую зашифрованную переменную безопасным образом. При создании переменной можно указать ее шифрование и хранение с помощью службы автоматизации Azure в качестве безопасного ресурса. После создания переменной вы не сможете изменить ее состояние шифрования, не создав переменную снова. Если у вас есть переменные учетной записи службы автоматизации, в которых хранятся еще не зашифрованные конфиденциальные данные, удалите их и создайте снова в виде зашифрованных переменных. Центр безопасности Azure рекомендует шифровать все переменные службы автоматизации Azure, как описано в рекомендации [Переменные учетной записи службы автоматизации должны быть зашифрованы](../../security-center/recommendations-reference.md#recs-compute). Если у вас есть незашифрованные переменные, которые нужно исключить из этой рекомендации по безопасности, выполните указания, приведенные в статье [Исключение ресурса из рекомендаций и оценки безопасности](../../security-center/exempt-resource.md), чтобы создать правило исключения.

>[!NOTE]
>Безопасные средства в службе автоматизации Azure включают учетные данные, сертификаты, подключения и зашифрованные переменные. Эти ресурсы шифруются и хранятся в службе автоматизации Azure с помощью уникального ключа, который создается для каждой учетной записи службы автоматизации. Служба автоматизации Azure хранит ключ в управляемом системой хранилище ключей Key Vault. Перед сохранением защищенного ресурса служба автоматизации Azure загружает ключ из Key Vault, а затем использует его для шифрования ресурса.

## <a name="variable-types"></a>Типы переменных

При создании переменной с помощью портала Azure выберите тип данных с помощью раскрывающегося списка, чтобы портал мог вывести соответствующий элемент управления для ввода значения переменной. Ниже перечислены типы переменных, которые доступны в службе автоматизации Azure.

* Строка
* Целое число
* Дата и время
* Логическое
* NULL

Переменная не ограничена указанным типом данных. Если значение должно иметь другой тип, следует задать его с помощью Windows PowerShell. Если указать `Not defined`, значение переменной будет задано равным NULL. Значение задается с помощью командлета [Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable) или внутреннего командлета `Set-AutomationVariable`.

Создать или изменить значение сложной переменной на портале Azure невозможно. Однако с помощью Windows PowerShell можно указать значение любого типа. Сложные типы извлекаются в виде [Newtonsoft.Js. LINQ. Жпроперти](https://www.newtonsoft.com/json/help/html/N_Newtonsoft_Json_Linq.htm) для сложного типа объекта вместо типа PSObject [PSCustomObject](/dotnet/api/system.management.automation.pscustomobject).

Можно сохранить несколько значений в отдельной переменной, создав массив или хэш-таблицу и сохранив ее в переменную.

>[!NOTE]
>Длина значения переменной с именем виртуальной машины не может превышать 80 символов. Длина значения переменной с именем группы ресурсов не может превышать 90 символов. См. статью [Правила и ограничения именования для ресурсов Azure](../../azure-resource-manager/management/resource-name-rules.md).

## <a name="powershell-cmdlets-to-access-variables"></a>Командлеты PowerShell для доступа к переменным

Командлеты, представленные в следующей таблице, используются для создания переменных службы автоматизации с помощью PowerShell и управления ими. Они поставляются в составе [модулей Az](modules.md#az-modules).

| Командлет | Описание |
|:---|:---|
|[Get-AzAutomationVariable](/powershell/module/az.automation/get-azautomationvariable) | Получает значение существующей переменной. Если значение простого типа, возвращается тот же тип. Если это сложный тип, возвращается тип `PSCustomObject`. <br>**Примечание.** Этот командлет нельзя использовать для получения значения зашифрованной переменной. Единственный способ получить такое значение — использовать внутренний командлет `Get-AutomationVariable` в модуле runbook или конфигурации DSC. См. раздел [Внутренние командлеты для доступа к переменным](#internal-cmdlets-to-access-variables). |
|[New-AzAutomationVariable](/powershell/module/az.automation/new-azautomationvariable) | Создает новую переменную и устанавливает ее значение.|
|[Remove-AzAutomationVariable](/powershell/module/az.automation/remove-azautomationvariable)| Удаляет существующую переменную.|
|[Set-AzAutomationVariable](/powershell/module/az.automation/set-azautomationvariable)| Получает значение существующей переменной. |

## <a name="internal-cmdlets-to-access-variables"></a>Внутренние командлеты для доступа к переменным

Внутренние командлеты в приведенной ниже таблице используются для доступа к переменным в модулях runbook и конфигурациях DSC. Они входят в состав глобального модуля `Orchestrator.AssetManagement.Cmdlets`. Дополнительные сведения см. в статье [Внутренние командлеты](modules.md#internal-cmdlets).

| Внутренний командлет | Описание |
|:---|:---|
|`Get-AutomationVariable`|Получает значение существующей переменной.|
|`Set-AutomationVariable`|Получает значение существующей переменной.|

> [!NOTE]
> Не следует использовать переменные в параметре `Name` командлета `Get-AutomationVariable`, вызываемого из runbook или конфигурации DSC. Такие переменные могут усложнить обнаружение зависимостей между модулями runbook и переменными службы автоматизации во время разработки.

Командлет `Get-AutomationVariable` не работает в PowerShell. Он работает только в модуле runbook или конфигурации DSC. Например, чтобы увидеть значение зашифрованной переменной, можно создать модуль runbook для получения переменной, а затем записать ее в выходной поток:

```powershell
$mytestencryptvar = Get-AutomationVariable -Name TestVariable
Write-output "The encrypted value of the variable is: $mytestencryptvar"
```

## <a name="python-functions-to-access-variables"></a>Функции Python для доступа к переменным

Функции в следующей таблице используются для доступа к переменным в модуле Runbook Python 2 и 3. Модули Runbook Python 3 сейчас доступны в предварительной версии.

|Функции Python|Описание|
|:---|:---|
|`automationassets.get_automation_variable`|Получает значение существующей переменной. |
|`automationassets.set_automation_variable`|Получает значение существующей переменной. |

> [!NOTE]
> Импортируйте модуль `automationassets` в верхнюю часть модуля runbook на Python, чтобы получить доступ к функциям ресурса.

## <a name="create-and-get-a-variable"></a>Создание и получение переменной

>[!NOTE]
>Если требуется отменить шифрование для переменной, необходимо удалить переменную и создать ее повторно как незашифрованную.

### <a name="create-and-get-a-variable-using-the-azure-portal"></a>Создание и получение переменной с помощью портала Azure

1. В учетной записи службы автоматизации в области слева выберите **переменные** в разделе **Общие ресурсы**.
2. На странице **переменные** выберите **Добавить переменную**.
3. Заполните параметры на странице **Новая переменная** , а затем нажмите кнопку **создать** , чтобы сохранить новую переменную.

> [!NOTE]
> После сохранения зашифрованной переменной ее нельзя просмотреть на портале. Ее можно только изменять.

### <a name="create-and-get-a-variable-in-windows-powershell"></a>Создание и получение переменной в Windows PowerShell

В модуле runbook или конфигурации DSC для создания переменной и задания ее начального значения используется командлет `New-AzAutomationVariable`. Если переменная зашифрована, в вызове должен использоваться параметр `Encrypted`. Скрипт может получить значение переменной с помощью `Get-AzAutomationVariable`.

>[!NOTE]
>Скрипт PowerShell не может получить зашифрованное значение. Единственный способ сделать это — использовать внутренний командлет `Get-AutomationVariable`.

В приведенном ниже примере показано, как создать строковую переменную, а затем вернуть ее значение.

```powershell
New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" 
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable' `
–Encrypted $false –Value 'My String'
$string = (Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyStringVariable').Value
```

В приведенном ниже примере показано, как создать переменную сложного типа и вернуть ее свойства. В этом случае используется объект виртуальной машины из [Get-AzVM](/powershell/module/Az.Compute/Get-AzVM) , указывающий подмножество его свойств.

```powershell
$vm = Get-AzVM -ResourceGroupName "ResourceGroup01" –Name "VM01" | Select Name, Location, Extensions
New-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable" –Encrypted $false –Value $vm

$vmValue = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name "MyComplexVariable"

$vmName = $vmValue.Name
$vmExtensions = $vmValue.Extensions
```

## <a name="textual-runbook-examples"></a>Примеры текстовых модулей runbook

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

В приведенном ниже примере показано, как задать и получить переменную в текстовом модуле runbook. В этом примере предполагается создание целочисленных переменных с именами `NumberOfIterations` и `NumberOfRunnings` и строковой переменной с именем `SampleMessage`.

```powershell
$NumberOfIterations = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfIterations'
$NumberOfRunnings = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" -Name 'NumberOfRunnings'
$SampleMessage = Get-AutomationVariable -Name 'SampleMessage'

Write-Output "Runbook has been run $NumberOfRunnings times."

for ($i = 1; $i -le $NumberOfIterations; $i++) {
    Write-Output "$i`: $SampleMessage"
}
Set-AzAutomationVariable -ResourceGroupName "ResourceGroup01" –AutomationAccountName "MyAutomationAccount" –Name NumberOfRunnings –Value ($NumberOfRunnings += 1)
```

# <a name="python-2"></a>[Python 2;](#tab/python2)

В приведенном ниже примере показано, как получить и задать переменную, а также обработать исключение для несуществующей переменной в модуле runbook Python 2.

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print "variable not found"
```

# <a name="python-3"></a>[Python 3](#tab/python3);

В следующем примере показано, как получить переменную, задать переменную и выполнить обработку исключения для несуществующей переменной в модуле Runbook Python 3 (Предварительная версия).

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a variable
value = automationassets.get_automation_variable("test-variable")
print value

# set a variable (value can be int/bool/string)
automationassets.set_automation_variable("test-variable", True)
automationassets.set_automation_variable("test-variable", 4)
automationassets.set_automation_variable("test-variable", "test-string")

# handle a non-existent variable exception
try:
    value = automationassets.get_automation_variable("nonexisting variable")
except AutomationAssetNotFound:
    print ("variable not found")
```

---

## <a name="graphical-runbook-examples"></a>Примеры графических модулей runbook

В графическом модуле runbook можно добавлять действия для внутренних командлетов `Get-AutomationVariable` и `Set-AutomationVariable`. Щелкните правой кнопкой мыши каждую переменную в области "Библиотека" графического редактора и выберите нужное действие.

![Добавление переменной в полотно](../media/variables/runbook-variable-add-canvas.png)

На рисунке ниже приведен пример действий для добавления в переменную простого значения в графическом модуле runbook. В этом примере действие для `Get-AzVM` извлекает одну виртуальную машину Azure и сохраняет имя компьютера в существующей строковой переменной службы автоматизации. Не имеет значения, [является ли ссылка конвейером или последовательностью](../automation-graphical-authoring-intro.md#use-links-for-workflow), так как в выходных данных ожидается только одно значение.

![Задание простой переменной](../media/variables/runbook-set-simple-variable.png)

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о командлетах, используемых для доступа к переменным, см. в статье [Управление модулями в службе автоматизации Azure](modules.md).
* Общие сведения о модулях runbook см. в статье [Выполнение модуля Runbook в службе автоматизации Azure](../automation-runbook-execution.md).
* Дополнительные сведения о конфигурациях DSC см. в разделе [Общие сведения о настройке состояния службы автоматизации Azure](../automation-dsc-overview.md).
