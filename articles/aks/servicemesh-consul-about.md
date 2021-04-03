---
title: Общие сведения о Consul
description: Получите общие сведения о Consul
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 77cb6a693c5a73d0498b0acf9bc4ad8cc9f8f32f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "83774005"
---
# <a name="consul"></a>Consul

## <a name="overview"></a>Обзор

[Consul][consul] — это решение для взаимодействия службы из нескольких центров обработки данных, которое позволяет подключать и защищать службы на разных платформах среды выполнения. Компонент [Connect][consul-features] предоставляет функции сетки служб.

## <a name="architecture"></a>Architecture

Consul реализует плоскость данных, в которую по умолчанию входят [службы-расширения][consul-sidecar] на основе [Envoy][envoy-proxy]. Consul использует архитектуру подключаемых прокси-серверов. Интеллектуальные прокси-серверы контролируют весь входящий и исходящий сетевой трафик для приложений и рабочих нагрузок сетки.

Плоскость управления отвечает за конфигурацию и политики, распространяемые с помощью следующих [компонентов][consul-architecture]:

- **серверный агент Consul**, который поддерживает состояние кластера Consul в режиме сервера;

- **агент клиента Consul**, который работает в упрощенном режиме. На каждом вычислительном узле должен выполняться агент клиента. Этот клиент распространяет настройки и политики между рабочими нагрузками и конфигурацией Consul. 

На следующей схеме архитектуры показано взаимодействие компонентов из плоскости данных и плоскости управления.

![Обзор компонентов и архитектуры Consul.](media/servicemesh/consul/about-architecture.png)


## <a name="selection-criteria"></a>Условия выбора

Чтобы оценить, подходит ли Consul для ваших рабочих нагрузок, важно понимать и учитывать следующие аспекты.

- [Принципы работы Consul](#consul-principles)
- [Capabilities](#capabilities)
- [Сценарии](#scenarios)


### <a name="consul-principles"></a>Принципы работы Consul

Следующие принципы являются [основой][consul-principles] проекта Consul:

- **На основе API** — все настройки и политики выражаются в коде.

- **Запуск и подключение из любого места** — объединение рабочих нагрузок, выполняемых на любых платформах среды выполнения (Kubernetes, виртуальные машины, бессерверные решения).

- **Расширение и интеграция** — безопасное подключение рабочих нагрузок в любой инфраструктуре.


### <a name="capabilities"></a>Возможности

Consul предоставляет следующие возможности:

- **Сетка** — шлюз (на нескольких центрах обработки данных), виртуальные машины (узлы вне кластера), синхронизация служб, встроенные возможности для отладки.

- **Прокси** — Envoy, встроенный прокси-сервер, подключаемый прокси-сервер уровня 4 для рабочих нагрузок Windows.

- **Управление трафиком** — маршрутизация, разделение, разрешение.

- **Политика** — намерения, списки управления доступом.

- **Безопасность** — авторизация, проверка подлинности, шифрование, удостоверения на основе SPIFFE, внешний ЦС (Key Vault), управление сертификатами и смена сертификатов.

- **Наблюдаемость** — метрики, панель мониторинга пользовательского интерфейса, prometheus, grafana.


### <a name="scenarios"></a>Сценарии

Consul хорошо подходит и рекомендуется для следующих сценариев:

- расширение существующих подключенных рабочих нагрузок Consul;

- обеспечение соответствия требованиям в управлении сертификатами;

- сетка служб на нескольких кластерах;

- включение рабочих нагрузок на основе виртуальных машин в сетку служб.



## <a name="next-steps"></a>Дальнейшие действия

В следующих документах описывается, как установить Consul в Службе Azure Kubernetes (AKS).

> [!div class="nextstepaction"]
> [Установка Consul в Службе Azure Kubernetes (AKS)][consul-install]

А здесь вы можете подробнее изучить функции и архитектуру Consul:

- [Руководства по началу работы с Consul][consul-getting-started]
- [Компоненты Consul][consul-features]
- [Архитектура Consul][consul-architecture]
- [Принципы работы Connect (Consul)][consul-how-connect-works]

<!-- LINKS - external -->
[consul]: https://www.consul.io/mesh.html
[consul-features]: https://www.consul.io/docs/connect/index.html
[consul-architecture]: https://www.consul.io/docs/internals/architecture.html
[consul-sidecar]: https://www.consul.io/docs/connect/proxies.html
[consul-how-connect-works]: https://www.consul.io/docs/connect/connect-internals.html
[consul-principles]: https://www.consul.io/
[consul-getting-started]:https://learn.hashicorp.com/consul?track=gs-consul-service-mesh#gs-consul-service-mesh

[envoy-proxy]: https://www.envoyproxy.io/
[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/

<!-- LINKS - internal -->
[consul-install]: ./servicemesh-consul-install.md
