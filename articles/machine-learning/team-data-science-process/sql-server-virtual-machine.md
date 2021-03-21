---
title: Изучение данных на виртуальной машине сервера SQL Server — Процесс обработки и анализа данных группы
description: Изучите и создайте данные и создавайте функции с помощью Python или SQL на SQL Server виртуальной машине в Azure.
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
ms.openlocfilehash: 8be878cf40967356d68e9be0765e898c81b5ba0a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93314620"
---
# <a name="process-data-in-sql-server-virtual-machine-on-azure"></a><a name="heading"></a>Обработка данных в виртуальной машине SQL Server на платформе Azure
В этом документе описывается изучение данных и создание характеристик для данных, хранящихся в виртуальной машине SQL Server в Azure. Эта цель может быть выполнена структурирование данных с помощью SQL или с помощью языка программирования, такого как Python.

> [!NOTE]
> В образцах инструкций SQL, содержащихся в данном документе, предполагается, что данные находятся на сервере SQL Server. Если это не так, обратитесь к карте обработки облачных данных, чтобы узнать, как переместить данные в SQL Server.
> 
> 

## <a name="using-sql"></a><a name="SQL"></a>Использование SQL
В данном разделе с помощью SQL описываются следующие задачи по структурированию данных:

1. [Исследование данных](#sql-dataexploration)
2. [Создание компонентов](#sql-featuregen)

### <a name="data-exploration"></a><a name="sql-dataexploration"></a>Исследование данных
Вот несколько примеров сценариев SQL, которые можно использовать для изучения хранилищ данных в SQL Server.

> [!NOTE]
> Для практического примера можно использовать [набор данных о такси Нью-Йорка](https://www.andresmh.com/nyctaxitrips/) и статью IPNB под названием [Структурирование данных Нью-Йорка с помощью IPython Notebook и SQL Server](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-sql-walkthrough.ipynb), содержащую полное пошаговое руководство.
> 
> 

1. Получение количества наблюдений за день
   
    `SELECT CONVERT(date, <date_columnname>) as date, count(*) as c from <tablename> group by CONVERT(date, <date_columnname>)` 
2. Получение уровней в столбце категорий 
   
    `select  distinct <column_name> from <databasename>`
3. Получение числа уровней в сочетании двух столбцов категорий 
   
    `select <column_a>, <column_b>,count(*) from <tablename> group by <column_a>, <column_b>`
4. Получение распределения для числовых столбцов
   
    `select <column_name>, count(*) from <tablename> group by <column_name>`

### <a name="feature-generation"></a><a name="sql-featuregen"></a>Создание компонентов
В этом разделе мы опишем способы создания характеристик с помощью SQL:  

1. [Создание компонентов на основе количества](#sql-countfeature)
2. [Создание характеристик путем группирования данных](#sql-binningfeature)
3. [Развертывание характеристик из одного столбца](#sql-featurerollout)

> [!NOTE]
> После создания дополнительных компонентов можно либо добавить их в виде столбцов в существующую таблицу, либо создать новую таблицу с дополнительными компонентами и первичным ключом, которую можно будет объединить с исходной таблицей. 
> 
> 

### <a name="count-based-feature-generation"></a><a name="sql-countfeature"></a>Создание компонентов на основе количества
В следующих примерах показаны два способа создания количественных характеристик. В первом способе используется условная сумма, а во втором — предложение where. Затем эти результаты можно объединить с исходной таблицей (с использованием столбцов первичного ключа), чтобы они были подсчета вместе с исходными данными.

```sql
select <column_name1>,<column_name2>,<column_name3>, COUNT(*) as Count_Features from <tablename> group by <column_name1>,<column_name2>,<column_name3> 

select <column_name1>,<column_name2> , sum(1) as Count_Features from <tablename> 
where <column_name3> = '<some_value>' group by <column_name1>,<column_name2> 
```

### <a name="binning-feature-generation"></a><a name="sql-binningfeature"></a>Создание характеристик путем группирования данных
В следующем примере показано, как создать группированные характеристики, сгруппировав (с использованием пяти ячеек) числовой столбец, который взамен можно использовать как характеристику:

```sql
SELECT <column_name>, NTILE(5) OVER (ORDER BY <column_name>) AS BinNumber from <tablename>
```

### <a name="rolling-out-the-features-from-a-single-column"></a><a name="sql-featurerollout"></a>Развертывание характеристик из одного столбца
В этом разделе мы покажем, как развернуть одиночный столбец в таблице для создания дополнительных характеристик. В примере предполагается, что в таблице, из которой вы намерены создать характеристики, содержится столбец широты или долготы.

Вот краткое руководство по данным широты/долготы расположения (на основе ресурса stackoverflow): [Как измерить точность широты и долготы](https://gis.stackexchange.com/questions/8650/how-to-measure-the-accuracy-of-latitude-and-longitude). Это руководство полезно для понимания, прежде чем включать расположение как одну или несколько функций:

* Знак сообщает, в каком полушарии мы находимся: северном или южном, восточном или западном.
* Ненулевая цифра сотен указывает, что используется долгота, а не широта!
* Цифра десятков сообщает положение с точностью примерно 1000 километров. Она указывает, на каком континенте или в каком океане мы находимся.
* Цифра единиц (один десятичный градус) дает положение с точностью до 111 километров (60 морских миль, или около 69 миль). Он может сообщить вам о том, какое состояние, страна или регион вы используете.
* Первый десятичный разряд соответствует точности до 11,1 км: он позволяет различать положение одного крупного города относительно соседнего крупного города.
* Второй десятичный разряд соответствует точности до 1,1 км: он позволяет различать мелкие населенные пункты.
* Третий десятичный разряд соответствует точности до 110 м: он позволяет идентифицировать крупное сельскохозяйственное поле или кампус учреждения.
* Четвертый десятичный разряд соответствует точности до 11 м: он позволяет идентифицировать участок земли. Это сравнимо с типичной точностью устройства GPS без коррекции при отсутствии помех.
* Пятый десятичный разряд соответствует точности до 1,1 м: он позволяет отличать деревья друг от друга. Точность этого уровня с помощью коммерческих GPS-устройств может быть достигнута только при использовании дифференциальной коррекции.
* Шестой десятичный разряд соответствует точности до 0,11 м: это значение можно использовать для подробной детализации сооружений, для проектирования ландшафтов, строительства дорог. Этого уровня более чем достаточно для отслеживания движения ледников и рек. Его можно достичь с помощью тщательного измерения с использованием GPS, например при дифференциальной коррекции GPS.

Сведения о местоположении можно характеризовать следующим образом, выделяя информацию о регионе, местоположении и городе. Кроме того, можно вызвать конечную точку RESTFUL, например API Карт Bing, доступной на странице [Поиск расположения по точке](/bingmaps/rest-services/locations/find-a-location-by-point) , чтобы получить сведения о регионе или районе.

```sql
select 
    <location_columnname>
    ,round(<location_columnname>,0) as l1        
    ,l2=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 1 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),1,1) else '0' end     
    ,l3=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 2 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),2,1) else '0' end     
    ,l4=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 3 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),3,1) else '0' end     
    ,l5=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 4 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),4,1) else '0' end     
    ,l6=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 5 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),5,1) else '0' end     
    ,l7=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 6 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),6,1) else '0' end     
from <tablename>
```

Характеристики на основе расположения можно в дальнейшем использовать для создания дополнительных количественных характеристик, как было описано ранее. 

> [!TIP]
> Можно программным путем вставлять записи с использованием выбранного языка. Для повышения эффективности записи данные можно вставлять блоками — пример такого решения с использованием pyodbc см. в статье [Пример HelloWorld с обращением к SQL Server с помощью Python](https://code.google.com/p/pypyodbc/wiki/A_HelloWorld_sample_to_access_mssql_with_python). Другой вариант — вставить данные в базу данных с использованием [служебной программы BCP](/sql/tools/bcp-utility).
> 
> 

### <a name="connecting-to-azure-machine-learning"></a><a name="sql-aml"></a>Подключение к службе машинного обучения Azure
Новую созданную характеристику можно добавить в виде столбца в существующую таблицу или сохранить в новой таблице и объединить с существующей таблицей для машинного обучения. Вы можете создать характеристики и открыть доступ к ним, если они уже созданы, с помощью модуля [Импорт данных][import-data] в Машинном обучении Azure, как показано ниже.

![считыватели azureml][1] 

## <a name="using-a-programming-language-like-python"></a><a name="python"></a>Использование языка программирования, например Python
Использование языка Python для просмотра данных и создания характеристик, когда данные находятся в SQL Server, подобно обработке данных в большом двоичном объекте Azure с использованием Python, как описано в статье [Обработка больших двоичных данных Azure с применением методов расширенного анализа](data-blob.md). Загрузите данные из базы данных в кадр данных Pandas для дополнительной обработки. В этом разделе документирован процесс подключения к базе данных и загрузки данных во фрейм данных.

Для подключения к базе данных SQL Server из языка Python с использованием pyodbc можно применить следующий формат строки подключения (замените servername, dbname, username и password соответствующими значениями имени сервера, имени БД, имени пользователя и пароля):

```python
#Set up the SQL Azure connection
import pyodbc    
conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')
```

[Библиотека Pandas](https://pandas.pydata.org/) в Python предоставляет обширный набор структур данных и средств анализа данных для манипулирования данными при программировании на Python. Приведенный ниже код считывает результаты, возвращенные из базы данных SQL Server, во фрейм данных Pandas:

```python
# Query database and load the returned results in pandas data frame
data_frame = pd.read_sql('''select <columnname1>, <columnname2>... from <tablename>''', conn)
```

Теперь можно работать с фреймом данных Pandas, как описано в статье [Обработка больших двоичных данных Azure с применением методов расширенного анализа](data-blob.md).

## <a name="azure-data-science-in-action-example"></a>Пример применения обработки данных в Azure на практике
Пример комплексного пошагового руководства по обработке и анализу данных в Azure с использованием общедоступного набора данных см. в статье [Процесс обработки и анализа данных группы на практике: использование SQL Server](sql-walkthrough.md).

[1]: ./media/sql-server-virtual-machine/reader_db_featurizedinput.png


<!-- Module References -->
[import-data]: /azure/machine-learning/studio-module-reference/import-data