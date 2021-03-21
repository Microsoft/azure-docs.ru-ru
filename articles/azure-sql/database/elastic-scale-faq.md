---
title: Вопросы и ответы по гибкому масштабированию
description: Часто задаваемые вопросы об эластичном масштабировании базы данных SQL Azure.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 51e15a8dc5e9f918c630397d6d6593f5bf561755
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92786910"
---
# <a name="elastic-database-tools-frequently-asked-questions-faq"></a>Часто задаваемые вопросы об инструментах эластичных баз данных
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="if-i-have-a-single-tenant-per-shard-and-no-sharding-key-how-do-i-populate-the-sharding-key-for-the-schema-info"></a>На каждый сегмент приходится по одному клиенту, но нет ключа сегментирования. Как заполнить ключ сегментирования для сведений схемы?

Объект сведений схемы используется только в сценариях разделения и слияния. Если приложение изначально однотенантное, инструмент разбиения и объединения для него не требуется, поэтому заполнять объект данных схемы не нужно.

## <a name="ive-provisioned-a-database-and-i-already-have-a-shard-map-manager-how-do-i-register-this-new-database-as-a-shard"></a>База данных подготовлена, и уже назначен диспетчер сопоставления сегментов. Как зарегистрировать эту новую базу данных как сегмент?

Обратитесь к статье [Добавление сегмента с использованием средств эластичных баз данных](elastic-scale-add-a-shard.md).

## <a name="how-much-do-elastic-database-tools-cost"></a>Какова стоимость инструментов эластичных баз данных?

Плата за использование клиентской библиотеки эластичной базы данных не взимается. Затраты начисляются только для баз данных в базе данных SQL Azure, которые используются для сегментов и диспетчера сопоставления сегментов, а также для веб-ролей и рабочей роли, которые подготавливаются для инструмента разбиения слияния.

## <a name="why-are-my-credentials-not-working-when-i-add-a-shard-from-a-different-server"></a>Почему мои учетные данные не работают при добавлении сегмента с другого сервера?

Не используйте учетные данные в формате User ID = username@servername. Мы советуем использовать User ID = username.  Кроме того, убедитесь, что у имени пользователя username есть разрешения на доступ к сегменту.

## <a name="do-i-need-to-create-a-shard-map-manager-and-populate-shards-every-time-i-start-my-applications"></a>Нужно ли создавать диспетчер сопоставления сегментов и заполнять сегменты при каждом запуске приложений?

Нет. Диспетчер сопоставления сегментов (например, [ShardMapManagerFactory.CreateSqlShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager)) создается только один раз.  Во время запуска приложение должно вызывать диспетчер [ShardMapManagerFactory.TryGetSqlShardMapManager()](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager).  В каждом домене приложения должен быть только один такой вызов.

## <a name="i-have-questions-about-using-elastic-database-tools-how-do-i-get-them-answered"></a>У меня есть вопросы об использовании инструментов эластичной базы данных. Как найти ответы?

Свяжитесь с нами на [странице Microsoft Q&A вопроса для базы данных SQL](/answers/topics/azure-sql-database.html).

## <a name="when-i-get-a-database-connection-using-a-sharding-key-i-can-still-query-data-for-other-sharding-keys-on-the-same-shard--is-this-by-design"></a>При подключении к базе данных с помощью ключа сегментирования все равно удается запрашивать данные для других ключей сегментирования в том же сегменте.  Так и должно быть?

Интерфейсы API гибкого масштабирования позволяют подключаться к нужным базам данных с помощью ключа сегментирования, но не обеспечивают фильтрацию этих ключей.  Добавьте предложения **WHERE** к своему запросу, чтобы ограничить масштаб до соответствующего ключа сегментирования, если потребуется.

## <a name="can-i-use-a-different-sql-database-edition-for-each-shard-in-my-shard-set"></a>Можно ли использовать разные выпуски Базы данных SQL для каждого сегмента моего набора сегментов?

Да. Ваш сегмент является отдельной базой данных и поэтому один сегмент может быть выпуском Premium, тогда как другой — выпуском Standard. Кроме того, выпуск сегмента можно многократно масштабировать во время срока жизни сегмента.

## <a name="does-the-split-merge-tool-provision-or-delete-a-database-during-a-split-or-merge-operation"></a>Подготавливает (или удаляет) ли инструмент разбиения и объединения базу данных в процессе разбиения или объединения?

Нет. Для выполнения операций **разбиения** целевая база данных должна существовать в пределах соответствующей схемы и быть зарегистрированной в диспетчере сопоставления сегментов.  Для выполнения операций **объединения** необходимо удалить сегмент из диспетчера сопоставления сегментов, а затем удалить базу данных.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]