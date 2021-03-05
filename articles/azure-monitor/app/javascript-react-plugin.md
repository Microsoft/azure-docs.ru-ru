---
title: Подключаемый модуль реагирования для Application Insights SDK JavaScript
description: Установка и использование подключаемого модуля реагирования для Application Insights пакета SDK для JavaScript.
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: 44554434eee51c11e7f89007c532f1a142fc998c
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102199345"
---
# <a name="react-plugin-for-application-insights-javascript-sdk"></a>Подключаемый модуль реагирования для Application Insights SDK JavaScript

Подключаемый модуль, реагирующий на Application Insights SDK для JavaScript, включает:

- Отслеживание изменений маршрута
- Статистика использования компонентов для реагирования

## <a name="getting-started"></a>Начало работы

Установить пакет NPM:

```bash

npm install @microsoft/applicationinsights-react-js

```

## <a name="basic-usage"></a>Основное использование

Инициализация подключения к Application Insights:

```javascript
// AppInsights.js
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ReactPlugin } from '@microsoft/applicationinsights-react-js';
import { createBrowserHistory } from 'history';

const browserHistory = createBrowserHistory({ basename: '' });
const reactPlugin = new ReactPlugin();
const appInsights = new ApplicationInsights({
    config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [reactPlugin],
        extensionConfig: {
          [reactPlugin.identifier]: { history: browserHistory }
        }
    }
});
appInsights.loadAppInsights();
export { reactPlugin, appInsights };
```

Поместите компонент в оболочку с помощью функции компонента более высокого порядка, чтобы включить Application Insights.

```javascript
import React from 'react';
import { withAITracking } from '@microsoft/applicationinsights-react-js';
import { reactPlugin, appInsights } from './AppInsights';

// To instrument various React components usage tracking, apply the `withAITracking` higher-order
// component function.

class MyComponent extends React.Component {
    ...
}

// withAITracking takes 4 parameters ( reactPlugin, Component, ComponentName, className) 
// the first two are required and the other two are optional.

export default withAITracking(reactPlugin, MyComponent);
```

## <a name="configuration"></a>Конфигурация

| Имя    | По умолчанию | Описание                                                                                                    |
|---------|---------|----------------------------------------------------------------------------------------------------------------|
| history | null    | Отреагировать на журнал маршрутизатора. Дополнительные сведения см. в [документации по пакету реагирования маршрутизатора](https://reactrouter.com/web/api/history). Сведения о том, как получить доступ к объекту журнала за пределами компонентов, см. в разделе [часто задаваемые вопросы о маршрутизаторе](https://github.com/ReactTraining/react-router/blob/master/FAQ.md#how-do-i-access-the-history-object-outside-of-components)    |

### <a name="react-components-usage-tracking"></a>Отслеживание использования компонентов отклика

Для инструментирования различных отслеживаний использования компонентов отклика примените `withAITracking` функцию компонента более высокого порядка.

Оно будет измерять время от `ComponentDidMount` события через `ComponentWillUnmount` событие. Однако, чтобы сделать это более точным, будет вычтено время бездействия пользователя `React Component Engaged Time = ComponentWillUnmount timestamp - ComponentDidMount timestamp - idle time` .

Чтобы увидеть эту метрику в портал Azure необходимо перейти к ресурсу Application Insights, перейдите на вкладку "метрики" и настройте пустые диаграммы для вывода имени пользовательской метрики "отреагировать на время отклика компонента (секунды)", выберите агрегирование (SUM, AVG и т. д.) метрики и примените Split — "имя компонента".

![Снимок экрана с диаграммой, отображающей пользовательскую метрику "отклики на время обработки компонента" (в секундах) "разделение по" имени компонента "](./media/javascript-react-plugin/chart.png)

Можно также выполнять пользовательские запросы, чтобы разделить Application Insights данные для создания отчетов и визуализаций в соответствии с вашими требованиями. В портал Azure перейдите к ресурсу Application Insights, выберите "аналитика" в верхнем меню вкладки "Обзор" и выполните запрос.

![Снимок экрана результатов пользовательского запроса метрик.](./media/javascript-react-plugin/query.png)

> [!NOTE]
> Чтобы новые пользовательские метрики отображались на портале Azure, может потребоваться до 10 минут.

## <a name="using-react-hooks"></a>Использование обработчиков реагирующих

[Реагирующие обработчики](https://reactjs.org/docs/hooks-reference.html) — это подход к управлению состоянием и жизненным циклом в приложении с реагированием, не полагаясь на компоненты реагирования на основе классов. Подключаемый модуль Application Insights реагирует на несколько модулей интеграции, которые работают так же, как и компонент более высокого порядка.

### <a name="using-react-context"></a>Использование контекста реагирования

Обработчики реагирования для Application Insights предназначены для использования [контекста реагирования](https://reactjs.org/docs/context.html) в качестве содержащегося в нем аспекта. Чтобы использовать контекст, инициализируйте Application Insights, как описано выше, а затем импортируйте объект Context:

```javascript
import React from "react";
import { AppInsightsContext } from "@microsoft/applicationinsights-react-js";
import { reactPlugin } from "./AppInsights";

const App = () => {
    return (
        <AppInsightsContext.Provider value={reactPlugin}>
            /* your application here */
        </AppInsightsContext.Provider>
    );
};
```

Этот поставщик контекста сделает Application Insights доступным в качестве `useContext` обработчика во всех его дочерних компонентах.

```javascript
import React from "react";
import { useAppInsightsContext } from "@microsoft/applicationinsights-react-js";

const MyComponent = () => {
    const appInsights = useAppInsightsContext();
    
    appInsights.trackMetric("Component 'MyComponent' is in use");
    
    return (
        <h1>My Component</h1>
    );
}
export default MyComponent;
```

### `useTrackMetric`

`useTrackMetric`Обработчик реплицирует функциональные возможности `withAITracking` компонента более высокого порядка, не добавляя к структуре компонента дополнительный компонент. Обработчик принимает два аргумента — первый — Application Insights экземпляр (который может быть получен от `useAppInsightsContext` обработчика) и идентификатор компонента для отслеживания (например, его имя).

```javascript
import React from "react";
import { useAppInsightsContext, useTrackMetric } from "@microsoft/applicationinsights-react-js";

const MyComponent = () => {
    const appInsights = useAppInsightsContext();
    const trackComponent = useTrackMetric(appInsights, "MyComponent");
    
    return (
        <h1 onHover={trackComponent} onClick={trackComponent}>My Component</h1>
    );
}
export default MyComponent;
```

Он будет работать как компонент более высокого порядка, но реагировать на события жизненного цикла обработчиков, а не на жизненный цикл компонента. Обработчик должен быть явно предоставлен для событий пользователя, если требуется выполнить определенные взаимодействия.

### `useTrackEvent`

`useTrackEvent`Обработчик используется для отслеживания любого пользовательского события, которое может потребоваться отслеживанию приложения, например нажатия кнопки или другого вызова API. Он принимает четыре аргумента:
-   Application Insights экземпляр (который можно получить из `useAppInsightsContext` обработчика).
-   Имя события.
-   Объект данных события, инкапсулирующий изменения, которые должны быть отслеживанием.
-   Скипфирструн (необязательный) флаг для пропуска вызова `trackEvent` вызова при инициализации. Значение по умолчанию — `true`.

```javascript
import React, { useState, useEffect } from "react";
import { useAppInsightsContext, useTrackEvent } from "@microsoft/applicationinsights-react-js";

const ProductCart = () => {
    const appInsights = useAppInsightsContext();
    const trackCheckout = useTrackEvent(appInsights, "Checkout");
    const trackCartUpdate = useTrackEvent(appInsights, "Cart Updated");
    const [cart, setCart] = useState([]);
    
    useEffect(() => {
        trackCartUpdate({ cartCount: cart.length });
    }, [cart]);
    
    const performCheckout = () => {
        trackCheckout();
        // submit data
    };
    
    return (
        <div>
            <ul>
                <li>Product 1 <button onClick={() => setCart([...cart, "Product 1"])}>Add to Cart</button>
                <li>Product 2 <button onClick={() => setCart([...cart, "Product 2"])}>Add to Cart</button>
                <li>Product 3 <button onClick={() => setCart([...cart, "Product 3"])}>Add to Cart</button>
                <li>Product 4 <button onClick={() => setCart([...cart, "Product 4"])}>Add to Cart</button>
            </ul>
            <button onClick={performCheckout}>Checkout</button>
        </div>
    );
}
export default MyComponent;
```

При использовании обработчика в него можно предоставить полезные данные, чтобы добавить дополнительные данные в событие, когда оно будет сохранено в Application Insights.

## <a name="react-error-boundaries"></a>Границы ошибок реагирования

[Границы ошибок](https://reactjs.org/docs/error-boundaries.html) дают возможность корректно обрабатывать исключение, когда оно происходит в приложении-отклике, и при возникновении такой ошибки вполне вероятно, что исключение необходимо регистрировать. Подключаемый модуль реагирования для Application Insights предоставляет компонент границы ошибки, который автоматически регистрирует ошибку при ее возникновении.

```javascript
import React from "react";
import { reactPlugin } from "./AppInsights";
import { AppInsightsErrorBoundary } from "@microsoft/applicationinsights-react-js";

const App = () => {
    return (
        <AppInsightsErrorBoundary onError={() => <h1>I believe something went wrong</h1>} appInsights={reactPlugin}>
            /* app here */
        </AppInsightsErrorBoundary>
    );
};
```

Для необходимо `AppInsightsErrorBoundary` передать в него два свойства Prop, `ReactPlugin` экземпляр, созданный для приложения, и компонент для отображения при возникновении ошибки. При возникновении необработанной ошибки `trackException` вызывается с информацией, предоставленной для границы ошибки, и `onError` отображается компонент.

## <a name="sample-app"></a>Пример приложения

Ознакомьтесь с [демонстрацией Application Insights реагировать](https://github.com/Azure-Samples/application-insights-react-demo).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о пакете SDK для JavaScript см. в [документации по Application Insights JavaScript SDK](javascript.md).
- Дополнительные сведения о языке запросов Kusto и запросе данных в Log Analytics см. в разделе [Общие сведения о запросе журнала](../../azure-monitor/logs/log-query-overview.md).
