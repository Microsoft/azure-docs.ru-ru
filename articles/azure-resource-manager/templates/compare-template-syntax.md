---
title: Синтаксис Compare для шаблонов Azure Resource Manager в JSON и Бицеп
description: Сравнение шаблонов Azure Resource Manager, разработанных с помощью JSON и Бицеп, и показано, как выполнять преобразование между языками.
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: 29c2b9948957ebc10a26f22f0fe3daf383dfe5ba
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102036220"
---
# <a name="comparing-json-and-bicep-for-templates"></a>Сравнение JSON и Бицеп для шаблонов

В этой статье сравнивается синтаксис Бицеп с синтаксисом JSON для шаблонов Azure Resource Manager (шаблоны ARM). В большинстве случаев Бицеп предоставляет синтаксис, который менее подробный, чем эквивалент в JSON.

## <a name="syntax-equivalents"></a>Эквиваленты синтаксиса

Если вы знакомы с использованием JSON для разработки шаблонов ARM, воспользуйтесь следующей таблицей, чтобы узнать о эквивалентном синтаксисе для Бицеп.

| Сценарий | Bicep | JSON |
| -------- | ------------ | ----- |
| Создание выражения | `func()` | `"[func()]"` |
| Получение значения параметра | `exampleParameter` | `[parameters('exampleParameter'))]` |
| Получить значение переменной | `exampleVar` | `[variables('exampleVar'))]` |
| Объединение строк | `'${namePrefix}-vm'` | `[concat(parameters('namePrefix'), '-vm')]` |
| Задание свойства ресурса | `sku: '2016-Datacenter'` | `"sku": "2016-Datacenter",` |
| Возврат логического и | `isMonday && isNovember` | `[and(parameter('isMonday'), parameter('isNovember'))]` |
| Получение идентификатора ресурса в шаблоне | `nic1.id` | `[resourceId('Microsoft.Network/networkInterfaces', variables('nic1Name'))]` |
| Получение свойства из ресурса в шаблоне | `diagsAccount.properties.primaryEndpoints.blob` | `[reference(resourceId('Microsoft.Storage/storageAccounts', variables('diagStorageAccountName'))).primaryEndpoints.blob]` |
| Условно задается значение | `isMonday ? 'valueIfTrue' : 'valueIfFalse'` | `[if(parameters('isMonday'), 'valueIfTrue', 'valueIfFalse')]` |
| Разделение решения на несколько файлов | Использование модулей | Использование связанных шаблонов |
| Задание целевой области развертывания | `targetScope = 'subscription'` | `"$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#"` |
| Задать зависимость | Следует использовать автоматическое обнаружение зависимостей или задать зависимость вручную с помощью `dependsOn: [ stg ]` | `"dependsOn": ["[resourceId('Microsoft.Storage/storageAccounts', 'parameters('storageAccountName'))]"]` |
| Объявление ресурса | `resource vm 'Microsoft.Compute/virtualMachines@2020-06-01' = {...}` | `"resources": [ { "type": "Microsoft.Compute/virtualMachines", "apiVersion": "2020-06-01", ... } ]` |

## <a name="recommendations"></a>Рекомендации

* По возможности избегайте использования функций [Reference](template-functions-resource.md#reference) и [ResourceId](template-functions-resource.md#resourceid) в файле бицеп. При ссылке на ресурс в том же развертывании Бицеп используйте вместо него идентификатор ресурса. Например, если вы развернули ресурс в файле Бицеп с `stg` идентификатором ресурса, используйте синтаксис, например или, `stg.id` `stg.properties.primaryEndpoints.blob` для получения значений свойств. С помощью идентификатора ресурса можно создать неявную зависимость между ресурсами. Не нужно явно задавать зависимость с помощью свойства dependsOn.
* Используйте постоянный регистр для идентификаторов. Если вы не знаете, какой тип регистра используется, попробуйте использовать прописные буквы в стиле Camel. Например, `param myCamelCasedParameter string`.
* Добавьте описание к параметру только в том случае, если описание предоставляет пользователям нужные сведения. `//`Для получения некоторых сведений можно использовать комментарии.

## <a name="decompile-json-to-bicep"></a>Декомпилировать JSON в Бицеп

Интерфейс командной строки Бицеп предоставляет команду для декомпиляции любого существующего шаблона ARM в файл Бицеп. Чтобы декомпилировать JSON-файл, используйте: `bicep decompile "path/to/file.json"`

Эта команда предоставляет отправную точку для создания Бицеп, но команда не работает для всех шаблонов. Команда может завершиться ошибкой, или после декомпиляции может потребоваться устранить проблемы. В настоящее время вложенные шаблоны можно декомпилировать только в том случае, если они используют область вычисления выражения Inner.

Вы можете экспортировать шаблон для группы ресурсов, а затем передать его непосредственно в команду декомпиляции бицеп. В следующем примере показано, как декомпилировать экспортированный шаблон.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group export --name "your_resource_group_name" > main.json
bicep decompile main.json
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Export-AzResourceGroup -ResourceGroupName "your_resource_group_name" -Path ./main.json
bicep decompile main.json
```

# <a name="portal"></a>[Портал](#tab/azure-portal)

[Экспортируйте шаблон](export-template-portal.md) на портале. Используйте `bicep decompile <filename>` для скачанного файла.

---

## <a name="build-json-from-bicep"></a>Сборка JSON из Бицеп

Интерфейс командной строки Бицеп также предоставляет команду для преобразования Бицеп в JSON. Чтобы создать JSON-файл, используйте: `bicep build "path/to/file.json"`

## <a name="side-by-side-view"></a>Параллельное представление

[Бицеп Playground](https://aka.ms/bicepdemo) позволяет просматривать эквивалентные файлы JSON и бицеп параллельно. Можно выбрать пример шаблона для просмотра обеих версий. Или выберите, `Decompile` чтобы отправить собственный шаблон JSON и просмотреть эквивалентный файл бицеп.

## <a name="next-steps"></a>Дальнейшие действия

Сведения о Бицеп см. в разделе [учебник по бицеп](./bicep-tutorial-create-first-bicep.md).
