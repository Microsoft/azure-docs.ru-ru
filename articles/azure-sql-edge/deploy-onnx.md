---
title: Развертывание и создание прогнозов с помощью ONNX
titleSuffix: SQL machine learning
description: Узнайте, как обучить модель, преобразовать ее в ONNX, развернуть в SQL Azure для пограничных вычислений или Управляемом экземпляре SQL Azure (предварительная версия), а затем выполнить собственную функцию прогноза (PREDICT) с данными с помощью переданной модели ONNX.
keywords: Развертывание SQL для пограничных вычислений
ms.prod: sql
ms.technology: machine-learning
ms.topic: quickstart
author: dphansen
ms.author: davidph
ms.date: 10/13/2020
ms.openlocfilehash: 755111b2fc48ec119c30d09f2e51b9db6c333848
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100653216"
---
# <a name="deploy-and-make-predictions-with-an-onnx-model-and-sql-machine-learning"></a>Развертывание и создание прогнозов с помощью модели ONNX и машинного обучения SQL

Из этого руководства вы узнаете, как обучить модель, преобразовать ее в ONNX, развернуть в [SQL Azure для пограничных вычислений](onnx-overview.md) или [Управляемом экземпляре SQL Azure (предварительная версия)](../azure-sql/managed-instance/machine-learning-services-overview.md), а затем выполнить собственную функцию прогноза (PREDICT) с данными с помощью переданной модели ONNX.

Это краткое руководство основано на библиотеке машинного обучения **scikit-learn** и в нем используется [набор данных о жилье в Бостоне](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_boston.html).

## <a name="before-you-begin"></a>Перед началом

* Если вы используете SQL Azure для пограничных вычислений или еще не развернули модуль SQL Azure для пограничных вычислений, выполните действия по [развертыванию SQL Azure для пограничных вычислений (предварительная версия) с помощью портала Azure](deploy-portal.md).

* Установите [Azure Data Studio](/sql/azure-data-studio/download).

* Установите пакеты Python, необходимые для этого краткого руководства:

  1. Откройте [новую записную книжку](/sql/azure-data-studio/sql-notebooks), подключенную к ядру Python 3. 
  1. Щелкните **Управление пакетами**.
  1. На вкладке **Установлено** найдите следующие пакеты Python в списке установленных пакетов. Если какой-либо из этих пакетов не установлен, выберите вкладку **Добавить новую**, найдите пакет и нажмите кнопку **Установить**.
     - **scikit-learn**;
     - **numpy**
     - **onnxmltools**
     - **onnxruntime**
     - **pyodbc**
     - **setuptools**
     - **skl2onnx**
     - **sqlalchemy**

* Каждый фрагмент приведенного ниже скрипта введите в ячейку в записной книжке Azure Data Studio и выполните код в ячейке.

## <a name="train-a-pipeline"></a>Обучение конвейера

Разделите набор данных, чтобы спрогнозировать медиану дома на основе признаков.

```python
import numpy as np
import onnxmltools
import onnxruntime as rt
import pandas as pd
import skl2onnx
import sklearn
import sklearn.datasets

from sklearn.datasets import load_boston
boston = load_boston()
boston

df = pd.DataFrame(data=np.c_[boston['data'], boston['target']], columns=boston['feature_names'].tolist() + ['MEDV'])
 
target_column = 'MEDV'
 
# Split the data frame into features and target
x_train = pd.DataFrame(df.drop([target_column], axis = 1))
y_train = pd.DataFrame(df.iloc[:,df.columns.tolist().index(target_column)])

print("\n*** Training dataset x\n")
print(x_train.head())

print("\n*** Training dataset y\n")
print(y_train.head())
```

**Выходные данные**:

```text
*** Training dataset x

        CRIM    ZN  INDUS  CHAS    NOX     RM   AGE     DIS  RAD    TAX  \
0  0.00632  18.0   2.31   0.0  0.538  6.575  65.2  4.0900  1.0  296.0
1  0.02731   0.0   7.07   0.0  0.469  6.421  78.9  4.9671  2.0  242.0
2  0.02729   0.0   7.07   0.0  0.469  7.185  61.1  4.9671  2.0  242.0
3  0.03237   0.0   2.18   0.0  0.458  6.998  45.8  6.0622  3.0  222.0
4  0.06905   0.0   2.18   0.0  0.458  7.147  54.2  6.0622  3.0  222.0

    PTRATIO       B  LSTAT  
0     15.3  396.90   4.98  
1     17.8  396.90   9.14  
2     17.8  392.83   4.03  
3     18.7  394.63   2.94  
4     18.7  396.90   5.33  

*** Training dataset y

0    24.0
1    21.6
2    34.7
3    33.4
4    36.2
Name: MEDV, dtype: float64
```

Создайте конвейер для обучения модели LinearRegression. Можно также использовать другие модели регрессии.

```python
from sklearn.compose import ColumnTransformer
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import RobustScaler

continuous_transformer = Pipeline(steps=[('scaler', RobustScaler())])

# All columns are numeric - normalize them
preprocessor = ColumnTransformer(
    transformers=[
        ('continuous', continuous_transformer, [i for i in range(len(x_train.columns))])])

model = Pipeline(
    steps=[
        ('preprocessor', preprocessor),
        ('regressor', LinearRegression())])

# Train the model
model.fit(x_train, y_train)
```

Проверьте точность модели, а затем вычислите коэффициент детерминации и среднеквадратическую погрешность.

```python
# Score the model
from sklearn.metrics import r2_score, mean_squared_error
y_pred = model.predict(x_train)
sklearn_r2_score = r2_score(y_train, y_pred)
sklearn_mse = mean_squared_error(y_train, y_pred)
print('*** Scikit-learn r2 score: {}'.format(sklearn_r2_score))
print('*** Scikit-learn MSE: {}'.format(sklearn_mse))
```

**Выходные данные**:

```text
*** Scikit-learn r2 score: 0.7406426641094094
*** Scikit-learn MSE: 21.894831181729206
```

## <a name="convert-the-model-to-onnx"></a>Преобразование модели в ONNX

Преобразуйте типы данных в поддерживаемые типы данных SQL. Это преобразование также потребуется для других кадров данных.

```python
from skl2onnx.common.data_types import FloatTensorType, Int64TensorType, DoubleTensorType

def convert_dataframe_schema(df, drop=None, batch_axis=False):
    inputs = []
    nrows = None if batch_axis else 1
    for k, v in zip(df.columns, df.dtypes):
        if drop is not None and k in drop:
            continue
        if v == 'int64':
            t = Int64TensorType([nrows, 1])
        elif v == 'float32':
            t = FloatTensorType([nrows, 1])
        elif v == 'float64':
            t = DoubleTensorType([nrows, 1])
        else:
            raise Exception("Bad type")
        inputs.append((k, t))
    return inputs
```

С помощью `skl2onnx` преобразуйте модель LinearRegression в формат ONNX и сохраните ее локально.

```python
# Convert the scikit model to onnx format
onnx_model = skl2onnx.convert_sklearn(model, 'Boston Data', convert_dataframe_schema(x_train), final_types=[('variable1',FloatTensorType([1,1]))])
# Save the onnx model locally
onnx_model_path = 'boston1.model.onnx'
onnxmltools.utils.save_model(onnx_model, onnx_model_path)
```

## <a name="test-the-onnx-model"></a>Тестирование модели ONNX

После преобразования модели в формат ONNX оцените модель и убедитесь, что она снижает производительность незначительно или не оказывает на нее никакого влияния.

> [!NOTE]
> Среда выполнения ONNX использует тип данных float вместо double, поэтому возможны небольшие несоответствия.

```python
import onnxruntime as rt
sess = rt.InferenceSession(onnx_model_path)

y_pred = np.full(shape=(len(x_train)), fill_value=np.nan)

for i in range(len(x_train)):
    inputs = {}
    for j in range(len(x_train.columns)):
        inputs[x_train.columns[j]] = np.full(shape=(1,1), fill_value=x_train.iloc[i,j])

    sess_pred = sess.run(None, inputs)
    y_pred[i] = sess_pred[0][0][0]

onnx_r2_score = r2_score(y_train, y_pred)
onnx_mse = mean_squared_error(y_train, y_pred)

print()
print('*** Onnx r2 score: {}'.format(onnx_r2_score))
print('*** Onnx MSE: {}\n'.format(onnx_mse))
print('R2 Scores are equal' if sklearn_r2_score == onnx_r2_score else 'Difference in R2 scores: {}'.format(abs(sklearn_r2_score - onnx_r2_score)))
print('MSE are equal' if sklearn_mse == onnx_mse else 'Difference in MSE scores: {}'.format(abs(sklearn_mse - onnx_mse)))
print()
```

**Выходные данные**:

```text
*** Onnx r2 score: 0.7406426691136831
*** Onnx MSE: 21.894830759270633

R2 Scores are equal
MSE are equal
```

## <a name="insert-the-onnx-model"></a>Вставка модели ONNX

Сохраните модель в SQL Azure для пограничных вычислений или в Управляемом экземпляре SQL Azure в таблице `models` базы данных `onnx`. В строке подключения укажите **адрес сервера**, **имя пользователя** и **пароль**.

```python
import pyodbc

server = '' # SQL Server IP address
username = '' # SQL Server username
password = '' # SQL Server password

# Connect to the master DB to create the new onnx database
connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=master;UID=" + username + ";PWD=" + password + ";"

conn = pyodbc.connect(connection_string, autocommit=True)
cursor = conn.cursor()

database = 'onnx'
query = 'DROP DATABASE IF EXISTS ' + database
cursor.execute(query)
conn.commit()

# Create onnx database
query = 'CREATE DATABASE ' + database
cursor.execute(query)
conn.commit()

# Connect to onnx database
db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"

conn = pyodbc.connect(db_connection_string, autocommit=True)
cursor = conn.cursor()

table_name = 'models'

# Drop the table if it exists
query = f'drop table if exists {table_name}'
cursor.execute(query)
conn.commit()

# Create the model table
query = f'create table {table_name} ( ' \
    f'[id] [int] IDENTITY(1,1) NOT NULL, ' \
    f'[data] [varbinary](max) NULL, ' \
    f'[description] varchar(1000))'
cursor.execute(query)
conn.commit()

# Insert the ONNX model into the models table
query = f"insert into {table_name} ([description], [data]) values ('Onnx Model',?)"

model_bits = onnx_model.SerializeToString()

insert_params  = (pyodbc.Binary(model_bits))
cursor.execute(query, insert_params)
conn.commit()
```

## <a name="load-the-data"></a>Загрузка данных

Загрузите данные в SQL.

Сначала создайте две таблицы, **features** (для признаков) и **target** (целевое значение), чтобы сохранить в них подмножества набора данных о жилье в Бостоне.

* Таблица **features** содержит все данные, используемые для прогнозирования целевого значения (медианы). 
* Таблица **target** содержит медианы для каждой записи в наборе данных. 

```python
import sqlalchemy
from sqlalchemy import create_engine
import urllib

db_connection_string = "Driver={ODBC Driver 17 for SQL Server};Server=" + server + ";Database=" + database + ";UID=" + username + ";PWD=" + password + ";"

conn = pyodbc.connect(db_connection_string)
cursor = conn.cursor()

features_table_name = 'features'

# Drop the table if it exists
query = f'drop table if exists {features_table_name}'
cursor.execute(query)
conn.commit()

# Create the features table
query = \
    f'create table {features_table_name} ( ' \
    f'    [CRIM] float, ' \
    f'    [ZN] float, ' \
    f'    [INDUS] float, ' \
    f'    [CHAS] float, ' \
    f'    [NOX] float, ' \
    f'    [RM] float, ' \
    f'    [AGE] float, ' \
    f'    [DIS] float, ' \
    f'    [RAD] float, ' \
    f'    [TAX] float, ' \
    f'    [PTRATIO] float, ' \
    f'    [B] float, ' \
    f'    [LSTAT] float, ' \
    f'    [id] int)'

cursor.execute(query)
conn.commit()

target_table_name = 'target'

# Create the target table
query = \
    f'create table {target_table_name} ( ' \
    f'    [MEDV] float, ' \
    f'    [id] int)'

x_train['id'] = range(1, len(x_train)+1)
y_train['id'] = range(1, len(y_train)+1)

print(x_train.head())
print(y_train.head())
```

Наконец, используйте `sqlalchemy`, чтобы вставить кадры данных Pandas `x_train` и `y_train` в таблицы `features` и `target`соответственно. 

```python
db_connection_string = 'mssql+pyodbc://' + username + ':' + password + '@' + server + '/' + database + '?driver=ODBC+Driver+17+for+SQL+Server'
sql_engine = sqlalchemy.create_engine(db_connection_string)
x_train.to_sql(features_table_name, sql_engine, if_exists='append', index=False)
y_train.to_sql(target_table_name, sql_engine, if_exists='append', index=False)
```

Теперь можно просмотреть данные в базе данных.

## <a name="run-predict-using-the-onnx-model"></a>Выполнение функции PREDICT с помощью модели ONNX

Когда модель будет подготовлена в SQL, выполните собственную функцию PREDICT с данными, используя переданную модель ONNX.

> [!NOTE]
> Измените ядро записной книжки на SQL, чтобы выполнить оставшуюся ячейку.

```sql
USE onnx

DECLARE @model VARBINARY(max) = (
        SELECT DATA
        FROM dbo.models
        WHERE id = 1
        );

WITH predict_input
AS (
    SELECT TOP (1000) [id]
        , CRIM
        , ZN
        , INDUS
        , CHAS
        , NOX
        , RM
        , AGE
        , DIS
        , RAD
        , TAX
        , PTRATIO
        , B
        , LSTAT
    FROM [dbo].[features]
    )
SELECT predict_input.id
    , p.variable1 AS MEDV
FROM PREDICT(MODEL = @model, DATA = predict_input, RUNTIME=ONNX) WITH (variable1 FLOAT) AS p;
```

## <a name="next-steps"></a>Next Steps

* [Машинное обучение и ИИ с применением ONNX в SQL для пограничных вычислений](onnx-overview.md)
* [Службы машинного обучения в Управляемом экземпляре SQL Azure (предварительная версия)](../azure-sql/managed-instance/machine-learning-services-overview.md)
