---
title: Выбор целевых версий среды выполнения Функций Azure
description: Решение "Функции Azure" поддерживает разные версии среды выполнения. Узнайте, как указать версию среды выполнения приложения-функции, размещенного в Azure.
ms.topic: conceptual
ms.date: 07/22/2020
ms.openlocfilehash: e9aa5546b5f07b724fe22bc1e20a2e97feb2aec2
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102435568"
---
# <a name="how-to-target-azure-functions-runtime-versions"></a>Выбор целевых версий среды выполнения Функций Azure

Приложение-функция работает в определенной версии среды выполнения для решения "Функции Azure". Существует три основных версии: [3. x, 2. x и 1. x](functions-versions.md). По умолчанию приложения функций создаются в версии 3. x среды выполнения. Из этой статьи вы узнаете, как настроить приложение-функцию Azure для выполнения в выбранной версии. Сведения о настройке локальной среды разработки для определенной версии см. в разделе [Как программировать и тестировать функции Azure в локальной среде](functions-run-local.md).

Способ назначения конкретной версии вручную зависит от того, используете ли вы Windows или Linux.

## <a name="automatic-and-manual-version-updates"></a>Обновление версий автоматически и вручную

_Этот раздел не применяется при запуске приложения функции [в Linux](#manual-version-updates-on-linux)._

Функции Azure позволяют ориентироваться на конкретную версию среды выполнения в Windows с помощью `FUNCTIONS_EXTENSION_VERSION` параметра приложения в приложении-функции. Приложение-функция будет работать в указанной основной версии, пока вы явно не переместите его в новую. Если указать только основной номер версии, приложение-функция автоматически обновится до новых дополнительных версий среды выполнения, когда они станут доступны. Новые дополнительные версии не должны вносить критические изменения. 

При указании дополнительного номера версии (например, "2.0.12345") приложение-функция будет закреплено за этой версией, пока вы явно не укажете другую. Более старые промежуточные версии регулярно удаляются из рабочей среды. Если дополнительный номер версии удаляется, приложение функции возвращается к работе в последней версии вместо версии, заданной в `FUNCTIONS_EXTENSION_VERSION` . Таким образом, следует быстро устранить все проблемы, связанные с приложением-функцией, для которого требуется определенная дополнительная версия. Затем можно вернуться к основной версии. Дополнительные удаления версий объявляются в [объявлениях службы приложений](https://github.com/Azure/app-service-announcements/issues).

> [!NOTE]
> Если вы запишете на конкретную основную версию функций Azure и попытаетесь опубликовать в Azure с помощью Visual Studio, откроется диалоговое окно с предложением выполнить обновление до последней версии или отменить публикацию. Чтобы избежать этого, добавьте `<DisableFunctionExtensionVersionUpdate>true</DisableFunctionExtensionVersionUpdate>` в `.csproj` файл свойство.

Если новая версия общедоступна, выполнить обновление до нее можно при помощи запроса на портале. Перейдя на новую версию, вы всегда сможете вернуться к предыдущей с помощью параметра приложения `FUNCTIONS_EXTENSION_VERSION`.

В следующей таблице показаны `FUNCTIONS_EXTENSION_VERSION` значения для каждой основной версии, позволяющие включить автоматическое обновление.

| Основной номер версии | Значение `FUNCTIONS_EXTENSION_VERSION` |
| ------------- | ----------------------------------- |
| 3.x  | `~3` |
| 2.x  | `~2` |
| 1.x  | `~1` |

Изменение версии среды выполнения сопровождается перезапуском приложения-функции.

>[!NOTE]
>Приложения функции .NET закреплены, чтобы `~2.0` отказаться от автоматического обновления до .NET Core 3,1. Дополнительные сведения см. в статье [рекомендации по функциям версии 2. x](functions-dotnet-class-library.md#functions-v2x-considerations).  

## <a name="view-and-update-the-current-runtime-version"></a>Просмотр и обновление текущей версии среды выполнения

_Этот раздел не применяется при запуске приложения функции [в Linux](#manual-version-updates-on-linux)._

Вы можете изменить версию среды выполнения, используемую приложением функции. Из-за возможных критических изменений можно изменить только версию среды выполнения до создания функций в приложении-функции. 

> [!IMPORTANT]
> Несмотря на то, что версия среды выполнения определяется `FUNCTIONS_EXTENSION_VERSION` параметром, это изменение следует вносить только в портал Azure, а не напрямую путем изменения параметра. Это обусловлено тем, что на портале изменения проверяются и при необходимости вносятся другие связанные с ними изменения.

# <a name="portal"></a>[Портал](#tab/portal)

[!INCLUDE [Set the runtime version in the portal](../../includes/functions-view-update-version-portal.md)]

> [!NOTE]
> С помощью портал Azure нельзя изменить версию среды выполнения для приложения-функции, которое уже содержит функции.

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

Параметр `FUNCTIONS_EXTENSION_VERSION` также можно просмотреть и задать с помощью интерфейса командной строки Azure.  

В Azure CLI просмотрите текущую версию среды выполнения с помощью команды [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings).

```azurecli-interactive
az functionapp config appsettings list --name <function_app> \
--resource-group <my_resource_group>
```

В этом коде следует заменить `<function_app>` именем приложения-функции. Также замените `<my_resource_group>` именем группы ресурсов для приложения-функции. 

В следующих выходных данных (сокращенных для ясности) есть параметр `FUNCTIONS_EXTENSION_VERSION`:

```output
[
  {
    "name": "FUNCTIONS_EXTENSION_VERSION",
    "slotSetting": false,
    "value": "~2"
  },
  {
    "name": "FUNCTIONS_WORKER_RUNTIME",
    "slotSetting": false,
    "value": "dotnet"
  },
  
  ...
  
  {
    "name": "WEBSITE_NODE_DEFAULT_VERSION",
    "slotSetting": false,
    "value": "8.11.1"
  }
]
```

Обновить параметр `FUNCTIONS_EXTENSION_VERSION` в приложении-функции можно с помощью команды [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings).

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP> \
--resource-group <RESOURCE_GROUP> \
--settings FUNCTIONS_EXTENSION_VERSION=<VERSION>
```

Замените `<FUNCTION_APP>` на имя приложения-функции. Также замените `<RESOURCE_GROUP>` именем группы ресурсов для приложения-функции. Кроме того, замените `<VERSION>` либо определенной версией, либо `~3` , `~2` или `~1` .

Выберите **попробовать** в предыдущем примере кода, чтобы выполнить команду в [Azure Cloud Shell](../cloud-shell/overview.md). Для выполнения этой команды можно также запустить [Azure CLI локально](/cli/azure/install-azure-cli) . При локальном запуске необходимо сначала выполнить команду [AZ login](/cli/azure/reference-index#az-login) для входа.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Чтобы проверить среду выполнения функций Azure, используйте следующий командлет: 

```powershell
Get-AzFunctionAppSetting -Name "<FUNCTION_APP>" -ResourceGroupName "<RESOURCE_GROUP>"
```

Замените `<FUNCTION_APP>` именем приложения функции и `<RESOURCE_GROUP>` . Текущее значение `FUNCTIONS_EXTENSION_VERSION` параметра возвращается в хэш-таблице.

Чтобы изменить среду выполнения функций, используйте следующий скрипт:

```powershell
Update-AzFunctionAppSetting -Name "<FUNCTION_APP>" -ResourceGroupName "<RESOURCE_GROUP>" -AppSetting @{"FUNCTIONS_EXTENSION_VERSION" = "<VERSION>"} -Force
```

Как и ранее, замените `<FUNCTION_APP>` именем приложения функции и `<RESOURCE_GROUP>` именем группы ресурсов. Кроме того, замените на `<VERSION>` конкретную версию или основную версию, например `~2` или `~3` . Вы можете проверить обновленное значение `FUNCTIONS_EXTENSION_VERSION` параметра в возвращенной хэш-таблице. 

---

Приложение-функция перезапускается после внесения изменений в параметр приложения.

## <a name="manual-version-updates-on-linux"></a>Обновление версий вручную в Linux

Чтобы закрепить приложение-функцию Linux для определенной версии узла, укажите URL изображения в поле "Линуксфксверсион" в файле конфигурации сайта. Например, если мы хотим закрепить приложение-функцию 10, чтобы сказать версию узла 3.0.13142-

Для **приложений службы приложений Linux или эластичных** баз данных уровня Premium задайте значение `LinuxFxVersion` `DOCKER|mcr.microsoft.com/azure-functions/node:3.0.13142-node10-appservice` .

Для **приложений "использование Linux** " — значение `LinuxFxVersion` `DOCKER|mcr.microsoft.com/azure-functions/mesh:3.0.13142-node10` .

# <a name="portal"></a>[Портал](#tab/portal)

Просмотр и изменение параметров конфигурации сайта для приложений-функций не поддерживается в портал Azure. Вместо этого используйте Azure CLI.

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

Можно просмотреть и задать с `LinuxFxVersion` помощью Azure CLI.  

Чтобы просмотреть текущую версию среды выполнения, используйте команду [AZ functionapp config Показать](/cli/azure/functionapp/config) .

```azurecli-interactive
az functionapp config show --name <function_app> \
--resource-group <my_resource_group> --query 'linuxFxVersion' -o tsv
```

В этом коде следует заменить `<function_app>` именем приложения-функции. Также замените `<my_resource_group>` именем группы ресурсов для приложения-функции. Возвращается текущее значение `linuxFxVersion` .

Чтобы обновить `linuxFxVersion` параметр в приложении функции, используйте команду [AZ functionapp config Set](/cli/azure/functionapp/config) .

```azurecli-interactive
az functionapp config set --name <FUNCTION_APP> \
--resource-group <RESOURCE_GROUP> \
--linux-fx-version <LINUX_FX_VERSION>
```

Замените `<FUNCTION_APP>` на имя приложения-функции. Также замените `<RESOURCE_GROUP>` именем группы ресурсов для приложения-функции. Кроме того, замените `<LINUX_FX_VERSION>` на значение определенного образа, как описано выше.

Эту команду можно выполнить в [Azure Cloud Shell](../cloud-shell/overview.md), выбрав **Попробовать** в предыдущем примере кода. Также можно использовать [Azure CLI локально](/cli/azure/install-azure-cli) для выполнения этой команды после выполнения команды [az login](/cli/azure/reference-index#az-login) для входа.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В настоящее время невозможно использовать Azure PowerShell для установки `linuxFxVersion` . Вместо этого используйте Azure CLI.

---

После внесения изменений в конфигурацию сайта приложение-функция перезапускается.

> [!NOTE]
> Для приложений, выполняемых в плане потребления, установка `LinuxFxVersion` определенного образа может увеличить время холодного запуска. Это связано с тем, что закрепление в определенном образе не позволяет функциям использовать некоторые оптимизации холодного запуска. 

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Выбор среды выполнения 2.0 в локальной среде разработки](functions-run-local.md)

> [!div class="nextstepaction"]
> [См. заметки о выпуске для версий среды выполнения](https://github.com/Azure/azure-webjobs-sdk-script/releases)
