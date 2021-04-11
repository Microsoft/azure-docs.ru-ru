---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 01/28/2020
ms.author: larryfr
ms.openlocfilehash: a03f71adc99063fee4374b1436b08adf5bab783d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102510936"
---
Записи в документе `inferenceconfig.json` соответствуют параметрам класса [InferenceConfig](/python/api/azureml-core/azureml.core.model.inferenceconfig). В следующей таблице представлено сопоставление сущностей в документе JSON и параметров метода.

| Сущность JSON | Параметр метода | Описание: |
| ----- | ----- | ----- |
| `entryScript` | `entry_script` | Путь к локальному файлу, содержащему код, выполняемый для образа. |
| `sourceDirectory` | `source_directory` | Необязательный элемент. Путь к папкам, содержащим все файлы для создания образа, что упрощает доступ к любым файлам в этой папке или вложенной папке. Вы можете отправить всю папку с локального компьютера в качестве зависимостей для веб-службы. Примечание. Пути entry_script, conda_file и extra_docker_file_steps являются относительными по отношению к пути source_directory. |
| `environment` | `environment` | Необязательный элемент.  [Среда](/python/api/azureml-core/azureml.core.environment.environment) Машинного обучения Azure.|

В файл конфигурации вывода можно включить полные спецификации [среды](/python/api/azureml-core/azureml.core.environment.environment) Машинного обучения Azure. Если эта среда не существует в рабочей области, Машинное обучение Azure создаст ее. В противном случае Машинное обучение Azure обновит среду при необходимости. Приведенный ниже код JSON является примером.

```json
{
    "entryScript": "score.py",
    "environment": {
        "docker": {
            "arguments": [],
            "baseDockerfile": null,
            "baseImage": "mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04",
            "enabled": false,
            "sharedVolumes": true,
            "shmSize": null
        },
        "environmentVariables": {
            "EXAMPLE_ENV_VAR": "EXAMPLE_VALUE"
        },
        "name": "my-deploy-env",
        "python": {
            "baseCondaEnvironment": null,
            "condaDependencies": {
                "channels": [
                    "conda-forge"
                ],
                "dependencies": [
                    "python=3.6.2",
                    {
                        "pip": [
                            "azureml-defaults",
                            "azureml-telemetry",
                            "scikit-learn==0.22.1",
                            "inference-schema[numpy-support]"
                        ]
                    }
                ],
                "name": "project_environment"
            },
            "condaDependenciesFile": null,
            "interpreterPath": "python",
            "userManagedDependencies": false
        },
        "version": "1"
    }
}
```

Вы также можете использовать существующую [среду](/python/api/azureml-core/azureml.core.environment.environment) Машинного обучения Azure в отдельных параметрах CLI и удалить ключ environment из файла конфигурации вывода. Используйте параметр -e для имени среды и --ev для версии среды. Если --ev не указывать, будет использоваться последняя версия. Ниже приведен пример файла конфигурации вывода.

```json
{
    "entryScript": "score.py",
    "sourceDirectory": null
}
```

Следующая команда развертывает модель с использованием предыдущего файла конфигурации вывода (с именем myInferenceConfig.json). 

При этом также используется последняя версия существующей [среды](/python/api/azureml-core/azureml.core.environment.environment) Машинного обучения Azure (с именем AzureML-Minimal).

```azurecli-interactive
az ml model deploy -m mymodel:1 --ic myInferenceConfig.json -e AzureML-Minimal --dc deploymentconfig.json
```