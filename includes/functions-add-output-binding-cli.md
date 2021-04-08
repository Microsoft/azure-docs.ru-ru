---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 7ef3bd0f401ba54d56ed42df34cd2e761681dbc7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96904079"
---
::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"

## <a name="add-an-output-binding-definition-to-the-function"></a>Добавление определения выходной привязки к функции

У функции может быть только один триггер, но у нее может быть несколько входных и выходных привязок, которые позволяют подключаться к другим службам и ресурсам Azure без написания пользовательского кода интеграции. 
::: zone-end
::: zone pivot="programming-language-python,programming-language-javascript,programming-language-powershell,programming-language-typescript"  
Эти привязки объявляются в файле *function.json* в папке функции. В предыдущем кратком руководстве файл *function.json* в папке *HttpExample* содержит две привязки в коллекции `bindings`:  
::: zone-end

::: zone pivot="programming-language-javascript,programming-language-typescript"  
:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-JavaScript/function.json" range="2-18":::  
::: zone-end

::: zone pivot="programming-language-python"  
:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json" range="2-18":::  
::: zone-end

::: zone pivot="programming-language-powershell"  
:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-PowerShell/function.json" range="2-18":::
::: zone-end  

::: zone pivot="programming-language-python,programming-language-javascript, programming-language-powershell, programming-language-typescript"  
У каждой привязки есть по крайней мере такие параметры, как тип, направление и имя. В примере выше у первой привязки тип `httpTrigger` и направление `in`. Для направления `in` `name` указывает имя входного параметра, который отправляется в функцию, когда ее вызывает триггер.  
::: zone-end

::: zone pivot="programming-language-javascript,programming-language-typescript"  
Вторая привязка в коллекции называется `res`. Эта привязка `http` является выходной привязкой (`out`), которая используется для записи HTTP-ответа. 

Чтобы записать в очередь службы хранилища Azure данные из этой функции, добавьте привязку `out` типа `queue` с именем `msg`, как показано в коде ниже:

:::code language="json" source="~/functions-docs-javascript/functions-add-output-binding-storage-queue-cli/HttpExample/function.json" range="3-26":::
::: zone-end  

::: zone pivot="programming-language-python"  
Вторая привязка в коллекции относится к типу `http` с направлением `out`. В этом случае специальный параметр `name` `$return` указывает, что эта привязка использует возвращаемое значение функции, а не предоставляет входной параметр.

Чтобы записать в очередь службы хранилища Azure данные из этой функции, добавьте привязку `out` типа `queue` с именем `msg`, как показано в коде ниже:

:::code language="json" source="~/functions-docs-python/functions-add-output-binding-storage-queue-cli/HttpExample/function.json" range="3-26":::
::: zone-end  

::: zone pivot="programming-language-powershell"  
Вторая привязка в коллекции называется `res`. Эта привязка `http` является выходной привязкой (`out`), которая используется для записи HTTP-ответа. 

Чтобы записать в очередь службы хранилища Azure данные из этой функции, добавьте привязку `out` типа `queue` с именем `msg`, как показано в коде ниже:

:::code language="json" source="~/functions-docs-powershell/functions-add-output-binding-storage-queue-cli/HttpExample/function.json" range="3-26":::
::: zone-end  

::: zone pivot="programming-language-python,programming-language-javascript,programming-language-powershell,programming-language-typescript"  
В этом случае `msg` передается функции в качестве выходного аргумента. Для типа `queue` необходимо также указать имя очереди в `queueName` и указать *имя* подключения к службе хранилища Azure (из *local.settings.json*) в `connection`. 
::: zone-end  
