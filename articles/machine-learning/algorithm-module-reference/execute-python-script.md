---
title: 'Выполнение скрипта Python: Справочник по модулям'
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать модуль выполнение скрипта Python в Машинное обучение Azure Designer для выполнения кода Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.custom: devx-track-python
author: likebupt
ms.author: keli19
ms.date: 01/02/2021
ms.openlocfilehash: 6003ca9156d8553604d7ebbf94c5c3373d077f0f
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102500705"
---
# <a name="execute-python-script-module"></a>Выполнить модуль скрипта Python

В этой статье описывается модуль выполнение скрипта Python в конструкторе Машинное обучение Azure.

Используйте этот модуль для выполнения кода Python. Дополнительные сведения об архитектуре и принципах проектирования Python см. [в статье выполнение кода Python в конструкторе машинное обучение Azure](../how-to-designer-python.md).

С помощью Python можно выполнять задачи, которые не поддерживаются существующими модулями, например:

+ Визуализация данных с помощью `matplotlib` .
+ Использование библиотек Python для перечисления наборов данных и моделей в рабочей области.
+ Чтение, Загрузка и обработка данных из источников, которые не поддерживаются модулем [импорта данных](./import-data.md) .
+ Запустите собственный код глубокого обучения. 

## <a name="supported-python-packages"></a>Поддерживаемые пакеты Python

Машинное обучение Azure использует дистрибутив Python Anaconda, который включает множество стандартных служебных программ для обработки данных. Мы будем обновлять версию Anaconda автоматически. Текущая версия:
 -  Anaconda 4.5 + дистрибутив для Python 3,6 

Полный список см. в разделе с [предварительно установленными пакетами Python](#preinstalled-python-packages).

Чтобы установить пакеты, отсутствующие в предварительно установленном списке (например, *scikit-Разное*), добавьте в сценарий следующий код: 

```python
import os
os.system(f"pip install scikit-misc")
```

Используйте следующий код, чтобы установить пакеты для повышения производительности, особенно для вывода:
```python
import importlib.util
package_name = 'scikit-misc'
spec = importlib.util.find_spec(package_name)
if spec is None:
    import os
    os.system(f"pip install scikit-misc")
```

> [!NOTE]
> Если конвейер содержит несколько модулей выполнения скриптов Python, для которых требуются пакеты, отсутствующие в предварительно установленном списке, установите пакеты в каждом модуле.

> [!WARNING]
> Модуль скрипта Python екскуте не поддерживает установку пакетов, зависящих от дополнительных собственных библиотек, с помощью команды вроде "apt-get", например Java, PyODBC и т. д. Это происходит потому, что этот модуль выполняется в простой среде с предварительно установленным Python и с разрешениями, не являющимися администраторами.  

## <a name="access-to-current-workspace-and-registered-datasets"></a>Доступ к текущей рабочей области и зарегистрированным наборам данных

Чтобы получить доступ к [зарегистрированным наборам данных](../how-to-create-register-datasets.md) в рабочей области, можно обратиться к следующему примеру кода:

```Python
def azureml_main(dataframe1 = None, dataframe2 = None):

    # Execution logic goes here
    print(f'Input pandas.DataFrame #1: {dataframe1}')
    from azureml.core import Run
    run = Run.get_context(allow_offline=True)
    #access to current workspace
    ws = run.experiment.workspace

    #access to registered dataset of current workspace
    from azureml.core import Dataset
    dataset = Dataset.get_by_name(ws, name='test-register-tabular-in-designer')
    dataframe1 = dataset.to_pandas_dataframe()
     
    # If a zip file is connected to the third input port,
    # it is unzipped under "./Script Bundle". This directory is added
    # to sys.path. Therefore, if your zip file contains a Python file
    # mymodule.py you can import it using:
    # import mymodule

    # Return value must be of a sequence of pandas.DataFrame
    # E.g.
    #   -  Single return value: return dataframe1,
    #   -  Two return values: return dataframe1, dataframe2
    return dataframe1,
```

## <a name="upload-files"></a>Отправка файлов
Модуль выполнение скрипта Python поддерживает отправку файлов с помощью [пакета SDK для машинное обучение Azure Python](/python/api/azureml-core/azureml.core.run%28class%29#upload-file-name--path-or-stream-).

В следующем примере показано, как передать файл изображения в модуль выполнение скрипта Python:

```Python

# The script MUST contain a function named azureml_main,
# which is the entry point for this module.

# Imports up here can be used to
import pandas as pd

# The entry point function must have two input arguments:
#   Param<dataframe1>: a pandas.DataFrame
#   Param<dataframe2>: a pandas.DataFrame
def azureml_main(dataframe1 = None, dataframe2 = None):

    # Execution logic goes here
    print(f'Input pandas.DataFrame #1: {dataframe1}')

    from matplotlib import pyplot as plt
    plt.plot([1, 2, 3, 4])
    plt.ylabel('some numbers')
    img_file = "line.png"
    plt.savefig(img_file)

    from azureml.core import Run
    run = Run.get_context(allow_offline=True)
    run.upload_file(f"graphics/{img_file}", img_file)

    # Return value must be of a sequence of pandas.DataFrame
    # For example:
    #   -  Single return value: return dataframe1,
    #   -  Two return values: return dataframe1, dataframe2
    return dataframe1,
}
```

После завершения выполнения конвейера можно просмотреть изображение на правой панели модуля.

> [!div class="mx-imgBorder"]
> ![Предварительный просмотр отправленного изображения](media/module/upload-image-in-python-script.png)

## <a name="how-to-configure-execute-python-script"></a>Настройка выполнения скрипта Python

Модуль выполнение скрипта Python содержит пример кода Python, который можно использовать в качестве отправной точки. Чтобы настроить модуль выполнения скрипта Python, укажите набор входных данных и код Python для выполнения в текстовом поле **скрипт Python** .

1. Добавьте модуль **выполнение скрипта Python** в конвейер.

2. Добавьте и подключитесь к наборам данных **dataSet1** из конструктора, которые вы хотите использовать для ввода. Сослаться на этот набор данных в скрипте Python как **DataFrame1**.

    Использовать набор данных необязательно. Используйте его, если хотите создать данные с помощью Python, или используйте код Python для импорта данных непосредственно в модуль.

    Этот модуль поддерживает добавление второго набора данных в **DataSet2**. Сослаться на второй набор данных в скрипте Python как **DataFrame2**.

    Наборы данных, хранящиеся в Машинное обучение Azure, автоматически преобразуются в кадры с данными Pandas при загрузке с этим модулем.

    ![Схема ввода для выполнения сценария Python](media/module/python-module.png)

4. Чтобы включить новые пакеты или код Python, подключите сжатый ZIP-файл, содержащий эти настраиваемые ресурсы, в порт **пакета сценариев** . Если размер скрипта превышает 16 КБ, используйте порт **пакета сценариев** , чтобы избежать ошибок, таких как *Командная строка, превышает ограничение в 16597 символов*. 

    
    1. Упакуйте скрипт и другие настраиваемые ресурсы в ZIP-файл.
    1. Отправьте ZIP-файл в качестве **файлового набора данных** в студию. 
    1. Перетащите модуль DataSet из списка *наборы* данных на панели слева модуль на странице Создание конструктора. 
    1. Подключите модуль набора данных к порту **пакета скрипта** для **выполнения модуля скрипта Python** .
    
    Любой файл, содержащийся в загруженном ZIP-архиве, можно использовать во время выполнения конвейера. Если архив содержит структуру каталогов, структура сохраняется.
 
    > [!WARNING]
    > **Не** используйте **приложение** в качестве имени папки или сценария, так как **приложение** является зарезервированным словом для встроенных служб. Но можно использовать и другие пространства имен `app123` , например.
   
    Ниже приведен пример пакета сценариев, который содержит файл скрипта Python и txt-файл:
      
    > [!div class="mx-imgBorder"]
    > ![Пример пакета сценариев](media/module/python-script-bundle.png)  

    Ниже приведено содержимое `my_script.py` :

    ```python
    def my_func(dataframe1):
    return dataframe1
    ```
    Ниже приведен пример кода, демонстрирующий использование файлов в пакете скриптов.    

    ```python
    import pandas as pd
    from my_script import my_func
 
    def azureml_main(dataframe1 = None, dataframe2 = None):
 
        # Execution logic goes here
        print(f'Input pandas.DataFrame #1: {dataframe1}')
 
        # Test the custom defined python function
        dataframe1 = my_func(dataframe1)
 
        # Test to read custom uploaded files by relative path
        with open('./Script Bundle/my_sample.txt', 'r') as text_file:
            sample = text_file.read()
    
        return dataframe1, pd.DataFrame(columns=["Sample"], data=[[sample]])
    ```

5. В текстовом поле **скрипт Python** введите или вставьте допустимый скрипт Python.

    > [!NOTE]
    >  Будьте внимательны при написании сценария. Убедитесь в отсутствии синтаксических ошибок, таких как использование необъявленных переменных или неимпортированных модулей или функций. Обратите особое внимание на список предварительно установленных модулей. Чтобы импортировать модули, которых нет в списке, установите соответствующие пакеты в скрипте, например:
    >  ``` Python
    > import os
    > os.system(f"pip install scikit-misc")
    > ```
    
    Текстовое поле **скрипта Python** заполняется с помощью некоторых инструкций в комментариях и примера кода для доступа к данным и вывода данных. Этот код необходимо изменить или заменить. Следуйте соглашениям Python для отступов и регистра:

    + Скрипт должен содержать функцию с именем `azureml_main` в качестве точки входа для этого модуля.
    + Функция точки входа должна иметь два входных аргумента, `Param<dataframe1>` и `Param<dataframe2>` , даже если эти аргументы не используются в скрипте.
    + ZIP-файлы, подключенные к третьему порту ввода, являются распакованными и хранятся в каталоге `.\Script Bundle` , который также добавляется в Python `sys.path` . 

    Если ZIP-файл содержит `mymodule.py` , импортируйте его с помощью команды `import mymodule` .

    В конструктор можно вернуть два набора данных, которые должны быть последовательностью типа `pandas.DataFrame` . Вы можете создавать другие выходные данные в коде Python и записывать их непосредственно в службу хранилища Azure.

    > [!WARNING]
    > **Не** рекомендуется подключаться к базе данных или другим внешним хранилищам в **модуле выполнение скрипта Python**. Вы можете использовать модуль [Импорт данных](./import-data.md) и [модуль экспорта данных](./export-data.md)     

6. Отправьте конвейер.

    Если модуль завершен, проверьте выходные данные, если они ожидались.

    Если произошел сбой модуля, необходимо выполнить некоторые действия по устранению неполадок. Выберите модуль и откройте **выходные данные и журналы** на правой панели. Откройте **70_driver_log.txt** и выполните поиск **в azureml_main**, после чего можно найти строку, вызвавшую ошибку. Например, "File"/tmp/tmp01_ID/user_script. Корректировка ", строка 17, в azureml_main" указывает, что эта ошибка произошла в 17-х строке скрипта Python.

## <a name="results"></a>Результаты

Результаты любых вычислений с помощью внедренного кода Python должны быть предоставлены как `pandas.DataFrame` , что автоматически преобразуется в формат набора данных машинное обучение Azure. Затем можно использовать результаты с другими модулями в конвейере.

Модуль возвращает два набора данных:  
  
+ **Набор данных Results 1**, определяемый первым возвращенным кадром данных Pandas в скрипте Python.

+ **Результирующий набор данных 2**, определяемый вторым возвращенным кадром данных Pandas в скрипте Python.

## <a name="preinstalled-python-packages"></a>Предварительно установленные пакеты Python
Предварительно установленные пакеты:
-    adal==1.2.2
-    applicationinsights==0.11.9
-    attr = = 19.3.0
-    Azure-Common = = 1.1.25
-    Azure-Core = = 1.3.0
-    azure-graphrbac==0.61.1
-    Azure-Identity = = 1.3.0
-    azure-mgmt-authorization==0.60.0
-    azure-mgmt-containerregistry==2.8.0
-    Azure-управление-keyvault = = 2.2.0
-    Azure-управление-ресурс = = 8.0.1
-    Azure-управление-хранилище = = 8.0.0
-    Azure-Storage-BLOB = = 1.5.0
-    Azure-Storage — Common = = 1.4.2
-    azureml-Core = = 1.1.5.5
-    azureml-onподготовительная — Native = = 14.1.0
-    azureml-"Подготовка к вы1.3.5У"
-    azureml — значения по умолчанию = = 1.1.5.1
-    azureml-Designer-Classic-modules = = 0.0.118
-    azureml-Designer-Core = = 0.0.31
-    azureml-Designer-Internal = = 0.0.18
-    azureml-Model-Management-SDK = = 1.0.1 B6. post1
-    azureml-конвейер-Core = = 1.1.5
-    azureml-телеметрии = = 1.1.5.3
-    backports.tempfile==1.0
-    backports.weakref==1.0.post1
-    boto3 = = 1.12.29
-    ботокоре = = 1.15.29
-    качетулс = = 4.0.0
-    Сертификация = = 2019.11.28
-    cffi==1.12.3
-    chardet==3.0.4
-    щелчок = = 7.1.1
-    клаудпиккле = = 1.3.0
-    конфигпарсер = = 3.7.4
-    contextlib2 = = 0.6.0. post1
-    Шифрование = = 2.8
-    cycler==0.10.0
-    dill==0.3.1.1
-    distro==1.4.0
-    DOCKER = = 4.2.0
-    docutils==0.15.2
-    dotnetcore2 = = 2.1.13
-    Flask = = 1.0.3
-    fusepy==3.0.1
-    gensim==3.8.1
-    Google-API-Core = = 1.16.0
-    Google-auth = = 1.12.0
-    Google-Cloud-Core = = 1.3.0
-    Google-Cloud-Storage = = 1.26.0
-    Google-reвозобновляет работу — носитель = = 0.5.0
-    гуглеапис-Common-идентификаторы = = 1.51.0
-    гуникорн = = 19.9.0
-    IDNA = = 2.9
-    дисбаланс — сведения = = 0.4.3
-    isodate==0.6.0
-    итсданжераус = = 1.1.0
-    жипнэй = = 0.4.3
-    Jinja2 = = 2.11.1
-    jmespath = = 0.9.5
-    жоблиб = = 0.14.0
-    JSON-Logging-корректировка = = 0,2
-    жсонпиккле = = 1.3
-    жсонсчема = = 3.0.1
-    kiwisolver==1.1.0
-    лиак-arff = = 2.4.0
-    lightgbm==2.2.3
-    маркупсафе = = 1.1.1
-    Matplotlib = = 3.1.3
-    more-итертулс = = 6.0.0
-    msal-Extensions = = 0.1.3
-    msal = = 1.1.0
-    мсрест = = 0.6.11
-    мсрестазуре = = 0.6.3
-    ndg-httpsclient==0.5.1
-    нимбусмл = = 1.6.1
-    NumPy = = 1.18.2
-    oauthlib==3.1.0
-    Pandas = = 0.25.3
-    пасспек = = 0.7.0
-    PIP = = 20.0.2
-    порталоккер = = 1.6.0
-    protobuf = = 3.11.3
-    pyarrow = = 0.16.0
-    pyasn1-modules = = 0.2.8
-    pyasn1 = = 0.4.8
-    пикпарсер = = 2,20
-    пикриптодомекс = = 3.7.3
-    пижвт = = 1.7.1
-    pyopenssl = = 19.1.0
-    пипарсинг = = 2.4.6
-    пирсистент = = 0.16.0
-    Python-датеутил = = 2.8.1
-    pytz==2019.3
-    запросы — оауслиб = = 1.3.0
-    запросы = = 2.23.0
-    RSA = = 4.0
-    ruamel.yaml==0.15.89
-    s3transfer = = 0.3.3
-    scikit — сведения = = 0.22.2
-    SciPy = = 1.4.1
-    секретстораже = = 3.1.2
-    setuptools = = 46.1.1. post20200323
-    шесть = = 1.14.0
-    Smart-Open = = 1.10.0
-    urllib3 = = 1.25.8
-    WebSocket-Client = = 0.57.0
-    веркзеуг = = 0.16.1
-    Wheel = = 0.34.2

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [набором доступных модулей](module-reference.md) в службе Машинного обучения Azure.