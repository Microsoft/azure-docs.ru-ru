---
title: Руководство по началу работы с анализом данных с помощью выделенных пулов SQL
description: В этом учебнике вы будете использовать пример данных по такси Нью-Йорка для изучения аналитических возможностей пула SQL.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: sql
ms.topic: tutorial
ms.date: 03/24/2021
ms.openlocfilehash: a1f15330a912c8a8a93fe1f74e88ef8d117441c2
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105047904"
---
# <a name="analyze-data-with-dedicated-sql-pools"></a>Анализ данных с помощью выделенных пулов SQL

В этом руководстве показано, как использовать данные о такси Нью-Йорка для изучения возможностей выделенного пула SQL.

## <a name="create-a-dedicated-sql-pool"></a>Создание выделенного пула SQL

1. В Synapse Studio в области навигации слева выберите **Управление** > **Пулы SQL**.
1. Выберите **Создать**.
1. В поле **Имя пула SQL** выберите **SQLPOOL1**.
1. В поле **Уровень производительности** выберите **DW100C**
1. Выберите команду **Просмотреть и создать** > **Создать**. Выделенный пул SQL будет готов через несколько минут. 

Ваш выделенный пул SQL связан с базой данных SQL, также называемой **SQLPOOL1**.
1. Перейдите в область **Данные** > **Рабочая область**.
1. Должна отобразиться база данных с именем **SQLPOOL1**. Если команда не отображается, нажмите кнопку **Обновить**.

Выделенный пул SQL использует платные ресурсы, пока он активен. Позже пул можно будет приостановить, чтобы снизить затраты.

> [!NOTE] 
> При создании выделенного пула SQL (ранее — Хранилище данных SQL) в рабочей области откроется страница подготовки выделенного пула SQL. Подготовка будет выполняться на логическом сервере SQL Server.
## <a name="load-the-nyc-taxi-data-into-sqlpool1"></a>Загрузка данных такси Нью-Йорка в SQLPOOL1

1. В Synapse Studio перейдите в центр **Разработка**, нажмите кнопку **+** для добавления нового ресурса, а затем создайте скрипт SQL.
1. В раскрывающемся списке "Подключение к" над скриптом выберите пул SQLPOOL1 (создан на [шаге 1](./get-started-create-workspace.md) этого руководства).
1. Введите приведенный ниже код.
    ```
    CREATE TABLE [dbo].[Trip]
    (
        [DateID] int NOT NULL,
        [MedallionID] int NOT NULL,
        [HackneyLicenseID] int NOT NULL,
        [PickupTimeID] int NOT NULL,
        [DropoffTimeID] int NOT NULL,
        [PickupGeographyID] int NULL,
        [DropoffGeographyID] int NULL,
        [PickupLatitude] float NULL,
        [PickupLongitude] float NULL,
        [PickupLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
        [DropoffLatitude] float NULL,
        [DropoffLongitude] float NULL,
        [DropoffLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
        [PassengerCount] int NULL,
        [TripDurationSeconds] int NULL,
        [TripDistanceMiles] float NULL,
        [PaymentType] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
        [FareAmount] money NULL,
        [SurchargeAmount] money NULL,
        [TaxAmount] money NULL,
        [TipAmount] money NULL,
        [TollsAmount] money NULL,
        [TotalAmount] money NULL
    )
    WITH
    (
        DISTRIBUTION = ROUND_ROBIN,
        CLUSTERED COLUMNSTORE INDEX
    );

    COPY INTO [dbo].[Trip]
    FROM 'https://nytaxiblob.blob.core.windows.net/2013/Trip2013/QID6392_20171107_05910_0.txt.gz'
    WITH
    (
        FILE_TYPE = 'CSV',
        FIELDTERMINATOR = '|',
        FIELDQUOTE = '',
        ROWTERMINATOR='0X0A',
        COMPRESSION = 'GZIP'
    )
    OPTION (LABEL = 'COPY : Load [dbo].[Trip] - Taxi dataset');
    ```
1. Нажмите кнопку "Выполнить", чтобы выполнить скрипт.
1. Выполнение скрипта займет менее 60 секунд. Скрипт загружает 2 млн строк данных о такси Нью-Йорка в таблицу с именем **dbo.Trip**.

## <a name="explore-the-nyc-taxi-data-in-the-dedicated-sql-pool"></a>Обзор данных о такси Нью-Йорка в выделенном пуле SQL

1. В Synapse Studio перейдите в центр **Данные**.
1. Выберите **SQLPOOL1** > **Таблицы**. 
3. Щелкните правой кнопкой мыши таблицу **dbo.Trip** и выберите команду **Создать скрипт SQL** > **Выбрать первые 100 строк**.
4. Подождите, пока новый скрипт SQL будет создан и запущен.
5. Обратите внимание, что в верхней части скрипта SQL для параметра **Connect to** (Подключиться к) автоматически задано значение пула SQL с именем **SQLPOOL1**.
6. Замените текст скрипта SQL этим кодом и запустите его.

    ```sql
    SELECT PassengerCount,
          SUM(TripDistanceMiles) as SumTripDistance,
          AVG(TripDistanceMiles) as AvgTripDistance
    FROM  dbo.Trip
    WHERE TripDistanceMiles > 0 AND PassengerCount > 0
    GROUP BY PassengerCount
    ORDER BY PassengerCount;
    ```

    Этот запрос показывает, как общее время поездки и среднее расстояние поездки связаны с количеством пассажиров.
1. В окне результатов скрипта SQL измените представление с **Обзор** на **График**, чтобы увидеть визуализацию результатов в виде графика.
    
## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Анализ данным в учетной записи хранения Azure](get-started-analyze-storage.md)
