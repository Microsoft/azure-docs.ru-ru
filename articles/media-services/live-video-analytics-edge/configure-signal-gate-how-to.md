---
title: Настройка шлюза сигнала для записи видео на основе событий — Azure
description: В этой статье приводятся рекомендации по настройке шлюза сигналов в графе мультимедиа.
ms.topic: how-to
ms.date: 11/3/2020
ms.openlocfilehash: afcec7c03f1353f08b58311278f5a533e0c911bc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94410799"
---
# <a name="configure-a-signal-gate-for-event-based-video-recording"></a>Настройка шлюза сигнала для записи видео на основе событий

В графе мультимедиа [узел процессора сигнального шлюза](media-graph-concept.md#signal-gate-processor) позволяет пересылать носители с одного узла на другой, когда шлюз активируется событием. При активации шлюза открывается и позволяет потоку мультимедиа проходить через определенное время. В случае отсутствия событий для активации шлюза шлюз закрывается, а носитель перестает поступать. Обработчик шлюза сигналов можно использовать для записи видео на основе событий.

В этой статье вы узнаете, как настроить обработчик сигнальных шлюзов.

## <a name="suggested-prereading"></a>Предлагаемый для чтения
-   [Граф мультимедиа](media-graph-concept.md)
-   [Запись видео на основе событий](event-based-video-recording-concept.md)


## <a name="problem"></a>Проблема
Пользователю может потребоваться начать запись в определенное время до или после активации шлюза событием. Пользователь знает допустимую задержку в своей системе. Поэтому им нужно указать задержку процессора сигнального шлюза. Они также хотят указать минимальную и максимальную продолжительность записи, независимо от того, сколько новых событий получено.
 
### <a name="use-case-scenario"></a>Сценарий использования
Предположим, вы хотите записать видео каждый раз, когда откроется Передняя дверца здания. Вы хотите, чтобы запись была: 

- Включить *X* секунд перед открытием двери. 
- Последний по крайней мере *Y* секунд, если дверца еще не открыта. 
- Последнее не более *Z* секунд, если дверца повторно открыта. 
 
Вы понимаете, что датчик двери имеет задержку в *K* секунд. Чтобы снизить вероятность возникновения событий, которые не учитываются как последние поступления, необходимо разрешить получение событий не *менее чем в секунду.*


## <a name="solution"></a>Решение

Чтобы устранить эту проблему, измените параметры процессора сигнального шлюза.

Чтобы настроить обработчик сигнальных шлюзов, используйте следующие четыре параметра:
- Окно "Оценка активации"
- Смещение сигнала активации
- Минимальное окно активации
- Максимальное окно активации

Когда запускается обработчик шлюза сигнала, он остается открытым в течение минимального времени активации. Событие активации начинается с метки времени для самого раннего события, а также от смещения сигнала активации. 

Если обработчик шлюза сигнала запускается снова, когда он открыт, таймер сбрасывается, а шлюз остается открытым по крайней мере с минимальным временем активации. Обработчик шлюза сигнала никогда не остается открытым дольше, чем максимальное время активации. 

Событие (событие 1), которое происходит перед другим событием (событие 2) на основе меток времени носителя, может игнорироваться, если система перестает работать, а событие 1 — в обработчике шлюза сигнала после события 2. Если событие 1 не поступает между поступлением события 2 и окна оценки активации, событие 1 не учитывается. Он не передается с помощью процессора сигнального шлюза. 

Идентификаторы корреляции задаются для каждого события. Эти идентификаторы задаются исходя из начального события. Они последовательно используются для каждого следующего события.

> [!IMPORTANT]
> Время носителя основано на метке времени носителя при возникновении события на носителе. Последовательность событий, поступающих в шлюз сигнала, может не отражать последовательность событий, поступающих в время мультимедиа.


### <a name="parameters-based-on-the-physical-time-that-events-arrive-at-the-signal-gate"></a>Параметры, основанные на физическом времени прибытия событий в шлюзе сигнала

* **минимумактиватионтиме (самая короткая возможная длительность записи)**: минимальное время в секундах, в течение которого обработчик шлюза сигнала остается открытым после активации для получения новых событий, если он не был прерван максимумактиватионтиме.
* **максимумактиватионтиме (наибольшая возможная длительность записи)**: максимальное количество секунд от начального события, которое обработчик шлюза сигнала остается открытым после активации для получения новых событий, независимо от того, какие события получены.
* **активатионсигналоффсет**: количество секунд между активацией процессора сигнального шлюза и началом записи видео. Как правило, это значение отрицательно, поскольку оно начинает запись перед срабатыванием события.
* **активатионевалуатионвиндов**: начиная с события первоначального запуска, время в секундах, в течение которого событие, произошедшее перед начальным событием в Media Time, должно поступать на процессор шлюза сигнала, прежде чем он не будет учитываться и считаться последним.

> [!NOTE]
> *Позднее* появление — это любое событие, которое появляется после того, как окно оценки активации прошло, но получено перед первоначальным событием во время мультимедиа.

### <a name="limits-of-parameters"></a>Ограничения параметров

* **активатионевалуатионвиндов**: от 0 секунд до 10 секунд
* **активатионсигналоффсет**: от-1 минуты до 1 минуты
* **минимумактиватионтиме**: от 1 секунды до 1 часа
* **максимумактиватионтиме**: от 1 секунды до 1 часа


В варианте использования параметры задаются следующим образом:

* **активатионевалуатионвиндов**: *Кбайт* в секунду
* **активатионсигналоффсет**: *-X* секунд
* **минимумактиватионвиндов**: *Y* с
* **максимумактиватионвиндов**: *Z* с


Ниже приведен пример того, как раздел узла **процессора сигнального шлюза** будет выглядеть в топологии графа мультимедиа для следующих значений параметров:
* **активатионевалуатионвиндов**: 1 секунда
* **активатионсигналоффсет**:-5 секунд
* **минимумактиватионтиме**: 20 секунд
* **максимумактиватионтиме**: 40 с

> [!IMPORTANT]
> Для каждого значения параметра ожидается [формат длительности ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations
) . Например, PT1S = 1 Second.


```
"processors":              
[
          {
            "@type": "#Microsoft.Media.MediaGraphSignalGateProcessor",
            "name": "signalGateProcessor",
            "inputs": [
              {
                "nodeName": "iotMessageSource"
              },
              {
                "nodeName": "rtspSource"
              }
            ],
            "activationEvaluationWindow": "PT1S",
            "activationSignalOffset": "-PT5S",
            "minimumActivationTime": "PT20S",
            "maximumActivationTime": "PT40S"
          }
]
```


Теперь рассмотрим, как эта конфигурация процессора сигнального шлюза будет работать в различных сценариях записи.

### <a name="recording-scenarios"></a>Сценарии записи

**Одно событие из одного источника (*Обычная активация*)**

Обработчик сигнальных шлюзов, получающий одно событие, приводит к записи, которая начинается 5 секунд (сигнал активации = 5 секунд), прежде чем событие поступит в шлюз. Оставшаяся часть записи составляет 20 секунд (минимальное время активации = 20 секунд), так как никакие другие события не поступают до окончания минимального времени активации для повторного запуска шлюза.

Пример диаграммы:
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/normal-activation.svg" alt-text="Схема, демонстрирующая нормальную активацию одного события из одного источника.":::

* Длительность записи =-offset + Минимумактиватионтиме = [E1 + offset, E1 + Минимумактиватионтиме]


**Два события из одного источника (*Активация активируется* повторно)**

Обработчик сигнального шлюза, получающий два события, приводит к записи, которая начинается 5 секунд (смещение сигнала активации = 5 секунд), прежде чем событие поступит в шлюз. Кроме того, событие 2 прибывает 5 секунд после события 1. Так как событие 2 прибывает до окончания минимального времени активации события 1 (20 секунд), шлюз активируется повторно. Оставшаяся часть записи составляет 20 секунд (минимальное время активации = 20 секунд), так как никакие другие события не поступают до окончания минимального периода активации из события 2 для повторного запуска шлюза.

Пример диаграммы:
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/retriggering-activation.svg" alt-text="Схема, на которой показано повторное активирование двух событий из одного источника.":::

* Длительность записи =-Offset + (прибытие события 2 — Прибытие события 1) + Минимумактиватионтиме


***N* событий из одного источника (*Максимальная активация*)**

Обработчик сигнальных шлюзов, который получает *N* событий, приводит к записи, которая начинается 5 секунд (смещение сигнала активации = 5 секунд) перед поступлением первого события в шлюз. Так как каждое событие прибывает до окончания минимального времени активации, равного 20 секундам предыдущего события, шлюз постоянно перезапускается. Он остается открытым до тех пор, пока не будет достигнуто максимальное время активации, равное 40 секундам после первого события. Затем шлюз закрывается и больше не принимает новые события.

Пример диаграммы:
> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/configure-signal-gate-how-to/maximum-activation.svg" alt-text="Схема, показывающая максимальную активацию N событий из одного источника.":::
 
* Продолжительность записи =-offset + Максимумактиватионтиме

> [!IMPORTANT]
> На приведенных выше схемах предполагается, что каждое событие прибывает в одно и то же время в физическом времени и на носителе. То есть предполагается, что нет последних появления.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [руководством по записи видео на основе событий](event-based-video-recording-tutorial.md). Начните с изменения [topology.jsв](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json). Измените параметры для узла Сигналгатепроцессор, а затем следуйте указаниям в остальной части этого руководства. Проверьте записи видео для анализа влияния параметров.



