---
title: Подключение к Azure Synapse Link для Azure Cosmos DB
description: Узнайте, как подключить базу данных Azure Cosmos DB к рабочей области Azure Synapse с помощью Azure Synapse Link.
services: synapse-analytics
author: Rodrigossz
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 03/02/2021
ms.author: rosouz
ms.reviewer: jrasnick
ms.custom: cosmos-db
ms.openlocfilehash: 9a73ea697a48a5b2514f4701b8a24896e9a04c1f
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105627699"
---
# <a name="connect-to-azure-synapse-link-for-azure-cosmos-db"></a>Подключение к Azure Synapse Link для Azure Cosmos DB

В этой статье описывается, как получить доступ к базе данных Azure Cosmos DB из Azure Synapse Analytics Studio с помощью Azure Synapse Link.

## <a name="prerequisites"></a>Предварительные требования

Перед подключением базы данных Azure Cosmos DB к рабочей области вам потребуется следующее:

* имеющаяся база данных Azure Cosmos DB (можно также создать учетную запись, следуя инструкциям из [краткого руководства по управлению учетной записью Azure Cosmos DB](../../cosmos-db/how-to-manage-database-account.md));
* имеющаяся рабочая область Azure Synapse (можно также создать рабочую область, следуя инструкциям из [краткого руководства по созданию рабочей области Synapse](../quickstart-create-workspace.md)).

## <a name="enable-synapse-link-on-an-azure-cosmos-db-database-account"></a>Включение Synapse Link для учетной записи базы данных Azure Cosmos DB

Чтобы запустить крупномасштабную аналитику в Azure Cosmos DB без негативных последствий для производительности в рабочей среде, рекомендуем включить Synapse Link для Azure Cosmos DB. Synapse Link предоставляет возможность HTAP для контейнера и встроенную поддержку в Azure Synapse.

## <a name="go-to-synapse-studio"></a>Перейдите в Synapse Studio

В рабочей области Azure Synapse выберите **Запуск Synapse Studio**. На домашней странице Synapse Studio выберите **Данные**, чтобы перейти в обозреватель объектов данных.

## <a name="connect-an-azure-cosmos-db-database-to-an-azure-synapse-workspace"></a>Подключение базы данных Azure Cosmos DB к рабочей области Azure Synapse

База данных Azure Cosmos DB подключается в качестве связанной службы. Связанная служба Azure Cosmos DB позволяет пользователям просматривать и изучать данные, а также считывать и записывать их из Apache Spark для Azure Synapse Analytics или SQL в Azure Cosmos DB.

В обозревателе объектов данных можно напрямую подключиться к базе данных Azure Cosmos DB, выполнив указанные ниже действия.

1. Выберите значок **+** рядом с областью **Данные**.
1. Выберите **Connect to external data** (Подключение к внешнем данным).
1. Выберите API, к которому необходимо подключиться, например **API SQL** или **API для MongoDB**.
1. Выберите **Continue** (Продолжить).
1. Чтобы присвоить имя связанной службе, используйте понятное имя. Имя появится в обозревателе объектов данных и будет использоваться средами выполнения Azure Synapse для подключения к базе данных и контейнерам.
1. Выберите **имя учетной записи Azure Cosmos DB** и **имя базы данных**.
1. (Необязательно) Если регион не указан, операции среды выполнения Azure Synapse будут направляться в ближайший регион, где включено аналитическое хранилище. Вы также можете вручную задать регион, который пользователи будут использовать для доступа к аналитическому хранилищу Azure Cosmos DB. Выберите **Additional connection properties** (Дополнительные свойства подключения), а затем — **Создать**. Для параметра **Имя свойства** введите **PreferredRegions**. Задайте **значение** необходимого региона, например **WestUS2**. (Между словами и числом нет пробелов.)
1. Нажмите кнопку **создания**.

Базы данных Azure Cosmos DB отображаются на вкладке **Связано** в разделе **Azure Cosmos DB**. Azure Cosmos DB позволяет отличить контейнер с поддержкой HTAP от контейнера, поддерживающего только OLTP, по указанным ниже значкам.

**Контейнер, поддерживающий только OLTP**:

![Визуализация, отображающая значок контейнера OLTP.](../media/quickstart-connect-synapse-link-cosmosdb/oltp-container.png)

**Контейнер с поддержкой HTAP**:

![Визуализация, отображающая значок контейнера HTAP.](../media/quickstart-connect-synapse-link-cosmosdb/htap-container.png)

## <a name="quickly-interact-with-code-generated-actions"></a>Быстрое взаимодействие с созданными кодом действиями

Щелкнув правой кнопкой мыши контейнер, вы получите список жестов, которые активируют среду выполнения Spark или SQL. Запись в контейнер происходит через хранилище транзакций Azure Cosmos DB и приводит к использованию единиц запросов.  

## <a name="next-steps"></a>Дальнейшие действия

* [Узнайте, какие общие возможности поддерживают Azure Synapse и Azure Cosmos DB](./concept-synapse-link-cosmos-db-support.md).
* [Настройка частных конечных точек для аналитического хранилища Azure Cosmos DB](../../cosmos-db/analytical-store-private-endpoints.md)
* [Узнайте, как отправлять запросы в аналитическое хранилище с помощью Spark](./how-to-query-analytical-store-spark.md).