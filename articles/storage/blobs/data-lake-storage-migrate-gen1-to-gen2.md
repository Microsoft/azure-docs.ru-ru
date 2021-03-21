---
title: Перенос Azure Data Lake Storage из Gen1 в Gen2
description: Перенос Azure Data Lake Storage из Gen1 в Gen2, который основан на хранилище BLOB-объектов Azure, и предоставляет набор возможностей, предназначенных для аналитики больших данных.
author: normesta
ms.topic: how-to
ms.author: normesta
ms.date: 03/11/2020
ms.service: storage
ms.reviewer: rukmani-msft
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: 875c046406b7989a39f09dde82640ed5567b6b3e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101714873"
---
# <a name="migrate-azure-data-lake-storage-from-gen1-to-gen2"></a>Перенос Azure Data Lake Storage из Gen1 в Gen2

Вы можете перенести данные, рабочие нагрузки и приложения из Data Lake Storage 1-го поколения в Data Lake Storage 2-го поколения.

Azure Data Lake Storage 2-го поколения построены в [хранилище BLOB-объектов Azure](storage-blobs-introduction.md) и предоставляет набор возможностей, предназначенных для аналитики больших данных. [Data Lake Storage 2-го поколения](https://azure.microsoft.com/services/storage/data-lake-storage/) объединяет функции из [Azure Data Lake Storage 1-го поколения](../../data-lake-store/index.yml), такие как семантика файловой системы, каталог и безопасность на уровне файлов, и масштабирование с использованием экономичных, многоуровневого хранилища, возможностей высокой доступности и аварийного восстановления из [хранилища BLOB-объектов Azure](storage-blobs-introduction.md).

> [!NOTE]
> Для упрощения чтения в этой статье используется термин *Gen1* для ссылки на Azure Data Lake Storage 1-го поколения, а термин *Gen2* — для ссылки на Azure Data Lake Storage 2-го поколения.

## <a name="recommended-approach"></a>Рекомендуемый подход

Для перехода на Gen2 рекомендуется использовать следующий подход.

: heavy_check_mark: шаг 1. Оценка готовности

: heavy_check_mark: шаг 2. Подготовка к миграции

: heavy_check_mark: шаг 3. Перенос рабочих нагрузок данных и приложений

: heavy_check_mark: шаг 4. прямую миграцию из Gen1 в Gen2

> [!NOTE]
> Gen1 и Gen2 — это разные службы, не существует возможности обновления на месте, преднамеренная миграция требует усилий. 

### <a name="step-1-assess-readiness"></a>Шаг 1. Оценка готовности

1. Сведения о [предложении Data Lake Storage 2-го поколения](https://azure.microsoft.com/services/storage/data-lake-storage/); Это преимущества, затраты и общая архитектура. 

2. [Сравните возможности](#gen1-gen2-feature-comparison) Gen1 с возможностями Gen2. 

3. Ознакомьтесь со списком [известных проблем](data-lake-storage-known-issues.md) , чтобы оценить зазоры в работе.

4. Gen2 поддерживает такие функции хранилища BLOB-объектов, как [ведение журнала диагностики](../common/storage-analytics-logging.md), [уровни доступа](storage-blob-storage-tiers.md)и [политики управления жизненным циклом хранилища BLOB-объектов](storage-lifecycle-management-concepts.md). Если вам интересно использовать какие – либо из этих функций, ознакомьтесь с [текущим уровнем поддержки](./data-lake-storage-supported-blob-storage-features.md).

5. Проверьте текущее состояние [поддержки экосистемы Azure](./data-lake-storage-multi-protocol-access.md) , чтобы убедиться, что Gen2 поддерживает любые службы, от которых зависит ваше решение.

### <a name="step-2-prepare-to-migrate"></a>Шаг 2. Подготовка к миграции

1. Найдите наборы данных, которые предстоит перенести.

   Воспользуйтесь этой возможностью для очистки наборов данных, которые больше не используются. Если вы не планируете выполнять миграцию всех данных одновременно, уделите это время для того, чтобы выделить логические группы данных, которые можно перенести поэтапно.
   
2. Определите влияние миграции на ваш бизнес.

   Например, подумайте, можно ли позволить себе время простоя во время миграции. Эти рекомендации помогут определить подходящий шаблон переноса и выбрать наиболее подходящие средства.

3. Создайте план миграции. 

   Мы рекомендуем использовать эти [шаблоны миграции](#migration-patterns). Можно выбрать один из этих шаблонов, объединить их вместе или разработать собственный собственный шаблон.

### <a name="step-3-migrate-data-workloads-and-applications"></a>Шаг 3. Перенос данных, рабочих нагрузок и приложений

Перенос данных, рабочих нагрузок и приложений с помощью предпочтительного шаблона. Рекомендуется выполнять инкрементную проверку сценариев.

1. [Создайте учетную запись хранения](../common/storage-account-create.md) и включите функцию иерархического пространства имен. 

2. Перенос данных. 

3. Настройте [службы в рабочих нагрузках](./data-lake-storage-supported-azure-services.md) так, чтобы они указывали на конечную точку Gen2. 
   
4. Обновите приложения для использования API-интерфейсов Gen2. См. руководства для [.NET](data-lake-storage-directory-file-acl-dotnet.md), [Java](data-lake-storage-directory-file-acl-java.md), [Python](data-lake-storage-directory-file-acl-python.md), [JavaScript](data-lake-storage-directory-file-acl-javascript.md) и [других](/rest/api/storageservices/data-lake-storage-gen2)компонентов. 
   
5. Обновите скрипты для использования [командлетов PowerShell](data-lake-storage-directory-file-acl-powershell.md)Data Lake Storage 2-го поколения и [Azure CLI команд](data-lake-storage-directory-file-acl-cli.md).
   
6. Поиск ссылок URI, содержащих строку `adl://` в файлах кода или в записных книжках, Apache Hive файлы HQL или любой другой файл, используемый в составе рабочих нагрузок. Замените эти ссылки на [универсальный код ресурса (URI) Gen2, отформатированный](data-lake-storage-introduction-abfs-uri.md) для новой учетной записи хранения. Например: URI Gen1: `adl://mydatalakestore.azuredatalakestore.net/mydirectory/myfile` может стать `abfss://myfilesystem@mydatalakestore.dfs.core.windows.net/mydirectory/myfile` . 

7. Настройте безопасность учетной записи, чтобы включить [роли Azure](../common/storage-auth-aad-rbac-portal.md), [безопасность на уровне файлов и папок](data-lake-storage-access-control.md), а также [брандмауэры службы хранилища Azure и виртуальные сети](../common/storage-network-security.md).

### <a name="step-4-cutover-from-gen1-to-gen2"></a>Шаг 4. прямую миграцию из Gen1 в Gen2

Убедившись, что ваши приложения и рабочие нагрузки стабильны в Gen2, вы можете начать использовать Gen2 для удовлетворения бизнес-сценариев. Отключите все оставшиеся конвейеры, работающие на Gen1, и выведите учетную запись Gen1. 

<a id="gen1-gen2-feature-comparison"></a>

## <a name="gen1-vs-gen2-capabilities"></a>Возможности Gen1 VS Gen2

В этой таблице сравниваются возможности Gen1 и Gen2.

|Область |Поколение 1   |Поколение 2 |
|---|---|---|
|Упорядочение данных|[Иерархическое пространство имен хранилища Azure Data Lake Storage Gen2 (предварительная версия)](data-lake-storage-namespace.md)<br>Поддержка файлов и папок|[Иерархическое пространство имен хранилища Azure Data Lake Storage Gen2 (предварительная версия)](data-lake-storage-namespace.md)<br>Поддержка контейнеров, файлов и папок |
|Геоизбыточность| [LRS](../common/storage-redundancy.md#locally-redundant-storage)| [LRS](../common/storage-redundancy.md#locally-redundant-storage), [ZRS](../common/storage-redundancy.md#zone-redundant-storage), [GRS](../common/storage-redundancy.md#geo-redundant-storage), [RA-GRS](../common/storage-redundancy.md#read-access-to-data-in-the-secondary-region) |
|Аутентификация|[Удостоверение, управляемое AAD](../../active-directory/managed-identities-azure-resources/overview.md)<br>[Субъекты-службы](../../active-directory/develop/app-objects-and-service-principals.md)|[Удостоверение, управляемое AAD](../../active-directory/managed-identities-azure-resources/overview.md)<br>[Субъекты-службы](../../active-directory/develop/app-objects-and-service-principals.md)<br>[Ключ общего доступа](/rest/api/storageservices/authorize-with-shared-key)|
|Авторизация|Управление — [Azure RBAC](../../role-based-access-control/overview.md)<br>Данные — [списки ACL](data-lake-storage-access-control.md)|Управление — [Azure RBAC](../../role-based-access-control/overview.md)<br>Данные —  [списки управления доступом](data-lake-storage-access-control.md), [Azure RBAC](../../role-based-access-control/overview.md) |
|Шифрование — неактивных данных|Серверная часть — ключи, управляемые [корпорацией Майкрософт](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) или [управляемые клиентом](../common/customer-managed-keys-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|Серверная часть — ключи, управляемые [корпорацией Майкрософт](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) или [управляемые клиентом](../common/customer-managed-keys-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Поддержка виртуальной сети|[интеграция с виртуальной сетью](../../data-lake-store/data-lake-store-network-security.md)|[Конечные точки службы](../common/storage-network-security.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json), [частные конечные точки](../common/storage-private-endpoints.md)|
|Взаимодействие с разработчиками|[Остальное](../../data-lake-store/data-lake-store-data-operations-rest-api.md), [.NET](../../data-lake-store/data-lake-store-data-operations-net-sdk.md), [Java](../../data-lake-store/data-lake-store-get-started-java-sdk.md), [Python](../../data-lake-store/data-lake-store-data-operations-python.md), [PowerShell](../../data-lake-store/data-lake-store-get-started-powershell.md), [Azure CLI](../../data-lake-store/data-lake-store-get-started-cli-2.0.md)|Общедоступная версия — [остальное](/rest/api/storageservices/data-lake-storage-gen2), [.NET](data-lake-storage-directory-file-acl-dotnet.md), [Java](data-lake-storage-directory-file-acl-java.md), [Python](data-lake-storage-directory-file-acl-python.md)<br>Общедоступная Предварительная версия — [JavaScript](data-lake-storage-directory-file-acl-javascript.md), [PowerShell](data-lake-storage-directory-file-acl-powershell.md), [Azure CLI](data-lake-storage-directory-file-acl-cli.md)|
|Журналы ресурсов|Классические журналы<br>[Встроенная Azure Monitor](../../data-lake-store/data-lake-store-diagnostic-logs.md)|[Классические журналы](../common/storage-analytics-logging.md) — общедоступные<br>[Встроенная Azure Monitor](monitor-blob-storage.md) — Предварительная версия|
|Экосистема|[HDInsight (3,6)](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md), [Azure Databricks (3,1 и выше)](https://docs.databricks.com/data/data-sources/azure/azure-datalake.html), [Azure синапсе Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-load-from-azure-data-lake-store.md), [ADF](../../data-factory/load-azure-data-lake-store.md)|[HDInsight (3,6, 4,0)](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md), [Azure Databricks (5,1 и более поздние версии)](/azure/databricks/data/data-sources/azure/azure-datalake-gen2), [Azure синапсе Analytics](../../azure-sql/database/vnet-service-endpoint-rule-overview.md), [ADF](../../data-factory/load-azure-data-lake-storage-gen2.md)|

<a id="migration-patterns"></a>

## <a name="gen1-to-gen2-patterns"></a>Gen1 к Gen2 шаблонам

Выберите шаблон миграции, а затем измените этот шаблон по необходимости.

|||
|---|---|
|**Прогноз и сдвиг**|Самый простой шаблон. Идеально подходит, если конвейеры данных могут предоставить простой.|
|**Добавочное копирование**|Аналогичен *точности и сдвигу*, но с меньшим временем простоя. Идеально подходит для больших объемов данных, которые можно копировать дольше.|
|**Два конвейера**|Идеально подходит для конвейеров, которые не могут позволить себе время простоя.|
|**Двунаправленная синхронизация**|Аналогично *двойному конвейеру*, но с более поэтапным подходом, который подходит для более сложных конвейеров.|

Давайте подробнее рассмотрим каждый шаблон.
 
### <a name="lift-and-shift-pattern"></a>Шаблон точности и сдвига

Это самый простой шаблон. 

1. Останавливает все операции записи в Gen1.

2. Перемещение данных из Gen1 в Gen2. Мы рекомендуем использовать [фабрику данных Azure](../../data-factory/connector-azure-data-lake-storage.md). Списки ACL копируются вместе с данными.

3. Укажите, что операции и рабочие нагрузки принимают Gen2.

4. Списание Gen1.

Ознакомьтесь с нашим образцом кода для шаблона "прогноз" и "сдвиг" в [примере миграции "прогноз" и "сдвиг](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Lift%20and%20Shift/README.md)".

> [!div class="mx-imgBorder"]
> ![шаблон точности и сдвига](./media/data-lake-storage-migrate-gen1-to-gen2/lift-and-shift.png)

#### <a name="considerations-for-using-the-lift-and-shift-pattern"></a>Рекомендации по использованию шаблона точности и сдвига

: heavy_check_mark: прямую миграцию из Gen1 в Gen2 для всех рабочих нагрузок одновременно.

: heavy_check_mark: ожидание простоя во время миграции и прямую миграцию периода.

: heavy_check_mark. идеально подходит для конвейеров, которые могут допустить время простоя, и все приложения могут быть обновлены одновременно.

### <a name="incremental-copy-pattern"></a>Шаблон добавочного копирования

1. Начало перемещения данных из Gen1 в Gen2. Мы рекомендуем использовать [фабрику данных Azure](../../data-factory/connector-azure-data-lake-storage.md). Списки ACL копируются вместе с данными.

2. Добавочное копирование новых данных из Gen1.

3. После копирования всех данных отключите все операции записи в Gen1 и наведите рабочие нагрузки на Gen2.

4. Списание Gen1.

Ознакомьтесь с нашим образцом кода для шаблона добавочного копирования в нашем [примере миграции добавочных копий](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Incremental/README.md).


> [!div class="mx-imgBorder"]
> ![Шаблон добавочного копирования](./media/data-lake-storage-migrate-gen1-to-gen2/incremental-copy.png)

#### <a name="considerations-for-using-the-incremental-copy-pattern"></a>Рекомендации по использованию шаблона добавочного копирования:

: heavy_check_mark: прямую миграцию из Gen1 в Gen2 для всех рабочих нагрузок одновременно.

: heavy_check_mark: ожидание простоя только в период прямую миграцию.

: heavy_check_mark: идеально подходит для конвейеров, где все приложения обновляются одновременно, но для копирования данных требуется больше времени.

### <a name="dual-pipeline-pattern"></a>Шаблон двойного конвейера

1. Перемещение данных из Gen1 в Gen2. Мы рекомендуем использовать [фабрику данных Azure](../../data-factory/connector-azure-data-lake-storage.md). Списки ACL копируются вместе с данными.

2. Прием новых данных как в Gen1, так и в Gen2.

3. Наведите рабочие нагрузки на Gen2.

4. Останавливает все операции записи в Gen1, а затем списание Gen1.

Ознакомьтесь с нашим образцом кода для шаблона двойного конвейера в нашем [примере переноса двух конвейеров](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Dual%20pipeline/README.md).

> [!div class="mx-imgBorder"]
> ![Шаблон двойного конвейера](./media/data-lake-storage-migrate-gen1-to-gen2/dual-pipeline.png)

#### <a name="considerations-for-using-the-dual-pipeline-pattern"></a>Рекомендации по использованию шаблона двойного конвейера:

: heavy_check_mark: Конвейеры Gen1 и Gen2 работают параллельно.

: heavy_check_mark: поддерживает нулевое время простоя.

: heavy_check_mark: идеально подходит в ситуациях, когда рабочие нагрузки и приложения не могут позволить какие бы то ни было простои, и вы можете принять их в обеих учетных записях хранения.

### <a name="bi-directional-sync-pattern"></a>Шаблон двунаправленной синхронизации

1. Настройте двунаправленную репликацию между Gen1 и Gen2. Мы рекомендуем [вандиско](https://docs.wandisco.com/bigdata/wdfusion/adls/). Она предлагает функцию восстановления для существующих данных.

3. После завершения всех перемещений следует завершить все операции записи в Gen1 и отключить двунаправленную репликацию.

4. Списание Gen1.

Ознакомьтесь с примером кода для шаблона двунаправленной синхронизации в нашем [примере переноса двунаправленной синхронизации](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Bi-directional/README.md).

> [!div class="mx-imgBorder"]
> ![Шаблон двунаправленного письма](./media/data-lake-storage-migrate-gen1-to-gen2/bidirectional-sync.png)

#### <a name="considerations-for-using-the-bi-directional-sync-pattern"></a>Рекомендации по использованию шаблона двунаправленной синхронизации:

: heavy_check_mark: идеально подходит для сложных сценариев, в которых задействовано большое количество конвейеров и зависимостей, где поэтапный подход может быть более осмысленным.  

: heavy_check_mark: высокая интенсивность миграции, но она обеспечивает параллельную поддержку Gen1 и Gen2.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте о различных частях настройки безопасности для учетной записи хранения. Ознакомьтесь с [руководством по безопасности службы хранилища Azure](./security-recommendations.md).
- Оптимизируйте производительность Data Lake Store. См. раздел [оптимизация Azure Data Lake Storage 2-го поколения для повышения производительности](data-lake-storage-performance-tuning-guidance.md) .
- Ознакомьтесь с рекомендациями по управлению Data Lake Store. См. рекомендации [по использованию Azure Data Lake Storage 2-го поколения](data-lake-storage-best-practices.md)