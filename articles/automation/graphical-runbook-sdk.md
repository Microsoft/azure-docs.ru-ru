---
title: Использование пакета средств разработки для графического модуля runbook службы автоматизации Azure (предварительная версия)
description: В этой статье описывается использование пакета средств разработки для графического модуля runbook службы автоматизации Azure (предварительная версия).
services: automation
ms.subservice: process-automation
ms.date: 07/20/2018
ms.topic: conceptual
ms.openlocfilehash: 969e60cd08a65adb1dd731aa7c6c3f9872e288fd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "83835042"
---
# <a name="use-the-azure-automation-graphical-runbook-sdk-preview"></a>Использование пакета средств разработки для графического модуля runbook службы автоматизации Azure (предварительная версия)

[Графические модули runbook](automation-graphical-authoring-intro.md) помогают справиться со сложностями базового кода Windows PowerShell или рабочего процесса PowerShell. Пакет средств разработки для графической разработки в службе автоматизации Microsoft Azure позволяет разработчикам создавать и изменять графические модули runbook для использования со службой автоматизации Azure. В этой статье описаны основные шаги по созданию графического модуля runbook на основе готового кода.

## <a name="prerequisites"></a>Предварительные требования

Импортируйте пакет `Orchestrator.GraphRunbook.Model.dll`, скачав [пакет средств разработки](https://www.microsoft.com/download/details.aspx?id=50734).

## <a name="create-a-runbook-object-instance"></a>Создание экземпляра объекта runbook

Укажите ссылку на сборку `Orchestrator.GraphRunbook.Model` и создайте экземпляр класса `Orchestrator.GraphRunbook.Model.GraphRunbook`:

```csharp
using Orchestrator.GraphRunbook.Model;
using Orchestrator.GraphRunbook.Model.ExecutableView;

var runbook = new GraphRunbook();
```

## <a name="add-runbook-parameters"></a>Добавление параметров runbook

Создайте экземпляры объектов `Orchestrator.GraphRunbook.Model.Parameter` и добавьте их в runbook:

```csharp
runbook.AddParameter(
 new Parameter("YourName") {
  TypeName = typeof(string).FullName,
   DefaultValue = "World",
   Optional = true
 });

runbook.AddParameter(
 new Parameter("AnotherParameter") {
  TypeName = typeof(int).FullName, ...
 });
```

## <a name="add-activities-and-links"></a>Добавление действий и ссылок

Создайте экземпляры действий соответствующих типов и добавьте их в runbook:

```csharp
var writeOutputActivityType = new CommandActivityType {
 CommandName = "Write-Output",
  ModuleName = "Microsoft.PowerShell.Utility",
 InputParameterSets = new [] {
  new ParameterSet {
   Name = "Default",
    new [] {
     new Parameter("InputObject"), ...
    }
  },
  ...
 }
};

var outputName = runbook.AddActivity(
 new CommandActivity("Output name", writeOutputActivityType) {
  ParameterSetName = "Default",
   Parameters = new ActivityParameters {
    {
     "InputObject",
     new RunbookParameterValueDescriptor("YourName")
    }
   },
   PositionX = 0,
   PositionY = 0
 });

var initializeRunbookVariable = runbook.AddActivity(
 new WorkflowScriptActivity("Initialize variable") {
  Process = "$a = 0",
   PositionX = 0,
   PositionY = 100
 });
```

Действия реализуются с помощью следующих классов в пространстве имен `Orchestrator.GraphRunbook.Model`.

|Class  |Действие  |
|---------|---------|
|CommandActivity     | Вызывает команду PowerShell (командлет, функция и т. д.).        |
|InvokeRunbookActivity     | Вызывает другой runbook изнутри.        |
|JunctionActivity     | Ожидает завершения всех входящих ветвей.        |
|WorkflowScriptActivity     | Выполняет блок кода PowerShell или рабочего процесса PowerShell (в зависимости от типа runbook) в контексте runbook. Это эффективное средство, но не злоупотребляйте им. В пользовательском интерфейсе этот блок сценария будет отображаться как текст. Подсистема выполнения будет обрабатывать этот предоставленный блок по принципу "черного ящика" и не предпримет попыток проанализировать его содержимое за исключением базовой проверки синтаксиса. Если необходимо просто вызвать одну команду PowerShell, воспользуйтесь CommandActivity.        |

> [!NOTE]
> Не наследуйте собственные действия от предоставленных классов. Служба автоматизации Azure не сможет использовать модули runbook с пользовательскими типами действий.

Необходимо указать параметры `CommandActivity` и `InvokeRunbookActivity` в виде дескрипторов значений, а не прямых значений. Дескрипторы значений указывают, как вычисляются фактические значения параметров. Сейчас указаны следующие дескрипторы значений:


|Дескриптор  |Определение  |
|---------|---------|
|ConstantValueDescriptor     | Ссылается на жестко заданное постоянное значение.        |
|RunbookParameterValueDescriptor     | Ссылается на параметр runbook по имени.        |
|ActivityOutputValueDescriptor     | Ссылается на вышестоящие выходные данные действия, позволяя одному действию "подписаться" на данные, создаваемые другим действием.        |
|AutomationVariableValueDescriptor     | Ссылается на ресурс переменной службы автоматизации по имени.         |
|AutomationCredentialValueDescriptor     | Ссылается на ресурс сертификата службы автоматизации по имени.        |
|AutomationConnectionValueDescriptor     | Ссылается на ресурс подключения службы автоматизации по имени.        |
|PowerShellExpressionValueDescriptor     | Указывает выражение PowerShell свободной формы, которое будет вычисляться непосредственно перед вызовом действия.  <br/>Это эффективное средство, но не злоупотребляйте им. В пользовательском интерфейсе это выражение будет отображаться как текст. Подсистема выполнения будет обрабатывать этот предоставленный блок по принципу "черного ящика" и не предпримет попыток проанализировать его содержимое за исключением базовой проверки синтаксиса. По возможности отдавайте предпочтение более конкретным дескрипторам значений.      |

> [!NOTE]
> Не наследуйте собственные дескрипторы значений от предоставленных классов. Служба автоматизации Azure не может использовать модули runbook с пользовательскими типами дескрипторов значений.

Создайте экземпляры связей, соединяющих действия, и добавьте их в runbook:

```csharp
runbook.AddLink(new Link(activityA, activityB, LinkType.Sequence));

runbook.AddLink(
 new Link(activityB, activityC, LinkType.Pipeline) {
  Condition = string.Format("$ActivityOutput['{0}'] -gt 0", activityB.Name)
 });
```

## <a name="save-the-runbook-to-a-file"></a>Сохранение runbook в файл

Сериализуйте runbook в строку с помощью `Orchestrator.GraphRunbook.Model.Serialization.RunbookSerializer`:

```csharp
var serialized = RunbookSerializer.Serialize(runbook);
```

Эту строку можно сохранить в файл с помощью расширения **.graphrunbook**. Соответствующий модуль runbook можно импортировать в службу автоматизации Azure.
Формат сериализации может измениться в будущих версиях `Orchestrator.GraphRunbook.Model.dll`. Мы гарантируем обратную совместимость: любой runbook, сериализованный с использованием более старой версии `Orchestrator.GraphRunbook.Model.dll`, может быть десериализован с использованием любой более поздней версии. Прямая совместимость не гарантируется: runbook, сериализованный с использованием более поздней версии, не может быть десериализован с использованием старых версий.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Графическая разработка в службе автоматизации Azure](automation-graphical-authoring-intro.md).