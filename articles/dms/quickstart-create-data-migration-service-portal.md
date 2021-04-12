---
title: Краткое руководство. Создание экземпляра с помощью портала Azure
titleSuffix: Azure Database Migration Service
description: Использование портала Azure для создания экземпляра Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: quickstart
ms.date: 01/29/2021
ms.openlocfilehash: 8a500104a0273b9e131815c4dc832bd33729cd51
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105566597"
---
# <a name="quickstart-create-an-instance-of-the-azure-database-migration-service-by-using-the-azure-portal"></a>Краткое руководство. Создание экземпляра службы Azure Database Migration Service с помощью портала Azure

В этом кратком руководстве вы создадите экземпляр Azure Database Migration Service с помощью портала Azure. После создания экземпляра его можно использовать для переноса данных из нескольких источников базы данных на платформы данных Azure, например из SQL Server в Базу данных SQL Azure или из SQL Server в Управляемый экземпляр SQL Azure.

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Откройте веб-браузер, перейдите к [порталу Microsoft Azure](https://portal.azure.com/) и введите учетные данные для входа на портал. Панель мониторинга службы является представлением по умолчанию.

> [!NOTE]
> В одной подписке можно создать до 10 экземпляров DMS на регион. Если требуется больше экземпляров, создайте запрос в службу поддержки.

## <a name="register-the-resource-provider"></a>Регистрация поставщика ресурсов

Прежде чем создать свой первый экземпляр Database Migration Service, зарегистрируйте поставщик ресурсов Microsoft.DataMigration.

1. На портале Azure найдите и выберите **Подписки**.

   ![Отображение подписок на портале](media/quickstart-create-data-migration-service-portal/portal-select-subscription.png)

2. Выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Отображение поставщиков ресурсов](media/quickstart-create-data-migration-service-portal/portal-select-resource-provider.png)

3. В поле поиска введите migration, а затем щелкните элемент **Зарегистрировать** рядом с **Microsoft.DataMigration**.

    ![Регистрация поставщика ресурсов](media/quickstart-create-data-migration-service-portal/dms-register-provider.png)

## <a name="create-an-instance-of-the-service"></a>Создание экземпляра службы

1. На **домашней странице** или в меню портала Azure щелкните элемент **Создать ресурс**. Найдите и выберите элемент **Azure Database Migration Service**.

    ![Azure Marketplace](media/quickstart-create-data-migration-service-portal/portal-marketplace.png)

2. На экране **Azure Database Migration Service** выберите **Создать**.

    ![Создание экземпляра Azure Database Migration Service](media/quickstart-create-data-migration-service-portal/dms-create.png)

3. На экране основных сведений **Создание Migration Service** сделайте следующее:

     - Выберите подписку.
     - Создайте группу ресурсов или выберите существующую.
     - Укажите имя для экземпляра Azure Database Migration Service.
     - Выберите расположение, в котором нужно создать экземпляр Azure Database Migration Service.
     - Выберите **Azure** в качестве режима службы.
     - Выберите ценовую категорию. Дополнительные сведения о ценовых категориях и затратах см. на [странице с описанием цен](https://aka.ms/dms-pricing).
     
    ![Настройка основных параметров экземпляра Azure Database Migration Service](media/quickstart-create-data-migration-service-portal/dms-create-basics.png)

     - Нажмите Далее: сеть.

4. На экране сети **Создание Migration Service** сделайте следующее:

    - Выберите существующую виртуальную сеть или создайте новую. Виртуальная сеть предоставляет Azure Database Migration Service доступ к базе данных-источнику и целевой среде. См. статью [Краткое руководство. Создание виртуальной сети с помощью портала Azure](../virtual-network/quick-create-portal.md).

    ![Настройка параметров сети экземпляра Azure Database Migration Service](media/quickstart-create-data-migration-service-portal/dms-network-settings.png)

    - Щелкните **Просмотреть и создать**, чтобы создать службу. 
    
    - Через несколько секунд экземпляр Azure Database Migration Service будет создан и готов к использованию:

    ![Созданная служба Migration Service](media/quickstart-create-data-migration-service-portal/dms-service-created.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим кратким руководством, удалите [группу ресурсов Azure](../azure-resource-manager/management/overview.md). Чтобы удалить группу ресурсов, перейдите к созданному экземпляру Azure Database Migration Service. Выберите имя **группы ресурсов** и щелкните **Удалить группу ресурсов**. Это действие удаляет все ресурсы в группе ресурсов, а также саму группу.

## <a name="next-steps"></a>Дальнейшие действия

* [Миграция из SQL Server в Базу данных SQL Azure в автономном режиме](tutorial-sql-server-to-azure-sql.md)
* [Миграция из SQL Server в Базу данных SQL Azure в подключенном режиме](./tutorial-sql-server-to-azure-sql.md)
* [Миграция SQL Server в Управляемый экземпляр SQL Azure в автономном режиме](tutorial-sql-server-to-managed-instance.md)
* [Миграция SQL Server в Управляемый экземпляр SQL Azure по сети](tutorial-sql-server-managed-instance-online.md)