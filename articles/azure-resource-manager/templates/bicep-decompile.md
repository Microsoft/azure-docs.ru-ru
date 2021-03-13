---
title: Преобразование шаблонов между JSON и Бицеп
description: Описывает команды для преобразования шаблонов Azure Resource Manager из Бицеп в JSON и из JSON в Бицеп.
ms.topic: conceptual
ms.date: 03/12/2021
ms.openlocfilehash: 6d242f5846996cd0f5b9510a1a2b9f2bf063a0c7
ms.sourcegitcommit: df1930c9fa3d8f6592f812c42ec611043e817b3b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/13/2021
ms.locfileid: "103422228"
---
# <a name="converting-arm-templates-between-json-and-bicep"></a>Преобразование шаблонов ARM между JSON и Бицеп

В этой статье описывается, как преобразовать шаблоны Azure Resource Manager (шаблоны ARM) из JSON в Бицеп и из Бицеп в JSON.

Для выполнения команд преобразования необходимо [установить CLI бицеп](bicep-install.md) .

Команды преобразования формируют шаблоны, которые функционально эквивалентны. Однако в реализации они могут не совпадать. Преобразование шаблона из JSON в Бицеп, а затем обратно в JSON, вероятно, приводит к подлинности шаблона с синтаксическим синтаксисом, отличным от исходного шаблона. При развертывании преобразованные шаблоны дают те же результаты.

## <a name="convert-from-json-to-bicep"></a>Преобразование из JSON в Бицеп

Интерфейс командной строки Бицеп предоставляет команду для декомпиляции любого существующего шаблона JSON в файл Бицеп. Чтобы декомпилировать JSON-файл, используйте:

```azurecli
bicep decompile mainTemplate.json
```

Эта команда предоставляет отправную точку для создания Бицеп. Команда не работает для всех шаблонов. В настоящее время вложенные шаблоны можно декомпилировать только в том случае, если они используют область вычисления выражения Inner. Шаблоны, использующие циклы копирования, не могут быть декомпилированы.

## <a name="convert-from-bicep-to-json"></a>Преобразование из Бицеп в JSON

Интерфейс командной строки Бицеп также предоставляет команду для преобразования Бицеп в JSON. Чтобы создать JSON-файл, используйте:

```azurecli
bicep build mainTemplate.bicep
```

## <a name="export-template-and-convert"></a>Экспорт шаблона и преобразование

Вы можете экспортировать шаблон для группы ресурсов, а затем передать его непосредственно в команду декомпиляции. В следующем примере показано, как декомпилировать экспортированный шаблон.

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

[Экспортируйте шаблон](export-template-portal.md) на портале.

Используйте `bicep decompile <filename>` для скачанного файла.

---

## <a name="side-by-side-view"></a>Параллельное представление

[Бицеп Playground](https://aka.ms/bicepdemo) позволяет просматривать эквивалентные файлы JSON и бицеп параллельно. Можно выбрать пример шаблона для просмотра обеих версий. Или выберите, `Decompile` чтобы отправить собственный шаблон JSON и просмотреть эквивалентный файл бицеп.

## <a name="next-steps"></a>Дальнейшие действия

Сведения о Бицеп см. в разделе [учебник по бицеп](./bicep-tutorial-create-first-bicep.md).
