---
author: gvashishtha
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: include
ms.date: 07/31/2020
ms.author: gopalv
ms.openlocfilehash: 95acb028867caf6f497dd99ad3082efcaab09c7b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93325270"
---
Скрипт входа принимает данные, отправляемые в развернутую веб-службу, и передает их в модель. Затем он принимает ответ, возвращенный моделью, и возвращает его клиенту. *Сценарий зависит от модели*. Он должен понимать данные, которые предположительно и возвращает модель.

В скрипте входа необходимо выполнить два действия:

1. Загрузка модели (с помощью функции с именем `init()` )
1. Выполнение модели на входных данных (с помощью функции с именем `run()` )

Давайте подробно рассмотрим эти шаги.

### <a name="writing-init"></a>Запись init () 

#### <a name="loading-a-registered-model"></a>Загрузка зарегистрированной модели

Зарегистрированные модели хранятся по пути, на который указывает переменная среды с именем `AZUREML_MODEL_DIR` . Дополнительные сведения о точной структуре каталогов см. в разделе [Поиск файлов модели в скрипте записи](../articles/machine-learning/how-to-deploy-advanced-entry-script.md#load-registered-models) .

#### <a name="loading-a-local-model"></a>Загрузка локальной модели

Если вы решили зарегистрировать модель и передали модель в качестве части исходного каталога, вы можете прочитать ее, как и локально, передав путь к модели относительно скрипта оценки. Например, если у вас есть каталог, структурированный как:

```bash

- source_dir
    - score.py
    - models
        - model1.onnx

```

Вы можете загрузить модели с помощью следующего кода Python:

```python
import os

model = open(os.path.join('.', 'models', 'model1.onnx'))
```

### <a name="writing-run"></a>Выполнение записи ()

`run()` выполняется каждый раз, когда модель получает запрос оценки и ждет, что текст запроса будет документом JSON со следующей структурой:

```json
{
    "data": <model-specific-data-structure>
}

```

Входные данные для `run()` — это строка Python, содержащая все, что следует за ключом "Data".

В следующем примере показано, как загрузить зарегистрированную модель scikit-учиться и оценить ее с помощью данных NumPy:

```python
import json
import numpy as np
import os
from sklearn.externals import joblib


def init():
    global model
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

def run(data):
    try:
        data = np.array(json.loads(data))
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

Дополнительные примеры, в том числе автоматическое создание схемы Swagger и двоичные данные (т. е. изображение), см. в [статье о создании расширенных сценариев входа](../articles/machine-learning/how-to-deploy-advanced-entry-script.md) .