---
title: Локальная отладка функций Azure PowerShell
description: Узнайте, как выполнять отладку функций PowerShell при локальном запуске.
author: tylerleonhardt
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: tyleonha
ms.reviewer: glenga
ms.openlocfilehash: a668024db126c82f96756555aba513b77f7d7366
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93422966"
---
# <a name="debug-powershell-azure-functions-locally"></a>Локальная отладка функций Azure PowerShell

Функции Azure позволяют разрабатывать функции в виде скриптов PowerShell.

Вы можете выполнять отладку функций PowerShell локально, как и любые сценарии PowerShell, используя следующие стандартные средства разработки:

* [Visual Studio Code](https://code.visualstudio.com/): бесплатный, упрощенный и открытый исходный редактор с открытым исходным кодом с помощью расширения PowerShell, которое предлагает полный интерфейс разработки PowerShell.
* Консоль PowerShell: Отладка с использованием тех же команд, которые используются для отладки любого другого процесса PowerShell.

[Azure functions Core Tools](functions-run-local.md) поддерживает локальную отладку функций Azure, включая функции PowerShell.

## <a name="example-function-app"></a>Пример приложения функции

Приложение-функция, используемое в этой статье, имеет одну функцию, активируемую HTTP, и имеет следующие файлы:

```
PSFunctionApp
 | - HttpTriggerFunction
 | | - run.ps1
 | | - function.json
 | - local.settings.json
 | - host.json
 | - profile.ps1
```

Это приложение-функция аналогично тому, которое вы получаете при выполнении [краткого руководства по PowerShell](./create-first-function-vs-code-powershell.md).

Код функции в `run.ps1` выглядит как следующий скрипт:

```powershell
param($Request)

$name = $Request.Query.Name

if($name) {
    $status = 200
    $body = "Hello $name"
}
else {
    $status = 400
    $body = "Please pass a name on the query string or in the request body."
}

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = $status
    Body = $body
})
```

## <a name="set-the-attach-point"></a>Установка точки подключения

Для отладки любой функции PowerShell необходимо, чтобы функция была приостановлена, чтобы присоединить отладчик. `Wait-Debugger`Командлет останавливает выполнение и ожидает отладчика.

>[!NOTE]
>При использовании PowerShell 7 не нужно добавлять `Wait-Debugger` вызов в код.

Все, что нужно сделать, — это добавить вызов `Wait-Debugger` командлета прямо над `if` инструкцией следующим образом:

```powershell
param($Request)

$name = $Request.Query.Name

# This is where we will wait for the debugger to attach
Wait-Debugger

if($name) {
    $status = 200
    $body = "Hello $name"
}
# ...
```

Отладка начинается с `if` инструкции. 

`Wait-Debugger`Теперь вы можете выполнять отладку функций с помощью Visual Studio Code или консоли PowerShell.

## <a name="debug-in-visual-studio-code"></a>Отладка в Visual Studio Code

Для отладки функций PowerShell в Visual Studio Code необходимо установить следующие компоненты:

* [Расширение PowerShell для Visual Studio Code](/powershell/scripting/components/vscode/using-vscode)
* [Расширение "Функции Azure" для Visual Studio Code](./create-first-function-cli-powershell.md)
* [PowerShell Core 6,2 или более поздней версии](/powershell/scripting/install/installing-powershell-core-on-windows)

После установки этих зависимостей загрузите существующий проект функций PowerShell или [Создайте свой первый проект функций PowerShell](./create-first-function-vs-code-powershell.md).

>[!NOTE]
> Если в проекте отсутствуют необходимые файлы конфигурации, вам будет предложено добавить их.

### <a name="set-the-powershell-version"></a>Установка версии PowerShell

PowerShell Core устанавливается параллельно с Windows PowerShell. Задайте PowerShell Core в качестве версии PowerShell для использования с расширением PowerShell для Visual Studio Code.

1. Нажмите клавишу F1, чтобы отобразить палету команды, а затем выполните поиск `Session` .

1. Выберите **PowerShell: отобразить меню сеанса**.

1. Если **текущий сеанс** не является **PowerShell Core 6**, выберите **параметр перейти к: PowerShell Core 6**.

При открытии файла PowerShell в нижней правой части окна отображается зеленая версия. При выборе этого текста также отображается меню сеанса. Дополнительные сведения см. в статье [Выбор версии PowerShell для использования с расширением](/powershell/scripting/components/vscode/using-vscode#choosing-a-version-of-powershell-to-use-with-the-extension).

### <a name="start-the-function-app"></a>Запуск приложения функции

Убедитесь, что параметр `Wait-Debugger` задан в функции, в которой необходимо подключить отладчик.  `Wait-Debugger`После добавления можно выполнить отладку приложения функции с помощью Visual Studio Code.

Выберите область **отладки** , а затем **присоединитесь к функции PowerShell**.

![отладчик](https://user-images.githubusercontent.com/2644648/56166073-8a7b3780-5f89-11e9-85ce-36ed38e221a2.png)

Можно также нажать клавишу F5, чтобы начать отладку.

Операция запуска отладки выполняет следующие задачи.

* Выполняется `func extensions install` в терминале для установки расширений функций Azure, необходимых для приложения функции.
* Запускается `func host start` в терминале для запуска приложения функции на узле функций.
* Присоедините отладчик PowerShell к пространству выполнения PowerShell в среде выполнения функций.

>[!NOTE]
> Необходимо убедиться, что Псворкеринпрокконкурренциуппербаунд имеет значение 1, чтобы обеспечить правильную работу по отладке в Visual Studio Code. Это значение по умолчанию.

После запуска приложения-функции вам потребуется отдельная консоль PowerShell для вызова функции, активируемой HTTP.

В этом случае консоль PowerShell является клиентом. `Invoke-RestMethod`Используется для активации функции.

В консоли PowerShell выполните следующую команду:

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

Обратите внимание, что ответ не возвращается немедленно. Это связано с тем, что `Wait-Debugger` подключен отладчик, и выполнение PowerShell перешло в режим приостановки, как только это может быть. Это обусловлено [концепцией BreakAll](#breakall-might-cause-your-debugger-to-break-in-an-unexpected-place), которая описывается далее. После нажатия `continue` кнопки отладчик теперь переключается на строку сразу после `Wait-Debugger` .

На этом этапе отладчик присоединен, и вы можете выполнять все обычные операции отладчика. Дополнительные сведения об использовании отладчика в Visual Studio Code см. [в официальной документации](https://code.visualstudio.com/Docs/editor/debugging#_debug-actions).

После продолжения и полного вызова скрипта вы заметите, что:

* Консоль PowerShell, которая `Invoke-RestMethod` вернула результат
* Интегрированная консоль PowerShell в Visual Studio Code ожидает выполнения скрипта

Позже при вызове той же функции отладчик в расширении PowerShell прерывает работу сразу после `Wait-Debugger` .

## <a name="debugging-in-a-powershell-console"></a>Отладка в консоли PowerShell

>[!NOTE]
> В этом разделе предполагается, что вы прочитали [документацию по Azure functions Core Tools](functions-run-local.md) и узнаете, как использовать `func host start` команду для запуска приложения функции.

Откройте консоль `cd` в каталоге приложения-функции и выполните следующую команду:

```sh
func host start
```

После запуска приложения функции и на `Wait-Debugger` месте можно присоединиться к процессу. Вам понадобятся две консоли PowerShell.

Одна из консолей выступает в качестве клиента. После этого вызывается метод, `Invoke-RestMethod` чтобы активировать функцию. Например, можно выполнить следующую команду:

```powershell
Invoke-RestMethod "http://localhost:7071/api/HttpTrigger?Name=Functions"
```

Вы заметите, что он не возвращает ответ, что является результатом `Wait-Debugger` . Теперь пространство выполнения PowerShell ожидает присоединения отладчика. Давайте присоединяемся.

В другой консоли PowerShell выполните следующую команду:

```powershell
Get-PSHostProcessInfo
```

Этот командлет возвращает таблицу, которая выглядит следующим образом:

```output
ProcessName ProcessId AppDomainName
----------- --------- -------------
dotnet          49988 None
pwsh            43796 None
pwsh            49970 None
pwsh             3533 None
pwsh            79544 None
pwsh            34881 None
pwsh            32071 None
pwsh            88785 None
```

Запишите значение `ProcessId` для элемента в таблице с помощью `ProcessName` As `dotnet` . Этот процесс является приложением-функцией.

Затем выполните следующий фрагмент кода:

```powershell
# This enters into the Azure Functions PowerShell process.
# Put your value of `ProcessId` here.
Enter-PSHostProcess -Id $ProcessId

# This triggers the debugger.
Debug-Runspace 1
```

После запуска отладчик прерывает работу и отображает нечто вроде приведенных ниже выходных данных.

```
Debugging Runspace: Runspace1

To end the debugging session type the 'Detach' command at the debugger prompt, or type 'Ctrl+C' otherwise.

At /Path/To/PSFunctionApp/HttpTriggerFunction/run.ps1:13 char:1
+ if($name) { ...
+ ~~~~~~~~~~~
[DBG]: [Process:49988]: [Runspace1]: PS /Path/To/PSFunctionApp>>
```

На этом этапе вы прекращаете работу в точке останова в [отладчике PowerShell](/powershell/module/microsoft.powershell.core/about/about_debuggers). Здесь можно выполнять все обычные операции отладки: шаг с обходом, шаг с заходом, продолжить, выйти и другие. Чтобы просмотреть полный набор команд отладки, доступных в консоли, выполните `h` `?` команды или.

На этом уровне можно также установить точки останова с помощью `Set-PSBreakpoint` командлета.

После продолжения и полного вызова скрипта вы заметите, что:

* Консоль PowerShell, в которой вы выполнялись, `Invoke-RestMethod` теперь вернула результат.
* Выполняемая консоль PowerShell `Debug-Runspace` ожидает выполнения скрипта.

Одну и ту же функцию можно вызвать снова ( `Invoke-RestMethod` например, с помощью), и отладчик переключается сразу после `Wait-Debugger` команды.

## <a name="considerations-for-debugging"></a>Рекомендации по отладке

При отладке кода функций учитывайте следующие моменты.

### <a name="breakall-might-cause-your-debugger-to-break-in-an-unexpected-place"></a>`BreakAll` может привести к сбою отладчика в непредвиденном месте

Расширение PowerShell использует `Debug-Runspace` , которое, в свою очередь, зависит от `BreakAll` функции PowerShell. Эта функция указывает, что PowerShell должен останавливаться на первой выполняемой команде. Такое поведение дает возможность задавать точки останова в отлаживаемом пространстве выполнения.

Среда выполнения функций Azure выполняет несколько команд до фактического вызова `run.ps1` скрипта, поэтому отладчик может завершить работу в `Microsoft.Azure.Functions.PowerShellWorker.psm1` или `Microsoft.Azure.Functions.PowerShellWorker.psd1` .

Если это произойдет, выполните `continue` команду или, `c` чтобы пропустить эту точку останова. Затем вы останавливается в ожидаемой точке останова.

## <a name="troubleshooting"></a>Устранение неполадок

При возникновении трудностей во время отладки необходимо проверить следующее:

| Проверить | Действие |
|------|------|
| Запустите `func --version` из терминала. Если появляется сообщение об ошибке, которое `func` не удается найти, то основные средства (func.exe) могут отсутствовать в локальной `path` переменной.| [Переустановите основные инструменты](functions-run-local.md#v2).|  
| В Visual Studio Code терминалу по умолчанию необходим доступ к func.exe. Убедитесь, что вы не используете терминал по умолчанию, на котором не установлены основные средства, например подсистема Windows для Linux (WSL).  | Задайте оболочку по умолчанию в Visual Studio Code либо PowerShell 7 (рекомендуется), либо Windows PowerShell 5,1.|
  

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о разработке функций с помощью PowerShell см. в статье с [руководством разработчика PowerShell для функций Azure](functions-reference-powershell.md).
