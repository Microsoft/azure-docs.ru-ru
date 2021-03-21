---
title: Отслеживание входящих запросов в Azure Application Insights с помощью Опенценсус Python | Документация Майкрософт
description: Отслеживайте вызовы запросов для приложений Python через Опенценсус Python.
ms.topic: conceptual
author: lzchen
ms.author: lechen
ms.date: 10/15/2019
ms.custom: devx-track-python
ms.openlocfilehash: 3b029a9cb14a81c80072847dc17d6b71f480743f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100585678"
---
# <a name="track-incoming-requests-with-opencensus-python"></a>Мониторинг входящих запросов с помощью Опенценсус Python

Данные входящего запроса собираются с помощью Опенценсус Python и различных интеграций. Отследите данные входящего запроса, отправляемые в веб-приложения, созданные на основе популярных веб-платформ `django` , `flask` и `pyramid` . Затем данные отправляются в Application Insights под Azure Monitor в качестве `requests` телеметрии.

Сначала выполните инструментирование приложения Python с помощью последнего [пакета SDK для Опенценсус Python](./opencensus-python.md).

## <a name="tracking-django-applications"></a>Отслеживание Django приложений

1. Скачайте и установите `opencensus-ext-django` из [PyPI](https://pypi.org/project/opencensus-ext-django/) и выполните инструментирование приложения с помощью по `django` промежуточного слоя. Входящие запросы, отправляемые в ваше `django` приложение, будут относиться к отслеживанию.

2. Включите `opencensus.ext.django.middleware.OpencensusMiddleware` в `settings.py` файл в `MIDDLEWARE` .

    ```python
    MIDDLEWARE = (
        ...
        'opencensus.ext.django.middleware.OpencensusMiddleware',
        ...
    )
    ```

3. Убедитесь, что Азурикспортер правильно настроен в вашей `settings.py` среде `OPENCENSUS` . Для запросов с URL-адресов, которые не нужно отслеживанию, добавьте их в `EXCLUDELIST_PATHS` .

    ```python
    OPENCENSUS = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>"
            )''',
            'EXCLUDELIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```

## <a name="tracking-flask-applications"></a>Отслеживание Flask приложений

1. Скачайте и установите `opencensus-ext-flask` из [PyPI](https://pypi.org/project/opencensus-ext-flask/) и выполните инструментирование приложения с помощью по `flask` промежуточного слоя. Входящие запросы, отправляемые в ваше `flask` приложение, будут относиться к отслеживанию.

    ```python
    
    from flask import Flask
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.ext.flask.flask_middleware import FlaskMiddleware
    from opencensus.trace.samplers import ProbabilitySampler
    
    app = Flask(__name__)
    middleware = FlaskMiddleware(
        app,
        exporter=AzureExporter(connection_string="InstrumentationKey=<your-ikey-here>"),
        sampler=ProbabilitySampler(rate=1.0),
    )
    
    @app.route('/')
    def hello():
        return 'Hello World!'
    
    if __name__ == '__main__':
        app.run(host='localhost', port=8080, threaded=True)
    
    ```

2. Вы также можете настроить `flask` приложение с помощью `app.config` . Для запросов с URL-адресов, которые не нужно отслеживанию, добавьте их в `EXCLUDELIST_PATHS` .

    ```python
    app.config['OPENCENSUS'] = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>",
            )''',
            'EXCLUDELIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```

## <a name="tracking-pyramid-applications"></a>Отслеживание пирамидальных приложений

1. Скачайте и установите `opencensus-ext-django` из [PyPI](https://pypi.org/project/opencensus-ext-pyramid/) и выполните инструментирование приложения с помощью `pyramid` анимации. Входящие запросы, отправляемые в ваше `pyramid` приложение, будут относиться к отслеживанию.

    ```python
    def main(global_config, **settings):
        config = Configurator(settings=settings)
    
        config.add_tween('opencensus.ext.pyramid'
                         '.pyramid_middleware.OpenCensusTweenFactory')
    ```

2. Вы можете настроить `pyramid` анимацию непосредственно в коде. Для запросов с URL-адресов, которые не нужно отслеживанию, добавьте их в `EXCLUDELIST_PATHS` .

    ```python
    settings = {
        'OPENCENSUS': {
            'TRACE': {
                'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
                'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                    connection_string="InstrumentationKey=<your-ikey-here>",
                )''',
                'EXCLUDELIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
            }
        }
    }
    config = Configurator(settings=settings)
    ```

## <a name="tracking-fastapi-applications"></a>Отслеживание Фастапи приложений

Опенценсус не имеет расширения для Фастапи. Чтобы создать собственное по промежуточного слоя Фастапи, выполните следующие действия.

1. Требуются следующие зависимости: 
    - [фастапи](https://pypi.org/project/fastapi/)
    - [увикорн](https://pypi.org/project/uvicorn/)

2. Добавьте по [промежуточного слоя фастапи](https://fastapi.tiangolo.com/tutorial/middleware/). Убедитесь, что установлен сервер типа Span: `span.span_kind = SpanKind.SERVER` .

3. Запустите приложение. Вызовы, выполняемые в приложении Фастапи, должны быть автоматически отслеживанием, и данные телеметрии должны быть записаны непосредственно в Azure Monitor.

    ```python 
    # Opencensus imports
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer
    from opencensus.trace.span import SpanKind
    from opencensus.trace.attributes_helper import COMMON_ATTRIBUTES
    # FastAPI imports
    from fastapi import FastAPI, Request
    # uvicorn
    import uvicorn

    app = FastAPI()

    HTTP_URL = COMMON_ATTRIBUTES['HTTP_URL']
    HTTP_STATUS_CODE = COMMON_ATTRIBUTES['HTTP_STATUS_CODE']

    # fastapi middleware for opencensus
    @app.middleware("http")
    async def middlewareOpencensus(request: Request, call_next):
        tracer = Tracer(exporter=AzureExporter(connection_string=f'InstrumentationKey={APPINSIGHTS_INSTRUMENTATIONKEY}'),sampler=ProbabilitySampler(1.0))
        with tracer.span("main") as span:
            span.span_kind = SpanKind.SERVER

            response = await call_next(request)

            tracer.add_attribute_to_current_span(
                attribute_key=HTTP_STATUS_CODE,
                attribute_value=response.status_code)
            tracer.add_attribute_to_current_span(
                attribute_key=HTTP_URL,
                attribute_value=str(request.url))

        return response

    @app.get("/")
    async def root():
        return "Hello World!"

    if __name__ == '__main__':
        uvicorn.run("example:app", host="127.0.0.1", port=5000, log_level="info")
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Схема сопоставления приложений](./app-map.md)
* [Доступность](./monitor-web-app-availability.md)
* [Поиск](./diagnostic-search.md)
* [Запрос к журналу (Analytics)](../logs/log-query-overview.md)
* [Диагностика транзакции](./transaction-diagnostics.md)

