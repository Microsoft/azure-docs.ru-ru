---
title: Выполнение фоновых задач с помощью веб-заданий
description: Узнайте, как использовать веб-задания для выполнения фоновых задач в службе приложений Azure. Выберите один из множества форматов скриптов и запустите их с помощью выражений CRON.
author: ggailey777
ms.assetid: af01771e-54eb-4aea-af5f-f883ff39572b
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: glenga
ms.reviewer: msangapu;suwatch;pbatum;naren.soni
ms.custom: seodec18
zone_pivot_groups: app-service-webjob
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 0d49323e2bc3c0522b1fb9ad49ffcc14f476e2dc
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102452804"
---
# <a name="run-background-tasks-with-webjobs-in-azure-app-service"></a>Выполнение фоновых задач с помощью веб-заданий в службе приложений Azure

Концепция выполнения [фоновых задач](./webjobs-create-ieux-conceptual.md) в Azure предоставляется с веб-заданиями службы приложений Azure. Сведения о развертывании <abbr title="Программа или сценарий в том же экземпляре, что и веб-приложение, приложение API или мобильное приложение.">веб-задания;</abbr> с помощью [портал Azure](https://portal.azure.com) для отправки исполняемого файла или скрипта. 

Ниже перечислены три поддерживаемые веб-задания.

* **Непрерывно**: запускается немедленно, обычно выполняется в бесконечном цикле.
* **Запланировано**: запуск из запланированного триггера
* **Вручную**: запуск с ручного триггера

::: zone pivot="webjob-type-continuous"

[!INCLUDE [Continuous](./includes/webjobs-create-ieux-continuous.md)]

::: zone-end

::: zone pivot="webjob-type-scheduled"

[!INCLUDE [Scheduled](./includes/webjobs-create-ieux-scheduled.md)]

::: zone-end

::: zone pivot="webjob-type-manual"

[!INCLUDE [Manual](./includes/webjobs-create-ieux-manual.md)]

::: zone-end
   
## <a name="next-steps"></a><a name="NextSteps"></a> Дальнейшие действия

* [Дополнительные сведения о фоновых задачах в качестве веб-заданий](./webjobs-create-ieux-conceptual.md)
* [Просмотр журнала веб-заданий](./webjobs-create-ieux-view-log.md)

* Использование [пакета SDK](https://github.com/Azure/azure-webjobs-sdk/wiki) для веб-заданий для упрощения многих задач программирования

* Изучение [разработки и развертывания веб-заданий с помощью Visual Studio](webjobs-dotnet-deploy-vs.md)
