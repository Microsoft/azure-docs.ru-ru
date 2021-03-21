---
title: Мониторинг экземпляров контейнеров
description: Как отслеживать потребление контейнерами вычислительных ресурсов, таких как ЦП и память, в службе "Экземпляры контейнеров Azure".
ms.topic: article
ms.date: 12/17/2020
ms.openlocfilehash: ae9725ffe66bebbed26745c311b2ada07d5d2c00
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589297"
---
# <a name="monitor-container-resources-in-azure-container-instances"></a>Мониторинг ресурсов контейнеров в службе "Экземпляры контейнеров Azure"

[Azure Monitor][azure-monitoring] предоставляет информацию о том, какие вычислительные ресурсы используют экземпляры контейнеров. Эти данные об использовании ресурсов помогут определить оптимальные настройки ресурса для конкретных групп контейнеров. Azure Monitor также предоставляет метрики для отслеживания сетевой активности в экземплярах контейнеров.

В этом документе описывается, как происходит сбор метрик Azure Monitor для экземпляров контейнеров с помощью портала Azure и Azure CLI.

> [!IMPORTANT]
> Метрики Azure Monitor в Экземплярах контейнеров Azure сейчас доступны в предварительной версии с некоторыми [ограничениями](#preview-limitations). Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования][terms-of-use]. Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.

## <a name="preview-limitations"></a>Ограничения предварительной версии

Сейчас метрики Azure Monitor доступны только для контейнеров Linux.

## <a name="available-metrics"></a>Доступные метрики

Azure Monitor предоставляет следующие [метрики для Экземпляров контейнеров Azure][supported-metrics]. Эти метрики доступны как для отдельных контейнеров, так и для групп контейнеров. По умолчанию метрики объединяются в виде средних значений.

- **Загрузка ЦП** измеряется в **миллиардах**. 
  - Одним из милликоре является 1/1000th ядра ЦП, поэтому 500 миллиардах представляет использование ядра ЦП 0,5.
- **Использование памяти** в байтах
- **Количество байт в сети, полученных** в секунду
- **Передано байт в сети** за секунду 

## <a name="get-metrics---azure-portal"></a>Получение метрик на портале Azure

При создании группы контейнеров в Azure Monitor данные отображаются на портале Azure. Чтобы просмотреть метрики для группы контейнеров, перейдите к странице **Обзор** для группы контейнеров. Здесь вы можете видеть предварительно созданные диаграммы для каждой из доступных метрик.

![Сдвоенные диаграммы][dual-chart]

В группе контейнеров, содержащей несколько контейнеров, используйте [измерение][monitor-dimension] для вывода метрик по контейнеру. Чтобы создать диаграмму с метриками по отдельному контейнеру, выполните следующие действия:

1. На странице **Обзор** выберите одну из диаграмм метрик, таких как **ЦП**. 
1. Нажмите кнопку **Применить разделение**, а затем выберите **Имя контейнера**.

![Снимок экрана показывает метрики для экземпляра контейнера с выбранным параметром применить разделение и имя контейнера.][dimension]

## <a name="get-metrics---azure-cli"></a>Получение метрик через Azure CLI

Метрики для экземпляров контейнеров можно также получить с помощью Azure CLI. Для начала получите идентификатор группы контейнеров, используя приведенную ниже команду. Замените `<resource-group>` именем нужной группы ресурсов, а `<container-group>` — именем группы контейнеров.


```console
CONTAINER_GROUP=$(az container show --resource-group <resource-group> --name <container-group> --query id --output tsv)
```

Следующая команда возвращает метрики потребления **ЦП**:

```azurecli
az monitor metrics list --resource $CONTAINER_GROUP --metric CPUUsage --output table
```

```output
Timestamp            Name       Average
-------------------  ---------  ---------
2020-12-17 23:34:00  CPU Usage
. . .
2020-12-18 00:25:00  CPU Usage
2020-12-18 00:26:00  CPU Usage  0.4
2020-12-18 00:27:00  CPU Usage  0.0
```

Измените значение параметра `--metric` в команде для получения других [поддерживаемых метрик][supported-metrics]. Например, используйте следующую команду для возвращения метрики потребления **памяти**. 

```azurecli
az monitor metrics list --resource $CONTAINER_GROUP --metric MemoryUsage --output table
```

```output
Timestamp            Name          Average
-------------------  ------------  ----------
2019-04-23 22:59:00  Memory Usage
2019-04-23 23:00:00  Memory Usage
2019-04-23 23:01:00  Memory Usage  0.0
2019-04-23 23:02:00  Memory Usage  8859648.0
2019-04-23 23:03:00  Memory Usage  9181184.0
2019-04-23 23:04:00  Memory Usage  9580544.0
2019-04-23 23:05:00  Memory Usage  10280960.0
2019-04-23 23:06:00  Memory Usage  7815168.0
2019-04-23 23:07:00  Memory Usage  7739392.0
2019-04-23 23:08:00  Memory Usage  8212480.0
2019-04-23 23:09:00  Memory Usage  8159232.0
2019-04-23 23:10:00  Memory Usage  8093696.0
```

В группе с несколькими контейнерами можно добавить измерение `containerName`, чтобы получить метрики по отдельным контейнерам.

```azurecli
az monitor metrics list --resource $CONTAINER_GROUP --metric MemoryUsage --dimension containerName --output table
```

```output
Timestamp            Name          Containername             Average
-------------------  ------------  --------------------  -----------
2019-04-23 22:59:00  Memory Usage  aci-tutorial-app
2019-04-23 23:00:00  Memory Usage  aci-tutorial-app
2019-04-23 23:01:00  Memory Usage  aci-tutorial-app      0.0
2019-04-23 23:02:00  Memory Usage  aci-tutorial-app      16834560.0
2019-04-23 23:03:00  Memory Usage  aci-tutorial-app      17534976.0
2019-04-23 23:04:00  Memory Usage  aci-tutorial-app      18329600.0
2019-04-23 23:05:00  Memory Usage  aci-tutorial-app      19742720.0
2019-04-23 23:06:00  Memory Usage  aci-tutorial-app      14786560.0
2019-04-23 23:07:00  Memory Usage  aci-tutorial-app      14651392.0
2019-04-23 23:08:00  Memory Usage  aci-tutorial-app      15470592.0
2019-04-23 23:09:00  Memory Usage  aci-tutorial-app      15450112.0
2019-04-23 23:10:00  Memory Usage  aci-tutorial-app      15339520.0
2019-04-23 22:59:00  Memory Usage  aci-tutorial-sidecar
2019-04-23 23:00:00  Memory Usage  aci-tutorial-sidecar
2019-04-23 23:01:00  Memory Usage  aci-tutorial-sidecar  0.0
2019-04-23 23:02:00  Memory Usage  aci-tutorial-sidecar  884736.0
2019-04-23 23:03:00  Memory Usage  aci-tutorial-sidecar  827392.0
2019-04-23 23:04:00  Memory Usage  aci-tutorial-sidecar  831488.0
2019-04-23 23:05:00  Memory Usage  aci-tutorial-sidecar  819200.0
2019-04-23 23:06:00  Memory Usage  aci-tutorial-sidecar  843776.0
2019-04-23 23:07:00  Memory Usage  aci-tutorial-sidecar  827392.0
2019-04-23 23:08:00  Memory Usage  aci-tutorial-sidecar  954368.0
2019-04-23 23:09:00  Memory Usage  aci-tutorial-sidecar  868352.0
2019-04-23 23:10:00  Memory Usage  aci-tutorial-sidecar  847872.0
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о мониторинге в Azure вы найдете [в этой статье][azure-monitoring].

Сведения о создании [оповещения о метриках][metric-alert] для получения уведомлений, когда значение метрики для Экземпляров контейнеров Azure достигает порогового значения.

<!-- IMAGES -->
[cpu-chart]: ./media/container-instances-monitor/cpu-multi.png
[dimension]: ./media/container-instances-monitor/dimension.png
[dual-chart]: ./media/container-instances-monitor/metrics.png
[memory-chart]: ./media/container-instances-monitor/memory-multi.png

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-monitoring]: ../azure-monitor/overview.md
[metric-alert]: ..//azure-monitor/alerts/alerts-metric.md
[monitor-dimension]: ../azure-monitor/essentials/data-platform-metrics.md#multi-dimensional-metrics
[supported-metrics]: ../azure-monitor/essentials/metrics-supported.md#microsoftcontainerinstancecontainergroups
