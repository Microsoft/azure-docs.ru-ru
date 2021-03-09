---
title: Переход с оценщиков на ScriptRunConfig
titleSuffix: Azure Machine Learning
description: Руководством по миграции из оценщиков в Скриптрунконфиг для настройки обучающих заданий.
services: machine-learning
author: mx-iao
ms.author: minxia
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 12/14/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperf-fy21q1
ms.openlocfilehash: ae0623a11b940a4d142f6bfae02d4b20727a6f55
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102518878"
---
# <a name="migrating-from-estimators-to-scriptrunconfig"></a>Миграция с оценщиков на Скриптрунконфиг

До настоящего момента существует несколько методов настройки учебного задания в Машинное обучение Azure через пакет SDK, включая средства оценки, Скриптрунконфиг и RunConfiguration низшего уровня.   Для решения этой неоднозначности и несогласованности мы упростили процесс настройки задания в МАШИНном обучении Azure.  Теперь следует использовать Скриптрунконфиг как рекомендуемый вариант для настройки учебных заданий. 

Средства оценки являются устаревшими в 1.19.0 выпуске пакета SDK для Python. Также следует избегать явного создания объекта RunConfiguration, а вместо этого настроить задание с помощью класса Скриптрунконфиг.

В этой статье рассматриваются распространенные вопросы, касающиеся перехода с оценщиков на Скриптрунконфиг.

> [!IMPORTANT]
> Чтобы выполнить миграцию в Скриптрунконфиг из оценщиков, убедитесь, что вы используете >= 1.15.0 пакета SDK для Python.

## <a name="scriptrunconfig-documentation-and-samples"></a>Документация и примеры Скриптрунконфиг
Машинное обучение Azure документация и примеры были обновлены, чтобы использовать [скриптрунконфиг](/python/api/azureml-core/azureml.core.script_run_config.scriptrunconfig) для настройки и отправки заданий.

Сведения об использовании Скриптрунконфиг см. в следующей документации:
* [Настройка и отправка запуска на выполнение обучения](how-to-set-up-training-targets.md)
* [Настройка запусков обучения PyTorch](how-to-train-pytorch.md)
* [Настройка запусков обучения TensorFlow](how-to-train-tensorflow.md)
* [Настройка scikit — обучение выполнению](how-to-train-scikit-learn.md)

Кроме того, ознакомьтесь со следующими примерами & учебники.
* [Azure или Мачинелеарнингнотебукс](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks)
* [Azure/azureml-examples](https://github.com/Azure/azureml-examples)

## <a name="defining-the-training-environment"></a>Определение среды обучения
В то время как различные средства оценки инфраструктуры имеют предварительно настроенные среды, которые поддерживаются образами DOCKER, файлы dockerfile для этих образов являются частными.  Поэтому у вас нет большого количества прозрачных элементов, которые содержат эти среды. Кроме того, средства оценки принимают настройки, связанные с средой, как отдельные параметры (например `pip_packages` , `custom_docker_image` ) в соответствующих конструкторах.

При использовании Скриптрунконфиг все конфигурации, связанные с окружением, инкапсулируются в `Environment` объект, который передается в `environment` параметр конструктора скриптрунконфиг. Чтобы настроить задание обучения, укажите среду со всеми зависимостями, необходимыми для сценария обучения. Если среда не указана, Azure ML будет использовать один из базовых образов машинного обучения Azure, в частности тот, который определен в `azureml.core.environment.DEFAULT_CPU_IMAGE` качестве среды по умолчанию. Существует несколько способов предоставления среды.

* [Использование проверенных](how-to-use-environments.md#use-a-curated-environment) сред, проверенных средой, по умолчанию — предопределенные среды, доступные в рабочей области. Имеется соответствующая проверенная среда для каждого из предварительно настроенных образов DOCKER платформы или версии, которые зарезервированы каждым из них.
* [Определение собственной настраиваемой среды](how-to-use-environments.md)

Ниже приведен пример использования проверенной среды PyTorch 1,6 для обучения.

```python
from azureml.core import Workspace, ScriptRunConfig, Environment

curated_env_name = 'AzureML-PyTorch-1.6-GPU'
pytorch_env = Environment.get(workspace=ws, name=curated_env_name)

compute_target = ws.compute_targets['my-cluster']
src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      compute_target=compute_target,
                      environment=pytorch_env)
```

Если необходимо указать **переменные среды** , которые будут заданы для процесса, в котором выполняется обучающий сценарий, используйте объект Environment:
```
myenv.environment_variables = {"MESSAGE":"Hello from Azure Machine Learning"}
```

Сведения о настройке сред машинного обучения Azure и управлении ими см. в следующих статьях:
* [Использование сред](how-to-use-environments.md)
* [Курируемые среды](resource-curated-environments.md)
* [Обучение с помощью пользовательского образа DOCKER](how-to-train-with-custom-image.md)

## <a name="using-data-for-training"></a>Использование данных для обучения
### <a name="datasets"></a>Наборы данных
Если для обучения используется набор данных МАШИНного обучения Azure, передайте набор данных в качестве аргумента скрипта с помощью `arguments` параметра. Это позволит получить путь к данным (точка подключения или путь загрузки) в обучающем скрипте через аргументы.

В следующем примере выполняется настройка учебного задания, в котором Филедатасет, `mnist_ds` , будет подключена к удаленному вычислению.
```python
src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      arguments=['--data-folder', mnist_ds.as_mount()], # or mnist_ds.as_download() to download
                      compute_target=compute_target,
                      environment=pytorch_env)
```

### <a name="datareference-old"></a>Ссылка на (Old)
Хотя мы рекомендуем использовать наборы данных машинного обучения Azure по старому принципу работы с ссылками, если по какой-либо причине вы все еще используете ссылки на сведения, необходимо настроить задание следующим образом:
```python
# if you want to pass a DataReference object, such as the below:
datastore = ws.get_default_datastore()
data_ref = datastore.path('./foo').as_mount()

src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      arguments=['--data-folder', str(data_ref)], # cast the DataReference object to str
                      compute_target=compute_target,
                      environment=pytorch_env)
src.run_config.data_references = {data_ref.data_reference_name: data_ref.to_config()} # set a dict of the DataReference(s) you want to the `data_references` attribute of the ScriptRunConfig's underlying RunConfiguration object.
```

Дополнительные сведения об использовании данных для обучения см. в следующих статьях:
* [Обучение с наборами данных в МАШИНном обучении Azure](./how-to-train-with-datasets.md)

## <a name="distributed-training"></a>Распределенное обучение
Если необходимо настроить распределенное задание для обучения, это можно сделать, указав `distributed_job_config` параметр в конструкторе скриптрунконфиг. Передайте [мпиконфигуратион](/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration), [питорчконфигуратион](/python/api/azureml-core/azureml.core.runconfig.pytorchconfiguration)или [тенсорфловконфигуратион](/python/api/azureml-core/azureml.core.runconfig.tensorflowconfiguration) для распределенных заданий соответствующих типов.

В следующем примере настраивается задание обучения PyTorch для использования распределенного обучения с помощью MPI или хоровод.
```python
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      compute_target=compute_target,
                      environment=pytorch_env,
                      distributed_job_config=MpiConfiguration(node_count=2, process_count_per_node=2))
```

Дополнительные сведения см. в разделе:
* [Распределенное обучение с использованием PyTorch](how-to-train-pytorch.md#distributed-training)
* [Распределенное обучение с помощью TensorFlow](how-to-train-tensorflow.md#distributed-training)

## <a name="miscellaneous"></a>Прочее
Если вам нужно получить доступ к базовому объекту RunConfiguration для Скриптрунконфиг по любой причине, это можно сделать следующим образом:
```
src.run_config
```

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка и отправка запуска на выполнение обучения](how-to-set-up-training-targets.md)