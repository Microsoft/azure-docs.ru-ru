---
title: Часто задаваемые вопросы об API Azure Cosmos DB для MongoDB
description: Получите ответы на часто задаваемые вопросы об API Azure Cosmos DB для MongoDB
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: conceptual
ms.date: 04/28/2020
ms.author: sngun
ms.openlocfilehash: 08899018d03209dab09f61d4dd74feceee03b246
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019025"
---
# <a name="frequently-asked-questions-about-the-azure-cosmos-dbs-api-for-mongodb"></a>Часто задаваемые вопросы об API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

API Azure Cosmos DB для MongoDB — это уровень совместимости протоколов, позволяющий приложениям легко и прозрачно взаимодействовать с собственным ядром СУБД Azure Cosmos, используя существующие, поддерживаемые сообществом пакеты SDK и драйверы для MongoDB. Теперь разработчики могут использовать существующие MongoDB цепочек инструментов и навыки для создания приложений, использующих преимущества Azure Cosmos DB. Разработчики получают преимущества от уникальных возможностей Azure Cosmos DB, включая глобальное распространение с помощью репликации с записью в нескольких регионах, автоматическое индексирование, Обслуживание резервного копирования, финансовые резервные соглашения об уровне обслуживания и т. д.

## <a name="how-do-i-connect-to-my-database"></a>Как подключиться к базе данных?

Чтобы быстро подключиться к базе данных Cosmos через API Azure Cosmos DB для MongoDB, откройте [портал Azure](https://portal.azure.com). Войдите в свою учетную запись и выберите в меню навигации слева **Быстрый запуск**. Быстрый запуск — это лучший способ получить фрагменты кода для подключения к базе данных.

В Azure Cosmos DB строгие требования к безопасности и стандарты. Для учетных записей Azure Cosmos DB требуется проверка подлинности и безопасная связь через TLS, поэтому используйте TLSv 1.2.

Дополнительные сведения см. в разделе [Подключение к базе данных Cosmos с помощью API Azure Cosmos DB для MongoDB](connect-mongodb-account.md).

## <a name="error-codes-while-using-azure-cosmos-dbs-api-for-mongodb"></a>Коды ошибок при использовании API Azure Cosmos DB для MongoDB?

Наряду с распространенными кодами ошибок MongoDB, API Azure Cosmos DB для MongoDB имеет собственные коды ошибок. Их можно найти в разделе [руководство по устранению неполадок](mongodb-troubleshoot.md).

## <a name="supported-drivers"></a>Поддерживаемые драйверы

### <a name="is-the-simba-driver-for-mongodb-supported-for-use-with-azure-cosmos-dbs-api-for-mongodb"></a>Можно ли использовать драйвер Simba для MongoDB с API Azure Cosmos DB для MongoDB?

Да, можно использовать драйвер ODBC для Mongo Simba с API Azure Cosmos DB для MongoDB

## <a name="next-steps"></a>Дальнейшие действия

* [Создание веб-приложения .NET с помощью API Azure Cosmos DB для MongoDB](create-mongodb-dotnet.md)
* [Создание консольного приложения с использованием Java и API MongoDB в Azure Cosmos DB](create-mongodb-java.md)
