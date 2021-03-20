---
title: Обработка потоков данных IoT в режиме реального времени с помощью Azure Stream Analytics
description: 'Теги и потоки данных датчиков IoT: обработка данных с использованием Stream Analytics в режиме реального времени'
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: how-to
ms.date: 11/26/2019
ms.openlocfilehash: a438401ff93c20d8759e6128936c3626bd3de484
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98012688"
---
# <a name="process-real-time-iot-data-streams-with-azure-stream-analytics"></a>Обработка потоков данных IoT в режиме реального времени с помощью Azure Stream Analytics

Из этой статьи вы узнаете, как создать логику обработки потоков для сбора данных с устройств Интернет вещей (IoT). Вы используете реальный вариант использования Интернет вещей (IoT), чтобы продемонстрировать, как быстро и появился создать решение.

## <a name="prerequisites"></a>Предварительные условия

* Создайте бесплатную [подписку Azure](https://azure.microsoft.com/pricing/free-trial/).
* Скачайте пример запроса и файлы данных с сайта [GitHub](https://aka.ms/azure-stream-analytics-get-started-iot).

## <a name="scenario"></a>Сценарий

Компания Contoso, специализирующаяся в области промышленной автоматизации, полностью автоматизировала свой процесс производства. Оборудование этой компании оснащено датчиками, которые передают потоки данных в режиме реального времени. В этом сценарии руководителю производственного участка необходимо в реальном времени получать аналитические данные с датчиков, чтобы определять шаблоны и предпринимать соответствующие действия. Можно использовать язык запросов Stream Analytics (SAQL) на данных датчика, чтобы найти интересные закономерности из входящего потока данных.

В этом примере данные формируются из устройства с тегом датчика инструментов Texas Instruments. Полезные данные хранятся в формате JSON и выглядят следующим образом:

```json
{
    "time": "2016-01-26T20:47:53.0000000",  
    "dspl": "sensorE",  
    "temp": 123,  
    "hmdt": 34  
}  
```

В реальном сценарии у вас могут быть сотни таких датчиков, создающих события в виде потока. В идеале необходимо устройство шлюза, выполняющее код для отправки этих событий в [Центры событий Azure](https://azure.microsoft.com/services/event-hubs/) или [центры Интернета вещей Azure](https://azure.microsoft.com/services/iot-hub/). Задание Stream Analytics будет принимать эти события из Центров событий и выполнять аналитику в режиме реального времени с помощью запросов. Затем можно отправить результаты в один из [поддерживаемых выходов](stream-analytics-define-outputs.md).

Для удобства в этом руководстве по началу работы приведен пример файла данных, полученных с реальных устройств SensorTag. На основе этого примера можно выполнять различные запросы и просматривать их результаты. В последующих руководствах вы узнаете, как подключить задание к источникам входных данных и местам назначения выходных данных, а также развернуть его в службе Azure.

## <a name="create-a-stream-analytics-job"></a>Создание задания Stream Analytics

1. В [портал Azure](https://portal.azure.com)выберите **+ создать ресурс** в меню навигации слева. Затем выберите **Stream Analytics задание** из **аналитики**.
   
    ![Создание нового задания Stream Analytics](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-02.png)

1. Введите уникальное имя задания и убедитесь, что отображается правильная подписка. Создайте новую группу ресурсов или выберите существующую из подписки.

1. Выберите расположение для задания. Используйте то же расположение для группы ресурсов и всех ресурсов, чтобы увеличить скорость обработки и снизить затраты. После завершения настройки нажмите кнопку **создать**.
   
    ![Создание нового задания Stream Analytics](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-03.png)

## <a name="create-an-azure-stream-analytics-query"></a>Создание запроса Azure Stream Analytics
Следующий шаг после создания задания — написание запроса. Вы можете проверить запросы к образцу данных, не подключив в задание входные или выходные данные.

Скачайте [HelloWorldASA-InputStream.jsна](https://github.com/Azure/azure-stream-analytics/blob/master/Samples/GettingStarted/HelloWorldASA-InputStream.json
) сайте GitHub. Затем перейдите к заданию Azure Stream Analytics в портал Azure.

Выберите **запрос** в разделе **топология задания** в меню слева. Затем выберите **отправить образец входных данных**. Отправьте `HelloWorldASA-InputStream.json` файл и нажмите кнопку **ОК**.

![Плитка "Запрос" на панели мониторинга Stream Analytics](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-05.png)

Обратите внимание, что предварительный просмотр данных автоматически заполняется в таблице **Предварительный просмотр ввода** .

![Предварительный просмотр образца входных данных](./media/stream-analytics-get-started-with-iot-devices/input-preview.png)

### <a name="query-archive-your-raw-data"></a>Запрос: архивация необработанных данных

Запрос к серверу — это самая простая форма запроса, который архивирует все входные данные в место назначения выходных данных. Этот запрос является запросом по умолчанию, заполненным в новом задании Azure Stream Analytics.

```sql
SELECT 
    *
INTO
    Output
FROM
    InputStream
```

Выберите **тестовый запрос** и просмотрите результаты в таблице **Результаты теста** .

![Результаты теста для запроса Stream Analytics](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-07.png)

### <a name="query-filter-the-data-based-on-a-condition"></a>Запрос: фильтрация данных по условию

Давайте попробуем отфильтровать результаты на основе условия. Нам требуется отобразить результаты только тех событий, которые получены из датчика sensorA.

```sql
SELECT 
    time,
    dspl AS SensorName,
    temp AS Temperature,
    hmdt AS Humidity
INTO
   Output
FROM
    InputStream
WHERE dspl='sensorA'
```

Вставьте запрос в редактор и выберите **проверить запрос** , чтобы проверить результаты.

![Фильтрация потока данных](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-08.png)

### <a name="query-alert-to-trigger-a-business-workflow"></a>Запрос: оповещение для активации рабочего бизнес-процесса

Теперь давайте детализируем наш запрос. Если требуется отслеживать среднее значение температуры за 30-секундный промежуток времени и отображать результаты только в том случае, если это значение превышает 100 градусов,

```sql
SELECT 
    System.Timestamp AS OutputTime,
    dspl AS SensorName,
    Avg(temp) AS AvgTemperature
INTO
   Output
FROM
    InputStream TIMESTAMP BY time
GROUP BY TumblingWindow(second,30),dspl
HAVING Avg(temp)>100
```

![Запрос с 30-секундным фильтром](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-10.png)

Должны отобразиться результаты, содержащие только 245 строк и имен датчиков, в которых среднее значение температур больше 100. В этом запросе поток событий сгруппирован по свойству **dspl**, представляющему собой имя датчика, в **"переворачивающемся" окне** длительностью 30 секунд. Временные запросы должны указать, как должно выполняться время выполнения. С помощью предложения **timestamp by** вы указали столбец **аутпуттиме** , чтобы связать время со всеми временными вычислениями. Подробные сведения см. в статье [Управление временем](/stream-analytics-query/time-management-azure-stream-analytics) и [функции для окон](/stream-analytics-query/windowing-azure-stream-analytics).

### <a name="query-detect-absence-of-events"></a>Запрос: обнаружение отсутствия событий

Как написать запрос, чтобы обнаружить отсутствие входящих событий? Давайте настроим время последней отправки данных датчиком, а затем не отправляли события в течение следующих 5 секунд.

```sql
SELECT 
    t1.time,
    t1.dspl AS SensorName
INTO
   Output
FROM
    InputStream t1 TIMESTAMP BY time
LEFT OUTER JOIN InputStream t2 TIMESTAMP BY time
ON
    t1.dspl=t2.dspl AND
    DATEDIFF(second,t1,t2) BETWEEN 1 and 5
WHERE t2.dspl IS NULL
```

![Обнаружение отсутствия событий](./media/stream-analytics-get-started-with-iot-devices/stream-analytics-get-started-with-iot-devices-11.png)

Здесь используется **LEFT OUTER JOIN** (левое внешнее соединение) для одного и того же потока данных (самосоединение). При **внутреннем соединении** результат возвращается, только если обнаружено совпадение.  Но если событие с левой стороны соединения не сопоставлено при использовании **левого внешнего соединения**, для всех столбцов справа возвращается строка со значением NULL. Этот метод очень удобно использовать для поиска отсутствия событий. Дополнительные сведения см. в разделе [Join](/stream-analytics-query/join-azure-stream-analytics).

## <a name="conclusion"></a>Заключение

В этой статье показано, как писать различные запросы на языке Stream Analytics запросов и просматривать результаты в браузере. Однако это просто приступить к работе. Обработчик Stream Analytics поддерживает разнообразные входные и выходные данные и может даже использовать функции машинного обучения Azure. Благодаря этому он является надежным средством для анализа потоков данных. Дополнительные сведения о написании запросов см. в статье [Примеры запросов для распространенных шаблонов использования Stream Analytics](stream-analytics-stream-analytics-query-patterns.md).