---
title: Массовое копирование из базы данных с использованием контрольной таблицы
description: Сведения о том, как с помощью шаблона решения выполнить массовое копирование данных из базы данных, используя внешнюю контрольную таблицу для хранения списка секций исходных таблиц в Фабрике данных Azure.
author: dearandyxu
ms.author: yexu
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 12/09/2020
ms.openlocfilehash: eed7a304bdd57846cd038cc9bf9a67e8150ca505
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100392465"
---
# <a name="bulk-copy-from-a-database-with-a-control-table"></a>Массовое копирование из базы данных с использованием контрольной таблицы

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Чтобы скопировать данные из хранилища данных на сервере Oracle, Netezza, Teradata или SQL Server в Azure синапсе Analytics, необходимо загрузить огромные объемы данных из нескольких таблиц. Обычно данные должны быть разделены в каждой таблице, чтобы можно было загружать строки из одной таблицы с использованием нескольких параллельных потоков. В этой статье описывается шаблон для этих сценариев.

 >! Примечание. Если вы хотите скопировать данные из небольшого числа таблиц с относительно небольшим объемом данных в Azure синапсе Analytics, более эффективно использовать [средство копирование данных фабрики данных Azure](copy-data-tool.md). Шаблон, описанный в этой статье, рекомендуется для большого объема данных.

## <a name="about-this-solution-template"></a>Информация о шаблоне решения

Этот шаблон получает список секций базы данных-источника для копирования из внешней контрольной таблицы. Затем он выполняет итерацию по каждой секции в базе данных-источнике и копирует данные в место назначения.

Шаблон состоит из трех действий.
- Действие **Lookup** извлекает список всех секций базы данных из внешней контрольной таблицы.
- Действие **ForEach** получает список секций из действия Lookup и выполняет итерацию каждой секции для действия Copy.
- Действие **Copy** копирует каждую секцию из базы данных-источника в целевое хранилище.

Ниже описаны параметры, которые определяет шаблон:
- *Control_Table_Name* — это внешняя контрольная таблица, в которой хранится список секций для базы данных-источника.
- *Control_Table_Schema_PartitionID* — имя столбца внешней контрольной таблицы, в котором хранятся идентификаторы всех секций. Убедитесь, что идентификатор секции уникален для каждой секции в базе данных-источнике.
- *Control_Table_Schema_SourceTableName* — внешняя контрольная таблица, в которой хранятся имена всех таблиц базы данных-источника.
- *Control_Table_Schema_FilterQuery* — это имя столбца внешней контрольной таблицы, в котором хранятся запросы на фильтрацию. Они получают данные из каждой секции базы данных-источника. Например, если разделить данные по годам, запрос, хранимый в каждой строке, может быть похож на select * from datasource where LastModifytime >= ''2015-01-01 00:00:00'' and LastModifytime <= ''2015-12-31 23:59:59.999".
- *Data_Destination_Folder_Path* — это путь, по которому данные копируются в целевое хранилище (применимо, если выбрано значение "Файловая система" или "Azure Data Lake Storage 1-го поколения"). 
- *Data_Destination_Folder_Path* — это путь к корневой папке, в которой находится целевое хранилище, куда копируются данные. 
- *Data_Destination_Directory* — это путь к каталогу в корневой папке, в которой находится целевое хранилище, куда копируются данные. 

Последние три параметра, определяющие путь в целевом хранилище, отображаются только в том случае, если выбранное назначение является хранилищем на основе файлов. Если в качестве целевого хранилища выбрано "Azure синапсе Analytics", эти параметры не требуются. Но имена таблиц и схемы в Azure синапсе Analytics должны совпадать с именами в базе данных источника.

## <a name="how-to-use-this-solution-template"></a>Использование шаблона решения

1. Создайте контрольную таблицу в SQL Server или базе данных SQL Azure, чтобы хранить там список секций базы данных-источника для массового копирования. В следующем примере вы видите пять секций в базе данных-источнике. Три секции предназначены для *datasource_table*, а две — для *project_table*. Столбец *LastModifytime* используется для разделения данных в таблице *datasource_table* из базы данных-источника. Запрос, который считывает первую секцию, выглядит следующим образом: select * from datasource_table where LastModifytime >= ''2015-01-01 00:00:00'' and LastModifytime <= ''2015-12-31 23:59:59.999". Вы можете использовать аналогичный запрос для считывания данных из других секций.

     ```sql
            Create table ControlTableForTemplate
            (
            PartitionID int,
            SourceTableName  varchar(255),
            FilterQuery varchar(255)
            );

            INSERT INTO ControlTableForTemplate
            (PartitionID, SourceTableName, FilterQuery)
            VALUES
            (1, 'datasource_table','select * from datasource_table where LastModifytime >= ''2015-01-01 00:00:00'' and LastModifytime <= ''2015-12-31 23:59:59.999'''),
            (2, 'datasource_table','select * from datasource_table where LastModifytime >= ''2016-01-01 00:00:00'' and LastModifytime <= ''2016-12-31 23:59:59.999'''),
            (3, 'datasource_table','select * from datasource_table where LastModifytime >= ''2017-01-01 00:00:00'' and LastModifytime <= ''2017-12-31 23:59:59.999'''),
            (4, 'project_table','select * from project_table where ID >= 0 and ID < 1000'),
            (5, 'project_table','select * from project_table where ID >= 1000 and ID < 2000');
    ```

2. Перейдите к шаблону **Массовое копирование из базы данных**. Выберите **Создать**, чтобы создать подключение к внешней контрольной таблице, созданной на шаге 1.

    ![Создание подключения к контрольной таблице](media/solution-template-bulk-copy-with-control-table/BulkCopyfromDB_with_ControlTable2.png)

3. Выберите **Создать**, чтобы создать подключение к базе данных-источнику, из которой вы копируете данные.

    ![Создание подключения к базе данных-источнику](media/solution-template-bulk-copy-with-control-table/BulkCopyfromDB_with_ControlTable3.png)
    
4. Выберите **Создать**, чтобы создать подключение к целевой базе данных, в которую вы копируете данные.

    ![Создание подключения к целевому хранилищу](media/solution-template-bulk-copy-with-control-table/BulkCopyfromDB_with_ControlTable4.png)

5. Щелкните **Использовать этот шаблон**.

6. Откроется конвейер, как показано в следующем примере:

    ![Проверка конвейера](media/solution-template-bulk-copy-with-control-table/BulkCopyfromDB_with_ControlTable6.png)

7. Выберите **Отладка**, введите **Параметры**, а затем нажмите **Готово**.

    ![Щелкните **Отладка**](media/solution-template-bulk-copy-with-control-table/BulkCopyfromDB_with_ControlTable7.png)

8. Вы увидите результат, аналогичный приведенному ниже:

    ![Просмотр результатов](media/solution-template-bulk-copy-with-control-table/BulkCopyfromDB_with_ControlTable8.png)

9. Используемых Если в качестве назначения данных выбрано "Azure синапсе Analytics", необходимо ввести подключение к хранилищу BLOB-объектов Azure для промежуточного хранения в соответствии с требованиями Azure синапсе Analytics polybase. Шаблон автоматически создаст путь к контейнеру для хранилища BLOB-объектов. Проверьте, создан ли контейнер, после выполнения конвейера.
    
    ![Настройка Polybase](media/solution-template-bulk-copy-with-control-table/BulkCopyfromDB_with_ControlTable9.png)
       
## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о службе фабрики данных Azure, службе интеграции данных в облаке](introduction.md)
