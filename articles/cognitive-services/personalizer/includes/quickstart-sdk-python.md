---
title: включить файл
description: включить файл
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: include
ms.custom: cog-serv-seo-aug-2020
ms.date: 08/25/2020
ms.openlocfilehash: 0298edc069ee1f61bd3ccd437914c9b01e083900
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102445658"
---
[Справочная документация](/python/api/azure-cognitiveservices-personalizer/azure.cognitiveservices.personalizer) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-personalizer) | [Пакет (pypi)](https://pypi.org/project/azure-cognitiveservices-personalizer/) | [Примеры](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/Personalizer)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [Python 3.x](https://www.python.org/)
* Получив подписку Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer"  title="Создание ресурса Персонализатора"  target="_blank">создайте ресурс Персонализатора </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Персонализатора потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

[!INCLUDE [Change model frequency](change-model-frequency.md)]

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

После установки Python вы можете установить клиентскую библиотеку с помощью следующей команды:

```console
pip install azure-cognitiveservices-personalizer
```

### <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте файл Python и переменные для конечной точки ресурса и ключа подписки.

[!INCLUDE [Personalizer find resource info](find-azure-resource-info.md)]

```python
from azure.cognitiveservices.personalizer import PersonalizerClient
from azure.cognitiveservices.personalizer.models import RankableAction, RewardRequest, RankRequest
from msrest.authentication import CognitiveServicesCredentials

import datetime, json, os, time, uuid

key = "<paste-your-personalizer-key-here>"
endpoint = "<paste-your-personalizer-endpoint-here>"
```

## <a name="object-model"></a>Объектная модель

Клиент Персонализатора — это объект [PersonalizerClient](/python/api/azure-cognitiveservices-personalizer/azure.cognitiveservices.personalizer.personalizer_client.personalizerclient), который проходит проверку подлинности в Azure с помощью объекта Microsoft.Rest.ServiceClientCredentials, который содержит ваш ключ.

Чтобы запросить единственный лучший элемент содержимого, создайте [RankRequest](/python/api/azure-cognitiveservices-personalizer/azure.cognitiveservices.personalizer.models.rankrequest),а затем передайте его в метод client.Rank. Метод ранжирования возвращает RankResponse.

Чтобы отправить оценку вознаграждения в Персонализатор, задайте идентификатор события и оценку вознаграждения (значение), которые будут отправлены в метод [Reward](/python/api/azure-cognitiveservices-personalizer/azure.cognitiveservices.personalizer.operations.events_operations.eventsoperations#reward-event-id--value--custom-headers-none--raw-false----operation-config-) в классе EventOperations.

Определение вознаграждения в этом кратком руководстве тривиально. Со временем определение факторов влияния на [оценку вознаграждения](../concept-rewards.md) и сложность процесса в рабочей системе может измениться. Это должно быть одним из основных проектных решений в вашей архитектуре Персонализатора.

## <a name="code-examples"></a>Примеры кода

Фрагменты кода по приведенным ниже ссылкам показывают, как выполнить одноименные действия с помощью клиентской библиотеки Персонализатора для Python:

* [аутентификация клиента](#authenticate-the-client);
* [API ранжирования](#request-the-best-action)
* [API вознаграждения](#send-a-reward)

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр `PersonalizerClient` с `key` и `endpoint`, которые вы создали ранее.

```python
# Instantiate a Personalizer client
client = PersonalizerClient(endpoint, CognitiveServicesCredentials(key))
```

## <a name="get-content-choices-represented-as-actions"></a>Получение вариантов содержимого, представленных в виде действий

Действия представляют варианты содержимого, из которых Персонализатор должен выбрать лучший элемент содержимого. Добавьте следующие методы в класс Program, чтобы представить набор действий и их признаки.

```python
def get_actions():
    action1 = RankableAction(id='pasta', features=[{"taste":"salty", "spice_level":"medium"},{"nutrition_level":5,"cuisine":"italian"}])
    action2 = RankableAction(id='ice cream', features=[{"taste":"sweet", "spice_level":"none"}, { "nutritional_level": 2 }])
    action3 = RankableAction(id='juice', features=[{"taste":"sweet", 'spice_level':'none'}, {'nutritional_level': 5}, {'drink':True}])
    action4 = RankableAction(id='salad', features=[{'taste':'salty', 'spice_level':'none'},{'nutritional_level': 2}])
    return [action1, action2, action3, action4]
```

```python
def get_user_timeofday():
    res={}
    time_features = ["morning", "afternoon", "evening", "night"]
    time = input("What time of day is it (enter number)? 1. morning 2. afternoon 3. evening 4. night\n")
    try:
        ptime = int(time)
        if(ptime<=0 or ptime>len(time_features)):
            raise IndexError
        res['time_of_day'] = time_features[ptime-1]
    except (ValueError, IndexError):
        print("Entered value is invalid. Setting feature value to", time_features[0] + ".")
        res['time_of_day'] = time_features[0]
    return res
```

```python
def get_user_preference():
    res = {}
    taste_features = ['salty','sweet']
    pref = input("What type of food would you prefer? Enter number 1.salty 2.sweet\n")
    
    try:
        ppref = int(pref)
        if(ppref<=0 or ppref>len(taste_features)):
            raise IndexError
        res['taste_preference'] = taste_features[ppref-1]
    except (ValueError, IndexError):
        print("Entered value is invalid. Setting feature value to", taste_features[0]+ ".")
        res['taste_preference'] = taste_features[0]
    return res
```

## <a name="create-the-learning-loop"></a>Создание цикла обучения

Цикл обучения Персонализатора — это цикл вызовов [ранжирования](#request-the-best-action) и [вознаграждений](#send-a-reward). В этом кратком руководстве за каждым вызовом ранжирования для персонализации содержимого следует вызов вознаграждения. Это необходимо, чтобы определить, насколько хорошо сработала служба.

Следующий код циклически запрашивает у пользователя предпочтения в командной строке, отправляя эти сведения в Персонализатор, чтобы выбрать лучшее действие. Затем он предоставляет клиенту для выбора список вариантов и отправляет вознаграждения в Персонализатор, указывая, насколько хорошо служба выполнила свой выбор.

```python
keep_going = True
while keep_going:

    eventid = str(uuid.uuid4())

    context = [get_user_preference(), get_user_timeofday()]
    actions = get_actions()

    rank_request = RankRequest( actions=actions, context_features=context, excluded_actions=['juice'], event_id=eventid)
    response = client.rank(rank_request=rank_request)
    
    print("Personalizer service ranked the actions with the probabilities listed below:")
    
    rankedList = response.ranking
    for ranked in rankedList:
        print(ranked.id, ':',ranked.probability)

    print("Personalizer thinks you would like to have", response.reward_action_id+".")
    answer = input("Is this correct?(y/n)\n")[0]

    reward_val = "0.0"
    if(answer.lower()=='y'):
        reward_val = "1.0"
    elif(answer.lower()=='n'):
        reward_val = "0.0"
    else:
        print("Entered choice is invalid. Service assumes that you didn't like the recommended food choice.")

    client.events.reward(event_id=eventid, value=reward_val)

    br = input("Press Q to exit, any other key to continue: ")
    if(br.lower()=='q'):
        keep_going = False
```

Перед запуском файла кода добавьте следующие методы для [получения вариантов содержимого](#get-content-choices-represented-as-actions):

* `get_user_preference`
* `get_user_timeofday`
* `get_actions`

## <a name="request-the-best-action"></a>Запрос лучшего действия

Чтобы выполнить запрос ранжирования, программа запрашивает предпочтения пользователя для создания `currentContent` вариантов содержимого. Процесс может создавать содержимое для исключения из действий. Это содержимое отображается в виде `excludeActions`. Запрос ранжирования требует действий и их признаков, признаков currentContext, excludeActions и уникального идентификатора события, чтобы получить ответ.

В этом кратком руководстве используются простые контекстные признаки времени суток и предпочтений пользователя в пище. В рабочих системах определение и [оценка](../concept-feature-evaluation.md) [действий и функций](../concepts-features.md) может отличаться.

```python
rank_request = RankRequest( actions=actions, context_features=context, excluded_actions=['juice'], event_id=eventid)
response = client.rank(rank_request=rank_request)
```

## <a name="send-a-reward"></a>Отправка вознаграждения

Чтобы получить оценку вознаграждения для отправки в запросе вознаграждения, программа получает выбор пользователя из командной строки, присваивает числовое значение каждому выбору, а затем отправляет уникальный идентификатор события и оценку вознаграждения в виде числового значения в API вознаграждения.

В этом кратком руководстве в качестве оценки вознаграждения используется простое число: ноль или 1. В рабочих системах определение времени и содержимого ответа на вызов [вознаграждения](../concept-rewards.md) может быть нетривиальным. Это зависит от конкретных потребностей.

```python
reward_val = "0.0"
if(answer.lower()=='y'):
    reward_val = "1.0"
elif(answer.lower()=='n'):
    reward_val = "0.0"
else:
    print("Entered choice is invalid. Service assumes that you didn't like the recommended food choice.")

client.events.reward(event_id=eventid, value=reward_val)
```

## <a name="run-the-program"></a>Запуск программы

Запустите приложение с Python из каталога приложения.

```console
python sample.py
```

![Программа быстрого запуска задает несколько вопросов, чтобы собрать настройки пользователя, известные как признаки, а затем предоставляет наиболее эффективные действия.](../media/csharp-quickstart-commandline-feedback-loop/quickstart-program-feedback-loop-example.png)
