---
title: Устранение неполадок в автоматизированных экспериментах ML
titleSuffix: Azure Machine Learning
description: Узнайте, как диагностировать и устранять проблемы в автоматизированных экспериментах машинного обучения.
author: nibaccam
ms.author: nibaccam
ms.reviewer: nibaccam
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 03/08/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, automl, references_regions
ms.openlocfilehash: 28aac830326d60161f54d7ad5fa03326c1d66462
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103563680"
---
# <a name="troubleshoot-automated-ml-experiments-in-python"></a>Устранение неполадок в автоматических экспериментах машинного обучения в Python

В этом руководство вы узнаете, как определить и устранить известные проблемы в автоматизированных экспериментах машинного обучения с помощью [пакета SDK для машинное обучение Azure](/python/api/overview/azure/ml/intro).

## <a name="version-dependencies"></a>Зависимости версий

**`AutoML` зависимости к более новым версиям пакета нарушают совместимость**. После пакета SDK версии 1.13.0 модели не загружаются в старые пакеты SDK из-за несовместимости старых версий, закрепленных в предыдущих `AutoML` пакетах, и более поздних версий, закрепленных сегодня.

При этом должны возникать такие ошибки, как:

* Модуль не нашел такие ошибки, как,

  `No module named 'sklearn.decomposition._truncated_svd`

* Импортировать ошибки, такие как,

  `ImportError: cannot import name 'RollingOriginValidator'`,
* Ошибки атрибутов, такие как,

  `AttributeError: 'SimpleImputer' object has no attribute 'add_indicator`

Разрешения зависят от `AutoML` обучающей версии пакета SDK:

* Если ваша `AutoML` учебная версия пакета SDK больше 1.13.0, вам потребуется `pandas == 0.25.1` и `scikit-learn==0.22.1` .

    * При обнаружении несоответствия версий обновите scikit-учиться и/или Pandas, чтобы исправить версию следующим,

      ```bash
          pip install --upgrade pandas==0.25.1
          pip install --upgrade scikit-learn==0.22.1
      ```

* Если ваша `AutoML` учебная версия пакета SDK меньше или равна 1.12.0, вам потребуется `pandas == 0.23.4` и `sckit-learn==0.20.3` .
  * Если обнаружено несовпадение версий, можно перейти на более раннюю версию scikit и/или Pandas для правильной версии, выполнив следующие инструкции.
  
    ```bash
      pip install --upgrade pandas==0.23.4
      pip install --upgrade scikit-learn==0.20.3
    ```

## <a name="setup"></a>Настройка

`AutoML` изменения пакета, начиная с версии 1.0.76, перед обновлением до новой версии необходимо удалить предыдущую версию.

* **`ImportError: cannot import name AutoMLConfig`**

    Если эта ошибка возникает после обновления с версии пакета SDK до 1.0.76 v 1.0.76 или более поздней, устраните ошибку, выполнив следующую `pip uninstall azureml-train automl` команду: и `pip install azureml-train-automl` . Сценарий automl_setup. cmd делает это автоматически.

* **Сбой automl_setup**

  * В Windows запустите automl_setup из командной строки Anaconda. [Установите Miniconda](https://docs.conda.io/en/latest/miniconda.html).

  * Убедитесь, что установлен conda 64-разрядная версия 4.4.10 или более поздняя. Бит можно проверить с помощью `conda info` команды. `platform` `win-64` Для Windows или `osx-64` для Mac. Чтобы проверить версию, используйте команду `conda -V` . Если установлена предыдущая версия, ее можно обновить с помощью команды: `conda update conda` . Чтобы проверить 32-bit, выполнив 

  * Убедитесь, что conda установлен. 

  * Linux `gcc: error trying to exec 'cc1plus'`

    1. Если возникает `gcc: error trying to exec 'cc1plus': execvp: No such file or directory` ошибка, установите средства сборки GCC для дистрибутива Linux. Например, в Ubuntu используйте команду `sudo apt-get install build-essential` .

    1. Передайте новое имя в качестве первого параметра, чтобы automl_setup для создания новой среды conda. Просмотрите существующие среды conda `conda env list` , используя и удалите их с помощью `conda env remove -n <environmentname>` .

* **automl_setup_linux. sh завершается ошибкой**: Если automl_setup_linus. sh завершается сбоем в Ubuntu Linux с ошибкой: `unable to execute 'gcc': No such file or directory`

  1. Убедитесь, что включены исходящие порты 53 и 80. На виртуальной машине Azure это можно сделать в портал Azure, выбрав виртуальную машину и щелкнув **сеть**.
  1. Выполните команду `sudo apt-get update`.
  1. Выполните команду `sudo apt-get install build-essential --fix-missing`.
  1. `automl_setup_linux.sh`Повторить запуск

* **сбой Configuration. ipynb**:

  * Для локальных conda сначала убедитесь, что `automl_setup` успешно выполнен.
  * Убедитесь, что subscription_id правильно. Найдите subscription_id в портал Azure, выбрав все службы, а затем — подписки. Символы "<" и ">" не должны включаться в значение subscription_id. Например, `subscription_id = "12345678-90ab-1234-5678-1234567890abcd"` имеет допустимый формат.
  * Убедитесь, что участник или владелец имеют доступ к подписке.
  * Убедитесь, что регион является одним из поддерживаемых регионов: `eastus2` , `eastus` ,,,, `westcentralus` `southeastasia` `westeurope` `australiaeast` , `westus2` , `southcentralus` .
  * Убедитесь, что доступ к региону осуществляется с помощью портал Azure.
  
* **сбой Workspace.from_config**:

  Если вызов `ws = Workspace.from_config()` завершается с ошибкой:

  1. Убедитесь, что Записная книжка Configuration. ipynb успешно запущена.
  1. Если Записная книжка запускается из папки, которая не находится в папке, в которой выполнялась эта папка `configuration.ipynb` , скопируйте папку aml_config и файл config.jsна том, который она содержит, в новую папку. Workspace.from_config считывает config.jsдля папки записной книжки или ее родительской папки.
  1. Если используется новая подписка, Группа ресурсов, Рабочая область или регион, убедитесь, что вы повторно запускаете `configuration.ipynb` записную книжку. Изменение config.jsнапрямую будет работать только в том случае, если Рабочая область уже существует в указанной группе ресурсов в указанной подписке.
  1. Если вы хотите изменить регион, измените рабочую область, группу ресурсов или подписку. `Workspace.create` не будет создавать или обновлять рабочую область, если она уже существует, даже если указана другая область.

## <a name="tensorflow"></a>TensorFlow

Начиная с версии 1.5.0 пакета SDK, автоматизированное машинное обучение по умолчанию не устанавливает модели TensorFlow. Чтобы установить TensorFlow и использовать его вместе с автоматизированными экспериментами ML, установите с `tensorflow==1.12.0` помощью `CondaDependencies` .

```python
  from azureml.core.runconfig import RunConfiguration
  from azureml.core.conda_dependencies import CondaDependencies
  run_config = RunConfiguration()
  run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['tensorflow==1.12.0'])
```

## <a name="numpy-failures"></a>Сбои NumPy

* **`import numpy` завершается ошибкой в Windows**: в некоторых средах Windows отображается ошибка при загрузке NumPy с последней версией Python 3.6.8. Если вы видите эту ошибку, попробуйте использовать Python версии 3.6.7.

* **`import numpy` сбой**: Проверьте версию TensorFlow в среде автоматизированного conda ml. Поддерживаемые версии: < 1,13. Удалите TensorFlow из среды, если версия >= 1,13.

Проверить версию TensorFlow и удалить можно следующим образом:

  1. Запустите командную оболочку и активируйте среду conda, в которой установлены автоматические пакеты ml.
  1. Введите `pip freeze` и найдите `tensorflow` , если он найден, указанная версия должна быть < 1,13
  1. Если указанная версия не является поддерживаемой, `pip uninstall tensorflow` в командной оболочке введите y в поле Подтверждение.

## `jwt.exceptions.DecodeError`

Точное сообщение об ошибке: `jwt.exceptions.DecodeError: It is required that you pass in a value for the "algorithms" argument when calling decode()` .

Для версий пакета SDK <= 1.17.0, установка может привести к неподдерживаемой версии Пижвт. Убедитесь, что версия Пижвт в среде автоматического conda ML является поддерживаемой версией. Это Пижвт версия < 2.0.0.

Вы можете проверить версию Пижвт следующим образом:

1. Запустите командную оболочку и активируйте среду conda, в которой установлены автоматические пакеты ML.

1. Введите `pip freeze` и найдите `PyJWT` , если он найден, указанная версия должна быть < 2.0.0

Если указанная версия не является поддерживаемой версией, выполните следующие действия.

1. Рассмотрите возможность обновления до последней версии пакета SDK для Аутомл: `pip install -U azureml-sdk[automl]`

1. Если это неприемлемо, удалите Пижвт из среды и установите правильную версию следующим образом:

    1. `pip uninstall PyJWT` в командной оболочке и введите `y` для подтверждения.
    1. Установите с помощью `pip install 'PyJWT<2.0.0'` .
  
## <a name="databricks"></a>Databricks
См. статью [Настройка автоматического эксперимента ml с модулями](how-to-configure-databricks-automl-environment.md#troubleshooting).

## <a name="forecasting-r2-score-is-always-zero"></a>Оценка прогнозирования R2 всегда равна нулю

 Эта проблема возникает, если в предоставленных данных для обучения есть временные ряды, которые содержат одинаковые значения для последних `n_cv_splits`  +  `forecasting_horizon` точек данных.

Если этот шаблон ожидается в ряде временных рядов, можно переключить основную метрику на **нормализованное среднее значение ошибки в корне**.

## <a name="failed-deployment"></a>Сбой развертывания

 Для версий <= 1.18.0 пакета SDK, базовый образ, созданный для развертывания, может завершиться со следующей ошибкой: `ImportError: cannot import name cached_property from werkzeug` .

  Для устранения этой проблемы можно выполнить следующие действия.

  1. Скачивание пакета модели
  1. Распакуйте пакет
  1. Развертывание с помощью распакованных ресурсов

## <a name="azure-functions-application"></a>Приложение функций Azure
  
  В настоящее время автоматический ML не поддерживает приложения функций Azure. 

## <a name="sample-notebook-failures"></a>Примеры сбоев записной книжки

 Если пример записной книжки завершается ошибкой, что свойство, метод или библиотека не существуют:

* Убедитесь, что в Jupyter Notebook выбран правильный kernel. Ядро отображается в правом верхнем углу страницы записной книжки. Значение по умолчанию — *azure_automl*. Ядро сохраняется как часть записной книжки. При переключении в новую среду conda необходимо выбрать новый ядро в записной книжке.

  * Для записных книжек Azure это должен быть Python 3,6.
  * Для локальных сред conda это должно быть имя среды conda, указанное в automl_setup.

* Чтобы убедиться, что Записная книжка предназначена для используемой версии пакета SDK,
  * Проверьте версию пакета SDK, выполнив `azureml.core.VERSION` в Jupyter Notebook ячейку.
  * Вы можете скачать предыдущую версию образцов записных книжек из GitHub, выполнив следующие действия:
    1. Нажмите `Branch` кнопку
    1. Перейдите на `Tags` вкладку
    1. Выберите версию

## <a name="next-steps"></a>Дальнейшие действия

+ Узнайте больше о том, [как обучить модель регрессии с помощью автоматизированного машинного обучения](tutorial-auto-train-models.md) или [как обучить ее с использованием автоматизированного машинного обучения на удаленном ресурсе](how-to-auto-train-remote.md).

+ Узнайте больше о том, [как и где можно развернуть модель](how-to-deploy-and-where.md).
