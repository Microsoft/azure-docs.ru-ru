---
title: Ограничения и квоты подписки Azure
description: В этой статье приводится перечень наиболее распространенных ограничений, относящихся к подписке Azure и различным службам, квот и границ. Эта статья содержит сведения о том, как увеличить пределы и максимальное значение.
ms.topic: conceptual
ms.date: 03/15/2021
ms.openlocfilehash: 4dd62c926cce765ebc2b21378308650e313716ff
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103471017"
---
# <a name="azure-subscription-and-service-limits-quotas-and-constraints"></a>Подписка Azure, границы, квоты и ограничения службы

В этом документе указаны некоторые из наиболее распространенных ограничений Microsoft Azure, которые иногда называются квотами.

Дополнительные сведения о ценах Azure см. в статье [Общие сведения о ценах на Azure](https://azure.microsoft.com/pricing/). Здесь вы можете оценить затраты с помощью [калькулятора цен](https://azure.microsoft.com/pricing/calculator/). Вы также можете посетить страницу сведений о ценах для конкретной службы, например [виртуальных машин Windows](https://azure.microsoft.com/pricing/details/virtual-machines/#Windows). Советы по управлению затратами приведены в статье [Предотвращение непредвиденных расходов с помощью функции выставления счетов и управления затратами в Azure](../../cost-management-billing/cost-management-billing-overview.md).

## <a name="managing-limits"></a>Управление ограничениями

> [!NOTE]
> Некоторые службы имеют регулируемые ограничения.
>
> Если служба не имеет регулируемых ограничений, в следующих таблицах используется **ограничение** заголовка. В таких случаях значения по умолчанию и максимальное ограничение одинаковы.
>
> Если ограничение может быть скорректировано, таблицы включают заголовки **ограничения по умолчанию** и **максимального предела** . Это ограничение может быть вызвано выше ограничения по умолчанию, но не выше максимального ограничения.
>
> Если вы хотите увеличить лимит или квоту выше предельного значения по умолчанию, отправьте [запрос в службу поддержки клиентов через Интернет](../templates/error-resource-quota.md)бесплатно.
>
> Термины *мягкое ограничение* и *жесткое ограничение* часто используются для описания текущего, изменяемого ограничения (мягкого ограничения) и максимального ограничения (жесткого ограничения). Если ограничение не регулируется, то мягкое ограничение не будет ограничено, а только жестким ограничением.
>

[Бесплатные пробные подписки](https://azure.microsoft.com/offers/ms-azr-0044p) не подходят для увеличения лимита или квоты. При наличии [бесплатной пробной версии подписки](https://azure.microsoft.com/offers/ms-azr-0044p) ее можно обновить до подписки [с оплатой по мере использования](https://azure.microsoft.com/offers/ms-azr-0003p/). Дополнительные сведения см. в статье [Обновление бесплатной пробной подписки Azure до подписки с оплатой по мере использования](../../cost-management-billing/manage/upgrade-azure-subscription.md) и [бесплатной пробной подписки](https://azure.microsoft.com/free/free-account-faq).

Некоторые ограничения управляются на региональном уровне.

В качестве примера используем квоты виртуальных процессоров. Чтобы запросить увеличение квоты с поддержкой виртуальных ЦП, необходимо решить, сколько виртуальных ЦП вы хотите использовать в разных регионах. Затем вы выполните Специальный запрос квоты виртуальных ЦП для группы ресурсов Azure для нужных вам объемов и регионов. Если вам нужно использовать 30 виртуальных ЦП в Западной Европе для запуска приложения, вы запрашиваете 30 виртуальных ЦП в Западной Европе. Квота виртуальных ЦП не увеличивается в любом другом регионе. в Западной Европе есть 30-виртуальных цпная квота.

В результате определите, какие квоты группы ресурсов Azure должны быть для вашей рабочей нагрузки в одном регионе. Затем запросите эту сумму в каждом регионе, в котором требуется выполнить развертывание. Сведения о том, как определить текущие квоты для конкретных регионов, см. в разделе [Устранение ошибок квот ресурсов](../templates/error-resource-quota.md).

## <a name="general-limits"></a>Общие ограничения

Ограничения по именам ресурсов см. в статье [правила именования и ограничения для ресурсов Azure](resource-name-rules.md).

Сведения об ограничениях на операции записи и чтения API Resource Manager см. в разделе [Регулирование запросов Resource Manager](request-limits-and-throttling.md).

### <a name="management-group-limits"></a>Ограничения группы управления

К [группам управления](../../governance/management-groups/overview.md)применяются следующие ограничения.

[!INCLUDE [management-group-limits](../../../includes/management-group-limits.md)]

### <a name="subscription-limits"></a>Ограничения подписки

При использовании Azure Resource Manager и групп ресурсов Azure действуют следующие ограничения.

[!INCLUDE [azure-subscription-limits-azure-resource-manager](../../../includes/azure-subscription-limits-azure-resource-manager.md)]

### <a name="resource-group-limits"></a>Ограничения группы ресурсов

[!INCLUDE [azure-resource-groups-limits](../../../includes/azure-resource-groups-limits.md)]

## <a name="active-directory-limits"></a>Ограничения Active Directory

[!INCLUDE [AAD-service-limits](../../../includes/active-directory-service-limits-include.md)]

## <a name="api-management-limits"></a>Ограничения управления API

[!INCLUDE [api-management-service-limits](../../../includes/api-management-service-limits.md)]

## <a name="app-service-limits"></a>Ограничения службы приложений

Следующие ограничения Службы приложения включают ограничения для веб-приложений, мобильных приложений, приложений API.

[!INCLUDE [azure-websites-limits](../../../includes/azure-websites-limits.md)]

## <a name="automation-limits"></a>Ограничения автоматизации

[!INCLUDE [automation-limits](../../../includes/azure-automation-service-limits.md)]

## <a name="azure-app-configuration"></a>конфигурация приложения Azure;

[!INCLUDE [app-configuration-limits](../../../includes/app-configuration-limits.md)]

## <a name="azure-cache-for-redis-limits"></a>Ограничения кэша Azure для Redis

[!INCLUDE [redis-cache-service-limits](../../../includes/redis-cache-service-limits.md)]

## <a name="azure-cloud-services-limits"></a>Ограничения облачных служб Azure

[!INCLUDE [azure-cloud-services-limits](../../../includes/azure-cloud-services-limits.md)]

## <a name="azure-cognitive-search-limits"></a>Ограничения Когнитивный поиск Azure

Ценовые категории определяют емкость и ограничения службы поиска. Существуют следующие категории:

* **Бесплатная** служба с несколькими клиентами, совместно используемая с другими подписчиками Azure, предназначена для оценки и небольших проектов разработки.
* **Базовый** предоставляет выделенные вычислительные ресурсы для небольших рабочих нагрузок в рабочей среде, поддерживая до трех реплик для обработки запросов с высоким уровнем доступности.
* **Стандартная**, включающая высокую плотность S1, S2, S3 и S3, предназначена для больших рабочих нагрузок. На уровне Standard существует несколько уровней, чтобы можно было выбрать конфигурацию ресурсов, которая лучше соответствует профилю рабочей нагрузки.

**Ограничения на одну подписку**

[!INCLUDE [azure-search-limits-per-subscription](../../../includes/azure-search-limits-per-subscription.md)]

**Ограничения на одну службу поиска**

[!INCLUDE [azure-search-limits-per-service](../../../includes/azure-search-limits-per-service.md)]

Дополнительные сведения об ограничениях на более детализированном уровне, таких как размер документа, количество запросов в секунду, ключи, запросы и ответы, см. [в статье ограничения служб в Azure когнитивный Поиск](../../search/search-limits-quotas-capacity.md).

## <a name="azure-cognitive-services-limits"></a>Ограничения Cognitive Services Azure

[!INCLUDE [azure-cognitive-services-limits](../../../includes/azure-cognitive-services-limits.md)]

## <a name="azure-cosmos-db-limits"></a>Ограничения Azure Cosmos DB

Ограничения Azure Cosmos DB см. [в разделе ограничения в Azure Cosmos DB](../../cosmos-db/concepts-limits.md).

## <a name="azure-data-explorer-limits"></a>Ограничения обозреватель данных Azure

[!INCLUDE [azure-data-explorer-limits](../../../includes/data-explorer-limits.md)]

## <a name="azure-database-for-mysql"></a>База данных Azure для MySQL

Дополнительные сведения об ограничениях базы данных Azure для MySQL см. в статье [Ограничения базы данных Azure для MySQL](../../mysql/concepts-limits.md).

## <a name="azure-database-for-postgresql"></a>База данных Azure для PostgreSQL

Сведения об ограничениях базы данных Azure для PostgreSQL см. в статье [Ограничения базы данных Azure для PostgreSQL](../../postgresql/concepts-limits.md).

## <a name="azure-functions-limits"></a>Ограничения функций Azure

[!INCLUDE [functions-limits](../../../includes/functions-limits.md)]

Дополнительные сведения см. в разделе [функции сравнение планов размещения](../../azure-functions/functions-scale.md).

## <a name="azure-kubernetes-service-limits"></a>Ограничения службы Kubernetes Azure

[!INCLUDE [container-service-limits](../../../includes/container-service-limits.md)]

## <a name="azure-machine-learning-limits"></a>Ограничения Машинное обучение Azure

Последние значения квоты на Машинное обучение Azure вычислений можно найти на [странице квота машинное обучение Azure](../../machine-learning/how-to-manage-quotas.md) .

## <a name="azure-maps-limits"></a>Ограничения службы "Карты Azure"

[!INCLUDE [maps-limits](../../../includes/maps-limits.md)]

## <a name="azure-monitor-limits"></a>Ограничения Azure Monitor

### <a name="alerts"></a>видны узлы

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-alerts.md)]

### <a name="action-groups"></a>Группы действий

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-action-groups.md)]

### <a name="autoscale"></a>Автомасштабирование

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-autoscale.md)]

### <a name="log-queries-and-language"></a>Ведение журнала запросов и языка

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-log-queries.md)]

### <a name="log-analytics-workspaces"></a>Рабочие области Log Analytics

[!INCLUDE [monitoring-limits](../../../includes/azure-monitor-limits-workspaces.md)]

### <a name="application-insights"></a>Application Insights

[!INCLUDE [monitoring-limits](../../../includes/application-insights-limits.md)]

## <a name="azure-policy-limits"></a>Ограничения политики Azure

[!INCLUDE [policy-limits](../../../includes/azure-policy-limits.md)]

## <a name="azure-quantum-limits"></a>Пределы тактовой задержки Azure

[!INCLUDE [quantum-limits](../../../includes/azure-quantum-limits.md)]

## <a name="azure-role-based-access-control-limits"></a>Ограничения управления доступом на основе ролей в Azure

[!INCLUDE [role-based-access-control-limits](../../../includes/role-based-access-control/limits.md)]

## <a name="azure-signalr-service-limits"></a>Ограничения службы Azure SignalR

[!INCLUDE [signalr-service-limits](../../../includes/signalr-service-limits.md)]

## <a name="azure-vmware-solution-limits"></a>Ограничения решения Azure VMware

[!INCLUDE [azure-vmware-solutions-limits](../../azure-vmware/includes/azure-vmware-solutions-limits.md)]

## <a name="backup-limits"></a>Ограничения резервного копирования

[!INCLUDE [azure-backup-limits](../../../includes/azure-backup-limits.md)]

## <a name="batch-limits"></a>Ограничения пакета

[!INCLUDE [azure-batch-limits](../../../includes/azure-batch-limits.md)]

## <a name="classic-deployment-model-limits"></a>Ограничения классической модели развертывания

Если вместо модели развертывания Azure Resource Manager используется классическая модель развертывания, применяются следующие ограничения.

[!INCLUDE [azure-subscription-limits](../../../includes/azure-subscription-limits.md)]

## <a name="container-instances-limits"></a>Ограничения для экземпляров контейнеров Azure

[!INCLUDE [container-instances-limits](../../../includes/container-instances-limits.md)]

## <a name="container-registry-limits"></a>Ограничения реестра контейнера

В следующей таблице описаны функции и ограничения [уровней служб](../../container-registry/container-registry-skus.md)"базовый", "Стандартный" и "Премиум".

[!INCLUDE [container-registry-limits](../../../includes/container-registry-limits.md)]

## <a name="content-delivery-network-limits"></a>Ограничения сети доставки содержимого

[!INCLUDE [cdn-limits](../../../includes/cdn-limits.md)]

## <a name="data-factory-limits"></a>Ограничения фабрики данных

[!INCLUDE [azure-data-factory-limits](../../../includes/azure-data-factory-limits.md)]

## <a name="data-lake-analytics-limits"></a>Ограничения Data Lake Analytics

[!INCLUDE [azure-data-lake-analytics-limits](../../../includes/azure-data-lake-analytics-limits.md)]

## <a name="data-lake-storage-limits"></a>Ограничения Data Lake Storage

[!INCLUDE [azure-data-lake-store-limits](../../../includes/azure-data-lake-store-limits.md)]

## <a name="data-share-limits"></a>Ограничения общего ресурса данных

[!INCLUDE [azure-data-share-limits](../../../includes/azure-data-share-limits.md)]

## <a name="database-migration-service-limits"></a>Ограничения Database Migration Service

[!INCLUDE [database-migration-service-limits](../../../includes/database-migration-service-limits.md)]

## <a name="digital-twins-limits"></a>Ограничения цифровых двойников

> [!NOTE]
> Некоторые области этой службы имеют регулируемые ограничения, а другие — нет. Это представлено в приведенных ниже таблицах с *изменяемым* столбцом?. Если ограничение может быть скорректировано *, это значение равно* *Да*.

[!INCLUDE [digital-twins-limits](../../../includes/digital-twins-limits.md)]

## <a name="event-grid-limits"></a>Ограничения сетки событий

[!INCLUDE [event-grid-limits](../../../includes/event-grid-limits.md)]

## <a name="event-hubs-limits"></a>Ограничения Центров событий

[!INCLUDE [azure-servicebus-limits](../../../includes/event-hubs-limits.md)]

## <a name="iot-central-limits"></a>Ограничения IoT Central
[!INCLUDE [iot-central-limits](../../../includes/iot-central-limits.md)]

## <a name="iot-hub-limits"></a>Пределы для Центра Интернета вещей

[!INCLUDE [azure-iothub-limits](../../../includes/iot-hub-limits.md)]

## <a name="iot-hub-device-provisioning-service-limits"></a>Ограничения службы подготовки устройств для Центра Интернета вещей

[!INCLUDE [azure-iotdps-limits](../../../includes/iot-dps-limits.md)]

## <a name="key-vault-limits"></a>Ограничения хранилища ключей

[!INCLUDE [key-vault-limits](../../../includes/key-vault-limits.md)]

## <a name="managed-identity-limits"></a>Ограничения управляемого удостоверения

[!INCLUDE [Managed-Identity-Limits](../../../includes/managed-identity-limits.md)]


## <a name="media-services-limits"></a>Ограничения служб мультимедиа

[!INCLUDE [azure-mediaservices-limits](../../../includes/media-servieces-limits-quotas-constraints.md)]

### <a name="media-services-v2-legacy"></a>Службы мультимедиа версии 2 (прежняя версия)

Ограничения, характерные для служб мультимедиа версии 2 (устаревшие), см. в разделе [службы мультимедиа версии 2 (устаревшие)](../../media-services/previous/media-services-quotas-and-limitations.md) .

## <a name="mobile-services-limits"></a>Ограничения мобильных служб

[!INCLUDE [mobile-services-limits](../../../includes/mobile-services-limits.md)]

## <a name="multi-factor-authentication-limits"></a>Ограничения многофакторной проверки подлинности

[!INCLUDE [azure-mfa-service-limits](../../../includes/azure-mfa-service-limits.md)]

## <a name="networking-limits"></a>Ограничения сети

[!INCLUDE [azure-virtual-network-limits](../../../includes/azure-virtual-network-limits.md)]

### <a name="expressroute-limits"></a>Ограничения ExpressRoute

[!INCLUDE [expressroute-limits](../../../includes/expressroute-limits.md)]

### <a name="virtual-network-gateway-limits"></a>Ограничения шлюза виртуальной сети

[!INCLUDE [virtual-network-gateway-limits](../../../includes/azure-virtual-network-gateway-limits.md)]

### <a name="nat-gateway-limits"></a>Ограничения шлюза NAT

[!INCLUDE [nat-gateway-limits](../../../includes/azure-nat-gateway-limits.md)]

### <a name="virtual-wan-limits"></a>Ограничения виртуальной глобальной сети

[!INCLUDE [virtual-wan-limits](../../../includes/virtual-wan-limits.md)]

### <a name="application-gateway-limits"></a>Ограничения шлюза приложений

Сведения в следующей таблице относится к номерам SKU версии 1, версии 2, ценовой категории "Стандартный" и WAF, если не указано иное.
[!INCLUDE [application-gateway-limits](../../../includes/application-gateway-limits.md)]

### <a name="network-watcher-limits"></a>Пределы наблюдателя за сетями

[!INCLUDE [network-watcher-limits](../../../includes/network-watcher-limits.md)]

### <a name="private-link-limits"></a>Ограничения для Приватного канала

[!INCLUDE [private-link-limits](../../../includes/private-link-limits.md)]

## <a name="purview-limits"></a>Ограничения зрения

Последние значения квот Azure зрения можно найти на [странице квота Azure зрения](../../purview/how-to-manage-quotas.md) .

### <a name="traffic-manager-limits"></a>Ограничения диспетчера трафика

[!INCLUDE [traffic-manager-limits](../../../includes/traffic-manager-limits.md)]

### <a name="azure-bastion-limits"></a>Ограничения Azure бастиона

[!INCLUDE [Azure Bastion limits](../../../includes/bastion-limits.md)]

### <a name="azure-dns-limits"></a>Ограничения Azure DNS

[!INCLUDE [dns-limits](../../../includes/dns-limits.md)]

### <a name="azure-firewall-limits"></a>Ограничения брандмауэра Azure

[!INCLUDE [azure-firewall-limits](../../../includes/firewall-limits.md)]

### <a name="azure-front-door-service-limits"></a>Ограничения службы Azure Front Door Service

[!INCLUDE [azure-front-door-service-limits](../../../includes/front-door-limits.md)]

## <a name="notification-hubs-limits"></a>Ограничения концентраторов уведомлений

[!INCLUDE [notification-hub-limits](../../../includes/notification-hub-limits.md)]

## <a name="service-bus-limits"></a>Ограничения служебной шины

[!INCLUDE [azure-servicebus-limits](../../../includes/service-bus-quotas-table.md)]

## <a name="site-recovery-limits"></a>Ограничения Site Recovery

[!INCLUDE [site-recovery-limits](../../../includes/site-recovery-limits.md)]

## <a name="sql-database-limits"></a>Ограничения базы данных SQL

Ограничения для базы данных SQL см. в разделе [ограничения ресурсов базы данных SQL для отдельных баз данных](../../azure-sql/database/resource-limits-vcore-single-databases.md), [ограничения ресурсов базы данных SQL для эластичных пулов и баз данных](../../azure-sql/database/resource-limits-vcore-elastic-pools.md)в составе пула и [ограничения ресурсов базы данных SQL для SQL управляемый экземпляр](../../azure-sql/managed-instance/resource-limits.md).

## <a name="azure-synapse-analytics-limits"></a>Ограничения Azure синапсе Analytics

Ограничения Azure синапсе Analytics см. в статье [ограничения ресурсов Azure синапсе](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md).

## <a name="azure-files-and-azure-file-sync"></a>Службы файлов и Синхронизация файлов Azure Azure
Дополнительные сведения об ограничениях для файлов и Синхронизация файлов Azure см. в статье [целевые показатели масштабируемости и производительности службы файлов Azure](../../storage/files/storage-files-scale-targets.md).

## <a name="storage-limits"></a>Ограничения хранилища

<!--like # storage accts -->
[!INCLUDE [azure-storage-account-limits-standard](../../../includes/azure-storage-account-limits-standard.md)]

Дополнительные сведения об ограничениях для учетных записей хранения уровня "Стандартный" см. в статье [целевые показатели масштабируемости для стандартных учетных записей хранения](../../storage/common/scalability-targets-standard-account.md).

### <a name="storage-resource-provider-limits"></a>Ограничения поставщика ресурсов хранилища

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

### <a name="azure-blob-storage-limits"></a>Ограничения хранилища BLOB-объектов Azure

[!INCLUDE [storage-blob-scale-targets](../../../includes/storage-blob-scale-targets.md)]

### <a name="azure-queue-storage-limits"></a>Ограничения для службы Azure "Хранилище очередей"

[!INCLUDE [storage-queues-scale-targets](../../../includes/storage-queues-scale-targets.md)]

### <a name="azure-table-storage-limits"></a>Ограничения для службы Azure "Хранилище таблиц"

[!INCLUDE [storage-tables-scale-targets](../../../includes/storage-tables-scale-targets.md)]

<!-- conceptual info about disk limits -- applies to unmanaged and managed -->
### <a name="virtual-machine-disk-limits"></a>Ограничения для дисков виртуальной машины

[!INCLUDE [azure-storage-limits-vm-disks](../../../includes/azure-storage-limits-vm-disks.md)]

Дополнительные сведения см. в статье [размеры виртуальных машин](../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

#### <a name="disk-encryption-sets"></a>Наборы шифрования дисков

Существует ограничение в 1000 наборов шифрования дисков на регион для каждой подписки. Дополнительные сведения см. в документации по шифрованию виртуальных машин [Linux](../../virtual-machines/disk-encryption.md#restrictions) или [Windows](../../virtual-machines/disk-encryption.md#restrictions) . Если вам нужно увеличить квоту, обратитесь в службу поддержки Azure.

### <a name="managed-virtual-machine-disks"></a>Управляемые диски виртуальной машины

[!INCLUDE [azure-storage-limits-vm-disks-managed](../../../includes/azure-storage-limits-vm-disks-managed.md)]

### <a name="unmanaged-virtual-machine-disks"></a>Неуправляемые диски виртуальной машины

[!INCLUDE [azure-storage-limits-vm-disks-standard](../../../includes/azure-storage-limits-vm-disks-standard.md)]

[!INCLUDE [azure-storage-limits-vm-disks-premium](../../../includes/azure-storage-limits-vm-disks-premium.md)]

## <a name="storsimple-system-limits"></a>Ограничения системы StorSimple

[!INCLUDE [storsimple-limits-table](../../../includes/storsimple-limits-table.md)]

## <a name="stream-analytics-limits"></a>Ограничения Stream Analytics

[!INCLUDE [stream-analytics-limits-table](../../../includes/stream-analytics-limits-table.md)]

## <a name="virtual-machines-limits"></a>Ограничения виртуальных машин

### <a name="virtual-machines-limits"></a>Ограничения виртуальных машин

[!INCLUDE [azure-virtual-machines-limits](../../../includes/azure-virtual-machines-limits.md)]

### <a name="virtual-machines-limits---azure-resource-manager"></a>Ограничения виртуальных машин — диспетчер ресурсов Azure

При использовании Azure Resource Manager и групп ресурсов Azure действуют следующие ограничения.

[!INCLUDE [azure-virtual-machines-limits-azure-resource-manager](../../../includes/azure-virtual-machines-limits-azure-resource-manager.md)]

### <a name="shared-image-gallery-limits"></a>Ограничения общей коллекции образов

В отношении развертывания ресурсов с использованием Общих коллекций ресурсов установлены следующие ограничения в расчете на подписку:

- 100 общих коллекций образов на одну подписку на один регион.
- 1000 определений образов на одну подписку на один регион.
- 10000 версий образов на одну подписку на один регион.

## <a name="virtual-machine-scale-sets-limits"></a>Ограничения для масштабируемых наборов виртуальных машин

[!INCLUDE [virtual-machine-scale-sets-limits](../../../includes/azure-virtual-machine-scale-sets-limits.md)]

## <a name="see-also"></a>См. также раздел

* [Общие сведения об ограничениях и росте Azure](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/)
* [Размеры виртуальных машин и облачных служб для Azure](../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Размеры для облачных служб Azure](../../cloud-services/cloud-services-sizes-specs.md)
* [Правила и ограничения именования для ресурсов Azure](resource-name-rules.md)
