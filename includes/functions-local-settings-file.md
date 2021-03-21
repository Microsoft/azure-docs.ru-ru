---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 04/14/2019
ms.author: glenga
ms.openlocfilehash: d944d1d3e9c72471fab2435430a7d13e1770e807
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96010488"
---
## <a name="local-settings-file"></a>Файл с локальными параметрами

В локальном файле local.settings.json хранятся параметры приложения, строки подключения и параметры для локальных средств разработки. Параметры в файле local.settings.json используются только при локальном выполнении проектов. Файл локальных параметров имеет следующую структуру:

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "<language worker>",
    "AzureWebJobsStorage": "<connection-string>",
    "AzureWebJobsDashboard": "<connection-string>",
    "MyBindingConnection": "<binding-connection-string>",
    "AzureWebJobs.HttpExample.Disabled": "true"
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*",
    "CORSCredentials": false
  },
  "ConnectionStrings": {
    "SQLConnectionString": "<sqlclient-connection-string>"
  }
}
```

При локальном выполнении проектов поддерживаются следующие параметры:

| Параметр      | Описание                            |
| ------------ | -------------------------------------- |
| **`IsEncrypted`** | Если этот параметр имеет значение `true`, все значения шифруются с помощью ключа локального компьютера. Используется с командами `func settings`. Значение по умолчанию — `false`. Возможно, вам потребуется зашифровать файл local.settings.json на локальном компьютере, если он содержит секреты, например строки подключения службы. Узел автоматически расшифровывает параметры при запуске. Перед чтением локально зашифрованных параметров используйте команду `func settings decrypt`. |
| **`Values`** | Массив параметров приложения и строк подключения, используемых при локальном выполнении проекта. Эти пары "ключ — значение" (строковые выражения) соответствуют параметрам приложения-функции в Azure, например [`AzureWebJobsStorage`]. Многие триггеры и привязки имеют свойство, относящееся к строке подключения параметра приложения, например `Connection` для [триггера хранилища BLOB-объектов](../articles/azure-functions/functions-bindings-storage-blob-trigger.md#configuration). Для таких свойств требуется определить параметр приложения в массиве `Values`. Список часто используемых параметров см. в следующей таблице. <br/>Значения должны быть строками, а не объектами JSON или массивами. Имена параметров не могут содержать двоеточие (`:`) или двойное подчеркивание (`__`). Символы двойного подчеркивания зарезервированы средой выполнения, а двоеточие зарезервировано для поддержки [внедрения зависимостей](../articles/azure-functions/functions-dotnet-dependency-injection.md#working-with-options-and-settings). |
| **`Host`** | Параметры в этом разделе служат для настройки хост-процесса Функций при выполнении проектов в локальной среде. Эти параметры отделены от параметров host.json, которые также применяются при выполнении проектов в Azure. |
| **`LocalHttpPort`** | Задает порт по умолчанию, используемый при выполнении локального узла Функций (`func host start` и `func run`). Аргумент командной строки `--port` имеет приоритет над этим параметром. |
| **`CORS`** | Определяет источники, для которых разрешен [общий доступ к ресурсам независимо от источника (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing). Источники указываются в виде разделенного запятыми списка без пробелов. Допускается подстановочное значение (\*), разрешающее запросы из любого источника. |
| **`CORSCredentials`** |  Если задано значение `true`, разрешает запросы `withCredentials`. |
| **`ConnectionStrings`** | Коллекция. Не применяйте эту коллекцию для строк подключения, используемых привязками функций. Эта коллекция используется только платформами, которые обычно получают строки подключений из раздела `ConnectionStrings` в файле конфигурации, как например [Entity Framework](/ef/ef6/). Строки подключения, содержащиеся в этом объекте, добавляются в среду с типом поставщика [System.Data.SqlClient](/dotnet/api/system.data.sqlclient). Элементы этой коллекции не публикуются в Azure с другими параметрами приложения. Необходимо явным образом добавить эти значения в коллекцию `Connection strings` в настройках приложения-функции. Если вы создаете параметр [`SqlConnection`](/dotnet/api/system.data.sqlclient.sqlconnection) в коде функции, следует сохранить значение строки подключения на портале в **Параметрах приложения**, вместе с другими подключениями. |

Следующие параметры приложения можно включать в массив **`Values`** при локальном запуске.

| Параметр | Значения | Описание |
|-----|-----|-----|
|**`AzureWebJobsStorage`**| Строка подключения учетной записи хранения или<br/>`UseDevelopmentStorage=true`| Содержит строку подключения для учетной записи хранения Azure Требуется при использовании триггеров, отличающихся от HTTP. См. справочник по [`AzureWebJobsStorage`].<br/>При наличии локального [эмулятора службы хранилища Azure](../articles/storage/common/storage-use-emulator.md) вы можете задать для [`AzureWebJobsStorage`] значение `UseDevelopmentStorage=true`, чтобы в Core Tools использовался эмулятор. Эмулятор удобно использовать во время разработки, но перед развертыванием следует проверить работу с реальным подключением к хранилищу.| 
|**`AzureWebJobs.<FUNCTION_NAME>.Disabled`**| `true`\|`false` | Чтобы отключить функцию при локальном выполнении, добавьте в коллекцию `"AzureWebJobs.<FUNCTION_NAME>.Disabled": "true"`, где `<FUNCTION_NAME>` — имя функции. См. сведения о том, как [отключать и включать функции в Функциях Azure](../articles/azure-functions/disable-function.md#localsettingsjson). |
|**`FUNCTIONS_WORKER_RUNTIME`** | `dotnet`<br/>`node`<br/>`java`<br/>`powershell`<br/>`python`| Указывает целевой язык среды выполнения Функций. Требуется для среды выполнения Функций версии 2.x и выше. Этот параметр создается для проекта с помощью Core Tools. См. справочник по [`FUNCTIONS_WORKER_RUNTIME`](../articles/azure-functions/functions-app-settings.md#functions_worker_runtime).|
| **`FUNCTIONS_WORKER_RUNTIME_VERSION`** | `~7` |Указывает, что PowerShell 7 будет использоваться при локальном запуске. Если параметр не задан, используется PowerShell Core 6. Этот параметр используется только при локальном запуске. При запуске в Azure версия среды выполнения PowerShell определяется параметром конфигурации сайта `powerShellVersion`, который можно [настроить на портале](../articles/azure-functions/functions-reference-powershell.md#changing-the-powershell-version). | 

[`AzureWebJobsStorage`]: ../articles/azure-functions/functions-app-settings.md#azurewebjobsstorage