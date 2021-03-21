---
title: Обновление команды из конечной точки в Интернете
titleSuffix: Azure Cognitive Services
description: Узнайте, как обновить состояние команды с помощью вызова конечной веб-точки.
services: cognitive-services
author: nitinme
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/20/2020
ms.author: nitinme
ms.openlocfilehash: d0b77e6af36f0a71405f6c032bfdd121abeb0071
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97560276"
---
# <a name="update-a-command-from-a-web-endpoint"></a>Обновление команды из конечной точки в Интернете

Если клиентское приложение требует обновления состояния текущей команды без речевого ввода, можно использовать вызов конечной веб-точки для обновления команды.

В этой статье вы узнаете, как обновить текущую команду из конечной веб-точки.

## <a name="prerequisites"></a>Предварительные условия
> [!div class = "checklist"]
> * Предварительно [созданное приложение Пользовательских голосовых команд](quickstart-custom-commands-application.md).

## <a name="create-an-azure-function"></a>Создание функции Azure 

В этом примере вам потребуется [функция Azure](../../azure-functions/index.yml) , активируемая по HTTP, которая поддерживает следующие входные данные (или подмножество этих входных данных):

```JSON
{
  "conversationId": "SomeConversationId",
  "currentCommand": {
    "name": "SomeCommandName",
    "parameters": {
      "SomeParameterName": "SomeParameterValue",
      "SomeOtherParameterName": "SomeOtherParameterValue"
    }
  },
  "currentGlobalParameters": {
      "SomeGlobalParameterName": "SomeGlobalParameterValue",
      "SomeOtherGlobalParameterName": "SomeOtherGlobalParameterValue"
  }
}
```

Давайте рассмотрим ключевые атрибуты этого входа:

| Атрибут | Пояснение |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **conversationId** | Уникальный идентификатор диалога. Обратите внимание, что этот идентификатор может быть создан из клиентского приложения. |
| **курренткомманд** | Команда, которая в данный момент активна в диалоге. |
| **name** | Имя команды. `parameters`Атрибут представляет собой карту с текущими значениями параметров. |
| **куррентглобалпараметерс** | Такое соответствие `parameters` , как, но используется для глобальных параметров. |

Выходные данные функции Azure должны поддерживать следующий формат:

```JSON
{
  "updatedCommand": {
    "name": "SomeCommandName",
    "updatedParameters": {
      "SomeParameterName": "SomeParameterValue"
    },
    "cancel": false
  },
  "updatedGlobalParameters": {
    "SomeGlobalParameterName": "SomeGlobalParameterValue"
  }
}
```

Вы можете распознать этот формат, так как он совпадает с тем, который использовался при [обновлении команды с клиента](./how-to-custom-commands-update-command-from-client.md). 

Теперь создайте функцию Azure на основе Node.js. Скопируйте и вставьте следующий код:

```nodejs
module.exports = async function (context, req) {
    context.log(req.body);
    context.res = {
        body: {
            updatedCommand: {
                name: "IncrementCounter",
                updatedParameters: {
                    Counter: req.body.currentCommand.parameters.Counter + 1
                }
            }
        }
    };
}
```

При вызове этой функции Azure из пользовательских команд вы отправляете текущие значения диалога. Вы вернете параметры, которые необходимо обновить, или хотите отменить текущую команду.

## <a name="update-the-existing-custom-commands-app"></a>Обновление существующего приложения пользовательских команд

Давайте подключим функцию Azure к существующему приложению с пользовательскими командами:

1. Добавьте новую команду с именем `IncrementCounter` .
1. Добавьте только один пример предложения со значением `increment` .
1. Добавьте новый параметр с именем `Counter` (то же имя, как указано в функции Azure) типа `Number` со значением по умолчанию `0` .
1. Добавьте новую веб-конечную точку `IncrementEndpoint` с именем с URL-адресом функции Azure, если для параметра **Удаленные обновления** задано значение **включено**.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/set-web-endpoint-with-remote-updates.png" alt-text="Снимок экрана, на котором показана настройка конечной точки веб-узла с удаленными обновлениями.":::
1. Создайте новое правило взаимодействия с именем **инкрементруле** и добавьте действие **вызов конечной веб-точки** .
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/increment-rule-web-endpoint.png" alt-text="Снимок экрана, на котором показано создание правила взаимодействия.":::
1. В конфигурации действия выберите `IncrementEndpoint` . Настройте **при успешном выполнении** **отправку речевого ответа** со значением `Counter` и настройте **при сбое** сообщение об ошибке.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/set-increment-counter-call-endpoint.png" alt-text="Снимок экрана, показывающий задание счетчика приращения для вызова конечной веб-точки.":::
1. Задайте для правила состояние после выполнения, чтобы **дождаться ввода данных пользователем**.

## <a name="test-it"></a>Тестирование

1. Сохраните и обучите приложение.
1. Выберите **Тест**.
1. Отправить `increment` несколько раз (это пример предложения для `IncrementCounter` команды).
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-commands/increment-counter-example.png" alt-text="Снимок экрана, на котором показан пример счетчика инкремента.":::

Обратите внимание, что функция Azure увеличивает значение `Counter` параметра при каждом включении.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Включение процесса CI/CD для вашего приложения Пользовательских голосовых команд](./how-to-custom-commands-deploy-cicd.md)