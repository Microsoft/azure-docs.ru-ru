---
title: Торговля с высокой частотой с помощью Azure Stream Analytics
description: Обучение и оценка модели линейной регрессии в задании Stream Analytics
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/16/2021
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: f632c916c3de61b94532e96be23da511ad5863ea
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104593039"
---
# <a name="high-frequency-trading-simulation-with-stream-analytics"></a>Моделирование высокочастотного трейдинга с помощью Stream Analytics
Совместное использование языка SQL, пользовательских функций и пользовательских агрегатных функций в Azure Stream Analytics позволяет пользователям выполнять расширенную аналитику. Расширенная аналитика может включать обучающие онлайн-сценарии машинного обучения и оценки, а также моделирование процесса с отслеживанием состояния. В этой статье описывается, как работать с линейной регрессией в задании Azure Stream Analytics, которое выполняет непрерывное обучение и оценку в сценарии с высокочастным трейдингом.

## <a name="high-frequency-trading"></a>Высокочастотный трейдинг
Функции логической последовательности высокочастного трейдинга:
1. Получение котировок из защищенного канала обмена в режиме реального времени.
2. Создание прогнозной модели котировок, на основе которой можно прогнозировать изменения цен.
3. Создание запросов на покупку или продажу для получения прибыли в результате успешного прогноза изменения цен. 

В итоге нам нужно следующее:
* получение котировок в режиме реального времени;
* прогнозная модель, которая может обрабатывать котировки в режиме реального времени;
* модель трейдинга, которая показывает соотношение доходов или расходов, полученное с использованием алгоритма трейдинга.

### <a name="real-time-quote-feed"></a>Получение котировок в режиме реального времени
IEX предлагает бесплатные [котировки бид/аск в режиме реального времени](https://iextrading.com/developer/docs/#websockets) с помощью socket.io. Для получения котировок в режиме реального времени и их отправки в Центры событий Azure в качестве источника данных можно написать простое консольное приложение. Ниже приведен код, который составляет основу программы. В коде для краткости пропущена обработка ошибок. Вам также потребуется включить в проект пакеты NuGet SocketIoClientDotNet и WindowsAzure.ServiceBus.

```csharp
using Quobject.SocketIoClientDotNet.Client;
using Microsoft.ServiceBus.Messaging;
var symbols = "msft,fb,amzn,goog";
var eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, eventHubName);
var socket = IO.Socket("https://ws-api.iextrading.com/1.0/tops");
socket.On(Socket.EVENT_MESSAGE, (message) =>
{
    eventHubClient.Send(new EventData(Encoding.UTF8.GetBytes((string)message)));
});
socket.On(Socket.EVENT_CONNECT, () =>
{
    socket.Emit("subscribe", symbols);
});
```

Ниже приведены некоторые созданные примеры событий.

```json
{"symbol":"MSFT","marketPercent":0.03246,"bidSize":100,"bidPrice":74.8,"askSize":300,"askPrice":74.83,volume":70572,"lastSalePrice":74.825,"lastSaleSize":100,"lastSaleTime":1506953355123,lastUpdated":1506953357170,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"GOOG","marketPercent":0.04825,"bidSize":114,"bidPrice":870,"askSize":0,"askPrice":0,volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,lastUpdated":1506953357633,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"MSFT","marketPercent":0.03244,"bidSize":100,"bidPrice":74.8,"askSize":100,"askPrice":74.83,volume":70572,"lastSalePrice":74.825,"lastSaleSize":100,"lastSaleTime":1506953355123,lastUpdated":1506953359118,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"FB","marketPercent":0.01211,"bidSize":100,"bidPrice":169.9,"askSize":100,"askPrice":170.67,volume":39042,"lastSalePrice":170.67,"lastSaleSize":100,"lastSaleTime":1506953351912,lastUpdated":1506953359641,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"GOOG","marketPercent":0.04795,"bidSize":100,"bidPrice":959.19,"askSize":0,"askPrice":0,volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,lastUpdated":1506953360949,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"FB","marketPercent":0.0121,"bidSize":100,"bidPrice":169.9,"askSize":100,"askPrice":170.7,volume":39042,"lastSalePrice":170.67,"lastSaleSize":100,"lastSaleTime":1506953351912,lastUpdated":1506953362205,"sector":"softwareservices","securityType":"commonstock"}
{"symbol":"GOOG","marketPercent":0.04795,"bidSize":114,"bidPrice":870,"askSize":0,"askPrice":0,volume":11240,"lastSalePrice":959.47,"lastSaleSize":60,"lastSaleTime":1506953317571,lastUpdated":1506953362629,"sector":"softwareservices","securityType":"commonstock"}
```

>[!NOTE]
>Метка времени события — **lastUpdated** в формате UNIX-времени.

### <a name="predictive-model-for-high-frequency-trading"></a>Прогнозная модель высокочастотного трейдинга
В целях демонстрации мы используем линейную модель, описанную в Дэррил Shen) в [своем документе](https://docplayer.net/23038840-Order-imbalance-based-strategy-in-high-frequency-trading.html).

Диспропорция объемных биржевых заявок (VOI) — это функция текущей цены бид/аск и объема, а также соотношения цены бид/аск и объема в пределах последнего тика. В документе определяется корреляция между VOI и будущим изменением цены, а также создается линейная модель между предыдущими 5 значениями VOI и изменением цены в течение следующих 10 тиков. Обучение модели выполняется на основе данных предыдущего дня с использованием линейной регрессии. 

Обученная модель затем используется для прогнозирования изменения цены котировок за текущий торговый день в режиме реального времени. Сделка производится при прогнозировании существенного изменения цены. В зависимости от заданного порогового значения можно ожидать совершения тысяч сделок по одной акции в течение торгового дня.

![Определение диспропорции объемных биржевых заявок](./media/stream-analytics-high-frequency-trading/volume-order-imbalance-formula.png)

Теперь давайте включим операции обучения и прогнозирования в задание Azure Stream Analytics.

Во-первых, мы удалим входные данные. Формат UNIX-времени преобразуется в значение даты и времени с помощью функции **DATEADD**. Функция **TRY_CAST** используется для присвоения типов данных без ошибок в запросе. Рекомендуется всегда приводить в соответствие поля ввода с типами ожидаемых данных, чтобы избежать непредвиденного поведения при обработке или сравнении полей.

```SQL
WITH
typeconvertedquotes AS (
    /* convert all input fields to proper types */
    SELECT
        System.Timestamp AS lastUpdated,
        symbol,
        DATEADD(millisecond, CAST(lastSaleTime as bigint), '1970-01-01T00:00:00Z') AS lastSaleTime,
        TRY_CAST(bidSize as bigint) AS bidSize,
        TRY_CAST(bidPrice as float) AS bidPrice,
        TRY_CAST(askSize as bigint) AS askSize,
        TRY_CAST(askPrice as float) AS askPrice,
        TRY_CAST(volume as bigint) AS volume,
        TRY_CAST(lastSaleSize as bigint) AS lastSaleSize,
        TRY_CAST(lastSalePrice as float) AS lastSalePrice
    FROM quotes TIMESTAMP BY DATEADD(millisecond, CAST(lastUpdated as bigint), '1970-01-01T00:00:00Z')
),
timefilteredquotes AS (
    /* filter between 7am and 1pm PST, 14:00 to 20:00 UTC */
    /* clean up invalid data points */
    SELECT * FROM typeconvertedquotes
    WHERE DATEPART(hour, lastUpdated) >= 14 AND DATEPART(hour, lastUpdated) < 20 AND bidSize > 0 AND askSize > 0 AND bidPrice > 0 AND askPrice > 0
),
```

Затем мы воспользуемся функцией **LAG** для получения данных за последний тик. Мы выберем для функции **LIMIT DURATION** произвольное значение в один час. Учитывая частоту обновления котировок, на основе этого значения можно с уверенностью предположить, что мы определим предыдущий тик.  

```SQL
shiftedquotes AS (
    /* get previous bid/ask price and size in order to calculate VOI */
    SELECT
        symbol,
        (bidPrice + askPrice)/2 AS midPrice,
        bidPrice,
        bidSize,
        askPrice,
        askSize,
        LAG(bidPrice) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS bidPricePrev,
        LAG(bidSize) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS bidSizePrev,
        LAG(askPrice) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS askPricePrev,
        LAG(askSize) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS askSizePrev
    FROM timefilteredquotes
),
```

Затем мы сможем вычислить значение VOI. На всякий случай мы отфильтруем значения NULL, если предыдущий тик не существует.

```SQL
currentPriceAndVOI AS (
    /* calculate VOI */
    SELECT
        symbol,
        midPrice,
        (CASE WHEN (bidPrice < bidPricePrev) THEN 0
            ELSE (CASE WHEN (bidPrice = bidPricePrev) THEN (bidSize - bidSizePrev) ELSE bidSize END)
         END) -
        (CASE WHEN (askPrice < askPricePrev) THEN askSize
            ELSE (CASE WHEN (askPrice = askPricePrev) THEN (askSize - askSizePrev) ELSE 0 END)
         END) AS VOI
    FROM shiftedquotes
    WHERE
        bidPrice IS NOT NULL AND
        bidSize IS NOT NULL AND
        askPrice IS NOT NULL AND
        askSize IS NOT NULL AND
        bidPricePrev IS NOT NULL AND
        bidSizePrev IS NOT NULL AND
        askPricePrev IS NOT NULL AND
        askSizePrev IS NOT NULL
),
```

Теперь мы снова воспользуемся функцией **LAG** для создания последовательности с двумя идущими подряд значениями VOI, за которыми следуют десять идущих подряд значений средней цены.

```SQL
shiftedPriceAndShiftedVOI AS (
    /* get 10 future prices and 2 previous VOIs */
    SELECT
        symbol,
        midPrice AS midPrice10,
        LAG(midPrice, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice9,
        LAG(midPrice, 2) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice8,
        LAG(midPrice, 3) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice7,
        LAG(midPrice, 4) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice6,
        LAG(midPrice, 5) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice5,
        LAG(midPrice, 6) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice4,
        LAG(midPrice, 7) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice3,
        LAG(midPrice, 8) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice2,
        LAG(midPrice, 9) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice1,
        LAG(midPrice, 10) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS midPrice,
        LAG(VOI, 10) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI1,
        LAG(VOI, 11) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI2
    FROM currentPriceAndVOI
),
```

Затем мы преобразуем данные во входные данные для линейной модели с двумя переменными. Мы снова отфильтруем события, для которых у нас есть не все данные.

```SQL
modelInput AS (
    /* create feature vector, x being VOI, y being delta price */
    SELECT
        symbol,
        (midPrice1 + midPrice2 + midPrice3 + midPrice4 + midPrice5 + midPrice6 + midPrice7 + midPrice8 + midPrice9 + midPrice10)/10.0 - midPrice AS y,
        VOI1 AS x1,
        VOI2 AS x2
    FROM shiftedPriceAndShiftedVOI
    WHERE
        midPrice1 IS NOT NULL AND
        midPrice2 IS NOT NULL AND
        midPrice3 IS NOT NULL AND
        midPrice4 IS NOT NULL AND
        midPrice5 IS NOT NULL AND
        midPrice6 IS NOT NULL AND
        midPrice7 IS NOT NULL AND
        midPrice8 IS NOT NULL AND
        midPrice9 IS NOT NULL AND
        midPrice10 IS NOT NULL AND
        midPrice IS NOT NULL AND
        VOI1 IS NOT NULL AND
        VOI2 IS NOT NULL
),
```

Так как Azure Stream Analytics не имеет встроенную функцию линейной регрессии, мы воспользуемся статистическими выражениями **SUM** и **AVG**, чтобы вычислить коэффициенты для линейной модели.

![Математическая формула линейной регрессии](./media/stream-analytics-high-frequency-trading/linear-regression-formula.png)

```SQL
modelagg AS (
    /* get aggregates for linear regression calculation,
     http://faculty.cas.usf.edu/mbrannick/regression/Reg2IV.html */
    SELECT
        symbol,
        SUM(x1 * x1) AS x1x1,
        SUM(x2 * x2) AS x2x2,
        SUM(x1 * y) AS x1y,
        SUM(x2 * y) AS x2y,
        SUM(x1 * x2) AS x1x2,
        AVG(y) AS avgy,
        AVG(x1) AS avgx1,
        AVG(x2) AS avgx2
    FROM modelInput
    GROUP BY symbol, TumblingWindow(hour, 24, -4)
),
modelparambs AS (
    /* calculate b1 and b2 for the linear model */
    SELECT
        symbol,
        (x2x2 * x1y - x1x2 * x2y)/(x1x1 * x2x2 - x1x2 * x1x2) AS b1,
        (x1x1 * x2y - x1x2 * x1y)/(x1x1 * x2x2 - x1x2 * x1x2) AS b2,
        avgy,
        avgx1,
        avgx2
    FROM modelagg
),
model AS (
    /* calculate a for the linear model */
    SELECT
        symbol,
        avgy - b1 * avgx1 - b2 * avgx2 AS a,
        b1,
        b2
    FROM modelparambs
),
```

Чтобы использовать модель предыдущего дня для оценки текущего события, мы соединяем котировки с моделью. Но вместо функции **JOIN** мы используем **UNION**, чтобы объединить события модели и события котировок. Затем мы используем функцию **LAG**, чтобы сопоставить события с моделью за предыдущий день и получить ровно одно совпадение. Из-за выходных нам нужно учесть три дня. Если бы мы использовали простую функцию **JOIN**, мы бы получили три модели для каждого события котировки.

```SQL
shiftedVOI AS (
    /* get two consecutive VOIs */
    SELECT
        symbol,
        midPrice,
        VOI AS VOI1,        
        LAG(VOI, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 1)) AS VOI2
    FROM currentPriceAndVOI
),
VOIAndModel AS (
    /* combine VOIs and models */
    SELECT
        'voi' AS type,
        symbol,
        midPrice,
        VOI1,
        VOI2,
        0.0 AS a,
        0.0 AS b1,
        0.0 AS b2
    FROM shiftedVOI
    UNION
    SELECT
        'model' AS type,
        symbol,
        0.0 AS midPrice,
        0 AS VOI1,
        0 AS VOI2,
        a,
        b1,
        b2
    FROM model
),
VOIANDModelJoined AS (
    /* match VOIs with the latest model within 3 days (72 hours, to take the weekend into account) */
    SELECT
        symbol,
        midPrice,
        VOI1 as x1,
        VOI2 as x2,
        LAG(a, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS a,
        LAG(b1, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS b1,
        LAG(b2, 1) OVER (PARTITION BY symbol LIMIT DURATION(hour, 72) WHEN type = 'model') AS b2
    FROM VOIAndModel
    WHERE type = 'voi'
),
```

Теперь мы можем выполнить прогнозирование и создать на основе модели сигналы для покупки и продажи с пороговым значением 0,02. Значение продажи: 10. Значение покупки: -10.

``` SQL
prediction AS (
    /* make prediction if there is a model */
    SELECT
        symbol,
        midPrice,
        a + b1 * x1 + b2 * x2 AS efpc
    FROM VOIANDModelJoined
    WHERE
        a IS NOT NULL AND
        b1 IS NOT NULL AND
        b2 IS NOT NULL AND
        x1 IS NOT NULL AND
        x2 IS NOT NULL
),
tradeSignal AS (
    /* generate buy/sell signals */
    SELECT
        DateAdd(hour, -7, System.Timestamp) AS time,
        symbol,     
        midPrice,
        efpc,
        CASE WHEN (efpc > 0.02) THEN 10 ELSE (CASE WHEN (efpc < -0.02) THEN -10 ELSE 0 END) END AS trade,
        DATETIMEFROMPARTS(DATEPART(year, System.Timestamp), DATEPART(month, System.Timestamp), DATEPART(day, System.Timestamp), 0, 0, 0, 0) as date
    FROM prediction
),
```

### <a name="trading-simulation"></a>Модель трейдинга
Получив трейдинговые сигналы, мы проверим эффективность торговой стратегии, не совершая реальную сделку. 

Этот тест выполняется с помощью пользовательской агрегатной функции с "прыгающим" окном. Прыжок составляет одну минуту. Дополнительная группировка по дате и использование предложения HAVING позволяют обрабатывать в окне события за один и тот же день. Для "прыгающего" окна, включающего события за два дня, функция **GROUP BY** разделяет группирование на два дня: предыдущий и текущий. Предложение **HAVING** фильтрует окна, включающие события за сегодняшний день, но сгруппированные по предыдущему.

```SQL
simulation AS
(
    /* perform trade simulation for the past 7 hours to cover an entire trading day, and generate output every minute */
    SELECT
        DateAdd(hour, -7, System.Timestamp) AS time,
        symbol,
        date,
        uda.TradeSimulation(tradeSignal) AS s
    FROM tradeSignal
    GROUP BY HoppingWindow(minute, 420, 1), symbol, date
    Having DateDiff(day, date, time) < 1 AND DATEPART(hour, time) < 13
)
```

Пользовательская агрегатная функция JavaScript инициализирует все операторы накопления в функции `init`, вычисляет переход состояния для каждого события в окне и возвращает результаты моделирования в конце окна. В общих чертах процедуру заключения сделок можно представить так:

- покупка выполняется, если получен сигнал на покупку и активов нет;
- продажа выполняется, если получен сигнал на продажу и активы есть;
- если активов нет, открывается короткая позиция. 

Если открыта короткая позиция и получен сигнал на покупку, выполняется ее закрытие. Мы будем хранить или короткие 10 долей акций в этой модели. Затраты по сделкам фиксированные и составляют 8 долл. США.

```javascript
function main() {
    var TRADE_COST = 8.0;
    var SHARES = 10;
    this.init = function () {
        this.own = false;
        this.pos = 0;
        this.pnl = 0.0;
        this.tradeCosts = 0.0;
        this.buyPrice = 0.0;
        this.sellPrice = 0.0;
        this.buySize = 0;
        this.sellSize = 0;
        this.buyTotal = 0.0;
        this.sellTotal = 0.0;
    }
    this.accumulate = function (tradeSignal, timestamp) {
        if(!this.own && tradeSignal.trade == 10) {
          // Buy to open
          this.own = true;
          this.pos = 1;
          this.buyPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.buySize += SHARES;
          this.buyTotal += SHARES * tradeSignal.midprice;
        } else if(!this.own && tradeSignal.trade == -10) {
          // Sell to open
          this.own = true;
          this.pos = -1
          this.sellPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.sellSize += SHARES;
          this.sellTotal += SHARES * tradeSignal.midprice;
        } else if(this.own && this.pos == 1 && tradeSignal.trade == -10) {
          // Sell to close
          this.own = false;
          this.pos = 0;
          this.sellPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.pnl += (this.sellPrice - this.buyPrice)*SHARES - 2*TRADE_COST;
          this.sellSize += SHARES;
          this.sellTotal += SHARES * tradeSignal.midprice;
          // Sell to open
          this.own = true;
          this.pos = -1;
          this.sellPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.sellSize += SHARES;        
          this.sellTotal += SHARES * tradeSignal.midprice;
        } else if(this.own && this.pos == -1 && tradeSignal.trade == 10) {
          // Buy to close
          this.own = false;
          this.pos = 0;
          this.buyPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.pnl += (this.sellPrice - this.buyPrice)*SHARES - 2*TRADE_COST;
          this.buySize += SHARES;
          this.buyTotal += SHARES * tradeSignal.midprice;
          // Buy to open
          this.own = true;
          this.pos = 1;
          this.buyPrice = tradeSignal.midprice;
          this.tradeCosts += TRADE_COST;
          this.buySize += SHARES;         
          this.buyTotal += SHARES * tradeSignal.midprice;
        }
    }
    this.computeResult = function () {
        var result = {
            "pnl": this.pnl,
            "buySize": this.buySize,
            "sellSize": this.sellSize,
            "buyTotal": this.buyTotal,
            "sellTotal": this.sellTotal,
            "tradeCost": this.tradeCost
            };
        return result;
    }
}
```

Наконец, мы выведем данные на панель мониторинга Power BI для визуализации.

```SQL
SELECT * INTO tradeSignalDashboard FROM tradeSignal /* output tradeSignal to PBI */
SELECT
    symbol,
    time,
    date,
    TRY_CAST(s.pnl as float) AS pnl,
    TRY_CAST(s.buySize as bigint) AS buySize,
    TRY_CAST(s.sellSize as bigint) AS sellSize,
    TRY_CAST(s.buyTotal as float) AS buyTotal,
    TRY_CAST(s.sellTotal as float) AS sellTotal
    INTO pnlDashboard
FROM simulation /* output trade simulation to PBI */
```

![Точечная диаграмма сделок в Power BI](./media/stream-analytics-high-frequency-trading/trades-power-bi-chart.png)

![Точечная диаграмма прибыли и убытков в Power BI](./media/stream-analytics-high-frequency-trading/pnl-power-bi-chart.png)


## <a name="summary"></a>Итоги
Мы можем реализовать реалистичную модель высокочастотного трейдинга, выполнив в Azure Stream Analytics запрос умеренной сложности. Из-за отсутствия встроенной функции линейной регрессии мы упростили эту модель, сократив число переменных с пяти до двух. Но при необходимости вы также можете реализовать более сложные алгоритмы с большим числом измерений в качестве пользовательской агрегатной функции JavaScript. 

Следует отметить, что большую часть запроса (кроме пользовательской агрегатной функции JavaScript), можно протестировать и отладить в Visual Studio с помощью [инструментов Azure Stream Analytics для Visual Studio](stream-analytics-tools-for-visual-studio-install.md). Написав начальный запрос, автор потратил меньше 30 минут на тестирование и отладку запроса в Visual Studio. 

Сейчас отладка пользовательских агрегатных функций не поддерживается в Visual Studio. Мы работаем над включением этой функции, с возможностью выполнять пошаговую отладку кода JavaScript. Также учитывайте то, что для имен полей, связанных с пользовательскими агрегатными функциями, используется нижний регистр. При проверке запроса такое поведение не является очевидным. Но для уровня совместимости Azure Stream Analytics 1.1 разрешается сохранять в именах полей используемый регистр, что является более естественным.

Надеюсь, эта статья вдохновит всех пользователей Azure Stream Analytics, которые смогут постоянно применять нашу службу для выполнения расширенной аналитики практически в режиме реального времени. Оставьте свои отзывы, чтобы мы могли еще больше упростить реализацию запросов в сценариях расширенной аналитики.
