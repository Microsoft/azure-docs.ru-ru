---
title: Краткое руководство. Подключение Azure Data Explorer к рабочей области Azure Synapse Analytics
description: Подключение кластера Azure Data Explorer к рабочей области Azure Synapse Analytics с помощью Apache Spark для Azure Synapse Analytics.
services: synapse-analytics
author: manojraheja
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: workspace
ms.date: 10/07/2020
ms.author: maraheja
ms.reviewer: jrasnick
ms.openlocfilehash: ee9d137973bfa4eeb28bc6526437e76e781f3199
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92172273"
---
# <a name="connect-to-azure-data-explorer-using-apache-spark-for-azure-synapse-analytics"></a>Подключение к Azure Data Explorer с помощью Apache Spark для Azure Synapse Analytics

В этой статье описывается, как получить доступ к базам данных Azure Data Explorer из Synapse Studio с помощью Apache Spark для Azure Synapse Analytics.

## <a name="prerequisites"></a>Предварительные требования

* [Создайте кластер и базу данных Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal).
* Если у вас нет рабочей области Azure Synapse Analytics, создайте новую рабочую область, выполнив действия, описанные в [кратком руководстве по созданию рабочей области Azure Synapse](./quickstart-create-workspace.md).
* Если у вас нет существующего пула Apache Spark, создайте новый пул, выполнив действия, описанные в руководстве [ Создание пула Apache Spark с помощью портала Azure](./quickstart-create-apache-spark-pool-portal.md).
* [Создание приложения Azure Active Directory (Azure AD) путем подготовки приложения Azure AD](/azure/data-explorer/kusto/management/access-control/how-to-provision-aad-app).
* Предоставьте приложению Azure AD доступ к базе данных, выполнив действия, описанные в разделе [Управление разрешениями базы данных Azure Data Explorer](/azure/data-explorer/manage-database-permissions).

## <a name="go-to-synapse-studio"></a>Перейдите в Synapse Studio

В рабочей области Azure Synapse выберите **Запуск Synapse Studio**. На домашней странице Synapse Studio выберите **Данные**, чтобы открыть **обозреватель объектов данных**.

## <a name="connect-an-azure-data-explorer-database-to-an-azure-synapse-workspace"></a>Подключение базы данных Azure Data Explorer к рабочей области Azure Synapse

Подключение базы данных Azure Data Explorer к рабочей области выполняется через связанную службу. С помощью связанной службы Azure Data Explorer вы можете просматривать, изучать, читать и записывать данные из Apache Spark для Azure Synapse. Вы также можете запускать задания интеграции в конвейере.

В обозревателе объектов данных выполните следующие действия, чтобы создать прямое соединение с кластером Azure Data Explorer.

1. Выберите значок **+** рядом с областью **Данные**.
1. Выберите **Подключиться**, чтобы подключиться к внешним данным.
1. Выберите **Azure Data Explorer (Kusto)**
1. Выберите **Continue** (Продолжить).
1. Чтобы присвоить имя связанной службе, используйте понятное имя. Имя появится в обозревателе объектов данных и будет использоваться средами выполнения Azure Synapse для подключения к базе данных.
1. Выберите кластер Azure Data Explorer из подписки или введите универсальный код ресурса (URI).
1. Введите **идентификатор субъекта-службы** и **ключ субъекта-службы**. Убедитесь, что субъект-служба имеет доступ на просмотр к базе данных для чтения и получения доступа для приема данных.
1. Введите имя базы данных Azure Data Explorer.
1. Щелкните **Проверить подключение**, чтобы убедиться в наличии нужных разрешений.
1. Нажмите кнопку **создания**.

    ![Снимок экрана, на котором показана новая связанная служба.](./media/quickstart-connect-azure-data-explorer/003-new-linked-service.png)

    > [!NOTE]
    > (Необязательно) **Проверка подключения** не проверяет доступ на запись. Убедитесь, что у идентификатора субъекта-службы есть доступ на запись к базе данных Azure Data Explorer.

1. Кластеры и базы данных Azure Data Explorer отображаются на вкладке **Связанные** в разделе **Azure Data Explorer**.

    ![Снимок экрана, на котором показаны кластеры.](./media/quickstart-connect-azure-data-explorer/004-browse-clusters.png)

    > [!NOTE]
    > В текущем выпуске объекты базы данных заполняются на основе разрешений учетной записи Azure AD в базах данных Azure Data Explorer. При запуске записных книжек Apache Spark или заданий интеграции будут использоваться учетные данные в службе каналов (например, субъект-служба).

## <a name="quickly-interact-with-code-generated-actions"></a>Быстрое взаимодействие с созданными кодом действиями

Если щелкнуть правой кнопкой мыши базу данных или таблицу, появится список примеров записных книжек Spark. Выберите параметр для чтения, записи или потоковой передачи данных в Azure Data Explorer.

[![Снимок экрана, на котором показаны новые примеры записных книжек.](./media/quickstart-connect-azure-data-explorer/005-new-notebook.png)](./media/quickstart-connect-azure-data-explorer/005-new-notebook.png#lightbox)

Ниже приведен пример считывания данных. Подключите записную книжку к пулу Spark и выполните эту ячейку.

[![Снимок экрана, на котором показана новая записная книжка для чтения.](./media/quickstart-connect-azure-data-explorer/006-read-data.png)](./media/quickstart-connect-azure-data-explorer/006-read-data.png#lightbox)

   > [!NOTE]
   > Первый запуск сеанса Spark может занять более трех минут. Последующие выполнения будут завершаться значительно быстрее.

## <a name="limitations"></a>Ограничения

Соединитель Azure Data Explorer в настоящее время не поддерживается в управляемых виртуальных сетях Azure Synapse.

## <a name="next-steps"></a>Дальнейшие действия

* [Пример кода с дополнительными параметрами](https://github.com/Azure/azure-kusto-spark/blob/master/samples/src/main/python/SynapseSample.py)
* [Соединитель Spark Azure Data Explorer (Kusto)](https://github.com/Azure/azure-kusto-spark)