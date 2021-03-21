---
title: Выполнение конвейеров Машинное обучение Azure
description: Узнайте, как выполнять конвейеры Машинное обучение Azure в конвейерах фабрики данных Azure.
ms.service: data-factory
ms.topic: conceptual
ms.author: daperlov
author: djpmsft
ms.date: 07/16/2020
ms.openlocfilehash: db1816baf0ebc476a132b344d8db3cdbdd170e50
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100373935"
---
# <a name="execute-azure-machine-learning-pipelines-in-azure-data-factory"></a>Выполнение конвейеров Машинное обучение Azure в фабрике данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Запустите конвейеры Машинное обучение Azure в качестве шага в конвейерах фабрики данных Azure. Действие Машинное обучение выполнение конвейера позволяет выполнять сценарии прогнозирования пакетной обработки, такие как определение возможных значений по умолчанию для ссуд, определение тональности и анализ шаблонов поведения клиентов.

В приведенном ниже видеоролике представлено представление и демонстрация этой функции в течение шести минут.

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/How-to-execute-Azure-Machine-Learning-service-pipelines-in-Azure-Data-Factory/player]

## <a name="syntax"></a>Синтаксис

```json
{
    "name": "Machine Learning Execute Pipeline",
    "type": "AzureMLExecutePipeline",
    "linkedServiceName": {
        "referenceName": "AzureMLService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "mlPipelineId": "machine learning pipeline ID",
        "experimentName": "experimentName",
        "mlPipelineParameters": {
            "mlParameterName": "mlParameterValue"
        }
    }
}

```

## <a name="type-properties"></a>Свойства типа

Свойство | Описание | Допустимые значения | Обязательно
-------- | ----------- | -------------- | --------
name | Имя действия в конвейере. | Строка | Да
type | Тип действия — "Азуремлексекутепипелине" | Строка | Да
linkedServiceName | Связанная служба для Машинное обучение Azure | Ссылка на связанную службу | Да
млпипелинеид | Идентификатор опубликованного конвейера Машинное обучение Azure | Строка (или выражение с типом результата "строка") | Да
експериментнаме | Имя эксперимента журнала выполнения Машинное обучение выполнения конвейера | Строка (или выражение с типом результата "строка") | Нет
млпипелинепараметерс | Пары "ключ — значение", которые будут переданы в опубликованную конечную точку конвейера Машинное обучение Azure. Ключи должны соответствовать именам параметров конвейера, определенных в опубликованном конвейере Машинное обучение | Объект с парами "ключ-значение" (или выражение с объектом resultType) | Нет
млпарентрунид | Идентификатор выполнения родительского Машинное обучение Azure конвейера | Строка (или выражение с типом результата "строка") | Нет
континуеонстепфаилуре | Следует ли продолжать выполнение других шагов в Машинное обучение выполнении конвейера в случае сбоя шага | Логическое | Нет

> [!NOTE]
> Чтобы заполнить раскрывающиеся элементы в Машинное обучение имя конвейера и идентификатор, пользователь должен иметь разрешение на перечисление конвейеров ML. UI ADF вызывает API Азуремлсервице напрямую, используя учетные данные пользователя, вошедшего в систему.  

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими ссылками, в которых описаны способы преобразования данных другими способами:

* [Выполнение действия потока данных](control-flow-execute-data-flow-activity.md)
* [Действие U-SQL](transform-data-using-data-lake-analytics.md)
* [Действие Hive](transform-data-using-hadoop-hive.md)
* [Действие Pig](transform-data-using-hadoop-pig.md)
* [Действие MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Действие потоковой передачи Hadoop](transform-data-using-hadoop-streaming.md)
* [Действие Spark](transform-data-using-spark.md)
* [Настраиваемое действие .NET](transform-data-using-dotnet-custom-activity.md)
* [Действие хранимой процедуры](transform-data-using-stored-procedure.md)
