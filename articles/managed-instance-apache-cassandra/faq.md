---
title: Часто задаваемые вопросы об использовании Управляемого экземпляра Azure для Apache Cassandra на портале Azure
description: Часто задаваемые вопросы об использовании Управляемого экземпляра Azure для Apache Cassandra. В этой статье описано, когда следует использовать управляемые экземпляры, какие преимущества они предоставляют, пределы их пропускной способности, поддерживаемые регионы и другие сведения о конфигурации.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 03/02/2021
ms.openlocfilehash: 1ba2b7d648c86912118b83a566bf2eb0800baee2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101747858"
---
# <a name="frequently-asked-questions-about-azure-managed-instance-for-apache-cassandra-preview"></a>Часто задаваемые вопросы об использовании Управляемого экземпляра Azure для Apache Cassandra (предварительная версия)

В этой статье представлены часто задаваемые вопросы об использовании Управляемого экземпляра Azure для Apache Cassandra. Вы узнаете, когда следует использовать управляемые экземпляры, какие у них есть преимущества, пределы пропускной способности, поддерживаемые регионы и сведения об их конфигурации.

> [!IMPORTANT]
> Служба "Управляемый экземпляр Azure для Apache Cassandra" сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="general-faq"></a>Общие вопросы и ответы

### <a name="what-are-the-benefits-azure-managed-instance-for-apache-cassandra"></a>Какие преимущества дает Управляемый экземпляр Azure для Apache Cassandra?

База данных Apache Cassandra будет правильным выбором, если требуется масштабируемость и высокий уровень доступности без ущерба для производительности. Это отличная платформа для критически важных данных, так как она обеспечивает линейную масштабируемость и подтвержденную отказоустойчивость на типичном оборудовании или в облачной инфраструктуре. Управляемый экземпляр Azure для Apache Cassandra — это служба для управления экземплярами центров обработки данных Apache Cassandra с открытым кодом, развернутых в Azure.

Ее можно развернуть полностью в облаке или в составе гибридного кластера в облачной и локальной средах. Эта служба отлично подходит в тех случаях, когда вам важно сохранить широкие возможности настройки и контроля, предоставляемые Apache Cassandra с открытым кодом, без излишних издержек на обслуживание.

### <a name="why-should-i-use-this-service-instead-of-azure-cosmos-db-cassandra-api"></a>Почему эта служба лучше, чем API Cassandra для Azure Cosmos DB?

Управляемый экземпляр Azure для Apache Cassandra предоставляется разработчиками Azure Cosmos DB. Это автономная управляемая служба для развертывания, обслуживания и масштабирования центров обработки данных и кластеров Apache Cassandra с открытым кодом. В свою очередь, [Cassandra API для Azure Cosmos DB](../cosmos-db/cassandra-introduction.md) предоставляется в формате PaaS (платформа как услуга), то есть как дополнительный уровень взаимодействия для сетевого протокола Apache Cassandra. Если вы вам нужна платформа, которая будет вести себя точно так же, как обычный кластер Apache Cassandra, следует выбрать службу управляемого экземпляра. Подробные сведения см. в статье [Сравнение Управляемого экземпляра Azure для Apache Cassandra с API Cassandra для Azure Cosmos DB](compare-cosmosdb-managed-instance.md).

### <a name="is-azure-managed-instance-for-apache-cassandra-dependent-on-azure-cosmos-db"></a>Есть ли у Управляемого экземпляра Azure для Apache Cassandra зависимость от Azure Cosmos DB?

Нет, не существует архитектурных зависимостей между Управляемым экземпляром Azure для Apache Cassandra и серверной частью Azure Cosmos DB. 

#### <a name="can-i-deploy-azure-managed-instance-for-apache-cassandra-in-any-region"></a>Во всех ли регионах можно развернуть Управляемый экземпляр Azure для Apache Cassandra?

Этот управляемый экземпляр в период предварительной версии будет доступен в ограниченном количестве регионов.

### <a name="what-are-the-storage-and-throughput-limits-of-azure-managed-instance-for-apache-cassandra"></a>Каковы ограничения хранилища и пропускной способности для Управляемого экземпляра Azure для Apache Cassandra?

Эти ограничения зависят от выбранных ценовых категорий для виртуальных машин.

### <a name="what-is-the-cost-of-azure-managed-instance-for-apache-cassandra"></a>Сколько стоит Управляемый экземпляр Azure для Apache Cassandra?

Плата за управляемый экземпляр зависит от стоимости базовой виртуальной машины и включает небольшую наценку. Дополнительные сведения см. на [странице с расценками](https://azure.microsoft.com/pricing/details/managed-instance-apache-cassandra/).

### <a name="can-i-use-yaml-file-settings-to-configure-behavior"></a>Можно ли использовать параметры в YAML-файле для настройки поведения?

Да, вы можете внедрить конфигурации в формате YAML-файлов в процессе развертывания шаблона Azure Resource Manager.

### <a name="how-can-i-monitor-infrastructure-along-with-throughput"></a>Как можно отслеживать инфраструктуру и пропускную способность?

Для отслеживания активности в кластере размещается сервер [Prometheus](https://prometheus.io/docs/introduction/overview/), который предоставляет свою конечную точку. Она сохраняет данные за 10 минут, но не более 10 ГБ. Чтобы использовать этот мониторинг, необходимо настроить [федерацию](https://prometheus.io/docs/prometheus/latest/federation/) и подходящее средство для панели мониторинга, например Grafana.

### <a name="does-azure-managed-instance-for-apache-cassandra-provide-full-backups"></a>Предоставляет ли Управляемый экземпляр Azure для Apache Cassandra полные резервные копии?

Да, он предоставляет полные резервные копии в службе хранилища Azure и восстанавливает их в новом кластере.

### <a name="how-can-i-migrate-data-from-my-existing-apache-cassandra-cluster-to-azure-managed-instance-for-apache-cassandra"></a>Как перенести данные из существующего кластера Apache Cassandra в Управляемый экземпляр Azure для Apache Cassandra?

Управляемый экземпляр Azure для Apache Cassandra поддерживает все функции Apache Cassandra для репликации данных и потоковой передачи данных между центрами обработки данных.

### <a name="can-i-pair-an-on-premises-apache-cassandra-cluster-with-the-azure-managed-instance-for-apache-cassandra"></a>Можно ли связать локальный кластер Apache Cassandra с Управляемым экземпляром Azure для Apache Cassandra?

Да, вы можете настроить гибридный кластер с внедренными в виртуальную сеть центрами обработки данных Azure, которые развертывает эта служба. Центры обработки данных Управляемого экземпляра могут взаимодействовать с локальными центрами обработки данных в пределах одного круга кластеров.

### <a name="where-can-i-give-feedback-on-azure-managed-instance-for-apache-cassandra-features"></a>Где можно оставить отзыв о возможностях Управляемого экземпляра Azure для Apache Cassandra?

Вы можете отправить отзывы через интерфейс [отзывов пользователей](https://feedback.azure.com/forums/263030-azure-cosmos-db?category_id=398548), указав категорию "Managed Apache Cassandra" (Управляемый экземпляр Apache Cassandra).

Чтобы устранить проблему, связанную с учетной записью, отправьте [запрос в службу поддержки](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) на портале Azure.

## <a name="deployment-specific-faq"></a>Вопросы и ответы, относящиеся к развертыванию

### <a name="will-the-managed-instance-support-node-addition-cluster-status-and-node-status-commands"></a>Будет ли управляемый экземпляр поддерживать команды добавления узлов, состояния кластера и состояния узла?

В Azure CLI поддерживаются все *команды чтения* средства Nodetool, например `status`. Но такие операции, как *добавление узлов*, недоступны, так как мы контролируем работоспособность узлов в управляемом экземпляре. В гибридном режиме вы можете подключаться к кластеру с помощью *Nodetool*. Но мы не рекомендуем использовать средство Nodetool, так как это может нарушить стабильность кластера. Также оно может привести к нарушению соглашений об уровне обслуживания, относящихся к работоспособности центров обработки данных для управляемых экземпляров в кластере.

### <a name="what-happens-with-various-settings-for-table-metadata"></a>А как насчет параметров для метаданных таблиц?

Полностью поддерживаются все параметры метаданных таблиц, такие как фильтр раскрытия, кэширование, возможность восстановления чтения, gc_grace и memtable_flush_period для сжатия, как в любой локальной среде Apache Cassandra.

## <a name="next-steps"></a>Дальнейшие действия

Часто задаваемые вопросы о других API см. в следующих статьях:

* [Создание кластера управляемых экземпляров на портале Azure](create-cluster-portal.md)
* [Развертывание управляемого кластера Apache Spark с Azure Databricks](deploy-cluster-databricks.md)
* [Управление ресурсами службы "Управляемый экземпляр Azure для ресурсов Apache Cassandra" с помощью Azure CLI](manage-resources-cli.md)