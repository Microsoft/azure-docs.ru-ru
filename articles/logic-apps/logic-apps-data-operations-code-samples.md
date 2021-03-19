---
title: Примеры кода для использования операций с данными
description: Воспользуйтесь примерами кода, показывающими, как можно использовать операции с данными в автоматизированных рабочих процессах, созданных с помощью Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 09/01/2019
ms.openlocfilehash: c56c31bb6f56efb10808ce25b6b232089391e831
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84712451"
---
# <a name="data-operation-code-samples-for-azure-logic-apps"></a>Примеры кода операций с данными для Azure Logic Apps

Примеры кода для определения действия операций с данными см. в статье [Выполнение операций с данными](../logic-apps/logic-apps-perform-data-operations.md). Можно использовать эти образцы при запуске примеров с определением рабочего процесса собственного приложения логики, подписки Azure и подключения API. Просто скопируйте и вставьте эти определения действия в редактор представления кода для определения рабочего процесса приложения логики, а затем измените определения для конкретного рабочего процесса. 

Исходя из стандартов нотации объектов JavaScript (JSON), эти определения действия отображаются в алфавитном порядке. Однако в конструкторе приложений логики каждое определение внутри рабочего процесса отображается в правильной последовательности, так как свойство `runAfter` каждого определения действия устанавливает порядок выполнения.

<a name="compose-action-example"></a>

## <a name="compose"></a>Docker Compose

Ниже приведены определения действия, которые можно использовать, чтобы попробовать [пример действия **Создать**](../logic-apps/logic-apps-perform-data-operations.md#compose-action).

```json
"actions": {
  "Compose": {
    "type": "Compose",
    "inputs": {
      "age": "@variables('ageVar')",
      "fullName": "@{variables('lastNameVar')}, @{variables('firstNameVar')}"
    },
    "runAfter": {
      "Initialize_variable_-_ageVar": [
          "Succeeded"
      ]
    }
  },
  "Initialize_variable_-_ageVar": {
    "type": "InitializeVariable",
    "inputs": {
      "variables": [
        {
          "name": "ageVar",
          "type": "Integer",
          "value": 35
        }
      ]
    },
    "runAfter": {
      "Initialize_variable_-_lastNameVar": [
        "Succeeded"
      ]
    }
  },
  "Initialize_variable_-_firstNameVar": {
    "type": "InitializeVariable",
    "inputs": {
      "variables": [
        {
          "name": "firstNameVar",
          "type": "String",
          "value": "Sophie "
        }
      ]
    },
    "runAfter": {}
  },
  "Initialize_variable_-_lastNameVar": {
    "type": "InitializeVariable",
    "inputs": {
      "variables": [
        {
          "name": "lastNameVar",
          "type": "String",
          "value": "Owen"
        }
      ]
    },
    "runAfter": {
      "Initialize_variable_-_firstNameVar": [
        "Succeeded"
      ]
    }
  }
},
```

<a name="create-csv-table-action-example"></a>

## <a name="create-csv-table"></a>Создать таблицу CSV

Ниже приведены определения действия, которые можно использовать, чтобы попробовать [пример действия **Создать таблицу CSV**](../logic-apps/logic-apps-perform-data-operations.md#create-csv-table-action).

```json
"actions": {
   "Create_CSV_table": {
      "type": "Table",     
      "inputs": {
         "format": "CSV",
         "from": "@variables('myJSONArray')"
      },
      "runAfter": {
         "Initialize_variable_-_JSON_array": [
            "Succeeded"
         ]
      }
   },
   "Initialize_variable_-_JSON_array": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ 
            {
               "name": "myJSONArray",
               "type": "Array",
                  "value": [
                     {
                        "Description": "Apples",
                        "Product_ID": 1
                     },
                     {
                        "Description": "Oranges",
                        "Product_ID": 2
                     }
                  ]
            }
         ]
      },
      "runAfter": {}
   }
},
```

<a name="create-html-table-action-example"></a>

## <a name="create-html-table"></a>Создать таблицу HTML

Ниже приведены определения действия, которые можно использовать, чтобы попробовать [пример действия **Создать таблицу HTML**](../logic-apps/logic-apps-perform-data-operations.md#create-html-table-action).

```json
"actions": {
   "Create_HTML_table": {
      "type": "Table",     
      "inputs": {
         "format": "HTML",
         "from": "@variables('myJSONArray')"
      },
      "runAfter": {
         "Initialize_variable_-_JSON_array": [
            "Succeeded"
         ]
      }
   },
   "Initialize_variable_-_JSON_array": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ 
            {
               "name": "myJSONArray",
               "type": "Array",
                  "value": [
                     {
                        "Description": "Apples",
                        "Product_ID": 1
                     },
                     {
                        "Description": "Oranges",
                        "Product_ID": 2
                     }
                  ]
            }
         ]
      },
      "runAfter": {}
   }
},
```

<a name="filter-array-action-example"></a>

## <a name="filter-array"></a>Фильтрация массива

Ниже приведены определения действия, которые можно использовать, чтобы попробовать [пример действия **Фильтровать массив**](../logic-apps/logic-apps-perform-data-operations.md#filter-array-action).

```json
"actions": {
   "Filter_array": {
      "type": "Query",
      "inputs": {
         "from": "@variables('myIntegerArray')",
         "where": "@greater(item(), 1)"
      },
      "runAfter": {
         "Initialize_variable_-_integer_array": [
            "Succeeded"
         ]
      }
   },
   "Initialize_variable_-_integer_array": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ 
            {
               "name": "myIntegerArray",
               "type": "Array",
               "value": [
                  1,
                  2,
                  3,
                  4
               ]
            }
         ]
      },
      "runAfter": {}
   }
},
```

<a name="join-action-example"></a>

## <a name="join"></a>Join

Ниже приведены определения действия, которые можно использовать, чтобы попробовать [пример действия **Объединить**](../logic-apps/logic-apps-perform-data-operations.md#join-action).

```json
"actions": {
   "Initialize_variable_-_integer_array": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ 
            {
               "name": "myIntegerArray",
               "type": "Array",
               "value": [
                  1,
                  2,
                  3,
                  4
               ]
            }
         ]
      },
      "runAfter": {}
   },
   "Join": {
      "type": "Join",
      "inputs": {
         "from": "@variables('myIntegerArray')",
         "joinWith": ":"
      },
      "runAfter": {
         "Initialize_variable_-_integer_array": [
             "Succeeded"
         ]
      }
   }
},
```

<a name="parse-json-action-example"></a>

## <a name="parse-json"></a>Parse JSON

Ниже приведены определения действия, которые можно использовать, чтобы попробовать [пример действия **Анализ JSON**](../logic-apps/logic-apps-perform-data-operations.md#parse-json-action).

```json
"actions": {
   "Initialize_variable_-_JSON_object": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [
            {
               "name": "myJSONObject",
               "type": "Object",
               "value": {
                  "Member": {
                     "Email": "Sophie.Owen@contoso.com",
                     "FirstName": "Sophie",
                     "LastName": "Owen"
                  }
               }
            }
         ]
      },
      "runAfter": {}
   },
   "Parse_JSON": {
      "type": "ParseJson",
      "inputs": {
         "content": "@variables('myJSONObject')",
         "schema": {
            "type": "object",
            "properties": {
               "Member": {
                  "type": "object",
                  "properties": {
                     "Email": {
                        "type": "string"
                     },
                     "FirstName": {
                        "type": "string"
                     },
                     "LastName": {
                        "type": "string"
                     }
                  }
               }
            }
         }
      },
      "runAfter": {
         "Initialize_variable_-_JSON_object": [
            "Succeeded"
         ]
      }
},
```

<a name="select-action-example"></a>

## <a name="select"></a>Выберите пункт

Ниже приведены определения действия, которые можно использовать, чтобы попробовать [пример действия **Выбрать пункт**](../logic-apps/logic-apps-perform-data-operations.md#select-action).

```json
"actions": {
   "Initialize_variable_-_integer_array": {
      "type": "InitializeVariable",
      "inputs": {
         "variables": [ 
            {
               "name": "myIntegerArray",
               "type": "Array",
               "value": [
                  1,
                  2,
                  3,
                  4
               ]
            }
         ]
      },
      "runAfter": {}
   },
   "Select": {
      "type": "Select",
      "inputs": {
         "from": "@variables('myIntegerArray')",
         "select": {
            "Product_ID": "@item()"
         }
      },
      "runAfter": {
         "Initialize_variable_-_integer_array": [
           "Succeeded"
         ]
      }
   }
},
```

## <a name="next-steps"></a>Дальнейшие действия

* [Выполнение операций с данными](../logic-apps/logic-apps-perform-data-operations.md)
