---
title: Руководство по Прогнозирование намерений — LUIS
description: В этом руководстве показано, как создать пользовательское приложение, которое прогнозирует пользовательские намерения на основе речевых фрагментов (текста).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 05/05/2020
ms.openlocfilehash: 04ba7dc79deef2dbad12c3f65a324201d3e9f598
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102612768"
---
# <a name="tutorial-build-a-luis-app-to-determine-user-intentions"></a>Руководство по созданию приложения LUIS для определения намерений пользователя

В этом руководстве вы создадите пользовательское приложение, которое прогнозирует пользовательские намерения на основе речевых фрагментов (текста).

**В этом руководстве рассмотрено, как выполнять следующие задачи.**

> [!div class="checklist"]
> * Создание нового приложения
> * Создание намерений
> * Добавление примеров высказываний
> * Обучать приложения
> * Публикация приложения
> * Получение прогноза намерений из конечной точки

## <a name="user-intentions-as-intents"></a>Намерения пользователей

Приложение предназначено для определения намерения беседы на естественном языке.

`I'd like to order a veggie pizza with a salad on the side.`

Эти **Намерения** подразделяются на категории.

|Блокировка с намерением|Назначение|
|--|--|
|`ModifyOrder`|Определение порядка заказа пиццы пользователем.|
|`Greeting`|Начало диалога с ботом.|
|`ConfirmOrder`|Подтверждение заказа пиццы.|
|`None`|Определение того, спрашивает ли пользователь что-либо, на что приложение LUIS не должно отвечать. Это намерение предоставляется как часть создания приложения и не может быть удалено. |

## <a name="create-a-new-app"></a>Создание нового приложения

[!INCLUDE [Follow these steps to create a new LUIS app](includes/create-pizza-app.md)]

## <a name="create-a-new-intent"></a>Создание намерения

Намерение используется для классификации речевых фрагментов пользователей на основе намерения пользователя, определенного на основе текста естественного языка.

Чтобы классифицировать речевой фрагмент, намерение требует примеры речевых фрагментов пользователей, которые следует классифицировать с помощью этого намерения.

1. В разделе **Сборка** на странице **Намерения** выберите **+ Создать**, чтобы создать новое намерение. Введите новое имя намерения `OrderPizza` и нажмите кнопку **Готово**.

    Намерение `OrderPizza` прогнозируется, когда пользователь хочет заказать пиццу.

1. Добавьте в намерение несколько примеров высказываний, которые ожидаются от пользователя.

    |`OrderPizza` Примеры высказываний|
    |--|
    |`can i get a pepperoni pizza and a can of coke please`|
    |`can i get a small pizza with onions peppers and olives`|
    |`delivery for a small pepperoni pizza`|
    |`pickup a cheddar cheese pizza large with extra anchovies`|
    |`i need 2 large cheese pizzas 6 large pepperoni pizzas and 1 large supreme pizza`|
    |`Order a pizza for me`|

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана: добавление примера речевых фрагментов на странице "Намерения" на портале LUIS](media/tutorial-intents-only/add-example-utterances-for-pizza-order.png)

    Предоставляя _примеры речевых фрагментов_, вы обучаете службу LUIS прогнозированию определенного типа высказываний для этого намерения. Это положительные примеры. Речевые фрагменты во всех других намерениях обрабатываются как отрицательные примеры для этого намерения.

    [!INCLUDE [Do not use too few utterances](includes/do-not-use-too-few-utterances.md)]

## <a name="create-remaining-intents"></a>Создание оставшихся намерений

1. Создайте намерение `Greeting` и добавьте следующие примеры высказываний. Это делается для того, чтобы определить, начинает ли пользователь новую беседу о заказе пиццы.

    |`Greeting` Примеры высказываний|
    |--|
    |`Hi`|
    |`Hello`|
    |`Hey`|
    |`Start`|
    |`Begin`|

1. Создайте намерение `Confirm` и добавьте следующие примеры высказываний. Это делается для того, чтобы определить, был ли выполнен заказ и согласен ли пользователь с деталями заказа.

    |`Confirm` Примеры высказываний|
    |--|
    |`Go ahead`|
    |`ok`|
    |`Yes`|
    |`Sure`|


## <a name="none-intent-example-utterances"></a>Примеры речевых фрагментов намерения None

[!INCLUDE [Follow these steps to add the None intent to the app](includes/add-example-utterances-none-intent.md)]

## <a name="train-the-app"></a>Обучение приложения

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

## <a name="publish-the-app"></a>Публикация приложения

[!INCLUDE [LUIS How to Publish steps](includes/howto-publish.md)]

## <a name="get-intent-prediction"></a>Получение прогноза намерения

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. Перейдите в конец URL-адреса в адресной строке и введите:

    `get a medium vegetarian pizza for delivery`

  
    `query` — это параметр строки запроса. Это **запрос** фразы, который передается в универсальном коде ресурса (URI). Это высказывание не похоже ни на один из примеров высказываний. Это хороший способ проверить, может ли LUIS обучаться и прогнозировать, что намерение `OrderPizza` является намерением с наивысшей оценкой.

    ```JSON
    {
        "query": "get a medium vegetarian pizza for delivery",
        "prediction": {
            "topIntent": "OrderPizza",
            "intents": {
                "OrderPizza": {
                    "score": 0.6488959
                },
                "None": {
                    "score": 0.139966831
                },
                "Confirm": {
                    "score": 0.00736504374
                },
                "Greeting": {
                    "score": 0.003970454
                }
            },
            "entities": {}
        }
    }
    ```

    Массив сущностей пуст, потому что в настоящее время у этого приложения нет никаких сущностей (единица данных в извлекаемом речевом фрагменте).

    Результат JSON определяет намерение с наивысшим показателем как свойство **`prediction.topIntent`** . Все оценки находятся в диапазоне 0–1, наилучшие оценки — те, которые ближе к 1.

1. Измените параметр URL **запроса** , чтобы он полагался на намерение **Приветствие**:

    `Howdy`

    Это не совсем то же самое, что пример речевого фрагмента, поэтому следует проверить, сможет ли LUIS узнать, что следует предсказать с помощью этого намерения.

    ```json
    {
        "query": "howdy",
        "prediction": {
            "topIntent": "Greeting",
            "intents": {
                "Greeting": {
                    "score": 0.446016937
                },
                "Confirm": {
                    "score": 0.2390079
                },
                "None": {
                    "score": 0.09119555
                },
                "OrderPizza": {
                    "score": 0.00109590159
                }
            },
            "entities": {}
        }
    }
    ```

    Этот прогноз имеет оценку достоверности в 44 %. Чтобы увеличить показатель достоверности, добавьте от 15 до 30 примеров речевых фрагментов.

## <a name="client-application-next-steps"></a>Дальнейшие действия клиентского приложения

В этом руководстве создано приложение LUIS и намерения, добавлены примеры речевых фрагментов для каждого намерения, в том числе, для намерения "None", которые обучены, опубликованы и протестированы на конечной точке. Ниже приведены основные этапы создания модели LUIS.

Выполнив запрос, приложение LUIS возвращает ответ JSON. LUIS не предоставляет ответы на пользовательские высказывания, а только определяет, какой тип информации запрашивается на естественном языке. Дальнейшие строки при общении предоставляются клиентским приложением, таким как служба Azure Bot.


[!INCLUDE [LUIS How to clean up resources](includes/quickstart-tutorial-cleanup-resources.md)]

## <a name="related-information"></a>Дополнительные сведения

* [Типы сущностей](luis-concept-entity-types.md)
* [Обучение активной версии приложения LUIS](luis-how-to-train.md)
* [Как опубликовать предложение](luis-how-to-publish-app.md)
* [Тестирование приложения LUIS на портале LUIS](luis-interactive-test.md)
* [Служба Azure Bot](/azure/bot-service/)


## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление делимой сущности в это приложение](tutorial-machine-learned-entity.md)