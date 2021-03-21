---
title: Создание, изменение и расширение определения рабочего процесса JSON приложения логики
description: Как создавать, изменять и расширять определения рабочих процессов JSON приложения логики в Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/01/2018
ms.openlocfilehash: 9163071237041d7c8510a644c573e3763434bb0c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96014403"
---
# <a name="create-edit-or-extend-json-for-logic-app-workflow-definitions-in-azure-logic-apps"></a>Создание, редактирование и расширение JSON для определений рабочего процесса приложений логики в Azure Logic Apps

При создании корпоративных решений по интеграции с автоматизированными рабочими процессами на основе [Azure Logic Apps](../logic-apps/logic-apps-overview.md) для описания и проверки определений используется простой декларативный язык JSON (JavaScript Object Notation) и [схема WDL (языка определения рабочего процесса)](../logic-apps/logic-apps-workflow-definition-language.md). Эти форматы позволяют легко читать и понимать определения приложений логики даже без глубоких знаний о программировании.
Если вы хотите автоматизировать создание и развертывание приложений логики, включите определения приложений логики в [шаблоны Azure Resource Manager](../azure-resource-manager/templates/overview.md) в качестве [ресурсов Azure](../azure-resource-manager/management/overview.md).
Тогда вы сможете создавать приложения логики, управлять ими и развертывать их с помощью [Azure PowerShell](/powershell/module/az.logicapp), [Azure CLI](../azure-resource-manager/templates/deploy-cli.md) или [интерфейсов API REST для Azure Logic Apps](/rest/api/logic/).

Вы можете открыть определения приложений логики в формате JSON, используя редактор представления кода на портале Azure или в Visual Studio, или любой другой текстовый редактор.
Если вы пока не знакомы с приложениями логики, изучите статью [о создании первого приложения логики](../logic-apps/quickstart-create-first-logic-app-workflow.md).

> [!NOTE]
> Некоторые возможности Azure Logic Apps, например определение параметров и множественные триггеры, можно включить в определения только напрямую через JSON, но нельзя использовать в конструкторе Logic Apps.
> Если вам нужны эти функции, используйте представление кода или другой текстовый редактор.

## <a name="edit-json---azure-portal"></a>Редактирование JSON на портале Azure

1. Войдите на <a href="https://portal.azure.com" target="_blank">портал Azure</a>.

2. В меню слева выберите **Все службы**.
В поле поиска введите строку "приложения логики" и выберите нужное приложение в результатах поиска.

3. В меню для этого приложения логики откройте раздел **Средства разработки** и выберите **Представление кода приложения логики**.

   Откроется окно представления кода, в котором вы увидите определения приложения логики в формате JSON.

## <a name="edit-json---visual-studio"></a>Редактирование JSON в Visual Studio

Прежде чем редактировать определения приложений логики в Visual Studio, следует [установить все необходимые средства](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#prerequisites).
Чтобы создать приложение логики в Visual Studio, воспользуйтесь [кратким руководством по автоматизации задач и процессов с помощью Azure Logic Apps](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

Visual Studio позволяет открыть приложения логики, которые были созданы и развернуты напрямую через портал Azure или из Visual Studio в виде проектов Azure Resource Manager.

1. Откройте решение Visual Studio или проект [группы ресурсов Azure](../azure-resource-manager/management/overview.md), который содержит нужное приложение логики.

2. Найдите и откройте определение приложения логики. По умолчанию они содержатся в [шаблоне диспетчера ресурсов](../azure-resource-manager/templates/overview.md) под именем **LogicApp.json**.
Вы можете применять и корректировать этот шаблон для развертывания в разных средах.

3. Откройте контекстное меню для определения и шаблона приложения логики.
Выберите действие **Открыть в конструкторе приложений логики**.

   ![Открытие приложения логики в решении Visual Studio](./media/logic-apps-author-definitions/open-logic-app-designer.png)

   > [!TIP]
   > Если у вас нет этой команды в Visual Studio 2019, убедитесь, что установлены последние обновления для Visual Studio.

4. В нижней части конструктора выберите **Представление кода**.

   Откроется окно представления кода, в котором вы увидите определения приложения логики в формате JSON.

5. Чтобы вернуться в представление конструктора, в нижней части редактора представления кода, выберите **Конструктор**.

## <a name="parameters"></a>Параметры

Жизненный цикл развертывания обычно имеет различные среды для разработки, тестирования, промежуточного хранения и производства. При наличии значений, которые вы хотите повторно использовать в приложении логики без прописано или в зависимости от потребностей развертывания, можно создать [шаблон Azure Resource Manager](../azure-resource-manager/management/overview.md) для определения рабочего процесса, чтобы можно было автоматизировать развертывание приложений логики.

Выполните эти общие действия для *параметризации* или определения и использования параметров для, а не для этих значений. Затем можно указать значения в отдельном файле параметров, который передает эти значения в шаблон. Таким образом, вы сможете легко изменить эти значения без необходимости обновлять и повторно развертывать приложение логики. Дополнительные сведения см. в разделе [Обзор: Автоматизация развертывания приложений логики с помощью шаблонов Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md).

1. В шаблоне определите параметры шаблона и параметры определения рабочего процесса, чтобы принимать значения, которые должны использоваться в развертывании и среде выполнения соответственно.

   Параметры шаблона определяются в разделе параметров, который находится за пределами определения рабочего процесса, а параметры определения рабочего процесса определяются в разделе параметров, который находится внутри определения рабочего процесса.

1. Замените жестко закодированные значения выражениями, которые ссылаются на эти параметры. Выражения шаблона используют синтаксис, отличный от выражений определения рабочего процесса.

   Избегайте усложнения кода, не используя выражения шаблона, которые оцениваются при развертывании, в выражениях определения рабочего процесса, которые оцениваются во время выполнения. Используйте только выражения шаблона за пределами определения рабочего процесса. Используйте только выражения определения рабочего процесса в определении рабочего процесса.

   При указании значений для параметров определения рабочего процесса можно ссылаться на параметры шаблона, используя раздел параметров, который находится за пределами определения рабочего процесса, но по-прежнему находится в определении ресурса для приложения логики. Таким образом, можно передать значения параметров шаблона в параметры определения рабочего процесса.

1. Сохраните значения параметров в отдельном [файле параметров](../azure-resource-manager/templates/parameter-files.md) и включите этот файл в развертывание.

## <a name="process-strings-with-functions"></a>Обработка строк с помощью функций

Logic Apps имеет различные функции для работы со строками.
Например, предположим, что вам требуется передать имя компании из заказа в другую систему.
Однако вы не уверены, как правильно обработать кодировку символов.
К этой строке можно применить кодировку base64, однако чтобы избежать символов экранирования в URL-адресе, можно заменить несколько символов. Кроме того, вам нужна только подстрока для названия компании, так как первые пять символов не используются.

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder1",
        "companyName": "NAME=Contoso"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "request": {
      "type": "Request",
      "kind": "Http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
      }
    }
  },
  "outputs": {}
}
```

Ниже показано, как этот пример обрабатывает строку, работая "наизнанку":

```
"uri": "https://www.example.com/?id=@{replace(replace(base64(substring(parameters('order').companyName,5,sub(length(parameters('order').companyName), 5) )),'+','-') ,'/' ,'_' )}"
```

1. Получите [`length()`](../logic-apps/logic-apps-workflow-definition-language.md) имя компании, чтобы получить общее количество символов.

2. Чтобы получить более короткую строку, вычтите `5`.

3. Теперь получите [`substring()`](../logic-apps/logic-apps-workflow-definition-language.md) .
Начнем с индекса `5` и перейдем к оставшейся части строки.

4. Преобразовать эту подстроку в [`base64()`](../logic-apps/logic-apps-workflow-definition-language.md) строку.

5. Теперь [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) все `+` символы имеют `-` символы.

6. Наконец, [`replace()`](../logic-apps/logic-apps-workflow-definition-language.md) все `/` символы с `_` символами.

## <a name="map-list-items-to-property-values-then-use-maps-as-parameters"></a>Сопоставление элементов списка со значениями свойств с последующим использованием сопоставления в качестве параметра

Чтобы получить различные результаты на основе значения свойства, можно создать схему, сопоставляющую каждое значение свойства с результатом, а затем использовать сопоставление в качестве параметра.

Например, этот рабочий процесс определяет некоторые категории как параметры и схему, которая сопоставляет категории с определенным URL-адресом.
Сначала рабочий процесс получает список статей. Затем он использует схему для поиска соответствия URL-адреса и категории для каждой статьи.

*   [`intersection()`](../logic-apps/logic-apps-workflow-definition-language.md)Функция проверяет, соответствует ли категория известной определенной категории.

*   После получения соответствующей категории пример извлекает элемент из схемы, используя квадратные скобки: `parameters[...]`.

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "specialCategories": {
      "defaultValue": [
        "science",
        "google",
        "microsoft",
        "robots",
        "NSA"
      ],
      "type": "Array"
    },
    "destinationMap": {
      "defaultValue": {
        "science": "https://www.nasa.gov",
        "microsoft": "https://www.microsoft.com/en-us/default.aspx",
        "google": "https://www.google.com",
        "robots": "https://en.wikipedia.org/wiki/Robot",
        "NSA": "https://www.nsa.gov/"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "Request",
      "kind": "http"
    }
  },
  "actions": {
    "getArticles": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://ajax.googleapis.com/ajax/services/feed/load?v=1.0&q=https://feeds.wired.com/wired/index"
      }
    },
    "forEachArticle": {
      "type": "foreach",
      "foreach": "@body('getArticles').responseData.feed.entries",
      "actions": {
        "ifGreater": {
          "type": "if",
          "expression": "@greater(length(intersection(item().categories, parameters('specialCategories'))), 0)",
          "actions": {
            "getSpecialPage": {
              "type": "Http",
              "inputs": {
                "method": "GET",
                "uri": "@parameters('destinationMap')[first(intersection(item().categories, parameters('specialCategories')))]"
              }
            }
          }
        }
      },
      "runAfter": {
        "getArticles": [
          "Succeeded"
        ]
      }
    }
  }
}
```

## <a name="get-data-with-date-functions"></a>Получение данных с помощью функций данных

Чтобы получить данные из источника данных, который не поддерживает *триггеры*, для работы со временем и датами можно использовать функции данных.
Например, это выражение определяет, как долго выполняются шаги этого рабочего процесса, работая "наизнанку":

``` json
"expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
```

1. Из действия `order` извлеките `startTime`.
2. Получите текущее время с помощью `utcNow()`.
3. Вычтите одну секунду:

   [`addseconds(..., -1)`](../logic-apps/logic-apps-workflow-definition-language.md)

   Вы можете использовать другие единицы времени, например `minutes` или `hours`.

3. Теперь вы можете сравнить эти два значения.

   Если первое значение меньше второго, то это значит, что с момента первоначального размещения заказа прошло больше секунды.

Для форматирования дат можно использовать модули форматирования строк. Например, чтобы получить RFC1123, используйте [`utcnow('r')`](../logic-apps/logic-apps-workflow-definition-language.md) .
См. дополнительные сведения о [форматировании даты](../logic-apps/logic-apps-workflow-definition-language.md).

``` json
{
  "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "order": {
      "defaultValue": {
        "quantity": 10,
        "id": "myorder-id"
      },
      "type": "Object"
    }
  },
  "triggers": {
    "Request": {
      "type": "request",
      "kind": "http"
    }
  },
  "actions": {
    "order": {
      "type": "Http",
      "inputs": {
        "method": "GET",
        "uri": "https://www.example.com/?id=@{parameters('order').id}"
      }
    },
    "ifTimingWarning": {
      "type": "If",
      "expression": "@less(actions('order').startTime,addseconds(utcNow(),-1))",
      "actions": {
        "timingWarning": {
          "type": "Http",
          "inputs": {
            "method": "GET",
            "uri": "https://www.example.com/?recordLongOrderTime=@{parameters('order').id}&currentTime=@{utcNow('r')}"
          }
        }
      },
      "runAfter": {
        "order": [
          "Succeeded"
        ]
      }
    }
  },
  "outputs": {}
}
```

## <a name="next-steps"></a>Дальнейшие действия

* [Conditional statements: Run steps based on a condition in logic apps](../logic-apps/logic-apps-control-flow-conditional-statement.md) (Условные операторы. Выполнение шагов на основе условия в приложениях логики)
* [Операторы switch. Выполнение различных шагов на основе различных значений в приложениях логики](../logic-apps/logic-apps-control-flow-switch-statement.md)
* [Loops: Process arrays or repeat actions until a condition is met](../logic-apps/logic-apps-control-flow-loops.md) (Циклы. Обработка массивов или повторение действий до выполнения условия)
* [Create or join parallel branches in your logic app](../logic-apps/logic-apps-control-flow-branches.md) (Создание или присоединение параллельных ветвей в приложении логики)
* [Области. Выполнение шагов на основе состояния группы в приложениях логики](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md)
* [Схема языка определения рабочих процессов в Azure Logic Apps](../logic-apps/logic-apps-workflow-definition-language.md)
* [Действия и триггеры рабочих процессов в Azure Logic Apps](../logic-apps/logic-apps-workflow-actions-triggers.md)
