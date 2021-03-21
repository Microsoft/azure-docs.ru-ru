---
title: Обработка больших двоичных данных Azure с применением методов расширенного анализа. Процесс обработки и анализа данных группы
description: Работайте с данными в хранилище BLOB-объектов Azure и создавайте признаки на их основе с помощью углубленной аналитики.
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
ms.openlocfilehash: 5a088d5918a957036b905db9136f9b16e5f0527e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93307156"
---
# <a name="process-azure-blob-data-with-advanced-analytics"></a><a name="heading"></a>Обработка данных больших двоичных объектов Azure с применением методов расширенного анализа
Этот документ содержит сведения о работе с данными в хранилище больших двоичных объектов Azure и создании характеристик на их основе. 

## <a name="load-the-data-into-a-pandas-data-frame"></a>Загрузка данных во фрейм данных Pandas
Чтобы исследовать и манипулировать набором данных, его необходимо загрузить из источника BLOB-объектов в локальный файл, который затем можно загрузить в кадр данных Pandas. Ниже приведен порядок выполнения данной процедуры.

1. Скачайте данные из большого двоичного объекта Azure с помощью следующего примера кода Python, используя службу BLOB-объектов. Замените переменные этого кода своими значениями. 
   
    ```python
    from azure.storage.blob import BlobService
    import tables
   
    STORAGEACCOUNTNAME= <storage_account_name>
    STORAGEACCOUNTKEY= <storage_account_key>
    LOCALFILENAME= <local_file_name>        
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>
   
    #download from blob
    t1=time.time()
    blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
    blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILENAME)
    t2=time.time()
    print(("It takes %s seconds to download "+blobname) % (t2 - t1))
    ```
2. Прочитайте данные, которые содержит скачанный файл, в блоке данных Pandas.
   
    ```python
    #LOCALFILE is the file path    
    dataframe_blobdata = pd.read_csv(LOCALFILE)
    ```

Теперь вы готовы просматривать эти данные и создавать функции на основе этого набора данных.

## <a name="data-exploration"></a><a name="blob-dataexploration"></a>Исследование данных
Вот несколько примеров того, как можно просматривать данные с помощью Pandas:

1. Проверьте количество строк и столбцов. 
   
    ```python
    print 'the size of the data is: %d rows and  %d columns' % dataframe_blobdata.shape
    ```
2. Проверьте первые или последние несколько строк в наборе данных так, как показано ниже.
   
    ```python
    dataframe_blobdata.head(10)
   
    dataframe_blobdata.tail(10)
    ```
3. Проверьте тип данных, импортированный в каждый столбец, с помощью приведенного ниже образца кода.
   
    ```python
    for col in dataframe_blobdata.columns:
        print dataframe_blobdata[col].name, ':\t', dataframe_blobdata[col].dtype
    ```
4. Проверьте основную статистику, относящуюся к столбцам набора данных, так, как показано ниже.
   
    ```python
    dataframe_blobdata.describe()
    ```
5. Проверьте количество записей в каждом столбце так, как показано ниже.
   
    ```python
    dataframe_blobdata['<column_name>'].value_counts()
    ```
6. Проверьте соотношение числа отсутствующих значений к фактическому количеству записей в каждом столбце. Воспользуйтесь приведенным ниже образцом кода.
   
    ```python
    miss_num = dataframe_blobdata.shape[0] - dataframe_blobdata.count()
    print miss_num
    ```
7. Если в том или ином столбце данных отсутствуют значения, вы можете заменить их так, как показано ниже.
   
    ```python
    dataframe_blobdata_noNA = dataframe_blobdata.dropna()
    dataframe_blobdata_noNA.shape
    ```
   
   Другой способ заменить отсутствующие значения — воспользоваться функцией режима.
   
    ```python
    dataframe_blobdata_mode = dataframe_blobdata.fillna({'<column_name>':dataframe_blobdata['<column_name>'].mode()[0]})  
    ```      
8. Создайте гистограмму, используя переменное количество ячеек, чтобы построить распределение переменной.    
   
    ```python
    dataframe_blobdata['<column_name>'].value_counts().plot(kind='bar')
   
    np.log(dataframe_blobdata['<column_name>']+1).hist(bins=50)
    ```
9. Проверьте корреляции между переменными, используя диаграмму рассеяния или встроенную функцию корреляции.
   
    ```python
    #relationship between column_a and column_b using scatter plot
    plt.scatter(dataframe_blobdata['<column_a>'], dataframe_blobdata['<column_b>'])
   
    #correlation between column_a and column_b
    dataframe_blobdata[['<column_a>', '<column_b>']].corr()
    ```

## <a name="feature-generation"></a><a name="blob-featuregen"></a>Создание компонентов
Создавать функции с помощью Python вы можете приведенным ниже способом.

### <a name="indicator-value-based-feature-generation"></a><a name="blob-countfeature"></a>Создание признаков со значениями индикатора
Вот как можно создавать категориальные функции:

1. Проверьте распределение категориального столбца.
   
    ```python
    dataframe_blobdata['<categorical_column>'].value_counts()
    ```
2. Создайте значения индикатора для каждого значения столбца.
   
    ```python
    #generate the indicator column
    dataframe_blobdata_identity = pd.get_dummies(dataframe_blobdata['<categorical_column>'], prefix='<categorical_column>_identity')
    ```
3. Объедините столбец индикатора с исходным блоком данных. 
   
    ```python
    #Join the dummy variables back to the original data frame
    dataframe_blobdata_with_identity = dataframe_blobdata.join(dataframe_blobdata_identity)
    ```
4. Удалите исходную переменную.
   
    ```python
    #Remove the original column rate_code in df1_with_dummy
    dataframe_blobdata_with_identity.drop('<categorical_column>', axis=1, inplace=True)
    ```

### <a name="binning-feature-generation"></a><a name="blob-binningfeature"></a>Создание характеристик путем группирования данных
Вот как можно создавать функции группирования:

1. Добавьте последовательность столбцов, чтобы создать числовой столбец.
   
    ```python
    bins = [0, 1, 2, 4, 10, 40]
    dataframe_blobdata_bin_id = pd.cut(dataframe_blobdata['<numeric_column>'], bins)
    ```
2. Преобразуйте группирование в последовательность логических переменных.
   
    ```python
    dataframe_blobdata_bin_bool = pd.get_dummies(dataframe_blobdata_bin_id, prefix='<numeric_column>')
    ```
3. Наконец, объедините фиктивные переменные с исходным блоком данных.
   
    ```python
    dataframe_blobdata_with_bin_bool = dataframe_blobdata.join(dataframe_blobdata_bin_bool)  
    ```  

## <a name="writing-data-back-to-azure-blob-and-consuming-in-azure-machine-learning"></a><a name="sql-featuregen"></a>Запись данных обратно в большой двоичный объект Azure и их использование в Студии машинного обучения Azure
После изучения данных и создания необходимых компонентов можно передать данные (Samples или признаками) в большой двоичный объект Azure и использовать их в Машинное обучение Azure выполните следующие действия. Дополнительные компоненты можно создать в Машинное обучение Azure Studio (классическая модель). 

1. Запишите блок данных в локальный файл.
   
    ```python
    dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)
    ```
2. Отправьте данные в большой двоичный объект Azure так, как указано ниже.
   
    ```python
    from azure.storage.blob import BlobService
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
3. Теперь данные можно считывать из большого двоичного объекта с помощью модуля [Импорт данных][import-data] машинного обучения Azure (см. рисунок ниже).

![большой двоичный объект считывателя][1]

[1]: ./media/data-blob/reader_blob.png


<!-- Module References -->
[import-data]: /azure/machine-learning/studio-module-reference/import-data