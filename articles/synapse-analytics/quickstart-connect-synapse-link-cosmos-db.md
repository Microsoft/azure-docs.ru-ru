---
title: Краткое руководство. Подключение к Azure Synapse Link для Azure Cosmos DB
description: Подключение Azure Cosmos DB к рабочей области Synapse с помощью Synapse Link
services: synapse-analytics
author: Rodrigossz
ms.service: synapse-analytics
ms.subservice: synapse-link
ms.topic: quickstart
ms.date: 04/21/2020
ms.author: rosouz
ms.reviewer: jrasnick
ms.custom: cosmos-db
ms.openlocfilehash: 7d77431f5caa1a2ac67428326dcd6d4ce75a4a93
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105625846"
---
# <a name="quickstart-connect-to-azure-synapse-link-for-azure-cosmos-db"></a>Краткое руководство. Подключение к Azure Synapse Link для Azure Cosmos DB

В этой статье описывается, как получить доступ к базе данных Azure Cosmos DB из Azure Synapse Analytics Studio с помощью Synapse Link. 

## <a name="prerequisites"></a>Предварительные требования

Для подключения учетной записи Azure Cosmos DB к рабочей области потребуется следующее:

* имеющаяся учетная запись Azure Cosmos DB (можно также создать новую учетную запись, следуя инструкциям в этом [кратком руководстве](../cosmos-db/how-to-manage-database-account.md));
* имеющаяся рабочая область Synapse (можно также создать новую рабочую область, следуя инструкциям в этом [кратком руководстве](./quickstart-create-workspace.md)). 

## <a name="enable-azure-cosmos-db-analytical-store"></a>Включение аналитического хранилища Azure Cosmos DB

Для запуска крупномасштабной аналитики в Azure Cosmos DB без негативных последствий для производительности в рабочей среде рекомендуем включить Synapse Link для Azure Cosmos DB. Эта функция предоставляет возможность HTAP для контейнера и встроенную поддержку в Azure Synapse. Выполните инструкции в этом кратком руководстве, чтобы включить Synapse Link для контейнеров Cosmos DB.

## <a name="navigate-to-synapse-studio"></a>Перейдите в Synapse Studio

В рабочей области Synapse выберите **Запуск Synapse Studio**. На домашней странице Synapse Studio выберите "**Данные", чтобы перейти в **обозреватель объектов данных**.

## <a name="connect-an-azure-cosmos-db-database-to-a-synapse-workspace"></a>Подключение базы данных Azure Cosmos DB к рабочей области Synapse

База данных Azure Cosmos DB подключается в качестве связанной службы. Связанная служба Cosmos DB позволяет пользователям просматривать и изучать данные, а также считывать и записывать их из Apache Spark для Azure Synapse Analytics или SQL в Azure Cosmos DB.

В обозревателе объектов данных можно напрямую подключиться к базе данных Azure Cosmos DB, выполнив следующие действия.

1. Щелкните значок ***+*** рядом с данными.
2. Выберите **Connect to external data** (Подключение к внешнем данным).
3. Укажите API для подключения: SQL или MongoDB.
4. Выберите ***Продолжить***
5. Присвойте имя связанной службе. Имя будет отображаться в обозревателе объектов и использоваться средой выполнения Synapse для подключения к базе данных и контейнерам. Рекомендуем использовать понятное имя.
6. Выберите **имя учетной записи Cosmos DB** и **имя базы данных**.
7. (Необязательно.) Если регион не указан, операции среды выполнения Synapse будут направляться в ближайший регион, где включено аналитическое хранилище. Однако можно вручную задать регион, который пользователи будут использовать для доступа к аналитическому хранилищу Cosmos DB. Выберите **Additional connection properties** (Дополнительные свойства подключения) — а затем **Создать**. В разделе **Имя свойства** введите **_PreferredRegions_ *_ и задайте в поле _* Значение** нужный регион (например, WestUS2 — между словами и числом нет пробелов).
8. Нажмите кнопку ***Создать***

Базы данных Azure Cosmos DB отображаются на вкладке **Подключено** в разделе Azure Cosmos DB. Контейнер Azure Cosmos DB с поддержкой HTAP можно отличить от контейнера, поддерживающего только OLTP, по следующим значкам:

**Контейнер Synapse**:

![Контейнер HTAP](./media/quickstart-connect-synapse-link-cosmosdb/htap-container.png)

**Контейнер, поддерживающий только OLTP**:

![Контейнер OLTP](./media/quickstart-connect-synapse-link-cosmosdb/oltp-container.png)

## <a name="quickly-interact-with-code-generated-actions"></a>Быстрое взаимодействие с созданными кодом действиями

Если щелкнуть правой кнопкой мыши контейнер, отобразится список жестов, которые активируют среду выполнения Spark или SQL. Запись в контейнер происходит через хранилище транзакций Azure Cosmos DB и приводит к использованию единиц запросов.  

## <a name="next-steps"></a>Дальнейшие действия

* [Узнайте, какие общие возможности поддерживают Synapse и Azure Cosmos DB](./synapse-link/concept-synapse-link-cosmos-db-support.md).
* [Узнайте, как отправлять запросы в аналитическое хранилище Azure Cosmos DB с помощью Apache Spark для Azure Synapse Analytics](synapse-link/how-to-query-analytical-store-spark.md).