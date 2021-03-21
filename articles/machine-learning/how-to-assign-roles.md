---
title: Управление ролями в рабочей области
titleSuffix: Azure Machine Learning
description: Узнайте, как получить доступ к рабочей области Машинное обучение Azure с помощью управления доступом на основе ролей Azure (Azure RBAC).
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: Blackmist
ms.author: nigup
author: nishankgu
ms.date: 01/20/2020
ms.custom: how-to, seodec18, devx-track-azurecli, contperf-fy21q2
ms.openlocfilehash: 8420aecbc160fa6df2640d2ba0ae8a8b77702b67
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98624546"
---
# <a name="manage-access-to-an-azure-machine-learning-workspace"></a>Управление доступом к рабочей области Машинного обучения Azure

Из этой статьи вы узнаете, как управлять доступом (авторизацией) в Машинное обучение Azure рабочей области. [Управление доступом на основе ролей Azure (Azure RBAC)](../role-based-access-control/overview.md) используется для управления доступом к ресурсам Azure, таким как возможность создавать новые ресурсы или использовать существующие. Пользователям в Azure Active Directory (Azure AD) назначаются определенные роли, которые предоставляют доступ к ресурсам. Azure предоставляет встроенные роли и возможность создания пользовательских ролей.

> [!TIP]
> Хотя эта статья посвящена Машинное обучение Azure, отдельные службы, на которых основывается Служба машинного обучения Azure, предоставляют собственные параметры RBAC. Например, с помощью сведений в этой статье можно настроить, кто может отправлять запросы оценки в модель, развернутую в качестве веб-службы в службе Azure Kubernetes. Но служба Azure Kubernetes предоставляет собственный набор ролей Azure. Сведения о специфических для службы данных RBAC, которые могут быть полезны для Машинное обучение Azure, см. по следующим ссылкам:
>
> * [Управление доступом к ресурсам кластера Azure Kubernetes](../aks/azure-ad-rbac.md)
> * [Использование Azure RBAC для авторизации Kubernetes](../aks/manage-azure-rbac.md)
> * [Использование Azure RBAC для доступа к данным BLOB-объектов](../storage/common/storage-auth-aad-rbac-portal.md)

> [!WARNING]
> Применение некоторых ролей может ограничить функциональность пользовательского интерфейса в Машинное обучение Azure Studio для других пользователей. Например, если роль пользователя не может создать вычислительный экземпляр, параметр создания вычислительного экземпляра будет недоступен в студии. Это ожидаемое поведение, которое предотвращает попытки пользователя выполнить операции, возвращающие ошибку отказа в доступе.

## <a name="default-roles"></a>Роли по умолчанию

Рабочая область Машинного обучения Azure — это ресурс Azure. Как и в случае других ресурсов Azure, при создании рабочая область Машинного обучения Azure снабжается тремя ролями по умолчанию. Вы можете добавить пользователей в рабочую область и назначить их одной из этих встроенных ролей.

| Роль | Уровень доступа |
| --- | --- |
| **Читатель** | Действия только для чтения в рабочей области. Читатели могут перечислять и просматривать ресурсы, включая учетные данные [хранилища](how-to-access-data.md) данных, в рабочей области. Читатели не могут создавать или обновлять эти ресурсы. |
| **Участник** | Просмотр, создание, изменение или удаление (если применимо) ресурсов в рабочей области. Например, участники могут создать эксперимент, создать или подключить вычислительный кластер, выполнить запуск и развернуть веб-службу. |
| **Владелец** | Полный доступ к рабочей области, включая возможность просмотра, создания, изменения или удаления (если применимо) ресурсов в рабочей области. Кроме того, можно изменить назначения ролей. |
| **Настраиваемая роль** | Позволяет настраивать доступ к конкретным операциям управления или плоскости данных в рабочей области. Например, отправка запуска, создание вычислений, развертывание модели или регистрация набора данных. |

> [!IMPORTANT]
> Доступ к роли можно ограничить несколькими уровнями в Azure. Например, пользователь с доступом владельца к рабочей области может не иметь права владельца на группу ресурсов, содержащую рабочую область. Дополнительные сведения см. в статье [работа Azure RBAC](../role-based-access-control/overview.md#how-azure-rbac-works).

В настоящее время отсутствуют дополнительные встроенные роли, характерные для Машинное обучение Azure. Дополнительные сведения о встроенных ролях см. в статье [встроенные роли Azure](../role-based-access-control/built-in-roles.md).

## <a name="manage-workspace-access"></a>Управление доступом к рабочей области

Если вы являетесь владельцем рабочей области, вы можете добавлять и удалять роли для рабочей области. Можно также назначать роли пользователям. Воспользуйтесь следующими ссылками, чтобы узнать, как управлять доступом:
- [Пользовательский интерфейс портала Azure](../role-based-access-control/role-assignments-portal.md)
- [PowerShell](../role-based-access-control/role-assignments-powershell.md)
- [Azure CLI](../role-based-access-control/role-assignments-cli.md)
- [REST API](../role-based-access-control/role-assignments-rest.md)
- [Шаблоны диспетчера ресурсов Azure](../role-based-access-control/role-assignments-template.md)

Если вы установили [машинное обучение Azure CLI](reference-azure-machine-learning-cli.md), можно использовать команды интерфейса командной строки для назначения ролей пользователям:

```azurecli-interactive 
az ml workspace share -w <workspace_name> -g <resource_group_name> --role <role_name> --user <user_corp_email_address>
```

`user`Поле является адресом электронной почты существующего пользователя в экземпляре Azure Active Directory, где находится родительская подписка рабочей области. Ниже приведен пример использования этой команды.

```azurecli-interactive 
az ml workspace share -w my_workspace -g my_resource_group --role Contributor --user jdoe@contoson.com
```

> [!NOTE]
> Команда AZ ML Workspace Share не работает для Федеративной учетной записи Azure Active Directory B2B. Вместо команды используйте портал пользовательского интерфейса Azure.

## <a name="create-custom-role"></a>Создание настраиваемой роли

Если встроенных ролей недостаточно, можно создать пользовательские роли. Пользовательские роли могут иметь разрешения на чтение, запись, удаление и вычисление ресурсов в этой рабочей области. Роль можно сделать доступной на определенном уровне рабочей области, определенном уровне группы ресурсов или определенном уровне подписки.

> [!NOTE]
> Для создания пользовательских ролей в этом ресурсе необходимо быть владельцем ресурса на этом уровне.

Чтобы создать пользовательскую роль, сначала создайте JSON-файл определения роли, указывающий разрешение и область действия для роли. В следующем примере определяется пользовательская роль с именем "Data анализу Custom" с областью действия на определенном уровне рабочей области:

`data_scientist_custom_role.json` :
```json
{
    "Name": "Data Scientist Custom",
    "IsCustom": true,
    "Description": "Can run experiment but can't create or delete compute.",
    "Actions": ["*"],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/*/delete",
        "Microsoft.MachineLearningServices/workspaces/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/delete", 
        "Microsoft.Authorization/*/write"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>/resourceGroups/<resource_group_name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace_name>"
    ]
}
```

> [!TIP]
> Вы можете изменить `AssignableScopes` это поле, чтобы задать область этой пользовательской роли на уровне подписки, на уровне группы ресурсов или на определенном уровне рабочей области.
> Приведенная выше пользовательская роль является примером, см. Некоторые Рекомендуемые [пользовательские роли для службы машинное обучение Azure](#customroles).

Эта пользовательская роль может выполнять все действия в рабочей области, за исключением следующих действий.

- Он не может создавать или обновлять ресурсы вычислений.
- Он не может удалить ресурс вычислений.
- Он не может добавлять, удалять или изменять назначения ролей.
- Она не может удалить рабочую область.

Чтобы развернуть эту настраиваемую роль, используйте следующую команду Azure CLI.

```azurecli-interactive 
az role definition create --role-definition data_scientist_role.json
```

После развертывания эта роль станет доступной в указанной рабочей области. Теперь можно добавить и назначить эту роль в портал Azure. Вы также можете назначить эту роль пользователю с помощью `az ml workspace share` команды CLI:

```azurecli-interactive
az ml workspace share -w my_workspace -g my_resource_group --role "Data Scientist" --user jdoe@contoson.com
```

Дополнительные сведения о пользовательских ролях см. в статье [пользовательские роли Azure](../role-based-access-control/custom-roles.md). 

### <a name="azure-machine-learning-operations"></a>Машинное обучение Azure операции

Дополнительные сведения об операциях (действиях и не действиях), которые можно использовать с пользовательскими ролями, см. в разделе [операции с поставщиками ресурсов](../role-based-access-control/resource-provider-operations.md#microsoftmachinelearningservices). Для вывода списка операций можно также использовать следующую команду Azure CLI:

```azurecli-interactive
az provider operation show –n Microsoft.MachineLearningServices
```

## <a name="list-custom-roles"></a>Вывод списка настраиваемых ролей

В Azure CLI выполните следующую команду:

```azurecli-interactive
az role definition list --subscription <sub-id> --custom-role-only true
```

Чтобы просмотреть определение роли для определенной пользовательской роли, используйте следующую команду Azure CLI. Значение `<role-name>` должно быть в том же формате, что и команда выше:

```azurecli-interactive
az role definition list -n <role-name> --subscription <sub-id>
```

## <a name="update-a-custom-role"></a>Обновление пользовательской роли

В Azure CLI выполните следующую команду:

```azurecli-interactive
az role definition update --role-definition update_def.json --subscription <sub-id>
```

Необходимо иметь разрешения на всю область определения новой роли. Например, если эта новая роль имеет область действия в трех подписках, необходимо иметь разрешения для всех трех подписок. 

> [!NOTE]
> Обновление ролей может занять от 15 до часа, чтобы применить их во всех назначениях ролей в этой области.

## <a name="use-azure-resource-manager-templates-for-repeatability"></a>Использование шаблонов Azure Resource Manager для повторяемости

Если предполагается, что потребуется повторное создание сложных назначений ролей, шаблон Azure Resource Manager может быть большой справкой. [Шаблон 201-Machine-Learning-зависимостей-Role-назначение](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-dependencies-role-assignment) показывает, как можно указать назначения ролей в исходном коде для повторного использования. 

## <a name="common-scenarios"></a>Распространенные сценарии

В следующей таблице приведена сводка действий Машинное обучение Azure и разрешений, необходимых для их выполнения в наименьшей области. Например, если действие может быть выполнено с областью действия рабочей области (столбец 4), то все более высокие области с этим разрешением также будут работать автоматически:

> [!IMPORTANT]
> Все пути в этой таблице, начинающиеся с, `/` являются **относительными путями** к `Microsoft.MachineLearningServices/` :

| Действие | Область действия уровня подписки | Область действия уровня группы ресурсов | Область уровня рабочей области |
| ----- | ----- | ----- | ----- |
| Создать рабочую область | Не требуется | Владелец или участник | Н/д (владелец или наследует роль более высокого уровня после создания) |
| Запросить квоту на уровне подписки Амлкомпуте или задать квоту на уровне рабочей области | Владелец, участник или пользовательская роль </br>позволяя `/locations/updateQuotas/action`</br> в области действия подписки | Не санкционировано | Не санкционировано |
| Создать новый кластер вычислений | Не требуется | Не требуется | Владелец, участник или пользовательская роль, которая позволяет: `/workspaces/computes/write` |
| Создание нового вычислительного экземпляра | Не требуется | Не требуется | Владелец, участник или пользовательская роль, которая позволяет: `/workspaces/computes/write` |
| Отправка любого типа запуска | Не требуется | Не требуется | Владелец, участник или пользовательская роль, которая позволяет: `"/workspaces/*/read", "/workspaces/environments/write", "/workspaces/experiments/runs/write", "/workspaces/metadata/artifacts/write", "/workspaces/metadata/snapshots/write", "/workspaces/environments/build/action", "/workspaces/experiments/runs/submit/action", "/workspaces/environments/readSecrets/action"` |
| Публикация конвейеров и конечных точек | Не требуется | Не требуется | Владелец, участник или пользовательская роль, которая позволяет: `"/workspaces/endpoints/pipelines/*", "/workspaces/pipelinedrafts/*", "/workspaces/modules/*"` |
| Развертывание зарегистрированной модели в ресурсе AKS/ACI | Не требуется | Не требуется | Владелец, участник или пользовательская роль, которая позволяет: `"/workspaces/services/aks/write", "/workspaces/services/aci/write"` |
| Оценка по развернутой конечной точке AKS | Не требуется | Не требуется | Владелец, участник или настраиваемая роль: `"/workspaces/services/aks/score/action", "/workspaces/services/aks/listkeys/action"` (если не используется Azure Active Directory auth) или `"/workspaces/read"` (при использовании токена проверки подлинности) |
| Доступ к хранилищу с помощью интерактивных записных книжек | Не требуется | Не требуется | Владелец, участник или пользовательская роль, которая позволяет: `"/workspaces/computes/read", "/workspaces/notebooks/samples/read", "/workspaces/notebooks/storage/*", "/workspaces/listKeys/action"` |
| Создать новую настраиваемую роль | Владелец, участник или пользовательская роль, допускающая `Microsoft.Authorization/roleDefinitions/write` | Не требуется | Владелец, участник или пользовательская роль, которая позволяет: `/workspaces/computes/write` |

> [!TIP]
> Если при первом создании рабочей области возникает ошибка, убедитесь, что она разрешена `Microsoft.MachineLearningServices/register/action` . Это действие позволяет зарегистрировать поставщик Машинное обучение Azure ресурсов в подписке Azure.

### <a name="user-assigned-managed-identity-with-azure-ml-compute-cluster"></a>Назначенное пользователем управляемое удостоверение с помощью вычислений в кластере машинного обучения Azure

Чтобы назначить назначенное пользователем удостоверение кластеру Машинное обучение Azure COMPUTE, необходимы разрешения на запись для создания роли вычислений и [управляемого оператора идентификации](../role-based-access-control/built-in-roles.md#managed-identity-operator). Дополнительные сведения об Azure RBAC с управляемыми удостоверениями см. [в статье управление назначенными пользователями удостоверениями](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) .

### <a name="mlflow-operations"></a>Операции Млфлов

Для выполнения операций Млфлов с рабочей областью Машинное обучение Azure используйте следующие области настраиваемой роли:

| Операция Млфлов | Область |
| --- | --- |
| Перечислите все эксперименты в хранилище отслеживания рабочих областей, получите эксперимент по идентификатору, получите эксперимент по имени | `Microsoft.MachineLearningServices/workspaces/experiments/read` |
| Создание эксперимента с именем, задание тега в эксперименте, восстановление эксперимента, помеченного для удаления| `Microsoft.MachineLearningServices/workspaces/experiments/write` | 
| Удаление эксперимента | `Microsoft.MachineLearningServices/workspaces/experiments/delete` |
| Получение запуска и связанных данных и метаданных, получение списка всех значений для указанной метрики для данного запуска, список артефактов для запуска | `Microsoft.MachineLearningServices/workspaces/experiments/runs/read` |
| Создание нового запуска в эксперименте, удаление запусков, восстановление удаленных запусков, метрики журнала в текущем запуске, установка тегов для запуска, удаление тегов для выполнения, параметры журнала (пара "ключ-значение"), используемые для запуска, запись пакета метрик, параметров и тегов для запуска, обновление состояния выполнения. | `Microsoft.MachineLearningServices/workspaces/experiments/runs/write` |
| Получение зарегистрированной модели по имени, получение списка всех зарегистрированных моделей в реестре, поиск зарегистрированных моделей, последние модели версий для каждого этапа запросов, получение версии зарегистрированной модели, версии модели поиска, получение URI, где хранятся артефакты версии модели, Поиск запусков по идентификаторам экспериментов | `Microsoft.MachineLearningServices/workspaces/models/read` |
| Создайте новую зарегистрированную модель, обновите имя или описание зарегистрированной модели, переименуйте существующую зарегистрированную модель, создайте новую версию модели, обновите описание версии модели, перенесите зарегистрированную модель на один из этапов. | `Microsoft.MachineLearningServices/workspaces/models/write` |
| Удаление зарегистрированной модели вместе со всеми ее версиями, удаление конкретных версий зарегистрированной модели | `Microsoft.MachineLearningServices/workspaces/models/delete` |

<a id="customroles"></a>

## <a name="example-custom-roles"></a>Примеры пользовательских ролей

### <a name="data-scientist"></a>Специалист по обработке и анализу данных

Позволяет анализу данных выполнять все операции внутри рабочей области, **за исключением**:

* Создание вычислений
* Развертывание моделей в рабочем кластере AKS
* Развертывание конечной точки конвейера в рабочей среде

`data_scientist_custom_role.json` :
```json
{
    "Name": "Data Scientist Custom",
    "IsCustom": true,
    "Description": "Can run experiment but can't create or delete compute or deploy production endpoints.",
    "Actions": [
        "Microsoft.MachineLearningServices/workspaces/*/read",
        "Microsoft.MachineLearningServices/workspaces/*/action",
        "Microsoft.MachineLearningServices/workspaces/*/delete",
        "Microsoft.MachineLearningServices/workspaces/*/write"
    ],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/delete",
        "Microsoft.MachineLearningServices/workspaces/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/delete", 
        "Microsoft.Authorization/*",
        "Microsoft.MachineLearningServices/workspaces/computes/listKeys/action",
        "Microsoft.MachineLearningServices/workspaces/listKeys/action",
        "Microsoft.MachineLearningServices/workspaces/services/aks/write",
        "Microsoft.MachineLearningServices/workspaces/services/aks/delete",
        "Microsoft.MachineLearningServices/workspaces/endpoints/pipelines/write"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>"
    ]
}
```

### <a name="data-scientist-restricted"></a>Анализу данных ограничена

Более ограниченное определение роли без подстановочных знаков в разрешенных действиях. Она может выполнять все операции внутри рабочей области, **за исключением** следующих:

* Создание вычислений
* Развертывание моделей в рабочем кластере AKS
* Развертывание конечной точки конвейера в рабочей среде

`data_scientist_restricted_custom_role.json` :
```json
{
    "Name": "Data Scientist Restricted Custom",
    "IsCustom": true,
    "Description": "Can run experiment but can't create or delete compute or deploy production endpoints",
    "Actions": [
        "Microsoft.MachineLearningServices/workspaces/*/read",
        "Microsoft.MachineLearningServices/workspaces/computes/start/action",
        "Microsoft.MachineLearningServices/workspaces/computes/stop/action",
        "Microsoft.MachineLearningServices/workspaces/computes/restart/action",
        "Microsoft.MachineLearningServices/workspaces/computes/applicationaccess/action",
        "Microsoft.MachineLearningServices/workspaces/notebooks/storage/read",
        "Microsoft.MachineLearningServices/workspaces/notebooks/storage/write",
        "Microsoft.MachineLearningServices/workspaces/notebooks/storage/delete",
        "Microsoft.MachineLearningServices/workspaces/notebooks/samples/read",
        "Microsoft.MachineLearningServices/workspaces/experiments/runs/write",
        "Microsoft.MachineLearningServices/workspaces/experiments/write",
        "Microsoft.MachineLearningServices/workspaces/experiments/runs/submit/action",
        "Microsoft.MachineLearningServices/workspaces/pipelinedrafts/write",
        "Microsoft.MachineLearningServices/workspaces/metadata/snapshots/write",
        "Microsoft.MachineLearningServices/workspaces/metadata/artifacts/write",
        "Microsoft.MachineLearningServices/workspaces/environments/write",
        "Microsoft.MachineLearningServices/workspaces/models/write",
        "Microsoft.MachineLearningServices/workspaces/modules/write",
        "Microsoft.MachineLearningServices/workspaces/datasets/registered/write", 
        "Microsoft.MachineLearningServices/workspaces/datasets/registered/delete",
        "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/write",
        "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/delete",
        "Microsoft.MachineLearningServices/workspaces/computes/listNodes/action",
        "Microsoft.MachineLearningServices/workspaces/environments/build/action"
    ],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/computes/write",
        "Microsoft.MachineLearningServices/workspaces/write",
        "Microsoft.MachineLearningServices/workspaces/computes/delete",
        "Microsoft.MachineLearningServices/workspaces/delete",
        "Microsoft.MachineLearningServices/workspaces/computes/listKeys/action",
        "Microsoft.MachineLearningServices/workspaces/listKeys/action",
        "Microsoft.Authorization/*",
        "Microsoft.MachineLearningServices/workspaces/datasets/registered/profile/read",
        "Microsoft.MachineLearningServices/workspaces/datasets/registered/preview/read",
        "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/profile/read",
        "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/preview/read",
        "Microsoft.MachineLearningServices/workspaces/datasets/registered/schema/read",    
        "Microsoft.MachineLearningServices/workspaces/datasets/unregistered/schema/read",
        "Microsoft.MachineLearningServices/workspaces/datastores/write",
        "Microsoft.MachineLearningServices/workspaces/datastores/delete"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>"
    ]
}
```
     
### <a name="mlflow-data-scientist"></a>Млфлов данных анализу

Позволяет анализу данных выполнять все поддерживаемые операции Млфлов AzureML, **за исключением** следующих:

* Создание вычислений
* Развертывание моделей в рабочем кластере AKS
* Развертывание конечной точки конвейера в рабочей среде

`mlflow_data_scientist_custom_role.json` :
```json
{
    "Name": "MLFlow Data Scientist Custom",
    "IsCustom": true,
    "Description": "Can perform azureml mlflow integrated functionalities that includes mlflow tracking, projects, model registry",
    "Actions": [
        "Microsoft.MachineLearningServices/workspaces/experiments/read",
        "Microsoft.MachineLearningServices/workspaces/experiments/write",
        "Microsoft.MachineLearningServices/workspaces/experiments/delete",
        "Microsoft.MachineLearningServices/workspaces/experiments/runs/read",
        "Microsoft.MachineLearningServices/workspaces/experiments/runs/write",
        "Microsoft.MachineLearningServices/workspaces/models/read",
        "Microsoft.MachineLearningServices/workspaces/models/write",
        "Microsoft.MachineLearningServices/workspaces/models/delete"
    ],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/delete",
        "Microsoft.MachineLearningServices/workspaces/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/delete", 
        "Microsoft.Authorization/*",
        "Microsoft.MachineLearningServices/workspaces/computes/listKeys/action",
        "Microsoft.MachineLearningServices/workspaces/listKeys/action",
        "Microsoft.MachineLearningServices/workspaces/services/aks/write",
        "Microsoft.MachineLearningServices/workspaces/services/aks/delete",
        "Microsoft.MachineLearningServices/workspaces/endpoints/pipelines/write"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>"
    ]
}
```   

### <a name="mlops"></a>MLOps

Позволяет назначить роль субъекту-службе и использовать ее для автоматизации конвейеров Млопс. Например, для отправки запусков в уже опубликованном конвейере:

`mlops_custom_role.json` :
```json
{
    "Name": "MLOps Custom",
    "IsCustom": true,
    "Description": "Can run pipelines against a published pipeline endpoint",
    "Actions": [
        "Microsoft.MachineLearningServices/workspaces/read",
        "Microsoft.MachineLearningServices/workspaces/endpoints/pipelines/read",
        "Microsoft.MachineLearningServices/workspaces/metadata/artifacts/read",
        "Microsoft.MachineLearningServices/workspaces/metadata/snapshots/read",
        "Microsoft.MachineLearningServices/workspaces/environments/read",    
        "Microsoft.MachineLearningServices/workspaces/metadata/secrets/read",
        "Microsoft.MachineLearningServices/workspaces/modules/read",
        "Microsoft.MachineLearningServices/workspaces/experiments/runs/read",
        "Microsoft.MachineLearningServices/workspaces/datasets/registered/read",
        "Microsoft.MachineLearningServices/workspaces/datastores/read",
        "Microsoft.MachineLearningServices/workspaces/environments/write",
        "Microsoft.MachineLearningServices/workspaces/experiments/runs/write",
        "Microsoft.MachineLearningServices/workspaces/metadata/artifacts/write",
        "Microsoft.MachineLearningServices/workspaces/metadata/snapshots/write",
        "Microsoft.MachineLearningServices/workspaces/environments/build/action",
        "Microsoft.MachineLearningServices/workspaces/experiments/runs/submit/action"
    ],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/computes/write",
        "Microsoft.MachineLearningServices/workspaces/write",
        "Microsoft.MachineLearningServices/workspaces/computes/delete",
        "Microsoft.MachineLearningServices/workspaces/delete",
        "Microsoft.MachineLearningServices/workspaces/computes/listKeys/action",
        "Microsoft.MachineLearningServices/workspaces/listKeys/action",
        "Microsoft.Authorization/*"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>"
    ]
}
```

### <a name="workspace-admin"></a>Администратор рабочей области

Позволяет выполнять все операции в области действия рабочей области, **за исключением** следующих:

* Создание новой рабочей области
* Назначение квот уровня подписки или рабочей области

Администратор рабочей области также не может создать новую роль. Он может назначать существующие встроенные или пользовательские роли в области действия рабочей области.

`workspace_admin_custom_role.json` :
```json
{
    "Name": "Workspace Admin Custom",
    "IsCustom": true,
    "Description": "Can perform all operations except quota management and upgrades",
    "Actions": [
        "Microsoft.MachineLearningServices/workspaces/*/read",
        "Microsoft.MachineLearningServices/workspaces/*/action",
        "Microsoft.MachineLearningServices/workspaces/*/write",
        "Microsoft.MachineLearningServices/workspaces/*/delete",
        "Microsoft.Authorization/roleAssignments/*"
    ],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/write"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>"
    ]
}
```

<a name="labeler"></a>
### <a name="data-labeler"></a>Метка данных

Позволяет определить роль, ограниченную только подписями данных:

`labeler_custom_role.json` :
```json
{
    "Name": "Labeler Custom",
    "IsCustom": true,
    "Description": "Can label data for Labeling",
    "Actions": [
        "Microsoft.MachineLearningServices/workspaces/read",
        "Microsoft.MachineLearningServices/workspaces/labeling/projects/read",
        "Microsoft.MachineLearningServices/workspaces/labeling/labels/write"
    ],
    "NotActions": [
        "Microsoft.MachineLearningServices/workspaces/labeling/projects/summary/read"
    ],
    "AssignableScopes": [
        "/subscriptions/<subscription_id>"
    ]
}
```

## <a name="troubleshooting"></a>Устранение неполадок

Ниже приведены некоторые моменты, которые следует учитывать при использовании управления доступом на основе ролей Azure (Azure RBAC).

- При создании ресурса в Azure, например рабочей области, вы не являетесь владельцем ресурса напрямую. Роль наследуется от самой высокой роли области, которой вы имеете право в этой подписке. Например, если вы являетесь администратором сети и имеете разрешения на создание рабочей области Машинное обучение, вам будет назначена роль администратора сети для этой рабочей области, а не роль владельца.

- Для выполнения операций с квотами в рабочей области требуются разрешения уровня подписки. Это означает, что Настройка квоты на уровне подписки или квоты на уровне рабочей области для управляемых ресурсов вычислений может произойти только при наличии разрешений на запись в области подписки.

- При наличии двух назначений ролей для одного и того же Azure Active Directory пользователя с конфликтующими разделами действий и отсутствия операции, перечисленные в неизменности одной роли, могут не действовать, если они также перечислены как действия в другой роли. Дополнительные сведения о том, как Azure анализирует назначения ролей, см. в статье [Определение того, как Azure RBAC определяет, имеет ли пользователь доступ к ресурсу](../role-based-access-control/overview.md#how-azure-rbac-determines-if-a-user-has-access-to-a-resource) .

- Чтобы развернуть ресурсы для вычислений в виртуальной сети, необходимо явно иметь разрешения для следующих действий.
    - `Microsoft.Network/virtualNetworks/join/action` в ресурсе виртуальной сети.
    - `Microsoft.Network/virtualNetworks/subnet/join/action` в ресурсе подсети.
    
    Дополнительные сведения об Azure RBAC с сетью см. в разделе [встроенные сетевые роли](../role-based-access-control/built-in-roles.md#networking).

- Иногда для того, чтобы новые назначения ролей вступили в силу с кэшированными разрешениями в стеке, может потребоваться до 1 часа.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об обеспечении безопасности на уровне предприятия](concept-enterprise-security.md)
- [Общие сведения о изоляции и конфиденциальности виртуальной сети](how-to-network-security-overview.md)
- [Руководство. Обучение моделей](tutorial-train-models-with-aml.md)
- [Операции с поставщиками ресурсов](../role-based-access-control/resource-provider-operations.md#microsoftmachinelearningservices)
