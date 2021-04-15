---
title: Включить файл
description: включить файл
services: functions
author: nzthiago
ms.service: azure-functions
ms.topic: include
ms.date: 02/21/2018
ms.author: nzthiago
ms.custom: include file
ms.openlocfilehash: eca2d3359614875e5bff0c9bb67f006f0a8cdba1
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "77198332"
---
## <a name="function-app-timeout-duration"></a><a name="timeout"></a>Время ожидания приложения-функции 

Время ожидания приложения-функции определяется свойством `functionTimeout` в файле проекта [host.json](../articles/azure-functions/functions-host-json.md#functiontimeout). В следующей таблице показаны значения по умолчанию и максимальные значения в минутах для обоих планов и различных версий среды выполнения.

| План | Версия среды выполнения | По умолчанию | Максимум |
|------|---------|---------|---------|
| Потребление | 1.x | 5 | 10 |
| Потребление | 2.x | 5 | 10 |
| Потребление | 3.x | 5 | 10 |
| Premium | 1.x | 30 | Без ограничений |
| Premium | 2.x | 30 | Без ограничений |
| Premium | 3.x | 30 | Без ограничений |
| Служба приложений | 1.x | Без ограничений | Без ограничений |
| Служба приложений | 2.x | 30 | Без ограничений |
| Служба приложений | 3.x | 30 | Без ограничений |

> [!NOTE] 
> Независимо от параметра времени ожидания приложения-функции 230 секунд — это максимальное время, в течение которого функция, активируемая HTTP, может реагировать на запрос. Это определяется [стандартным временем простоя Azure Load Balancer](../articles/app-service/faq-availability-performance-application-issues.md#why-does-my-request-time-out-after-230-seconds). При более длительной обработке рассмотрите возможность использования [асинхронного шаблона Устойчивых функций](../articles/azure-functions/durable/durable-functions-overview.md#async-http) или [отложите работу и получите немедленный ответ](../articles/azure-functions/functions-best-practices.md#avoid-long-running-functions).
