---
title: Часто задаваемые вопросы об Azure Synapse Link для Azure Cosmos DB
description: Получите ответы на часто задаваемые вопросы об Synapse Link для Azure Cosmos DB в таких областях, как выставление счетов, аналитическое хранилище, безопасность и срок жизни в аналитическом хранилище.
author: Rodrigossz
ms.author: rosouz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/30/2020
ms.custom: synapse-cosmos-db
ms.openlocfilehash: d871a33b6d8adbae179e592122878eee5db1c0b5
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104869010"
---
# <a name="frequently-asked-questions-about-azure-synapse-link-for-azure-cosmos-db"></a>Часто задаваемые вопросы об Azure Synapse Link для Azure Cosmos DB
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

Решение Azure Synapse Link для Azure Cosmos DB обеспечивает тесную интеграцию между Azure Cosmos DB и Azure Synapse Analytics. Оно позволяет клиентам выполнять аналитику операционных данных почти в реальном времени с полной изоляцией выполнения от транзакционных рабочих нагрузок и без применения конвейера извлечения, преобразования и загрузки. В этой статье даны ответы на часто задаваемые вопросы об Azure Synapse Link для Azure Cosmos DB.

## <a name="general-faq"></a>Общие вопросы и ответы

### <a name="is-azure-synapse-link-supported-for-all-azure-cosmos-db-apis"></a>Поддерживается ли ссылка Azure синапсе для всех Azure Cosmos DB API?

Ссылка Azure синапсе поддерживается для API Azure Cosmos DB SQL (Core) и для Azure Cosmos DB API для MongoDB. 

### <a name="is-azure-synapse-link-supported-for-multi-region-azure-cosmos-db-accounts"></a>Поддерживается ли ссылка Azure синапсе для Azure Cosmos DB учетных записей в нескольких регионах?

Да, для учетных записей Azure Cosmos в нескольких регионах данные, хранящиеся в аналитическом хранилище, также распределены глобально. Независимо от количества регионов записи (один или несколько), аналитические запросы, выполненные из Azure Synapse Analytics, могут обрабатываться из ближайших локальных регионов.

При планировании настройки учетной записи Azure Cosmos DB в нескольких регионах с поддержкой аналитических хранилищ рекомендуется добавить все необходимые регионы во время создания учетной записи.

### <a name="can-i-choose-to-enable-azure-synapse-link-for-only-certain-region-and-not-all-regions-in-a-multi-region-account-set-up"></a>Можно ли включить ссылку Azure синапсе только для определенного региона, а не всех регионов в учетной записи с несколькими регионами?

Если для учетной записи с несколькими регионами включена ссылка Azure синапсе, аналитическое хранилище создается во всех регионах. Базовые данные оптимизированы для обеспечения согласованности пропускной способности и транзакций в хранилище транзакций.

### <a name="is-analytical-store-supported-in-all-azure-cosmos-db-regions"></a>Поддерживается ли аналитическое хранилище во всех регионах Azure Cosmos DB?

Да.

### <a name="is-backup-and-restore-supported-for-azure-synapse-link-enabled-accounts"></a>Поддерживается ли резервное копирование и восстановление для учетных записей с поддержкой канала Azure синапсе?

Для контейнеров с включенным аналитическим хранилищем в настоящее время автоматическое резервное копирование и восстановление данных в аналитическом хранилище не поддерживается. 

Если в учетной записи базы данных включена ссылка синапсе, Azure Cosmos DB продолжит автоматически [выполнять резервное копирование](./online-backup-and-restore.md) данных в хранилище транзакций (только) контейнеров с запланированным интервалом резервного копирования, как всегда. Важно отметить, что при восстановлении контейнера с включенным аналитическим хранилищем в новую учетную запись контейнер будет восстановлен только с хранилищем транзакций и без включенного аналитического хранилища. 

### <a name="can-i-disable-the-azure-synapse-link-feature-for-my-azure-cosmos-db-account"></a>Можно ли отключить компонент Azure синапсе Link для учетной записи Azure Cosmos DB?

В настоящее время после включения возможности Synapse Link на уровне учетной записи ее отключить нельзя. Обратите внимание: если возможность Synapse Link включена на уровне учетной записи и контейнеры с поддержкой аналитического хранилища отсутствуют, ее включение никак не влияет на выставляемые счета.

Если вам нужно отключить эту возможность, у вас есть 2 варианта. Первый — удалите и повторно создайте учетную запись Azure Cosmos DB, при необходимости перенеся данные. Второй — создайте запрос в службу поддержки, чтобы получить помощь с переносом данных в другую учетную запись.

### <a name="does-analytical-store-have-any-impact-on-cosmos-db-transactional-slas"></a>Влияет ли аналитическое хранилище на Cosmos DB соглашения об уровне обслуживания транзакций?

Нет, никаких последствий не возникает.

## <a name="azure-cosmos-db-analytical-store"></a>Аналитическое хранилище Azure Cosmos DB

### <a name="can-i-enable-analytical-store-on-existing-containers"></a>Можно ли включить аналитическое хранилище для имеющихся контейнеров?

В настоящее время аналитическое хранилище можно включить только для новых контейнеров (как в новых, так и в имеющихся учетных записях).

### <a name="can-i-disable-analytical-store-on-my-azure-cosmos-db-containers-after-enabling-it-during-container-creation"></a>Можно ли отключить аналитическое хранилище в контейнерах Azure Cosmos DB после включения его во время создания контейнера?

В настоящее время аналитическое хранилище нельзя отключить для контейнера Azure Cosmos DB после его включения во время создания контейнера.

### <a name="is-analytical-store-supported-for-azure-cosmos-db-containers-with-autoscale-provisioned-throughput"></a>Поддерживается ли аналитическое хранилище для Azure Cosmos DB контейнеров с подготовленной пропускной способностью автомасштабирования?

Да, аналитическое хранилище можно включить для контейнеров с подготовленной пропускной способностью с возможностью автомасштабирования.

### <a name="is-there-any-effect-on-azure-cosmos-db-transactional-store-provisioned-rus"></a>Влияет ли эта возможность на количество подготовленных единиц запросов для хранилища транзакций Azure Cosmos DB?

Azure Cosmos DB гарантирует изолированное выполнение транзакционных и аналитических рабочих нагрузок. Включение аналитического хранилища для контейнера не повлияет на показатель ЕЗ/с, подготовленный для хранилища транзакций Azure Cosmos DB. Плата за транзакции (чтение и запись) и плата за хранение для аналитического хранилища будет начисляться отдельно. Дополнительные сведения см. в [разделе цен статьи об аналитическом хранилище Azure Cosmos DB](analytical-store-introduction.md#analytical-store-pricing).

### <a name="can-i-restrict-access-to-azure-cosmos-db-analytical-store"></a>Можно ли ограничить доступ к Azure Cosmos DBному аналитическому хранилищу?

Да, вы можете настроить [управляемую закрытую конечную точку](analytical-store-private-endpoints.md) и ограничить сетевой доступ аналитического хранилища к управляемой виртуальной сети Azure синапсе. Управляемые частные конечные точки устанавливают закрытую ссылку на Ваше аналитическое хранилище. Эта частная конечная точка также будет ограничивать доступ на запись к хранилищу транзакций, помимо других служб данных Azure.

Вы можете добавить частные конечные точки хранилища транзакций и службы аналитики хранилища в одну учетную запись Azure Cosmos DB в рабочей области Azure синапсе Analytics. Если вы хотите выполнять только аналитические запросы, может потребоваться только сопоставленная частная конечная точка аналитики.

### <a name="can-i-use-customer-managed-keys-with-the-azure-cosmos-db-analytical-store"></a>Можно ли использовать управляемые клиентом ключи с Azure Cosmos DBным аналитическим хранилищем?

Вы можете легко зашифровать данные в транзакциях и аналитических хранилищах, используя те же ключи, управляемые клиентом, в автоматическом и прозрачном режиме. Использование управляемых клиентом ключей с Azure Cosmos DBным аналитическим хранилищем в настоящее время требует дополнительной настройки вашей учетной записи. Для получения дополнительных сведений обратитесь к [группе Azure Cosmos DB](mailto:azurecosmosdbcmk@service.microsoft.com)  .

### <a name="are-delete-and-update-operations-on-the-transactional-store-reflected-in-the-analytical-store"></a>Отражаются ли в аналитическом хранилище операции удаления и обновления в хранилище транзакций?

Да, операции удаления и обновления в хранилище транзакций будут отражены в аналитическом хранилище. Настроив для контейнера срок жизни, можно включить в него данные за предыдущие периоды, чтобы в аналитическом хранилище содержались все версии элементов, удовлетворяющие критериям аналитического срока жизни. Дополнительные сведения см. в [обзор аналитического срока жизни](analytical-store-introduction.md#analytical-ttl).

### <a name="can-i-connect-to-analytical-store-from-analytics-engines-other-than-azure-synapse-analytics"></a>Можно ли подключиться к аналитическому хранилищу из модулей аналитики, отличных от Azure Synapse Analytics?

Вы можете получать доступ к аналитическому хранилищу и отправлять в него запросы только с помощью различных сред выполнения, предоставляемых Azure Synapse Analytics. Отправлять запросы в аналитическое хранилище и анализировать его можно с помощью следующих средств:

* Synapse Spark с полной поддержкой Scala, Python, SparkSQL и C# (Synapse Spark — это основное средство для инжиниринга, обработки и анализа данных);
* Бессерверный пул SQL с языком T-SQL и поддержкой привычных средств бизнес-аналитики (например, Power BI Premium и т. д.).

### <a name="can-i-connect-to-analytical-store-from-synapse-sql-provisioned"></a>Можно ли подключиться к аналитическому хранилищу из подготовленной системы Synapse SQL?

В настоящее время доступ к аналитическому хранилищу нельзя получить из подготовленной системы Synapse SQL.

### <a name="can-i-write-back-the-query-aggregation-results-from-synapse-back-to-the-analytical-store"></a>Можно ли записать результаты агрегирования запросов из Synapse в аналитическое хранилище?

Аналитическое хранилище — это хранилище, доступное только для чтения, в контейнере Azure Cosmos DB. Поэтому вы не можете напрямую записывать в него результаты агрегирования, но можете записать их в хранилище транзакций Azure Cosmos DB другого контейнера, который впоследствии можно использовать в качестве уровня обслуживания.

### <a name="is-the-autosync-replication-from-transactional-store-to-the-analytical-store-asynchronous-or-synchronous-and-what-are-the-latencies"></a>Как выполняется автосинхронизируемая репликация из хранилища транзакций в аналитическое хранилище (асинхронно или синхронно) и каковы при этом задержки?

Задержка автоматической синхронизации обычно составляет в течение 2 минут. В случае с общей пропускной способностью базы данных с большим количеством контейнеров, задержка автоматической синхронизации отдельных контейнеров может быть выше и займет до 5 минут. Мы хотели бы узнать больше о том, как эта задержка подходит для ваших сценариев. Для этого обратитесь к [группе Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

### <a name="are-there-any-scenarios-where-the-items-from-the-transactional-store-are-not-automatically-propagated-to-the-analytical-store"></a>Существуют ли сценарии, в которых элементы из хранилища транзакций не распространяются автоматически в аналитическое хранилище?

Если определенные элементы в контейнере нарушают [четко определенную схему аналитики](analytical-store-introduction.md#analytical-schema), они не будут включаться в аналитическое хранилище. Если у вас есть сценарии, блокируемые четко определенной схемой для аналитики, обратитесь за справкой к [команде Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com) по электронной почте.

### <a name="can-i-partition-the-data-in-analytical-store-differently-from-transactional-store"></a>Можно ли по-разному секционировать данные в аналитическом хранилище и хранилища транзакций?

Данные в аналитическом хранилище секционируются на основе горизонтального секционирования сегментов в хранилище транзакций. В настоящее время для аналитического хранилища нельзя выбрать другую стратегию секционирования.

### <a name="can-i-customize-or-override-the-way-transactional-data-is-transformed-into-columnar-format-in-the-analytical-store"></a>Можно ли настроить или переопределить способ преобразования данных о транзакциях в формат столбцов в аналитическом хранилище?

В настоящее время элементы данных, которые автоматически распространяются из хранилища транзакций в аналитическое хранилище, преобразовать невозможно. При наличии сценариев, блокируемых этим ограничением, отправьте сообщение электронной почты [команде Azure Cosmos DB](mailto:cosmosdbsynapselink@microsoft.com).

### <a name="is-analytical-store-supported-by-terraform"></a>Поддерживается ли в Terraform аналитическое хранилище?

В настоящее время Terraform не поддерживает контейнеры аналитических хранилищ. Дополнительные сведения см. в [Проблемы Terraform на GitHub](https://github.com/hashicorp/terraform/issues).

## <a name="analytical-time-to-live-ttl"></a>Аналитический срок жизни

### <a name="is-ttl-for-analytical-data-supported-at-both-container-and-item-level"></a>Поддерживается ли TTL для аналитических данных на уровне контейнеров и элементов?

В настоящее время срок жизни для аналитических данных можно настроить только на уровне контейнера, а задание аналитического срока жизни на уровне элемента не поддерживается.

### <a name="after-setting-the-container-level--analytical-ttl-on-an-azure-cosmos-db-container-can-i-change-to-a-different-value-later"></a>Можно изменить значение аналитического срока жизни на уровне контейнера в контейнере Azure Cosmos DB после его настройки?

Да, аналитический срок жизни можно обновить любым допустимым значением. Дополнительные сведения см. в разделе об [аналитическом сроке жизни](analytical-store-introduction.md#analytical-ttl).

### <a name="can-i-update-or-delete-an-item-from-the-analytical-store-after-it-has-been-ttld-out-from-the-transactional-store"></a>Можно ли обновить или удалить элемент из аналитического хранилища после истечения их срока жизни в хранилище транзакций?

Все операции обновления и удаления транзакций копируются в аналитическое хранилище, но если элемент был удален из хранилища транзакций, то его нельзя обновить в аналитическом хранилище. Дополнительные сведения см. в разделе об [аналитическом сроке жизни](analytical-store-introduction.md#analytical-ttl).

## <a name="billing"></a>Выставление счетов

### <a name="what-is-the-billing-model-of-azure-synapse-link-for-azure-cosmos-db"></a>Какова модель выставления счетов по ссылке Azure синапсе для Azure Cosmos DB?

Модель выставления счетов по Azure Synapse Link включает в себя затраты, связанные с использованием аналитического хранилища Azure Cosmos DB и среды выполнения Synapse. Дополнительные сведения см. в статьях [Цены на аналитическое хранилище Azure Cosmos DB](analytical-store-introduction.md#analytical-store-pricing) и [Цены на Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/synapse-analytics/).

### <a name="what-is-the-billing-impact-if-i-enable-synapse-link-in-my-azure-cosmos-db-database-account"></a>Какова влияние на выставление счетов, если включить ссылку синапсе в учетной записи базы данных Azure Cosmos DB?

Отсутствует. При создании контейнера с включенным аналитическим хранилищем и запуске загрузки данных будет использоваться только накладная.


## <a name="security"></a>Безопасность

### <a name="what-are-the-ways-to-authenticate-with-the-analytical-store"></a>Какие способы проверки подлинности применяются в аналитическом хранилище?

Проверка подлинности в аналитическом хранилище такая же, как и в хранилище транзакций. Для данной базы данных можно выполнить проверку подлинности с помощью первичного или доступного только для чтения ключа. Вы можете использовать связанную службу в Azure синапсе Studio, чтобы не вставлять ключи Azure Cosmos DB в записные книжки Spark. Доступ к этой связанной службе предоставляется всем, у кого есть доступ к рабочей области.

## <a name="synapse-run-times"></a>Среды выполнения Synapse

### <a name="what-are-the-currently-supported-synapse-run-times-to-access-azure-cosmos-db-analytical-store"></a>Использование каких сред выполнения Synapse для доступа к аналитическому хранилищу Azure Cosmos DB поддерживается в настоящее время?

|Среда выполнения Azure синапсе |Текущая поддержка |
|---------|---------|
|Пулы Azure синапсе Spark | Чтение, запись (с помощью хранилища транзакций), таблица, временное представление |
|Несинапсеный пул SQL Azure    | Чтение, просмотр |
|Подготовленный SQL Azure синапсе   |  Недоступно |

### <a name="do-my-azure-synapse-spark-tables-sync-with-my-azure-synapse-serverless-sql-pool-tables-the-same-way-they-do-with-azure-data-lake"></a>Синхронизируются ли таблицы Azure синапсе Spark с моими таблицами пула SQL Azure синапсе без сервера так же, как с Azure Data Lake?

Сейчас такая возможность недоступна.

### <a name="can-i-do-spark-structured-streaming-from-analytical-store"></a>Можно ли осуществлять структурированную потоковую передачу Spark из аналитического хранилища?

В настоящее время поддержка структурированной потоковой передачи Spark для Azure Cosmos DB реализуется с помощью функциональности канала изменений в хранилище транзакций, однако в аналитическом хранилище она еще не поддерживается.

### <a name="is-streaming-supported"></a>Поддерживается ли потоковая передача?

Потоковая передача данных из аналитического хранилища не поддерживается.

## <a name="azure-synapse-studio"></a>Azure Synapse Studio

### <a name="in-the-azure-synapse-studio-how-do-i-recognize-if-im-connected-to-an-azure-cosmos-db-container-with-the-analytics-store-enabled"></a>Как узнать, подключен ли Azure Cosmos DB контейнер с включенным хранилищем аналитики в Azure синапсе Studio?

Контейнер Azure Cosmos DB с поддержкой аналитического хранилища обозначается следующим значком:

:::image type="content" source="./media/synapse-link-frequently-asked-questions/analytical-store-icon.png" alt-text="Значок контейнера Azure Cosmos DB с поддержкой аналитического хранилища":::

Контейнер хранилища транзакций будет представлен следующим значком:

:::image type="content" source="./media/synapse-link-frequently-asked-questions/transactional-store-icon.png" alt-text="Значок контейнера Azure Cosmos DB с поддержкой хранилища транзакций":::
 
### <a name="how-do-you-pass-azure-cosmos-db-credentials-from-azure-synapse-studio"></a>Как передавать учетные данные Azure Cosmos DB из Azure синапсе Studio?

В настоящее время учетные данные Azure Cosmos DB передаются при создании связанной службы пользователем, имеющим доступ к базам данных Azure Cosmos DB. Доступ к этому хранилищу предоставляется другим пользователям, имеющим доступ к рабочей области.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте о [преимуществах ссылки Azure синапсе](synapse-link.md#synapse-link-benefits)

* Сведения о [интеграции между Azure синапсе Link и Azure Cosmos DB](synapse-link.md#synapse-link-integration).
