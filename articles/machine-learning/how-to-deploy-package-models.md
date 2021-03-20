---
title: Упаковка моделей
titleSuffix: Azure Machine Learning
description: Упаковка модели. Модели можно упаковать как образ DOCKER, который затем можно загрузить, или создать Dockerfile и использовать его для создания образа.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: 299eb316b534ddc9d5eee934cc15eae841276038
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93312628"
---
# <a name="how-to-package-a-registered-model-with-docker"></a>Упаковка зарегистрированной модели с помощью DOCKER

В этой статье показано, как упаковать зарегистрированную модель Машинное обучение Azure с помощью DOCKER.

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что модель уже обучена и зарегистрирована в рабочей области машинного обучения. Чтобы узнать, как обучить и зарегистрировать модель scikit-учиться, [следуйте указаниям этого руководства](how-to-train-scikit-learn.md).


## <a name="package-models"></a>Упаковка моделей

В некоторых случаях может потребоваться создать образ DOCKER без развертывания модели (например, если планируется [развертывание в службе приложений Azure](how-to-deploy-app-service.md)). Также можно загрузить образ и запустить его в локальной установке DOCKER. Может даже потребоваться загрузить файлы, используемые для создания образа, проверить их, изменить и создать образ вручную.

Упаковка модели позволяет выполнять эти задачи. Он упаковывает все ресурсы, необходимые для размещения модели в качестве веб-службы, и позволяет скачать полностью созданный образ DOCKER или файлы, необходимые для его создания. Существует два способа использования упаковки модели.

**Скачайте упакованную модель:** Скачайте образ DOCKER, содержащий модель и другие файлы, необходимые для размещения его в качестве веб-службы.

**Создайте Dockerfile:** Скачайте Dockerfile, модель, сценарий входа и другие ресурсы, необходимые для создания образа DOCKER. Затем можно проверить файлы или внести изменения, прежде чем создавать образ локально.

Оба пакета можно использовать для получения образа локального DOCKER.

> [!TIP]
> Создание пакета аналогично развертыванию модели. Вы используете зарегистрированную модель и конфигурацию вывода.

> [!IMPORTANT]
> Чтобы загрузить полностью собранный образ или создать образ локально, необходимо установить [DOCKER](https://www.docker.com) в среде разработки.

### <a name="download-a-packaged-model"></a>Загрузка упакованной модели

В следующем примере создается образ, который регистрируется в реестре контейнеров Azure для вашей рабочей области:

```python
package = Model.package(ws, [model], inference_config)
package.wait_for_creation(show_output=True)
```

После создания пакета можно использовать `package.pull()` для извлечения образа в локальную среду DOCKER. Выходные данные этой команды будут отображать имя изображения. Пример: 

`Status: Downloaded newer image for myworkspacef78fd10.azurecr.io/package:20190822181338`. 

После загрузки модели используйте `docker images` команду, чтобы вывести список локальных образов:

```text
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
myworkspacef78fd10.azurecr.io/package    20190822181338      7ff48015d5bd        4 minutes ago       1.43 GB
```

Чтобы запустить локальный контейнер на основе этого образа, используйте следующую команду для запуска именованного контейнера из оболочки или командной строки. Замените `<imageid>` значение идентификатором изображения, возвращенным `docker images` командой.

```bash
docker run -p 6789:5001 --name mycontainer <imageid>
```

Эта команда запускает последнюю версию образа с именем `myimage` . Он сопоставляет локальный порт 6789 с портом в контейнере, прослушиваемом веб-службой (5001). Он также назначает имя `mycontainer` контейнеру, что делает контейнер более легким для его завершения. После запуска контейнера можно отправлять запросы в `http://localhost:6789/score` .

### <a name="generate-a-dockerfile-and-dependencies"></a>Создание Dockerfile и зависимостей

В следующем примере показано, как загрузить Dockerfile, модель и другие ресурсы, необходимые для создания образа локально. `generate_dockerfile=True`Параметр указывает, что требуется, чтобы файлы, а не полностью построенные образы.

```python
package = Model.package(ws, [model], inference_config, generate_dockerfile=True)
package.wait_for_creation(show_output=True)
# Download the package.
package.save("./imagefiles")
# Get the Azure container registry that the model/Dockerfile uses.
acr=package.get_container_registry()
print("Address:", acr.address)
print("Username:", acr.username)
print("Password:", acr.password)
```

Этот код скачивает файлы, необходимые для сборки образа, в `imagefiles` каталог. Dockerfile, включаемые в сохраненные файлы, ссылаются на базовый образ, хранящийся в реестре контейнеров Azure. При создании образа в локальной установке DOCKER необходимо использовать адрес, имя пользователя и пароль для проверки подлинности в реестре. Выполните следующие действия, чтобы создать образ с помощью локальной установки docker:

1. Чтобы проверить подлинность DOCKER с помощью реестра контейнеров Azure, из оболочки или сеанса командной строки используйте следующую команду. Замените `<address>` , `<username>` и `<password>` значениями, полученными `package.get_container_registry()` .

    ```bash
    docker login <address> -u <username> -p <password>
    ```

2. Чтобы создать образ, используйте следующую команду. Замените `<imagefiles>` на путь к каталогу, в котором `package.save()` сохранены файлы.

    ```bash
    docker build --tag myimage <imagefiles>
    ```

    Эта команда задает имя образа в `myimage` .

Чтобы убедиться, что образ построен, используйте `docker images` команду. Вы должны увидеть `myimage` изображение в списке:

```text
REPOSITORY      TAG                 IMAGE ID            CREATED             SIZE
<none>          <none>              2d5ee0bf3b3b        49 seconds ago      1.43 GB
myimage         latest              739f22498d64        3 minutes ago       1.43 GB
```

Чтобы запустить новый контейнер на основе этого образа, используйте следующую команду:

```bash
docker run -p 6789:5001 --name mycontainer myimage:latest
```

Эта команда запускает последнюю версию образа с именем `myimage` . Он сопоставляет локальный порт 6789 с портом в контейнере, прослушиваемом веб-службой (5001). Он также назначает имя `mycontainer` контейнеру, что делает контейнер более легким для его завершения. После запуска контейнера можно отправлять запросы в `http://localhost:6789/score` .

### <a name="example-client-to-test-the-local-container"></a>Пример клиента для тестирования локального контейнера

Следующий код является примером клиента Python, который можно использовать с контейнером:

```python
import requests
import json

# URL for the web service.
scoring_uri = 'http://localhost:6789/score'

# Two sets of data to score, so we get two results back.
data = {"data":
        [
            [ 1,2,3,4,5,6,7,8,9,10 ],
            [ 10,9,8,7,6,5,4,3,2,1 ]
        ]
        }
# Convert to JSON string.
input_data = json.dumps(data)

# Set the content type.
headers = {'Content-Type': 'application/json'}

# Make the request and display the response.
resp = requests.post(scoring_uri, input_data, headers=headers)
print(resp.text)
```

Примеры клиентов на других языках программирования см. в разделе [Использование моделей, развернутых в качестве веб-служб](how-to-consume-web-service.md).

### <a name="stop-the-docker-container"></a>Останавливает контейнер DOCKER

Чтобы прерывать контейнер, используйте следующую команду из другой оболочки или командной строки:

```bash
docker kill mycontainer
```

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок при развертывании](how-to-troubleshoot-deployment.md)
* [развертывание в Службе Azure Kubernetes](how-to-deploy-azure-kubernetes-service.md).
* [Создание клиентских приложений для использования веб-служб](how-to-consume-web-service.md)
* [Обновление веб-службы](how-to-deploy-update-web-service.md)
* [Развертывание модели с помощью пользовательского образа DOCKER](how-to-deploy-custom-docker-image.md)
* [Использование TLS для защиты веб-службы с помощью Машинного обучения Azure](how-to-secure-web-service.md).
* [Мониторинг моделей Машинное обучение Azure с помощью Application Insights](how-to-enable-app-insights.md)
* [Сбор данных для моделей в рабочей среде](how-to-enable-data-collection.md)
* [Создание предупреждений и триггеров событий для развертываний моделей](how-to-use-event-grid.md)
