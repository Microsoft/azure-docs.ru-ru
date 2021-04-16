---
title: Общие сведения об API Azure Cosmos DB для MongoDB
description: Узнайте, как с применением Azure Cosmos DB хранить и запрашивать большие объемы данных с помощью API Azure Cosmos DB для MongoDB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 03/02/2021
author: sivethe
ms.author: sivethe
ms.openlocfilehash: dfaa04077a5148f19f06aa5e1257b986377ea8c0
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307135"
---
# <a name="azure-cosmos-dbs-api-for-mongodb"></a>API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

[Azure Cosmos DB](introduction.md) — это глобально распределенная, многомодельная служба базы данных Майкрософт, необходимая для работы с критически важными приложениями. Эта служба базы данных обеспечивает [готовое к использованию глобальное распределение](distribute-data-globally.md), [гибкое масштабирование пропускной способности и хранилища](partitioning-overview.md) по всему миру, задержки менее 10 миллисекунд на уровне 99-го процентиля, а также гарантированную высокую доступность — все это согласно [ведущим в отрасли соглашениям об уровне обслуживания](https://azure.microsoft.com/support/legal/sla/cosmos-db/). Azure Cosmos DB [автоматически индексирует данные](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf) без необходимости управлять схемой и индексом. Так как эта база данных является многомодельной, она поддерживает модели данных документа, "ключ — значение", графа и столбчатые модели данных. Служба Azure Cosmos DB реализует сетевые протоколы для стандартных API NoSQL, включая Cassandra, MongoDB, Gremlin и Хранилище таблиц Azure. Это позволяет использовать привычные клиентские драйверы и средства NoSQL для взаимодействия с базой данных Cosmos.

> [!NOTE]
> В API Azure Cosmos DB для MongoDB теперь доступен [режим бессерверной емкости](serverless.md).

## <a name="wire-protocol-compatibility"></a>Совместимость с сетевыми протоколами

Azure Cosmos DB реализует протокол проводной сети для MongoDB. Эта реализация обеспечивает прозрачную совместимость с собственными клиентскими пакетами SDK, драйверами и инструментами для MongoDB. В Azure Cosmos DB не размещено ядро СУБД MongoDB. Сведения о функциях, поддерживаемых MongoDB, можно найти здесь: 
- [API Azure Cosmos DB для MongoDB версии 4.0](mongodb-feature-support-40.md)
- [API Azure Cosmos DB для MongoDB версии 3.6](mongodb-feature-support-36.md)

По умолчанию новые учетные записи, созданные с помощью API Azure Cosmos DB для MongoDB, совместимы с версией 3.6 сетевого протокола MongoDB. Любой драйвер клиента MongoDB, который распознает эти версии протокола, должен иметь возможность собственного подключения к Cosmos DB.

:::image type="content" source="./media/mongodb-introduction/cosmosdb-mongodb.png" alt-text="API Azure Cosmos DB для MongoDB" border="false":::

## <a name="key-benefits"></a>Основные преимущества

Основные преимущества Cosmos DB как полностью управляемой, глобально распределенной базы данных как услуги описаны [здесь](introduction.md). Кроме того, благодаря собственной реализации сетевых протоколов популярных API NoSQL служба Cosmos DB обеспечивает следующие преимущества:

* Простой перенос приложений в Cosmos DB с сохранением значительной части логики приложения.
* Сохранение переносимости приложения и независимости от поставщика облачной платформы.
* Лучшие в отрасли и подкрепленные финансовыми обязательствами Соглашения об уровне обслуживания для распространенных API NoSQL на базе Cosmos DB.
* Гибкое масштабирование подготовленной пропускной способности и хранилища для баз данных Cosmos с учетом текущих потребностей, что позволяет оплачивать только необходимую пропускную способность и хранилище. Все это приводит к значительному снижению затрат.
* Глобальное распространение "под ключ" с репликацией и выполнением операций записи в нескольких регионах.

## <a name="cosmos-dbs-api-for-mongodb"></a>API Cosmos DB для MongoDB

Ознакомьтесь с краткими руководствами, чтобы создать учетную запись Azure Cosmos и перенести имеющееся приложение MongoDB в Azure Cosmos DB (либо создать новое):

* [Azure Cosmos DB — перенос существующего веб-приложения MongoDB на Node.js](create-mongodb-nodejs.md)
* [Создание веб-приложения с помощью API Azure Cosmos DB для MongoDB и пакета SDK для .NET](create-mongodb-dotnet.md)
* [Создание консольного приложения с помощью API Azure Cosmos DB для MongoDB на Java и портале Azure](create-mongodb-java.md)

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с приведенными ниже ресурсами:

* Инструкции по получению сведений о строке подключения к учетной записи см. в руководстве [Подключение приложения MongoDB к Azure Cosmos DB](connect-mongodb-account.md).
* Сведения о создании подключения между базой данных Cosmos и приложением MongoDB в Studio 3T см. в статье [Подключение к учетной записи MongoDB с помощью Studio 3T (MongoChef)](mongodb-mongochef.md).
* Следуйте указаниям статьи [Руководство по переносу данных в учетную запись API MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json), чтобы импортировать данные в базу данных Cosmos.
* Сведения о подключении к учетной записи Cosmos с помощью Robo 3T см. [здесь](mongodb-robomongo.md).
* Узнайте, как [настроить параметры чтения для глобально распределенных приложений](../cosmos-db/tutorial-global-distribution-mongodb.md).
* Узнайте, как устранить распространенные ошибки, в нашем [руководстве по устранению неполадок](mongodb-troubleshoot.md).


<sup>Примечание. В этой статье описывается функция Azure Cosmos DB, обеспечивающая совместимость сетевого протокола с базами данных MongoDB. Корпорация Майкрософт не использует базы данных MongoDB для предоставления этой службы. Azure Cosmos DB не связана с MongoDB, Inc.</sup>
