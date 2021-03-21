---
title: Способы отключения функций в решении "Функции Azure"
description: Сведения о том, как отключать и включать функции в решении "Функции Azure".
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 1ad484804f66a2e2d4d0f1da4a37cf0d6c485f38
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104584743"
---
# <a name="how-to-disable-functions-in-azure-functions"></a>Способы отключения функций в решении "Функции Azure"

В этой статье объясняется, как отключить функцию в решении "Функции Azure". *Отключить* функцию означает сделать так, чтобы среда выполнения игнорировала автоматический триггер, который определен для этой функции. Это позволяет запретить выполнение определенной функции без остановки всего приложения-функции.

Рекомендуемым способом отключения функции является параметр приложения в формате `AzureWebJobs.<FUNCTION_NAME>.Disabled` `true` . Вы можете создать и изменить этот параметр приложения несколькими способами, в том числе с помощью [Azure CLI](/cli/azure/) и вкладки **обзор** функции в [портал Azure](https://portal.azure.com). 

> [!NOTE]  
> Если вы отключите функцию, активируемую по HTTP, с помощью описанных в этой статье способов, ее конечная точка может остаться доступной, если она работает на локальном компьютере.  

## <a name="disable-a-function"></a>Отключение функции

# <a name="portal"></a>[Портал](#tab/portal)

Используйте кнопки **включить** и **Отключить** на странице **Обзор** функции. Эти кнопки работают, изменяя значение `AzureWebJobs.<FUNCTION_NAME>.Disabled` параметра приложения. Этот параметр, зависящий от функции, создается при первом отключении. 

![Переключатель состояния функции](media/disable-function/function-state-switch.png)

Даже при публикации в приложении-функции из локального проекта вы все еще можете использовать портал для отключения функций в приложении-функции. 

> [!NOTE]  
> Компоненты тестирования, интегрированные с порталом, игнорируют параметр `Disabled`. Это означает, что отключенная функция будет работать обычным образом, когда она запускается из окна **Тест** на портале. 

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

В Azure CLI команда [`az functionapp config appsettings set`](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-set) позволяет создать или изменить параметр приложения. Следующая команда отключает функцию с именем `QueueTrigger` , создавая параметр приложения с именем `AzureWebJobs.QueueTrigger.Disabled` и присвоив ему значение `true` . 

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings AzureWebJobs.QueueTrigger.Disabled=true
```

Чтобы снова включить эту функцию, повторите ту же команду со значением `false`.

```azurecli-interactive
az functionapp config appsettings set --name <myFunctionApp> \
--resource-group <myResourceGroup> \
--settings AzureWebJobs.QueueTrigger.Disabled=false
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

[`Update-AzFunctionAppSetting`](/powershell/module/az.functions/update-azfunctionappsetting)Команда добавляет или обновляет параметр приложения. Следующая команда отключает функцию с именем `QueueTrigger` , создавая параметр приложения с именем `AzureWebJobs.QueueTrigger.Disabled` и присвоив ему значение `true` . 

```azurepowershell-interactive
Update-AzFunctionAppSetting -Name <FUNCTION_APP_NAME> -ResourceGroupName <RESOURCE_GROUP_NAME> -AppSetting @{"AzureWebJobs.QueueTrigger.Disabled" = "true"}
```

Чтобы снова включить эту функцию, повторите ту же команду со значением `false`.

```azurepowershell-interactive
Update-AzFunctionAppSetting -Name <FUNCTION_APP_NAME> -ResourceGroupName <RESOURCE_GROUP_NAME> -AppSetting @{"AzureWebJobs.QueueTrigger.Disabled" = "false"}
```
---

## <a name="functions-in-a-slot"></a>Функции в слоте

По умолчанию параметры приложения применяются и к приложениям, выполняемым в слотах развертывания. Однако можно переопределить параметр приложения, используемый слотом, задав параметр приложения для конкретного слота. Например, может потребоваться, чтобы функция была активной в рабочей среде, но не во время тестирования развертывания, например функция, активируемая по таймеру. 

Отключение функции только в промежуточном слоте:

# <a name="portal"></a>[Портал](#tab/portal)

Перейдите к экземпляру слота приложения функции, выбрав слоты **развертывания** в разделе **развертывание**, выбрав слот и выбрав **функции** в экземпляре слота.  Выберите свою функцию, а затем используйте кнопки **включить** и **Отключить** на странице **обзора** функции. Эти кнопки работают, изменяя значение `AzureWebJobs.<FUNCTION_NAME>.Disabled` параметра приложения. Этот параметр, зависящий от функции, создается при первом отключении. 

Можно также напрямую добавить параметр приложения `AzureWebJobs.<FUNCTION_NAME>.Disabled` с именем со значением `true` в **конфигурации** для экземпляра слота. При добавлении параметра приложения для конкретного слота обязательно установите флажок **параметр слота развертывания** . При этом значение параметра будет поддерживаться слотом во время замены.

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> --slot <SLOT_NAME> \
--slot-settings AzureWebJobs.QueueTrigger.Disabled=true
```
Чтобы снова включить эту функцию, повторите ту же команду со значением `false`.

```azurecli-interactive
az functionapp config appsettings set --name <myFunctionApp> \
--resource-group <myResourceGroup> --slot <SLOT_NAME> \
--slot-settings AzureWebJobs.QueueTrigger.Disabled=false
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Azure PowerShell в настоящее время не поддерживает эту функцию.

---

Дополнительные сведения см. в разделе [слоты развертывания функций Azure](functions-deployment-slots.md).

## <a name="localsettingsjson"></a>local.settings.json

Функции можно отключить тем же способом при локальном запуске. Чтобы отключить функцию с именем `HttpExample` , добавьте запись в коллекцию Values в local.settings.jsв файле, как показано ниже.

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsStorage": "UseDevelopmentStorage=true", 
    "AzureWebJobs.HttpExample.Disabled": "true"
  }
}
``` 

## <a name="other-methods"></a>Другие методы

Для всех языков и любых версий среды выполнения мы рекомендуем применять параметр приложения, но существуют и другие способы для отключения функций. Эти способы (зависят от языков и версий среды выполнения) поддерживаются только для обеспечения обратной совместимости. 

### <a name="c-class-libraries"></a>Библиотеки классов C#.

Для функции библиотеки классов вы также можете запретить активацию с помощью атрибута `Disable`. Этот атрибут позволяет настроить имя параметра, используемого для отключения функции. Используйте версию атрибута, которая позволяет определить параметр конструктора, ссылающийся на параметр логического приложения, как показано в следующем примере:

```csharp
public static class QueueFunctions
{
    [Disable("MY_TIMER_DISABLED")]
    [FunctionName("QueueTrigger")]
    public static void QueueTrigger(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        TraceWriter log)
    {
        log.Info($"C# function processed: {myQueueItem}");
    }
}
```

Этот метод позволяет включать и отключать функцию, изменив параметр приложения без повторной компиляции или повторного развертывания. Изменение параметра приложения вызывает перезапуск приложения-функции, поэтому изменение отключенного состоянии сразу же распознается.

Существует также конструктор для параметра, который не принимает строку для имени параметра. Эта версия атрибута не рекомендуется. При использовании этой версии необходимо повторно скомпилировать и развернуть проект, чтобы изменить отключенное состояние функции.

### <a name="functions-1x---scripting-languages"></a>Функции 1.x — языки сценариев

В версии 1.x можно использовать свойство `disabled` файла *function.json*, чтобы запретить среде выполнения активировать функцию. Этот метод работает только для скриптовых языков, например в скриптах на C# и JavaScript. Свойству `disabled` можно задать значение `true` или имя параметра приложения:

```json
{
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ],
    "disabled": true
}
```
или диспетчер конфигурации служб 

```json
    "bindings": [
        ...
    ],
    "disabled": "IS_DISABLED"
```

Во втором примере функция отключается, если имеется параметр приложения с именем IS_DISABLED и ему задано значение `true` или 1.

>[!IMPORTANT]  
>На портале для отключения функций v1. x используются параметры приложения. Если параметр приложения конфликтует с файлом function.json, может возникнуть ошибка. Чтобы избежать ошибок, следует удалить свойство `disabled` из файла function.json. 


## <a name="next-steps"></a>Дальнейшие действия

Эта статья посвящена отключению автоматических триггеров. Дополнительные сведения о триггерах см. в статье [Основные понятия триггеров и привязок в Функциях Azure](functions-triggers-bindings.md).
