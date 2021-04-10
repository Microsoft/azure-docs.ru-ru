---
title: Добавочное копирование данных с помощью решения "Отслеживание измененных данных"
description: В этом руководстве вы создадите конвейер Фабрики данных Azure, который пошагово копирует разностные данные из таблицы в базе данных SQL Azure управляемого экземпляра в Хранилище Azure.
ms.author: nihurt
author: hurtn
ms.service: data-factory
ms.topic: tutorial
ms.date: 02/18/2021
ms.openlocfilehash: a00ec8698b188b8fa87935e498e8cfab3aeab5aa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101724988"
---
# <a name="incrementally-load-data-from-azure-sql-managed-instance-to-azure-storage-using-change-data-capture-cdc"></a>Добавочная загрузка данных из управляемого экземпляра SQL Azure в хранилище Azure с использованием технологии "Отслеживание измененных данных" (CDC)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Из этого руководства вы узнаете, как создать фабрику данных Azure с конвейером, который копирует разностные данные на основе сведений об **отслеживании измененных данных (CDC)** в базе данных SQL Azure управляемого экземпляра в хранилище BLOB-объектов Azure.  

В этом руководстве вы выполните следующие шаги:

> [!div class="checklist"]
> * подготовите исходное хранилище данных;
> * Создали фабрику данных.
> * Создали связанные службы.
> * Создали наборы данных приемника и источника.
> * Создание, отладка и запуск конвейера для проверки измененных данных
> * Изменение данных в таблице-источнике
> * Создание, запуску и мониторинг конвейера добавочного копирования

## <a name="overview"></a>Обзор
Для определения измененных данных используется технология "Отслеживание измененных данных", поддерживаемая такими хранилищами данных, как Управляемый экземпляр (MI) SQL Azure и SQL Server.  В этом руководстве описано, как использовать службу "Фабрика данных Azure" с технологией "Отслеживание измененных данных" (SQL) для добавочной загрузки разностных данных из Управляемого экземпляра SQL Azure в хранилище BLOB-объектов Azure.  Дополнительные сведения о технологии "Отслеживание измененных данных" (SQL) см. в статье [Об отслеживании измененных данных (SQL Server)](/sql/relational-databases/track-changes/about-change-data-capture-sql-server).

## <a name="end-to-end-workflow"></a>Комплексный рабочий процесс
Ниже описан стандартный рабочий процесс по добавочной загрузке данных с помощью технологии "Отслеживание измененных данных".

> [!NOTE]
> И Управляемый экземпляр SQL Azure, и SQL Server поддерживают технологию "Отслеживание измененных данных". В этом руководстве в качестве исходного хранилища данных используется управляемый экземпляр SQL Azure. Вы также можете использовать локальную среду SQL Server.

## <a name="high-level-solution"></a>Общее решение
В этом руководстве описано, как создать конвейер, который выполняет следующую операцию:  

   1. Создайте **действие уточнения**, чтобы подсчитать количество измененных записей в таблице CDC базы данных SQL и передать их в действие условия ЕСЛИ.
   2. Создайте **условие ЕСЛИ**, чтобы проверить наличие измененных записей и, если это так, вызвать действие копирования.
   3. Создайте **действие копирования**, чтобы скопировать вставленные/обновленные/удаленные данные между таблицей CDC и хранилищем BLOB-объектов Azure.

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования
* **Управляемый экземпляр базы данных SQL Azure**. Используйте базу данных как **исходное** хранилище данных. Если у вас нет управляемого экземпляра базы данных SQL Azure, вы можете создать ее, выполнив шаги из статьи [Создание управляемого экземпляра базы данных SQL Azure на портале Azure](../azure-sql/managed-instance/instance-create-quickstart.md).
* **Учетная запись хранения Azure.** В этом руководстве в качестве **приемника** будет использоваться хранилище BLOB-объектов. Если у вас нет учетной записи хранения Azure, ознакомьтесь с разделом [Создание учетной записи хранения](../storage/common/storage-account-create.md). Создание контейнера с именем **raw**. 

### <a name="create-a-data-source-table-in-azure-sql-database"></a>Создание таблицы источника данных в Базе данных SQL Azure

1. Запустите **SQL Server Management Studio** и подключитесь к серверу управляемого экземпляра SQL Azure.
2. В **обозревателе сервера** щелкните правой кнопкой мыши **базу данных** и выберите **Создать запрос**.
3. Выполните указанную ниже команду SQL для базы данных управляемых экземпляров SQL Azure, чтобы создать таблицу с именем `customers` в качестве хранилища источника данных.  

    ```sql
    create table customers 
    (
    customer_id int, 
    first_name varchar(50), 
    last_name varchar(50), 
    email varchar(100), 
    city varchar(50), CONSTRAINT "PK_Customers" PRIMARY KEY CLUSTERED ("customer_id") 
     );
    ```
4. Включите функцию **Отслеживание измененных данных** для базы данных и исходной таблицы (customers), выполнив следующий SQL-запрос:

    > [!NOTE]
    > - Замените &lt;имя вашей исходной схемы&gt; схемой управляемого экземпляра SQL Azure с таблицей Customers.
    > - Система отслеживания измененных данных не выполняет никаких действий в рамках транзакций, изменяющих отслеживаемую таблицу. Вместо этого операции вставки, обновления и удаления записываются в журнал транзакций. Данные, хранящиеся в таблицах изменений, будут неуправляемо расти, если периодически и систематически не усекать эти данные. Дополнительные сведения см. в разделе [Включение отслеживания измененных данных для базы данных](/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server#enable-change-data-capture-for-a-database)

    ```sql
    EXEC sys.sp_cdc_enable_db 
    
    EXEC sys.sp_cdc_enable_table
    @source_schema = 'dbo',
    @source_name = 'customers', 
    @role_name = 'null',
    @supports_net_changes = 1
    ```
5. Вставьте данные в таблицу Customers, выполнив следующую команду:

    ```sql
     insert into customers 
         (customer_id, first_name, last_name, email, city) 
     values 
         (1, 'Chevy', 'Leward', 'cleward0@mapy.cz', 'Reading'),
         (2, 'Sayre', 'Ateggart', 'sateggart1@nih.gov', 'Portsmouth'),
        (3, 'Nathalia', 'Seckom', 'nseckom2@blogger.com', 'Portsmouth');
    ```

    > [!NOTE]
    > Перед включением системы отслеживания измененных данных никакие исторические изменения в таблице не фиксируются.

## <a name="create-a-data-factory"></a>Создание фабрики данных

1. Запустите веб-браузер **Microsoft Edge** или **Google Chrome**. Сейчас только эти браузеры поддерживают пользовательский интерфейс фабрики данных.
1. В меню слева выберите **Создать ресурс** > **Данные и аналитика** > **Фабрика данных**.

   ![Выбор фабрики данных в области "Создать"](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-azure-data-factory-menu.png)

2. На странице **Новая фабрика данных** введите **ADFTutorialDataFactory** в поле **Имя**.

     ![Страница "Новая фабрика данных"](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-azure-data-factory.png)

   Имя фабрики данных Azure должно быть **глобально уникальным**. При возникновении указанной ниже ошибки измените имя фабрики данных (например, на ваше_имя_ADFTutorialDataFactory) и попробуйте создать фабрику данных снова. Ознакомьтесь со статьей [Фабрика данных Azure — правила именования](naming-rules.md), чтобы узнать правила именования для артефактов службы "Фабрика данных".

    *Имя фабрики данных "ADFTutorialDataFactory" недоступно.*
3. Укажите **V2** при выборе **версии**.
4. Выберите **подписку** Azure, в рамках которой вы хотите создать фабрику данных.
5. Для **группы ресурсов** выполните одно из следующих действий.

   1. Выберите **Использовать существующую** и укажите существующую группу ресурсов в раскрывающемся списке.
   2. Выберите **Создать новую** и укажите имя группы ресурсов.   
         
    Сведения о группах ресурсов см. в статье, где описывается [использование групп ресурсов для управления ресурсами Azure](../azure-resource-manager/management/overview.md).  
5. Укажите **расположение** фабрики данных. В раскрывающемся списке отображаются только поддерживаемые расположения. Хранилища данных (служба хранилища Azure, база данных SQL Azure и т. д.) и вычисления (HDInsight и т. д.), используемые фабрикой данных, могут располагаться в других регионах.
6. Снимите флажок **включить GIT**.     
7. Нажмите кнопку **Создать**.
8. Когда развертывание будет завершено, выберите **Перейти к ресурсу**

   ![Снимок экрана: сообщение о том, что развертывание выполнено, с возможностью перейти к ресурсу.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/data-factory-deploy-complete.png)
9. Когда завершится создание, откроется страница **Фабрика данных**, как показано на рисунке ниже.

   ![Снимок экрана: развернутая фабрика данных.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/data-factory-home-page.png)
10. Щелкните плитку **Создание и мониторинг**, чтобы открыть на отдельной вкладке пользовательский интерфейс фабрики данных Azure.
11. На странице **начала работы** откройте вкладку **Изменить** на панели слева, как показано на следующем рисунке:

    ![Кнопка создания конвейера](./media/tutorial-incremental-copy-change-data-capture-feature-portal/get-started-page.png)

## <a name="create-linked-services"></a>Создание связанных служб
Связанная служба в фабрике данных связывает хранилища данных и службы вычислений с фабрикой данных. В этом разделе вы создадите связанные службы для учетной записи хранения Azure и управляемого экземпляра SQL Azure.

### <a name="create-azure-storage-linked-service"></a>Создание связанной службы хранилища Azure
На этом шаге вы свяжете учетную запись хранения Azure с фабрикой данных.

1. Нажмите кнопку **Подключения**, а затем **+ Создать**.

   ![Кнопка создания подключения](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-connection-button-storage.png)
2. В окне **New Linked Service** (Новая связанная служба) выберите **хранилище BLOB-объектов Azure** и щелкните **Continue** (Продолжить).

   ![Выбор хранилища BLOB-объектов Azure](./media/tutorial-incremental-copy-change-data-capture-feature-portal/select-azure-storage.png)
3. В окне **New Linked Service** (Новая связанная служба) выполните следующие действия:

   1. Введите **AzureStorageLinkedService** в поле **имени**.
   2. Выберите учетную запись хранения в списке **Storage account name** (Имя учетной записи хранения).
   3. Выберите команду **Сохранить**.

   ![Параметры учетной записи хранения Azure](./media/tutorial-incremental-copy-change-data-capture-feature-portal/azure-storage-linked-service-settings.png)


### <a name="create-azure-sql-mi-database-linked-service"></a>Создание связанной службы базы данных управляемого экземпляра SQL Azure.
На этом шаге вы свяжете базу данных управляемого экземпляра SQL Azure с фабрикой данных.

> [!NOTE]
> Для тех, кто использует SQL MI, см. [здесь](./connector-azure-sql-managed-instance.md#prerequisites) сведения о доступе через публичную и частную конечную точку. При использовании частной конечной точки необходимо запустить этот конвейер с помощью локальной среды выполнения интеграции. Это же относится к тем, у кого запущен SQL Server локально, в сценариях виртуальной машины или виртуальной сети.

1. Нажмите кнопку **Подключения**, а затем **+ Создать**.
2. В окне **New Linked Service** (Новая связанная служба) выберите **Azure SQL Database Managed Instance** (Управляемый экземпляр базы данных SQL Microsoft Azure) и щелкните **Continue** (Продолжить).
3. В окне **New Linked Service** (Новая связанная служба) выполните следующие действия:

   1. Введите **AzureSqlMI1** в поле **Имя**.
   2. Выберите нужный SQL-сервер в поле **Имя сервера**.
   4. Выберите базу данных SQL в поле **Имя базы данных**.
   5. Введите имя пользователя в поле **User name** (Имя пользователя).
   6. Введите пароль для этого пользователя в поле **Password** (Пароль).
   7. Нажмите кнопку **Test connection** (Проверить подключение), чтобы проверить подключение.
   8. Нажмите кнопку **Save** (Сохранить), чтобы сохранить связанную службу.

   ![Настройки связанной службы базы данных SQL Azure MI](./media/tutorial-incremental-copy-change-data-capture-feature-portal/azure-sql-managed-instance-database-linked-service-settings.png)

## <a name="create-datasets"></a>Создание наборов данных
На этом шаге вы создадите наборы данных, которые представляют данные источника и приемника.

### <a name="create-a-dataset-to-represent-source-data"></a>Создание набора данных для представления исходных данных
На этом шаге вы создадите набор данных для представления исходных данных.

1. В представлении в виде дерева щелкните значок **+ (плюс)** и выберите вариант **Набор данных**.

   ![Меню "Новый набор данных"](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-dataset-menu.png)
2. Выберите **Azure SQL Database Managed Instance** (Управляемый экземпляр базы данных SQL Microsoft Azure) и щелкните **Continue** (Продолжить).

   ![Тип исходного набора данных — база данных SQL Azure](./media/tutorial-incremental-copy-change-data-capture-feature-portal/select-azure-sql-database.png)
   
3. На вкладке **Свойства набора** задайте имя набора данных и сведения о соединении:
 
   1. Выберите **AzureSqlMI1** для **связанной службы**.
   2. Выберите **[dbo]. [dbo_customers_CT]** в качестве **имени таблицы**.  Примечание. Эта таблица была автоматически создана при включении CDC в таблице Customers. Измененные данные никогда не запрашиваются из этой таблицы напрямую, а извлекаются с помощью функций [CDC](/sql/relational-databases/system-functions/change-data-capture-functions-transact-sql).

   ![Подключение к источнику](./media/tutorial-incremental-copy-change-data-capture-feature-portal/source-dataset-configuration.png)

### <a name="create-a-dataset-to-represent-data-copied-to-sink-data-store"></a>Создайте набор данных, который будет представлять данные для копирования в целевое хранилище данных.
На этом шаге вы создадите набор данных для представления данных, которые копируются из исходного хранилища данных. Для работы с этим руководством нужен контейнер data lake, созданный в хранилище BLOB-объектов Azure. Создайте контейнер (если его еще нет) или присвойте ему имя имеющегося контейнера. В этом руководстве имя файла выходных данных создается динамически с помощью времени триггера, который будет настроен позже.

1. В представлении в виде дерева щелкните значок **+ (плюс)** и выберите вариант **Набор данных**.

   ![Меню "Новый набор данных"](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-dataset-menu.png)
2. Выберите **Хранилище BLOB-объектов Azure** и щелкните **Продолжить**.

   ![Тип целевого набора данных — хранилище BLOB-объектов Azure](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-type.png)
3. Выберите **DelimitedText** в качестве формата и щелкните **Продолжить**.

   ![Формат набора данных приемника — DelimitedText](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-format.png)
4. На вкладке **Свойства набора** задайте имя набора данных и сведения о соединении:

   1. Выберите **AzureStorageLinkedService** в списке **Связанная служба**.
   2. Введите **raw** для части **контейнера** **filePath**.
   3. Включите **Использовать первую строку в качестве заголовка**
   4. Нажмите кнопку **ОК**.

   ![Целевой набор данных — подключение](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration.png)

## <a name="create-a-pipeline-to-copy-the-changed-data"></a>Создание конвейера для копирования измененных данных
На этом шаге вы создадите конвейер, который сначала проверяет число измененных записей, имеющихся в таблице изменений, с помощью **действия уточнения**. Условие ЕСЛИ проверяет, что число измененных записей больше нуля, и запускает действие **копирования**, чтобы скопировать вставленные/обновленные/удаленные данные из базы данных SQL Azure в хранилище BLOB-объектов Azure. Наконец, триггер смены окна настроен на время начала и окончания, которые будут переданы действиям в качестве параметров начального и конечного окон. 

1. В пользовательском интерфейсе фабрики данных переключитесь на вкладку **Правка**. Щелкните значок **+ (плюс)** в левой панели и выберите **Конвейер**.

    ![Меню создания конвейера](./media/tutorial-incremental-copy-change-data-capture-feature-portal/new-pipeline-menu.png)
2. Вы увидите новую вкладку для настройки конвейера. Также этот конвейер появится в отображении дерева. В окне **Свойства** укажите имя **IncrementalCopyPipeline** для нового конвейера.

    ![Имя конвейера](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-name.png)
3. Разверните элемент **Общие** на панели **Действия** и перетащите действие **Поиск** в область конструктора конвейера. Для имени действия задайте значение **GetChangeCount**. Это действие получает количество записей в таблице изменений для заданного временного окна.

    ![Действие поиска — имя](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-name.png)
4. Перейдите на вкладку **Настройки** в окне **Свойства**:
   1. Укажите имя набора данных SQL MI для поля **Исходный набор данных** .
   2. Выберите параметр запроса и введите в поле запроса следующее:
    ```sql
    DECLARE  @from_lsn binary(10), @to_lsn binary(10);  
    SET @from_lsn =sys.fn_cdc_get_min_lsn('dbo_customers');  
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn('largest less than or equal',  GETDATE());
    SELECT count(1) changecount FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, 'all')
    ```
   3. Включите **Только первая строка**

    ![Действие поиска — настройки](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-settings.png)
5. Нажмите кнопку **Предварительный просмотр данных**, чтобы обеспечить получение достоверных выходных данных действием поиска

    ![Действие поиска — предварительный просмотр](./media/tutorial-incremental-copy-change-data-capture-feature-portal/first-lookup-activity-preview.png)
6. На панели **Действия** разверните элемент **Итерация и условия**, а затем перетащите действие **If Condition** в область конструктора конвейера. Для имени действия задайте значение **HasChangedRows**. 

    ![Действие условия If — имя](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-name.png)
7. Перейдите на вкладку **Действия** в окне **Свойства**:

   1. Введите следующее **Выражение**
   
    ```adf
    @greater(int(activity('GetChangeCount').output.firstRow.changecount),0)
    ```

   2. Щелкните значок с изображением карандаша, чтобы изменить истинное условие.

   ![Действие условия If — настройки](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-setting.png)

   3. Разверните **Общие** в панели элементов **Действия** и перетащите действие **Ждать** в область конструктора конвейера. Это временное действие для отладки условия If; оно будет изменено далее в этом руководстве. 

   ![Условие ЕСЛИ верно — ждать](./media/tutorial-incremental-copy-change-data-capture-feature-portal/if-condition-activity-wait.png)

   4. Щелкните элемент в пути IncrementalCopyPipeline, чтобы вернуться к основному конвейеру.

8. Запустите конвейер в режиме **Отладка**, чтобы убедиться, что конвейер успешно выполнен. 

   ![Конвейер — отладка](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug.png)
9. Затем вернитесь к верному шагу условия и удалите действие **Ждать**. На панели элементов **Действия** разверните узел **Переместить и преобразовать** и перетащите действие **Копирование** в область конструктора конвейера. Присвойте этому действию имя **IncrementalCopyActivity**. 

   ![Действие копирования — имя](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-name.png)
10. Откройте вкладку **Источник** в окне **Свойства** и выполните здесь следующие действия.

   1. Укажите имя набора данных SQL MI для поля **Исходный набор данных** . 
   2. Выберите **Запрос** в списке **Use Query** (Пользовательский запрос).
   3. Введите следующий **Запрос**.

      ```sql
      DECLARE @from_lsn binary(10), @to_lsn binary(10); 
      SET @from_lsn =sys.fn_cdc_get_min_lsn('dbo_customers'); 
      SET @to_lsn = sys.fn_cdc_map_time_to_lsn('largest less than or equal', GETDATE());
      SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, 'all')
      ```

   ![Действие копирования — настройки источника](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-settings.png)

11. Нажмите кнопку предварительного просмотра, чтобы убедиться, что запрос правильно возвращает измененные строки.

    ![Снимок экрана: предварительный просмотр для проверки запроса.](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-source-preview.png)
12. Перейдите на вкладку **Приемник** и укажите набор данных Хранилища Azure в поле **Sink Dataset** (Целевой набор данных).

    ![Снимок экрана: вкладка "Приемник".](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-sink-settings.png)
13. Нажмите кнопку назад на главном холсте конвейера и подключите действие **Поиск** к действию **If Condition** по одному. Перетащите **зеленую кнопку**, присоединенную к действию **Поиск**, на блок действия **If Condition**.

    ![Соединение действий поиска и копирования](./media/tutorial-incremental-copy-change-data-capture-feature-portal/connect-lookup-if.png)
14. Нажмите кнопку **Проверить** на панели инструментов. Убедитесь, что проверка завершается без ошибок. Закройте **окно отчета о проверке конвейера**, щелкнув **>>** .

    ![Кнопка проверки](./media/tutorial-incremental-copy-change-data-capture-feature-portal/validate-button.png)
15. Щелкните кнопку "Отладка", чтобы проверить конвейер и убедиться, что файл создается в месте хранения.

    ![Добавочная отладка конвейера — 2](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-2.png)
16. Опубликуйте сущности (связанные службы, наборы данных и конвейеры) в службе "Фабрика данных", нажав кнопку **Опубликовать все**. Дождитесь сообщения **Публикация успешно выполнена**.

    ![Кнопка "Опубликовать"](./media/tutorial-incremental-copy-change-data-capture-feature-portal/publish-button-2.png)    

### <a name="configure-the-tumbling-window-trigger-and-cdc-window-parameters"></a>Настройка триггера переворачивающегося окна и параметра окна CDC 
На этом шаге вы создадите триггер переворачивающегося окна для запуска задания по частому расписанию. Вы будете использовать системные переменные WindowStart и WindowEnd для триггера переворачивающегося окна и передавать их в качестве параметров конвейера для использования в запросе CDC.

1. Перейдите на вкладку **Параметры** в конвейере **IncrementalCopyPipeline** и с помощью кнопки **+ Новый** добавьте два параметра (**triggerStartTime** и **triggerEndTime**) в конвейер, который будет представлять время начала и окончания переворачивающегося окна. В целях отладки добавьте значения по умолчанию в формате **ГГГГ-ММ-ДД ЧЧ24:ММ:СС.ДОЛИСЕКУНД**, но убедитесь, что triggerStartTime не включен в таблицу CDC, в противном случае это приведет к ошибке.

    ![Меню Trigger Now (Активировать сейчас)](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-parameters.png)
2. Перейдите на вкладку "Параметры действия" **Поиск** и настройте запрос на использование параметров начала и окончания. Скопируйте следующий код в редактор запросов:
    ```sql
    @concat('DECLARE @begin_time datetime, @end_time datetime, @from_lsn binary(10), @to_lsn binary(10); 
    SET @begin_time = ''',pipeline().parameters.triggerStartTime,''';
    SET @end_time = ''',pipeline().parameters.triggerEndTime,''';
    SET @from_lsn = sys.fn_cdc_map_time_to_lsn(''smallest greater than or equal'', @begin_time);
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn(''largest less than'', @end_time);
    SELECT count(1) changecount FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, ''all'')')
    ```

3. Перейдите к действию **Копировать** в случае верного действия **If Condition** и щелкните по вкладке **Источник**. Скопируйте следующий код в редактор запросов:
    ```sql
    @concat('DECLARE @begin_time datetime, @end_time datetime, @from_lsn binary(10), @to_lsn binary(10); 
    SET @begin_time = ''',pipeline().parameters.triggerStartTime,''';
    SET @end_time = ''',pipeline().parameters.triggerEndTime,''';
    SET @from_lsn = sys.fn_cdc_map_time_to_lsn(''smallest greater than or equal'', @begin_time);
    SET @to_lsn = sys.fn_cdc_map_time_to_lsn(''largest less than'', @end_time);
    SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_customers(@from_lsn, @to_lsn, ''all'')')
    ```
4. Перейдите на вкладку **Приемник** действия **Копирование** и щелкните кнопку **Открыть**, чтобы изменить свойства набора данных. Перейдите на вкладку **Параметры** и добавьте новый параметр с именем **triggerStart**    

    ![Снимок экрана: добавление нового параметра на вкладке "Параметры".](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-2.png)
5. Затем настройте свойства набора данных для хранения данных в подкаталоге **клиенты/добавочный** с разделением по данным.
   1. Перейдите на вкладку **Подключение** в свойствах набора данных и добавьте динамическое содержимое для разделов **Каталог** и **Файл**. 
   2. Введите следующее выражение в разделе **Каталог**, щелкнув на динамическое содержимое в текстовом поле:
    
    ```sql
    @concat('customers/incremental/',formatDateTime(dataset().triggerStart,'yyyy/MM/dd'))
    ```
   3. Введите следующее выражение в разделе **Файл**. Это создаст имена файлов на основе даты и времени начала триггера с расширением CSV:
    
    ```sql
    @concat(formatDateTime(dataset().triggerStart,'yyyyMMddHHmmssfff'),'.csv')
    ```
    ![Конфигурация набора данных приемника — 3](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-3.png)

   4. Вернитесь к параметрам **Приемника** в действии **Копирование**, щелкнув вкладку **IncrementalCopyPipeline**. 
   5. Разверните свойства набора данных и введите динамическое содержимое в значение параметра triggerStart с помощью следующего выражения:
     ```sql
     @pipeline().parameters.triggerStartTime
     ```
    ![Конфигурация набора данных приемника — 4](./media/tutorial-incremental-copy-change-data-capture-feature-portal/sink-dataset-configuration-4.png)

6. Щелкните кнопку "Отладка", чтобы проверить конвейер и убедиться, что структура папок и выходной файл созданы должным образом. Скачайте и откройте файл, чтобы проверить его содержимое. 

    ![Отладка инкрементной копии — 3](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-3.png)
7. Убедитесь, что параметры добавлены в запрос, просмотрев входные параметры выполнения конвейера.

    ![Отладка инкрементной копии — 4](./media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-debug-4.png)
8. Опубликуйте сущности (связанные службы, наборы данных и конвейеры) в службе "Фабрика данных", нажав кнопку **Опубликовать все**. Дождитесь сообщения **Публикация успешно выполнена**.
9. Наконец, настройте триггер переворачивающегося окна для запуска конвейера через обычный интервал и задайте параметры времени начала и окончания. 
   1. Нажмите кнопку **Добавить триггер** и выберите **Создать/изменить**

   ![Добавить новый триггер](./media/tutorial-incremental-copy-change-data-capture-feature-portal/add-trigger.png)

   2. Введите имя триггера и укажите время начала, равное времени окончания окна отладки выше.

   ![Триггер "переворачивающегося" окна](./media/tutorial-incremental-copy-change-data-capture-feature-portal/tumbling-window-trigger.png)

   3. На следующем экране укажите следующие значения параметров начала и окончания соответственно.
    ```sql
    @formatDateTime(trigger().outputs.windowStartTime,'yyyy-MM-dd HH:mm:ss.fff')
    @formatDateTime(trigger().outputs.windowEndTime,'yyyy-MM-dd HH:mm:ss.fff')
    ```

   ![Триггер "переворачивающегося" окна — 2](./media/tutorial-incremental-copy-change-data-capture-feature-portal/tumbling-window-trigger-2.png)

> [!NOTE]
> Учтите, что триггер будет выполняться только после его публикации. Кроме того, ожидаемым поведением переворачивающегося окна является выполнение всех интервалов с даты начала до настоящего момента. Дополнительные сведения о триггерах переворачивающегося окна можно найти [здесь](./how-to-create-tumbling-window-trigger.md). 
  
10. С помощью **SQL Server Management Studio** внесите некоторые дополнительные изменения в таблицу Customer, выполнив следующий SQL:
    ```sql
    insert into customers (customer_id, first_name, last_name, email, city) values (4, 'Farlie', 'Hadigate', 'fhadigate3@zdnet.com', 'Reading');
    insert into customers (customer_id, first_name, last_name, email, city) values (5, 'Anet', 'MacColm', 'amaccolm4@yellowbook.com', 'Portsmouth');
    insert into customers (customer_id, first_name, last_name, email, city) values (6, 'Elonore', 'Bearham', 'ebearham5@ebay.co.uk', 'Portsmouth');
    update customers set first_name='Elon' where customer_id=6;
    delete from customers where customer_id=5;
    ```
11. Нажмите кнопку **Опубликовать все**. Дождитесь сообщения **Публикация успешно выполнена**.  
12. Через несколько минут сработает триггер конвейера, а новый файл будет загружен в службу хранилища Azure


### <a name="monitor-the-incremental-copy-pipeline"></a>Мониторинг конвейера добавочного копирования
1. Щелкните вкладку **Мониторинг** слева. В открывшемся списке вы увидите запуск конвейера и его текущее состояние. Щелкните **Refresh** (Обновить), чтобы обновить этот список. Наведите курсор мыши на имя конвейера, чтобы получить доступ к действию и отчету о потреблении.

    ![Запуски конвейера](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-pipeline-runs.png)
2. Чтобы просмотреть запуски действий, связанные с этим запуском конвейера, щелкните имя Конвейера. Если обнаружены измененные данные, в списке будут три действия, включая действие копирования, в противном случае в списке останутся только две записи. Чтобы вернуться к просмотру запусков конвейера, щелкните ссылку **Все конвейеры** в верхней части окна.

    ![Выполнение действия](./media/tutorial-incremental-copy-change-data-capture-feature-portal/copy-activity-runs.png)


### <a name="review-the-results"></a>Просмотр результатов
Вы увидите второй файл в папке `customers/incremental/YYYY/MM/DD` контейнера `raw`.

![Выходной файл, полученный в результате добавочного копирования](media/tutorial-incremental-copy-change-data-capture-feature-portal/incremental-copy-pipeline-run.png)
 

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве рассказывается о копировании новых и измененных файлов на основе параметра LastModifiedDate:

> [!div class="nextstepaction"]
>[Copy new files by lastmodifieddate](tutorial-incremental-copy-lastmodified-copy-data-tool.md) (Копирование новых файлов с использованием параметра LastModifiedDate)
