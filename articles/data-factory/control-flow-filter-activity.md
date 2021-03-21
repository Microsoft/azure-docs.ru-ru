---
title: Действие Filter в фабрике данных Azure
description: Действие Filter фильтрует входные данные.
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/04/2018
ms.openlocfilehash: 035ee5a5f1fe193fcb4579f1d46472ad9cd0b8b5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100385597"
---
# <a name="filter-activity-in-azure-data-factory"></a>Действие Filter в фабрике данных Azure
Вы можете использовать действие Filter в конвейере для применения выражения фильтра к входному массиву. 
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="syntax"></a>Синтаксис

```json
{
    "name": "MyFilterActivity",
    "type": "filter",
    "typeProperties": {
        "condition": "<condition>",
        "items": "<input array>"
    }
}
```

## <a name="type-properties"></a>Свойства типа

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | --------
name | Имя действия `Filter`. | Строка | Да
type | Должно быть задано значение **filter**. | Строка | Да
condition | Условие, которое можно использовать для фильтрации входных данных. | Expression | Да
items | Входной массив, к которому должен применяться фильтр. | Expression | Да

## <a name="example"></a>Пример

В этом примере конвейер имеет два вида действий: **Filter** и **ForEach**. Действие Filter настроено для фильтрации входного массива на наличие элементов со значением больше 3. Затем действие ForEach выполняет перебор отфильтрованных значений и **присваивает переменной значение** текущему значению.

```json
{
    "name": "PipelineName",
    "properties": {
        "activities": [{
                "name": "MyFilterActivity",
                "type": "filter",
                "typeProperties": {
                    "condition": "@greater(item(),3)",
                    "items": "@pipeline().parameters.inputs"
                }
            },
            {
            "name": "MyForEach",
            "type": "ForEach",
            "dependsOn": [
                {
                    "activity": "MyFilterActivity",
                    "dependencyConditions": [
                        "Succeeded"
                    ]
                }
            ],
            "userProperties": [],
            "typeProperties": {
                "items": {
                    "value": "@activity('MyFilterActivity').output.value",
                    "type": "Expression"
                },
                "isSequential": "false",
                "batchCount": 1,
                "activities": [
                    {
                        "name": "Set Variable1",
                        "type": "SetVariable",
                        "dependsOn": [],
                        "userProperties": [],
                        "typeProperties": {
                            "variableName": "test",
                            "value": {
                                "value": "@string(item())",
                                "type": "Expression"
                            }
                        }
                    }
                ]
            }
        }],
        "parameters": {
            "inputs": {
                "type": "Array",
                "defaultValue": [1, 2, 3, 4, 5, 6]
            }
        },
        "variables": {
            "test": {
                "type": "String"
            }
        },
        "annotations": []
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с другими действиями потока управления, которые поддерживаются фабрикой данных: 

- [Действие условия If](control-flow-if-condition-activity.md)
- [Действие выполнения конвейера](control-flow-execute-pipeline-activity.md)
- [Действие For Each](control-flow-for-each-activity.md)
- [Действие получения метаданных](control-flow-get-metadata-activity.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Веб-действие](control-flow-web-activity.md)
- [Действие Until](control-flow-until-activity.md)
