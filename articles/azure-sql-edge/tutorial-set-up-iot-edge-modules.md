---
title: Настройка модулей IoT Edge в службе SQL Azure для пограничных вычислений
description: В части 2 (из 3) этого руководства по SQL Azure для пограничных вычислений, посвященного прогнозированию примеси железной руды, вы настроите модули и подключения IoT Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sourabha, sstein
ms.date: 09/22/2020
ms.openlocfilehash: 5d768fdc2540496769883d839cfbb4f009a2000c
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106077641"
---
# <a name="set-up-iot-edge-modules-and-connections"></a>Настройка модулей и подключений IoT Edge

В части 2 (из 3) этого руководства по прогнозированию примеси железной руды в SQL Azure для пограничных вычислений вы настроите следующие модули IoT Edge:

- SQL Azure для пограничных вычислений
- Модуль генератора данных IoT Edge

## <a name="specify-container-registry-credentials"></a>Указание учетных данных реестра контейнеров

Необходимо указать учетные данные контейнера для образов модуля, на которых размещены образы модулей. Их можно найти в реестре контейнеров, созданном в группе ресурсов. Перейдите к разделу **Ключи доступа**. Запишите следующие поля:

- Имя реестра
- сервер входа;
- Имя пользователя
- Пароль

Теперь укажите учетные данные контейнера в модуле IoT Edge.

1. Перейдите в центр Интернета вещей, созданный в группе ресурсов.

2. В подразделе **IoT Edge** в разделе **Автоматическое управление устройствами** щелкните **Идентификатор устройства**. В этом руководстве используется идентификатор `IronOrePredictionDevice`.

3. Выберите раздел **Выбор модулей**.

4. В разделе **Учетные данные реестра контейнеров** введите следующие значения:

   | _Поле_   | _Значение_       |
   | -------   | -------       |
   | Имя      | Имя реестра |
   | Адрес   | сервер входа;  | 
   | Имя пользователя | Имя пользователя      | 
   | Пароль  | Пароль      | 
  
## <a name="build-push-and-deploy-the-data-generator-module"></a>Создание, отправка и развертывание модуля генератора данных

1. Клонируйте [файлы проекта](https://github.com/microsoft/sqlsourabh/tree/main/SQLEdgeSamples/IoTEdgeSamples/IronOreSilica) на компьютер.
2. Откройте файл **IronOre_Silica_Predict.sln** в Visual Studio 2019.
3. Обновите сведения о реестре контейнеров в файле **deployment.template.json**. 
   ```json
   "registryCredentials":{
        "RegistryName":{
            "username":"",
            "password":""
            "address":""
        }
    }
   ```
4. Обновите файл **modules.json**, чтобы указать целевой реестр контейнеров (или репозиторий для модуля).
   ```json
   "image":{
        "repository":"samplerepo.azurecr.io/ironoresilicapercent",
        "tag":
    }
   ```
5. Выполните проект в режиме отладки или выпуска, чтобы убедиться в отсутствии проблем. 
6. Отправьте проект в реестр контейнеров, щелкнув правой кнопкой мыши имя проекта и выбрав **Создать и отправить модули IoT Edge**.
7. Разверните модуль генератора данных в качестве модуля IoT Edge на устройстве Edge. 

## <a name="deploy-the-azure-sql-edge-module"></a>Развертывание модуля SQL Azure для пограничных вычислений

1. Разверните модуль SQL Azure для пограничных вычислений, щелкнув **+ Добавить**, а затем **Модуль Marketplace**. 

2. В колонке **Модуль IoT Edge в Marketplace** найдите *SQL Azure для пограничных вычислений* и выберите *Azure SQL Edge Developer* (Разработчик SQL Azure для пограничных вычислений). 

3. Щелкните добавленный модуль *SQL Azure для пограничных вычислений* в разделе **Модули Azure IoT Edge**, чтобы настроить модуль SQL Azure для пограничных вычислений. Дополнительные сведения о параметрах конфигурации см. в статье [Развертывание SQL Azure для пограничных вычислений](./deploy-portal.md).

4. Добавьте переменную среды `MSSQL_PACKAGE` в развертывание модуля *SQL Azure для пограничных вычислений* и укажите URL-адрес SAS DACPAC-файла базы данных, созданного на шаге 8 в [первой части](tutorial-deploy-azure-resources.md) этого учебника.

5. Нажмите кнопку **Update**(Обновить).

6. На странице **Настройка модулей на устройстве** щелкните **Далее: Маршруты >** .

7. В области "Маршруты" на странице **Настройка модулей на устройстве:** укажите маршруты для связи модуля с центром IoT Edge, как описано ниже. Обязательно обновите имена модулей в определениях маршрутов ниже.

   ```
   FROM /messages/modules/<your_data_generator_module>/outputs/IronOreMeasures INTO
   BrokeredEndpoint("/modules/<your_azure_sql_edge_module>/inputs/IronOreMeasures")
   ```

   Пример:

   ```
   FROM /messages/modules/ASEDataGenerator/outputs/IronOreMeasures INTO BrokeredEndpoint("/modules/AzureSQLEdge/inputs/IronOreMeasures")
   ```


7. На странице **Настройка модулей на устройстве** щелкните **Далее: Просмотр и создание >**

8. На странице **Настройка модулей на устройстве** щелкните **Создать**

## <a name="create-and-start-the-t-sql-streaming-job-in-azure-sql-edge"></a>Создайте и запустите задание потоковой передачи T-SQL в SQL Azure для пограничных вычислений.

1. Откройте Azure Data Studio.

2. На вкладке **Приветствие** запустите новое подключение со следующими сведениями:

   |_Поле_|_Значение_|
   |-------|-------|
   |Тип соединений| Microsoft SQL Server|
   |Сервер|Общедоступный IP-адрес, упомянутый в виртуальной машине, созданной для этой демо-версии|
   |Имя пользователя|sa|
   |Пароль|Надежный пароль, который использовался при создании экземпляра SQL Azure для пограничных вычислений.|
   |База данных|По умолчанию|
   |Группа серверов|По умолчанию|
   |Имя (необязательно)|Укажите необязательное имя|

3. Добавьте новый отчет, щелкнув **Подключить**

4. На вкладке меню **Файл** откройте новую записную книжку или используйте сочетание клавиш Ctrl+N.

5. В новом окне запроса выполните приведенный ниже скрипт, чтобы создать задание потоковой передачи T-SQL. Прежде чем выполнять скрипт, необходимо изменить следующие переменные. 
   - *SQL_SA_Password:* Значение MSSQL_SA_PASSWORD, указанное при развертывании модуля SQL Azure для пограничных вычислений. 
   
   ```sql
   Use IronOreSilicaPrediction
   Go

   Declare @SQL_SA_Password varchar(200) = '<SQL_SA_Password>'
   declare @query varchar(max) 

   /*
   Create Objects Required for Streaming
   */

   CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MyStr0ng3stP@ssw0rd';

   If NOT Exists (select name from sys.external_file_formats where name = 'JSONFormat')
   Begin
      CREATE EXTERNAL FILE FORMAT [JSONFormat]  
      WITH ( FORMAT_TYPE = JSON)
   End 


   If NOT Exists (select name from sys.external_data_sources where name = 'EdgeHub')
   Begin
      Create EXTERNAL DATA SOURCE [EdgeHub] 
      With(
         LOCATION = N'edgehub://'
      )
   End 

   If NOT Exists (select name from sys.external_streams where name = 'IronOreInput')
   Begin
      CREATE EXTERNAL STREAM IronOreInput WITH 
      (
         DATA_SOURCE = EdgeHub,
         FILE_FORMAT = JSONFormat,
         LOCATION = N'IronOreMeasures'
       )
   End


   If NOT Exists (select name from sys.database_scoped_credentials where name = 'SQLCredential')
   Begin
       set @query = 'CREATE DATABASE SCOPED CREDENTIAL SQLCredential
                 WITH IDENTITY = ''sa'', SECRET = ''' + @SQL_SA_Password + ''''
       Execute(@query)
   End 

   If NOT Exists (select name from sys.external_data_sources where name = 'LocalSQLOutput')
   Begin
      CREATE EXTERNAL DATA SOURCE LocalSQLOutput WITH (
      LOCATION = 'sqlserver://tcp:.,1433',CREDENTIAL = SQLCredential)
   End

   If NOT Exists (select name from sys.external_streams where name = 'IronOreOutput')
   Begin
      CREATE EXTERNAL STREAM IronOreOutput WITH 
      (
         DATA_SOURCE = LocalSQLOutput,
         LOCATION = N'IronOreSilicaPrediction.dbo.IronOreMeasurements'
      )
   End

   EXEC sys.sp_create_streaming_job @name=N'IronOreData',
   @statement= N'Select * INTO IronOreOutput from IronOreInput'

   exec sys.sp_start_streaming_job @name=N'IronOreData'
   ```

6. Используйте следующий запрос, чтобы убедиться, что данные модуля создания данных передаются в базу данных. 

   ```sql
   Select Top 10 * from dbo.IronOreMeasurements
   order by timestamp desc
   ```


В этом руководстве мы развернули модуль генератора данных и модуль SQL Azure для пограничных вычислений. Затем мы создали задание потоковой передачи, чтобы передать данные, созданные модулем создания данных, в SQL. 

## <a name="next-steps"></a>Next Steps

- [Развертывание модели ML в SQL Azure для пограничных вычислений с помощью ONNX](tutorial-run-ml-model-on-sql-edge.md)
