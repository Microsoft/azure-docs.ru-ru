---
title: Перераспределение сегментов — масштабирование (Цитус) — база данных Azure для PostgreSQL
description: Равномерное распределение сегментов между серверами для повышения производительности
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 11/17/2020
ms.openlocfilehash: b69c4fc3ff16cf30181a3529f61af7126b92950b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026392"
---
# <a name="rebalance-shards-in-hyperscale-citus-server-group"></a>Перераспределение сегментов в группе серверов "горизонтальное масштабирование" (Цитус)

Чтобы воспользоваться преимуществами новых добавленных узлов, необходимо перераспределить сегменты распределенной таблицы, что означает перемещение [некоторых сегментов из](concepts-hyperscale-distributed-data.md#shards)существующих узлов в новые. Сначала убедитесь, что подготовка новых рабочих ролей успешно завершена. Затем запустите подсистему балансировки сегментов, подключившись к узлу координатора кластера с psql и запустив:

```sql
SELECT rebalance_table_shards('distributed_table_name');
```

Функция [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards) перераспределяет все таблицы в группе совместного [размещения](concepts-hyperscale-colocation.md) таблицы с именем в аргументе. Поэтому нет необходимости вызывать функцию для каждой распределенной таблицы, просто вызвать ее в репрезентативной таблице из каждой группы совместного размещения.

**Дальнейшие действия**


- Дополнительные сведения о [параметрах производительности](concepts-hyperscale-configuration-options.md)группы серверов.
- Увеличение или уменьшение [масштаба группы серверов](howto-hyperscale-scale-grow.md)
- См. справочные материалы по [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards)
