---
title: Общие сведения о Linkerd
description: Получите общие сведения о Linkerd
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 3181be62a14ec1b3450bd181172b5323ca176427
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "77593773"
---
# <a name="linkerd"></a>Linkerd

## <a name="overview"></a>Обзор

[Linkerd][linkerd] — это простая в использовании и упрощенная сетка служб.

## <a name="architecture"></a>Architecture

Linkerd предоставляет плоскость данных, состоящую из ултралигхт [Linkerd][linkerd-proxy] специального прокси-сервера сидекарс. Интеллектуальные прокси-серверы контролируют весь входящий и исходящий сетевой трафик для приложений и рабочих нагрузок сетки. Прокси-серверы также предоставляют метрики через конечные точки метрик [Prometheus][prometheus] .

Плоскость управления управляет конфигурацией и агрегированными данными телеметрии с помощью следующих [компонентов][linkerd-architecture]:

- **Контроллер** — предоставляет API, который управляет ИНТЕРФЕЙСОМ командной строки Linkerd и панелью мониторинга. Предоставляет конфигурацию для прокси-серверов.

- **Коснитесь** — установите контрольные значения запросов и ответов в режиме реального времени.

- **Identity** — предоставляет возможности идентификации и безопасности, позволяющие использовать mTLS между службами.

- **Веб** — предоставляет панель мониторинга Linkerd.


На следующей схеме архитектуры показано взаимодействие компонентов из плоскости данных и плоскости управления.


![Общие сведения о компонентах и архитектуре Linkerd.](media/servicemesh/linkerd/about-architecture.png)


## <a name="selection-criteria"></a>Условия выбора

При оценке Linkerd для рабочих нагрузок важно понимать и учитывать следующие аспекты:

- [Принципы проектирования](#design-principles)
- [Capabilities](#capabilities)
- [Сценарии](#scenarios)


### <a name="design-principles"></a>Принципы проектирования

Ниже [приведены принципы разработки][design-principles] проекта Linkerd.

- **Не отключайте** простоту — он должен быть простым в использовании и понятен.

- **Минимизация требований к ресурсам** . накладываются минимальные затраты на производительность и ресурсы.

- **Только работа** — не нарушайте работу существующих приложений и не требует сложной конфигурации.


### <a name="capabilities"></a>Возможности

Linkerd предоставляет следующий набор возможностей:

- **Сетка** — встроенный параметр отладки

- **Управление трафиком** — разделение, время ожидания, повторы, входящий трафик

- **Безопасность** — шифрование (mTLS) — сертификаты переворачиваются каждые 24 часа

- **Наблюдаемость** — Золотой метрики, TAP, трассировка, профили служб и метрики маршрута, веб-панель мониторинга с графами топологии, Prometheus, grafana


### <a name="scenarios"></a>Сценарии

Linkerd хорошо подходит и предлагается для следующих сценариев:

- Простой в использовании только с основным набором требований к возможностям

- Низкая задержка, низкая нагрузка, с целью сосредоточиться на наблюдаемой и простой системе управления трафиком


## <a name="next-steps"></a>Дальнейшие действия

В следующей документации описывается, как можно установить Linkerd в службе Kubernetes Azure (AKS).

> [!div class="nextstepaction"]
> [Установка Linkerd в службе Kubernetes Azure (AKS)][linkerd-install]

Кроме того, вы можете дополнительно исследовать функции и архитектуру Linkerd:

- [Функции Linkerd][linkerd-features]
- [Архитектура Linkerd][linkerd-architecture]

<!-- LINKS - external -->
[linkerd]: https://linkerd.io/2/overview/
[linkerd-architecture]: https://linkerd.io/2/reference/architecture/
[linkerd-features]: https://linkerd.io/2/features/
[design-principles]: https://linkerd.io/2/design-principles/
[linkerd-proxy]: https://github.com/linkerd/linkerd2-proxy

[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/

<!-- LINKS - internal -->
[linkerd-install]: ./servicemesh-linkerd-install.md