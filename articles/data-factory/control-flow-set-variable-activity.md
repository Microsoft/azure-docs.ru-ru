---
title: Установка действия переменной в фабрике данных Azure
description: Узнайте, как использовать установку действия переменной, чтобы задать значение существующей переменной, определенной в конвейере Фабрики данных
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/07/2020
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.openlocfilehash: 122a0a01c420d5efa12fa267a0d3605fc7a25960
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100385342"
---
# <a name="set-variable-activity-in-azure-data-factory"></a>Установка действия переменной в фабрике данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Используйте установку действия переменной, чтобы задать значение существующей переменной с типом "Строка", "Bool" или "Массив", определенной в конвейере Фабрики данных.

## <a name="type-properties"></a>Свойства типа

Свойство | Описание | Обязательно
-------- | ----------- | --------
name | Имя действия в конвейере | да
description | Текст, описывающий действия | нет
type | Необходимо задать значение **SetVariable** | да
value | Строковый литерал или значение объекта выражения, которому назначена переменная | да
variableName | Имя переменной, которая задается этим действием | да

## <a name="incrementing-a-variable"></a>Увеличение переменной

Стандартный сценарий использования переменных в фабрике данных Azure заключается в использовании переменной в качестве итератора в действии until или foreach. В действии задания переменной невозможно ссылаться на переменную, задаваемую в поле `value`. Чтобы обойти это ограничение, задайте временную переменную, а затем создайте второе действие задания переменной. Второе действие задания переменной задает в качестве значения итератора временную переменную. 

Ниже приводится пример такого подхода.

![Увеличение переменной](media/control-flow-set-variable-activity/increment-variable.png "Увеличение переменной")

``` json
{
    "name": "pipeline3",
    "properties": {
        "activities": [
            {
                "name": "Set I",
                "type": "SetVariable",
                "dependsOn": [
                    {
                        "activity": "Increment J",
                        "dependencyConditions": [
                            "Succeeded"
                        ]
                    }
                ],
                "userProperties": [],
                "typeProperties": {
                    "variableName": "i",
                    "value": {
                        "value": "@variables('j')",
                        "type": "Expression"
                    }
                }
            },
            {
                "name": "Increment J",
                "type": "SetVariable",
                "dependsOn": [],
                "userProperties": [],
                "typeProperties": {
                    "variableName": "j",
                    "value": {
                        "value": "@string(add(int(variables('i')), 1))",
                        "type": "Expression"
                    }
                }
            }
        ],
        "variables": {
            "i": {
                "type": "String",
                "defaultValue": "0"
            },
            "j": {
                "type": "String",
                "defaultValue": "0"
            }
        },
        "annotations": []
    }
}
```


## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о деятельности связанного потока управления, который поддерживается Фабрикой данных: 

- [Добавление действия переменной](control-flow-append-variable-activity.md)
