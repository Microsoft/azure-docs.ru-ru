---
title: Как интегрировать общую схему предупреждений с Logic Apps
description: Узнайте, как создать приложение логики, использующее общую схему оповещений для обработки всех ваших оповещений.
ms.topic: conceptual
ms.date: 05/27/2019
ms.openlocfilehash: 459ac43a6260e046befbe58d3531bac494903d51
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102038345"
---
# <a name="how-to-integrate-the-common-alert-schema-with-logic-apps"></a>Как интегрировать общую схему предупреждений с Logic Apps

В этой статье показано, как создать приложение логики, использующее общую схему оповещений для обработки всех ваших оповещений.

## <a name="overview"></a>Обзор

[Общая схема предупреждений](./alerts-common-schema.md) предоставляет стандартизованную и расширяемую схему JSON для всех различных типов оповещений. Общая схема предупреждений наиболее полезна при программном использовании — через веб-перехватчики, модули Runbook и приложения логики. В этой статье мы продемонстрируем, как можно создать одно приложение логики для обработки всех ваших оповещений. Те же принципы могут применяться и к другим программным методам. Приложение логики, описываемое в этой статье, создает четко определенные переменные для [полей "важный"](alerts-common-schema-definitions.md#essentials), а также описывает способ обработки логики определенного [типа оповещений](alerts-common-schema-definitions.md#alert-context) .


## <a name="prerequisites"></a>Предварительные требования 

В этой статье предполагается, что читатель знаком с 
* Настройка правил оповещений ([Метрика](../alerts/alerts-metric.md), [Журнал](./alerts-log.md), [Журнал действий](./alerts-activity-log.md))
* Настройка [групп действий](./action-groups.md)
* Включение [общей схемы предупреждений](./alerts-common-schema.md#how-do-i-enable-the-common-alert-schema) из групп действий

## <a name="create-a-logic-app-leveraging-the-common-alert-schema"></a>Создание приложения логики, использующего общую схему предупреждений

1. Выполните действия, описанные [в статье Создание приложения логики](./action-groups-logic-app.md). 

1.  Выберите триггер **При получении HTTP-запроса**.

    ![Триггеры приложений логики](media/action-groups-logic-app/logic-app-triggers.png "Триггеры приложений логики")

1.  Выберите **Изменить**, чтобы изменить триггер HTTP-запроса.

    ![Триггеры HTTP-запросов](media/action-groups-logic-app/http-request-trigger-shape.png "Триггеры HTTP-запросов")


1.  Скопируйте и вставьте следующую схему:

    ```json
        {
            "type": "object",
            "properties": {
                "schemaId": {
                    "type": "string"
                },
                "data": {
                    "type": "object",
                    "properties": {
                        "essentials": {
                            "type": "object",
                            "properties": {
                                "alertId": {
                                    "type": "string"
                                },
                                "alertRule": {
                                    "type": "string"
                                },
                                "severity": {
                                    "type": "string"
                                },
                                "signalType": {
                                    "type": "string"
                                },
                                "monitorCondition": {
                                    "type": "string"
                                },
                                "monitoringService": {
                                    "type": "string"
                                },
                                "alertTargetIDs": {
                                    "type": "array",
                                    "items": {
                                        "type": "string"
                                    }
                                },
                                "originAlertId": {
                                    "type": "string"
                                },
                                "firedDateTime": {
                                    "type": "string"
                                },
                                "resolvedDateTime": {
                                    "type": "string"
                                },
                                "description": {
                                    "type": "string"
                                },
                                "essentialsVersion": {
                                    "type": "string"
                                },
                                "alertContextVersion": {
                                    "type": "string"
                                }
                            }
                        },
                        "alertContext": {
                            "type": "object",
                            "properties": {}
                        }
                    }
                }
            }
        }
    ```

1. Выберите **+** **новый шаг** и нажмите кнопку **Добавить действие**.

    ![Добавление действия](media/action-groups-logic-app/add-action.png "Добавление действия")

1. На этом этапе вы можете добавить различные соединители (Microsoft Teams, временной резерв, Salesforce и т. д.) в соответствии с конкретными бизнес-требованиями. Вы можете использовать встроенные поля "все". 

    ![Ключевые поля](media/alerts-common-schema-integrations/logic-app-essential-fields.png "Ключевые поля")
    
    Кроме того, условную логику можно создать на основе типа оповещения, используя параметр "выражение".

    ![Выражение приложения логики](media/alerts-common-schema-integrations/logic-app-expressions.png "Выражение приложения логики")
    
     [Поле "мониторингсервице"](alerts-common-schema-definitions.md#alert-context) позволяет уникальным образом идентифицировать тип оповещения, на основе которого можно создать условную логику.

    
    Например, приведенный ниже фрагмент кода проверяет, является ли предупреждение оповещением журнала на основе Application Insights, и, если да, выводит результаты поиска. В противном случае выводится «НД».

    ```text
      if(equals(triggerBody()?['data']?['essentials']?['monitoringService'],'Application Insights'),triggerBody()?['data']?['alertContext']?['SearchResults'],'NA')
    ```
    
     Дополнительные сведения о [написании выражений приложения логики](../../logic-apps/workflow-definition-language-functions-reference.md#logical-comparison-functions).

    


## <a name="next-steps"></a>Следующие шаги

* [Узнайте о группах действий.](./action-groups.md)
* Дополнительные [сведения о схеме общих предупреждений](./alerts-common-schema.md).