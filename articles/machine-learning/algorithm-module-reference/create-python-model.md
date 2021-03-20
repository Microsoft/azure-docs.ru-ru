---
title: 'Создание модели Python: Справочник по модулям'
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать модуль создания модели Python в Машинное обучение Azure для создания пользовательского модуля моделирования или обработки данных.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 06/18/2020
ms.openlocfilehash: 03f80ebeb7b30d8c0fe14060335541d77b7523b0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90898627"
---
# <a name="create-python-model-module"></a>Создать модуль модели Python

В этой статье описывается модуль в конструкторе Машинное обучение Azure.

Узнайте, как использовать модуль создания модели Python для создания обученной модели из скрипта Python. Модель можно создать на основе любого из сведений, включенного в пакет Python в среде Машинное обучение Azure конструктора. 

После создания модели можно использовать [обучение модели](train-model.md) для обучения модели на наборе данных, как и любые другие знания в машинное обучение Azure. Обученную модель можно передать в [модель оценки](score-model.md) для создания прогнозов. Затем можно сохранить обученную модель и опубликовать рабочий процесс оценки как веб-службу.

> [!WARNING]
> Сейчас невозможно подключить этот модуль для **настройки модуля параметров модели** или передать результаты оценки модели Python для [оценки модели](evaluate-model.md). Если необходимо настроить параметры или оценить модель, можно написать пользовательский скрипт Python с помощью модуля [выполнение скрипта Python](execute-python-script.md) .


## <a name="configure-the-module"></a>Настройка модуля

Для использования этого модуля необходимы промежуточные или квалифицированные знания о Python. Модуль поддерживает использование любых сведений, включенных в пакеты Python, уже установленные в Машинное обучение Azure. См. список предварительно установленного пакета Python в разделе [выполнение скрипта Python](execute-python-script.md).

> [!NOTE]
> Будьте внимательны при написании сценария и убедитесь в отсутствии синтаксических ошибок, таких как использование необъявленного объекта или неимпортированного модуля.

> [!NOTE]
> Также обратите особое внимание на список предварительно установленных модулей в [сценарии выполнения Python](execute-python-script.md). Импортировать только предварительно установленные модули. Не устанавливайте в этот сценарий дополнительные пакеты, например "PIP Install xgboost", иначе при чтении моделей в модулях нижнего уровня будут возникать ошибки.
  
В этой статье показано, как использовать **модель создания Python** с простым конвейером. Вот схема конвейера:

![Схема создания модели Python](./media/module/create-python-model.png)

1. Выберите **создать модель Python** и измените скрипт, чтобы реализовать процесс моделирования или управления данными. Модель можно создать на основе любого из сведений, включенного в пакет Python в среде Машинное обучение Azure.

> [!NOTE]
> Обратите особое внимание на комментарии в образце кода сценария и убедитесь, что сценарий строго соответствует требованию, включая имя класса, методы и сигнатуру метода. Нарушение приводит к исключениям. 

   В следующем примере кода классификатора упрощенного алгоритма Байеса с двумя классами используется популярный пакет *sklearn* :

   ```Python

   # The script MUST define a class named AzureMLModel.
   # This class MUST at least define the following three methods:
       # __init__: in which self.model must be assigned,
       # train: which trains self.model, the two input arguments must be pandas DataFrame,
       # predict: which generates prediction result, the input argument and the prediction result MUST be pandas DataFrame.
   # The signatures (method names and argument names) of all these methods MUST be exactly the same as the following example.

   # Please do not install extra packages such as "pip install xgboost" in this script,
   # otherwise errors will be raised when reading models in down-stream modules.
   
   import pandas as pd
   from sklearn.naive_bayes import GaussianNB


   class AzureMLModel:
       def __init__(self):
           self.model = GaussianNB()
           self.feature_column_names = list()

       def train(self, df_train, df_label):
           # self.feature_column_names records the column names used for training.
           # It is recommended to set this attribute before training so that the
           # feature columns used in predict and train methods have the same names.
           self.feature_column_names = df_train.columns.tolist()
           self.model.fit(df_train, df_label)

       def predict(self, df):
           # The feature columns used for prediction MUST have the same names as the ones for training.
           # The name of score column ("Scored Labels" in this case) MUST be different from any other columns in input data.
           return pd.DataFrame(
               {'Scored Labels': self.model.predict(df[self.feature_column_names]), 
                'probabilities': self.model.predict_proba(df[self.feature_column_names])[:, 1]}
           )


   ```

2. Подключите только что созданный модуль **создания модели Python** для обучения модели **модели** и **оценки**.

3. Если необходимо оценить модель, добавьте модуль [выполнение скрипта Python](execute-python-script.md) и измените скрипт Python.

   Следующий скрипт является образцом кода вычисления:

   ```Python


   # The script MUST contain a function named azureml_main
   # which is the entry point for this module.

   # imports up here can be used to 
   import pandas as pd

   # The entry point function MUST have two input arguments:
   #   Param<dataframe1>: a pandas.DataFrame
   #   Param<dataframe2>: a pandas.DataFrame
   def azureml_main(dataframe1 = None, dataframe2 = None):
    
       from sklearn.metrics import accuracy_score, precision_score, recall_score, roc_auc_score, roc_curve
       import pandas as pd
       import numpy as np
    
       scores = dataframe1.ix[:, ("income", "Scored Labels", "probabilities")]
       ytrue = np.array([0 if val == '<=50K' else 1 for val in scores["income"]])
       ypred = np.array([0 if val == '<=50K' else 1 for val in scores["Scored Labels"]])    
       probabilities = scores["probabilities"]
    
       accuracy, precision, recall, auc = \
       accuracy_score(ytrue, ypred),\
       precision_score(ytrue, ypred),\
       recall_score(ytrue, ypred),\
       roc_auc_score(ytrue, probabilities)
    
       metrics = pd.DataFrame();
       metrics["Metric"] = ["Accuracy", "Precision", "Recall", "AUC"];
       metrics["Value"] = [accuracy, precision, recall, auc]
    
       return metrics,

   ```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [набором доступных модулей](module-reference.md) в службе Машинного обучения Azure. 