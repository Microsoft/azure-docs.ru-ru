---
title: Создание и развертывание модели с помощью Azure синапсе Analytics. процесс обработки и анализа данных группы
description: Создание и развертывание модели машинного обучения с помощью Azure синапсе Analytics и общедоступного набора данных.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, devx-track-python, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: b638cb2b33f24220e7ceb852402862c707cc7bc6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93315995"
---
# <a name="the-team-data-science-process-in-action-using-azure-synapse-analytics"></a>Процесс обработки и анализа данных группы в действии: использование Azure синапсе Analytics
В этом учебнике мы пошаговым руководством по созданию и развертыванию модели машинного обучения с помощью Azure синапсе Analytics для общедоступного набора данных — набора данных о [поездках в такси Нью](https://www.andresmh.com/nyctaxitrips/) . Созданная модель двоичной классификации прогнозирует, оплачивается ли TIP для поездки.  Модели включают в себя многоклассовую классификацию (будь то TIP) и регрессия (распределение для оплаченных сумм советов).

Ниже представлена процедура рабочего процесса [обработки и анализа данных группы (TDSP)](./index.yml) . Мы покажем, как настроить среду обработки и анализа данных, как загрузить данные в Azure синапсе Analytics, а также как использовать Azure синапсе Analytics или записную книжку IPython Notebook, чтобы исследовать функции данных и инженеров для моделирования. Затем мы рассмотрим, как создать и развернуть модель в службе машинного обучения Azure.

## <a name="the-nyc-taxi-trips-dataset"></a><a name="dataset"></a>Набор данных "Поездки такси Нью-Йорка"
Данные о поездках такси Нью-Йорка содержатся в сжатых CSV-файлах размером около 20 ГБ (примерно 48 ГБ в распакованном виде), которые включают в себя сведения о более 173 млн отдельных поездок и платежах за каждую поездку. Каждая запись о командировке включает в себя расположение и время отправки и высадки, номер лицензии для анонимных хакеров и номер Medallion (уникальный идентификатор такси). Данные включают в себя все поездки за 2013 год и предоставляются в виде следующих двух наборов данных за каждый месяц:

1. CSV-файл **trip_data.csv** содержит подробные сведения о поездке, например число пассажиров, пункты отправления и назначения, продолжительность поездки и ее расстояние. Вот несколько примеров записей:

`medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude`

`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868`

2. CSV-файл **trip_fare** содержит подробные сведения об оплате каждой поездки, такие как тип оплаты, сумма тарифа, надбавка и налоги, чаевые и пошлины, а также общая выплаченная сумма. Вот несколько примеров записей:

`medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount`

`89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7`

`0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6`

`DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5`

**Уникальный ключ** для соединения trip\_data и trip\_fare состоит из следующих трех полей:

* medallion,
* hack\_license и
* pickup\_datetime.

## <a name="address-three-types-of-prediction-tasks"></a><a name="mltasks"></a>Определение трех типов задач прогнозирования
На основе *tip\_amount* мы сформулировали три проблемы прогнозирования, чтобы проиллюстрировать три типа задач моделирования.

1. **Двоичная классификация**: для прогнозирования того, оплачивается ли TIP для поездки, то есть, то *есть \_ Сумма tip* , превышающая $0, является положительным примером, а *\_ количество советов* $0 — отрицательный пример.
2. **Мультиклассовая классификация**: спрогнозировать диапазон суммы чаевых за поездку. Мы разделяем *tip\_amount* на пять ячеек или классов:

`Class 0 : tip_amount = $0`

`Class 1 : tip_amount > $0 and tip_amount <= $5`

`Class 2 : tip_amount > $5 and tip_amount <= $10`

`Class 3 : tip_amount > $10 and tip_amount <= $20`

`Class 4 : tip_amount > $20`

3. **Регрессионная задача**: прогнозирование количества подсказок, оплаченных за поездку.

## <a name="set-up-the-azure-data-science-environment-for-advanced-analytics"></a><a name="setup"></a>Настройка среды обработки и анализа данных Azure для расширенной аналитики
Чтобы настроить среду обработки и анализа данных в Azure, выполните следующие шаги.

**Создайте собственную учетную запись хранилища BLOB-объектов Azure**

* Во время подготовки своего хранилища BLOB-объектов Azure выберите географическое расположение как можно ближе к **центрально-южной части США**, где будут храниться данные о такси Нью-Йорка. С помощью AzCopy данные будут скопированы из контейнера общедоступного хранилища BLOB-объектов в контейнер в вашей учетной записи хранения. Чем ближе хранилище BLOB-объектов Azure к центрально-южная часть США, тем быстрее будет выполнена эта задача (шаг 4).
* Чтобы создать собственную учетную запись хранения Azure, выполните действия, описанные в статье [об учетных записях хранения Azure](../../storage/common/storage-account-create.md). Убедитесь, что вы записали значения данных учетной записи хранения, так как они потребуются позже в этом пошаговом руководстве.

  * **Имя учетной записи хранения**
  * **Ключ учетной записи хранения**
  * **Имя контейнера** (контейнер в хранилище BLOB-объектов Azure, где будут храниться данные)

**Подготавливает экземпляр Azure синапсе Analytics.**
Следуйте указаниям в документации по [созданию и запросу Azure синапсе Analytics в портал Azure](../../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md) для предоставления экземпляра Azure синапсе Analytics. Убедитесь, что вы захотите создать нотации на следующих учетных данных Azure синапсе Analytics, которые будут использоваться на последующих шагах.

* **Имя сервера**: \<server Name> . Database.Windows.NET
* **имя (базы данных) хранилища данных SQL;**
* **Имя пользователя**
* **Пароль**

**Установите Visual Studio и SQL Server Data Tools.** Инструкции см. в статье [Приступая к работе с Visual Studio 2019 для Azure синапсе Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-install-visual-studio.md).

**Подключитесь к Azure синапсе Analytics с помощью Visual Studio.** Инструкции см. в разделе шаги 1 & 2 раздела [Подключение к аналитике SQL в Azure синапсе Analytics](../../synapse-analytics/sql/connect-overview.md).

> [!NOTE]
> Выполните следующий SQL-запрос к базе данных, созданной в Azure синапсе Analytics (вместо запроса, предоставленного на шаге 3 раздела Connect), чтобы **создать главный ключ**.
>
>

```sql
BEGIN TRY
       --Try to create the master key
    CREATE MASTER KEY
END TRY
BEGIN CATCH
       --If the master key exists, do nothing
END CATCH;
```

**Создайте рабочую область Машинного обучения Azure, используя подписку Azure.** Инструкции можно найти в статье [Создание рабочей области машинного обучения Azure](../classic/create-workspace.md).

## <a name="load-the-data-into-azure-synapse-analytics"></a><a name="getdata"></a>Загрузка данных в Azure синапсе Analytics
Откройте командную консоль Windows PowerShell. Выполните следующие команды PowerShell, чтобы скачать примеры файлов скриптов SQL, которые мы предоставляем Вам в GitHub, в локальный каталог, указанный с параметром *-DestDir*. Значение параметра *-DestDir* можно изменить и указать любой другой локальный каталог. Если *-DestDir* отсутствует, он будет создан в рамках сценария PowerShell.

> [!NOTE]
> При выполнении следующего сценария PowerShell может потребоваться **запуск от имени администратора** , если для создания каталога *DestDir* или записи в него нужны права администратора.
>
>

```azurepowershell
$source = "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/Download_Scripts_SQLDW_Walkthrough.ps1"
$ps1_dest = "$pwd\Download_Scripts_SQLDW_Walkthrough.ps1"
$wc = New-Object System.Net.WebClient
$wc.DownloadFile($source, $ps1_dest)
.\Download_Scripts_SQLDW_Walkthrough.ps1 –DestDir 'C:\tempSQLDW'
```

После успешного выполнения текущий рабочий каталог изменится на *-DestDir*. Должен отобразиться экран, аналогичный показанному ниже:

![Изменения в текущем рабочем каталоге][19]

В каталоге *-DestDir* выполните следующий сценарий PowerShell в режиме администратора:

```azurepowershell
./SQLDW_Data_Import.ps1
```

При первом запуске сценария PowerShell вам будет предложено ввести сведения из Azure синапсе Analytics и учетную запись хранилища BLOB-объектов Azure. После первого завершения этого сценария PowerShell введенные учетные данные будут записаны в файл конфигурации SQLDW.conf в используемом рабочем каталоге. При последующем запуске этого файла сценария PowerShell можно будет считать все необходимые параметры из файла конфигурации. Если некоторые параметры нужно изменить, их можно ввести на экране, когда отобразится запрос. Для этого нужно удалить файл конфигурации и ввести значения параметров по запросу. Кроме того, значения параметров можно изменить в файле SQLDW.conf в каталоге *-DestDir*.

> [!NOTE]
> Чтобы избежать конфликтов имен схем с теми, которые уже существуют в Azure Azure синапсе Analytics, при чтении параметров непосредственно из файла SQLDW. conf в имя схемы из файла SQLDW. conf добавляется случайное число из 3 цифр, которое используется в качестве имени схемы по умолчанию для каждого запуска. Сценарий PowerShell может запросить имя схемы. Имя указывается по усмотрению пользователя.
>
>

Запуск этого файла **сценария PowerShell** предусматривает выполнение следующих задач:

* **Скачивание и установка AzCopy**, если она еще не установлена.

  ```azurepowershell
  $AzCopy_path = SearchAzCopy
  if ($AzCopy_path -eq $null){
         Write-Host "AzCopy.exe is not found in C:\Program Files*. Now, start installing AzCopy..." -ForegroundColor "Yellow"
      InstallAzCopy
      $AzCopy_path = SearchAzCopy
  }
      $env_path = $env:Path
      for ($i=0; $i -lt $AzCopy_path.count; $i++){
          if ($AzCopy_path.count -eq 1){
              $AzCopy_path_i = $AzCopy_path
          } else {
              $AzCopy_path_i = $AzCopy_path[$i]
          }
          if ($env_path -notlike '*' +$AzCopy_path_i+'*'){
              Write-Host $AzCopy_path_i 'not in system path, add it...'
              [Environment]::SetEnvironmentVariable("Path", "$AzCopy_path_i;$env_path", "Machine")
              $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
              $env_path = $env:Path
          }
  ```

* **Копирование данных в учетную запись частного хранилища BLOB-объектов** из общедоступного BLOB-объекта с помощью AzCopy.

  ```azurepowershell
  Write-Host "AzCopy is copying data from public blob to yo storage account. It may take a while..." -ForegroundColor "Yellow"
  $start_time = Get-Date
  AzCopy.exe /Source:$Source /Dest:$DestURL /DestKey:$StorageAccountKey /S
  $end_time = Get-Date
  $time_span = $end_time - $start_time
  $total_seconds = [math]::Round($time_span.TotalSeconds,2)
  Write-Host "AzCopy finished copying data. Please check your storage account to verify." -ForegroundColor "Yellow"
  Write-Host "This step (copying data from public blob to your storage account) takes $total_seconds seconds." -ForegroundColor "Green"
  ```

* **Загружает данные с помощью polybase (путем запуска лоаддататосклдв. SQL) в Azure синапсе Analytics** из частной учетной записи хранения BLOB-объектов с помощью следующих команд.

  * Создание схемы

    ```sql
    EXEC (''CREATE SCHEMA {schemaname};'');
    ```

  * Создание учетных данных для определенной базы данных

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL {KeyAlias}
    WITH IDENTITY = ''asbkey'' ,
    Secret = ''{StorageAccountKey}''
    ```

  * Создание внешнего источника данных для большого двоичного объекта службы хранилища Azure

    ```sql
    CREATE EXTERNAL DATA SOURCE {nyctaxi_trip_storage}
    WITH
    (
        TYPE = HADOOP,
        LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
        CREDENTIAL = {KeyAlias}
    )
    ;

    CREATE EXTERNAL DATA SOURCE {nyctaxi_fare_storage}
    WITH
    (
        TYPE = HADOOP,
        LOCATION =''wasbs://{ContainerName}@{StorageAccountName}.blob.core.windows.net'',
        CREDENTIAL = {KeyAlias}
    )
    ;
    ```

  * Создание формата внешнего файла для CSV-файла. Данные не сжимаются, а поля разделяются символом вертикальной черты.

    ```sql
    CREATE EXTERNAL FILE FORMAT {csv_file_format}
    WITH
    (
        FORMAT_TYPE = DELIMITEDTEXT,
        FORMAT_OPTIONS
        (
            FIELD_TERMINATOR ='','',
            USE_TYPE_DEFAULT = TRUE
        )
    )
    ;
    ```

  * Создание внешних таблиц (trip и fare) для хранения набора данных такси Нью-Йорка в хранилище BLOB-объектов Azure.

    ```sql
    CREATE EXTERNAL TABLE {external_nyctaxi_fare}
    (
        medallion varchar(50) not null,
        hack_license varchar(50) not null,
        vendor_id char(3),
        pickup_datetime datetime not null,
        payment_type char(3),
        fare_amount float,
        surcharge float,
        mta_tax float,
        tip_amount float,
        tolls_amount float,
        total_amount float
    )
    with (
        LOCATION    = ''/nyctaxifare/'',
        DATA_SOURCE = {nyctaxi_fare_storage},
        FILE_FORMAT = {csv_file_format},
        REJECT_TYPE = VALUE,
        REJECT_VALUE = 12
    )

      CREATE EXTERNAL TABLE {external_nyctaxi_trip}
      (
             medallion varchar(50) not null,
             hack_license varchar(50)  not null,
             vendor_id char(3),
             rate_code char(3),
             store_and_fwd_flag char(3),
             pickup_datetime datetime  not null,
             dropoff_datetime datetime,
             passenger_count int,
             trip_time_in_secs bigint,
             trip_distance float,
             pickup_longitude varchar(30),
             pickup_latitude varchar(30),
             dropoff_longitude varchar(30),
             dropoff_latitude varchar(30)
      )
      with (
          LOCATION    = ''/nyctaxitrip/'',
          DATA_SOURCE = {nyctaxi_trip_storage},
          FILE_FORMAT = {csv_file_format},
          REJECT_TYPE = VALUE,
          REJECT_VALUE = 12
      )
    ```

    - Загрузка данных из внешних таблиц в хранилище BLOB-объектов Azure в Azure синапсе Analytics

      ```sql
      CREATE TABLE {schemaname}.{nyctaxi_fare}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      SELECT *
      FROM   {external_nyctaxi_fare}
      ;

      CREATE TABLE {schemaname}.{nyctaxi_trip}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      SELECT *
      FROM   {external_nyctaxi_trip}
      ;
      ```

    - Создание примера таблицы данных (NYCTaxi_Sample) и вставка данных из него. Предполагает выбор SQL-запросов на таблицы trip и fare. (В некоторых шагах этого пошагового руководства необходимо использовать этот образец таблицы.)

      ```sql
      CREATE TABLE {schemaname}.{nyctaxi_sample}
      WITH
      (
          CLUSTERED COLUMNSTORE INDEX,
          DISTRIBUTION = HASH(medallion)
      )
      AS
      (
          SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount, f.total_amount, f.tip_amount,
          tipped = CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END,
          tip_class = CASE
                  WHEN (tip_amount = 0) THEN 0
                  WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
                  WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
                  WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
                  ELSE 4
              END
          FROM {schemaname}.{nyctaxi_trip} t, {schemaname}.{nyctaxi_fare} f
          WHERE datepart("mi",t.pickup_datetime) = 1
          AND t.medallion = f.medallion
          AND   t.hack_license = f.hack_license
          AND   t.pickup_datetime = f.pickup_datetime
          AND   pickup_longitude <> ''0''
          AND   dropoff_longitude <> ''0''
      )
      ;
      ```

Географическое расположение учетных записей хранения влияет на время загрузки.

> [!NOTE]
> В зависимости от географического расположения учетной записи частного хранилища BLOB-объектов процесс копирования данных из общедоступного большого двоичного объекта в закрытую учетную запись хранения может занять около 15 минут, а процесс загрузки данных из вашей учетной записи хранения в Azure синапсе Analytics может занять 20 минут или больше.
>
>

Необходимо решить, какие действия предпринимать, если исходные и конечные файлы совпадают.

> [!NOTE]
> Если CSV-файлы, копируемые из общедоступного хранилища BLOB-объектов в учетную запись частного хранилища BLOB-объектов, уже существуют в учетной записи частного хранилища BLOB-объектов, AzCopy выведет запрос на их перезапись. Если перезаписывать их не нужно, по запросу системы введите **n** . Чтобы перезаписать **все** файлы, по запросу системы введите **a**. Чтобы перезаписать отдельные CSV-файлы, по запросу системы введите **y** .
>
>

![Выходные данные из AzCopy][21]

Можно использовать собственные данные. Если данные находятся на локальном компьютере в работающем приложении, AzCopy можно использовать для передачи локальных данных в частное хранилище BLOB-объектов Azure. Для этого в команде AzCopy в файле скрипта PowerShell необходимо лишь изменить расположение **источника**`$Source = "http://getgoing.blob.core.windows.net/public/nyctaxidataset"`, указав для него локальный каталог, содержащий данные.

> [!TIP]
> Если ваши данные уже находятся в частном хранилище BLOB-объектов Azure в реальных приложениях, вы можете пропустить шаг AzCopy в сценарии PowerShell и напрямую отправить данные в Azure Azure синапсе Analytics. В сценарий потребуется внести дополнительные изменения, чтобы настроить его согласно формату данных.
>
>

Этот сценарий PowerShell также подключается к сведениям Azure синапсе Analytics в файлах примеров для просмотра данных SQLDW_Explorations. SQL, SQLDW_Explorations. ipynb и SQLDW_Explorations_Scripts. корректировка, чтобы эти три файла были готовы к выполнению мгновенной попытки после завершения сценария PowerShell.

После успешного выполнения вы увидите экран, аналогичный этому:

![Результат успешного выполнения скрипта][20]

## <a name="data-exploration-and-feature-engineering-in-azure-synapse-analytics"></a><a name="dbexplore"></a>Исследование данных и проектирование характеристик в Azure синапсе Analytics
В этом разделе мы выполняем исследование данных и создание признаков, выполняя запросы SQL к Azure синапсе Analytics непосредственно с помощью **Visual Studio Data Tools**. Все SQL-запросы, используемые в этом разделе, можно найти в примере скрипта *SQLDW_Explorations.sql*. Этот файл скачан в локальный каталог во время выполнения сценария PowerShell. Его также можно получить из [GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/SQLDW/SQLDW_Explorations.sql). Но у файла в GitHub нет данных Azure синапсе Analytics, не подключенных к сети.

Подключитесь к Azure синапсе Analytics с помощью Visual Studio с именем входа и паролем Azure синапсе Analytics и откройте **Обозреватель объектов SQL** , чтобы убедиться, что база данных и таблицы были импортированы. Получите файл *SQLDW_Explorations.sql*.

> [!NOTE]
> Чтобы открыть редактор запросов хранилища параллельных данных (PDW), используйте команду **New Query**, выбрав PDW в **обозревателе объектов SQL**. PDW не поддерживает стандартный редактор SQL-запросов.
>
>

Ниже приведены типы задач по исследованию данных и созданию признаков, выполненных в этом разделе.

* Просмотрим распределение данных по нескольким полям в различных временных окнах.
* Исследуем качество данных по полям долготы и широты.
* Создадим метки двоичной и мультиклассовой классификации на основе **tip\_amount**.
* Создадим характеристики и вычислим/сравним расстояния поездок.
* Соединим две таблицы и извлечем случайную выборку, которая послужит основой для построения моделей.

### <a name="data-import-verification"></a>Проверка импорта данных
Эти запросы обеспечивают быструю проверку числа строк и столбцов в таблицах, заполненных ранее с помощью параллельного массового импорта Polybase.

--Отчет число строк в таблице <nyctaxi_trip> без просмотра таблицы

   ```sql
   SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
   ```

--Отчет число столбцов в таблице <nyctaxi_trip>

   ```sql
   SELECT COUNT(*) FROM information_schema.columns WHERE table_name = '<nyctaxi_trip>' AND table_schema = '<schemaname>'
   ```

**Выходные данные.** Вы должны получить 173 179 759 строк и 14 столбцов.

### <a name="exploration-trip-distribution-by-medallion"></a>Просмотр: распределение поездок по параметру medallion
В этом примере запроса определяются медальоны (номера) такси, которые осуществили больше 100 поездок за заданный период времени. Запрос будет использовать секционированный доступ к таблице, так как он обусловлен схемой секционирования **pickup\_datetime**. При запросе к полному набору данных также будет задействовано сканирование секционированной таблицы и/или индекса.

```sql
SELECT medallion, COUNT(*)
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
GROUP BY medallion
HAVING COUNT(*) > 100
```

**Выходные данные:** Запрос должен возвращать таблицу со строками, в которых задано значение 13 369 медальонов (такси) и количество поездок, выполненных в 2013. В последнем столбце отображается счетчик поездок.

### <a name="exploration-trip-distribution-by-medallion-and-hack_license"></a>Просмотр: распределение поездок по параметрам medallion и hack_license
В этом примере определяются медальоны (номера) такси и номера hack_license (лицензий) водителей, которые осуществили больше 100 поездок за заданный период времени.

```sql
SELECT medallion, hack_license, COUNT(*)
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20130131'
GROUP BY medallion, hack_license
HAVING COUNT(*) > 100
```

**Выходные данные.** По результатам запроса возвращается таблица, в 13 369 строках которой перечислены 13 369 автомобилей (водителей), осуществивших в 2013 году более 100 поездок. В последнем столбце отображается счетчик поездок.

### <a name="data-quality-assessment-verify-records-with-incorrect-longitude-andor-latitude"></a>Оценка качества данных: проверка записей с неправильными значениями долготы и/или широты
В этом примере анализируется, содержится ли в каких-либо полях долготы и/или широты неверное значение (количество градусов должно быть в пределах от -90 до 90) или значение координат (0, 0).

```sql
SELECT COUNT(*) FROM <schemaname>.<nyctaxi_trip>
WHERE pickup_datetime BETWEEN '20130101' AND '20130331'
AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(pickup_latitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND 90
OR    CAST(dropoff_latitude AS float) NOT BETWEEN -90 AND 90
OR    (pickup_longitude = '0' AND pickup_latitude = '0')
OR    (dropoff_longitude = '0' AND dropoff_latitude = '0'))
```

**Выходные данные.** По результатам запроса возвращаются 837 467 поездок с недопустимыми значениями долготы и (или) широты.

### <a name="exploration-tipped-vs-not-tipped-trips-distribution"></a>Изучение: распределение поездок с чаевыми и без чаевых
В этом примере пользователь узнает о соотношении количества поездок, когда водителю дали на чай, к количеству поездок, когда чаевых не было (пользователь ищет сведения за определенный период времени или, если нужны сведения за полный год, во всех данных набора). Это распределение отражает распределение двоичных меток, которые в дальнейшем будут использоваться для моделирования двоичной классификации.

```sql
SELECT tipped, COUNT(*) AS tip_freq FROM (
  SELECT CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped, tip_amount
  FROM <schemaname>.<nyctaxi_fare>
  WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tipped
```

**Выходные данные.** По результатам запроса возвращаются следующие результаты по частотности чаевых за 2013 год: 90 447 622 поездок с чаевыми и 82 264 709 — без чаевых.

### <a name="exploration-tip-classrange-distribution"></a>Изучение: распределение классов и диапазонов чаевых
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
FROM <schemaname>.<nyctaxi_fare>
WHERE pickup_datetime BETWEEN '20130101' AND '20131231') tc
GROUP BY tip_class
```

**Выходные данные:**

| tip_class | tip_freq |
| --- | --- |
| 1 |82230915 |
| 2 |6198803 |
| 3 |1932223 |
| 0 |82264625 |
| 4 |85765 |

### <a name="exploration-compute-and-compare-trip-distance"></a>Изучение: вычисление и сравнение расстояния поездок
В этом примере значения долготы и широты начальных и конечных пунктов поездок преобразуются в географические точки SQL, вычисляется расстояние поездки по разности географических точек и возвращается случайная выборка результатов для сравнения. В примере результаты ограничиваются только допустимыми координатами с помощью запроса оценки качества данных, описанного ранее.

```sql
/****** Object:  UserDefinedFunction [dbo].[fnCalculateDistance] ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
  DROP FUNCTION fnCalculateDistance
GO

-- User-defined function to calculate the direct distance  in mile between two geographical coordinates.
CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

RETURNS float
AS
BEGIN
      DECLARE @distance decimal(28, 10)
      -- Convert to radians
      SET @Lat1 = @Lat1 / 57.2958
      SET @Long1 = @Long1 / 57.2958
      SET @Lat2 = @Lat2 / 57.2958
      SET @Long2 = @Long2 / 57.2958
      -- Calculate distance
      SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
      --Convert to miles
      IF @distance <> 0
      BEGIN
        SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
      END
      RETURN @distance
END
GO

SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
FROM <schemaname>.<nyctaxi_trip>
WHERE datepart("mi",pickup_datetime)=1
AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND pickup_longitude != '0' AND dropoff_longitude != '0'
```

### <a name="feature-engineering-using-sql-functions"></a>Проектирование признаков с помощью функций SQL
Иногда функции SQL могут быть полезны при проектировании признаков. В этом пошаговом руководстве мы определили функцию SQL для расчета прямого расстояния между расположениями посадки и высадки. Описанные ниже сценарии SQL можно выполнять в **средствах Visual Studio для работы с данными**.

Вот сценарий SQL для определения функции расстояния:

```sql
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

IF EXISTS (SELECT * FROM sys.objects WHERE type IN ('FN', 'IF') AND name = 'fnCalculateDistance')
  DROP FUNCTION fnCalculateDistance
GO

-- User-defined function calculate the direct distance between two geographical coordinates.
CREATE FUNCTION [dbo].[fnCalculateDistance] (@Lat1 float, @Long1 float, @Lat2 float, @Long2 float)

RETURNS float
AS
BEGIN
      DECLARE @distance decimal(28, 10)
      -- Convert to radians
      SET @Lat1 = @Lat1 / 57.2958
      SET @Long1 = @Long1 / 57.2958
      SET @Lat2 = @Lat2 / 57.2958
      SET @Long2 = @Long2 / 57.2958
      -- Calculate distance
      SET @distance = (SIN(@Lat1) * SIN(@Lat2)) + (COS(@Lat1) * COS(@Lat2) * COS(@Long2 - @Long1))
      --Convert to miles
      IF @distance <> 0
      BEGIN
        SET @distance = 3958.75 * ATAN(SQRT(1 - POWER(@distance, 2)) / @distance);
      END
      RETURN @distance
END
GO
```

Вот пример вызова этой функции в SQL-запросе для создания признаков:

--Пример запроса для вызова функции для создания функций

   ```sql
SELECT pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude,
dbo.fnCalculateDistance(pickup_latitude, pickup_longitude, dropoff_latitude, dropoff_longitude) AS DirectDistance
FROM <schemaname>.<nyctaxi_trip>
WHERE datepart("mi",pickup_datetime)=1
AND CAST(pickup_latitude AS float) BETWEEN -90 AND 90
AND CAST(dropoff_latitude AS float) BETWEEN -90 AND 90
AND pickup_longitude != '0' AND dropoff_longitude != '0'
   ```

**Выходные данные.** По результатам запроса формируется таблица (из 2 803 538 строк) с координатами посадки и высадки пассажиров и дальностью поездок в милях. Ниже приведены результаты для первых трех строк:

| (Номер строки) | pickup_latitude | pickup_longitude | dropoff_latitude | dropoff_longitude | DirectDistance |
| --- | --- | --- | --- | --- | --- |
| 1 |40,731804 |–74,001083 |40,736622 |–73,988953 |0,7169601222 |
| 2 |40,715794 |–74,010635 |40,725338 |–74,00399 |0,7448343721 |
| 3 |40,761456 |–73,999886 |40,766544 |–73,988228 |0,7037227967 |

### <a name="prepare-data-for-model-building"></a>Подготовка данных для построения модели
Следующий запрос соединяет таблицы **nyctaxi\_trip** и **nyctaxi\_fare**, создает метку двоичной классификации **tipped**, метку многоклассовой классификации **tip\_class**, а также извлекает выборку из полного соединенного набора данных. Выборка предполагает получение подмножества поездок на основе данных времени посадок.  Этот запрос можно скопировать, а затем вставлять непосредственно в модуль[импорта] [данных] [(классическая модель) машинное обучение Azure Studio](https://studio.azureml.net) для прямого приема данных из экземпляра базы данных SQL в Azure. Запрос исключает записи с неверными (0, 0) координатами.

```sql
SELECT t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,     f.total_amount, f.tip_amount,
    CASE WHEN (tip_amount > 0) THEN 1 ELSE 0 END AS tipped,
    CASE WHEN (tip_amount = 0) THEN 0
        WHEN (tip_amount > 0 AND tip_amount <= 5) THEN 1
        WHEN (tip_amount > 5 AND tip_amount <= 10) THEN 2
        WHEN (tip_amount > 10 AND tip_amount <= 20) THEN 3
        ELSE 4
    END AS tip_class
FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
WHERE datepart("mi",t.pickup_datetime) = 1
AND   t.medallion = f.medallion
AND   t.hack_license = f.hack_license
AND   t.pickup_datetime = f.pickup_datetime
AND   pickup_longitude != '0' AND dropoff_longitude != '0'
```

Когда вы будете готовы перейти к машинному обучению Azure, вы можете:

1. Сохраните окончательный SQL-запрос для извлечения и выборки данных и скопируйте и вставьте запрос непосредственно в модуль импорт данных][Import-Data] в машинное обучение Azure или
2. Сохранить выбранные и сконструированные данные, которые планируется использовать для создания модели в новой таблице Azure синапсе Analytics, и использовать новую таблицу в модуле [импорта данных][Import-Data] в машинное обучение Azure. Сценарий PowerShell на более раннем этапе выполнил эту задачу. Данные можно считать прямо из этой таблицы в модуле "Импорт данных".

## <a name="data-exploration-and-feature-engineering-in-ipython-notebook"></a><a name="ipnb"></a>Просмотр данных и проектирование характеристик в IPython Notebook
В этом разделе мы выполним исследование данных и создание признаков с помощью запросов Python и SQL к созданной ранее службе аналитики Azure синапсе. Пример файла IPython Notebook с именем **SQLDW_Explorations.ipynb** и файла скрипта Python **SQLDW_Explorations_Scripts.py** скачаны в локальный каталог. Они также доступны на веб-сайте [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/SQLDW). В сценариях Python эти два файла идентичны. Файл сценария Python предоставляется, если нет сервера IPython Notebook. Эти два примера файлов Python предназначены для использования с **Python 2.7**.

Необходимые сведения об Azure синапсе Analytics в примере записной книжке IPython Notebook и файле сценария Python, скачанном на локальный компьютер, были подключены ранее с помощью сценария PowerShell. Они выполняются без изменений.

Если вы уже настроили рабочую область Машинное обучение Azure, вы можете напрямую отправить пример записной книжки IPython Notebook в службу записных книжек IPython Notebook AzureML и запустить ее. Ниже приведены шаги для отправки в службу записных книжек AzureML IPython Notebook.

1. Войдите в рабочую область Машинное обучение Azure, щелкните **студия** в верхней части экрана и щелкните **записные книжки** в левой части веб-страницы.

    ![Щелкните Studio, а затем — NOTEBOOKS][22]
2. Щелкните **создать** в левом нижнем углу веб-страницы и выберите **Python 2**. Затем введите имя для Notebook и щелкните значок галочки для создания пустого IPython Notebook.

    ![Щелкните "СОЗДАТЬ", а затем выберите Python 2][23]
3. Щелкните символ **Jupyter** в левом верхнем углу новой записной книжки IPython Notebook.

    ![Щелкните символ Jupyter][24]
4. Перетащите пример IPython Notebook на страницу **дерева** службы IPython Notebook машинного обучения Azure и нажмите кнопку **Отправить**. После этого пример IPython Notebook будет отправлен в службу IPython Notebook Машинного обучения Azure.

    ![Щелкните "Отправить".][25]

Чтобы запустить пример IPython Notebook или файл сценария Python, необходимо установить следующие пакеты Python. Если вы используете службу IPython Notebook Машинного обучения Azure, эти пакеты уже установлены.

- pandas
- numpy
- matplotlib
- pyodbc
- PyTables

При создании расширенных аналитических решений на Машинное обучение Azure с большими данными Рекомендуемая последовательность:

* Считайте небольшую выборку данных во фрейм данных, размещенный в памяти.
* Выполните несколько визуализаций и просмотров с использованием выборки данных.
* Поэкспериментируйте с проектированием характеристик с использованием выборки данных.
* Для просмотра больших данных, обработки данных и проектирования признаков используйте Python, чтобы выдавать SQL запросы непосредственно в Azure синапсе Analytics.
* Определите размер выборки, подходящий для создания модели машинного обучения Azure.

Ниже приведены несколько примеров изучения данных, визуализации данных и проектирования признаков. Дополнительные исследования данных можно найти в примере IPython Notebook и файле сценария Python.

### <a name="initialize-database-credentials"></a>Инициализация учетных данных базы данных
Инициализируйте параметры подключения к базе данных в следующих переменных:

```sql
SERVER_NAME=<server name>
DATABASE_NAME=<database name>
USERID=<user name>
PASSWORD=<password>
DB_DRIVER = <database driver>
```

### <a name="create-database-connection"></a>Создание подключения к базе данных
Вот строка, которая создает подключение к базе данных.

```sql
CONNECTION_STRING = 'DRIVER={'+DRIVER+'};SERVER='+SERVER_NAME+';DATABASE='+DATABASE_NAME+';UID='+USERID+';PWD='+PASSWORD
conn = pyodbc.connect(CONNECTION_STRING)
```

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_trip"></a>Сообщение числа строк и столбцов в таблице <nyctaxi_trip>

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_trip>')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('<nyctaxi_trip>') AND table_schema = ('<schemaname>')
''', conn)

print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Общее число строк = 173179759
* Общее число столбцов = 14

### <a name="report-number-of-rows-and-columns-in-table-nyctaxi_fare"></a>Сообщение числа строк и столбцов в таблице <nyctaxi_fare>

```sql
nrows = pd.read_sql('''
    SELECT SUM(rows) FROM sys.partitions
    WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_fare>')
''', conn)

print 'Total number of rows = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''
    SELECT COUNT(*) FROM information_schema.columns
    WHERE table_name = ('<nyctaxi_fare>') AND table_schema = ('<schemaname>')
''', conn)

    print 'Total number of columns = %d' % ncols.iloc[0,0]
```

* Общее число строк = 173179759
* Общее количество столбцов — 11.

### <a name="read-in-a-small-data-sample-from-the-azure-synapse-analytics-database"></a>Чтение — пример небольшого объема данных из базы данных Azure синапсе Analytics

```sql
t0 = time.time()

query = '''
    SELECT TOP 10000 t.*, f.payment_type, f.fare_amount, f.surcharge, f.mta_tax,
        f.tolls_amount, f.total_amount, f.tip_amount
    FROM <schemaname>.<nyctaxi_trip> t, <schemaname>.<nyctaxi_fare> f
    WHERE datepart("mi",t.pickup_datetime) = 1
    AND   t.medallion = f.medallion
    AND   t.hack_license = f.hack_license
    AND   t.pickup_datetime = f.pickup_datetime
'''

df1 = pd.read_sql(query, conn)

t1 = time.time()
print 'Time to read the sample table is %f seconds' % (t1-t0)

print 'Number of rows and columns retrieved = (%d, %d)' % (df1.shape[0], df1.shape[1])
```

Время чтения таблицы выборки — 14,096 495 секунды.
Количество полученных строк и столбцов — 1000 и 21.

### <a name="descriptive-statistics"></a>Описательная статистика
Теперь все готово для изучения данных выборки. Начнем с рассмотрения описательной статистики для **trip\_distance** (или любых других выбранных полей).

```sql
df1['trip_distance'].describe()
```

### <a name="visualization-box-plot-example"></a>Визуализация: пример блочной диаграммы
Далее рассмотрим блочную диаграмму расстояний поездок для визуализации квантилей.

```sql
df1.boxplot(column='trip_distance',return_type='dict')
```

![Вывод блочной диаграммы][1]

### <a name="visualization-distribution-plot-example"></a>Визуализация: пример графика распределения
Это график, на котором показано распределение и гистограмма для примеров расстояний выборки.

```sql
fig = plt.figure()
ax1 = fig.add_subplot(1,2,1)
ax2 = fig.add_subplot(1,2,2)
df1['trip_distance'].plot(ax=ax1,kind='kde', style='b-')
df1['trip_distance'].hist(ax=ax2, bins=100, color='k')
```

![Вывод графика распределения][2]

### <a name="visualization-bar-and-line-plots"></a>Визуализация: гистограммы и линейные графики
В этом примере мы сегментируем расстояния поездок на пять ячеек и визуализируем результаты сегментирования.

```sql
trip_dist_bins = [0, 1, 2, 4, 10, 1000]
df1['trip_distance']
trip_dist_bin_id = pd.cut(df1['trip_distance'], trip_dist_bins)
trip_dist_bin_id
```

Мы можем изобразить описанное выше распределение по ячейкам в виде гистограммы или линейного графика, выполнив

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='bar')
```

![вывод гистограммы][3]

и

```sql
pd.Series(trip_dist_bin_id).value_counts().plot(kind='line')
```

![вывод линейного графика][4]

### <a name="visualization-scatterplot-examples"></a>Визуализация: примеры точечных диаграмм
Мы отобразим точечную диаграмму для параметров **trip\_time\_in\_secs** и **trip\_distance**, чтобы проверить возможную корреляцию.

```sql
plt.scatter(df1['trip_time_in_secs'], df1['trip_distance'])
```

![Вывод точечной диаграмме связи между временем и расстоянием][6]

Точно также мы можем проверить связь между **rate\_code** и **trip\_distance**.

```sql
plt.scatter(df1['passenger_count'], df1['trip_distance'])
```

![Вывод точечной диаграмме связи между кодом и расстоянием][8]

### <a name="data-exploration-on-sampled-data-using-sql-queries-in-ipython-notebook"></a>Изучение выборки данных с помощью SQL-запросов в IPython Notebook
В этом разделе рассматривается распределение данных с использованием данных выборки, сохраненных в новой таблице, созданной ранее. Аналогичные исследования могут быть выполнены с помощью исходных таблиц.

#### <a name="exploration-report-number-of-rows-and-columns-in-the-sampled-table"></a>Исследование: сообщение количества строк и столбцов в таблице выборки

```sql
nrows = pd.read_sql('''SELECT SUM(rows) FROM sys.partitions WHERE object_id = OBJECT_ID('<schemaname>.<nyctaxi_sample>')''', conn)
print 'Number of rows in sample = %d' % nrows.iloc[0,0]

ncols = pd.read_sql('''SELECT count(*) FROM information_schema.columns WHERE table_name = ('<nyctaxi_sample>') AND table_schema = '<schemaname>'''', conn)
print 'Number of columns in sample = %d' % ncols.iloc[0,0]
```

#### <a name="exploration-tippednot-tripped-distribution"></a>Исследование: распределение поездок с чаевыми и без чаевых

```sql
query = '''
SELECT tipped, count(*) AS tip_freq
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY tipped
    '''

    pd.read_sql(query, conn)
```

#### <a name="exploration-tip-class-distribution"></a>Изучение: распределение классов чаевых

```sql
query = '''
    SELECT tip_class, count(*) AS tip_freq
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY tip_class
'''

tip_class_dist = pd.read_sql(query, conn)
```

#### <a name="exploration-plot-the-tip-distribution-by-class"></a>Исследование: построение диаграммы классового распределения чаевых

```sql
tip_class_dist['tip_freq'].plot(kind='bar')
```

![График № 26][26]

#### <a name="exploration-daily-distribution-of-trips"></a>Исследование: ежедневное распределение поездок

```sql
query = '''
    SELECT CONVERT(date, dropoff_datetime) AS date, COUNT(*) AS c
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY CONVERT(date, dropoff_datetime)
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-per-medallion"></a>Исследование: распределение поездок по параметру medallion

```sql
query = '''
    SELECT medallion,count(*) AS c
    FROM <schemaname>.<nyctaxi_sample>
    GROUP BY medallion
'''

pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>Изучение: распределение поездок по медальону и номеру лицензии водителя

```sql
query = '''select medallion, hack_license,count(*) from <schemaname>.<nyctaxi_sample> group by medallion, hack_license'''
pd.read_sql(query,conn)
```

#### <a name="exploration-trip-time-distribution"></a>Просмотр: распределение поездок по времени

```sql
query = '''select trip_time_in_secs, count(*) from <schemaname>.<nyctaxi_sample> group by trip_time_in_secs order by count(*) desc'''
pd.read_sql(query,conn)
```

#### <a name="exploration-trip-distance-distribution"></a>Изучение: распределение поездок по расстоянию

```sql
query = '''select floor(trip_distance/5)*5 as tripbin, count(*) from <schemaname>.<nyctaxi_sample> group by floor(trip_distance/5)*5 order by count(*) desc'''
pd.read_sql(query,conn)
```

#### <a name="exploration-payment-type-distribution"></a>Изучение: распределение поездок по типу оплаты

```sql
query = '''select payment_type,count(*) from <schemaname>.<nyctaxi_sample> group by payment_type'''
pd.read_sql(query,conn)
```

#### <a name="verify-the-final-form-of-the-featurized-table"></a>Проверка конечной формы таблицы с признаками

```sql
query = '''SELECT TOP 100 * FROM <schemaname>.<nyctaxi_sample>'''
pd.read_sql(query,conn)
```

## <a name="build-models-in-azure-machine-learning"></a><a name="mlmodel"></a>Построение моделей в компоненте машинного обучения Azure
Теперь мы готовы перейти к созданию модели и развертыванию модели в [машинное обучение Azure](https://studio.azureml.net). Данные готовы для любой из задач прогнозирования, определенных ранее:

1. **Двоичная классификация**: для прогнозирования того, оплачивается ли TIP для поездки.
2. **Многоклассовая классификация**: для прогнозирования диапазона Tip в соответствии с ранее определенными классами.
3. **Регрессионная задача**: прогнозирование количества подсказок, оплаченных за поездку.

Чтобы начать упражнение по моделированию, войдите в рабочую область **машинное обучение Azure (классическая)** . Если вы еще не создали рабочую область машинного обучения, см. статью [Создание рабочей области машинное обучение Azure Studio (классическая модель)](../classic/create-workspace.md).

1. Чтобы приступить к работе с Машинное обучение Azure, см [. раздел что такое машинное обучение Azure Studio (классическая модель)?](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)
2. Войдите в [машинное обучение Azure Studio (классическая модель)](https://studio.azureml.net).
3. На домашней странице Машинное обучение Studio (классическая модель) содержится множество информации, видеороликов, руководств, ссылок на модули и другие ресурсы. Дополнительные сведения о Машинное обучение Azure см. в [машинное обучение Azure центре документации](https://azure.microsoft.com/documentation/services/machine-learning/).

Типичный обучающий эксперимент предусматривает следующие шаги:

1. Создайте **Новый** эксперимент.
2. Получение данных в Машинное обучение Azure Studio (классическая модель).
3. Предварительная обработка, преобразование и обработка данных по мере необходимости.
4. Создать необходимые характеристики.
5. Разбить данные на учебные/проверочные/тестовые наборы данных (или иметь отдельные наборы данных для каждой из этих целей).
6. Выбрать один или несколько алгоритмов машинного обучения в зависимости от решаемой задачи обучения. Например, двоичная классификация, многоклассовая классификация, регрессия.
7. Обучить одну или несколько моделей с помощью учебного набора данных.
8. Оценить проверочный набор данных с помощью обученных моделей.
9. Вычислить модель (модели) для расчета соответствующих показателей для задачи по обучению.
10. Настройте модели и выберите наилучшую модель для развертывания.

В этом упражнении мы уже изучили и разработали данные в Azure синапсе Analytics и решили принять пример размера для приема в Машинное обучение Azure Studio (классическая модель). Вот процедура, позволяющая создать одну или несколько моделей прогнозирования:

1. Данные можно получить в Машинное обучение Azure Studio (классическая модель) с помощью модуля [Импорт данных][Import-Data] , который доступен в разделе **входные и выходные данные** . Дополнительные сведения см. на странице со справочной информацией о модуле [импорта данных][import-data].

    ![Модуль "Импорт данных" в Машинном обучении Azure][17]
2. На панели **Properties** (Свойства) в списке **Data source** (Источник данных) выберите **Azure SQL Database** (База данных SQL Azure).
3. Введите DNS-имя базы данных в поле **Имя сервера базы данных** . Формат: `tcp:<your_virtual_machine_DNS_name>,1433`
4. Введите **Имя базы данных** в соответствующее поле.
5. Введите *имя пользователя SQL* в поле **Server user account name** (Имя учетной записи пользователя сервера) и *пароль* в поле **Server user account password** (Пароль учетной записи пользователя сервера).
7. В области текст **запроса к базе данных** вставьте запрос, извлекающий необходимые поля базы данных (включая все вычисленные поля, такие как метки), а затем выберите для данных нужный размер выборки.

Пример эксперимента двоичной классификации, считывающего данные непосредственно из базы данных Azure синапсе Analytics, приведен на рисунке ниже (не забудьте заменить имена таблиц nyctaxi_trip и nyctaxi_fare именем схемы и именами таблиц, использованными в пошаговом руководстве). Подобные эксперименты можно сконструировать для задач мультиклассовой классификации и регрессии.

![Чат Машинного обучения Azure][10]

> [!IMPORTANT]
> В примерах запросов на извлечение и выборку данных для моделирования, указанных в предыдущих разделах, **все метки для трех упражнений по моделированию включены в запрос**. Важным (обязательным) шагом каждого из упражнений по моделированию является **исключение** ненужных меток для двух остальных задач, а также любых **целевых утечек**. Например, при двоичной классификации используйте метку **tipped** и исключите поля **tip\_class**, **tip\_amount** и **total\_amount**. Последние являются целевыми утечками, поскольку подразумевают, что чаевые выплачены.
>
> Чтобы исключить все ненужные столбцы или целевые утечки, можно воспользоваться модулем [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных) или [Edit Metadata][edit-metadata] (Изменение метаданных). Дополнительные сведения см. на страницах справки модулей [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных) и [Edit Metadata][edit-metadata] (Изменение метаданных).
>
>

## <a name="deploy-models-in-azure-machine-learning"></a><a name="mldeploy"></a>Развертывание моделей в службе Машинного обучения Azure
Когда модель готова, ее можно легко развернуть в виде веб-службы непосредственно из эксперимента. Дополнительные сведения о развертывании веб-служб Azure ML см. в статье [Развертывание веб-службы Машинного обучения Azure](../classic/deploy-a-machine-learning-web-service.md).

Чтобы развернуть новую веб-службу, необходимо выполнить такие действия.

1. Создать эксперимент по количественной оценке.
2. Развернуть веб-службу.

Чтобы создать оценивающий эксперимент на основе учебного эксперимента со статусом **Завершено**, щелкните **Create scoring experiment** (Создать оценивающий эксперимент) на нижней панели действий.

![Система оценок Azure][18]

Служба машинного обучения Azure попытается создать эксперимент по количественной оценке на основе компонентов учебного эксперимента. В частности, она:

1. Сохранит обученную модель и удалит модули обучения модели.
2. Идентифицирует логический **порт ввода** для представления ожидаемой схемы входных данных.
3. Идентифицирует логический **порт вывода** для представления ожидаемой схемы вывода веб-службы.

После создания эксперимента оценки Проверьте результаты и внесите необходимые изменения. Типичная корректировка заключается в замене входного набора данных или запроса на другой, исключающий поля меток, поскольку эти поля меток не будут сопоставляться со схемой при вызове службы. Также рекомендуется уменьшить размер входного набора данных и (или) запроса на несколько записей, чтобы указать входную схему. Для порта вывода зачастую исключаются все входные поля и в вывод включаются только **Scored Labels** (Оцененные метки) и **Scored Probabilities** (Оцененные вероятности) с помощью модуля [Select Columns in Dataset][select-columns] (Выбор столбцов в наборе данных).

Пример оценивающего эксперимента показан на рисунке ниже. Когда все будет готово к развертыванию, нажмите кнопку **ОПУБЛИКОВАТЬ ВЕБ-СЛУЖБУ** на нижней панели действий.

![Публикация Машинного обучения Azure][11]

## <a name="summary"></a>Итоги
Итак, в этом пошаговом руководстве мы создали среду обработки и анализа данных, работали с большим общедоступным набором данных на протяжении всего процесса обработки и анализа данных группы от получения данных до обучения модели и развертывания веб-службы машинного обучения Azure.

### <a name="license-information"></a>Сведения о лицензии
Этот образец пошагового руководства и сопровождающие его сценарии и файлы IPython Notebook предоставлены корпорацией Майкрософт на условиях лицензии MIT. Чтобы получить дополнительные сведения, проверьте файл LICENSE.txt в каталоге примера кода на сайте GitHub.

## <a name="references"></a>Ссылки
- [Страница загрузки андрéс Монрой Нью такси](https://www.andresmh.com/nyctaxitrips/)
- [Получение Нью данные о поездках в такси Whong)](https://chriswhong.com/open-data/foil_nyc_taxi/)
- [Исследование и статистика Нью такси и Лимузиновных Комиссии](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

[1]: ./media/sqldw-walkthrough/sql-walkthrough_26_1.png
[2]: ./media/sqldw-walkthrough/sql-walkthrough_28_1.png
[3]: ./media/sqldw-walkthrough/sql-walkthrough_35_1.png
[4]: ./media/sqldw-walkthrough/sql-walkthrough_36_1.png
[5]: ./media/sqldw-walkthrough/sql-walkthrough_39_1.png
[6]: ./media/sqldw-walkthrough/sql-walkthrough_42_1.png
[7]: ./media/sqldw-walkthrough/sql-walkthrough_44_1.png
[8]: ./media/sqldw-walkthrough/sql-walkthrough_46_1.png
[9]: ./media/sqldw-walkthrough/sql-walkthrough_71_1.png
[10]: ./media/sqldw-walkthrough/azuremltrain.png
[11]: ./media/sqldw-walkthrough/azuremlpublish.png
[12]: ./media/sqldw-walkthrough/ssmsconnect.png
[13]: ./media/sqldw-walkthrough/executescript.png
[14]: ./media/sqldw-walkthrough/sqlserverproperties.png
[15]: ./media/sqldw-walkthrough/sqldefaultdirs.png
[16]: ./media/sqldw-walkthrough/bulkimport.png
[17]: ./media/sqldw-walkthrough/amlreader.png
[18]: ./media/sqldw-walkthrough/amlscoring.png
[19]: ./media/sqldw-walkthrough/ps_download_scripts.png
[20]: ./media/sqldw-walkthrough/ps_load_data.png
[21]: ./media/sqldw-walkthrough/azcopy-overwrite.png
[22]: ./media/sqldw-walkthrough/ipnb-service-aml-1.png
[23]: ./media/sqldw-walkthrough/ipnb-service-aml-2.png
[24]: ./media/sqldw-walkthrough/ipnb-service-aml-3.png
[25]: ./media/sqldw-walkthrough/ipnb-service-aml-4.png
[26]: ./media/sqldw-walkthrough/tip_class_hist_1.png


<!-- Module References -->
[edit-metadata]: /azure/machine-learning/studio-module-reference/edit-metadata
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[import-data]: /azure/machine-learning/studio-module-reference/import-data