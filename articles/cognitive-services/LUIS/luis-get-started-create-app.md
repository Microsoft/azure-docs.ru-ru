---
title: Краткое руководство. Создание приложения — LUIS
description: В этом кратком руководстве описано, как создать приложение LUIS, которое использует предварительно созданную предметную область `HomeAutomation` для включения и отключения освещения и устройств. Эта предварительно созданная предметная область предоставляет для использования намерения, сущности и фразы. Когда все будет готово, вы получите работающую в облаке конечную точку LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: quickstart
ms.date: 03/29/2021
ms.openlocfilehash: e0972998fad647c1c70be814cb7c13f14a6e1141
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106277034"
---
# <a name="quickstart-use-prebuilt-home-automation-app"></a>Краткое руководство. Использование предварительно созданного приложения для системы домашней автоматики

В этом кратком руководстве описано, как создать приложение LUIS, которое использует предварительно созданную предметную область `HomeAutomation` для включения и отключения освещения и устройств. Эта предварительно созданная предметная область предоставляет для использования намерения, сущности и фразы. Когда все будет готово, вы получите работающую в облаке конечную точку LUIS.

[!INCLUDE [Sign in to LUIS](./includes/sign-in-process.md)]

[!INCLUDE [Select authoring resource](./includes/select-authoring-resource.md)]

## <a name="create-a-new-app"></a>Создание нового приложения
Создавать и администрировать приложения можно на странице **Мои приложения**.

### <a name="create-an-application"></a>Создание приложения

Чтобы создать приложение, щелкните **+ Создать приложение**. 

В появившемся окне введите следующую информацию:

|Имя  |Описание  |
|---------|---------|
|AName     | Имя приложения, например "home automation".        |
|culture     | Язык, поддерживаемый приложением.   |
|Описание | Описание приложения.
|Ресурс для прогнозирования | Ресурс для прогнозирования, который будет принимать запросы. |

Нажмите кнопку **Готово**.

>[!NOTE]
>Язык и региональные параметры нельзя изменить после создания приложения.

## <a name="add-prebuilt-domain"></a>Добавление готовой предметной области

1. В области навигации слева выберите **Предварительно созданные предметные области**.
1. Выполните поиск по фразе **HomeAutomation**.
1. Нажмите кнопку **Add domain** (Добавить предметную область) на карточке HomeAutomation.

    > [!div class="mx-imgBorder"]
    > ![Выберите Prebuilt domains (Предварительно созданные предметные области) и найдите HomeAutomation. Нажмите кнопку Add domain (Добавить предметную область) на карточке HomeAutomation.](media/luis-quickstart-new-app/home-automation.png)

    Когда предметная область будет успешно добавлена, в области предварительно созданных предметных областей отобразится кнопка **Remove domain** (Удалить предметную область).

## <a name="intents-and-entities"></a>Намерения и сущности

1. Выберите **Intents** (Намерения) в области навигации слева, чтобы просмотреть намерения из домена HomeAutomation. Он включает примеры речевых фрагментов, например `HomeAutomation.QueryState` и `HomeAutomation.SetDevice`.

    > [!NOTE]
    > Намерение **None** (Нет) стандартно предоставляется всеми приложениями LUIS. Оно используется для обработки фраз, которые не соответствуют функциям приложения.

1. Выберите намерение **HomeAutomation.TurnOff**. Намерение содержит список речевых фрагментов с обозначенными сущностями.

    > [!div class="mx-imgBorder"]
    > [![Снимок экрана с намерением HomeAutomation.TurnOff](media/luis-quickstart-new-app/home-automation-turnoff.png "Снимок экрана с намерением HomeAutomation.TurnOff")](media/luis-quickstart-new-app/home-automation-turnoff.png)

1. Если вы хотите просмотреть сущности в приложении, выберите **Entities** (Сущности). Если щелкнуть одну из сущностей, например **HomeAutomation.DeviceName**, отобразится список связанных с ней значений. 
 
    :::image type="content" source="media/luis-quickstart-new-app/entities-page.png" alt-text="Замещающий текст изображения" lightbox="media/luis-quickstart-new-app/entities-page.png":::

## <a name="train-the-luis-app"></a>Обучение приложения LUIS

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

## <a name="test-your-app"></a>Тестирование приложения
После обучения можно выполнить тестирование приложения.

1. Выберите **Тест** в правом верхнем меню навигации.

1. Введите тестовый речевой фрагмент, например `Turn off the lights`, в области интерактивного тестирования и нажмите клавишу ВВОД. Например, *Turn off the lights*.

    В нашем примере `Turn off the lights` правильно распознано как намерение с высокой оценкой **HomeAutomation.TurnOff**.

    ![Снимок экрана с панелью тестирования, на которой выделено намерение](media/luis-quickstart-new-app/review-test-inspection-pane-in-portal.png)

1. Выберите **Проверить**, чтобы просмотреть дополнительные сведения о прогнозировании.

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана: панель тестирования со сведениями о проверке](media/luis-quickstart-new-app/test.png)

1. Закройте область тестирования.

<a name="publish-your-app"></a>

## <a name="publish-the-app-to-get-the-endpoint-url"></a>Публикация приложения для получения URL-адреса конечной точки

Чтобы получать результаты прогнозирования LUIS в чат-боте или другом приложении, необходимо опубликовать приложение в конечной точке прогнозирования.

1. Выберите **Опубликовать** в правом верхнем углу окна.

1. Выберите слот **Рабочий** и нажмите кнопку **Готово**.

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана: публикация в конечной точке с помощью LUIS](media/howto-publish/publish-app-popup.png)

1. Перейдите по ссылке **Получить доступ к URL-адресам конечных точек** в уведомлении, чтобы открыть страницу **Ресурсы Azure**. URL-адреса конечных точек перечислены в **примере запроса**.

<!-- [!INCLUDE [LUIS How to Publish steps](./includes/howto-publish.md)] -->

<a name="query-the-v2-api-prediction-endpoint"></a>

## <a name="query-the-v3-api-prediction-endpoint"></a>Запрос конечной точки прогнозирования API V3

[!INCLUDE [LUIS How to get endpoint first step](./includes/v3-prediction-endpoint.md)]

2. В адресной строке браузера для строки запроса проверьте, указаны ли в URL-адресе следующие строки имени и значения. Если они отсутствуют в строке запроса, добавьте их:

    |Пара "имя — значение"|
    |--|
    |`verbose=true`|
    |`show-all-intents=true`|

3. В адресной строке браузера перейдите в конец URL-адреса и введите `turn off the living room light` для значения _запроса_, а затем нажмите клавишу ВВОД.

    ```json
    {
        "query": "turn off the living room light",
        "prediction": {
            "topIntent": "HomeAutomation.TurnOff",
            "intents": {
                "HomeAutomation.TurnOff": {
                    "score": 0.969448864
                },
                "HomeAutomation.QueryState": {
                    "score": 0.0122336326
                },
                "HomeAutomation.TurnUp": {
                    "score": 0.006547436
                },
                "HomeAutomation.TurnDown": {
                    "score": 0.0050634006
                },
                "HomeAutomation.SetDevice": {
                    "score": 0.004951761
                },
                "HomeAutomation.TurnOn": {
                    "score": 0.00312553928
                },
                "None": {
                    "score": 0.000552945654
                }
            },
            "entities": {
                "HomeAutomation.Location": [
                    "living room"
                ],
                "HomeAutomation.DeviceName": [
                    [
                        "living room light"
                    ]
                ],
                "HomeAutomation.DeviceType": [
                    [
                        "light"
                    ]
                ],
                "$instance": {
                    "HomeAutomation.Location": [
                        {
                            "type": "HomeAutomation.Location",
                            "text": "living room",
                            "startIndex": 13,
                            "length": 11,
                            "score": 0.902181149,
                            "modelTypeId": 1,
                            "modelType": "Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "HomeAutomation.DeviceName": [
                        {
                            "type": "HomeAutomation.DeviceName",
                            "text": "living room light",
                            "startIndex": 13,
                            "length": 17,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "HomeAutomation.DeviceType": [
                        {
                            "type": "HomeAutomation.DeviceType",
                            "text": "light",
                            "startIndex": 25,
                            "length": 5,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ]
                }
            }
        }
    }
    ```

    Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).


## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Итеративная разработка приложений для LUIS](./luis-concept-app-iteration.md)