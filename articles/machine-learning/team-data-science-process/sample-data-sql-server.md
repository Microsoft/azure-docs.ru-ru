---
title: Выборка данных на сервере SQL Server в Azure — командный процесс обработки и анализа данных
description: Выборка данных, хранящихся на сервере SQL Server в Azure с помощью SQL или языка программирования Python и последующее их перемещение в машинное обучение Azure.
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
ms.openlocfilehash: 7ac1fc5688dad3406041f36ff858e6fd27c7272f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93321869"
---
# <a name="sample-data-in-sql-server-on-azure"></a><a name="heading"></a>Выборка данных на сервере SQL Server в Azure

В этой статье описывается процесс выборки данных, хранящихся на сервере SQL Server в Azure, с помощью SQL или языка программирования Python. В нем также показано, как переместить данные выборки в службу машинного обучения Azure, сохранив их в файл, передав его в BLOB-объект Azure, а затем считав в студии машинного обучения Azure.

Процедура выборки Python использует библиотеку [pyodbc](https://code.google.com/p/pyodbc/) ODBC для подключения к SQL Server в Azure и библиотеку [Pandas](https://pandas.pydata.org/) для выполнения выборки.

> [!NOTE]
> Образец кода SQL в этом документе предполагает, что данные содержатся на сервере SQL Server на платформе Azure. Если это не так, воспользуйтесь инструкциями по переносу данных на сервер SQL Server в среде Azure, изложенными в статье [Перемещение данных в SQL Server на виртуальной машине Azure](move-sql-server-virtual-machine.md).
> 
> 

**Для чего нужна выборка данных?**
Если размер набора данных, который планируется проанализировать, слишком большой, обычно рекомендуется уменьшить выборку данных до размера, который останется репрезентативным и будет более управляемым. Выборка упрощает понимание данных, исследование и проектирование признаков. Роль этой операции в [процессе обработки и анализа данных группы (TDSP)](./index.yml) состоит в том, чтобы сделать возможным быстрое прототипирование функций обработки данных и моделей машинного обучения.

Эта задача выборки является одним из этапов [процесса обработки и анализа данных группы (TDSP)](./index.yml).

## <a name="using-sql"></a><a name="SQL"></a>Использование SQL
В этом разделе описываются несколько методов использования SQL для выполнения простой случайной выборки из данных, содержащихся в базе данных. Выберите нужный метод в зависимости от размера ваших данных и их распределения.

Следующие два элемента показывают, как использовать `newid` в SQL Server для выполнения выборки. Выбор метода зависит от того, насколько случайным должен быть образец (pk_id в следующем образце кода считается автоматически созданным первичным ключом).

1. Менее строгая случайная выборка

    ```sql
    select  * from <table_name> where <primary_key> in 
    (select top 10 percent <primary_key> from <table_name> order by newid())
    ```

2. Более случайная выборка 

    ```sql
    SELECT * FROM <table_name>
    WHERE 0.1 >= CAST(CHECKSUM(NEWID(), <primary_key>) & 0x7fffffff AS float)/ CAST (0x7fffffff AS int)
    ```

Для выборки данных можно также использовать предложение TABLESAMPLE. Этот вариант может быть лучшим подходом, если размер данных велик (предполагая, что данные на разных страницах не сопоставлены) и для выполнения запроса в разумное время.

```sql
SELECT *
FROM <table_name> 
TABLESAMPLE (10 PERCENT)
```

> [!NOTE]
> Можно просматривать данные этой выборки и создавать характеристики, сохранив ее в новой таблице
> 
> 

### <a name="connecting-to-azure-machine-learning"></a><a name="sql-aml"></a>Подключение к службе машинного обучения Azure
Приведенные выше примеры запросов можно использовать непосредственно в модуле [Импорт данных][import-data] Машинного обучения Azure для оперативного сокращения выборки данных и их передачи в эксперимент машинного обучения Azure. Ниже показан снимок экрана с использованием модуля чтения для чтения выборки данных.

![считыватель sql][1]

## <a name="using-the-python-programming-language"></a><a name="python"></a>Использование языка программирования Python
В этом разделе демонстрируется использование [библиотеки pyodbc](https://code.google.com/p/pyodbc/) для установки подключения ODBC к Базе данных SQL Server на языке Python. Строка подключения к базе данных выглядит следующим образом: (замените ServerName, dbname, username и Password своей конфигурацией):

```python
#Set up the SQL Azure connection
import pyodbc    
conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')
```

Библиотека [Pandas](https://pandas.pydata.org/) в языке Python предоставляет широкий набор структур данных и средств анализа данных для манипуляций с данными при программировании на языке Python. Следующий код считывает образец 0,1% данных из таблицы в базе данных SQL Azure в данные Pandas:

```python
import pandas as pd

# Query database and load the returned results in pandas data frame
data_frame = pd.read_sql('''select column1, column2... from <table_name> tablesample (0.1 percent)''', conn)
```

Теперь можно работать с данными выборки во фрейме данных Pandas. 

### <a name="connecting-to-azure-machine-learning"></a><a name="python-aml"></a>Подключение к службе машинного обучения Azure
С помощью следующего образца кода можно сохранить данные уменьшенной выборки в файл и передать его в BLOB-объект Azure. Данные из большого двоичного объекта можно считать непосредственно в эксперимент машинного обучения Azure с помощью модуля [Импорт данных][import-data]. Для этого необходимо выполнить следующие шаги: 

1. Запись фрейма данных pandas в локальный файл

    ```python
    dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)
    ```

2. Передача локального файла в BLOB-объект Azure

    ```python
    from azure.storage import BlobService
    import tables

    STORAGEACCOUNTNAME= <storage_account_name>
    LOCALFILENAME= <local_file_name>
    STORAGEACCOUNTKEY= <storage_account_key>
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>

    output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
    localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory

    try:

    #perform upload
    output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)

    except:            
        print ("Something went wrong with uploading blob:"+BLOBNAME)
    ```

3. Считать данные из BLOB-объекта Azure с помощью модуля [Импорт данных][import-data] службы Машинного обучения Azure, как показано на снимке экрана ниже.

![большой двоичный объект считывателя][2]

## <a name="the-team-data-science-process-in-action-example"></a>Пример применения процесса обработки и анализа данных группы на практике
Пример процесса обработки и анализа данных группы с использованием общедоступного набора данных см. в разделе [процесс обработки и анализа данных группы в действии: использование SQL Server](sql-walkthrough.md).

[1]: ./media/sample-sql-server-virtual-machine/reader_database.png
[2]: ./media/sample-sql-server-virtual-machine/reader_blob.png

[import-data]: /azure/machine-learning/studio-module-reference/import-data