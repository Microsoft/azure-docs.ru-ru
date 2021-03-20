---
title: Сборка и развертывание модели в виртуальной машине SQL Server – командный процесс обработки и анализа данных
description: Создание и развертывание модели машинного обучения с помощью SQL Server на виртуальной машине Azure с общедоступным набором данных.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 580181aaaea975ee07bcec8108297079c5373b92
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96007415"
---
# <a name="the-team-data-science-process-in-action-using-sql-server"></a>Процесс обработки и анализа данных группы на практике: использование SQL Server
В этом руководстве представлено пошаговое руководство по процессу создания и развертывания модели машинного обучения с помощью SQL Server и общедоступного набора данных [NYC Taxi Trips](https://www.andresmh.com/nyctaxitrips/). Процедура соответствует стандартному рабочему процессу обработки и аналитики данных: прием и анализ данных, разработка функций для упрощения обучения, а затем сборка и развертывание модели.

## <a name="nyc-taxi-trips-dataset-description"></a><a name="dataset"></a>Описание набора данных о поездках в такси Нью
Данные о поездках в такси Нью составляют около 20 ГБ сжатых CSV-файлов (~ 48 ГБ без сжатия), состоящие из более чем 173 000 000 индивидуальных поездок и году, оплаченных для каждого поездки. Каждая запись о поездке содержит расположение пунктов отправления и назначения и время, анонимизированный номер лицензии водителя и номер медальона (уникальный идентификатор такси). Данные включают в себя все поездки за 2013 год и предоставляются в виде следующих двух наборов данных за каждый месяц:

1. CSV-файл trip_data содержит подробную информацию о поездке, например число пассажиров, пункты отправления и назначения, продолжительность поездки и ее расстояние. Вот несколько примеров записей:
   
    `medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

    `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

2. CSV-файл trip_fare содержит подробную информацию об оплате каждой поездки, такие как тип оплаты, сумма тарифа, надбавка и налоги, чаевые и пошлины, а также общая выплаченная сумма. Вот несколько примеров записей:
   
    `medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

    `89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

    `0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

    `DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

Уникальный ключ для объединения trip\_data и trip\_fare состоит из полей medallion, hack\_licence и pickup\_datetime.

## <a name="examples-of-prediction-tasks"></a><a name="mltasks"></a>Примеры задач прогнозирования
Мы сформулируем три перечисленные ниже задачи прогнозирования на основе *tip\_amount*.

* Двоичная классификация: прогнозирование того, была ли взята подсказка для поездки, то *есть \_ Сумма TIP* , превышающая $0, является положительным примером, а *\_ количество советов* $0 — отрицательный пример.
* Мультиклассовая классификация: Спрогнозировать диапазон суммы чаевых за поездку. Мы разделяем *tip\_amount* на пять ячеек или классов:

   `Class 0 : tip_amount = $0`

   `Class 1 : tip_amount > $0 and tip_amount <= $5`

   `Class 2 : tip_amount > $5 and tip_amount <= $10`

   `Class 3 : tip_amount > $10 and tip_amount <= $20`

   `Class 4 : tip_amount > $20`

* Задача регрессии: Спрогнозировать сумму чаевых, выплаченных за поездку.  

## <a name="setting-up-the-azure-data-science-environment-for-advanced-analytics"></a><a name="setup"></a>Настройка среды обработки данных Azure для расширенной аналитики
Как можно видеть по руководству [Планирование вашей среды](plan-your-environment.md) , существует несколько вариантов работы с набором данных "Поездки такси Нью-Йорка" в Azure.

* Работа с данными в BLOB-объектах Azure и последующее моделирование в машинном обучении Azure.
* Загрузка данных в базу данных SQL Server и последующее моделирование в машинном обучении Azure.

В этом учебнике мы продемонстрируем параллельный импорт данных в SQL Server, исследование данных, проектирование признаков и понижение выборки с помощью SQL Server Management Studio, а также с помощью записной книжки IPython Notebook. [Примеры скриптов](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts) и файлы [IPython Notebook](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/iPythonNotebooks) предоставлены в общий доступ на GitHub. На этом же ресурсе доступен и образец файла IPython Notebook для работы с данными в BLOB-объектах.

Чтобы настроить среду научной обработки данных в Azure, выполните следующие действия.

1. [создать учетную запись хранения;](../../storage/common/storage-account-create.md)
2. [Создание рабочей области машинного обучения Azure](../classic/create-workspace.md)
3. [Подготовьте виртуальную машину для обработки и анализа данных](../data-science-virtual-machine/overview.md), которая будет служить сервером SQL Server и сервером IPython Notebook.
   
   > [!NOTE]
   > Примеры сценариев и файлов IPython Notebook будут загружены в вашу виртуальную машину для обработки данных в процессе установки. После завершения сценария, выполняемого после установки ВМ, примеры будут располагаться в библиотеке документов вашей виртуальной машины.  
   > 
   > * Примеры сценариев: `C:\Users\<user_name>\Documents\Data Science Scripts`  
   > * Примеры IPython Notebook: `C:\Users\<user_name>\Documents\IPython Notebooks\DataScienceSamples`  
   >   where `<user_name>` является именем для входа Windows виртуальной машины. Эти папки с примерами в дальнейшем будут называться **Примеры скриптов** и **Примеры файлов IPython Notebook**.
   > 
   > 

По размеру набора данных, расположению источника данных и выбранной целевой среде Azure этот пример похож на [Сценарий \#№ 5. Большой набор данных в локальных файлах, загружаемый на сервер SQL Server в виртуальной машине Azure](plan-sample-scenarios.md#largelocaltodb).

## <a name="get-the-data-from-public-source"></a><a name="getdata"></a>Получение данных из общедоступного источника
Чтобы получить [набор данных о поездках такси Нью-Йорка](https://www.andresmh.com/nyctaxitrips/) из общедоступного расположения, скопируйте данные на новую виртуальную машину. Для этого можно воспользоваться любым из методов, описанных в статье [Перемещение данных в хранилище больших двоичных объектов Azure и из него](move-azure-blob.md).

Чтобы скопировать данные с помощью AzCopy, выполните следующие действия.

1. Войдите на виртуальную машину (ВМ).
2. Создайте новый каталог на диске данных виртуальной машины (Примечание. не используйте временный диск, входящий в состав виртуальной машины в качестве диска данных).
3. В окне командной строки выполните следующую команду Azcopy, заменив <path_to_data_folder> соответствующей папкой данных, созданной на шаге (2):

    ```console
    "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:https://nyctaxitrips.blob.core.windows.net/data /Dest:<path_to_data_folder> /S
    ```

    Когда программа AzCopy завершит работу, в папке данных появится 24 сжатых CSV-файла (по 12 для trip\_data и trip\_fare).
4. Распакуйте загруженные файлы. Обратите внимание на папку, в которой располагаются распакованные файлы. Эта папка будет называться <path\_to\_data\_files\>.

## <a name="bulk-import-data-into-sql-server-database"></a><a name="dbload"></a>Выполните массовый импорт данных в базу данных SQL Server
Производительность загрузки и передачи больших объемов данных в базу данных SQL и последующие запросы можно улучшить с помощью *секционированных таблиц и представлений*. В этом разделе мы будем следовать инструкциям, описанным в разделе [Параллельный массовый импорт данных с помощью таблиц разделов SQL](parallel-load-sql-partitioned-tables.md) для создания новой базы данных и параллельной загрузки данных в секционированные таблицы.

1. Войдите на виртуальную машину и запустите **SQL Server Management Studio**.
2. Подключитесь с использованием проверки подлинности Windows.
   
    ![Подключение SSMS][12]
3. Если вы еще не изменили режим проверки подлинности SQL и не создали пользователя для входа в SQL, откройте файл сценария с именем **change\_auth.sql** в папке **Примеры скриптов**. Измените имя пользователя и пароль, используемые по умолчанию. Нажмите кнопку **выполнить** на панели инструментов, чтобы запустить скрипт.
   
    ![Выполнение сценария][13]
4. Проверьте и/или измените используемые по умолчанию папки базы данных и журнала SQL Server, чтобы обеспечить хранение новых создаваемых баз данных на диске с данными. SQL Serverный образ виртуальной машины, оптимизированный для загрузки хранилищ данных, предварительно настроен с использованием дисков данных и журналов. Если ваша виртуальная машина не содержит диска с данными и вы добавили новые виртуальные жесткие диски в процессе установки ВМ, измените папки по умолчанию следующим образом.
   
   * Щелкните правой кнопкой мыши имя сервера SQL Server на левой панели и выберите пункт **Свойства**.
     
       ![Свойства сервера SQL][14]
   * Щелкните **Параметры базы** данных в списке **Выбор страницы** слева.
   * Проверьте и (или) измените **расположения базы данных** по умолчанию на выбранные вами расположения **диска с данными**. В этом расположении новые базы данных размещаются, если они созданы с параметрами по умолчанию.
     
       ![База данных SQL по умолчанию][15]  
5. Чтобы создать базу данных и набор файловых групп для размещения секционированных таблиц, откройте образец скрипта **create\_db\_default.sql**. Сценарий создаст новую базу данных с именем **TaxiNYC** и 12 файловых групп в расположении данных по умолчанию. Каждая файловая группа будет содержать данные trip\_data и trip\_fare за один месяц. При желании можно изменить имя базы данных. Нажмите кнопку **выполнить** , чтобы запустить скрипт.
6. Затем создайте две таблицы секционирования: по одной для trip\_data и trip\_fare. Откройте образец скрипта **create\_partitioned\_table.sql**, который выполнит следующие действия.
   
   * Создаст функцию секционирования для разделения данных по месяцам.
   * Создаст схему секционирования для сопоставления данных за каждый месяц с отдельной файловой группой.
   * Создайте две таблицы секционирования для сопоставления со схемой секционирования: **nyctaxi\_trip** будет содержать данные trip\_data, а **nyctaxi\_fare** — данные trip\_fare.
     
     Нажмите кнопку **выполнить** , чтобы запустить скрипт и создать секционированные таблицы.
7. Папка **Примеры сценариев** содержит два образца сценариев PowerShell, предоставленные для демонстрации параллельного массового импорта данных в таблицы SQL Server.
   
   * **bcp\_parallel\_generic.ps1** — универсальный скрипт для параллельного массового импорта данных в таблицу. Измените этот сценарий для установки входных и целевых переменных, как указано в строках комментариев в сценарии.
   * **bcp\_parallel\_nyctaxi.ps1** — предварительно настроенная версия универсального скрипта, которая может использоваться для загрузки обеих таблиц с данными о поездках на такси Нью-Йорка.  
8. Щелкните правой кнопкой мыши имя сценария **bcp\_parallel\_nyctaxi.ps1** и выберите **Изменить**, чтобы открыть его в PowerShell. Просмотрите предустановленные переменные и измените их в соответствии с выбранным именем базы данных, папкой входных данных, целевой папкой журнала и путями к образцам файлов форматирования **nyctaxi_trip.xml** и **nyctaxi\_fare.xml** (расположены в папке **Примеры скриптов**).
   
    ![Массовый импорт данных][16]
   
    Также можно выбрать режим проверки подлинности (по умолчанию — проверка подлинности Windows). Для запуска щелкните зеленую стрелку на панели инструментов. Сценарий параллельно запустит 24 операции массового импорта — по 12 на каждую секционированную таблицу. Чтобы отслеживать ход импорта данных, откройте папку данных SQL Server по умолчанию, настроенную выше.
9. Сценарий PowerShell сообщает о времени начала и завершения операций. После завершения всех операций массового импорта сообщается время завершения. Проверьте папку "Целевая папка журнала", чтобы убедиться, что операции массовых импортов выполнены успешно, т. е. в папке журнала назначения ошибки не обнаружены.
10. Теперь база данных готова к просмотру, проектированию характеристик и другим операциям. Так как таблицы секционированы в соответствии с полем **\_ даты отправки** , запросы, включающие условия **отправки \_ даты и времени** в предложении **WHERE** , получат преимущество от схемы секционирования.
11. В среде **SQL Server Management Studio** изучите предоставленный образец скрипта **sample\_queries.sql**. Чтобы запустить любой из образцов запросов, выделите строки запроса и нажмите кнопку **выполнить** на панели инструментов.
12. Данные "Поездки такси Нью-Йорка" будут загружены в две отдельные таблицы. Для улучшения операций объединения настоятельно рекомендуется проиндексировать таблицы. Образец скрипта **create\_partitioned\_index.sql** создает секционированные индексы по ключу составного соединения **medallion, hack\_license и pickup\_datetime**.

## <a name="data-exploration-and-feature-engineering-in-sql-server"></a><a name="dbexplore"></a>Просмотр данных и проектирование характеристик в SQL Server
В этом разделе мы выполним просмотр данных и создадим характеристики путем запуска SQL-запросов непосредственно в среде **SQL Server Management Studio** с использованием ранее созданной базы данных SQL Server. Образец скрипта с именем **sample\_queries.sql** расположен в папке **Примеры скриптов**. Измените в сценарии имя базы данных, если оно отличается от установленного по умолчанию **TaxiNYC**.

В этом упражнении мы выполним такие действия.

* Подключимся к среде **SQL Server Management Studio** с помощью проверки подлинности Windows или проверки подлинности SQL, имени пользователя SQL и пароля.
* Просмотрим распределение данных по нескольким полям в различных временных окнах.
* Исследуем качество данных по полям долготы и широты.
* Создадим метки двоичной и мультиклассовой классификации на основе **tip\_amount**.
* Создадим характеристики и вычислим/сравним расстояния поездок.
* Соединим две таблицы и извлечем случайную выборку, которая послужит основой для построения моделей.

Когда вы будете готовы перейти к машинному обучению Azure, вы можете:  

1. Сохраните окончательный SQL-запрос для извлечения и выборки данных и скопируйте и вставьте запрос непосредственно в модуль [Импорт данных][import-data] в машинное обучение Azure или
2. Или сохранить выбранные и спроектированные данные, которые планируется использовать для построения модели в новой таблице базы данных, и использовать новую таблицу в модуле [Импорт данных][import-data] в Машинном обучении Azure.

В этом разделе мы сохраняем окончательный запрос для извлечения и выборки данных. Второй метод демонстрируется в разделе [Просмотр данных и проектирование характеристик в IPython Notebook](#ipnb) .

Быстрая проверка числа строк и столбцов в таблицах, заполненных ранее с помощью параллельного массового импорта:

- Отчет число строк в таблице nyctaxi_trip без просмотра таблицы: `SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('nyctaxi_trip')`
- Отчет число столбцов в nyctaxi_trip таблицы: `SELECT COUNT(*) FROM information_schema.columns WHERE table_name = 'nyctaxi_trip'`

#### <a name="exploration-trip-distribution-by-medallion"></a>Просмотр: распределение поездок по параметру medallion
В этом примере определяются медальоны (номера такси) с более чем 100 поездками за заданный период времени. Запрос будет использовать секционированный доступ к таблице, так как он обусловлен схемой секционирования **pickup\_datetime**. При запросе к полному набору данных также будет задействовано сканирование секционированной таблицы и/или индекса.

```sql
SELECT medallion, COUNT(*)
FROM nyctaxi_fare
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
GROUP BY medallion
HAVING COUNT(*) > 100
```

#### <a name="exploration-trip-distribution-by-medallion-and-hack_license"></a>Просмотр: распределение поездок по параметрам medallion и hack_license

```sql
SELECT medallion, hack_license, COUNT(*)
FROM nyctaxi_fare
WHERE pickup_datetime BETWEEN '20130101' AND '20130131'
GROUP BY medallion, hack_license
HAVING COUNT(*) > 100
```

#### <a name="data-quality-assessment-verify-records-with-incorrect-longitude-andor-latitude"></a>Оценка качества данных: Проверка записей с неверными значениями долготы и/или широты
В этом примере анализируется, содержится ли в каких-либо полях долготы и/или широты неверное значение (количество градусов должно быть в пределах от -90 до 90) или значение координат (0, 0).

```sql
SELECT COUNT(*) FROM nyctaxi_trip
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(pickup_latitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_latitude AS float) NOT BETWEEN -90 AND 90
OR    (pickup_longitude = '0' AND pickup_latitude = '0')
OR    (dropoff_longitude = '0' AND dropoff_latitude = '0'))
```

#### <a name="exploration-tipped-vs-not-tipped-trips-distribution"></a>Просмотр: Выплаченные чаевые против Распределение поездок с чаевыми и без чаевых
В этом примере определяется число поездок, в которых были выплачены чаевые, против тех, в которых чаевые не были выплачены, за заданный период времени (или по полному набору данных в случае охвата целого года). Это распределение отражает распределение двоичных меток, которые в дальнейшем будут использоваться для моделирования двоичной классификации.

```sql
SELECT tipped, COUNT(*) AS tip_freq FROM (
  SELECT CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped, tip_amount
  FROM nyctaxi_fare
  WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tipped
```

#### <a name="exploration-tip-classrange-distribution"></a>Просмотр: распределение классов/диапазонов чаевых
В этом примере вычисляется распределение диапазонов чаевых за заданный период времени (или по полному набору данных в случае охвата целого года). Это распределение классов меток будет использоваться позже для моделирования многоклассовой классификации.

```sql
SELECT tip_class, COUNT(*) AS tip_freq FROM (
    SELECT CASE
        WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM nyctaxi_fare
WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tip_class
```

#### <a name="exploration-compute-and-compare-trip-distance"></a>Просмотр: Вычисление и сравнение расстояния поездок
В этом примере значения долготы и широты начальных и конечных пунктов поездок преобразуются в географические точки SQL, вычисляется расстояние поездки по разности географических точек и возвращается случайная выборка результатов для сравнения. В примере результаты ограничиваются только допустимыми координатами с помощью запроса оценки качества данных, описанного ранее.

```sql
SELECT
pickup_location=geography::STPointFromText('POINT(' + pickup_longitude + ' ' + pickup_latitude + ')', 4326)
,dropoff_location=geography::STPointFromText('POINT(' + dropoff_longitude + ' ' + dropoff_latitude + ')', 4326)
,trip_distance
,computedist=round(geography::STPointFromText('POINT(' + pickup_longitude + ' ' + pickup_latitude + ')', 4326).STDistance(geography::STPointFromText('POINT(' + dropoff_longitude + ' ' + dropoff_latitude + ')', 4326))/1000, 2)
FROM nyctaxi_trip
tablesample(0.01 percent)
WHERE CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND   CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND   pickup_longitude != '0' AND dropoff_longitude != '0'
```

#### <a name="feature-engineering-in-sql-queries"></a>Проектирование характеристик в запросах SQL
Запросы на создание меток и просмотр преобразования географии можно также использовать для создания меток/характеристик путем удаления части, производящей подсчет. Дополнительные примеры проектирования характеристик на SQL содержатся в разделе [Просмотр данных и проектирование характеристик в IPython Notebook](#ipnb) . Более эффективным является выполнение запросов на создание компонентов к полному набору данных или крупному подмножеству с помощью запросов SQL, которые выполняются непосредственно в экземпляре базы данных SQL Server. Запросы могут выполняться в **SQL Server Management Studio**, записной книжке IPython Notebook или в любом средстве разработки или среде, которая может осуществлять доступ к базе данных локально или удаленно.

#### <a name="preparing-data-for-model-building"></a>Подготовка данных для построения модели
Следующий запрос соединяет таблицы **nyctaxi\_trip** и **nyctaxi\_fare**, создает метку двоичной классификации **tipped**, метку многоклассовой классификации **tip\_class**, а также извлекает малую выборку из полного соединенного набора данных. Этот запрос можно скопировать, а затем вставить непосредственно в модуль [Импорт данных](https://studio.azureml.net) [Студии машинного обучения Azure][import-data] для прямого приема данных из экземпляра базы данных SQL Server в Azure. Запрос исключает записи с неверными (0, 0) координатами.

```sql
SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,     f.total_amount, f.tip_amount,
    CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped,
    CASE WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM nyctaxi_trip t, nyctaxi_fare f
TABLESAMPLE (1 percent)
WHERE t.medallion = f.medallion
AND   t.hack_license = f.hack_license
AND   t.pickup_datetime = f.pickup_datetime
AND   pickup_longitude != '0' AND dropoff_longitude != '0'
```

## <a name="data-exploration-and-feature-engineering-in-ipython-notebook"></a><a name="ipnb"></a>Исследование данных и проектирование характеристик в записной книжке IPython Notebook
В этом разделе мы выполним просмотр данных и проектирование характеристик используя как Python, так и SQL-запросы, в отличии от созданной ранее базы данных SQL Server. Образец файла IPython Notebook с именем **machine-Learning-data-science-process-sql-story.ipynb** расположен в папке **Примеры файлов IPython Notebook**. Этот файл также доступен на [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/iPythonNotebooks).

При работе с большими данными используйте следующую рекомендуемую последовательность:

* Считайте небольшую выборку данных во фрейм данных, размещенный в памяти.
* Выполните несколько визуализаций и просмотров с использованием выборки данных.
* Поэкспериментируйте с проектированием характеристик с использованием выборки данных.
* Для просмотра большего объема данных, манипуляций с данными и проектирования характеристик используйте Python для подачи SQL-запросов непосредственно к базе данных SQL Server на виртуальной машине Azure.
* Определите размер выборки, который будет использоваться для построения модели в службе машинного обучения Azure.

Когда вы будете готовы перейти к машинному обучению Azure, можно выполнить одно из следующих действий.  

1. Сохраните окончательный SQL-запрос для извлечения и выборки данных и скопируйте и вставьте запрос непосредственно в модуль [Импорт данных][import-data] в машинное обучение Azure. Этот метод демонстрируется в разделе [Построение моделей в машинном обучении Azure](#mlmodel) .    
2. Сохранить выбранные и спроектированные данные, которые планируется использовать для построения модели, в новой таблице базы данных, затем использовать эту новую таблицу в модуле [Импорт данных][import-data].

Ниже приведены несколько примеров просмотра данных, визуализации данных и проектирования характеристик. Дополнительные примеры см. в образце файла IPython Notebook с SQL-запросом в папке **Примеры файлов IPython Notebook**.

#### <a name="initialize-database-credentials"></a>Инициализация учетных данных базы данных
Инициализируйте параметры подключения к базе данных в следующих переменных:

```sql
SERVER_NAME=<server name>
DATABASE_NAME=<database name>
USERID=<user name>
PASSWORD=<password>
DB_DRIVER = <database server>
```

#### <a name="create-database-connection"></a>Создание подключения к базе данных

```sql
CONNECTION_STRING = 'DRIVER={'+DRIVER+'};SERVER='+SERVER_NAME+';DATABASE='+DATABASE_NAME+';UID='+USERID+';PWD='+PASSWORD
conn = pyodbc.connect(CONNECTION_STRING)
```

#### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_trip"></a>Сообщение числа строк и столбцов в таблице nyctaxi_trip

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('nyctaxi_trip')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('nyctaxi_trip')
''', conn)

print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Общее число строк = 173179759  
* Общее число столбцов = 14

#### <a name="read-in-a-small-data-sample-from-the-sql-server-database"></a>Считывание небольшой выборки данных из базы данных SQL Server

```sql
t0 = time.time()

query = '''
    SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax,
        f.tolls_amount, f.total_amount, f.tip_amount
    FROM nyctaxi_trip t, nyctaxi_fare f
    TABLESAMPLE (0.05 PERCENT)
    WHERE t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
'''

df1 = pd.read_sql(query, conn)

t1 = time.time()
print 'Time to read the sample table is %f seconds' % (t1-t0)

print 'Number of rows and columns retrieved = (%d, %d)' % (df1.shape[0], df1.shape[1])
```

Время чтения таблицы выборки равно 6,492000 секунды  
Число извлеченных строк и столбцов = (84952, 21)

#### <a name="descriptive-statistics"></a>Описательная статистика
Теперь все готово для просмотра выборки данных. Начнем с изучения описательной статистики для **trip\_distance** (или любых других полей).

```sql
df1['trip_distance'].describe()
```

#### <a name="visualization-box-plot-example"></a>Визуализация: Пример блочной диаграммы
Далее рассмотрим блочную диаграмму расстояний поездок для визуализации квантилей

```sql
df1.boxplot(column='trip_distance',return_type='dict')
```

![График #1][1]

#### <a name="visualization-distribution-plot-example"></a>Визуализация: Пример графика распределения

```sql
fig = plt.figure()
ax1 = fig.add_subplot(1,2,1)
ax2 = fig.add_subplot(1,2,2)
df1['trip_distance'].plot(ax=ax1,kind='kde', style='b-')
df1['trip_distance'].hist(ax=ax2, bins=100, color='k')
```

![График #2][2]

#### <a name="visualization-bar-and-line-plots"></a>Визуализация: Гистограммы и линейные графики
В этом примере мы сегментируем расстояния поездок на пять ячеек и визуализируем результаты сегментирования.

```sql
trip_dist_bins = [0, 1, 2, 4, 10, 1000]
df1['trip_distance']
trip_dist_bin_id = pd.cut(df1['trip_distance'], trip_dist_bins)
trip_dist_bin_id
```

Мы можем изобразить описанное выше распределение по ячейкам в виде гистограммы или линейного графика, как показано ниже

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='bar')
```

![График #3][3]

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='line')
```
![График #4][4]

#### <a name="visualization-scatterplot-example"></a>Визуализация: Пример точечной диаграммы
Мы отобразим точечную диаграмму для параметров **trip\_time\_in\_secs** и **trip\_distance**, чтобы проверить возможную корреляцию.

```sql
plt.scatter(df1['trip_time_in_secs'], df1['trip_distance'])
```

![График #6][6]

Точно также мы можем проверить связь между **rate\_code** и **trip\_distance**.

```sql
plt.scatter(df1['passenger_count'], df1['trip_distance'])
```

![График #8][8]

### <a name="sub-sampling-the-data-in-sql"></a>Вложенная выборка данных в SQL
Подготавливая данные к созданию модели в [Студии машинного обучения Azure](https://studio.azureml.net), вы можете определить **SQL-запрос для использования непосредственно в модуле импорта данных** или сохранить спроектированные и выбранные данные в новой таблице, которую можно использовать в модуле [Импорт данных][import-data] с помощью простого запроса **SELECT * FROM <имя\_вашей\_новой\_таблицы>**.

В этом разделе мы создадим новую таблицу для хранения выборки и инженерных данных. Пример прямого SQL-запроса для построения модели содержится в разделе [Просмотр данных и проектирование характеристик в SQL Server](#dbexplore) .

#### <a name="create-a-sample-table-and-populate-with-1-of-the-joined-tables-drop-table-first-if-it-exists"></a>Создайте таблицу выборки и заполните ее 1 % данных из соединенных таблиц. Сначала удалите таблицу, если она существует.
В этом разделе мы соединим таблицы **nyctaxi\_trip** и **nyctaxi\_fare**, извлечем однопроцентную случайную выборку и сохраним выбранные данные в новой таблице с именем **nyctaxi\_one\_percent**:

```sql
cursor = conn.cursor()

drop_table_if_exists = '''
    IF OBJECT_ID('nyctaxi_one_percent', 'U') IS NOT NULL DROP TABLE nyctaxi_one_percent
'''

nyctaxi_one_percent_insert = '''
    SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount, f.total_amount, f.tip_amount
    INTO nyctaxi_one_percent
    FROM nyctaxi_trip t, nyctaxi_fare f
    TABLESAMPLE (1 PERCENT)
    WHERE t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
    AND   pickup_longitude <> '0' AND dropoff_longitude <> '0'
'''

cursor.execute(drop_table_if_exists)
cursor.execute(nyctaxi_one_percent_insert)
cursor.commit()
```

### <a name="data-exploration-using-sql-queries-in-ipython-notebook"></a>Просмотр данных с помощью SQL-запросов в IPython Notebook
В этом разделе мы рассмотрим распределение данных с использованием данных выборки в 1%, которые хранятся в новой таблице, созданной ранее. Аналогичные исследования могут быть выполнены с помощью исходных таблиц, при необходимости с помощью **предложения TABLESAMPLE** для ограничения образца просмотра или путем ограничения результатов на заданный период времени с помощью секций **\_ даты** отсчета, как показано в статье [исследование данных и проектирование](#dbexplore) признаков в SQL Server разделе.

#### <a name="exploration-daily-distribution-of-trips"></a>Исследование: ежедневное распределение поездок

```sql
query = '''
    SELECT CONVERT(date, dropoff_datetime) AS date, COUNT(*) AS c
    FROM nyctaxi_one_percent
    GROUP BY CONVERT(date, dropoff_datetime)
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-per-medallion"></a>Исследование: распределение поездок по параметру medallion

```sql
query = '''
    SELECT medallion,count(*) AS c
    FROM nyctaxi_one_percent
    GROUP BY medallion
'''

pd.read_sql(query,conn)
```

### <a name="feature-generation-using-sql-queries-in-ipython-notebook"></a>Создание характеристик с помощью SQL-запросов в IPython Notebook
В этом разделе мы создадим новые метки и характеристики непосредственно с помощью SQL-запросов, работая над таблицей 1%-ной выборки, которую мы создали в предыдущем разделе.

#### <a name="label-generation-generate-class-labels"></a>Создание метки: Создать метки классов
В следующем примере мы создадим два набора меток, используемых для моделирования.

1. Метки двоичной классификации **tipped** (прогнозирование, будут ли выплачены чаевые).
2. Мультиклассовые метки **tip\_class** (прогнозирование сегмента или диапазона чаевых).

```sql   
    nyctaxi_one_percent_add_col = '''
        ALTER TABLE nyctaxi_one_percent ADD tipped bit, tip_class int
    '''
   
    cursor.execute(nyctaxi_one_percent_add_col)
    cursor.commit()
   
    nyctaxi_one_percent_update_col = '''
        UPDATE nyctaxi_one_percent
        SET
           tipped = CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END,
           tip_class = CASE WHEN (tip_amount = 0) THEN 0
                            WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
                            WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
                            WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
                            ELSE 4
                        END
    '''
   
    cursor.execute(nyctaxi_one_percent_update_col)
    cursor.commit()
```

#### <a name="feature-engineering-count-features-for-categorical-columns"></a>Проектирование характеристик: Количественные характеристики для столбцов категорий
Этот пример преобразует поле категории в числовое поле, заменяя каждую категорию количеством ее появлений в данных.

```sql
nyctaxi_one_percent_insert_col = '''
    ALTER TABLE nyctaxi_one_percent ADD cmt_count int, vts_count int
'''

cursor.execute(nyctaxi_one_percent_insert_col)
cursor.commit()

nyctaxi_one_percent_update_col = '''
    WITH B AS
    (
        SELECT medallion, hack_license,
            SUM(CASE WHEN vendor_id = 'cmt' THEN 1 ELSE 0 END) AS cmt_count,
            SUM(CASE WHEN vendor_id = 'vts' THEN 1 ELSE 0 END) AS vts_count
        FROM nyctaxi_one_percent
        GROUP BY medallion, hack_license
    )

    UPDATE nyctaxi_one_percent
    SET nyctaxi_one_percent.cmt_count = B.cmt_count,
        nyctaxi_one_percent.vts_count = B.vts_count
    FROM nyctaxi_one_percent A INNER JOIN B
    ON A.medallion = B.medallion AND A.hack_license = B.hack_license
'''

cursor.execute(nyctaxi_one_percent_update_col)
cursor.commit()
```

#### <a name="feature-engineering-bin-features-for-numerical-columns"></a>Проектирование характеристик: Ячейка характеристик для числовых столбцов
Этот пример преобразует непрерывное числовое поле в Предустановленные диапазоны категорий, т. е. Преобразование числового поля в поле категории.

```sql
nyctaxi_one_percent_insert_col = '''
    ALTER TABLE nyctaxi_one_percent ADD trip_time_bin int
'''

cursor.execute(nyctaxi_one_percent_insert_col)
cursor.commit()

nyctaxi_one_percent_update_col = '''
    WITH B(medallion,hack_license,pickup_datetime,trip_time_in_secs, BinNumber ) AS
    (
        SELECT medallion,hack_license,pickup_datetime,trip_time_in_secs,
        NTILE(5) OVER (ORDER BY trip_time_in_secs) AS BinNumber from nyctaxi_one_percent
    )

    UPDATE nyctaxi_one_percent
    SET trip_time_bin = B.BinNumber
    FROM nyctaxi_one_percent A INNER JOIN B
    ON A.medallion = B.medallion
    AND A.hack_license = B.hack_license
    AND A.pickup_datetime = B.pickup_datetime
'''

cursor.execute(nyctaxi_one_percent_update_col)
cursor.commit()
```

#### <a name="feature-engineering-extract-location-features-from-decimal-latitudelongitude"></a>Проектирование характеристик: Извлечение характеристик местоположения из десятичных значений широты/долготы
В этом примере разбивается десятичное представление поля широты и/или долготы на несколько полей области с различной степенью детализации, например страна/регион, город, город, блок и т. д. Новые географические поля не сопоставлены с фактическими расположениями. Дополнительные сведения о сопоставлении местоположений геокодов см. в статье о [службах REST карт Bing](/bingmaps/rest-services/locations/find-a-location-by-point).

```sql
nyctaxi_one_percent_insert_col = '''
    ALTER TABLE nyctaxi_one_percent
    ADD l1 varchar(6), l2 varchar(3), l3 varchar(3), l4 varchar(3),
        l5 varchar(3), l6 varchar(3), l7 varchar(3)
'''

cursor.execute(nyctaxi_one_percent_insert_col)
cursor.commit()

nyctaxi_one_percent_update_col = '''
    UPDATE nyctaxi_one_percent
    SET l1=round(pickup_longitude,0)
        , l2 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 1 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),1,1) ELSE '0' END     
        , l3 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 2 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),2,1) ELSE '0' END     
        , l4 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 3 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),3,1) ELSE '0' END     
        , l5 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 4 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),4,1) ELSE '0' END     
        , l6 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 5 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),5,1) ELSE '0' END     
        , l7 = CASE WHEN LEN (PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1)) >= 6 THEN SUBSTRING(PARSENAME(ROUND(ABS(pickup_longitude) - FLOOR(ABS(pickup_longitude)),6),1),6,1) ELSE '0' END
'''

cursor.execute(nyctaxi_one_percent_update_col)
cursor.commit()
```

#### <a name="verify-the-final-form-of-the-featurized-table"></a>Проверка конечной формы таблицы с признаками

```sql
query = '''SELECT TOP 100 * FROM nyctaxi_one_percent'''
pd.read_sql(query,conn)
```

Теперь мы готовы перейти к созданию модели и развертыванию модели в [машинное обучение Azure](https://studio.azureml.net). Данные готовы для любой из задач прогнозирования, определенных ранее.

1. Двоичная классификация: Спрогнозировать, были ли выплачены чаевые за поездку.
2. Мультиклассовая классификация. Спрогнозировать диапазон выплаченных чаевых в соответствии с ранее определенными классами.
3. Задача регрессии: Спрогнозировать сумму чаевых, выплаченных за поездку.  

## <a name="building-models-in-azure-machine-learning"></a><a name="mlmodel"></a>Построение моделей в машинном обучении Azure
Чтобы начать упражнение по моделированию, войдите в рабочую область машинного обучения Azure. Если вы еще не создали рабочую область машинного обучения, см. статью [Создание рабочей области машинного обучения Azure и предоставление к ней общего доступа](../classic/create-workspace.md).

1. Чтобы приступить к работе с машинным обучением Azure, см. статью [Что такое студия машинного обучения Azure?](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)
2. Войдите в [Студию машинного обучения Azure](https://studio.azureml.net).
3. На главной странице Студии содержится множество информации, видеороликов, учебников, ссылок на справочник модулей и другие ресурсы. Дополнительные сведения о Машинном обучении Azure см. в [центре документации по машинному обучению Azure](https://azure.microsoft.com/documentation/services/machine-learning/).

Типичный обучающий эксперимент предусматривает следующие шаги:

1. Создайте **Новый** эксперимент.
2. Получить данные для Машинного обучения Azure.
3. Предварительная обработка, преобразование и обработка данных по мере необходимости.
4. Создать необходимые характеристики.
5. Разбить данные на учебные/проверочные/тестовые наборы данных (или иметь отдельные наборы данных для каждой из этих целей).
6. Выбрать один или несколько алгоритмов машинного обучения в зависимости от решаемой задачи обучения. Например, двоичная классификация, многоклассовая классификация, регрессия.
7. Обучить одну или несколько моделей с помощью учебного набора данных.
8. Оценить проверочный набор данных с помощью обученных моделей.
9. Вычислить модель (модели) для расчета соответствующих показателей для задачи по обучению.
10. Настройте модели и выберите наилучшую модель для развертывания.

В этом упражнении мы уже просмотрели и спроектировали данные на сервере SQL Server и определились с размером выборки для приема в Машинное обучение Azure. Для создания одной или нескольких моделей прогнозирования мы решили:

1. Загрузите данные в Машинное обучение Azure, используя модуль [Импорт данных][import-data], доступный в разделе **Data Input and Output** (Ввод и вывод данных). Дополнительные сведения см. на странице справочника по модулю [импорта данных][import-data] .
   
    ![Импорт данных в Машинное обучение Azure][17]
2. На панели **Properties** (Свойства) в списке **Data source** (Источник данных) выберите **Azure SQL Database** (База данных SQL Azure).
3. Введите DNS-имя базы данных в поле **Имя сервера базы данных** . Формат: `tcp:<your_virtual_machine_DNS_name>,1433`
4. Введите **Имя базы данных** в соответствующее поле.
5. Введите **имя пользователя SQL** в поле **Server user account name** (Имя учетной записи пользователя сервера) и **пароль** в поле **Server user account password** (Пароль учетной записи пользователя сервера).
7. В области текст **запроса к базе данных** вставьте запрос, извлекающий необходимые поля базы данных (включая все вычисленные поля, такие как метки), а затем выберите для данных нужный размер выборки.

Пример эксперимента по двоичной классификации со считыванием данных непосредственно из базы данных SQL Server показан на рисунке ниже. Подобные эксперименты можно сконструировать для задач мультиклассовой классификации и регрессии.

![Обучение моделей Машинного обучения Azure][10]

> [!IMPORTANT]
> В примерах запросов на извлечение и выборку данных для моделирования, указанных в предыдущих разделах, **все метки для трех упражнений по моделированию включены в запрос**. Важным (обязательным) шагом каждого из упражнений по моделированию является **исключение** ненужных меток для двух остальных задач, а также любых **целевых утечек**. Например, при двоичной классификации используйте метку **tipped** и исключите поля **tip\_class**, **tip\_amount** и **total\_amount**. Последние являются целевыми утечками, поскольку подразумевают, что чаевые выплачены.
> 
> Чтобы исключить ненужные столбцы и/или целевые утечки, можно воспользоваться модулем [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных) или [Edit Metadata][edit-metadata] (Изменение метаданных). Дополнительные сведения см. на страницах справки модулей [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных) и [Edit Metadata][edit-metadata] (Изменение метаданных).
> 
> 

## <a name="deploying-models-in-azure-machine-learning"></a><a name="mldeploy"></a>Развертывание моделей в машинном обучении Azure
Когда модель готова, ее можно легко развернуть в виде веб-службы непосредственно из эксперимента. Дополнительные сведения см. в статье [Развертывание веб-службы машинного обучения Azure](../classic/deploy-a-machine-learning-web-service.md).

Чтобы развернуть новую веб-службу, необходимо выполнить такие действия.

1. Создать эксперимент по количественной оценке.
2. Развернуть веб-службу.

Чтобы создать оценивающий эксперимент на основе учебного эксперимента со статусом **Завершено**, щелкните **Create scoring experiment** (Создать оценивающий эксперимент) на нижней панели действий.

![Система оценок Azure][18]

Служба машинного обучения Azure попытается создать эксперимент по количественной оценке на основе компонентов учебного эксперимента. В частности, она:

1. Сохранит обученную модель и удалит модули обучения модели.
2. Идентифицирует логический **порт ввода** для представления ожидаемой схемы входных данных.
3. Идентифицирует логический **порт вывода** для представления ожидаемой схемы вывода веб-службы.

При создании эксперимента по количественной оценке просмотрите его и скорректируйте по необходимости. Типичная корректировка заключается в замене входного набора данных и (или) запроса на один, исключающий поля меток, поскольку эти метки не будут доступны в схеме при вызове службы. Также рекомендуется уменьшить размер входного набора данных и (или) запроса на несколько записей, чтобы указать входную схему. Для порта вывода зачастую исключаются все входные поля и в вывод включаются только **Scored Labels** (Оцененные метки) и **Scored Probabilities** (Оцененные вероятности) с помощью модуля [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных).

Образец эксперимента по количественной оценке показан на рисунке ниже. Когда все будет готово к развертыванию, нажмите кнопку **ОПУБЛИКОВАТЬ ВЕБ-СЛУЖБУ** на нижней панели действий.

![Публикация в Машинном обучении Azure][11]

Итак, в этом пошаговом руководстве мы создали среду обработки данных, работали с большим общедоступным набором данных на протяжении всего процесса от получения данных до обучения модели и развертывания веб-службы машинного обучения Azure.

### <a name="license-information"></a>Сведения о лицензии
Этот образец пошагового руководства и сопровождающие его сценарии и файлы IPython Notebook предоставлены корпорацией Майкрософт на условиях лицензии MIT. Чтобы получить дополнительные сведения, проверьте файл LICENSE.txt в каталоге примера кода на сайте GitHub.

### <a name="references"></a>Ссылки
•    [Страница Андреса Монройя для скачивания данных о поездках в такси Нью-Йорка](https://www.andresmh.com/nyctaxitrips/)  
•    [Данные о поездках в такси получение Нью от Крис whong)](https://chriswhong.com/open-data/foil_nyc_taxi/)   
•    [Исследование и статистика Нью такси и лимузинов](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[1]: ./media/sql-walkthrough/sql-walkthrough_26_1.png
[2]: ./media/sql-walkthrough/sql-walkthrough_28_1.png
[3]: ./media/sql-walkthrough/sql-walkthrough_35_1.png
[4]: ./media/sql-walkthrough/sql-walkthrough_36_1.png
[5]: ./media/sql-walkthrough/sql-walkthrough_39_1.png
[6]: ./media/sql-walkthrough/sql-walkthrough_42_1.png
[7]: ./media/sql-walkthrough/sql-walkthrough_44_1.png
[8]: ./media/sql-walkthrough/sql-walkthrough_46_1.png
[9]: ./media/sql-walkthrough/sql-walkthrough_71_1.png
[10]: ./media/sql-walkthrough/azuremltrain.png
[11]: ./media/sql-walkthrough/azuremlpublish.png
[12]: ./media/sql-walkthrough/ssmsconnect.png
[13]: ./media/sql-walkthrough/executescript.png
[14]: ./media/sql-walkthrough/sqlserverproperties.png
[15]: ./media/sql-walkthrough/sqldefaultdirs.png
[16]: ./media/sql-walkthrough/bulkimport.png
[17]: ./media/sql-walkthrough/amlreader.png
[18]: ./media/sql-walkthrough/amlscoring.png


<!-- Module References -->
[edit-metadata]: /azure/machine-learning/studio-module-reference/edit-metadata
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[import-data]: /azure/machine-learning/studio-module-reference/import-data