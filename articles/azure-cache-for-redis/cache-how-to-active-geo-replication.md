---
title: Настройка активной георепликации для кэша предприятия Azure для экземпляров Redis
description: Узнайте, как реплицировать кэш Azure для экземпляров Redis Enterprise в регионах Azure.
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 02/08/2021
ms.author: yegu
ms.openlocfilehash: 3fe3131263d3cf1984eae1692854d8d6bcd2746a
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105109494"
---
# <a name="configure-active-geo-replication-for-enterprise-azure-cache-for-redis-instances-preview"></a>Настройка активной георепликации для кэша предприятия Azure для экземпляров Redis (Предварительная версия)

Из этой статьи вы узнаете, как настроить активный кэш Azure с георепликацией с помощью портал Azure.

Активная Георепликация группирует два или более кэша предприятия Azure для экземпляров Redis в один кэш, охватывающий регионы Azure. Все экземпляры действуют как локальные первичные реплики. Приложение определяет, какие экземпляры следует использовать для запросов на чтение и запись.

## <a name="create-or-join-an-active-geo-replication-group"></a>Создание или присоединение активной группы георепликации

> [!IMPORTANT]
> Активная Георепликация должна быть включена во время создания кэша Azure для Redis.
>
>

1. На вкладке **Дополнительно** в новом пользовательском интерфейсе создания **кэша Redis** выберите **Корпоративный** для **политики кластеризации**.

    ![Настройка активной георепликации](./media/cache-how-to-active-geo-replication/cache-active-geo-replication-not-configured.png)

1. Нажмите кнопку **настроить** , чтобы настроить **активную георепликацию**.

1. Создайте новую группу репликации для первого экземпляра кэша или выберите имеющуюся в списке.

    ![Связывание кэшей](./media/cache-how-to-active-geo-replication/cache-active-geo-replication-new-group.png)

1. Нажмите кнопку **настроить** , чтобы завершить работу.

    ![Активная Георепликация настроена](./media/cache-how-to-active-geo-replication/cache-active-geo-replication-configured.png)

1. Дождитесь успешного создания первого кэша. Повторите описанные выше шаги для каждого дополнительного экземпляра кэша в группе георепликации.

## <a name="remove-from-an-active-geo-replication-group"></a>Удаление из активной группы георепликации

Чтобы удалить экземпляр кэша из активной группы георепликации, просто удалите экземпляр. Остальные экземпляры будут автоматически перенастроены.

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения о кэше Azure для функций Redis.

* [Кэш Azure для уровней служб Redis](cache-overview.md#service-tiers)
* [Высокий уровень доступности кэша Azure для Redis](cache-high-availability.md)
