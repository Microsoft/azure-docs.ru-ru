---
title: Просмотр метрик с помощью Azure Monitor
description: В этой статье показано, как отслеживать метрики с помощью портал Azureных диаграмм и Azure CLI.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/17/2021
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: 306381465919d3fde7135315b69d450a496c2898
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609734"
---
# <a name="monitor-media-services-metrics"></a>Monitor Media Services metrics (Мониторинг метрик службы мультимедиа)

[!INCLUDE [media services api v3 logo](../includes/v3-hr.md)]

[Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview.md) позволяет отслеживать метрики и журналы диагностики, которые помогут понять, как работают приложения. Подробное описание этой функции и сведения о том, почему следует использовать метрики и журналы диагностики служб мультимедиа Azure, см. в статье [мониторинг метрик служб мультимедиа и журналов диагностики](monitor-media-services-data-reference.md).

Azure Monitor предоставляет несколько способов взаимодействия с метриками, включая их диаграмму на портале, доступ к ним через REST API или запросы с помощью Azure CLI. В этой статье показано, как отслеживать метрики с помощью портал Azureных диаграмм и Azure CLI.

## <a name="prerequisites"></a>Предварительные требования

- [Создание учетной записи Служб мультимедиа](../create-account-howto.md)
- Просмотр  [мониторинга метрик и журналов диагностики служб мультимедиа](monitor-media-services-data-reference.md)

## <a name="view-metrics-in-azure-portal"></a>Просмотр метрик в портал Azure

1. Войдите на портал Azure по адресу https://portal.azure.com.
1. Перейдите к учетной записи служб мультимедиа Azure и выберите **метрики**.
1. Щелкните поле **область** и выберите ресурс, который требуется отслеживать.

    Справа появится окно **Выбор области** со списком доступных ресурсов. В этом случае вы увидите:

    * &lt;Имя учетной записи служб мультимедиа&gt;
    * &lt;Имя учетной записи служб мультимедиа имя &gt; / &lt; конечной точки потоковой передачи&gt;
    * &lt;имя учетной записи хранения&gt;

    Фильтр выберите ресурс и нажмите кнопку **Применить**. Дополнительные сведения о поддерживаемых ресурсах и метриках см. в статье [мониторинг метрик служб мультимедиа](monitor-media-services-data-reference.md).

    > [!NOTE]
    > Чтобы переключиться между ресурсами, которые необходимо отслеживать, снова щелкните поле **источник** и повторите этот шаг.

1. Необязательно. присвойте диаграмме имя (измените имя, нажав карандаш вверху).
1. Добавьте метрики, которые необходимо просмотреть.
1. Вы можете закрепить диаграмму на панели мониторинга.

## <a name="view-metrics-with-azure-cli"></a>Просмотр метрик с помощью Azure CLI

Чтобы получить метрики "исходящего трафика" с Azure CLI, выполните следующую `az monitor metrics` команду:

```azurecli-interactive
az monitor metrics list --resource \
   "/subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/Microsoft.Media/mediaservices/<Media Services account name>/streamingendpoints/<streaming endpoint name>" \
   --metric "Egress"
```

Чтобы получить другие метрики, замените "исходящий" на имя метрики, которое вас интересует.

## <a name="see-also"></a>См. также раздел

- [Метрики Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/data-platform.md)
- [Создание, Просмотр оповещений метрик и управление ими с помощью Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-metric.md).

## <a name="next-steps"></a>Дальнейшие действия

[Журналы диагностики](../media-services-diagnostic-logs-howto.md)
