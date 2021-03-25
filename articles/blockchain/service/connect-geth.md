---
title: Подключение к службе "Блокчейн Azure" с помощью Geth
description: Присоединение к экземпляру Geth на узле транзакций службы "Блокчейн Azure"
ms.date: 05/26/2020
ms.topic: quickstart
ms.reviewer: maheshna
ms.openlocfilehash: 2c5b6a56d96ac132949052e9879c8f61d41ffb06
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83994795"
---
# <a name="quickstart-use-geth-to-attach-to-an-azure-blockchain-service-transaction-node"></a>Краткое руководство. Подключение к узлу транзакций службы "Блокчейн Azure" с помощью Geth

При работе с этим кратким руководством вы будете использовать клиент Geth для подключения к экземпляру Geth на узле транзакций службы "Блокчейн Azure". После присоединения вы используете консоль Geth для вызова JavaScript API для Ethereum.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* Установите [Geth](https://github.com/ethereum/go-ethereum/wiki/geth).
* См. подробнее об [использовании созданию участника блокчейна на портале Azure](create-member.md) и [руководство по созданию участника блокчейна службы "Блокчейн Azure" с помощью Azure CLI](create-member-cli.md);

## <a name="get-geth-connection-string"></a>Получение строки подключения Geth

Строку подключения Geth можно получить для узла транзакций службы "Блокчейн Azure" на портале Azure.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите в область участника службы "Блокчейн Azure". Выберите **Transaction nodes** (Узлы транзакций) и щелкните ссылку на узел транзакций по умолчанию.

    ![Выбор узла транзакций по умолчанию](./media/connect-geth/transaction-nodes.png)

1. Выберите элемент **Строки подключения**.
1. Скопируйте строку подключения в разделе **HTTPS (Access key 1)** (HTTPS (ключ доступа 1)). Строка понадобится вам при работе со следующим разделом.

    ![Строка подключения](./media/connect-geth/connection-string.png)

## <a name="connect-to-geth"></a>Подключение к Geth

1. Откройте окно командной строки или оболочку.
1. Используйте подкоманду geth attach для присоединения к запущенному экземпляру Geth на узле транзакций. Вставьте строку подключения в качестве аргумента для подкоманды attach. Пример:

    ``` bash
    geth attach <connection string>
    ```

1. После подключения к консоли Ethereum для узла транзакций вы можете использовать API JavaScript для Ethereum.

    Например, используйте указанный ниже API-интерфейс, чтобы получить значение chainId.

    ``` bash
    admin.nodeInfo.protocols.istanbul.config.chainId
    ```

    В этом примере значением для chainId будет 661.

    ![Параметр службы "Блокчейн Azure"](./media/connect-geth/geth-attach.png)

1. Чтобы отключиться от консоли, введите `exit`.

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы использовали клиент Geth для подключения к экземпляру Geth на узле транзакций службы "Блокчейн Azure". В следующем учебнике описано, как с помощью комплекта SDK службы "Блокчейн Azure" для Ethereum создать, собрать, развернуть и выполнить функцию смарт-контракта путем совершения транзакции.

> [!div class="nextstepaction"]
> [Создание, сборка и развертывание смарт-контрактов в службе "Блокчейн Azure"](send-transaction.md)
