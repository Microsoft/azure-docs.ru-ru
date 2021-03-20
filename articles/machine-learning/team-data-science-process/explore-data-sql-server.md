---
title: Изучение данных на виртуальной машине сервера SQL Server — командный процесс обработки и анализа данных
description: Сведения о том, как просматривать данные, хранящиеся на виртуальной машине SQL Server в Azure, с помощью SQL или языка программирования, например Python.
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
ms.openlocfilehash: 33b55afb7796b197f7130ec9288abb01cc115651
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86085656"
---
# <a name="explore-data-in-sql-server-virtual-machine-on-azure"></a>Просмотр данных в виртуальной машине SQL Server на платформе Azure

В этой статье описывается, как просматривать данные, хранящиеся в виртуальной машине SQL Server на платформе Azure. Используйте SQL или Python для проверки данных.

Эта задача является одним из этапов [процесса обработки и анализа данных группы](overview.md).

> [!NOTE]
> В образцах инструкций SQL, содержащихся в данном документе, предполагается, что данные находятся на сервере SQL Server. Если это не так, обратитесь к карте обработки облачных данных, чтобы узнать, как переместить данные в SQL Server.
> 
> 

## <a name="explore-sql-data-with-sql-scripts"></a><a name="sql-dataexploration"></a>Просмотр данных SQL с помощью сценариев SQL
Вот несколько примеров сценариев SQL, которые можно использовать для изучения хранилищ данных в SQL Server.

1. Получение количества наблюдений за день
   
    `SELECT CONVERT(date, <date_columnname>) as date, count(*) as c from <tablename> group by CONVERT(date, <date_columnname>)` 
2. Получение уровней в столбце категорий 
   
    `select  distinct <column_name> from <databasename>`
3. Получение числа уровней в сочетании двух столбцов категорий 
   
    `select <column_a>, <column_b>,count(*) from <tablename> group by <column_a>, <column_b>`
4. Получение распределения для числовых столбцов
   
    `select <column_name>, count(*) from <tablename> group by <column_name>`

> [!NOTE]
> Для практического примера можно использовать [набор данных о такси Нью-Йорка](https://www.andresmh.com/nyctaxitrips/) и статью IPNB под названием [Структурирование данных Нью-Йорка с помощью IPython Notebook и SQL Server](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-sql-walkthrough.ipynb), содержащую полное пошаговое руководство.
> 
> 

## <a name="explore-sql-data-with-python"></a><a name="python"></a>Просмотр данных SQL с помощью Python
Использование Python для просмотра данных и создания функций при работе с данными в SQL Server аналогично обработке данных в большом двоичном объекте Azure с помощью Python, как описано в статье [обработка данных BLOB-объектов Azure в среде обработки и анализа данных](data-blob.md). Загрузить данные из базы данных в Pandas таблицу данных, а затем обработать их дальше. В этом разделе задокументирован процесс подключения к базе данных и загрузки данных в кадр данных.

Для подключения к базе данных SQL Server из языка Python с использованием pyodbc можно применить следующий формат строки подключения (замените servername, dbname, username и password соответствующими значениями имени сервера, имени БД, имени пользователя и пароля):

```python
#Set up the SQL Azure connection
import pyodbc    
conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')
```

[Библиотека Pandas](https://pandas.pydata.org/) в Python предоставляет обширный набор структур данных и средств анализа данных для манипулирования данными при программировании на Python. Следующий код считывает результаты, возвращенные из базы данных SQL Server, в кадр данных Pandas:

```python
# Query database and load the returned results in pandas data frame
data_frame = pd.read_sql('''select <columnname1>, <columnname2>... from <tablename>''', conn)
```

Теперь можно работать с кадром данных Pandas, как описано в статье [Обработка больших двоичных данных Azure с применением методов расширенного анализа](data-blob.md).

## <a name="the-team-data-science-process-in-action-example"></a>Пример применения командного процесса обработки и анализа данных на практике
Полноценный пошаговый пример применения процесса Cortana Analytics с использованием общедоступного набора данных см. в статье [Процесс обработки и анализа данных группы на практике: использование SQL Server](sql-walkthrough.md).

