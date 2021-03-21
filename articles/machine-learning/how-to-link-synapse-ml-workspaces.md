---
title: Создание связанной службы с синапсе и Машинное обучение Azure рабочими областями (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как связать рабочие области синапсе и Машинное обучение Azure Azure с единым интерфейсом структурирование данных.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 03/08/2021
ms.custom: how-to, devx-track-python, data4ml, synapse-azureml
ms.openlocfilehash: d1c4defc53c4af0fb481a57c0a455e987fdd480a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520002"
---
# <a name="link-azure-synapse-analytics-and-azure-machine-learning-workspaces-preview"></a>Связывание Azure синапсе Analytics и рабочих областей Машинное обучение Azure (Предварительная версия)

В этой статье вы узнаете, как создать связанную службу, которая связывает рабочую область [Azure синапсе Analytics](/synapse-analytics/overview-what-is.md) и [рабочую область машинное обучение Azure](concept-workspace.md).

С помощью рабочей области Машинное обучение Azure, связанной с рабочей областью Azure синапсе, можно подключить пул Apache Spark в качестве выделенного набора вычислений для структурирование данных в масштабе и провести обучение модели из той же записной книжки.

Вы можете связать рабочую область ML и рабочую область синапсе с помощью [пакета SDK для Python](#link-sdk) или [машинное обучение Azure Studio](#link-studio).

Вы также можете связать рабочие области и подключить пул синапсе Spark с помощью одного [шаблона Azure Resource Manager (ARM)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-machine-learning-linkedservice-create/azuredeploy.json).

>[!IMPORTANT]
> Машинное обучение Azure и интеграция Azure синапсе находятся в общедоступной предварительной версии. Функциональные возможности, представленные в `azureml-synapse` пакете, являются [экспериментальными](/python/api/overview/azure/ml/#stable-vs-experimental) функциями предварительной версии и могут быть изменены в любое время.

## <a name="prerequisites"></a>Предварительные условия

* [Создайте рабочую область Машинного обучения Azure](how-to-manage-workspace.md?tabs=python).

* [Создайте рабочую область синапсе в портал Azure](/synapse-analytics/quickstart-create-workspace.md).

* [Создание пула Apache Spark с помощью портал Azure, веб-средств или синапсе Studio](/synapse-analytics/quickstart-create-apache-spark-pool-portal.md)

* Установка [пакета SDK для машинное обучение Azure Python](/python/api/overview/azure/ml/intro)

* Доступ к [машинное обучение Azure Studio](https://ml.azure.com/).

<a name="link-sdk"></a>
## <a name="link-workspaces-with-the-python-sdk"></a>Связывание рабочих областей с пакетом SDK для Python

> [!IMPORTANT]
> Для успешного связывания с рабочей областью синапсе необходимо иметь роль **владельца** рабочей области синапсе. Проверьте доступ в [портал Azure](https://ms.portal.azure.com/).
>
> Если вы не являетесь **владельцем** и являетесь **участником** рабочей области синапсе, можно использовать только существующие связанные службы. Узнайте, как [повторить попытку и использовать существующую связанную службу](how-to-data-prep-synapse-spark-pool.md#get-an-existing-linked-service).

В следующем коде [`LinkedService`](/python/api/azureml-core/azureml.core.linked_service.linkedservice) классы и используются [`SynapseWorkspaceLinkedServiceConfiguration`](/python/api/azureml-core/azureml.core.linked_service.synapseworkspacelinkedserviceconfiguration) в,

* Свяжите рабочую область машинного обучения `ws` с рабочей областью Azure синапсе.
* Зарегистрируйте рабочую область синапсе с Машинное обучение Azure в качестве связанной службы.

``` python
import datetime  
from azureml.core import Workspace, LinkedService, SynapseWorkspaceLinkedServiceConfiguration

# Azure Machine Learning workspace
ws = Workspace.from_config()

#link configuration 
synapse_link_config = SynapseWorkspaceLinkedServiceConfiguration(
    subscription_id=ws.subscription_id,
    resource_group= 'your resource group',
    name='mySynapseWorkspaceName')

# Link workspaces and register Synapse workspace in Azure Machine Learning
linked_service = LinkedService.register(workspace = ws,              
                                            name = 'synapselink1',    
                                            linked_service_config = synapse_link_config)
```

> [!IMPORTANT] 
> Управляемое удостоверение `system_assigned_identity_principal_id` создается для каждой связанной службы. Перед началом сеанса синапсе для этого управляемого удостоверения необходимо предоставить роль **администратора синапсе Apache Spark** рабочей области синапсе. [Назначьте управляемому удостоверению роль администратора синапсе Apache Spark в синапсе Studio](../synapse-analytics/security/how-to-manage-synapse-rbac-role-assignments.md).
>
> Чтобы найти `system_assigned_identity_principal_id` конкретную связанную службу, используйте `LinkedService.get('<your-mlworkspace-name>', '<linked-service-name>')` .

### <a name="manage-linked-services"></a>Управление связанными службами

Просмотрите все связанные службы, связанные с рабочей областью машинного обучения.

```python
LinkedService.list(ws)
```

Чтобы удалить связь с рабочими областями, используйте `unregister()` метод.

``` python
linked_service.unregister()
```

<a name="link-studio"></a>
## <a name="link-workspaces-via-studio"></a>Связывание рабочих областей с помощью студии

Свяжите рабочую область машинного обучения и рабочую область синапсе с помощью Машинное обучение Azure Studio, выполнив следующие действия. 

1. Войдите в [машинное обучение Azure Studio](https://ml.azure.com/).
1. Выберите **связанные службы** в разделе **Управление** левой панели.
1. Выберите **добавить интеграцию**.
1. В форме " **связать рабочую область** " заполните поля 
    Поле| Описание    
    ---|---
    Имя| Укажите имя для связанной службы. Это имя будет использоваться для ссылки на эту конкретную связанную службу.
    имя подписки; | Выберите имя подписки, связанное с рабочей областью машинного обучения. 
    Рабочая область синапсе | Выберите рабочую область синапсе, с которой необходимо установить связь.
1. Нажмите кнопку **Далее** , чтобы открыть форму **Выбор пулов Spark (необязательно)** . В этой форме вы выберите пул синапсе Spark для присоединения к рабочей области.

1. Нажмите кнопку **Далее** , чтобы открыть форму **проверки** и проверить выбранные параметры.
1. Выберите **создать** , чтобы завершить процесс создания связанной службы.

## <a name="next-steps"></a>Дальнейшие действия

* [Присоедините пулы синапсе Spark для подготовки данных с помощью Azure синапсе (Предварительная версия)](how-to-data-prep-synapse-spark-pool.md).
* [Использование Apache Spark в конвейере машинного обучения с помощью Azure синапсе (Предварительная версия)](how-to-use-synapsesparkstep.md)
* [Обучение модели](how-to-set-up-training-targets.md).
