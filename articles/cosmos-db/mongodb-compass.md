---
title: Подключение к Azure Cosmos DB с помощью Compass
description: Узнайте, как использовать MongoDB Compass для хранения и администрирования данных в Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 06/05/2020
author: christopheranderson
ms.author: chrande
ms.openlocfilehash: 43bcd54955cb1a8aaf08785368faf13c14f8322c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94413060"
---
# <a name="use-mongodb-compass-to-connect-to-azure-cosmos-dbs-api-for-mongodb"></a>Подключение к API Azure Cosmos DB для MongoDB с помощью MongoDB Compass
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Этот учебник содержит сведения об использовании [MongoDB Compass](https://www.mongodb.com/products/compass) при хранении и администрировании данных в Cosmos DB. В этом пошаговом руководстве используется API Azure Cosmos DB для MongoDB. Для тех из вас, кто еще не знаком с Compass, — это графический пользовательский интерфейс для MongoDB. Обычно он используется для визуализации данных, выполнения ad-hoc-запросов, наряду с управлением данными.

Cosmos DB — это глобально распределенная многомодельная служба базы данных Майкрософт. Вы можете быстро создавать и запрашивать документы, пары "ключ-значение" и графовые базы данных, используя преимущества глобального распределения и горизонтального масштабирования Cosmos DB.

## <a name="pre-requisites"></a>Предварительные требования

Чтобы подключиться к учетной записи Cosmos DB с помощью MongoDB компаса, необходимо:

* Скачайте и установите [Compass](https://www.mongodb.com/download-center/compass?jmp=hero).
* Получите сведений о [строке подключения](connect-mongodb-account.md) Cosmos DB.

## <a name="connect-to-cosmos-dbs-api-for-mongodb"></a>Подключение к API Cosmos DB для MongoDB

Чтобы подключить учетную запись Cosmos DB к Compass, выполните следующие шаги:

1. Получите сведения о подключении для учетной записи Cosmos, настроенной с помощью API Azure Cosmos DB для MongoDB, следуя инструкциям [здесь](connect-mongodb-account.md).

    :::image type="content" source="./media/mongodb-compass/mongodb-compass-connection.png" alt-text="Снимок экрана колонки строки подключения":::

2. Нажмите кнопку с текстом **Копировать в буфер обмена** рядом с **основной или дополнительной строкой подключения** в Cosmos DB. После нажатия этой кнопки строка подключения будет полностью скопирована в буфер обмена.

    :::image type="content" source="./media/mongodb-compass/mongodb-connection-copy.png" alt-text="Снимок экрана кнопки копирования в буфер обмена":::

3. Откройте Compass на рабочем столе или компьютере и щелкните **Подключить**, а затем **Подключение к...**.

4. Compass автоматически обнаруживает строку подключения в буфере обмена и спросит, хотите ли вы ее использовать для подключения. Щелкните **Да**, как показано на следующем снимке экрана.

    :::image type="content" source="./media/mongodb-compass/mongodb-compass-detect.png" alt-text="На снимке экрана показано диалоговое окно, объясняющее, что у вас есть строка подключения в буфере обмена.":::

5. После нажатия кнопки **Да** на предыдущем шаге данные из строки подключения будут заполнены автоматически. Удалите автоматически заполненное значение в поле **Replica Set Name** (Имя набора реплик), убедитесь, что поле пустое.

    :::image type="content" source="./media/mongodb-compass/mongodb-compass-replica.png" alt-text="На снимке экрана показано текстовое поле Имя набора реплик.":::

6. В нижней части страницы щелкните **Подключиться**. Базы данных и учетная запись Cosmos DB теперь должны отображаться в MongoDB Compass.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [использовать Studio 3T](mongodb-mongochef.md) с API Azure Cosmos DB для MongoDB.
- Ознакомьтесь с [примерами](mongodb-samples.md) MongoDB с API Azure Cosmos DB для MongoDB.
