---
title: Краткое руководство. Создание выделенного пула SQL (ранее — Хранилище данных SQL) и отправка в него запросов (портал Azure)
description: Создание выделенного пула SQL (ранее — Хранилище данных SQL) и отправка в него запросов с помощью портала Azure
services: synapse-analytics
author: pimorano
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 05/28/2019
ms.author: pimorano
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: dd9d66683d5f2349807915651c50bf9eedd285c1
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104602236"
---
# <a name="quickstart-create-and-query-a-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics-using-the-azure-portal"></a>Краткое руководство. Создание выделенного пула SQL (ранее — Хранилище данных SQL) в Azure Synapse Analytics и отправка в него запросов с помощью портала Azure

Вы можете быстро создать выделенный пул SQL (ранее — Хранилище данных SQL) в Azure Synapse Analytics и отправлять в него запросы с помощью портала Azure.

## <a name="prerequisites"></a>Предварительные требования

1. Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

   > [!NOTE]
   > Создание выделенного пула SQL (ранее — Хранилище данных SQL) в Azure Synapse может повлечь дополнительные расходы. Дополнительные сведения см. на странице [цен на Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/synapse-analytics/).

2. Скачайте и установите последнюю версию [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS). Примечание. Среда SSMS доступна только на платформах Windows. Полный список поддерживаемых платформ см. [здесь](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15&preserve-view=true#supported-operating-systems-ssms-185t).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-a-sql-pool"></a>Создание пула SQL

Хранилища данных создаются с помощью выделенного пула SQL (ранее — Хранилище данных SQL) в Azure Synapse Analytics. Выделенный пул SQL (ранее — Хранилище данных SQL) создается с определенным набором [вычислительных ресурсов](memory-concurrency-limits.md). База данных создается в пределах [группы ресурсов Azure](../../azure-resource-manager/management/overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) и [логического сервера SQL](../../azure-sql/database/logical-servers.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

Следуйте приведенным инструкциям по созданию выделенного пула SQL (ранее — Хранилище данных SQL), содержащего пример данных **AdventureWorksDW**.

1. Щелкните **Создать ресурс** в верхнем левом углу окна портала Azure.

   ![Создание ресурса на портале Azure](./media/create-data-warehouse-portal/create-a-resource.png)

2. В строке поиска введите "выделенный пул SQL" и выберите выделенный пул SQL (ранее — Хранилище данных SQL). На открывшейся странице выберите **Создать**.

   ![создание пустого хранилища данных](./media/create-data-warehouse-portal/create-a-data-warehouse.png)

3. На вкладке **Основные сведения** укажите подписку, группу ресурсов, имя выделенного пула SQL (ранее — Хранилище данных SQL), а также имя сервера.

   | Параметр | Рекомендуемое значение | Описание |
   | :------ | :-------------- | :---------- |
   | **Подписка** | Ваша подписка | Дополнительные сведения о подписках см. [здесь](https://account.windowsazure.com/Subscriptions). |
   | **Группа ресурсов** | myResourceGroup | Допустимые имена групп ресурсов см. в статье о [правилах и ограничениях именования](/azure/architecture/best-practices/resource-naming?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). |
   | **Имя пула SQL** | Любое глобально уникальное имя (например, *mySampleDataWarehouse*). | Допустимые имена баз данных см. в статье об [идентификаторах базы данных](/sql/relational-databases/databases/database-identifiers?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  |
   | **Server** | Любое глобально уникальное имя | Выберите существующий сервер или создайте сервер с новым именем, щелкнув ссылку **Создать новый**. Допустимые имена серверов см. в статье о [правилах и ограничениях именования](/azure/architecture/best-practices/resource-naming?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). |

   ![Основные сведения при создании хранилища данных](./media/create-data-warehouse-portal/create-sql-pool-basics.png)

4. В подразделе **Уровень производительности** щелкните ссылку **Выберите уровень производительности**, чтобы дополнительно изменить конфигурацию с помощью ползунка.

   ![Изменение уровня производительности хранилища данных](./media/create-data-warehouse-portal/create-sql-pool-performance-level.png)  

   Дополнительные сведения об уровне производительности см. в статье [Управление вычислительными ресурсами в Azure Synapse Analytics](sql-data-warehouse-manage-compute-overview.md).

5. В разделе **Использовать существующие данные** выберите **Дополнительные параметры** и щелкните **Пример**, чтобы создать пример базы данных AdventureWorksDW.

    ![Выберите "Использовать существующие данные"](./media/create-data-warehouse-portal/create-sql-pool-additional-1.png)

6. Заполнив форму Azure Synapse Analytics на вкладке "Основные сведения", выберите элемент **Просмотр и создание**, а затем нажмите кнопку **Создать**, чтобы создать пул SQL. Подготовка занимает несколько минут.

   ![Нажатая кнопка "Просмотреть и создать"](./media/create-data-warehouse-portal/create-sql-pool-review-create.png)

   ![Выбор создания](./media/create-data-warehouse-portal/create-sql-pool-create.png)

7. На панели инструментов щелкните значок **Уведомления**, чтобы отслеживать процесс развертывания.

   ![Снимок экрана: вкладка "Уведомления" с сообщением о выполнении развертывания](./media/create-data-warehouse-portal/notification.png)

## <a name="create-a-server-level-firewall-rule"></a>создадим правило брандмауэра на уровне сервера;

Служба Azure Synapse создает брандмауэр на уровне сервера. Этот брандмауэр предотвращает подключение внешних приложений и инструментов к серверу или базам данных на сервере. Чтобы разрешить это подключение, можно добавить правила брандмауэра, открывающие подключение для определенных IP-адресов. Выполните приведенные ниже действия, чтобы создать [правило брандмауэра уровня сервера](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) для IP-адреса клиента.

> [!NOTE]
> Azure Synapse обменивается данными через порт 1433. Если вы пытаетесь подключиться из корпоративной сети, то исходящий трафик через порт 1433 может быть запрещен сетевым брандмауэром. В таком случае вы не сможете подключиться к серверу, пока ваш ИТ-отдел не откроет порт 1433.

1. После завершения развертывания в меню слева выберите **Все службы**. Щелкните значок **Базы данных**, затем выберите значок звездочки рядом с полем **Azure Synapse Analytics**, чтобы добавить Azure Synapse Analytics в избранное.

2. Выберите **Azure Synapse Analytics** в меню слева, а затем — **mySampleDataWarehouse** на странице **Azure Synapse Analytics**. После этого откроется страница обзора базы данных, где будет указано полное имя сервера (например, **sqlpoolservername.database.windows.net**) и будут предоставлены параметры для дальнейшей настройки.

3. Скопируйте полное имя сервера. Оно понадобится для подключения к серверу и размещенным на нем базам данных при работе с этим и последующими краткими руководствами. Щелкните имя сервера, чтобы открыть параметры этого сервера.

   ![поиск имени сервера](./media/create-data-warehouse-portal/find-server-name.png)

4. Щелкните ссылку **Показать параметры брандмауэра**.

   ![параметры сервера](./media/create-data-warehouse-portal/server-settings.png)

5. Откроется страница **Параметры брандмауэра** сервера.

   ![правило брандмауэра для сервера](./media/create-data-warehouse-portal/server-firewall-rule.png)

6. На панели инструментов нажмите кнопку **Добавить IP-адрес клиента**, чтобы добавить текущий IP-адрес в новое правило брандмауэра. С использованием правила брандмауэра можно открыть порт 1433 для одного IP-адреса или диапазона IP-адресов.

7. Щелкните **Save** (Сохранить). Для текущего IP-адреса будет создано правило брандмауэра на уровне сервера, с помощью которого можно открыть порт 1433 на сервере.

8. Нажмите кнопку **ОК**, а затем закройте страницу **Параметры брандмауэра**.

Теперь с помощью этого IP-адреса можно подключиться к серверу и его пулам SQL. Подключение выполняется из SQL Server Management Studio или другого инструмента на ваше усмотрение. При подключении используйте созданную ранее учетную запись ServerAdmin.

> [!IMPORTANT]
> По умолчанию доступ через брандмауэр базы данных SQL включен для всех служб Azure. На этой странице щелкните положение **Выкл.** переключателя, а затем нажмите кнопку **Сохранить**, чтобы отключить брандмауэр для всех служб Azure.

## <a name="get-the-fully-qualified-server-name"></a>Получение полного имени сервера

Получите полное имя сервера на портале Azure. Позже это полное имя понадобится при подключении к серверу.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Выберите элемент **Azure Synapse Analytics** в меню слева, а затем выберите нужное значение на странице **Azure Synapse Analytics**.

3. На странице портала Azure вашей базы данных в области **Основные компоненты** найдите и скопируйте **имя сервера**. В этом примере полное имя — sqlpoolservername.database.windows.net.

    ![Сведения о подключении](./media/create-data-warehouse-portal/find-server-name.png)

## <a name="connect-to-the-server-as-server-admin"></a>Подключение к серверу от имени администратора сервера

В этом разделе для подключения к серверу используется [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS).

1. Откройте среду SQL Server Management Studio.

2. В диалоговом окне **Соединение с сервером** введите следующие данные:

   | Параметр | Рекомендуемое значение | Описание |
   | :------ | :-------------- | :---------- |
   | Тип сервера | Ядро СУБД | Это обязательное значение |
   | Имя сервера | Полное имя сервера | Пример: **sqlpoolservername.database.windows.net**. |
   | Аутентификация | Проверка подлинности SQL Server | В рамках работы с этим руководством мы настроили только один тип аутентификации — аутентификацию SQL. |
   | Имя входа | Учетная запись администратора сервера | Это учетная запись, указанная при создании сервера. |
   | Пароль | Пароль для учетной записи администратора сервера | Пароль, указанный при создании сервера. |
   ||||

   ![подключение к серверу](./media/create-data-warehouse-portal/connect-to-server-ssms.png)

3. Нажмите кнопку **Подключиться**. В SSMS открывается окно обозревателя объектов.

4. В обозревателе объектов разверните узел **Базы данных**. Затем разверните папку **mySampleDatabase**, чтобы просмотреть объекты в новой базе данных.

   ![объекты базы данных](./media/create-data-warehouse-portal/connected-ssms.png)

## <a name="run-some-queries"></a>Выполнение запросов

Не рекомендуем выполнять большие запросы при входе в систему в качестве администратора сервера, так как при этом используется [ограниченный класс ресурсов](resource-classes-for-workload-management.md). Вместо этого настройте [изоляцию рабочей нагрузки](./quickstart-configure-workload-isolation-tsql.md), как [показано в руководствах](./load-data-wideworldimportersdw.md#create-a-user-for-loading-data).

В Azure Synapse Analytics в качестве языка запросов используется T-SQL. Чтобы открыть окно запросов и выполнить несколько запросов T-SQL, выполните следующие действия.

1. Щелкните правой кнопкой мыши **mySampleDataWarehouse** и выберите команду **Создать запрос**. Откроется новое окно запроса.

2. В окне запросов введите приведенную ниже команду, чтобы просмотреть список баз данных.

    ```sql
    SELECT * FROM sys.databases
    ```

3. Щелкните **Выполнить**. Результаты запроса содержат две базы данных: **master** и **mySampleDataWarehouse**.

   ![Отправка запросов к базам данных](./media/create-data-warehouse-portal/query-databases.png)

4. Чтобы просмотреть некоторые данные, используйте следующую команду. Она позволяет узнать количество клиентов с фамилией "Adams", у которых трое детей. Список результатов содержит шесть клиентов.

    ```sql
    SELECT LastName, FirstName FROM dbo.dimCustomer
    WHERE LastName = 'Adams' AND NumberChildrenAtHome = 3;
    ```

   ![Отправка запроса к dbo.dimCustomer](./media/create-data-warehouse-portal/query-customer.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Плата взимается за единицы хранилища данных и данные, хранящиеся в выделенном пуле SQL (ранее — Хранилище данных SQL). Плата за вычислительные ресурсы и ресурсы хранилища взимается отдельно.

- Если вы хотите сохранить данные в хранилище, то можете приостановить работу вычислительных ресурсов, когда не используете выделенный пул SQL (ранее — Хранилище данных SQL). При приостановке вычислений плата взимается только за хранение данных. Когда вы будете готовы работать с данными, можно будет возобновить вычисления.

- Если вы хотите исключить будущие расходы, то можете удалить выделенный пул SQL (ранее — Хранилище данных SQL).

Выполните следующие действия, чтобы очистить ресурсы, которые больше не нужны.

1. Войдите на [портал Azure](https://portal.azure.com) и выберите выделенный пул SQL (ранее — Хранилище данных SQL).

   ![Очистка ресурсов](./media/create-data-warehouse-portal/clean-up-resources.png)

2. Чтобы приостановить вычисление, нажмите кнопку **Пауза**. Если работа выделенного пула SQL (ранее — Хранилище данных SQL) приостановлена, вы увидите кнопку **Возобновить**. Чтобы возобновить вычисление, нажмите кнопку **Возобновить**.

3. Чтобы удалить выделенный пул SQL (ранее — Хранилище данных SQL) во избежание дальнейших платежей за вычисление или хранение, нажмите кнопку **Удалить**.

4. Чтобы удалить созданный вами сервер, выберите сервер **sqlpoolservername.database.windows.net**, выделенный на предыдущем изображении, и нажмите кнопку **Удалить**. Будьте внимательны, так как удаление сервера приведет к удалению всех баз данных, назначенных этому серверу.

5. Чтобы удалить группу ресурсов, выберите **myResourceGroup**, а затем **Удалить группу ресурсов**.

Хотите оптимизировать и сократить ваши расходы на облако?

[!INCLUDE [cost-management-horizontal](../../../includes/cost-management-horizontal.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье о [загрузке данных в выделенный пул SQL (ранее — Хранилище данных SQL)](load-data-from-azure-blob-storage-using-copy.md).