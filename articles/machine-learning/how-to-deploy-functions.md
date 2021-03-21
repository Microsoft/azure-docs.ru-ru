---
title: Развертывание моделей ML в приложениях функций Azure (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать Машинное обучение Azure для упаковки и развертывания модели в качестве веб-службы в приложении "функции Azure".
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: vaidyas
author: vaidya-s
ms.reviewer: larryfr
ms.date: 03/06/2020
ms.topic: conceptual
ms.custom: how-to, racking-python, devx-track-azurecli
ms.openlocfilehash: abb38ebbdacdf4f153148fbe121e54ede88f066a
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519271"
---
# <a name="deploy-a-machine-learning-model-to-azure-functions-preview"></a>Развертывание модели машинного обучения в функциях Azure (Предварительная версия)


Узнайте, как развернуть модель из Машинное обучение Azure как приложения-функции в службе "функции Azure".

> [!IMPORTANT]
> Хотя функции Машинное обучение Azure и Azure общедоступны, возможность упаковать модель из службы Машинное обучение для функций доступна в предварительной версии.

С помощью Машинное обучение Azure можно создавать образы DOCKER из обученных моделей машинного обучения. Машинное обучение Azure теперь имеет функции предварительной версии для создания этих моделей машинного обучения в приложениях функций, которые можно [развернуть в функциях Azure](../azure-functions/functions-deployment-technologies.md#docker-container).

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [Создание рабочей области](how-to-manage-workspace.md) .
* [Интерфейс командной строки Azure](/cli/azure/install-azure-cli).
* Обученная модель машинного обучения, зарегистрированная в вашей рабочей области. Если модель отсутствует, используйте [учебник по классификации образов: обучение модели](tutorial-train-models-with-aml.md) для обучения и регистрации.

    > [!IMPORTANT]
    > В фрагментах кода в этой статье предполагается, что вы установили следующие переменные:
    >
    > * `ws` — Рабочая область Машинное обучение Azure.
    > * `model` — Зарегистрированная модель, которая будет развернута.
    > * `inference_config` — Конфигурация вывода для модели.
    >
    > Дополнительные сведения об установке этих переменных см. в разделе [Развертывание моделей с помощью машинное обучение Azure](how-to-deploy-and-where.md).

## <a name="prepare-for-deployment"></a>Подготовка к развертыванию

Перед развертыванием необходимо определить, что нужно для запуска модели в качестве веб-службы. В следующем списке описываются основные элементы, необходимые для развертывания.

* __Скрипт записи__. Этот скрипт принимает запросы, оценивает запрос с помощью модели и возвращает результаты.

    > [!IMPORTANT]
    > Сценарий записи зависит от модели. Он должен понимать формат данных входящего запроса, формат данных, ожидаемых моделью, и формат данных, возвращаемых клиентам.
    >
    > Если данные запроса имеют формат, непригодный для использования в вашей модели, скрипт может преобразовать его в допустимый формат. Он также может преобразовать ответ перед возвратом клиенту.
    >
    > По умолчанию при упаковке для функций входные данные обрабатываются как текст. Если вы заинтересованы в использовании необработанных байт входных данных (например, для триггеров BLOB-объектов), следует использовать [амлрекуест, чтобы принимать необработанные данные](./how-to-deploy-advanced-entry-script.md#binary-data).

Дополнительные сведения о скрипте входа см. в разделе [Определение кода оценки](./how-to-deploy-and-where.md#define-an-entry-script) .

* **Зависимости**, такие как вспомогательные скрипты или пакеты Python или Conda, необходимые для запуска сценария записи или модели

Эти сущности инкапсулированы в __конфигурацию вывода__. В конфигурации зависимостей указываются скрипт входа и другие зависимости.

> [!IMPORTANT]
> При создании конфигурации вывода для использования с функциями Azure необходимо использовать объект [среды](/python/api/azureml-core/azureml.core.environment%28class%29) . Обратите внимание, что при определении пользовательской среды необходимо добавить azureml-Defaults с версией >= 1.0.45 в качестве зависимости PIP. Этот пакет содержит функции, необходимые для размещения модели в качестве веб-службы. В следующем примере демонстрируется создание объекта среды и его использование с конфигурацией вывода:
>
> ```python
> from azureml.core.environment import Environment
> from azureml.core.conda_dependencies import CondaDependencies
>
> # Create an environment and add conda dependencies to it
> myenv = Environment(name="myenv")
> # Enable Docker based environment
> myenv.docker.enabled = True
> # Build conda dependencies
> myenv.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'],
>                                                            pip_packages=['azureml-defaults'])
> inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
> ```

Дополнительные сведения о средах см. в статье [создание сред для обучения и развертывания и управление ими](how-to-use-environments.md).

Дополнительные сведения о конфигурации вывода см. в разделе [Развертывание моделей с помощью машинное обучение Azure](how-to-deploy-and-where.md).

> [!IMPORTANT]
> При развертывании в функциях не нужно создавать __конфигурацию развертывания__.

## <a name="install-the-sdk-preview-package-for-functions-support"></a>Установка пакета SDK для поддержки функций

Чтобы создать пакеты для функций Azure, необходимо установить пакет SDK предварительной версии.

```bash
pip install azureml-contrib-functions
```

## <a name="create-the-image"></a>Создание образа

Чтобы создать образ DOCKER, развернутый в функциях Azure, используйте [azureml. от участников сообщества. functions. Package](/python/api/azureml-contrib-functions/azureml.contrib.functions) или конкретную функцию пакета для триггера, который вы хотите использовать. В следующем фрагменте кода показано, как создать новый пакет с триггером большого двоичного объекта из модели и конфигурации вывода:

> [!NOTE]
> В фрагменте кода предполагается, что `model` содержит зарегистрированную модель и `inference_config` содержит конфигурацию для среды вывода. Дополнительные сведения см. в разделе [Развертывание моделей с помощью машинное обучение Azure](how-to-deploy-and-where.md).

```python
from azureml.contrib.functions import package
from azureml.contrib.functions import BLOB_TRIGGER
blob = package(ws, [model], inference_config, functions_enabled=True, trigger=BLOB_TRIGGER, input_path="input/{blobname}.json", output_path="output/{blobname}_out.json")
blob.wait_for_creation(show_output=True)
# Display the package location/ACR path
print(blob.location)
```

Если `show_output=True` задано значение, выводятся выходные данные процесса сборки DOCKER. После завершения процесса образ будет создан в реестре контейнеров Azure для вашей рабочей области. После построения образа в реестре контейнеров Azure отобразится расположение. Возвращаемое расположение имеет формат `<acrinstance>.azurecr.io/package@sha256:<imagename>` .

> [!NOTE]
> Упаковка для функций в настоящее время поддерживает триггеры HTTP, триггеры больших двоичных объектов и триггеры служебной шины. Дополнительные сведения о триггерах см. в статье [привязки функций Azure](../azure-functions/functions-bindings-storage-blob-trigger.md#blob-name-patterns).

> [!IMPORTANT]
> Сохраните сведения о расположении, так как они используются при развертывании образа.

## <a name="deploy-image-as-a-web-app"></a>Развертывание образа как веб-приложения

1. Используйте следующую команду, чтобы получить учетные данные входа для реестра контейнеров Azure, содержащего образ. Замените на `<myacr>` значение, возвращенное ранее из `blob.location` : 

    ```azurecli-interactive
    az acr credential show --name <myacr>
    ```

    Выходные данные этой команды похожи на следующий документ JSON:

    ```json
    {
    "passwords": [
        {
        "name": "password",
        "value": "Iv0lRZQ9762LUJrFiffo3P4sWgk4q+nW"
        },
        {
        "name": "password2",
        "value": "=pKCxHatX96jeoYBWZLsPR6opszr==mg"
        }
    ],
    "username": "myml08024f78fd10"
    }
    ```

    Сохраните значение __username__ и один из __паролей__.

1. Если у вас еще нет группы ресурсов или плана службы приложений для развертывания службы, в следующих командах показано, как создать оба:

    ```azurecli-interactive
    az group create --name myresourcegroup --location "West Europe"
    az appservice plan create --name myplanname --resource-group myresourcegroup --sku B1 --is-linux
    ```

    В этом примере используется ценовая категория _Linux Basic_ ( `--sku B1` ).

    > [!IMPORTANT]
    > Образы, созданные Машинное обучение Azure используют Linux, поэтому необходимо использовать `--is-linux` параметр.

1. Создайте учетную запись хранения, которая будет использоваться для хранения веб-заданий, и получите ее строку подключения. Замените на `<webjobStorage>` имя, которое вы хотите использовать.

    ```azurecli-interactive
    az storage account create --name <webjobStorage> --location westeurope --resource-group myresourcegroup --sku Standard_LRS
    ```
    ```azurecli-interactive
    az storage account show-connection-string --resource-group myresourcegroup --name <webJobStorage> --query connectionString --output tsv
    ```

1. Чтобы создать приложение функции, используйте следующую команду. Замените на `<app-name>` имя, которое вы хотите использовать. Замените `<acrinstance>` и `<imagename>` значениями, полученными `package.location` ранее. Замените на `<webjobStorage>` имя учетной записи хранения из предыдущего шага:

    ```azurecli-interactive
    az functionapp create --resource-group myresourcegroup --plan myplanname --name <app-name> --deployment-container-image-name <acrinstance>.azurecr.io/package:<imagename> --storage-account <webjobStorage>
    ```

    > [!IMPORTANT]
    > На этом этапе приложение функции создано. Однако, так как вы не указали строку подключения для триггера большого двоичного объекта или учетных данных в реестре контейнеров Azure, который содержит образ, приложение-функция неактивно. На следующих шагах вы укажите строку подключения и сведения для проверки подлинности для реестра контейнеров. 

1. Создайте учетную запись хранения, которая будет использоваться для хранилища триггеров BLOB-объектов, и получите ее строку подключения. Замените на `<triggerStorage>` имя, которое вы хотите использовать.

    ```azurecli-interactive
    az storage account create --name <triggerStorage> --location westeurope --resource-group myresourcegroup --sku Standard_LRS
    ```
    ```azurecli-interactive
    az storage account show-connection-string --resource-group myresourcegroup --name <triggerStorage> --query connectionString --output tsv
    ```
    Запишите эту строку подключения, чтобы предоставить приложению функции. Он будет использоваться позже при запросе `<triggerConnectionString>`

1. Создайте контейнеры для входных и выходных данных в учетной записи хранения. Замените на `<triggerConnectionString>` строку подключения, возвращенную ранее:

    ```azurecli-interactive
    az storage container create -n input --connection-string <triggerConnectionString>
    ```
    ```azurecli-interactive
    az storage container create -n output --connection-string <triggerConnectionString>
    ```

1. Чтобы связать строку подключения триггера с приложением функции, используйте следующую команду. Замените на `<app-name>` имя приложения функции. Замените на `<triggerConnectionString>` строку подключения, возвращенную ранее:

    ```azurecli-interactive
    az functionapp config appsettings set --name <app-name> --resource-group myresourcegroup --settings "TriggerConnectionString=<triggerConnectionString>"
    ```
1. Необходимо получить тег, связанный с созданным контейнером, с помощью следующей команды. Замените на `<username>` имя пользователя, возвращенное ранее из реестра контейнеров:

    ```azurecli-interactive
    az acr repository show-tags --repository package --name <username> --output tsv
    ```
    Сохраните возвращенное значение, оно будет использоваться как на `imagetag` следующем шаге.

1. Чтобы предоставить приложению функции учетные данные, необходимые для доступа к реестру контейнеров, используйте следующую команду. Замените на `<app-name>` имя приложения функции. Замените `<acrinstance>` и `<imagetag>` значениями из вызова AZ CLI на предыдущем шаге. Замените `<username>` и `<password>` на сведения об имени входа записи контроля доступа, полученные ранее:

    ```azurecli-interactive
    az functionapp config container set --name <app-name> --resource-group myresourcegroup --docker-custom-image-name <acrinstance>.azurecr.io/package:<imagetag> --docker-registry-server-url https://<acrinstance>.azurecr.io --docker-registry-server-user <username> --docker-registry-server-password <password>
    ```

    Эта команда возвращает сведения, аналогичные следующему документу JSON:

    ```json
    [
    {
        "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
        "slotSetting": false,
        "value": "false"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_URL",
        "slotSetting": false,
        "value": "https://myml08024f78fd10.azurecr.io"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_USERNAME",
        "slotSetting": false,
        "value": "myml08024f78fd10"
    },
    {
        "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
        "slotSetting": false,
        "value": null
    },
    {
        "name": "DOCKER_CUSTOM_IMAGE_NAME",
        "value": "DOCKER|myml08024f78fd10.azurecr.io/package:20190827195524"
    }
    ]
    ```

На этом этапе приложение функции начинает загрузку образа.

> [!IMPORTANT]
> Загрузка образа может занять несколько минут. Ход выполнения можно отслеживать с помощью портала Azure.

## <a name="test-the-deployment"></a>тестирование развертывания

После загрузки образа и доступности приложения выполните следующие действия, чтобы запустить приложение.

1. Создайте текстовый файл, содержащий данные, которые требуются для файла score.py. Следующий пример будет работать с score.py, который предполагает массив из 10 чисел:

    ```json
    {"data": [[1, 2, 3, 4, 5, 6, 7, 8, 9, 10], [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]]}
    ```

    > [!IMPORTANT]
    > Формат данных зависит от того, что предполагает score.py и модель.

2. Используйте следующую команду, чтобы передать этот файл в контейнер ввода в созданном ранее BLOB-объекте хранилища триггеров. Замените на `<file>` имя файла, содержащего данные. Замените на `<triggerConnectionString>` строку подключения, возвращенную ранее. В этом примере `input` — это имя созданного ранее контейнера входных данных. Если вы использовали другое имя, замените это значение:

    ```azurecli-interactive
    az storage blob upload --container-name input --file <file> --name <file> --connection-string <triggerConnectionString>
    ```

    Результат этой команды аналогичен следующему коду JSON:

    ```json
    {
    "etag": "\"0x8D7C21528E08844\"",
    "lastModified": "2020-03-06T21:27:23+00:00"
    }
    ```

3. Чтобы просмотреть выходные данные, созданные функцией, используйте следующую команду, чтобы получить список выходных файлов. Замените на `<triggerConnectionString>` строку подключения, возвращенную ранее. В этом примере `output` — это имя выходного контейнера, созданного ранее. Если вы использовали другое имя, замените это значение:

    ```azurecli-interactive
    az storage blob list --container-name output --connection-string <triggerConnectionString> --query '[].name' --output tsv
    ```

    Выходные данные этой команды похожи на `sample_input_out.json` .

4. Чтобы скачать файл и проверить его содержимое, используйте следующую команду. Замените на `<file>` имя файла, возвращенное предыдущей командой. Замените на `<triggerConnectionString>` строку подключения, возвращенную ранее: 

    ```azurecli-interactive
    az storage blob download --container-name output --file <file> --name <file> --connection-string <triggerConnectionString>
    ```

    После выполнения команды откройте файл. Он содержит данные, возвращенные моделью.

Дополнительные сведения об использовании триггеров больших двоичных объектов см. в статье [Создание функции, активируемой с помощью хранилища BLOB-объектов Azure](../azure-functions/functions-create-storage-blob-triggered-function.md) .

## <a name="next-steps"></a>Дальнейшие действия

* Научитесь настраивать приложение функций в документации по [функциям](../azure-functions/functions-create-function-linux-custom-image.md) .
* Дополнительные сведения о службе хранилища BLOB-объектов см. в статье триггеры [хранилища BLOB-объектов Azure](../azure-functions/functions-bindings-storage-blob.md).
* [Разверните модель в службе приложений Azure](how-to-deploy-app-service.md).
* [Использование модели Машинного обучения Azure, развернутой в качестве веб-службы](how-to-consume-web-service.md)
* [Справочник по интерфейсам API](/python/api/azureml-contrib-functions/azureml.contrib.functions)
