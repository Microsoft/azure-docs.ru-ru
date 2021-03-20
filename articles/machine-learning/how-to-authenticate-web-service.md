---
title: Настройка проверки подлинности для моделей, развернутых в виде веб-служб
titleSuffix: Azure Machine Learning
description: Узнайте, как настроить проверку подлинности для моделей машинного обучения, развернутых в веб-службах в Машинное обучение Azure.
services: machine-learning
author: cjgronlund
ms.author: cgronlun
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: core
ms.date: 11/06/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 5bd938fce347d439c2acb4e3fcace04d5b27d770
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94447779"
---
# <a name="configure-authentication-for-models-deployed-as-web-services"></a>Настройка проверки подлинности для моделей, развернутых в виде веб-служб

Машинное обучение Azure позволяет развертывать обученные модели машинного обучения как веб-службы. Из этой статьи вы узнаете, как настроить проверку подлинности для этих развертываний.

Развертывания модели, созданные Машинное обучение Azure, можно настроить для использования одного из двух методов проверки подлинности:

* **на основе ключа**: статический ключ используется для проверки подлинности веб-службы.
* **на основе токенов**: временный маркер должен быть получен из рабочей области машинное обучение Azure (с помощью Azure Active Directory) и использоваться для проверки подлинности в веб-службе. Срок действия маркера истекает через определенный период времени, и его необходимо обновить, чтобы продолжить работу с веб-службой.

    > [!NOTE]
    > Проверка подлинности на основе маркеров доступна только при развертывании в службе Kubernetes Azure.

## <a name="key-based-authentication"></a>Проверка подлинности на основе ключей

Веб-службы, развернутые в службе Azure Kubernetes Service (AKS), по умолчанию *включены* в проверку подлинности на основе ключей.

Развернутые службы "экземпляры контейнеров Azure" (ACI) по умолчанию *отключили* проверку подлинности на основе ключей, но ее можно включить, задав `auth_enabled=True` при создании веб-службы ACI. Следующий код является примером создания конфигурации развертывания ACI с включенной проверкой подлинности на основе ключей.

```python
from azureml.core.webservice import AciWebservice

aci_config = AciWebservice.deploy_configuration(cpu_cores = 1,
                                                memory_gb = 1,
                                                auth_enabled=True)
```

После этого можно будет использовать настраиваемую конфигурацию ACI в развертывании с помощью класса `Model`.

```python
from azureml.core.model import Model, InferenceConfig


inference_config = InferenceConfig(entry_script="score.py",
                                   environment=myenv)
aci_service = Model.deploy(workspace=ws,
                       name="aci_service_sample",
                       models=[model],
                       inference_config=inference_config,
                       deployment_config=aci_config)
aci_service.wait_for_deployment(True)
```

Чтобы получить ключи проверки подлинности, используйте `aci_service.get_keys()`. Чтобы повторно создать ключ, используйте функцию `regen_key()` и передайте значение **Primary** или **Secondary**.

```python
aci_service.regen_key("Primary")
# or
aci_service.regen_key("Secondary")
```

## <a name="token-based-authentication"></a>Проверка подлинности на основе токенов

При включении проверки подлинности с помощью маркеров для веб-службы пользователи должны предоставить веб-службе Машинного обучения Azure маркер JWT для доступа к ней. Срок действия маркера безопасности истекает через заданное время, и его необходимо обновить, чтобы продолжить выполнение вызовов.

* Проверка подлинности на основе маркеров безопасности **включена по умолчанию** при развертывании в Службе Azure Kubernetes.
* При развертывании в Экземплярах контейнеров Azure проверка подлинности на основе маркеров безопасности **не поддерживается**.
* Аутентификация по маркеру **не может использоваться одновременно с проверкой подлинности на основе ключей**.

Чтобы управлять проверкой подлинности на маркере, используйте `token_auth_enabled` параметр при создании или обновлении развертывания:

```python
from azureml.core.webservice import AksWebservice
from azureml.core.model import Model, InferenceConfig

# Create the config
aks_config = AksWebservice.deploy_configuration()

#  Enable token auth and disable (key) auth on the webservice
aks_config = AksWebservice.deploy_configuration(token_auth_enabled=True, auth_enabled=False)

aks_service_name ='aks-service-1'

# deploy the model
aks_service = Model.deploy(workspace=ws,
                           name=aks_service_name,
                           models=[model],
                           inference_config=inference_config,
                           deployment_config=aks_config,
                           deployment_target=aks_target)

aks_service.wait_for_deployment(show_output = True)
```

Если проверка подлинности на основе токенов включена, можно использовать метод `get_token` для получения маркера JWT и его срока действия:

> [!TIP]
> Если вы используете субъект-службу для получения токена и хотите, чтобы он имел минимальный необходимый доступ для получения маркера, назначьте его роли **читателя** для рабочей области.

```python
token, refresh_by = aks_service.get_token()
print(token)
```

> [!IMPORTANT]
> После истечения срока действия маркера безопасности `refresh_by` необходимо запрашивать новый маркер. Если необходимо обновить маркеры за пределами пакета SDK для Python, можно использовать REST API с проверкой подлинности на основе субъекта-службы, чтобы периодически выполнять вызов `service.get_token()`, как обсуждалось ранее.
>
> Настоятельно рекомендуется создать рабочую область Машинного обучения Azure в том же регионе, что и кластер службы Azure Kubernetes.
>
> Для проверки подлинности с помощью маркера веб-служба выполняет вызов в регион, в котором создана рабочая область Машинного обучения Azure. Если регион рабочей области недоступен, вы не сможете получить маркер для веб-службы, даже если ваш кластер находится в другом регионе из рабочей области. В результате аутентификация Azure AD будет недоступен, пока регион рабочей области не будет снова доступен.
>
> Кроме того, чем больше расстояние между регионом кластера и регионом рабочей области, тем дольше будет получена лексема.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о проверке подлинности в развернутой модели см. в разделе [Создание клиента для модели, развернутой в качестве веб-службы](how-to-consume-web-service.md).