---
title: Угловой подключаемый модуль для Application Insights SDK JavaScript
description: Как установить и использовать угловой подключаемый модуль для Application Insights SDK для JavaScript.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: lagayhar
ms.openlocfilehash: d45d8bed328dc91dfeeabd6ce878074fa1218623
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101737024"
---
# <a name="angular-plugin-for-application-insights-javascript-sdk"></a>Угловой подключаемый модуль для Application Insights SDK JavaScript

Угловой подключаемый модуль для пакета SDK для Application Insights JavaScript позволяет:

- Отслеживание изменений маршрутизатора

> [!WARNING]
> Угловой подключаемый модуль не совместим с ECMAScript 3 (ES3).

## <a name="getting-started"></a>Начало работы

Установить пакет NPM:

```bash
npm install @microsoft/applicationinsights-angularplugin-js
```

## <a name="basic-usage"></a>Основное использование

Настройте экземпляр Application Insights в компоненте записи в приложении:

```js
import { Component } from '@angular/core';
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { AngularPlugin } from '@microsoft/applicationinsights-angularplugin-js';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
    constructor(
        private router: Router
    ){
        var angularPlugin = new AngularPlugin();
        const appInsights = new ApplicationInsights({ config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [angularPlugin],
        extensionConfig: {
            [angularPlugin.identifier]: { router: this.router }
        }
        } });
        appInsights.loadAppInsights();
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о пакете SDK для JavaScript см. в [документации по Application Insights JavaScript SDK](javascript.md)
- [Угловой подключаемый модуль на GitHub](https://github.com/microsoft/ApplicationInsights-JS/tree/master/extensions/applicationinsights-angularplugin-js)
