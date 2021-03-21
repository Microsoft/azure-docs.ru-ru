---
title: Устранение ошибок модуля конструктора
titleSuffix: Azure Machine Learning
description: Узнайте, как читать и устранять неполадки в кодах ошибок автоматических модулей в Машинное обучение Azure Designer.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.custom: troubleshooting
author: likebupt
ms.author: keli19
ms.date: 11/25/2020
ms.openlocfilehash: b917e3fc93c59de85c5236c18e31d7bbc9d891f0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98065479"
---
# <a name="exceptions-and-error-codes-for-the-designer"></a>Исключения и коды ошибок для конструктора

В этой статье описываются сообщения об ошибках и коды исключений в Машинное обучение Azure Designer, помогающие устранять неполадки в конвейерах машинного обучения.

Сообщение об ошибке можно найти в конструкторе, выполнив следующие действия.  

- Выберите неисправный модуль, перейдите на вкладку **Outputs+logs** (Выходные данные и журналы). Подробный журнал можно найти в файле **70_driver_log.txt** в категории **azureml-logs**.

- Чтобы получить подробные сведения об ошибке модуля, его можно проверить в файле error_info.json в категории **module_statistics**.

Ниже приведены коды ошибок модулей в конструкторе.

## <a name="error-0001"></a>Ошибка 0001  
 Исключение возникает, если не удалось найти один или несколько столбцов указанного набора данных.  

 Эта ошибка возникает при выборе столбца для модуля, если выбранные столбцы не существуют во входном наборе данных. Эта ошибка может возникать, если вы вручную ввели имя столбца или если селектор столбцов предоставил предлагаемый столбец, который отсутствовал в наборе данных при выполнении конвейера.  

**Решение.** Повторно откройте модуль, в котором возникает это исключение, и убедитесь, что имя столбца или имена верны и все упомянутые столбцы действительно существуют.  

|Сообщения об исключении|
|------------------------|
|One or more specified columns were not found. (Не удалось найти один или несколько указанных столбцов).|
|Column with name or index "{column_id}" not found (Столбец с именем или индексом {column_id} не найден).|
|Column with name or index "{column_id}" does not exist in "{arg_name_missing_column}" (Столбец с именем или индексом {column_id} не существует в {arg_name_missing_column}).|
|Column with name or index "{column_id}" does not exist in "{arg_name_missing_column}", but exists in "{arg_name_has_column}" (Столбец с именем или индексом {column_id} не существует {arg_name_missing_column}, но существует в {arg_name_has_column}).|
|Columns with name or index "{column_names}" not found (Столбцы с именем или индексом {column_names} не найдены).|
|Columns with name or index "{column_names}" does not exist in "{arg_name_missing_column}" (Столбцы с именем или индексом {column_names} не существуют в {arg_name_missing_column}).|
|Columns with name or index "{column_names}" does not exist in "{arg_name_missing_column}", but exists in "{arg_name_has_column}" (Столбцы с именем или индексом {column_names} не существуют {arg_name_missing_column}, но существуют в {arg_name_has_column}).|


## <a name="error-0002"></a>Ошибка 0002  
 Исключение возникает, если один или несколько параметров не удалось проанализировать или преобразовать из заданного типа в необходимый для целевого метода тип.  

 Эта ошибка возникает в Машинном обучении Azure, когда вы указываете параметр в качестве входных данных, а тип значения отличается от ожидаемого и не удается выполнить неявное преобразование.  

**Решение.** Проверьте требования к модулю и определите требуемый тип значения (string, integer, double и т. д.).  

|Сообщения об исключении|
|------------------------|
|Failed to parse parameter (Не удалось выполнить синтаксический анализ параметра).|
|Failed to parse "{arg_name_or_column}" parameter (Не удалось выполнить синтаксический анализ параметра {arg_name_or_column}).|
|Failed to convert "{arg_name_or_column}" parameter to "{to_type}" (Не удалось преобразовать параметр {arg_name_or_column} в параметр {to_type}).|
|Failed to convert "{arg_name_or_column}" parameter from "{from_type}" to "{to_type}" (Не удалось преобразовать параметр {arg_name_or_column} из параметра {from_type} в {to_type}).|
|Failed to convert "{arg_name_or_column}" parameter value "{arg_value}" from "{from_type}" to "{to_type}" (Не удалось преобразовать значение {arg_value} параметра {arg_name_or_column} из {from_type} в {to_type}).|
|Failed to convert value "{arg_value}" in column "{arg_name_or_column}" from "{from_type}" to "{to_type}" with usage of the format "{fmt}" provided (Не удалось преобразовать значение {arg_value} в столбце {arg_name_or_column} из {from_type} в {to_type} с использованием указанного формата {fmt}).|


## <a name="error-0003"></a>Ошибка 0003  
 Исключение возникает, если один или несколько входных аргументов имеют значение NULL или пусты.  

 Эта ошибка возникает в Машинном обучении Azure, если какие-либо входные данные или параметры модуля пусты или имеют значение NULL.  Эта ошибка может возникать, например, если не введено какое-либо значение для параметра. Это также может произойти, если выбрать набор данных, содержащий отсутствующие значения или пустой набор данных.  

**Решение.**

+ Откройте модуль, в котором возникло исключение, и убедитесь, что указаны все входные данные. Убедитесь, что указаны все необходимые входные данные. 
+ Убедитесь, что данные, загружаемые из хранилища Azure, доступны, а имя или ключ учетной записи не изменились.  
+ Проверьте входные данные на наличие отсутствующих значений или значений NULL.
+ При использовании запроса к источнику данных убедитесь, что данные возвращаются в нужном формате. 
+ Проверьте наличие опечаток или других изменений в спецификации данных.
  
|Сообщения об исключении|
|------------------------|
|One or more of inputs are null or empty (Один или несколько входных данных имеют значение NULL или пусты).|
|Input "{name}" is null or empty (Входной параметр {name} имеет значение NULL или пустой).|


## <a name="error-0004"></a>Ошибка 0004  
 Исключение возникает, если параметр меньше или равен определенному значению.  

 Эта ошибка возникает в Машинном обучении Azure, если параметр в сообщении имеет значение ниже граничного, необходимого модулю для обработки данных.  

**Решение.** Повторно откройте модуль, вызывающий исключение, и измените значение параметра, чтобы оно было больше указанного значения.  

|Сообщения об исключении|
|------------------------|
|Parameter should be greater than boundary value (Значение параметра должно быть больше граничного значения).|
|Parameter "{arg_name}" value should be greater than {lower_boundary} (Значение параметра {arg_name} должно быть больше значения {lower_boundary}).|
|Parameter "{arg_name}" has value "{actual_value}" which should be greater than {lower_boundary} (Параметр {arg_name} имеет значение {actual_value}, которое должно быть больше значения {lower_boundary}).|


## <a name="error-0005"></a>Ошибка 0005  
 Исключение возникает, если параметр меньше определенного значения.  

 Эта ошибка возникает в Машинном обучении Azure, если параметр в сообщении имеет значение ниже или равное граничному значению, необходимому модулю для обработки данных.  

**Решение.** Повторно откройте модуль, вызывающий исключение, и измените значение параметра, чтобы оно было больше или равно указанному значению.  

|Сообщения об исключении|
|------------------------|
|Parameter should be greater than or equal to boundary value (Параметр должен быть больше или равен граничному значению).|
|Parameter "{arg_name}" value should be greater than or equal to {lower_boundary} (Значение параметра {arg_name} должно быть больше или равно значению {lower_boundary}).|
|Parameter "{arg_name}" has value "{value}" which should be greater than or equal to {lower_boundary} (Параметр {arg_name} имеет значение {value}, которое должно быть больше или равно значению {lower_boundary}).|


## <a name="error-0006"></a>Ошибка 0006  
 Исключение возникает, если значение параметра больше или равно определенному значению.  

 Эта ошибка возникает в Машинном обучении Azure, если параметр в сообщении имеет значение выше или равное граничному значению, необходимому модулю для обработки данных.  

**Решение.** Повторно откройте модуль, вызывающий исключение, и измените значение параметра, чтобы оно было меньше указанного значения.  

|Сообщения об исключении|
|------------------------|
|Parameters mismatch. One of the parameters should be less than another (Несоответствие параметров. Значение одного параметра должно быть меньше значения другого).|
|Parameter "{arg_name}" value should be less than parameter "{upper_boundary_parameter_name}" value (Значение параметра {arg_name} должно быть меньше значения параметра {upper_boundary_parameter_name}).|
|Parameter "{arg_name}" has value "{value}" which should be less than {upper_boundary_parameter_name} (Параметр {arg_name} имеет значение {value}, которое должно быть меньше значения {upper_boundary_parameter_name}).|


## <a name="error-0007"></a>Ошибка 0007  
 Исключение возникает, если параметр больше определенного значения.  

 Эта ошибка возникает в Машинном обучении Azure, если в свойствах модуля указано значение, превышающее разрешенное. Например, можно задать данные, которые находятся вне диапазона поддерживаемых дат, или указать, что будут использоваться пять столбцов, когда доступны только три. 

 Эта ошибка может также возникать, если указать два набора данных, которые необходимо сопоставить. Например, если вы переименовываете столбцы и указываете столбцы по индексу, количество указываемых имен должно совпадать с числом индексов столбцов. Другим примером может служить математическая операция, использующая два столбца, где столбцы должны иметь одинаковое число строк. 

**Решение.**

 + Откройте рассматриваемый модуль и проверьте параметры числовых свойств.
 + Убедитесь, что значения параметров находятся в пределах поддерживаемого диапазона значений для этого свойства.
 + Если модуль принимает несколько входных данных, убедитесь, что данные имеют одинаковый размер.
<!-- + If the module has multiple properties that can be set, ensure that related properties have appropriate values. For example, when using [Group Data into Bins](group-data-into-bins.md), if you use the option to specify custom bin edges, the number of bins must match the number of values you provide as bin boundaries.-->
 + Проверьте, изменился ли набор или источник данных. Иногда значение, которое работало с предыдущей версией данных, будет выдавать ошибку после того, как число столбцов, тип данных столбца или размер данных изменились.  

|Сообщения об исключении|
|------------------------|
|Parameters mismatch. One of the parameters should be less than or equal to another. (Несоответствие параметров. Значение одного из параметров должно быть меньше или равно значению другого).|
|Parameter "{arg_name}" value should be less than or equal to parameter "{upper_boundary_parameter_name}" value (Значение параметра {arg_name} должно быть меньше или равно значению параметра {upper_boundary_parameter_name}).|
|Parameter "{arg_name}" has value "{actual_value}" which should be less than or equal to {upper_boundary} (Параметр {arg_name} имеет значение {actual_value}, которое должно быть меньше или равно значению {upper_boundary}).|
|Parameter "{arg_name}" value {actual_value} should be less than or equal to parameter "{upper_boundary_parameter_name}" value {upper_boundary} (Значение {actual_value} параметра {arg_name} должно быть меньше или равно значению {upper_boundary} параметра {upper_boundary_parameter_name}).|
|Parameter "{arg_name}" value {actual_value} should be less than or equal to {upper_boundary_meaning} value {upper_boundary} (Значение {actual_value} параметра {arg_name} должно быть меньше или равно значению {upper_boundary} параметра {upper_boundary_meaning}).|


## <a name="error-0008"></a>Ошибка 0008  
 Исключение возникает, если параметр находится за пределами диапазона.  

 Эта ошибка возникает в Машинном обучении Azure, если значение параметра в сообщении выходит за пределы граничного значения, необходимого модулю для обработки данных.  

 Например, эта ошибка отображается при попытке использовать функции [добавления строк](add-rows.md) для объединения двух наборов данных, имеющих разное число столбцов.  

**Решение.** Повторно откройте модуль, вызывающий исключение, и измените значение параметра, чтобы оно находилось в заданном диапазоне.  

|Сообщения об исключении|
|------------------------|
|Parameter value is not in the specified range (Значение параметра находится за пределами указанного диапазона).|
|Parameter "{arg_name}" value is not in range (Значение параметра {arg_name} находится за пределами диапазона).|
|Parameter "{arg_name}" value should be in the range of [{lower_boundary}, {upper_boundary}] (Значение параметра {arg_name} должно находиться в диапазоне [{lower_boundary}, {upper_boundary}]).|
|Parameter "{arg_name}" value is not in range (Значение параметра {arg_name} находится за пределами диапазона). {reason}|


## <a name="error-0009"></a>Ошибка 0009  
 Исключение возникает, если имя учетной записи хранения Azure или имя контейнера указаны неверно.  

Эта ошибка возникает в конструкторе Машинного обучения Azure, когда вы задаете параметры для учетной записи хранения Azure, но имя или пароль не удается разрешить. Ошибки в паролях или именах учетных записей могут происходить по многим причинам.

 + Неверный тип учетной записи. Некоторые новые типы учетных записей не поддерживаются в конструкторе Машинного обучения. Дополнительные сведения см. в статье [Модуль импорта данных](import-data.md).
 + Введено неправильное имя учетной записи.
 + Учетная запись больше не существует.
 + Неправильный или измененный пароль учетной записи хранения.
 + Вы не указали имя контейнера или контейнер не существует.
 + Путь к файлу не был полностью указан (путь к большому двоичному объекту).
   

**Решение.**

Такие проблемы часто возникают при попытке вручную ввести имя учетной записи, пароль или путь к контейнеру. Рекомендуется использовать новый мастер для модуля [импорта данных](import-data.md), который помогает искать и проверять имена.

Также проверьте, удалена ли учетная запись, контейнер или большой двоичный объект. Используйте другую служебную программу службы хранилища Azure, чтобы убедиться, что имя учетной записи и пароль введены правильно и что контейнер существует. 

Некоторые новые типы учетных записей не поддерживаются Машинным обучением Azure. Например, новые типы хранилища ("горячее" или "холодное") нельзя использовать для машинного обучения. Как классические учетные записи хранения, так и учетные записи хранения общего назначения работают нормально.

Если указан полный путь к большому двоичному объекту, убедитесь, что путь указан как **container/blobname** и что контейнер и большой двоичный объект существуют в учетной записи.  

 Путь не должен содержать начальную косую черту. Например путь **/container/blob** неверен и должен иметь следующий формат: **container/blob**.  


|Сообщения об исключении|
|------------------------|
|The Azure storage account name or container name is incorrect (Неверное имя учетной записи хранилища или контейнера Azure).|
|The Azure storage account name "{account_name}" or container name "{container_name}" is incorrect; a container name of the format container/blob was expected (Имя учетной записи хранения {account_name} или контейнера {container_name} Azure указано неверно. Ожидалось имя container/blob).|


## <a name="error-0010"></a>Ошибка 0010  
 Исключение возникает, если имена столбцов входных наборов данных должны совпадать, но они не совпадают.  

 Эта ошибка возникает в Машинном обучении Azure, если индекс столбца в сообщении содержит разные имена столбцов в двух наборах входных данных.  

**Решение.** Используйте модуль [изменения метаданных](edit-metadata.md) или измените исходный набор данных так, чтобы он совпадал с именем столбца для указанного индекса столбца.  

|Сообщения об исключении|
|------------------------|
|Columns with corresponding index in input datasets have different names. (Имена столбцов с соответствующим индексом во входных наборах данных отличаются).|
|Column names are not the same for column {col_index} (zero-based) of input datasets ({dataset1} and {dataset2} respectively) (Не совпадают имена столбцов {col_index} (начиная с нуля) входных наборов данных ({DataSet1} и {DataSet2} соответственно)).|


## <a name="error-0011"></a>Ошибка 0011  
 Исключение возникает, если переданный аргумент набора столбцов не применяется к любому из столбцов набора данных.  

 Эта ошибка возникает в Машинном обучении Azure, если выбранный столбец не соответствует ни одному из столбцов в данном наборе данных.  

 Кроме того, эта ошибка может возникнуть, если вы не выбрали столбец, а для работы модуля требуется хотя бы один столбец.  

**Решение.** Измените выбор столбца в модуле, чтобы он применялся к столбцам в наборе данных.  

 Если для модуля требуется выбрать конкретный столбец, например столбец меток, убедитесь, что выбран нужный.  

 Если выбраны недопустимые столбцы, удалите их и перезапустите конвейер.  

|Сообщения об исключении|
|------------------------|
|Specified column set does not apply to any of dataset columns (Указанный набор столбцов не применяется к каким-либо из столбцов набора данных).|
|Specified column set "{column_set}" does not apply to any of dataset columns (Указанный набор столбцов {column_set} не применяется к каким-либо из столбцов набора данных).|


## <a name="error-0012"></a>Ошибка 0012  
 Исключение возникает, если не удалось создать экземпляр класса с переданным набором аргументов.  

**Решение.** Эта ошибка не может быть исправлена пользователем и будет признана устаревшей в будущем выпуске.  

|Сообщения об исключении|
|------------------------|
|Untrained model, please train model first (Модель не обучена. Сначала выполните обучение модели).|
|Untrained model ({arg_name}), use trained model (Модель ({arg_name}) не обучена. Используйте обученную модель).|


## <a name="error-0013"></a>Ошибка 0013  
 Исключение возникает при передаче в модуль неправильного типа ученика.  

 Эта ошибка возникает всякий раз, когда обученная модель несовместима с подключенным модулем оценки. <!--For example, connecting the output of [Train Matchbox Recommender](train-matchbox-recommender.md) to [Score Model](score-model.md) (instead of [Score Matchbox Recommender](score-matchbox-recommender.md)) will generate this error when the pipeline is run.  -->

**Решение.**

Определите тип ученика, создаваемого модулем обучения, и укажите, какой модуль оценки подходит для этого ученика. 

Если модель была обучена с помощью любого из специализированных обучающих модулей, подключайте обученную модель только к соответствующему специализированному модулю оценки. 


|Тип модели|Модуль обучения| Модуль оценки|
|----|----|----|
|любой классификатор|[Train Model](train-model.md) (Обучение модели); |[Оценка модели](score-model.md)|
|любая модель регрессии|[Train Model](train-model.md) (Обучение модели); |[Оценка модели](score-model.md)|

<!--| clustering models| [Train Clustering Model](train-clustering-model.md) or [Sweep Clustering](sweep-clustering.md)| [Assign Data to Clusters](assign-data-to-clusters.md)|
| anomaly detection - One-Class SVM | [Train Anomaly Detection Model](train-anomaly-detection-model.md) |[Score Model](score-model.md)|
| anomaly detection - PCA |[Train Model](train-model.md) |[Score Model](score-model.md) </br> Some additional steps are required to evaluate the model. |
| anomaly detection - time series|  [Time Series Anomaly Detection](time-series-anomaly-detection.md) |Model trains from data and generates scores. The module does not create a trained learner and no additional scoring is required. |
| recommendation model| [Train Matchbox Recommender](train-matchbox-recommender.md) | [Score Matchbox Recommender](score-matchbox-recommender.md) |
| image classification | [Pretrained Cascade Image Classification](pretrained-cascade-image-classification.md) | [Score Model](score-model.md) |
|Vowpal Wabbit models| [Train Vowpal Wabbit Version 7-4 Model](train-vowpal-wabbit-version-7-4-model.md) | [Score Vowpal Wabbit Version 7-4 Model](score-vowpal-wabbit-version-7-4-model.md) |   
|Vowpal Wabbit models| [Train Vowpal Wabbit Version 7-10 Model](train-vowpal-wabbit-version-7-10-model.md) | [Score Vowpal Wabbit Version 7-10 Model](score-vowpal-wabbit-version-7-10-model.md) |
|Vowpal Wabbit models| [Train Vowpal Wabbit Version 8 Model](score-vowpal-wabbit-version-8-model.md) | [Score Vowpal Wabbit Version 8 Model](score-vowpal-wabbit-version-8-model.md) |-->

|Сообщения об исключении|
|------------------------|
|Learner of invalid type is passed (Передан ученик недопустимого типа).|
|Learner "{arg_name}" has invalid type (Недопустимый тип ученика {arg_name}).|
|Learner "{arg_name}" has invalid type "{learner_type}" (Ученик {arg_name} имеет недопустимый тип параметра {learner_type}).|
|Learner of invalid type is passed (Передан ученик недопустимого типа). Exception message: {exception_message} (Передано средство обучения недопустимого типа. Сообщение об исключении: {сообщение_об_исключении}).|


## <a name="error-0014"></a>Ошибка 0014  
 Исключение возникает, если количество уникальных значений столбца больше чем разрешено.  

 Эта ошибка возникает, когда столбец содержит слишком много уникальных значений, таких как столбец ИДЕНТИФИКАТОРов или текстовый столбец. Эта ошибка может возникать, если указать, что столбец должен обрабатываться как данные категории, но в столбце слишком много уникальных значений, позволяющих завершить обработку. Эта ошибка может также возникнуть в случае несоответствия числа уникальных значений в двух входных значениях.   

Ошибка, связанная с уникальными значениями, больше, чем разрешено, будет иметь место при **выполнении следующих условий** :

- Более 97% экземпляров одного столбца являются уникальными значениями, что означает, что почти все категории отличаются друг от друга.
- Один столбец содержит более 1000 уникальных значений.

**Решение.**

Откройте модуль, создавший ошибку, и найдите столбцы, используемые в качестве входных данных. В некоторых модулях можно щелкнуть правой кнопкой мыши входные данные набора данных и выбрать пункт **Визуализировать**, чтобы получить статистику по отдельным столбцам, включая количество уникальных значений и их распределение.

Для столбцов, которые предполагается использовать для группирования или категоризации, следует сократить число уникальных значений в столбцах. В зависимости от типа данных столбца их можно уменьшить различными способами. 

Для столбцов ИДЕНТИФИКАТОРов, которые не являются значимыми функциями во время обучения модели, можно использовать функцию [изменить метаданные](../algorithm-module-reference/edit-metadata.md) , чтобы пометить этот столбец как **четкий** , и он не будет использоваться во время обучения модели. 

Для текстовых столбцов можно использовать [хэширование компонентов](../algorithm-module-reference/feature-hashing.md) или [извлечь N-граммные функции из текстового модуля](../algorithm-module-reference/extract-n-gram-features-from-text.md) для предварительной обработки текстовых столбцов.
<!--
+ For text data, you might be able to use [Preprocess Text](preprocess-text.md) to collapse similar entries. 
+ For numeric data, you can create a smaller number of bins using [Group Data into Bins](group-data-into-bins.md), remove or truncate values using [Clip Values](clip-values.md), or use machine learning methods such as [Principal Component Analysis](principal-component-analysis.md) or [Learning with Counts](data-transformation-learning-with-counts.md) to reduce the dimensionality of the data.  
-->
> [!TIP]
> Не удается найти решение, соответствующее вашему сценарию? Вы можете отправить отзыв об этом разделе, который содержит имя модуля, в котором возникла ошибка, а также тип данных и количество элементов в столбце. С помощью этих сведений мы сможем предоставить более подробные инструкции по устранению неполадок для распространенных сценариев.   

|Сообщения об исключении|
|------------------------|
|Amount of column unique values is greater than allowed (Количество уникальных значений столбца больше допустимого).|
|Number of unique values in column: "{column_name}" is greater than allowed (Число уникальных значений в столбце {column_name} больше допустимого).|
|Number of unique values in column: "{column_name}" exceeds tuple count of {limitation} (Число уникальных значений в столбце {column_name} превышает число кортежей {limitation}).|


## <a name="error-0015"></a>Ошибка 0015  
 Исключение возникает, если произошел сбой подключения к базе данных.  

 Эта ошибка возникает при вводе неправильного имени учетной записи SQL, пароля, сервера базы данных или имени базы данных, а также в случае, если подключение к базе данных не удается установить из-за проблем с базой данных или сервером.  

**Решение.** Убедитесь, что имя учетной записи, пароль, сервер базы данных и база данных введены правильно, а указанная учетная запись имеет правильный уровень разрешений. Убедитесь, что база данных в данный момент доступна.  

|Сообщения об исключении|
|------------------------|
|Error making database connection (Ошибка создания подключения к базе данных).|
|Error making database connection: {connection_str} (Ошибка создания подключения к базе данных: {connection_str}).|


## <a name="error-0016"></a>Ошибка 0016  
 Исключение возникает, если входные наборы данных, переданные в модуль, должны иметь совместимые типы столбцов, но на самом деле это не так.  

 Эта ошибка возникает в Машинном обучении Azure, если типы столбцов, переданные в двух или более наборах данных, несовместимы друг с другом.  

**Решение.** Используйте модуль [изменения метаданных](edit-metadata.md) или измените исходный входной набор данных,<!--, or use [Convert to Dataset](convert-to-dataset.md)--> чтобы обеспечить совместимость типов столбцов.  

|Сообщения об исключении|
|------------------------|
|Columns with corresponding index in input datasets do have incompatible types (Столбцы с соответствующим индексом во входных наборах данных имеют несовместимые типы).|
|Columns '{first_col_names}' are incompatible between train and test data (Столбцы {first_col_names} несовместимы в данных для обучения и тестирования).|
|Columns '{first_col_names}' and '{second_col_names}' are incompatible. (Столбцы {first_col_names} и {second_col_names} несовместимы).|
|Column element types are not compatible for column '{first_col_names}' (zero-based) of input datasets ({first_dataset_names} and {second_dataset_names} respectively) (Типы элементов столбцов несовместимы для столбца {first_col_names} (начиная с нуля) входных наборов данных ({first_dataset_names} и {second_dataset_names} соответственно)).|


## <a name="error-0017"></a>Ошибка 0017  
 Исключение возникает, если в выбранном столбце используется тип данных, который не поддерживается текущим модулем.  

 Например, эта ошибка может возникать в Машинном обучении Azure, если ваш выбранный столбец содержит столбец с типом данных, который не может быть обработан модулем, например строковый столбец для математической операции или столбец оценки, где требуется столбец категориальных признаков.  

**Решение.**
 1. Найдите столбец, содержащий проблему.
 2. Ознакомьтесь с требованиями модуля.
 3. Измените столбец, чтобы он соответствовал требованиям. Чтобы внести изменения, может потребоваться несколько следующих модулей в зависимости от используемого столбца и преобразования.
    + Используйте модуль [изменения метаданных](edit-metadata.md), чтобы изменить тип данных столбцов или изменить значение использования столбца с функционального на числовое, категориального на некатегориальное и т. д.
<!--    + Use [Convert to Dataset](convert-to-dataset.md) to ensure that all included columns use data types that are supported by Azure Machine Learning.  If you cannot convert the columns, consider removing them from the input dataset.
    + Use the [Apply SQL Transformation](apply-sql-transformation.md) or [Execute R Script](execute-r-script.md) modules to cast or convert any columns that cannot be modified using [Edit Metadata](edit-metadata.md). These modules provide more flexibility for working with datetime data types.
    + For numeric data types, you can use the [Apply Math Operation](apply-math-operation.md) module to round or truncate values, or use the [Clip Values](clip-values.md) module to remove out of range values.  -->
 4. В крайнем случае попробуйте изменить исходный входной набор данных.

> [!TIP]
> Не удается найти решение, соответствующее вашему сценарию? Вы можете отправить отзыв об этом разделе, который содержит имя модуля, в котором возникла ошибка, а также тип данных и количество элементов в столбце. С помощью этих сведений мы сможем предоставить более подробные инструкции по устранению неполадок для распространенных сценариев. 

|Сообщения об исключении|
|------------------------|
|Cannot process column of current type. The type is not supported by the module (Не удается обработать столбец текущего типа. Тип не поддерживается модулем).|
|Cannot process column of type {col_type}. The type is not supported by the module (Не удается обработать столбец типа {col_type}. Тип не поддерживается модулем).|
|Cannot process column "{col_name}" of type {col_type}. The type is not supported by the module. (Не удается обработать столбец {col_name} типа {col_type}. Тип не поддерживается модулем).|
|Cannot process column "{col_name}" of type {col_type}. The type is not supported by the module. Parameter name: {arg_name} (Не удается обработать столбец {col_name} типа {col_type}. Тип не поддерживается модулем). Имя параметра: {arg_name}.|


## <a name="error-0018"></a>Ошибка 0018  
 Исключение возникает, если входной набор данных не является допустимым.  

**Решение.** Эта ошибка в Машинном обучении Azure может возникать во многих контекстах, поэтому не существует единого решения. Как правило, эта ошибка означает, что данные, предоставленные в качестве входных данных для модуля, имеют неверное число столбцов или что тип данных не соответствует требованиям модуля. Пример:  

-   Для модуля требуется столбец метки, но ни один столбец не помечен как столбец меток или вы пока не выбрали столбец метки.  
  
-   Для модуля требуется, чтобы данные были упорядочены, а они представлены в числовом формате.  

<!---   The module requires a specific data type. For example, ratings provided to [Train Matchbox Recommender](train-matchbox-recommender.md) can be either numeric or categorical, but cannot be floating point numbers.  -->

-   Данные имеют неправильный формат.  
  
-   Импортированные данные содержат недопустимые символы, неверные значения или значения, которые выходят за пределы диапазона.  
-   Столбец пуст или содержит слишком много отсутствующих значений.  

 Чтобы определить требования и способ работы с данными, изучите раздел справки модуля, который будет использовать набор данных в качестве входных данных.  

 <!--We also recommend that you use [Summarize Data](summarize-data.md) or [Compute Elementary Statistics](compute-elementary-statistics.md) to profile your data, and use these modules to fix metadata and clean values: [Edit Metadata](edit-metadata.md) and [Clean Missing Data](clean-missing-data.md), [Clip Values](clip-values.md)-->.  

|Сообщения об исключении|
|------------------------|
|Dataset is not valid (Недопустимый набор данных).|
|{dataset1} contains invalid data (Параметр {dataset1} содержит недопустимые данные).|
|{dataset1} and {dataset2} should be consistent columnwise (Параметры {dataset1} и {dataset2} должны быть согласованы по столбцам).|
|{dataset1} contains invalid data, {reason} (Параметр {dataset1} содержит недопустимые данные, {reason}).|
|{dataset1} contains {invalid_data_category} (Параметр {dataset1} содержит {invalid_data_category}). {troubleshoot_hint}|
|{dataset1} is not valid, {reason} (Параметр {dataset1} является недопустимым, {reason}). {troubleshoot_hint}|


## <a name="error-0019"></a>Ошибка 0019  
 Исключение возникает, если столбец должен содержать отсортированные значения, но это не так.  

 Эта ошибка возникает в Машинном обучении Azure, если указанные значения столбца не упорядочены.  

**Решение.** Отсортируйте значения столбцов путем изменения входного набора данных вручную и повторно запустите модуль.  

|Сообщения об исключении|
|------------------------|
|Values in column are not sorted (Значения в столбце не сортированы).|
|Values in column "{col_index}" are not sorted (Значения в столбце {col_index} не сортированы).|
|Values in column "{col_index}" of dataset "{dataset}" are not sorted (Значения в столбце {col_index} набора данных {dataset} не отсортированы).|
|Values in argument "{arg_name}" are not sorted in "{sorting_order}" order (Значения в аргументе {arg_name} не сортируются в порядке {sorting_order}).|


## <a name="error-0020"></a>Ошибка 0020  
 Исключение возникает, если количество столбцов в некоторых наборах данных, переданных модулю, слишком мало.  

 Эта ошибка возникает в Машинном обучении Azure, если для модуля выбрано недостаточное количество столбцов.  

**Решение.** Повторно откройте модуль и убедитесь, что в селекторе столбцов выбрано правильное число столбцов.  

|Сообщения об исключении|
|------------------------|
|Number of columns in input dataset is less than allowed minimum (Число столбцов во входном наборе данных меньше допустимого минимального значения).|
|Number of columns in input dataset "{arg_name}" is less than allowed minimum (Число столбцов во входном наборе данных {arg_name} меньше допустимого минимального значения).|
|Number of columns in input dataset is less than allowed minimum of {required_columns_count} column(s) (Число столбцов во входном наборе данных меньше допустимого минимального значения столбца {required_columns_count}).|
|Number of columns in input dataset "{arg_name}" is less than allowed minimum of {required_columns_count} column(s) (Число столбцов во входном наборе данных {arg_name} меньше допустимого минимального значения столбца {required_columns_count}).|


## <a name="error-0021"></a>Ошибка 0021  
 Исключение возникает, если количество строк в некоторых наборах данных, переданных модулю, слишком мало.  

 Эта ошибка возникает в Машинном обучении Azure, когда в наборе данных недостаточно строк для выполнения указанной операции. Например, эта ошибка может возникать, если входной набор данных пуст или вы пытаетесь выполнить операцию, требующую минимального количества допустимых строк. Такие операции могут включать в себя (помимо прочего) группировку или классификацию на основе статистических методов, несколько типов квантования и обучение с учетом количества.  

**Решение.**

 + Откройте модуль, который возвратил ошибку, и проверьте входной набор данных и свойства модуля. 
 + Убедитесь, что входной набор данных не пуст и имеется достаточное количество строк данных для соответствия требованиям, описанным в разделе справки по модулям.  
 + Если данные загружаются из внешнего источника, убедитесь, что источник данных доступен и в определении данных нет ошибок или изменений, которые приведут к тому, что в процессе импорта будет меньше строк.
 + При выполнении операций с данными (размещенными выше модуля, который может повлиять на тип данных или на количество значений), таких как очистка, разделение или операции объединения, проверьте выходные данные этих операций, чтобы определить количество возвращаемых строк.  

|Сообщения об исключении|
|------------------------|
|Number of rows in input dataset is less than allowed minimum (Число строк во входном наборе данных меньше допустимого минимального значения).|
|Number of rows in input dataset is less than allowed minimum of {required_rows_count} row(s) (Число строк во входном наборе данных меньше допустимого минимального значения {required_rows_count}).|
|Number of rows in input dataset is less than allowed minimum of {required_rows_count} row(s) (Число строк во входном наборе данных меньше допустимого минимального значения {required_rows_count}). {reason}|
|Number of rows in input dataset "{arg_name}" is less than allowed minimum of {required_rows_count} row(s) (Число строк во входном наборе данных {arg_name} меньше допустимого минимального значения строк {required_rows_count}).|
|Number of rows in input dataset "{arg_name}" is {actual_rows_count}, less than allowed minimum of {required_rows_count} row(s) (Число строк во входном наборе данных {arg_name} равно {actual_rows_count}, что меньше допустимого минимума в строках {required_rows_count}).|
|Number of "{row_type}" rows in input dataset "{arg_name}" is {actual_rows_count}, less than allowed minimum of {required_rows_count} row(s) (Число строк {row_type} во входном наборе данных {arg_name} равно {actual_rows_count}, что меньше допустимого минимума в строках {required_rows_count}).|


## <a name="error-0022"></a>Ошибка 0022  
 Исключение возникает, если количество выбранных столбцов в наборе входных данных не равно ожидаемому числу.  

 Эта ошибка в Машинном обучении Azure может возникнуть, если нижестоящему модулю или операции требуется определенное количество столбцов или входных данных, а вы предоставили слишком малое их количество. Пример:  

-   Вы указываете один столбец меток или ключевой столбец и случайно выбрали несколько столбцов.  
  
-   Вы переименовываете столбцы, но указываете больше или меньше имен, чем количество столбцов.  
  
-   Число столбцов в источнике или назначении изменилось или не соответствует числу столбцов, используемых модулем.  
  
-   Вы указали список значений с разделителями-запятыми для входных данных, но число значений не совпадает или несколько входных данных не поддерживаются.  

**Решение.** Повторно откройте модуль и проверьте выбор столбца, чтобы убедиться, что выбрано правильное число столбцов. Проверьте выходные данные вышестоящих модулей и требования нисходящих операций.  

 Если вы использовали один из вариантов выбора столбцов, с помощью которого можно выбрать несколько столбцов (индексы столбцов, все объекты, все числовые значения и т. д.), проверьте точное количество столбцов, возвращаемых выделением.  

 <!--If you are trying to specify a comma-separated list of datasets as inputs to [Unpack Zipped Datasets](unpack-zipped-datasets.md), unpack only one dataset at a time. Multiple inputs are not supported.  -->

 Убедитесь, что число или тип вышестоящих столбцов не изменились.  

 Если для обучения модели используется набор данных рекомендаций, помните, что для рекомендуемого параметра предполагается наличие ограниченного числа столбцов, соответствующих парам "пользователь-элемент" или "пользователь-элемент-ранжирование". Удалите дополнительные столбцы перед обучением модели или разбиением наборов данных рекомендаций. Дополнительные сведения см. в статье [Модуль Split Data](split-data.md).  

|Сообщения об исключении|
|------------------------|
|Number of selected columns in input dataset does not equal to the expected number (Число выбранных столбцов во входном наборе данных не соответствует ожидаемому числу).|
|Number of selected columns in input dataset does not equal to {expected_col_count} (Число выбранных столбцов во входном наборе данных не соответствует {expected_col_count}).|
|Column selection pattern "{selection_pattern_friendly_name}" provides number of selected columns in input dataset not equal to {expected_col_count} (Шаблон выбора столбцов {selection_pattern_friendly_name} предоставляет число выбранных столбцов во входном наборе данных, которое не равно {expected_col_count}).|
|Column selection pattern "{selection_pattern_friendly_name}" is expected to provide {expected_col_count} column(s) selected in input dataset, but {selected_col_count} column(s) is/are actually provided (Ожидается, что шаблон выбора столбцов {selection_pattern_friendly_name} предоставит {expected_col_count} выбранных столбцов во входном наборе данных, но фактически предоставлено {selected_col_count} столбцов).|


## <a name="error-0023"></a>Ошибка 0023  

Исключение возникает, если целевой столбец входного набора данных не является допустимым для текущего учебного модуля.  

Эта ошибка в Машинном обучении Azure возникает, если целевой столбец (выбранный в параметрах модуля) не относится к допустимому типу данных, содержит все пропущенные значения или не является категориальным, как ожидалось.  

**Решение.** Повторно откройте входные данные модуля, чтобы проверить содержимое столбца меток или целевого столбца. Убедитесь, что в нем нет отсутствующих значений. Если модуль ожидает, что целевой столбец будет категориальным, убедитесь, что в целевом столбце имеется несколько различных значений.  

|Сообщения об исключении|
|------------------------|
|Input dataset has unsupported target column (Входной набор данных содержит неподдерживаемый целевой столбец).|
|Input dataset has unsupported target column "{column_index}" (Входной набор данных содержит неподдерживаемый целевой столбец {column_index}).|
|Input dataset has unsupported target column "{column_index}" for learner of type {learner_type} (Входной набор данных содержит неподдерживаемый целевой столбец {column_index} для ученика типа {learner_type}).|


## <a name="error-0024"></a>Ошибка 0024  
Исключение возникает, если набор данных не содержит столбец метки.  

 Эта ошибка в Машинном обучении Azure возникает, когда модулю требуется столбец меток, а набор данных его не имеет. Например, для вычисления оцененного набора данных обычно требуется наличие столбца меток, чтобы вычислить метрики точности.  

Также может произойти, что столбец меток присутствует в наборе данных, но не обнаружен должным образом Машинным обучением Azure.

**Решение.**

+ Откройте модуль, создавший ошибку, и определите, присутствует ли столбец меток. Имя или тип данных столбца не имеет значения, если столбец содержит один результат (или зависимую переменную), который вы пытаетесь спрогнозировать. Если вы не уверены, какой столбец имеет метку, найдите универсальное имя, например *Class* или *Target*. 
+  Если набор данных не содержит столбец меток, то возможно, что столбец был явно или случайно удален из вышестоящего модуля. Также может быть, что набор данных не является выходным для вышестоящего модуля оценки.
+ Чтобы явно пометить столбец как столбец меток, добавьте модуль [изменения метаданных](edit-metadata.md) и подключите набор данных. Выберите только столбец меток и щелкните **Label** (Метка) в раскрывающемся списке **Fields** (Поля). 
+ Если в качестве метки выбран неправильный столбец, можно выбрать **Clear label** (Удалить метку) в раскрывающемся списке **Fields** (Поля), чтобы исправить метаданные в столбце. 
  
|Сообщения об исключении|
|------------------------|
|There is no label column in dataset (В наборе данных отсутствует столбец меток).|
|There is no label column in "{dataset_name}" (В {dataset_name} отсутствует столбец метки).|


## <a name="error-0025"></a>Ошибка 0025  
 Исключение возникает, если набор данных не содержит столбец оценки.  

 Эта ошибка в Машинном обучении Azure возникает, если входные данные для модели оценки не содержат допустимых столбцов для оценки. Например, пользователь пытается оценить набор данных до того, как он был оценен с помощью правильной обученной модели, или столбец для оценки был явно удален из вышестоящего модуля. Это исключение также возникает, если столбцы оценки в двух наборах данных несовместимы. Например, возможно, вы пытаетесь сравнить точность линейного регрессора с двоичным классификатором.  

**Решение.** Пересмотрите входные данные в модели оценки и проверьте, содержит ли она один или несколько столбцов оценки. Если нет, то набор данных не был оценен или столбцы оценки были удалены в вышестоящем модуле.  

|Сообщения об исключении|
|------------------------|
|There is no score column in dataset (В наборе данных отсутствует столбец оценки).|
|There is no score column in "{dataset_name}" (В наборе данных отсутствует столбец оценки {dataset_name}).|
|There is no score column in "{dataset_name}" that is produced by a "{learner_type}". Score the dataset using the correct type of learner. (В {dataset_name} отсутствует столбец оценки, созданный {learner_type}. Оценка набора данных будет осуществляться с использованием ученика правильного типа).|


## <a name="error-0026"></a>Ошибка 0026  
 Исключение возникает, если не допускается использование столбцов с одинаковыми именами.  

 Эта ошибка в Машинном обучении Azure возникает, если несколько столбцов имеют одно и то же имя. Один из вариантов возникновения этой ошибки — если в наборе данных отсутствует имя строки заголовка, а имена столбцов автоматически назначаются: Col0, Col1 и т. д.  

**Решение.** Если столбцы имеют одинаковое имя, вставьте модуль [изменения метаданных](edit-metadata.md) между входным набором данных и модулем. Используйте селектор столбцов в модуле [изменения метаданных](edit-metadata.md), чтобы выбрать столбцы для переименования, и введите новые имена в текстовом поле **New column names** (Новые имена столбцов).  

|Сообщения об исключении|
|------------------------|
|Equal column names are specified in arguments. Equal column names are not allowed by module (В аргументах указаны одинаковые имена столбцов. В модуле не допускаются одинаковые имена столбцов).|
|Equal column names in arguments "{arg_name_1}" and "{arg_name_2}" are not allowed. Please specify different names (Одинаковые имена столбцов в аргументах {arg_name_1} и {arg_name_2} не допускаются. Укажите разные имена).|


## <a name="error-0027"></a>Ошибка 0027  
 Исключение возникает в случае, если два объекта должны иметь одинаковый размер, но на самом деле это не так.  

 Это распространенная ошибка в Машинном обучении Azure, которая может быть вызвана множеством условий.  

**Решение.** Для данной проблемы специального решения не существует. Тем не менее, можно проверить следующие условия:  

-   При переименовании столбцов убедитесь, что каждый список (входные столбцы и список новых имен) имеет одинаковое число элементов.  
  
-   При соединении или объединении двух наборов данных убедитесь, что они имеют одну и ту же схему.  
  
-   При соединении двух наборов данных с несколькими столбцами убедитесь, что ключевые столбцы имеют одинаковый тип и выберите параметр **Разрешить дубликаты и сохранять порядок столбцов в выделенном фрагменте**.  

|Сообщения об исключении|
|------------------------|
|The size of passed objects is inconsistent (Размер переданных объектов не согласован).|
|The size of "{friendly_name1}" is inconsistent with size of "{friendly_name2}" (Размер столбца {friendly_name1} не соответствует размеру столбца {friendly_name2}).|


## <a name="error-0028"></a>Ошибка 0028  
 Исключение возникает в случае, если набор столбцов содержит повторяющиеся имена столбцов, что недопустимо.  

 Эта ошибка в Машинном обучении Azure возникает при дублировании имен столбцов, которые не являются уникальными.  

**Решение.** Если столбцы имеют одинаковые имена, добавьте экземпляр модуля [изменения метаданных](edit-metadata.md) между входным набором данных и модулем, вызвавшим ошибку. Используйте селектор столбцов в модуле [изменения метаданных](edit-metadata.md), чтобы выбрать столбцы для переименования, и введите имена новых столбцов в текстовом поле **New column names** (Новые имена столбцов). При переименовании нескольких столбцов убедитесь, что значения, введенные в поле **Новые имена столбцов** (Новые имена столбцов), являются уникальными.  

|Сообщения об исключении|
|------------------------|
|Column set contains duplicated column name(s) (Набор столбцов содержит столбцы с повторяющимися именами).|
|The name "{duplicated_name}" is duplicated (Имя {duplicated_name} повторяется).|
|The name "{duplicated_name}" is duplicated in "{arg_name}" (Имя {duplicated_name} повторяется в {arg_name}).|
|The name "{duplicated_name}" is duplicated (Имя {duplicated_name} повторяется). Сведения: {details}|


## <a name="error-0029"></a>Ошибка 0029  
 Исключение возникает в случае передачи недопустимого универсального кода ресурса (URI).  

 Эта ошибка в Машинном обучении Azure возникает в случае, если передан недопустимый универсальный код ресурса (URI).  Эта ошибка возникает, если выполняется одно из следующих условий.  

-   Общедоступный URI или URI SAS, предоставленный для хранилища BLOB-объектов Azure для чтения или записи, содержит ошибку.  
  
-   Срок действия временного интервала для SAS истек.  
  
-   URL-адрес с использованием протокола HTTP представляет файл или URI с замыканием на себя.  
  
-   URL-адрес с использованием протокола HTTP содержит неверно отформатированный URL-адрес.  
  
-   URL-адрес не может быть разрешен удаленным источником.  

**Решение.** Повторно откройте модуль и проверьте формат универсального кода ресурса (URI). Если источником данных является URL-адрес с использованием протокола HTTP, убедитесь, что предполагаемый источник не является файлом или URI с замыканием на себя (localhost).  

|Сообщения об исключении|
|------------------------|
|Invalid URI is passed (Передан недопустимый URI).|
|The URI "{invalid_url}" is invalid (URI {invalid_url} недопустимый).|


## <a name="error-0030"></a>Ошибка 0030  
 Исключение возникает, если не удается загрузить файл.  

 Это исключение в Машинном обучении Azure возникает, если невозможно загрузить файл. Это исключение будет получено, если попытка чтения из источника HTTP завершилась сбоем после трех (3) повторных попыток.  

**Решение.** Убедитесь, что URI-адрес источника HTTP указан правильно и что сайт в данный момент доступен в сети Интернет.  

|Сообщения об исключении|
|------------------------|
|Unable to download a file (Не удалось скачать файл).|
|Error while downloading the file: {file_url} (Ошибка при скачивании файла: {file_url}).|


## <a name="error-0031"></a>Ошибка 0031  
 Исключение возникает, если количество столбцов в наборе столбцов меньше необходимого.  

 Эта ошибка в Машинном обучении Azure возникает, если выбранное число столбцов меньше необходимого.  Эта ошибка возникает, если не выбрано минимальное требуемое число столбцов.  

**Решение.** Добавьте дополнительные столбцы к выбранному столбцу с помощью **селектора столбцов**.  

|Сообщения об исключении|
|------------------------|
|Number of columns in column set is less than required (Количество столбцов в наборе столбцов меньше необходимого).|
|At least {required_columns_count} column(s) should be specified for input argument "{arg_name}" (Для входного аргумента {arg_name} необходимо указать по крайней мере такое количество столбцов: {required_columns_count}).|
|At least {required_columns_count} column(s) should be specified for input argument "{arg_name}" (Для входного аргумента {arg_name} необходимо указать по крайней мере такое количество столбцов: {required_columns_count}). The actual number of specified columns is {input_columns_count} (Фактическое число указанных столбцов: {input_columns_count}).|


## <a name="error-0032"></a>Ошибка 0032  
 Исключение возникает, если аргумент не является числом.  

 Эта ошибка возникает в Машинном обучении Azure, если аргумент дублируется или имеет значение NaN (не число).  

**Решение.** Измените указанный аргумент, чтобы использовать допустимое значение.  

|Сообщения об исключении|
|------------------------|
|Argument is not a number (Аргумент не является числом).|
|"{arg_name}" is not a number ({arg_name} не является числом).|


## <a name="error-0033"></a>Ошибка 0033  
 Исключение возникает, если аргумент — бесконечность.  

 Эта ошибка в Машинном обучении Azure возникает, если аргумент имеет бесконечное значение. Эта ошибка возникает, если аргумент имеет значение `double.NegativeInfinity` или `double.PositiveInfinity`.  

**Решение.** Измените указанный аргумент, чтобы использовать допустимое значение.  

|Сообщения об исключении|
|------------------------|
|Argument must be finite (Аргумент должен быть конечным).|
|"{arg_name}" is not finite ({arg_name} не является конечным).|
|Column "{column_name}" contains infinite values (Столбец {column_name} содержит бесконечные значения).|


## <a name="error-0034"></a>Ошибка 0034  
 Исключение возникает, если для данной пары "пользователь-элемент" существует более одной оценки.  

 Эта ошибка в Машинном обучении Azure возникает в рекомендации, если в паре "пользователь-элемент" содержится более одного значения оценки.  

**Решение.** Убедитесь, что в паре "пользователь-элемент" имеется только одно значение оценки.  

|Сообщения об исключении|
|------------------------|
|More than one rating exists for the value(s) in dataset (В наборе данных существует несколько оценок для значений).|
|More than one rating for user {user} and item {item} in rating prediction data table (В таблице прогнозирующих данных существует несколько оценок для одного пользователя {user} и элемента {item}).|
|More than one rating for user {user} and item {item} in {dataset} (В {dataset} существует несколько оценок для пользователя {user} и элемента {item} в {dataset}).|


## <a name="error-0035"></a>Ошибка 0035  
 Исключение возникает, если для данного пользователя или элемента не были предоставлены какие-либо признаки.  

 Эта ошибка в Машинном обучении Azure возникает при попытке использовать модель рекомендаций для оценки, когда невозможно найти вектор функций.  

**Решение.**

В модели рекомендаций Matchbox требуется соблюдать определенные требования, которые должны быть выполнены при использовании признаков элементов или признаков пользователя.  Эта ошибка означает, что для пользователя или элемента, указанного в качестве входных данных, отсутствует вектор признаков. Убедитесь, что в данных для каждого пользователя или элемента доступен вектор признаков.  

 Например, если вы обучили модель рекомендаций с помощью таких признаков, как возраст пользователя, местоположение или доход, но теперь хотите создать оценки для новых пользователей, которые не были оценены во время обучения, необходимо предоставить для новых пользователей какой-либо эквивалентный набор признаков (а именно, возраст, расположение и доходы), чтобы создать соответствующие прогнозы. 

 Если у вас нет признаков для этих пользователей, сконструируйте соответствующие признаки.  Например, если отсутствуют индивидуальные значения возраста или дохода пользователя, можно создать приблизительные значения, которые будут использоваться для группы пользователей. 

<!--When you are scoring from a recommendation mode, you can use item or user features only if you previously used item or user features during training. For more information, see [Score Matchbox Recommender](score-matchbox-recommender.md).

For general information about how the Matchbox recommendation algorithm works, and how to prepare a dataset of item features or user features, see [Train Matchbox Recommender](train-matchbox-recommender.md).  -->

 > [!TIP]
 > Решение неприменимо к вашему случаю? Вы можете отправить отзыв об этой статье и предоставить сведения о сценарии, включая модуль и число строк в столбце. Эти сведения будут использоваться для предоставления более подробных инструкций по устранению неполадок в будущем.

|Сообщения об исключении|
|------------------------|
|No features were provided for a required user or item (Признаки не предоставлены для пользователя или элемента).|
|Features for {required_feature_name} required but not provided (Не предоставлены обязательные признаки для {required_feature_name}).|


## <a name="error-0036"></a>Ошибка 0036  
 Исключение возникает, если для данного пользователя или элемента было предоставлено несколько векторов признаков.  

 Эта ошибка в Машинном обучении Azure возникает, если вектор признаков указан несколько раз.  

**Решение.** Убедитесь, что вектор признаков не указан несколько раз.  

|Сообщения об исключении|
|------------------------|
|Duplicate feature definition for a user or item (Повторяющееся определение признака для пользователя или элемента).|


## <a name="error-0037"></a>Ошибка 0037  
 Исключение возникает, если указано несколько столбцов метки, а допускается только один.  

 Эта ошибка в Машинном обучении Azure возникает, если выбрано несколько столбцов в качестве нового столбца меток. Для большинства управляемых алгоритмов обучения необходимо, чтобы один столбец был помечен как целевой столбец или столбец меток.  

**Решение.** Выберите один столбец в качестве нового столбца меток.  

|Сообщения об исключении|
|------------------------|
|Multiple label columns are specified (Указано несколько столбцов меток).|
|Multiple label columns are specified in "{dataset_name}" (Указано несколько столбцов меток в {dataset_name}).|


## <a name="error-0039"></a>Ошибка 0039  
 Исключение возникает, если не удалось выполнить операцию.  

 Эта ошибка в Машинном обучении Azure возникает, когда не удается выполнить внутреннюю операцию.  

**Решение.** Эта ошибка вызвана множеством условий, и не существует конкретного решения.  
 В следующей таблице содержатся общие сообщения об этой ошибке, за которыми следует определенное описание условия. 

 Если сведения недоступны, перейдите в [раздел вопросов и ответов на сайте Майкрософт](/answers/topics/azure-machine-learning-studio-classic.html), чтобы оставить отзыв и предоставить сведения о модулях, сформировавших ошибку и связанные с нею условия.

|Сообщения об исключении|
|------------------------|
|Ошибка при выполнении операции.|
|Error while completing operation: "{failed_operation}" (Ошибка при выполнении операции: {failed_operation}).|
|Error while completing operation: "{failed_operation}" (Ошибка при выполнении операции: {failed_operation}). Reason: {reason}. Причина: {reason}.|


## <a name="error-0042"></a>Ошибка 0042  
 Исключение возникает, если невозможно преобразовать столбец в другой тип.  

 Эта ошибка в Машинном обучении Azure возникает, когда невозможно преобразовать столбец в указанный тип.  Эта ошибка возникает, если модулю требуется определенный тип данных, например DateTime, текст, число с плавающей запятой или целое число, но невозможно преобразовать существующий столбец в требуемый тип.  

Например, можно выбрать столбец и попытаться преобразовать его в числовой тип данных для использования в математической операции и получить эту ошибку, если столбец содержал недопустимые данные. 

Другая причина может возникать при попытке использовать столбец, содержащий числа с плавающей запятой или множество уникальных значений в качестве столбца категорий. 

**Решение.**

+ Откройте страницу справки для модуля, создавшего ошибку, и проверьте требования к типу данных.
+ Проверьте типы данных столбцов во входном наборе данных.
+ Проверьте данные, исходящие из так называемых источников данных без схем.
+ Проверьте набор данных на наличие отсутствующих значений или специальных символов, которые могут блокировать преобразование в нужный тип данных. 
    + Числовые типы данных должны быть одинаковыми: например, проверьте наличие чисел с плавающей запятой в столбце целых чисел.
    + Поищите текстовые строки или значения Н/Д в столбце чисел. 
    + Логические значения можно преобразовать в соответствующее представление в зависимости от требуемого типа данных.
    + Проверьте текстовые столбцы на наличие символов в кодировке, отличной от Юникода, знаков табуляции или управляющих символов.
    + Данные типа Datetime должны быть согласованными, чтобы избежать ошибок моделирования, но очистка может быть сложной из-за множества форматов. Рекомендуемые сценарии <!--the [Execute R Script](execute-r-script.md) or -->Модули [Execute Python Script](execute-python-script.md) (Выполнение скрипта Python) можно использовать для выполнения очистки.  
+ При необходимости измените значения во входном наборе данных, чтобы столбец мог быть успешно преобразован. Изменения могут включать в себя операции группирования, усечения или округления, исключения выбросов или добавления отсутствующих значений. Сведения о некоторых распространенных сценариях преобразования данных в машинном обучении см. в следующих статьях:
    + [Модуль очистки отсутствующих данных](clean-missing-data.md)
    + [Модуль нормализации данных](normalize-data.md)
<!--+ [Clip Values](clip-values.md) 
    + [Group Data Into Bins](group-data-into-bins.md)
  -->

> [!TIP]
> Решение не понятно или неприменимо к вашему случаю? Вы можете отправить отзыв об этой статье и предоставить сведения о сценарии, включая модуль и тип данных в столбце. Эти сведения будут использоваться для предоставления более подробных инструкций по устранению неполадок в будущем.  

|Сообщения об исключении|
|------------------------|
|Not allowed conversion (Преобразование не допускается).|
|Could not convert column of type {type1} to column of type {type2} (Не удалось преобразовать столбец типа {type1} в столбец типа {type2}).|
|Could not convert column "{col_name1}" of type {type1} to column of type {type2} (Не удалось преобразовать столбец {col_name1} типа {type1} в столбец типа {type2}).|
|Could not convert column "{col_name1}" of type {type1} to column "{col_name2}" of type {type2} (Не удалось преобразовать столбец {col_name1} типа {type1} в столбец {col_name2} типа {type2}).|


## <a name="error-0044"></a>Ошибка 0044  
 Исключение возникает, если невозможно получить тип элемента столбца из существующих значений.  

 Эта ошибка в Машинном обучении Azure возникает, когда невозможно определить тип столбца или столбцов в наборе данных. Обычно это происходит при объединении двух или более наборов данных с разными типами элементов. Если Машинному обучению Azure не удается определить общий тип, который способен представлять все значения в столбце или столбцах без потери данных, это приведет к возникновению этой ошибки.  

**Решение.** Убедитесь, что все значения в данном столбце в обоих объединяемых наборах данных имеют один и тот же тип (числовой, логический, категориальный, строковый, дата и т. д.) или может быть приведен к тому же типу.  

|Сообщения об исключении|
|------------------------|
|Cannot derive element type of the column (Не удается создать производный тип элемента столбца).|
|Cannot derive element type for column "{column_name}" -- all the elements are null references. (Не удается создать производный тип элемента для столбца {column_name}: все элементы — пустые ссылки).|
|Cannot derive element type for column "{column_name}" of dataset "{dataset_name}" -- all the elements are null references (Не удается создать производный тип элемента для столбца {column_name} набора данных {dataset_name}: все элементы — пустые ссылки).|


## <a name="error-0045"></a>Ошибка 0045  
 Исключение возникает, если невозможно создать столбец из-за смешанных типов элементов в источнике.  

 Эта ошибка в Машинном обучении Azure возникает, если типы элементов двух объединяемых наборов данных отличаются.  

**Решение.** Убедитесь, что все значения в данном столбце в обоих объединяемых наборах данных имеют один и тот же тип (числовой, логический, категориальный, строковый, дата и т. д.).  

|Сообщения об исключении|
|------------------------|
|Cannot create column with mixed element types (Не удается создать столбец со смешанными типами элементов).|
|Cannot create column with id "{column_id}" of mixed element types (Не удается создать столбец с идентификатором {column_id} типов смешанных элементов):<br />Type of data[{row_1}, {column_id}] is "{type_1}" Тип данных [{row_1}, {column_id}] — {type_1}. <br />Type of data[{row_2}, {column_id}] is "{type_2}" (Тип данных [{row_2}, {column_id}] — {type_2}).|
|Cannot create column with id "{column_id}" of mixed element types (Не удается создать столбец с идентификатором {column_id} типов смешанных элементов):<br />Type in chunk {chunk_id_1} is "{type_1}" (Тип в блоке {chunk_id_1} — {type_1}). <br />Type in chunk {chunk_id_2} is "{type_2}" with chunk size: {chunk_size} (Тип в блоке {chunk_id_2} — {type_2} с размером блока: {chunk_size}).|


## <a name="error-0046"></a>Ошибка 0046  
 Исключение возникает, если невозможно создать каталог по указанному пути.  

 Эта ошибка в Машинном обучении Azure возникает, когда невозможно создать каталог по указанному пути. Эта ошибка возникает, если какая-либо часть пути к выходному каталогу для запроса Hive неверна или недоступна.  

**Решение.** Повторно откройте модуль и убедитесь, что путь к каталогу правильно отформатирован и доступен с использованием текущих учетных данных.  

|Сообщения об исключении|
|------------------------|
|Please specify a valid output directory (Укажите допустимый выходной каталог).|
|Directory: {path} cannot be created. Please specify valid path (Не удается создать каталог: {path}. Укажите допустимый путь).|


## <a name="error-0047"></a>Ошибка 0047  
 Исключение возникает, если количество столбцов признаков в некоторых наборах данных, переданных модулю, слишком мало.  

 Эта ошибка в Машинном обучении Azure возникает, если входной набор данных для обучения не содержит минимальное число столбцов, необходимое для алгоритма. Обычно либо набор данных пуст, либо содержит только столбцы для обучения.  

**Решение.** Повторно перейдите к входному набору данных, чтобы убедиться в наличии одного или нескольких дополнительных столбцов, помимо столбца меток.  

|Сообщения об исключении|
|------------------------|
|Number of feature columns in input dataset is less than allowed minimum (Число столбцов признаков во входном наборе данных меньше допустимого минимального значения).|
|Number of feature columns in input dataset is less than allowed minimum of {required_columns_count} column(s) (Число столбцов признаков во входном наборе данных меньше допустимого минимального значения {required_columns_count}).|
|Number of feature columns in input dataset "{arg_name}" is less than allowed minimum of {required_columns_count} column(s) (Число столбцов признаков во входном наборе данных {arg_name} меньше допустимого минимального значения {required_columns_count}).|


## <a name="error-0048"></a>Ошибка 0048  
 Исключение возникает в случае, если не удается открыть файл.  

 Эта ошибка в Машинном обучении Azure возникает, когда невозможно открыть файл для чтения или записи. Эта ошибка может возникнуть по следующим причинам:  

-   Контейнер или файл (большой двоичный объект) не существует.  
  
-   Уровень доступа к файлу или контейнеру не позволяет получить доступ к файлу.  
  
-   Файл слишком велик для чтения или имеет неправильный формат.  

**Решение.** Повторно откройте модуль и файл, который вы пытаетесь прочитать.  

 Проверьте правильность имен контейнера и файла.  

 Используйте классический портал Azure или средство хранилища Azure, чтобы убедиться, что у вас есть разрешение на доступ к файлу.  

  <!--If you are trying to read an image file, make sure that it meets the requirements for image files in terms of size, number of pixels, and so forth. For more information, see [Import Images](import-images.md).  -->

|Сообщения об исключении|
|------------------------|
|Unable to open a file (Не удалось открыть файл).|
|Error while opening the file: {file_name} (Ошибка при открытии файла: {file_name}).|
|Error while opening the file: {file_name}. Storage exception message: {exception}. (Ошибка при открытии файла: {file_name}. Сообщение об исключении хранилища: {exception}).|


## <a name="error-0049"></a>Ошибка 0049  
 Исключение возникает в случае, если не удается выполнить анализ файла.  

 Эта ошибка в Машинном обучении Azure возникает, когда невозможно выполнить анализ файла. Эта ошибка возникает, если формат файла, выбранный в модуле [импорта данных](import-data.md), не соответствует реальному формату файла или если файл содержит нераспознаваемый символ.  

**Решение.** Повторно откройте модуль и исправьте выбор формата файла, если он не соответствует формату файла. При возможности проверьте файл, чтобы убедиться, что он не содержит недопустимых символов.  

|Сообщения об исключении|
|------------------------|
|Unable to parse a file (Не удалось выполнить анализ файла).|
|Error while parsing the {file_format} file (Ошибка при анализе файла {file_format}).|
|Error while parsing the {file_format} file: {file_name} (Ошибка при анализе файла {file_format}: {file_name}).|
|Error while parsing the {file_format} file. Reason: {failure_reason}. (Ошибка при синтаксическом анализе файла {file_format}. Причина: {failure_reason}).|
|Error while parsing the {file_format} file: {file_name}. Reason: {failure_reason}. (Ошибка при синтаксическом анализе файла {file_format}: {file_name}. Причина: {failure_reason}).|


## <a name="error-0052"></a>Ошибка 0052  
 Исключение возникает, если ключ учетной записи хранилища Azure указан неправильно.  

 Эта ошибка в Машинном обучении Azure возникает, если ключ, используемый для доступа к учетной записи хранения Azure, неправильный. Например, эта ошибка может возникать, если ключ хранилища Azure был усечен при копировании и вставке, а также при использовании неправильного ключа.  

 Дополнительные сведения о том, как получить ключ для учетной записи хранения Azure, см. в статье [Создание учетной записи хранения Azure](../../storage/common/storage-account-create.md).  

**Решение.** Повторно откройте модуль и убедитесь, что ключ к хранилищу Azure указан правильно для учетной записи. При необходимости скопируйте ключ на классическом портале Azure.  

|Сообщения об исключении|
|------------------------|
|The Azure storage account key is incorrect (Неверный ключ учетной записи хранилища Azure).|


## <a name="error-0053"></a>Ошибка 0053  
 Исключение возникает, если отсутствуют признаки пользователей или элементы для рекомендаций подсистемы Matchbox.  

 Эта ошибка в Машинном обучении Azure возникает, если не удается найти вектор признаков.  

**Решение.** Убедитесь, что во входном наборе данных содержится вектор признаков.  

|Сообщения об исключении|
|------------------------|
|User features or/and items are required but not provided (Не предоставлены обязательные признаки пользователя или элементы).|


## <a name="error-0056"></a>Ошибка 0056  
 Исключение возникает, если выбранные для операции столбцы нарушают требования.  

 Эта ошибка в Машинном обучении Azure возникает при выборе столбцов для операции, для которой требуется, чтобы столбец был определенного типа данных. 

 Эта ошибка также может возникать, если столбец имеет правильный тип данных, но для модуля, который вы используете, требуется, чтобы столбец был также помечен как признак, метка или столбец категорий.  

  <!--For example, the [Convert to Indicator Values](convert-to-indicator-values.md) module requires that columns be categorical, and will raise this error if you select a feature column or label column.  -->

**Решение.**

1.  Проверьте тип данных столбцов, которые выбраны в данный момент. 

2. Проверьте, являются ли выбранные столбцы столбцами категорий, меток или признаков.  
  
3.  Изучите раздел справки для модуля, в котором сделан выбор столбцов, чтобы определить, существуют ли особые требования для использования типа данных или столбца.  
  
3.  Используйте модуль [изменения метаданных](edit-metadata.md), чтобы изменить тип столбца на время выполнения операции. Не забудьте изменить тип столбца обратно на его исходное значение, используя другой экземпляр модуля [изменения метаданных](edit-metadata.md), если он необходим для выполнения последующих операций.  

|Сообщения об исключении|
|------------------------|
|One or more selected columns were not in an allowed category (Один или несколько выделенных столбцов принадлежали неразрешенной категории).|
|Column with name "{col_name}" is not in an allowed category (Столбец с именем {col_name} принадлежит недопустимой категории).|


## <a name="error-0057"></a>Ошибка 0057  
 Исключение возникает при попытке создать уже существующий файл или большой двоичный объект.  

 Это исключение возникает, если вы используете модуль [экспорта данных](export-data.md) или другой модуль для сохранения результатов конвейера в Машинном обучении Azure в хранилище BLOB-объектов Azure, но пытаетесь создать уже существующий файл или большой двоичный объект.   

**Решение.**

 Эта ошибка возникает только в том случае, если ранее для параметра **Azure blob storage write mode** (Режим записи BLOB-объектов Azure) было задано значение **Error** (Ошибка). Этот модуль выдает ошибку при попытке записать набор данных в уже существующий большой двоичный объект.

 - Откройте свойства модуля и измените свойство **Azure blob storage write mode** (Режим записи BLOB-объектов Azure) на **Overwrite** (Перезаписать).
 - Кроме того, можно ввести имя другого целевого большого двоичного объекта или файла и указать большой двоичный объект, который еще не существует.  

|Сообщения об исключении|
|------------------------|
|File or Blob already exists (Файл или большой двоичный объект уже существует).|
|File or Blob "{file_path}" already exists (Файл или большой двоичный объект {file_path} уже существует).|


## <a name="error-0058"></a>Ошибка 0058  
 Эта ошибка в Машинном обучении Azure возникает, если набор данных не содержит ожидаемого столбца меток.  

 Это исключение также может возникать, если указанный столбец меток не соответствует данным или типам данных, ожидаемым учеником, или имеет неверные значения. Например, это исключение создается, если при обучении двоичного классификатора используется столбец меток с реальным значением.  

**Решение.** Решение зависит от выбранного ученика или модуля обучения, а также от типов данных столбцов в наборе данных. Сначала проверьте требования к алгоритму машинного обучения или модулю обучения.  

 Повторно откройте набор входных данных. Убедитесь, что столбец, который должен обрабатываться как метка, имеет правильный тип данных для создаваемой модели.  

 Проверьте отсутствующие значения во входных данных и исключите или замените их при необходимости.  

 При необходимости добавьте модуль [изменения метаданных](edit-metadata.md) и убедитесь, что столбец метки помечен как метка.  

|Сообщения об исключении|
|------------------------|
|The label column values and scored label column values are not comparable (Значения столбцов меток и вычисляемых столбцов меток не являются сравнимыми).|
|The label column is not as expected in "{dataset_name}" (Столбец меток отличается от ожидаемого в {dataset_name}).|
|The label column is not as expected in "{dataset_name}", {reason} (Столбец меток отличается от ожидаемого в {dataset_name}, {reason}).|
|The label column "{column_name}" is not expected in "{dataset_name}" (Столбец меток {column_name} не ожидается в {dataset_name}).|
|The label column "{column_name}" is not expected in "{dataset_name}", {reason} (Столбец меток {column_name} не ожидается в {dataset_name}, {reason}).|


## <a name="error-0059"></a>Ошибка 0059  
 Исключение возникает, если не удается проанализировать индекс столбца, указанный в средстве выбора столбцов.  

 Эта ошибка в Машинном обучении Azure возникает, если не удается выполнить синтаксический анализ индекса столбца, указанного при использовании селектора столбцов.  Эта ошибка возникает, если индекс столбца имеет недопустимый формат, который не удается проанализировать.  

**Решение.** Измените индекс столбца, чтобы использовать допустимое значение индекса.  

|Сообщения об исключении|
|------------------------|
|One or more specified column indexes or index ranges could not be parsed (Не удалось проанализировать один или несколько индексов столбца или диапазонов индекса).|
|Column index or range "{column_index_or_range}" could not be parsed (Не удалось проанализировать индекс столбца или диапазон {column_index_or_range}).|


## <a name="error-0060"></a>Ошибка 0060  
 Исключение возникает, если в средстве выбора столбцов указан ряд столбцов вне диапазона.  

 Эта ошибка в Машинном обучении Azure возникает, если диапазон столбцов вне допустимого диапазона указан в селекторе столбцов. Эта ошибка возникает, если диапазон столбцов в селекторе столбцов не соответствует столбцам в наборе данных.  

**Решение.** Измените диапазон столбцов в средстве выбора столбцов в соответствии со столбцами в наборе данных.  

|Сообщения об исключении|
|------------------------|
|Invalid or out of range column index range specified (Указан недопустимый индекс столбца или индекс выходит за диапазон).|
|Column range "{column_range}" is invalid or out of range (Диапазон столбцов {column_range} недопустим или выходит за пределы допустимого диапазона).|


## <a name="error-0061"></a>Ошибка 0061  
 Исключение возникает при попытке добавить строку в объект DataTable, количество столбцов в котором отличается от количества столбцов в таблице.  

 Эта ошибка в Машинном обучении Azure возникает при попытке добавить строку в набор данных, который имеет другое количество столбцов, чем набор данных.  Эта ошибка возникает, если строка, добавляемая в набор данных, имеет другое количество столбцов, чем входной набор данных.  Строку нельзя добавить к набору данных, если число столбцов отличается.  

**Решение.** Измените входной набор данных так, чтобы количество столбцов было таким же, как и в добавленной строке, или измените добавленную строку, чтобы она содержала то же количество столбцов, что и набор данных.  

|Сообщения об исключении|
|------------------------|
|All tables must have the same number of columns (Количество столбцов во всех таблицах должно быть одинаковым).|
|Columns in chunk "{chunk_id_1}" is different with chunk "{chunk_id_2}" with chunk size: {chunk_size} (Столбцы в блоке {chunk_id_1} отличаются от блока {chunk_id_2} с размером блока: {chunk_size}).|
|Column count in file "{filename_1}" (count={column_count_1}) is different with file "{filename_2}" (count={column_count_2}) (Число столбцов в файле {filename_1} (count={column_count_1}) отличается от файла {filename_2} (count={column_count_2})).|


## <a name="error-0062"></a>Ошибка 0062  
 Исключение возникает при попытке сравнить две модели с различными типами учеников.  

 Эта ошибка в Машинном обучении Azure возникает, если не удается сравнить метрики оценки двух разных оценочных наборов данных. В этом случае невозможно сравнить эффективность моделей, используемых для создания двух оценочных наборов данных.  

**Решение.** Убедитесь, что результаты оценки создаются тем же типом модели машинного обучения (двоичная классификация, регрессия, многоклассовая классификация, рекомендация, кластеризация, обнаружение аномалий и т. д.). Все сопоставляемые модели должны использовать один тип ученика.  

|Сообщения об исключении|
|------------------------|
|All models must have the same learner type (Все модели должны использовать один тип ученика).|
|Got incompatible learner type: "{actual_learner_type}". Expected learner types are: "{expected_learner_type_list}" (Получен несовместимый тип ученика: {actual_learner_type}. Ожидаемые типы ученика: {expected_learner_type_list}).|


## <a name="error-0064"></a>Ошибка 0064  
 Исключение возникает, если имя учетной записи хранилища Azure или ключ хранилища указаны неправильно.  

 Эта ошибка в Машинном обучении Azure возникает, если имя учетной записи хранения Azure или ключ хранилища указаны неправильно. Эта ошибка возникает при вводе неправильного имени учетной записи или пароля для учетной записи хранения. Это может произойти, если ввести имя или пароль учетной записи вручную. Это также может произойти, если учетная запись была удалена.  

**Решение.** Убедитесь, что имя учетной записи и пароль введены правильно и что контейнер существует.  

|Сообщения об исключении|
|------------------------|
|The Azure storage account name or storage key is incorrect (Неверное имя учетной записи хранилища или ключа хранилища Azure).|
|The Azure storage account name "{account_name}" or storage key for the account name is incorrect (Неверное имя учетной записи хранилища Azure {account_name} или ключа хранилища для имени учетной записи).|


## <a name="error-0065"></a>Ошибка 0065  
 Исключение возникает, если имя большого двоичного объекта Azure указано неправильно.  

 Эта ошибка в Машинном обучении Azure возникает, если имя большого двоичного объекта указано неправильно.  Вы получите ошибку, если:  

-   В указанном контейнере не удается найти большой двоичный объект.  

 <!---   The fully qualified name of the blob specified for output in one of the [Learning with Counts](data-transformation-learning-with-counts.md) modules is greater than 512 characters.  -->

-   Только контейнер был указан в качестве источника в запросе модуля [импорта данных](import-data.md), когда формат был Excel или CSV с кодировкой. Объединение содержимого всех больших двоичных объектов в контейнере не допускается с этими форматами.  
  
-   URI SAS не содержит имя допустимого большого двоичного объекта.  

**Решение.** Повторно откройте модуль, вызывающий исключение. Убедитесь, что указанный большой двоичный объект существует в контейнере в учетной записи хранения и разрешения позволяют его просматривать. Убедитесь, что входные данные имеют форму **имя контейнера/имя файла**, если у вас есть файл Excel или CSV с форматами кодирования. Убедитесь, что URI SAS содержит имя допустимого большого двоичного объекта.  

|Сообщения об исключении|
|------------------------|
|The Azure storage blob name is incorrect (Неверное имя большого двоичного объекта хранилища Azure).|
|The Azure storage blob name "{blob_name}" is incorrect (Неверное имя {blob_name} большого двоичного объекта хранилища Azure).|
|The Azure storage blob name with prefix "{blob_name_prefix}" does not exist (Имя большого двоичного объекта службы хранилища Azure с префиксом {blob_name_prefix} не существует).|
|Failed to find any Azure storage blobs under container "{container_name}" (Не удалось найти большие двоичные объекты службы хранилища Azure в контейнере {container_name}).|
|Failed to find any Azure storage blobs with wildcard path "{blob_wildcard_path}" (Не удалось найти большие двоичные объекты службы хранилища Azure с путем с подстановочными знаками {blob_wildcard_path}).|


## <a name="error-0066"></a>Ошибка 0066  
 Исключение возникает, если не удалось загрузить ресурс в большой двоичный объект Azure.  

 Эта ошибка в Машинном обучении Azure возникает, если не удалось загрузить ресурс в большой двоичный объект Azure.  <!--You will receive this message if [Train Vowpal Wabbit 7-4 Model](train-vowpal-wabbit-version-7-4-model.md) encounters an error attempting to save either the model or the hash created when training the model.--> Оба сохраняются в той же учетной записи хранения Azure, что и учетная запись, содержащая входной файл.  

**Решение.** Повторно откройте модуль. Убедитесь, что имя учетной записи Azure, ключ хранилища и контейнер указаны правильно, а учетная запись имеет разрешение на запись в контейнер.  

|Сообщения об исключении|
|------------------------|
|The resource could not be uploaded to Azure storage (Не удалось отправить ресурс в хранилище Azure).|
|The file "{source_path}" could not be uploaded to Azure storage as "{dest_path}" (Не удалось отправить файл {source_path} в службу хранилища Azure как {dest_path}).|


## <a name="error-0067"></a>Ошибка 0067  
 Исключение возникает, если количество столбцов в наборе данных отличается от ожидаемого.  

 Эта ошибка в Машинном обучении Azure возникает, если набор данных имеет другое количество столбцов, чем ожидалось.  Эта ошибка возникает, когда число столбцов в наборе данных отличается от числа столбцов, которые модуль ожидает во время выполнения.  

**Решение.** Измените входной набор данных или параметры.  

|Сообщения об исключении|
|------------------------|
|Unexpected number of columns in the datatable (Непредвиденное количество столбцов в объекте datatable).|
|Unexpected number of columns in the dataset "{dataset_name}" (Непредвиденное количество столбцов в наборе данных {dataset_name}).|
|Expected "{expected_column_count}" column(s) but found "{actual_column_count}" column(s) instead (Ожидалось {expected_column_count} столбцов, но вместо этого обнаружены {actual_column_count} столбцов).|
|In input dataset "{dataset_name}", expected "{expected_column_count}" column(s) but found "{actual_column_count}" column(s) instead (Во входном наборе данных {dataset_name} ожидались {expected_column_count} столбцов, но вместо этого обнаружены {actual_column_count} столбцов).|


## <a name="error-0068"></a>Ошибка 0068  
 Исключение возникает, если указан неправильный скрипт Hive.  

 Эта ошибка в Машинном обучении Azure возникает, если в скрипте Hive QL есть синтаксические ошибки или если интерпретатор Hive обнаруживает ошибку при выполнении запроса или скрипта.  

**Решение.**

Сообщение об ошибке из Hive обычно возвращается в журнал ошибок, так что вы можете принять меры на основе конкретной ошибки. 

+ Откройте модуль и проверьте запрос на наличие ошибок.  
+ Убедитесь, что запрос работает правильно вне Машинного обучения Azure, войдя в консоль Hive кластера Hadoop и выполнив запрос.  
+ Попробуйте разместить комментарии в скрипте Hive в отдельной строке, а не смешивать исполняемые инструкции и комментарии в одной строке.  

### <a name="resources"></a>Ресурсы

Дополнительные сведения о запросах Hive для машинного обучения см. в следующих статьях:

+ [Создание таблиц Hive и загрузка данных из хранилища BLOB-объектов Azure](../team-data-science-process/move-hive-tables.md)
+ [Просмотр данных в таблицах Hive с помощью запросов Hive](../team-data-science-process/explore-data-hive-tables.md)
+ [Создание характеристик для данных в кластере Hadoop с помощью запросов Hive](../team-data-science-process/create-features-hive.md)
+ [Памятка по Hive для пользователей SQL (PDF)](http://hortonworks.com/wp-content/uploads/2013/05/hql_cheat_sheet.pdf)

  
|Сообщения об исключении|
|------------------------|
|Hive script is incorrect (Неверный скрипт Hive).|


## <a name="error-0069"></a>Ошибка 0069  
 Исключение возникает, если указан неправильный скрипт SQL.  

 Эта ошибка в Машинном обучении Azure возникает, если указанный скрипт SQL содержит синтаксические ошибки или если столбцы или таблицы, указанные в скрипте, недопустимы. 

 Эта ошибка возникает, если ядро SQL обнаруживает ошибку при выполнении запроса или скрипта. Сообщение об ошибке из SQL обычно возвращается в журнал ошибок, так что вы можете принять меры на основе конкретной ошибки.  

**Решение.** Повторно откройте модуль и проверьте запрос SQL на наличие ошибок.  

 Убедитесь, что запрос работает правильно за пределами Машинного обучения Azure, выполнив вход непосредственно на сервер базы данных и выполнив запрос.  

 Если в исключении модуля сообщается о сформированном сообщении SQL, примите меры на основе сообщенной ошибки. Например, сообщения об ошибках иногда включают в себя определенные рекомендации по наиболее вероятной ошибке:
+ Ошибка *No such column or missing database* (Нет такого столбца или базы данных) означает, что вы могли указать неправильное имя столбца. Если вы уверены, что имя столбца указано правильно, попробуйте использовать квадратные скобки или кавычки для заключения идентификатора столбца.
+ *Ошибка логики SQL \<SQL keyword\> рядом* с тем, что может иметь синтаксическую ошибку перед указанным ключевым словом

  
|Сообщения об исключении|
|------------------------|
|SQL script is incorrect (Неверный скрипт SQL).|
|SQL query "{sql_query}" is not correct (SQL-запрос {sql_query} неверен).|
|SQL query "{sql_query}" is not correct. Exception message: {exception}. (SQL-запрос {sql_query} неверен. Сообщение об исключении: {exception}).|


## <a name="error-0070"></a>Ошибка 0070  
 Исключение возникает при попытке доступа к несуществующей таблице Azure.  

 Эта ошибка в Машинном обучении Azure возникает при попытке доступа к несуществующей таблице Azure. Эта ошибка возникает, если указать в службе хранилища Azure таблицу, которой не существует, при чтении из хранилища таблиц Azure или записи в него. Это может произойти в случае неправильного ввода имени нужной таблицы или несоответствия между именем целевого объекта и типом хранилища. Например, вы предполагали считывать данные из таблицы, но указали вместо этого имя большого двоичного объекта.  

**Решение.** Повторно откройте модуль, чтобы убедиться в правильности имени таблицы.  

|Сообщения об исключении|
|------------------------|
|Azure table does not exist (Таблица Azure не существует).|
|Azure table "{table_name}" does not exist (Таблица Azure {table_name} не существует).|


## <a name="error-0072"></a>Ошибка 0072  
 Исключение возникает в случае истечения времени ожидания соединения.  

 Эта ошибка в Машинном обучении Azure возникает при истечении времени ожидания соединения. Эта ошибка возникает при наличии проблем с подключением к источнику или расположению данных, например при плохом подключении к Интернету или если набор данных имеет большой размер, или SQL-запрос, предназначенный для чтения данных, выполняет сложную обработку.  

**Решение.** Определите, имеются ли в настоящее время проблемы с подключением к службе хранилища Azure или к Интернету.  

|Сообщения об исключении|
|------------------------|
|Connection timeout occurred (Истекло время ожидания подключения).|


## <a name="error-0073"></a>Ошибка 0073  
 Исключение возникает при возникновении ошибки во время преобразования столбца в другой тип.  

 Эта ошибка в Машинном обучении Azure возникает, когда невозможно преобразовать столбец в другой тип.  Эта ошибка возникает, если для модуля требуется определенный тип и невозможно преобразовать столбец в новый тип.  

**Решение.** Измените входной набор данных так, чтобы столбец можно было преобразовать на основе внутреннего исключения.  

|Сообщения об исключении|
|------------------------|
|Failed to convert column (Не удалось преобразовать столбец).|
|Failed to convert column to {target_type} (Не удалось преобразовать столбец в {target_type}).|


## <a name="error-0075"></a>Ошибка 0075  
Исключение возникает, если при квантовании набора данных используется недопустимая функция классификации.  

Эта ошибка в Машинном обучении Azure возникает при попытке размещения данных с помощью неподдерживаемого метода или при недопустимых сочетаниях параметров.  

**Решение.**

Обработка ошибок для этого события была представлена в более ранней версии Машинного обучения Azure, которая расширяла возможности настроек методов квантования. Сейчас все методы квантования основаны на выборе из раскрывающегося списка, поэтому технически в них не должна возникать эта ошибка.

 <!--If you get this error when using the [Group Data into Bins](group-data-into-bins.md) module, consider reporting the issue in the [Microsoft Q&A question page for Azure Machine Learning](/answers/topics/azure-machine-learning-studio-classic.html), providing the data types, parameter settings, and the exact error message.  -->

|Сообщения об исключении|
|------------------------|
|Invalid binning function used (Используется недопустимая функция сегментирования).|


## <a name="error-0077"></a>Ошибка 0077  
 Исключение возникает при передаче режима записи неизвестного файла большого двоичного объекта.  

 Эта ошибка в Машинном обучении Azure возникает, если в спецификациях назначения или источника файла большого двоичного объекта передан недопустимый аргумент.  

**Решение.** Почти во всех модулях, которые импортируют или экспортируют данные в хранилище BLOB-объектов Azure и из него, значения параметров, управляющих режимом записи, назначаются с помощью раскрывающегося списка. Таким образом, невозможно передать недопустимое значение, и эта ошибка не должна появляться. Эта ошибка будет удалена в более позднем выпуске.  

|Сообщения об исключении|
|------------------------|
|Unsupported blob write mode (Режим записи большого двоичного объекта не поддерживается).|
|Unsupported blob write mode: {blob_write_mode} (Режим записи большого двоичного объекта {blob_write_mode} не поддерживается).|


## <a name="error-0078"></a>Ошибка 0078  
 Исключение возникает, когда параметр HTTP для модуля [импорта данных](import-data.md) получает код состояния 3xx, указывающий перенаправление.  

 Эта ошибка в Машинном обучении Azure возникает, когда параметр HTTP для модуля [импорта данных](import-data.md) получает код состояния 3XX (301, 302, 304 и т. д.), указывающий на перенаправление. Эта ошибка возникает при попытке подключиться к источнику HTTP, который перенаправляет браузер на другую страницу. По соображениям безопасности перенаправление веб-сайтов не допускается в качестве источников данных для Машинного обучения Azure.  

**Решение.** Если веб-сайт является доверенным, введите URL-адрес перенаправления напрямую.  

|Сообщения об исключении|
|------------------------|
|Http redirection not allowed (Перенаправление HTTP не разрешено).|


## <a name="error-0079"></a>Ошибка 0079  
 Исключение возникает, если имя контейнера хранилища Azure указано неверно.  

 Эта ошибка в Машинном обучении Azure возникает, если имя контейнера хранилища Azure указано неверно. Эта ошибка возникает, если вы не указали имя контейнера и большого двоичного объекта (файла), используя параметр **Path to blob beginning with container** (Путь к большому двоичному объекту, начинающемуся с контейнера), при записи в хранилище BLOB-объектов Azure.  

**Решение.** Повторно откройте модуль [экспорта данных](export-data.md) и убедитесь, что указанный путь к большому двоичному объекту содержит как контейнер, так и имя файла в формате **контейнер/имя файла**.  

|Сообщения об исключении|
|------------------------|
|The Azure storage container name is incorrect (Неверное имя контейнера хранилища Azure).|
|The Azure storage container name "{container_name}" is incorrect; a container name of the format container/blob was expected (Неверное имя контейнера хранилища Azure {container_name}. Ожидалось имя контейнера или большого двоичного объекта).|


## <a name="error-0080"></a>Ошибка 0080  
 Исключение возникает, если в модуле не допускается использование столбца со всеми недостающими значениями.  

 Эта ошибка возникает в Машинном обучении Azure, если один или несколько столбцов, используемых модулем, содержат все отсутствующие значения. Например, если модуль вычисляет статистические данные для каждого столбца, он не может обрабатывать столбец, не содержащий данные. В таких случаях выполнение модуля прерывается с помощью этого исключения.  

**Решение.** Повторно откройте входной набор данных и удалите все столбцы, содержащие все пропущенные значения.  

|Сообщения об исключении|
|------------------------|
|Columns with all values missing are not allowed (Столбцы со всеми отсутствующими значениями не допускаются).|
|Column {col_index_or_name} has all values missing (В столбце {col_index_or_name} отсутствуют все значения).|


## <a name="error-0081"></a>Ошибка 0081  
 Исключение возникает в модуле PCA, если количество измерений, до которого нужно выполнить сокращение, равно количеству столбцов компонентов во входном наборе данных, содержащем по крайней мере один разреженный столбец компонентов.  

 Эта ошибка в Машинном обучении Azure возникает, если выполняются следующие условия: (а) входной набор данных содержит по крайней мере один разреженный столбец и (б) конечное число запрашиваемых измерений совпадает с числом входных измерений.  

**Решение.** Рекомендуется уменьшить число измерений в выходных данных, чтобы оно было меньше числа измерений во входных данных. Это распространено в приложениях PCA.   <!--For more information, see [Principal Component Analysis](principal-component-analysis.md).  -->

|Сообщения об исключении|
|------------------------|
|For dataset containing sparse feature columns number of dimensions to reduce to should be less than number of feature columns (Для набора данных, содержащего столбцы разреженных признаков, число размерностей для сокращения должно быть меньше количества столбцов признаков).|


## <a name="error-0082"></a>Ошибка 0082  
 Исключение возникает, если не удается успешно десериализовать модель.  

 Эта ошибка в Машинном обучении Azure возникает, когда сохраненная модель машинного обучения или преобразование не могут быть загружены более новой версией среды выполнения Машинного обучения Azure в результате критического изменения.  

**Решение.** Обучающий конвейер, который создал модель или преобразование, следует запустить повторно, а модель или преобразование необходимо повторно сохранить.  

|Сообщения об исключении|
|------------------------|
|Модель не может быть десериализована, так как скорее всего она сериализована с использованием старого формата. Повторите обучение и повторно сохраните модель.|


## <a name="error-0083"></a>Ошибка 0083  
 Исключение возникает, если набор данных, используемый для обучения, не может применяться для определенного типа ученика.  

 Эта ошибка в Машинном обучении Azure возникает, если набор данных несовместим с учеником. Например, набор данных может содержать по крайней мере одно отсутствующее значение в каждой строке, и в результате весь набор данных будет пропущен во время обучения. В других случаях некоторые алгоритмы машинного обучения, такие как обнаружение аномалий, не предполагают присутствие меток и могут вызывать это исключение, если в наборе данных есть метки.  

**Решение.** Просмотрите документацию по ученику для проверки требований к входному набору данных. Изучите столбцы, чтобы увидеть, что все необходимые столбцы присутствуют.  

|Сообщения об исключении|
|------------------------|
|Dataset used for training is invalid (Для обучения используется недопустимый набор данных).|
|{data_name} contains invalid data for training ({data_name} содержит недопустимые данные для обучения).|
|{data_name} contains invalid data for training. Learner type: {learner_type} ({data_name} содержит недопустимые данные для обучения. Тип ученика: {learner_type}).|
|{data_name} contains invalid data for training. Learner type: {learner_type} ({data_name} содержит недопустимые данные для обучения. Тип ученика: {learner_type}). Reason: {reason}. Причина: {reason}.|
|Failed to apply "{action_name}" action on training data {data_name} (Не удалось применить действие {action_name} к данным обучения {data_name}). Reason: {reason}. Причина: {reason}.|


## <a name="error-0084"></a>Ошибка 0084  
 Исключение возникает, когда обрабатываются оценки, полученные из сценария R. Это в настоящее время не поддерживается.  

 Эта ошибка в Машинном обучении Azure возникает при попытке использовать один из модулей для оценки модели с выходными данными из скрипта R, содержащего оценки.  

**Решение.**

|Сообщения об исключении|
|------------------------|
|Evaluating scores produced by Custom Model is currently unsupported (Обработка оценок, созданных настраиваемой моделью, в настоящее время не поддерживается).|


## <a name="error-0085"></a>Ошибка 0085  
 Исключение возникает, если оценка скрипта завершается ошибкой.  

 Эта ошибка в Машинном обучении Azure возникает при выполнении пользовательского скрипта, содержащего синтаксические ошибки.  

**Решение.** Просмотрите код во внешнем редакторе и проверьте наличие ошибок.  

|Сообщения об исключении|
|------------------------|
|Error during evaluation of script (Ошибка во время вычисления скрипта).|
|The following error occurred during script evaluation, please view the output log for more information (Во время вычисления скрипта произошла следующая ошибка, дополнительные сведения см. в выходном файле журнала).<br />---------- Start of error message from {script_language} interpreter ---------- (Начало сообщения об ошибке из интерпретатора {script_language})<br />{message}<br />---------- End of error message from {script_language}  interpreter ----------(Конец сообщения об ошибке из интерпретатора {script_language})|


## <a name="error-0090"></a>Ошибка 0090  
 Исключение возникает, если создание таблицы Hive завершается сбоем.  

 Эта ошибка в Машинном обучении Azure возникает, если вы используете модуль [экспорта данных](export-data.md) или другой параметр для сохранения данных в кластере HDInsight и не удается создать указанную таблицу Hive.  

**Решение.** Проверьте имя учетной записи хранения Azure, связанной с кластером, и убедитесь, что вы используете ту же учетную запись в свойствах модуля.  

|Сообщения об исключении|
|------------------------|
|The Hive table could not be created. For a HDInsight cluster, please ensure the Azure storage account name associated with cluster is the same as what is passed in through the module parameter (Не удалось создать таблицу Hive. Для кластера HDInsight убедитесь, что имя учетной записи хранения Azure, связанной с кластером, совпадает с именем, передаваемым через параметр модуля).|
|The Hive table "{table_name}" could not be created. For a HDInsight cluster, please ensure the Azure storage account name associated with cluster is the same as what is passed in through the module parameter (Не удалось создать таблицу Hive. Для кластера HDInsight убедитесь, что имя учетной записи хранения Azure, связанной с кластером, совпадает с именем, передаваемым через параметр модуля).|
|The Hive table "{table_name}" could not be created. For a HDInsight cluster, please ensure the Azure storage account name associated with cluster is "{cluster_name}". (Не удалось создать таблицу Hive {table_name}. Для кластера HDInsight убедитесь, что имя учетной записи хранения Azure, связанной с кластером, — {cluster_name}).|


## <a name="error-0102"></a>Ошибка 0102  
 Возникает, если не удается извлечь ZIP-файл  

 Эта ошибка в Машинном обучении Azure возникает, когда вы импортируете сжатый пакет с расширением ZIP, но пакет либо не является ZIP-файлом, либо файл не использует поддерживаемый формат ZIP.  

**Решение.** Убедитесь, что выбранный файл является допустимым ZIP-файлом и был сжат с помощью одного из поддерживаемых алгоритмов сжатия.  

 Если при импорте наборов данных в сжатом формате возникает эта ошибка, убедитесь, что все автономные файлы используют один из поддерживаемых форматов файлов и имеют формат Юникода.  <!--For more information, see [Unpack Zipped Datasets](unpack-zipped-datasets.md).  -->

 Попробуйте прочитать нужные файлы в новой сжатой ZIP-папке и добавьте пользовательский модуль еще раз.  

|Сообщения об исключении|
|------------------------|
|Given ZIP file is not in the correct format (Неверный формат ZIP-файла).|


## <a name="error-0105"></a>Ошибка 0105  
 Эта ошибка отображается, если файл определения модуля содержит неподдерживаемый тип параметра  
  
 Эта ошибка в Машинном обучении Azure возникает при создании пользовательского XML-определения модуля, а тип параметра или аргумента в определении не соответствует поддерживаемому типу.  
  
**Решение.** Убедитесь, что свойство типа любого элемента **Arg** в файле XML-определения пользовательского модуля поддерживается.  
  
|Сообщения об исключении|  
|------------------------|  
|Unsupported parameter type (Неподдерживаемый тип параметра).|  
|Unsupported parameter type '{0}' specified (Указан неподдерживаемый тип параметра {0}).|  


## <a name="error-0107"></a>Ошибка 0107  
 Возникает, когда файл определения модуля задает неподдерживаемый тип выходных данных.  
  
 Эта ошибка в Машинном обучении Azure возникает, когда тип выходного порта в XML-определении пользовательского модуля не соответствует поддерживаемому типу.  
  
**Решение.** Убедитесь, что свойство типа любого элемента вывода в файле XML-определения пользовательского модуля поддерживается.  
  
|Сообщения об исключении|  
|------------------------|  
|Unsupported output type (Неподдерживаемый тип выходных данных).|  
|Unsupported output type '{output_type}' specified (Указан неподдерживаемый тип выходных данных {output_type}).|  


## <a name="error-0125"></a>Ошибка 0125  
 Возникает при несоответствии схем нескольких наборов данных.  

**Решение.**

|Сообщения об исключении|
|------------------------|
|Dataset schema does not match (Несоответствие схемы набора данных).|


## <a name="error-0127"></a>Ошибка 0127  
 Размер изображения в пикселях превышает допустимый предел  

 Эта ошибка возникает при чтении изображений из набора данных изображений для классификации, если размер изображений больше, чем модель может обработать.  

 <!--**Resolution:**
 For more information about the image size and other requirements, see these topics:  

-   [Import Images](import-images.md)  
  
-   [Pretrained Cascade Image Classification](pretrained-cascade-image-classification.md)  -->

|Сообщения об исключении|
|------------------------|
|Image pixel size exceeds allowed limit (Размер изображения в пикселях превышает допустимый предел).|
|Image pixel size in the file '{file_path}' exceeds allowed limit: '{size_limit}' (Размер изображения в пикселях в файле {file_path} превышает допустимый предел {size_limit}).|


## <a name="error-0128"></a>Ошибка 0128  
 Количество условных вероятностей столбцов категории превышает ограничение.  

**Решение.**

|Сообщения об исключении|
|------------------------|
|Number of conditional probabilities for categorical columns exceeds limit (Количество условных вероятностей столбцов категории превышает ограничение).|
|Number of conditional probabilities for categorical columns exceeds limit. Columns '{column_name_or_index_1}' and '{column_name_or_index_2}' are the problematic pair (Количество условных вероятностей для столбцов категории превышает ограничение. Столбцы {column_name_or_index_1} и {column_name_or_index_2} являются проблематичной парой).|


## <a name="error-0129"></a>Ошибка 0129  
 Number of columns in the dataset exceeds allowed limit (Число столбцов в наборе данных превышает допустимый предел).  

**Решение.**

|Сообщения об исключении|
|------------------------|
|Number of columns in the dataset exceeds allowed limit (Число столбцов в наборе данных превышает допустимый предел).|
|Number of columns in the dataset in '{dataset_name}' exceeds allowed (Число столбцов в наборе данных в {dataset_name} превышает допустимый предел).|
|Number of columns in the dataset in '{dataset_name}' exceeds allowed limit of '{component_name}' (Число столбцов в наборе данных в {dataset_name} превышает допустимый предел {component_name}).|
|Number of columns in the dataset in '{dataset_name}' exceeds allowed '{limit_columns_count}' limit of '{component_name}' (Число столбцов в наборе данных в {dataset_name} превышает допустимое значение {limit_columns_count} ({component_name})).|


## <a name="error-0134"></a>Ошибка 0134
Исключение возникает, если столбец метки отсутствует или имеет недостаточное количество помеченных строк.  

Эта ошибка возникает, когда модулю требуется столбец меток, но вы не включили его при выборе столбца или в столбце метки отсутствует слишком много значений.

Эта ошибка также может возникнуть, когда предыдущая операция изменяет набор данных таким образом, что для последующей операции будет недостаточно строк. Предположим, что вы используете выражение в модуле **секционирования и выборки** для разделения набора данных по значениям. Если для выражения не найдено совпадений, то один из наборов данных, полученных из секции, будет пустым.

Способы устранения: 

 Если столбец метки включен при выборе столбца, но он не распознан, используйте модуль [изменения метаданных](edit-metadata.md), чтобы пометить его как столбец меток.

  <!--Use the [Summarize Data](summarize-data.md) module to generate a report that shows how many values are missing in each column. -->
  Затем можно использовать модуль [удаления отсутствующих данных](clean-missing-data.md), чтобы удалить строки с отсутствующими значениями в столбце меток. 

 Проверьте входные наборы данных, чтобы убедиться, что они содержат допустимые данные и достаточно строк для удовлетворения требований операции. Многие алгоритмы выводят сообщение об ошибке, если им требуется некоторое минимальное количество строк данных, но данные содержат только несколько строк или только заголовок.

|Сообщения об исключении|
|------------------------|
|Exception occurs when label column is missing or has insufficient number of labeled rows (Исключение возникает, если столбец метки отсутствует или имеет недостаточное количество помеченных строк).|
|Exception occurs when label column is missing or has less than {required_rows_count} labeled rows (Исключение возникает, если столбец меток отсутствует или содержит меньше {required_rows_count} помеченных строк).|
|Exception occurs when label column in dataset {dataset_name} is missing or has less than {required_rows_count} labeled rows (Исключение возникает, когда столбец меток в наборе данных {dataset_name} отсутствует или содержит меньше {required_rows_count} помеченных).|


## <a name="error-0138"></a>Ошибка 0138  
 Превышение доступного объема памяти, не удалось завершить выполнение модуля. Уменьшение выборки набора данных может помочь решить проблему.  

 Эта ошибка возникает, когда выполняемому модулю требуется больше памяти, чем доступно в контейнере Azure. Это может произойти, если вы работаете с большим набором данных, а текущая операция не помещается в память.  

**Решение.** Если вы пытаетесь считать большой набор данных и операция не может быть завершена, может помочь уменьшение выборки набора данных.  

  <!--If you use the visualizations on datasets to check the cardinality of columns, only some rows are sampled. To get a full report, use [Summarize Data](summarize-data.md). You can also use the [Apply SQL Transformation](apply-sql-transformation.md) to check for the number of unique values in each column.  

 Sometimes transient loads can lead to this error. Machine support also changes over time. 

 Try using [Principal Component Analysis](principal-component-analysis.md) or one of the provided feature selection methods to reduce your dataset to a smaller set of more feature-rich columns: [Feature Selection](feature-selection-modules.md)  -->

|Сообщения об исключении|
|------------------------|
|Memory has been exhausted, unable to complete running of module (Превышение доступного объема памяти, не удалось завершить выполнение модуля).|
|Memory has been exhausted, unable to complete running of module (Превышение доступного объема памяти, не удалось завершить выполнение модуля). Сведения: {details}|


## <a name="error-0141"></a>Ошибка 0141  
 Исключение возникает, если число выбранных числовых столбцов и уникальных значений в столбцах категорий и строк слишком мало.  

 Эта ошибка в Машинном обучении Azure возникает, если в выбранном столбце недостаточно уникальных значений для выполнения операции.  

**Решение.** Некоторые операции выполняют статистические процедуры со столбцами признаков и категориальными столбцами, и если их недостаточно, то операция может завершиться неудачно или вернуть неверный результат. Проверьте набор данных, чтобы узнать, сколько значений есть в столбцах признаков и меток, и определите, является ли операция, которую вы пытаетесь выполнить, статистически допустимой.  

 Если исходный набор данных является допустимым, можно также проверить, изменились ли данные или удалились ли некоторые значения из-за обработки вышестоящих данных и операций с данными.  

 Если в вышестоящей операции выполняется разделение, выборка или повторная выборка, убедитесь, что выходные данные содержат ожидаемое число строк и значений.  

|Сообщения об исключении|
|------------------------|
|The number of the selected numerical columns and unique values in the categorical and string columns is too small (Число выбранных числовых столбцов и уникальных значений в столбцах категорий и строк слишком мало).|
|The total number of the selected numerical columns and unique values in the categorical and string columns (currently {actual_num}) should be at least {lower_boundary} (Общее число выбранных числовых столбцов и уникальных значений в столбцах категорий и строк (в настоящее время {actual_num}) должно быть не менее {lower_boundary}).|


## <a name="error-0154"></a>Ошибка 0154  
 Исключение возникает, когда пользователь пытается присоединить данные к ключевым столбцам с несовместимым типом столбца.

|Сообщения об исключении|
|------------------------|
|Key column element types are not compatible (Типы элементов ключевого столбца несовместимы).|
|Key column element types are not compatible.(left: {keys_left}; right: {keys_right}) (Типы элементов ключевого столбца несовместимы. (слева {keys_left}, справа {keys_right})).|


## <a name="error-0155"></a>Ошибка 0155  
 Исключение возникает, если имена столбцов набора данных не являются строковыми.

|Сообщения об исключении|
|------------------------|
|The dataframe column name must be string type. Column names are not string (Имя столбца данных кадров должно иметь строковый тип. Имена столбцов не являются строковыми).|
|The dataframe column name must be string type. Column names {column_names} are not string (Имя столбца данных кадров должно иметь строковый тип. Имена столбцов {column_names} не являются строковыми).|


## <a name="error-0156"></a>Ошибка 0156  
 Исключение возникает, если не удалось считать данные из Базы данных SQL Azure.

|Сообщения об исключении|
|------------------------|
|Failed to read data from Azure SQL Database (Не удалось прочитать данные из Базы данных SQL Azure).|
|Failed to read data from Azure SQL Database: {detailed_message} DB: {database_server_name}:{database_name} Query: {sql_statement} (Не удалось прочитать данные из Базы данных SQL Azure: {detailed_message} БД: {database_server_name}: {database_name} запрос: {sql_statement}).|


## <a name="error-0157"></a>Ошибка 0157  
 Хранилище данных не найдено.

|Сообщения об исключении|
|------------------------|
|Datastore information is invalid (Сведения о хранилище данных недопустимы).|
|Datastore information is invalid. Failed to get AzureML datastore '{datastore_name}' in workspace '{workspace_name}' (Сведения о хранилище данных недопустимы. Не удалось получить хранилище данных AzureML {datastore_name} в рабочей области {workspace_name}).|


## <a name="error-0158"></a>Ошибка 0158
 Возникает, если каталог преобразования недопустим.

|Сообщения об исключении|
|------------------------------------------------------------|
|Given TransformationDirectory is invalid (Указан недопустимый параметр TransformationDirectory).|
|TransformationDirectory "{arg_name}" is invalid. Reason: {reason}. Причина: {reason}. Please rerun training experiment which generates the Transform file. If training experiment was deleted, please recreate and save the Transform file. Недопустимый параметр TransformationDirectory {arg_name}. Причина: {reason}. Запустите обучающий эксперимент, который создает файл преобразования. Если обучающий эксперимент был удален, повторно создайте и сохраните файл преобразования.|
|TransformationDirectory "{arg_name}" is invalid. Reason: {reason}. Причина: {reason}. {troubleshoot_hint}|


## <a name="error-0159"></a>Ошибка 0159
 Исключение возникает, если каталог модели модуля недопустим. 

|Сообщения об исключении|
|------------------------------------------------------------|
|Given ModelDirectory is invalid (Указан недопустимый параметр ModelDirectory).|
|ModelDirectory "{arg_name}" is invalid (Недопустимый параметр ModelDirectory {arg_name}).|
|ModelDirectory "{arg_name}" is invalid (Недопустимый параметр ModelDirectory {arg_name}). Reason: {reason}. Причина: {reason}.|
|ModelDirectory "{arg_name}" is invalid (Недопустимый параметр ModelDirectory {arg_name}). Reason: {reason}. Причина: {reason}. {troubleshoot_hint}|


## <a name="error-1000"></a>Ошибка 1000  
Внутреннее исключение библиотеки.  

Эта ошибка выдается для перехвата необработанных ошибок внутреннего модуля. Поэтому причины этой ошибки могут отличаться в зависимости от модуля, создавшего ошибку.  

Чтобы получить дополнительную помощь, рекомендуется опубликовать подробное сообщение, сопровождающее ошибку, на [форуме машинное обучение Azure](/answers/topics/azure-machine-learning.html), а также описание сценария, включая данные, используемые в качестве входных данных. Эти отзывы помогут нам установить приоритеты для ошибок и найти наиболее важные проблемы для дальнейшей работы.  

|Сообщения об исключении|
|------------------------|
|Library exception (Исключение библиотеки).|
|Library exception: {exception} (Исключение библиотеки {exception}).|
|Unknown library exception: {exception} (Неизвестное исключение библиотеки: {exception}). {customer_support_guidance}.|


## <a name="execute-python-script-module"></a>Выполнить модуль скрипта Python

Выполните поиск **в azureml_main** в **70_Driver_logs** **модуля выполнение скрипта Python** , чтобы узнать, какая строка произошла ошибка. Например, "File"/tmp/tmp01_ID/user_script. Корректировка ", строка 17, в azureml_main" указывает, что эта ошибка произошла в 17-х строке скрипта Python.
