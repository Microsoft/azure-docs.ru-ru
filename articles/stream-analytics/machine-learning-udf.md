---
title: Интеграция Azure Stream Analytics со службой "Машинное обучение Azure"
description: В этой статье описывается, как интегрировать задание Azure Stream Analytics с Машинным обучением Azure.
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/21/2020
ms.custom: devx-track-js
ms.openlocfilehash: c35d5d2f63f4a7abe80a0ff19e5994013355c386
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98020525"
---
# <a name="integrate-azure-stream-analytics-with-azure-machine-learning-preview"></a>Интеграция Azure Stream Analytics со службой "Машинное обучение Azure" (предварительная версия)

Модели машинного обучения можно реализовывать в качестве определяемой пользователем функции (UDF) в заданиях Azure Stream Analytics для оценки и прогнозирования входных данных потоковой передачи в реальном времени. [Машинное обучение Azure](../machine-learning/overview-what-is-azure-ml.md) позволяет использовать любое популярное средство с открытым кодом, например Tensorflow, scikit-Training или PyTorch, для подготовки, обучения и развертывания моделей.

## <a name="prerequisites"></a>Предварительные требования

Перед добавлением модели машинного обучения в качестве функции в задание Stream Analytics выполните следующие действия.

1. [Разверните модель как веб-службу](../machine-learning/how-to-deploy-and-where.md) с помощью Машинного обучения Azure.

2. В сценарии оценки должен содержаться [образец входных и выходных данных](../machine-learning/how-to-deploy-and-where.md), который используется Машинным обучением Azure для создания спецификации схемы. Stream Analytics использует эту схему для понимания сигнатуры функции веб-службы. Вы можете использовать этот [Пример определения Swagger](https://github.com/Azure/azure-stream-analytics/blob/master/Samples/AzureML/swagger-example.json) в качестве ссылки, чтобы убедиться, что она правильно настроена.

3. Убедитесь, что веб-служба принимает и возвращает сериализованные данные JSON.

4. Для крупномасштабных развертываний в рабочей среде подходит [Служба Azure Kubernetes](../machine-learning/how-to-deploy-and-where.md#choose-a-compute-target), разверните свою модель в ней. Если веб-служба не справляется с количеством запросов, поступивших от задания, производительность задания Stream Analytics будет снижена, что влияет на задержку. Модели, развернутые в Экземплярах контейнеров Azure, поддерживаются только при использовании портала Azure. Модели, созданные с помощью [конструктора машинное обучение Azure](../machine-learning/concept-designer.md) , пока не поддерживаются в Stream Analytics.

## <a name="add-a-machine-learning-model-to-your-job"></a>Добавление модели машинного обучения к заданию

Вы можете добавить функции Машинное обучение Azure в задание Stream Analytics непосредственно из портал Azure или Visual Studio Code.

### <a name="azure-portal"></a>Портал Azure

1. Перейдите к заданию Stream Analytics в портале Azure и выберите **Функции** в разделе **Топология задания**. Затем в раскрывающемся меню **Добавить** выберите пункт **машинное обучение Azure служба** .

   ![Добавление Машинное обучение Azure UDF](./media/machine-learning-udf/add-azure-machine-learning-udf.png)

2. Заполните форму **Функция службы машинного обучение Azure** следующими значениями свойств:

   ![Настройка Машинное обучение Azure UDF](./media/machine-learning-udf/configure-azure-machine-learning-udf.png)

### <a name="visual-studio-code"></a>Visual Studio Code

1. Откройте проект Stream Analytics в Visual Studio Code и щелкните правой кнопкой мыши папку **функции** . Затем нажмите кнопку **Добавить функцию**. Выберите **машинное обучение UDF** из раскрывающегося списка.

   :::image type="content" source="media/machine-learning-udf/visual-studio-code-machine-learning-udf-add-function.png" alt-text="Добавление определяемой пользователем функции в VS Code":::

   :::image type="content" source="media/machine-learning-udf/visual-studio-code-machine-learning-udf-add-function-2.png" alt-text="Добавление Машинное обучение Azure UDF в VS Code":::

2. Введите имя функции и заполните параметры в файле конфигурации, используя **SELECT из подписок** в CodeLens.

   :::image type="content" source="media/machine-learning-udf/visual-studio-code-machine-learning-udf-function-name.png" alt-text="Выберите Машинное обучение Azure UDF в VS Code":::

   :::image type="content" source="media/machine-learning-udf/visual-studio-code-machine-learning-udf-configure-settings.png" alt-text="Настройка Машинное обучение Azure UDF в VS Code":::

В следующей таблице описаны все свойства Машинное обучение Azure функций службы в Stream Analytics.

|Свойство.|Описание|
|--------|-----------|
|Псевдоним функции|Введите имя для вызова функции в запросе.|
|Подписка|Ваша подписка Azure.|
|Рабочая область службы "Машинное обучение Azure"|Рабочая область машинного обучения Azure, используемая для развертывания модели как веб-службы.|
|Развернутые приложения|Веб-служба, в которой размещается ваша модель.|
|Сигнатура функции|Сигнатура веб-службы выводится из спецификации схемы API. Если не удается загрузить сигнатуру, убедитесь, что вы предоставили пример входных и выходных данных в сценарии оценки для автоматического создания схемы.|
|Число параллельных запросов на секцию|Это параметр расширенной конфигурации для оптимизации пропускной способности при большом масштабе. Это число представляет параллельные запросы, отправляемые из каждой секции задания в веб-службу. Задания с шестью единицами потоковой передачи (SU) и ниже имеют одну секцию. Задания с 12 SU имеют две секции, 18 — три секции и т. д.<br><br> Например, если в задании две секции и этот параметр установлен на 4, то в веб-службе будет содержаться восемь одновременных запросов от задания. Сейчас в общедоступной предварительной версии это значение по умолчанию равно 20 и его нельзя изменять.|
|Максимальное количество пакетов|Это параметр расширенной конфигурации для оптимизации пропускной способности при большом масштабе. Это число представляет максимальное число событий, сгруппированных в одном запросе, отправленном в веб-службу.|

## <a name="supported-input-parameters"></a>Поддерживаемые входные параметры

Когда запрос Stream Analytics вызывает определяемую пользователем функцию Машинного обучения Azure UDF, задание создает сериализованный запрос JSON к веб-службе. Запрос основан на схеме, зависящей от модели. Для [автоматического создания схемы](../machine-learning/how-to-deploy-and-where.md) необходимо предоставить пример входных и выходных данных в сценарии оценки. Схема позволяет Stream Analytics создавать сериализованный запрос JSON для любого из поддерживаемых типов данных, таких как NumPy, Pandas и PySpark. Несколько входных событий можно сгруппировать в одном запросе.

Следующий запрос Stream Analytics является примером вызова определяемой пользователем функции Машинного обучения Azure:

```SQL
SELECT udf.score(<model-specific-data-structure>)
INTO output
FROM input
```

Stream Analytics поддерживает передачу только одного параметра для функций Машинного обучения Azure. Возможно, потребуется подготовить данные перед передачей их в качестве входных данных в определяемую пользователем функцию Машинного обучения Azure. Необходимо убедиться, что входные данные для пользовательской UDF не равны NULL, так как входные значения NULL приведут к сбою задания.

## <a name="pass-multiple-input-parameters-to-the-udf"></a>Передача нескольких входных параметров в определяемую пользователем функцию

Наиболее распространенными примерами входных данных для моделей машинного обучения являются DataFrame и массивы NumPy. Массив можно создать с помощью определяемой пользователем функции JavaScript, а сериализованный в формате JSON DataFrame — с помощью предложения `WITH`.

### <a name="create-an-input-array"></a>Создание входного массива

Вы можете создать определяемую пользователем функцию JavaScript, которая принимает *N* входов и создает массив, который можно использовать в качестве входных данных для определяемой пользователем функции Машинного обучения Azure.

```javascript
function createArray(vendorid, weekday, pickuphour, passenger, distance) {
    'use strict';
    var array = [vendorid, weekday, pickuphour, passenger, distance]
    return array;
}
```

После добавления определяемой пользователем функции JavaScript в задание определяемую пользователем функцию Машинного обучение Azure можно вызвать с помощью следующего запроса:

```SQL
WITH 
ModelInput AS (
#use JavaScript UDF to construct array that will be used as input to ML UDF
SELECT udf.createArray(vendorid, weekday, pickuphour, passenger, distance) as inputArray
FROM input
)

SELECT udf.score(inputArray)
INTO output
FROM ModelInput
#validate inputArray is not null before passing it to ML UDF to prevent job from failing
WHERE inputArray is not null
```

Следующий JSON является примером запроса:

```JSON
{
    "data": [
        ["1","Mon","12","1","5.8"],
        ["2","Wed","10","2","10"]
    ]
}
```

### <a name="create-a-pandas-or-pyspark-dataframe"></a>Создание DataFrame Pandas или PySpark

Можно использовать предложение `WITH` для создания сериализованного в формате JSON DataFrame, который может быть передан в качестве входных данных для определяемой пользователем функции Машинного обучения Azure, как показано ниже.

Следующий запрос создает DataFrame, выбирая необходимые поля и используя DataFrame в качестве входных данных для функции Машинного обучения Azure.

```SQL
WITH 
Dataframe AS (
SELECT vendorid, weekday, pickuphour, passenger, distance
FROM input
)

SELECT udf.score(Dataframe)
INTO output
FROM Dataframe
```

Следующий JSON является примером запроса из состава предыдущего запроса:

```JSON
{
    "data": [{
            "vendorid": "1",
            "weekday": "Mon",
            "pickuphour": "12",
            "passenger": "1",
            "distance": "5.8"
        }, {
            "vendorid": "2",
            "weekday": "Tue",
            "pickuphour": "10",
            "passenger": "2",
            "distance": "10"
        }
    ]
}
```

## <a name="optimize-the-performance-for-azure-machine-learning-udfs"></a>Оптимизация производительности для определяемой пользователем функции Машинного обучения Azure

При развертывании модели в службе Kubernetes Azure можно [профилировать модель, чтобы определить использование ресурсов](../machine-learning/how-to-deploy-profile-model.md). Кроме того, вы можете [включить Application Insights для развертываний](../machine-learning/how-to-enable-app-insights.md), чтобы узнать их частоту запросов, время отклика и частоту сбоев.

При наличии сценария с высокой пропускной способностью событий для достижения оптимальной производительности с низкой задержкой может быть необходимо изменить следующие параметры в Stream Analytics.

1. Максимальное количество пакетов.
2. Число параллельных запросов на секцию.

### <a name="determine-the-right-batch-size"></a>Определение правильного размера пакета

После развертывания веб-службы вы отправляете пример запроса с различными размерами пакетов, начиная с 50 и увеличивая его на сотни. Например, 200, 500, 1000, 2000 и т. д. Вы заметите, что после определенного размера пакета задержка ответа увеличится. Точка, после которой увеличивается задержка ответа, должна быть максимальным числом пакетов для вашего задания.

### <a name="determine-the-number-of-parallel-requests-per-partition"></a>Определение числа параллельных запросов на секцию.

При оптимальном масштабе задание Stream Analytics должно иметь возможность отправить несколько параллельных запросов в веб-службу и получить ответ в течение нескольких миллисекунд. Задержка ответа веб-службы может напрямую влиять на задержку и производительность задания Stream Analytics. Если вызов из задания к веб-службе занимает много времени, то, скорее всего, будет расти предельная задержка, а также увеличиваться число отложенных событий ввода.

Чтобы предотвратить такую задержку, убедитесь, что кластер Службы Azure Kubernetes (AKS) был подготовлен с [правильным количеством узлов и реплик](../machine-learning/how-to-deploy-azure-kubernetes-service.md#using-the-cli). Очень важно, чтобы веб-служба была высокодоступна и возвращала успешные ответы. Если задание получает ответ о недоступности службы (503) от веб-службы, оно будет постоянно повторять попытки с экспоненциальным ухудшением показателей. Любой ответ, отличный от успеха (200) и недоступности службы (503), приведет к переходу задания в состояние сбоя.

## <a name="next-steps"></a>Дальнейшие действия

* [Руководство. Определяемые пользователем функции JavaScript в Azure Stream Analytics](stream-analytics-javascript-user-defined-functions.md)
* [Масштабирование заданий Stream Analytics с помощью функции Студии машинного обучения Azure (классической)](stream-analytics-scale-with-machine-learning-functions.md)
