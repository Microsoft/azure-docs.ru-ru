---
title: Сравнение функций продуктов сети доставки содержимого Azure (CDN)
description: Сведения о возможностях, которые поддерживает каждый продукт Azure доставки содержимого сети (CDN).
services: cdn
documentationcenter: ''
author: asudbring
ms.service: azure-cdn
ms.topic: overview
ms.date: 11/15/2019
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 9aa394cda245bd3a457a16c19660bfe08553d14d
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106058890"
---
# <a name="what-are-the-comparisons-between-azure-cdn-product-features"></a>Сравнение функций продуктов Azure CDN

Сеть доставки содержимого Microsoft Azure (CDN) включает в себя четыре продукта: 

* **Azure CDN уровня "Стандартный" от Майкрософт**;
* **Azure CDN уровня "Стандартный" от Akamai**;
* **Azure CDN уровня "Стандартный" от Verizon**;
* **Azure CDN уровня "Премиум" от Verizon**. 

В следующей таблице перечислены функции, доступные в каждом из продуктов.

| **Возможности для повышения производительности и оптимизации** | **Уровень "Стандартный" от корпорации Майкрософт** | **Akamai уровня "Стандартный"** | **Verizon уровня "Стандартный"** | **Verizon уровня "Премиум"** |
| --- | --- | --- | --- | --- |
| [Динамическое ускорение сайтов](./cdn-dynamic-site-acceleration.md)  | Доступны в [Azure Front Door Service](../frontdoor/front-door-overview.md) | **&#x2713;**  | **&#x2713;** | **&#x2713;** |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Динамическое ускорение сайтов — адаптивное сжатие изображений](./cdn-dynamic-site-acceleration.md#adaptive-image-compression-azure-cdn-from-akamai-only)  |  | **&#x2713;**  |  |  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Динамическое ускорение сайтов — предварительная выборка объектов](./cdn-dynamic-site-acceleration.md#object-prefetch-azure-cdn-from-akamai-only)  |  | **&#x2713;**  |  |  |
| [Оптимизация общей веб-доставки](./cdn-optimization-overview.md#general-web-delivery)  | **&#x2713;** | **&#x2713;** . Этот тип оптимизации используется для файлов со средним размером не более 10 МБ.  | **&#x2713;** |  **&#x2713;** |
| [Оптимизация потоковой передачи видео](./cdn-media-streaming-optimization.md)  | с помощью общей веб-доставки | **&#x2713;**  | с помощью общей веб-доставки |  с помощью общей веб-доставки |
| [Оптимизация больших файлов](./cdn-large-file-optimization.md)  | с помощью общей веб-доставки | **&#x2713;** . Этот тип оптимизации используется для файлов размером более 10 МБ.   | с помощью общей веб-доставки |  с помощью общей веб-доставки |
| Изменение типа оптимизации | |**&#x2713;** | | |
| Порт источника |Все TCP-порты |[Разрешенные порты источника](/previous-versions/azure/mt757337(v%3Dazure.100)#allowed-origin-ports) |Все TCP-порты |Все TCP-порты |
| [Глобальная балансировка нагрузки сервера (GSLB)](../traffic-manager/traffic-manager-load-balancing-azure.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Быстрая очистка.](cdn-purge-endpoint.md)  | **&#x2713;** |**&#x2713;** . "Очистить все" и "Очистка по подстановочным знакам" сейчас не поддерживается в Azure CDN от Akamai. |**&#x2713;** |**&#x2713;** |
| [Предварительная загрузка ресурса.](cdn-preload-endpoint.md)  |  | |**&#x2713;** |**&#x2713;** |
| Параметры кэша и заголовка (с использованием [правил кэширования](cdn-caching-rules.md))  |**✓** с помощью [стандартного обработчика правил](cdn-standard-rules-engine.md)  |**&#x2713;** |**&#x2713;** | |
| Настраиваемый модуль доставки контента на основе правил |**✓** с помощью [стандартного обработчика правил](cdn-standard-rules-engine.md)  | | |**✓** с помощью [обработчика правил](./cdn-verizon-premium-rules-engine.md) |
| Параметры кэша и заголовка  |**✓** с помощью [стандартного обработчика правил](cdn-standard-rules-engine.md) | | |**✓** с помощью [обработчика правил категории "Премиум"](./cdn-verizon-premium-rules-engine.md) |
| Перенаправление и перезапись URL-адреса |**✓** с помощью [стандартного обработчика правил](cdn-standard-rules-engine.md)  | | |**✓** с помощью [обработчика правил категории "Премиум"](./cdn-verizon-premium-rules-engine.md) |
| Правила для мобильных устройств  |**✓** с помощью [стандартного обработчика правил](cdn-standard-rules-engine.md) | | |**✓** с помощью [обработчика правил категории "Премиум"](./cdn-verizon-premium-rules-engine.md) |
| [Кэширование строк запроса.](cdn-query-string.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| Двойной стек IPv4/IPv6 | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Поддержка HTTP/2](cdn-http2.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
||||
 **Безопасность** | **Уровень "Стандартный" от корпорации Майкрософт** | **Akamai уровня "Стандартный"** | **Verizon уровня "Стандартный"** | **Verizon уровня "Премиум"** | 
| Поддержка HTTPS с конечной точкой CDN | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Пользовательский домен HTTPS](cdn-custom-ssl.md)  | **&#x2713;** | **&#x2713;** , для включения требуется прямое значение CNAME |**&#x2713;** |**&#x2713;** |
| [Поддержка пользовательских доменных имен.](cdn-map-content-to-custom-domain.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Геофильтрация](cdn-restrict-access-by-country.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Аутентификация на основе маркера безопасности](cdn-token-auth.md)  |  |  |  |**&#x2713;**| 
| [Защита от атак DDoS](https://www.us-cert.gov/ncas/tips/ST04-015)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Использование собственного сертификата](cdn-custom-ssl.md?tabs=option-2-enable-https-with-your-own-certificate#tlsssl-certificates) |**&#x2713;** |  | **&#x2713;** | **&#x2713;** |
| Поддерживаемые версии протокола TLS | TLS 1.2, TLS 1.0/1.1 — [с возможностью настройки](/rest/api/cdn/cdn/customdomains/enablecustomhttps#usermanagedhttpsparameters) | TLS 1.2 | TLS 1.2 | TLS 1.2 |
||||
| **Аналитика и отчетность** | **Уровень "Стандартный" от корпорации Майкрософт** | **Akamai уровня "Стандартный"** | **Verizon уровня "Стандартный"** | **Verizon уровня "Премиум"** | 
| [Журналы диагностики Azure](cdn-azure-diagnostic-logs.md)  | **&#x2713;** | **&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Основные отчеты из Verizon](cdn-analyze-usage-patterns.md)  |  | |**&#x2713;** |**&#x2713;** |
| [Настраиваемые отчеты из Verizon](cdn-verizon-custom-reports.md)  |  | |**&#x2713;** |**&#x2713;** |
| [Расширенные HTTP-отчеты.](cdn-advanced-http-reports.md)  |  | | |**&#x2713;** |
| [Статистика в режиме реального времени.](cdn-real-time-stats.md)  |  | | |**&#x2713;** |
| [Производительность граничного узла](cdn-edge-performance.md)  |  | | |**&#x2713;** |
| [Оповещения в реальном времени](cdn-real-time-alerts.md)  |  | | |**&#x2713;** |
||||
| **Простота использования** | **Уровень "Стандартный" от корпорации Майкрософт** | **Akamai уровня "Стандартный"** | **Verizon уровня "Стандартный"** | **Verizon уровня "Премиум"** | 
| Простая интеграция со службами Azure, такими как [служба хранилища](cdn-create-a-storage-account-with-cdn.md), [веб-приложения](cdn-add-to-web-app.md) и [Службы мультимедиа](../media-services/previous/media-services-portal-manage-streaming-endpoints.md).  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| Управление с помощью [REST API](/rest/api/cdn/), [.NET](cdn-app-dev-net.md), [Node.js](cdn-app-dev-node.md) или [PowerShell](cdn-manage-powershell.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Типы сжатия MIME](./cdn-improve-performance.md)  |Настраивается |Настраивается |Настраивается  |Настраивается  |
| Кодирование сжатия  |gzip, brotli |gzip |GZIP, DEFLATE, BZIP2, BROTLI  |GZIP, DEFLATE, BZIP2, BROTLI  |

## <a name="migration"></a>Миграция

Дополнительные сведения о переходе с профиля **Azure CDN ценовой категории "Стандартный" от Verizon** на профиль **Azure CDN ценовой категории "Премиум" от Verizon** см. [здесь](cdn-migrate.md). 

> [!NOTE]
> Существует вариант перехода с уровня "Стандартный" (Verizon) до уровня "Премиум" (Verizon), но механизм преобразования между другими продуктами в настоящее время отсутствует.

## <a name="next-steps"></a>Дальнейшие шаги

* Узнайте больше об [Azure CDN](cdn-overview.md).
