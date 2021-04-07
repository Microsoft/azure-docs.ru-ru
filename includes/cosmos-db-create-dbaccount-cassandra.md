---
title: включить файл
description: включить файл
services: cosmos-db
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 10/01/2020
ms.author: sngun
ms.custom: include file
ms.openlocfilehash: 30a7f3ae878cebcd1e58287fc59241651dac2bfd
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "94503896"
---
1. В новом окне браузера войдите на [портал Azure](https://portal.azure.com/).

2. В меню слева выберите **Создать ресурс**.
   
   ![Создание ресурса на портале Azure](./media/cosmos-db-create-dbaccount-cassandra/create-nosql-db-databases-json-tutorial-0.png)
   
3. На странице **New** (Новый) выберите **Базы данных** > **Azure Cosmos DB**.
   
   ![Область "Базы данных" на портале Azure](./media/cosmos-db-create-dbaccount-cassandra/create-nosql-db-databases-json-tutorial-1.png)
   
3. На странице **Создание учетной записи Azure Cosmos DB** введите параметры для новой учетной записи Azure Cosmos DB. 
 
    Параметр|Значение|Описание
    ---|---|---
    Подписка|Ваша подписка|Вы подписку Azure, которую нужно использовать для этой учетной записи Azure Cosmos DB. 
    Группа ресурсов|Создание<br><br>Затем введите имя, использованное для учетной записи|Выберите **Создать**. Затем введите новое имя группы ресурсов для учетной записи. Для удобства можно использовать то же имя, которое присвоено учетной записи Azure Cosmos. 
    Имя учетной записи|Введите уникальное имя.|Введите уникальное имя для идентификации вашей учетной записи Azure Cosmos DB. URI вашей учетной записи *cassandra.cosmos.azure.com* будет добавлен к уникальному имени учетной записи.<br><br>Имя может содержать только строчные буквы, цифры и дефисы. Его длина должна быть от 3 до 31 знаков.
    API|Cassandra|API определяет тип учетной записи, которую нужно создать. Azure Cosmos DB предоставляет пять API: API Core (SQL) для баз данных документов, API Gremlin для графовых баз данных, API MongoDB для баз данных документов, API таблиц Azure и API Cassandra. Для каждого API требуется создать отдельную учетную запись. <br><br>Выберите **Cassandra**, так как в рамках этого краткого руководства вы создадите таблицу, которая работает с API Cassandra. <br><br>[Дополнительные сведения об API Cassandra](../articles/cosmos-db/cassandra-introduction.md).|
    Расположение|Выберите ближайший к пользователям регион|Выберите географическое расположение для размещения учетной записи Azure Cosmos DB. Используйте ближайшее к пользователям расположение, чтобы предоставить им максимально быстрый доступ к данным.
    Режим емкости|"Подготовленная пропускная способность" или "Бессерверный"|Выберите **Подготовленная пропускная способность**, чтобы создать учетную запись в режиме [подготовленной пропускной способности](../articles/cosmos-db/set-throughput.md). Выберите **Бессерверный**, чтобы создать учетную запись в режиме [Бессерверный](../articles/cosmos-db/serverless.md).

    Выберите **Просмотр и создание**. Вы можете пропустить разделы **Сеть**, **Резервное копирование**, **Шифрование** и **Теги**. 

    ![Страница "Новая учетная запись" для Azure Cosmos DB](./media/cosmos-db-create-dbaccount-cassandra/azure-cosmos-db-create-new-account.png)

4. Создание учетной записи займет несколько минут. Дождитесь, пока на портале откроется страница с сообщением **Congratulations! Your Azure Cosmos DB account was created** (Поздравляем! Ваша учетная запись Azure Cosmos DB создана).

