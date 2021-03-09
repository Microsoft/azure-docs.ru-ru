---
title: Развертывание модели машинного обучения в службе "функции Azure" с помощью кэша Azure для Redis
description: В этой статье вы развернете модель из Машинное обучение Azure как приложение-функцию в службе "функции Azure" с помощью кэша Azure для экземпляра Redis. Кэш Azure для Redis является чрезвычайно производительным и масштабируемым — при связывании с моделью Машинное обучение Azure вы получаете низкую задержку и высокую пропускную способность в приложении.
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 09/30/2020
ms.openlocfilehash: ec8943bc73cac2020350dd4916f040f031cd842b
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102499702"
---
# <a name="deploy-a-machine-learning-model-to-azure-functions-with-azure-cache-for-redis"></a>Развертывание модели машинного обучения в службе "функции Azure" с помощью кэша Azure для Redis 

В этой статье вы развернете модель из Машинное обучение Azure как приложение-функцию в службе "функции Azure" с помощью кэша Azure для экземпляра Redis.  

Кэш Azure для Redis является чрезвычайно производительным и масштабируемым — при связывании с моделью Машинное обучение Azure вы получаете низкую задержку и высокую пропускную способность в приложении. Несколько сценариев, в которых кэш особенно полезен, заключается в том, что при указании данных и для реальных результатов определения модели. В любом сценарии метаданные или результаты хранятся в памяти, что приводит к повышению производительности. 

> [!NOTE]
> Хотя функции Машинное обучение Azure и Azure общедоступны, возможность упаковать модель из службы Машинное обучение для функций доступна в предварительной версии.  
>

## <a name="prerequisites"></a>Предварительные требования
* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
* Рабочая область машинного обучения Azure. Дополнительные сведения см. в статье [Создание рабочей области](../machine-learning/how-to-manage-workspace.md) .
* [Azure CLI](/cli/azure/install-azure-cli).
* Обученная модель машинного обучения, зарегистрированная в вашей рабочей области. Если модель отсутствует, используйте [учебник по классификации образов: обучение модели](../machine-learning/tutorial-train-models-with-aml.md) для обучения и регистрации.

> [!IMPORTANT]
> В фрагментах кода в этой статье предполагается, что вы установили следующие переменные:
>
> * `ws` — Рабочая область Машинное обучение Azure.
> * `model` — Зарегистрированная модель, которая будет развернута.
> * `inference_config` — Конфигурация вывода для модели.
>
> Дополнительные сведения об установке этих переменных см. в разделе [Развертывание моделей с помощью машинное обучение Azure](../machine-learning/how-to-deploy-and-where.md).

## <a name="create-an-azure-cache-for-redis-instance"></a>Создание экземпляра кэша Azure для Redis 
Вы сможете развернуть модель машинного обучения в службе "функции Azure" с любым экземпляром кэша Basic, Standard или Premium. Чтобы создать экземпляр кэша, выполните следующие действия.  

1. Перейдите на домашнюю страницу портал Azure или откройте меню боковой панели, а затем выберите **создать ресурс**. 
   
1. На странице **Создание** выберите **Базы данных**, а затем **Кэш Azure для Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Выбор элемента &quot;Кэш Azure для Redis&quot;.":::
   
1. На странице **Новый кэш Redis** настройте параметры для нового кэша.
   
   | Параметр      | Рекомендуемое значение  | Description |
   | ------------ |  ------- | -------------------------------------------------- |
   | **DNS-имя** | Введите глобально уникальное имя | Имя кэша должно быть строкой длиной от 1 до 63 символов и содержать только цифры, буквы и дефисы. Имя должно начинаться и заканчиваться цифрой или буквой и не может содержать более одного дефиса подряд. *Имя узла* для экземпляра кэша получит значение *\<DNS name>.redis.cache.windows.net*. | 
   | **подписка** | Раскройте список и выберите нужную подписку. | В этой подписке будет создан новый экземпляр кэша Redis для Azure. | 
   | **Группа ресурсов** | Раскройте список и выберите группу ресурсов или нажмите **Создать** и введите имя для новой группы ресурсов. | Имя группы ресурсов, в которой будут созданы кэш и другие ресурсы. Поместив все ресурсы приложения в одну группу ресурсов, вы сможете легко управлять ими и/или удалить их вместе. | 
   | **Местоположение** | Откройте список и выберите расположение. | Выберите оптимальный [регион](https://azure.microsoft.com/regions/) для других служб, которые будут использовать кэш. |
   | **Ценовая категория** | Откройте список и выберите [ценовую категорию](https://azure.microsoft.com/pricing/details/cache/). |  Ценовая категория определяет размер, производительность и функции, доступные для кэша. Дополнительные сведения см. в [обзоре предложения "Кэш Redis для Azure"](cache-overview.md). |

1. Выберите вкладку **Сети** или нажмите кнопку **Сети** в нижней части страницы.

1. На вкладке **Сети** выберите способ подключения.

1. Нажмите кнопку **Далее: дополнительно** в нижней части страницы или выберите вкладку **Далее: дополнительно**.

1. На вкладке **Дополнительно** для экземпляра кэша уровня "Базовый" или "Стандартный" установите переключатель, чтобы включить порт, отличный от TLS.

1. На вкладке **Дополнительно** для экземпляра кэша уровня "Премиум" настройте параметры для портов, отличных от TLS, а также кластеризацию и сохраняемость данных.

1. Нажмите кнопку **Далее: теги** или нажмите кнопку **Далее: теги** в нижней части страницы.

1. При необходимости на вкладке **Теги** введите имя и значение, чтобы классифицировать ресурс. 

1. Выберите **Проверить и создать**. Вы будете перенаправлены на вкладку "Просмотр и создание", где Azure проверит вашу конфигурацию.

1. Когда отобразится сообщение "Проверка пройдена" зеленого цвета, выберите **Создать**.

На создание кэша требуется некоторое время. Вы можете отслеживать ход выполнения на странице **обзорных сведений** кэша Azure для Redis. Когда **Состояние** примет значение **Running** (Выполняется), кэш будет готов к использованию. 

## <a name="prepare-for-deployment"></a>Подготовка к развертыванию

Перед развертыванием необходимо определить, что нужно для запуска модели в качестве веб-службы. В следующем списке описываются основные элементы, необходимые для развертывания.

* __Скрипт записи__. Этот скрипт принимает запросы, оценивает запрос с помощью модели и возвращает результаты.

    > [!IMPORTANT]
    > Сценарий записи зависит от модели. Он должен понимать формат данных входящего запроса, формат данных, ожидаемых моделью, и формат данных, возвращаемых клиентам.
    >
    > Если данные запроса имеют формат, непригодный для использования в вашей модели, скрипт может преобразовать его в допустимый формат. Он также может преобразовать ответ перед его возвратом клиенту.
    >
    > По умолчанию при упаковке для функций входные данные обрабатываются как текст. Если вы заинтересованы в использовании необработанных байт входных данных (например, для триггеров BLOB-объектов), следует использовать [амлрекуест, чтобы принимать необработанные данные](../machine-learning/how-to-deploy-advanced-entry-script.md#binary-data).

Для функции Run убедитесь, что она подключается к конечной точке Redis.

```python
import json
import numpy as np
import os
import redis
from sklearn.externals import joblib

def init():
    global model
    global azrediscache
    azrediscache = redis.StrictRedis(host='<host_url>', port=6380, password="<access_key>", ssl=True)
    model_path = os.path.join(os.getenv('AZUREML_MODEL_DIR'), 'sklearn_mnist_model.pkl')
    model = joblib.load(model_path)

@input_schema('data', NumpyParameterType(input_sample))
@output_schema(NumpyParameterType(output_sample))
def run(data):
    try:
        input = azrediscache.get(data)
        result = model.predict(input)
        data = np.array(json.loads(data))
        result = model.predict(data)
        # You can return any data type, as long as it is JSON serializable.
        return result.tolist()
    except Exception as e:
        error = str(e)
        return error
```

Дополнительные сведения о скрипте входа см. в разделе [Определение кода оценки.](../machine-learning/how-to-deploy-and-where.md?tabs=python#define-an-entry-script)

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
>                                                            pip_packages=['azureml-defaults', 'redis'])
> inference_config = InferenceConfig(entry_script="score.py", environment=myenv)
> ```

Дополнительные сведения о средах см. в статье [создание сред для обучения и развертывания и управление ими](../machine-learning/how-to-use-environments.md).

Дополнительные сведения о конфигурации вывода см. в разделе [Развертывание моделей с помощью машинное обучение Azure](../machine-learning/how-to-deploy-and-where.md?tabs=python#define-an-inference-configuration).

> [!IMPORTANT]
> При развертывании в функциях не нужно создавать __конфигурацию развертывания__.

## <a name="install-the-sdk-preview-package-for-functions-support"></a>Установка пакета SDK для поддержки функций

Чтобы создать пакеты для функций Azure, необходимо установить пакет SDK предварительной версии.

```bash
pip install azureml-contrib-functions
```

## <a name="create-the-image"></a>Создание образа

Чтобы создать образ DOCKER, развернутый в функциях Azure, используйте [azureml. от участников сообщества. functions. Package](/python/api/azureml-contrib-functions/azureml.contrib.functions) или конкретную функцию пакета для триггера, который вы хотите использовать. В следующем фрагменте кода показано, как создать новый пакет с триггером HTTP из модели и конфигурации вывода:

> [!NOTE]
> В фрагменте кода предполагается, что `model` содержит зарегистрированную модель и `inference_config` содержит конфигурацию для среды вывода. Дополнительные сведения см. в разделе [Развертывание моделей с помощью машинное обучение Azure](../machine-learning/how-to-deploy-and-where.md).

```python
from azureml.contrib.functions import package
from azureml.contrib.functions import HTTP_TRIGGER
model_package = package(ws, [model], inference_config, functions_enabled=True, trigger=HTTP_TRIGGER)
model_package.wait_for_creation(show_output=True)
# Display the package location/ACR path
print(model_package.location)
```

Если `show_output=True` задано значение, выводятся выходные данные процесса сборки DOCKER. После завершения процесса образ будет создан в реестре контейнеров Azure для вашей рабочей области. После построения образа в реестре контейнеров Azure отобразится расположение. Возвращаемое расположение имеет формат `<acrinstance>.azurecr.io/package@sha256:<imagename>` .

> [!NOTE]
> Упаковка для функций в настоящее время поддерживает триггеры HTTP, триггеры больших двоичных объектов и триггеры служебной шины. Дополнительные сведения о триггерах см. в статье [привязки функций Azure](../azure-functions/functions-bindings-storage-blob-trigger.md#blob-name-patterns).

> [!IMPORTANT]
> Сохраните сведения о расположении, так как они используются при развертывании образа.

## <a name="deploy-image-as-a-web-app"></a>Развертывание образа как веб-приложения

1. Используйте следующую команду, чтобы получить учетные данные входа для реестра контейнеров Azure, содержащего образ. Замените на `<myacr>` значение, возвращенное ранее из `package.location` : 

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
    > На этом этапе приложение функции создано. Однако так как вы не указали строку подключения для триггера HTTP или учетных данных в реестре контейнеров Azure, который содержит образ, приложение-функция неактивно. На следующих шагах вы укажите строку подключения и сведения для проверки подлинности для реестра контейнеров. 

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
> Загрузка образа может занять несколько минут. Ход выполнения можно отслеживать с помощью портал Azure.

## <a name="test-azure-functions-http-trigger"></a>Проверка триггера HTTP для функций Azure 

Теперь мы выполним и протестируем триггер HTTP для функций Azure.

1. Перейдите к приложению функции в портал Azure.
1. В разделе Разработчик выберите **код + тест**. 
1. На правой стороне выберите вкладку **входные данные** . 
1. Нажмите кнопку " **выполнить** ", чтобы проверить триггер HTTP для функций Azure. 

Теперь вы успешно развернули модель из Машинное обучение Azure как приложение-функцию, используя кэш Azure для экземпляра Redis. Узнайте больше о кэше Azure для Redis, перейдя по ссылкам в разделе ниже.

## <a name="clean-up-resources"></a>Очистка ресурсов

При переходе к следующему руководству в нем можно использовать ресурсы, созданные в рамках этого руководства.

В противном случае, если вы завершили работу с кратким руководством, вы можете удалить ресурсы Azure, созданные в этом кратком руководстве, чтобы избежать расходов. 

> [!IMPORTANT]
> Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если ресурсы для размещения этого примера созданы в имеющейся группе ресурсов, содержащей ресурсы, которые следует сохранить, можно удалить каждый ресурс отдельно в соответствующих колонках вместо удаления группы ресурсов.

### <a name="to-delete-a-resource-group"></a>Удаление группы ресурсов

1. Войдите на [портал Azure](https://portal.azure.com) и щелкните **Группы ресурсов**.

2. Введите имя группы ресурсов в поле **Фильтровать по имени...** В своей группе ресурсов в списке результатов щелкните **...**, а затем выберите **Удалить группу ресурсов**.

Подтвердите операцию удаления группы ресурсов. Введите имя группы ресурсов, которую необходимо удалить, и щелкните **Удалить**.

Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

## <a name="next-steps"></a>Дальнейшие действия 

* Дополнительные сведения о [кэше Azure для Redis](./cache-overview.md)
* Научитесь настраивать приложение функции в документации по [функциям](../azure-functions/functions-create-function-linux-custom-image.md) .
* [Справочник по интерфейсам API](/python/api/azureml-contrib-functions/azureml.contrib.functions) 
* Создание [приложения Python, использующего кэш Azure для Redis](./cache-python-get-started.md)
