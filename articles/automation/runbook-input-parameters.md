---
title: Настройка входных параметров runbook в службе автоматизации Azure
description: В этой статье рассказывается, как настроить входные параметры runbook, чтобы обеспечить передачу данных после запуска последовательности runbook.
services: automation
ms.subservice: process-automation
ms.date: 02/14/2019
ms.topic: conceptual
ms.openlocfilehash: 73e4dbb24b4e7c0c651f7d082c75b0f4a17158b5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98890887"
---
# <a name="configure-runbook-input-parameters"></a>Настройка входных параметров runbook

Входные параметры runbook увеличивают гибкость runbook, позволяя передавать данные в уже запущенную последовательность. За счет настройки параметров можно оптимизировать действия runbook согласно определенным сценариям и средам. В этой статье описывается конфигурация и использование входных параметров в последовательностях runbook.

Вы можете настроить входные параметры runbook PowerShell, рабочего процесса PowerShell и Python, а также графической последовательности runbook. Runbook может использовать несколько параметров с разными типами данных или вообще не использовать параметры. Входные параметры могут быть обязательными или необязательными. Для необязательных параметров можно использовать значения по умолчанию.

Назначение значений входным параметрам для последовательности runbook выполняется при ее запуске. Вы можете запустить последовательность runbook с помощью портала Azure, веб-службы или PowerShell. Модуль можно также запустить как дочерний модуль Runbook, который вызывается с помощью встроенного вызова в другом Runbook.

### <a name="configure-input-parameters-in-powershell-runbooks"></a>Настройка входных параметров в модулях Runbook PowerShell

Последовательности runbook рабочих процессов PowerShell и PowerShell в службе автоматизации Azure поддерживают входные параметры, для определения которых используются следующие свойства. 

| **Свойство** | **Описание** |
|:--- |:--- |
| Тип |Обязательный элемент. Предполагаемый тип данных значения параметра. Допускаются любые типы .NET. |
| Имя |Обязательный элемент. Имя параметра. Это имя должно быть уникальным для runbook, должно начинаться с буквы, может содержать только буквы, цифры или символы подчеркивания. |
| Обязательный |Необязательный параметр. Логическое значение, указывающее, требуется ли для параметра значение. Если установлено значение True, при запуске runbook будет необходимо указывать значение. Если установлено значение False, указывать значение необязательно. Если не указать значение для свойства `Mandatory`, PowerShell по умолчанию считает входной параметр необязательным. |
| Значение по умолчанию |Необязательный параметр. Значение, используемое для параметра, если при запуске runbook не передано входное значение. Последовательность runbook может установить значение по умолчанию для любого параметра. |

Наряду с перечисленными выше входными параметрами Windows PowerShell поддерживает дополнительные атрибуты входных параметров, например проверки, псевдонимы и наборы параметров. Однако в настоящее время служба автоматизации Azure поддерживает только перечисленные здесь свойства входных параметров.

Например, рассмотрим определение параметра в последовательности runbook рабочего процесса PowerShell. Для этого определения используется следующий общий формат, где несколько параметров разделяются запятыми.

```powershell
Param
(
  [Parameter (Mandatory= $true/$false)]
  [Type] $Name1 = <Default value>,

  [Parameter (Mandatory= $true/$false)]
  [Type] $Name2 = <Default value>
)
```

Теперь настроим входные параметры последовательности runbook рабочего процесса PowerShell, которая выводит сведения об одной или всех виртуальных машинах в группе ресурсов. Как показано на снимке экрана ниже, у такой последовательности runbook будет два параметра: имя виртуальной машины (`VMName`) и имя группы ресурсов (`resourceGroupName`).

![Рабочий процесс PowerShell службы автоматизации](media/automation-runbook-input-parameters/automation-01-powershellworkflow.png)

В этом определении входные параметры — это простые параметры строкового типа.

Обратите внимание, что последовательности runbook PowerShell и рабочих процессов PowerShell поддерживают для входных параметров все простые и сложные типы, например `Object` или `PSCredential`. Чтобы передать значение при наличии в runbook входного параметра object, следует использовать хэш-таблицу PowerShell с парами имя-значение. Предположим, например, что в runbook есть приведенный ниже параметр.

```powershell
[Parameter (Mandatory = $true)]
[object] $FullName
```

В этом случае вы можете передать для него следующее значение.

```powershell
@{"FirstName"="Joe";"MiddleName"="Bob";"LastName"="Smith"}
```

> [!NOTE]
> Если не передать значение необязательному строковому параметру со значением по умолчанию NULL, значением параметра будет не NULL, а пустая строка.

### <a name="configure-input-parameters-in-graphical-runbooks"></a>Настройка входных параметров в графических модулях Runbook

Чтобы продемонстрировать настройку входных параметров графической последовательности runbook, создадим последовательность runbook, которая выводит сведения об одной или всех виртуальных машинах в группе ресурсов. Дополнительные сведения см. в учебнике по [созданию графической последовательности runbook](./learn/automation-tutorial-runbook-graphical.md).

В графической последовательности runbook используются следующие основные действия runbook:

* настройка учетной записи запуска от имени Azure для проверки подлинности в Azure; 
* определение командлета [Get-AzVM](/powershell/module/az.compute/get-azvm) для получения свойств виртуальной машины;
* использование действия [Write-Output](/powershell/module/microsoft.powershell.utility/write-output) для вывода имен виртуальных машин. 

Действие `Get-AzVM` определяет два входных значения: имя виртуальной машины и имя группы ресурсов. Так как эти имена могут отличаться при каждом запуске последовательности runbook, необходимо добавить входные параметры, чтоб она их принимала. См. статью [Графическая разработка в службе автоматизации Azure](automation-graphical-authoring-intro.md).

Чтобы настроить входные параметры, выполните следующие действия.

1. На странице "Модули Runbook" выберите графическую последовательность runbook и щелкните **Изменить**.
2. В графическом редакторе нажмите кнопку **Входные и выходные данные**, а затем выберите **Добавить входные данные**, чтобы открыть область входных параметров runbook.

   ![Графический модуль Runbook службы автоматизации](media/automation-runbook-input-parameters/automation-02-graphical-runbok-editor.png)

3. В элементе управления "Входные и выходные данные" отображается список входных параметров, определенных для runbook. Здесь колонке можно добавить новый входной параметр или изменить конфигурацию имеющегося. Чтобы добавить новый параметр для последовательности runbook, нажмите кнопку **Добавить входные данные**. Откроется колонка **Входной параметр Runbook**, где можно настроить параметры с помощью свойств, определенных в статье [Графическая разработка в службе автоматизации Azure](automation-graphical-authoring-intro.md).

    ![Добавление новых входных данных](media/automation-runbook-input-parameters/automation-runbook-input-parameter-new.png)
4. Создайте два параметра со следующими свойствами, которые будут использоваться действием `Get-AzVM`, а затем нажмите кнопку **ОК**.

   * Параметр 1:
        * **Имя** -- **VMName**
        * **Тип** — строка
        * **Обязательный** -- **Нет**

   * Параметр 2:
        * **Имя** -- **resourceGroupName**
        * **Тип** — строка
        * **Обязательный** -- **Нет**
        * **Значение по умолчанию** -- **особый**
        * Настраиваемое значение по умолчанию — имя группы ресурсов, содержащей виртуальные машины

5. Просмотрите параметры в элементе управления для входных и выходных данных. 
6. Нажмите кнопку **ОК** снова, а затем нажмите кнопку **Сохранить**.
7. Щелкните **Опубликовать**, чтобы опубликовать runbook.

### <a name="configure-input-parameters-in-python-runbooks"></a>Настройка входных параметров в модулях Runbook Python

В отличие от последовательностей runbook PowerShell и рабочих процессов PowerShell, а также графических последовательностей runbook, последовательности runbook Python не принимают именованные параметры. Редактор runbook анализирует все входные параметры как массив значений аргументов. Для доступа к массиву можно импортировать модуль `sys` в свой скрипт Python, а затем используйте массив `sys.argv`. Обратите внимание, что первый элемент массива (`sys.argv[0]`) является именем скрипта. Поэтому фактически первым входным параметром является `sys.argv[1]`.

Пример использования входных параметров в модуле Runbook Python см. в статье [Мой первый модуль Runbook Python в службе автоматизации Azure](./learn/automation-tutorial-runbook-textual-python2.md).

## <a name="assign-values-to-input-parameters-in-runbooks"></a>Назначение значений входным параметрам в модулях Runbook

В этом разделе описаны несколько способов передачи значений во входные параметры последовательностей runbook. Значения параметров можно назначать при выполнении следующих действий:

* [Запуск модуля Runbook](#start-a-runbook-and-assign-parameters)
* [Тестирование модуля Runbook](#test-a-runbook-and-assign-parameters)
* [Подключение расписания для последовательности runbook](#link-a-schedule-to-a-runbook-and-assign-parameters)
* [Создание веб-перехватчика для runbook](#create-a-webhook-for-a-runbook-and-assign-parameters)

### <a name="start-a-runbook-and-assign-parameters"></a>Запуск Runbook и назначение параметров

Существует много способов запуска последовательности runbook: с помощью пользовательского интерфейса портала Azure, веб-перехватчика, командлетов PowerShell, REST API или пакета SDK. 

#### <a name="start-a-published-runbook-using-the-azure-portal-and-assign-parameters"></a>Запуск опубликованной последовательности runbook на портале Azure и назначение параметров

При [запуске последовательности runbook](start-runbooks.md#start-a-runbook-with-the-azure-portal) на портале Azure открывается колонка **Запуск Runbook**, где можно ввести значения для созданных параметров.

![Начало работы с порталом](media/automation-runbook-input-parameters/automation-04-startrunbookusingportal.png)

В метке под полем ввода отображаются свойства, заданные для определения атрибутов параметров, например обязательный или необязательный, тип, значение по умолчанию. Подсказка рядом с именем параметра также содержит основные сведения, необходимые для принятия решений об установке входных значений параметра. 

> [!NOTE]
> Параметры строкового типа поддерживают пустые строковые значения. Если ввести `[EmptyString]` в поле входного параметра, ему передастся пустая строка. Кроме того, строковые параметры не поддерживают значение NULL. Если строковому параметру не передать значение, PowerShell интерпретирует отсутствие значения как значение NULL.

#### <a name="start-a-published-runbook-using-powershell-cmdlets-and-assign-parameters"></a>Запуск опубликованной последовательности runbook с помощью командлетов PowerShell и назначение параметров

* **Командлеты Azure Resource Manager.** Последовательность runbook службы автоматизации, созданную в группе ресурсов, вы можете запустить с помощью командлета [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook).

   ```powershell
     $params = @{"VMName"="WSVMClassic";"resourceGroupeName"="WSVMClassicSG"}
  
     Start-AzAutomationRunbook -AutomationAccountName "TestAutomation" -Name "Get-AzureVMGraphical" –ResourceGroupName $resourceGroupName -Parameters $params
   ```

* **Командлеты классической модели развертывания Azure.** Модуль Runbook автоматизации, созданный в группе ресурсов по умолчанию, вы можете запустить с помощью командлета [Start-AzureAutomationRunbook](/powershell/module/servicemanagement/azure.service/start-azureautomationrunbook).
  
   ```powershell
     $params = @{"VMName"="WSVMClassic"; "ServiceName"="WSVMClassicSG"}
  
     Start-AzureAutomationRunbook -AutomationAccountName "TestAutomation" -Name "Get-AzureVMGraphical" -Parameters $params
   ```

> [!NOTE]
> При запуске последовательности runbook с помощью командлетов PowerShell создается параметр по умолчанию `MicrosoftApplicationManagementStartedBy` со значением `PowerShell`. Этот параметр вы можете просмотреть на странице "Сведения о задании".  

#### <a name="start-a-runbook-using-an-sdk-and-assign-parameters"></a>Запуск runbook с использованием пакета SDK и назначение параметров

* **Метод Azure Resource Manager.** Для запуска runbook можно использовать пакет SDK для языка программирования. Ниже приведен фрагмент кода C# для запуска модуля Runbook в вашей учетной записи службы автоматизации. Весь код можно просмотреть в нашем [репозитории GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/automation/Microsoft.Azure.Management.Automation/tests/TestSupport/AutomationTestBase.cs).

   ```csharp
   public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
      {
        var response = AutomationClient.Jobs.Create(resourceGroupName, automationAccount, new JobCreateParameters
         {
            Properties = new JobCreateProperties
             {
                Runbook = new RunbookAssociationProperty
                 {
                   Name = runbookName
                 },
                   Parameters = parameters
             }
         });
      return response.Job;
      }
   ```

* **Метод классической модели развертывания Azure.** Для запуска модуля Runbook можете использовать пакет SDK для языка программирования. Ниже приведен фрагмент кода C# для запуска модуля Runbook в вашей учетной записи службы автоматизации. Весь код можно просмотреть в нашем [репозитории GitHub](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/automation/Microsoft.Azure.Management.Automation/tests/TestSupport/AutomationTestBase.cs).

   ```csharp
  public Job StartRunbook(string runbookName, IDictionary<string, string> parameters = null)
    {
      var response = AutomationClient.Jobs.Create(automationAccount, new JobCreateParameters
    {
      Properties = new JobCreateProperties
         {
           Runbook = new RunbookAssociationProperty
         {
           Name = runbookName
              },
                Parameters = parameters
              }
       });
      return response.Job;
    }
   ```

   Чтобы запустить этот метод, создайте словарь для хранения параметров runbook `VMName` и `resourceGroupName`, а также их значений. Запустите модуль Runbook. Ниже приведен фрагмент кода C# для вызова метода, определенного выше.

   ```csharp
   IDictionary<string, string> RunbookParameters = new Dictionary<string, string>();
  
   // Add parameters to the dictionary.
  RunbookParameters.Add("VMName", "WSVMClassic");
   RunbookParameters.Add("resourceGroupName", "WSSC1");
  
   //Call the StartRunbook method with parameters
   StartRunbook("Get-AzureVMGraphical", RunbookParameters);
   ```

#### <a name="start-a-runbook-using-the-rest-api-and-assign-parameters"></a>Запуск runbook с использованием REST API и назначение параметров

Задание runbook можно создать и запустить с помощью REST API службы автоматизации Azure. Для этого необходимо использовать метод `PUT` и следующий универсальный URI запроса: `https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Automation/automationAccounts/{automationAccountName}/jobs/{jobName}?api-version=2017-05-15-preview`

В URI запроса замените следующие параметры:

* `subscriptionId`: идентификатор подписки Azure;  
* `resourceGroupName`: имя группы ресурсов для учетной записи службы автоматизации;
* `automationAccountName`: имя учетной записи службы автоматизации в указанной облачной службе;  
* `jobName`: GUID для задания. Идентификаторы GUID в PowerShell можно создать с помощью `[GUID]::NewGuid().ToString()*`.

Чтобы передать параметры в задание runbook, используйте текст запроса. Он принимает следующую информацию, представленную в формате JSON:

* Имя runbook. Обязательный элемент. Имя модуля Runbook для запуска задания.  
* Параметры runbook. Необязательный параметр. Это словарь со списком параметров (именами и значениями) в формате, где имя должно быть строкового типа, а значение может быть любым допустимым значением JSON.

Если вы хотите запустить последовательность runbook **Get-AzureVMTextual**, созданную ранее с параметрами `VMName` и `resourceGroupName`, используйте следующий формат JSON в тексте запроса.

```json
    {
      "properties":{
        "runbook":{
        "name":"Get-AzureVMTextual"},
      "parameters":{
         "VMName":"WindowsVM",
         "resourceGroupName":"ContosoSales"}
        }
    }
```

Если задание успешно создано, возвращается код состояния HTTP 201. Дополнительные сведения о заголовках и тексте ответа см. в статье о [создании задания runbook с помощью REST API](/rest/api/automation/job/create).

### <a name="test-a-runbook-and-assign-parameters"></a>Тестирование Runbook и назначение параметров

При [тестировании черновой версии runbook](./manage-runbooks.md) с помощью параметра тестирования откроется страница "Тест". Эта страница используется для настройки значений созданных вами параметров.

![Тестирование и назначение параметров](media/automation-runbook-input-parameters/automation-06-testandassignparameters.png)

### <a name="link-a-schedule-to-a-runbook-and-assign-parameters"></a>Связывание Runbook с расписанием и назначение параметров

Вы можете [связать расписание](./shared-resources/schedules.md) с модулем Runbook, чтобы он запускался в определенное время. При создании расписания вы назначаете входные параметры, значения которых модуль runbook использует во время запуска по расписанию. Расписание нельзя сохранить, если не указаны значения для всех обязательных параметров.

![Создание расписания и назначение параметров](media/automation-runbook-input-parameters/automation-07-scheduleandassignparameters.png)

### <a name="create-a-webhook-for-a-runbook-and-assign-parameters"></a>Создание объекта Webhook для модуля Runbook и назначение параметров

Для модуля Runbook можно создать объект [Webhook](automation-webhooks.md) и настроить входные параметры. Веб-перехватчик нельзя сохранить, если не указаны значения для всех обязательных параметров.

![Создание Webhook и назначение параметров](media/automation-runbook-input-parameters/automation-08-createwebhookandassignparameters.png)

При выполнении последовательности runbook с помощью веб-перехватчика происходит отправка предопределенного входного параметра `[WebhookData](automation-webhooks.md)` и определенных вами входных параметров. 

![Параметр WebhookData](media/automation-runbook-input-parameters/automation-09-webhook-data-parameters.png)

## <a name="pass-a-json-object-to-a-runbook"></a>Передача объекта JSON в Runbook

Иногда может быть полезным хранить данные, которые необходимо передать в модуль runbook, в файле JSON. Например, можно создать файл JSON, который содержит все параметры, которые необходимо передать в runbook. Чтобы сделать это, необходимо преобразовать код JSON в строку, а затем преобразовать строку в объект PowerShell перед его передачей в последовательность runbook.

В этом разделе используется пример, в котором скрипт PowerShell вызывает [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook) для запуска runbook PowerShell, передавая содержимое файла JSON в runbook. Последовательность runbook PowerShell запускает виртуальную машину Azure, получая параметры для нее из объекта JSON.

### <a name="create-the-json-file"></a>Создание файла JSON

Введите следующий код в текстовый файл и сохраните его как **test.json** в любом месте на своем локальном компьютере.

```json
{
   "VmName" : "TestVM",
   "ResourceGroup" : "AzureAutomationTest"
}
```

### <a name="create-the-runbook"></a>Создание модуля runbook

Создайте новую последовательность runbook PowerShell с именем **Test-Json** в службе автоматизации Azure. Дополнительные сведения см. в учебнике по [созданию последовательности runbook PowerShell](./learn/automation-tutorial-runbook-textual-powershell.md).

Чтобы принять данные JSON, модуль runbook должен принять объект в качестве входного параметра. Последовательность runbook может использовать свойства, определенные в файле JSON.

```powershell
Param(
     [parameter(Mandatory=$true)]
     [object]$json
)

# Connect to Azure account
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID `
    -ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

# Convert object to actual JSON
$json = $json | ConvertFrom-Json

# Use the values from the JSON object as the parameters for your command
Start-AzVM -Name $json.VMName -ResourceGroupName $json.ResourceGroup
```

Сохраните и опубликуйте этот модуль runbook в учетной записи службы автоматизации.

### <a name="call-the-runbook-from-powershell"></a>Вызов модуля runbook из PowerShell

Теперь можно вызвать модуль runbook с локального компьютера с помощью Azure PowerShell. 

1. Войдите в Azure, как показано ниже. При этом появится запрос ввести учетные данные Azure.

   ```powershell
   Connect-AzAccount
   ```

    >[!NOTE]
    >Для модулей runbook PowerShell `Add-AzAccount` и `Add-AzureRMAccount` являются псевдонимами для `Connect-AzAccount`. Обратите внимание, что эти псевдонимы недоступны для графических последовательностей runbook. Графическая последовательность runbook может использовать только `Connect-AzAccount`.

1. Получите содержимое сохраненного файла JSON и преобразуйте его в строку. `JsonPath` указывает путь, по которому был сохранен файл JSON.

   ```powershell
   $json =  (Get-content -path 'JsonPath\test.json' -Raw) | Out-string
   ```

1. Преобразуйте содержимое строки `$json` в объект PowerShell.

   ```powershell
   $JsonParams = @{"json"=$json}
   ```

1. Создайте хэш-таблицу для параметров `Start-AzAutomationRunbook`. 

   ```powershell
   $RBParams = @{
        AutomationAccountName = 'AATest'
        ResourceGroupName = 'RGTest'
        Name = 'Test-Json'
        Parameters = $JsonParams
   }
   ```

   Обратите внимание, что вы задаете значение `Parameters` объекту PowerShell, который содержит значения из файла JSON.
1. Запустите runbook.

   ```powershell
   $job = Start-AzAutomationRunbook @RBParams
   ```

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о подготовке текстовых последовательностей runbook см. в статье [Изменение текстовых последовательностей runbook в службе автоматизации Azure](automation-edit-textual-runbook.md).
* Сведения о подготовке графических последовательностей runbook см. в статье [о графических runbook в службе автоматизации Azure](automation-graphical-authoring-intro.md).
