---
title: Роли и разрешения реестра
description: Используйте управление доступом на основе ролей Azure (Azure RBAC) и управление удостоверениями и доступом (IAM), чтобы обеспечить детальные разрешения для ресурсов в реестре контейнеров Azure.
ms.topic: article
ms.date: 10/14/2020
ms.openlocfilehash: 097ccf89caf63d2a504d072cf04c2b534a57a031
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92207960"
---
# <a name="azure-container-registry-roles-and-permissions"></a>Роли и разрешения реестра контейнеров Azure

Служба реестра контейнеров Azure поддерживает набор [встроенных ролей Azure](../role-based-access-control/built-in-roles.md) , которые предоставляют различные уровни разрешений для реестра контейнеров Azure. Используйте [Управление доступом на основе ролей Azure (Azure RBAC)](../role-based-access-control/index.yml) , чтобы назначить определенные разрешения пользователям, субъектам-службам или другим удостоверениям, которые должны взаимодействовать с реестром, например для извлечения или отправки образов контейнеров. Можно также определить [пользовательские роли](#custom-roles) с детализированными разрешениями для различных операций в реестре.

| Роль или разрешение       | [Доступ к Resource Manager](#access-resource-manager) | [Создание и удаление реестра](#create-and-delete-registry) | [Отправка образа](#push-image) | [Получение образа](#pull-image) | [Удаление данных образа](#delete-image-data) | [Изменение политик](#change-policies) |   [Подписывание образов](#sign-images)  |
| ---------| --------- | --------- | --------- | --------- | --------- | --------- | --------- |
| Владелец | X | X | X | X | X | X |  |  
| Участник | X | X | X |  X | X | X |  |  
| Читатель | X |  |  | X |  |  |  |
| AcrPush |  |  | X | X | |  |  |  
| AcrPull |  |  |  | X |  |  |  |  
| AcrDelete |  |  |  |  | X |  |  |
| AcrImageSigner |  |  |  |  |  |  | X |

## <a name="assign-roles"></a>Назначение ролей

Дополнительные сведения о добавлении назначения ролей для существующего пользователя, группы, субъекта-службы или управляемого удостоверения см. в разделе [шаги по добавлению назначения ролей](../role-based-access-control/role-assignments-steps.md) для общих действий. Вы можете использовать портал Azure, Azure CLI или другие инструменты Azure.

При создании субъекта-службы вы также настраиваете его доступ и разрешения для ресурсов Azure, таких как реестр контейнеров. Пример сценария с использованием Azure CLI см. в статье [Проверка подлинности реестра контейнеров Azure с помощью субъектов-служб](container-registry-auth-service-principal.md#create-a-service-principal).

## <a name="differentiate-users-and-services"></a>Различие пользователей и служб

Каждый раз при применении разрешений рекомендуется задавать для пользователя или службы минимальный набор разрешений для выполнения задачи. Следующие наборы разрешений представляют наборы возможностей, которые могут использоваться пользователями и автономными службами.

### <a name="cicd-solutions"></a>Решения CI/CD

При автоматизации команд `docker build` от решений CI/CD вам необходимы возможности `docker push`. Для этих сценариев неавтономных служб рекомендуется назначить роль **акрпуш** . Эта роль, в отличие от более широкой роли **Участник**, не позволяет учетной записи выполнять другие операции с реестром и обращаться к Azure Resource Manager.

### <a name="container-host-nodes"></a>Узлы контейнеров

Точно так же узлам, в которых запущены контейнеры, необходима роль **AcrPull**, но им не требуются возможности роли **Читатель**.

### <a name="visual-studio-code-docker-extension"></a>Расширение Docker для Visual Studio Code

Для таких средств, как [расширение Docker для Visual Studio Code](https://code.visualstudio.com/docs/azure/docker), необходим доступ к дополнительному поставщику ресурсов для получения доступных реестров контейнеров Azure. В этом случае предоставьте своим пользователям доступ к роли **Читатель** или **Участник**. Эти роли позволяют использовать `docker pull`, `docker push`, `az acr list`, `az acr build` и другие возможности. 

## <a name="access-resource-manager"></a>Доступ к Resource Manager

Доступ к Azure Resource Manager является обязательным для управления порталом Azure и реестром с помощью [Azure CLI](/cli/azure/). Это набор разрешений потребуется, например, чтобы получить список реестров с помощью команды `az acr list`. 

## <a name="create-and-delete-registry"></a>Создание и удаление реестра

Возможность создавать и удалять реестры контейнеров Azure.

## <a name="push-image"></a>Отправка образа

Возможность `docker push` образа или отправки другого [поддерживаемого артефакта](container-registry-image-formats.md), например диаграммы Helm, в реестр. Требуется [проверка подлинности](container-registry-authentication.md) в реестре с помощью авторизованного удостоверения. 

## <a name="pull-image"></a>Получение образа

Возможность `docker pull` образа, не находящегося на карантине, или получения другого [поддерживаемого артефакта](container-registry-image-formats.md), например диаграммы Helm, из реестра. Требуется [проверка подлинности](container-registry-authentication.md) в реестре с помощью авторизованного удостоверения.

## <a name="delete-image-data"></a>Удаление данных образа

Возможность [удалять образы контейнеров](container-registry-delete.md)или удалять другие [Поддерживаемые артефакты](container-registry-image-formats.md) , например диаграммы Helm, из реестра.

## <a name="change-policies"></a>Изменение политик

Возможность настройки политик в реестре. Политики включают очистку образов, включение карантина и подписывание образов.

## <a name="sign-images"></a>Подписывание образов

Возможность подписывания образов обычно назначается автоматизированному процессу, который использует субъект-службу. Это разрешение обычно объединяется с разрешением на [отправку образа](#push-image), чтобы разрешить отправку доверенного образа в реестр. Дополнительные сведения см. [в статье о доверии содержимого в реестре контейнеров Azure](container-registry-content-trust.md).

## <a name="custom-roles"></a>Пользовательские роли

Как и в случае с другими ресурсами Azure, вы можете создавать [пользовательские роли](../role-based-access-control/custom-roles.md) с детализированными разрешениями для реестра контейнеров Azure. Затем назначьте пользовательские роли пользователям, субъектам-службам или другим удостоверениям, которые должны взаимодействовать с реестром. 

Чтобы определить, какие разрешения следует применить к пользовательской роли, ознакомьтесь со списком [действий](../role-based-access-control/resource-provider-operations.md#microsoftcontainerregistry)Microsoft. ContainerRegistry, просмотрите список разрешенных действий [встроенных ролей записи контроля](../role-based-access-control/built-in-roles.md)доступа или выполните следующую команду:

```azurecli
az provider operation show --namespace Microsoft.ContainerRegistry
```

Чтобы определить пользовательскую роль, см. раздел [шаги по созданию настраиваемой роли](../role-based-access-control/custom-roles.md#steps-to-create-a-custom-role).

> [!IMPORTANT]
> В пользовательской роли реестр контейнеров Azure в настоящее время не поддерживает подстановочные знаки, такие как `Microsoft.ContainerRegistry/*` или `Microsoft.ContainerRegistry/registries/*` , которые предоставляют доступ ко всем соответствующим действиям. Укажите в роли все необходимые действия по отдельности.

### <a name="example-custom-role-to-import-images"></a>Пример. настраиваемая роль для импорта изображений

Например, следующий код JSON определяет минимальные действия для пользовательской роли, позволяющей [импортировать изображения](container-registry-import-images.md) в реестр.

```json
{
   "assignableScopes": [
     "/subscriptions/<optional, but you can limit the visibility to one or more subscriptions>"
   ],
   "description": "Can import images to registry",
   "Name": "AcrImport",
   "permissions": [
     {
       "actions": [
         "Microsoft.ContainerRegistry/registries/push/write",
         "Microsoft.ContainerRegistry/registries/pull/read",
         "Microsoft.ContainerRegistry/registries/read",
         "Microsoft.ContainerRegistry/registries/importImage/action"
       ],
       "dataActions": [],
       "notActions": [],
       "notDataActions": []
     }
   ],
   "roleType": "CustomRole"
 }
```

Чтобы создать или обновить настраиваемую роль с помощью описания JSON, используйте [Azure CLI](../role-based-access-control/custom-roles-cli.md), [Azure Resource Manager шаблон](../role-based-access-control/custom-roles-template.md), [Azure PowerShell](../role-based-access-control/custom-roles-powershell.md)или другие инструменты Azure. Добавление или удаление назначений ролей для пользовательской роли аналогично управлению назначениями ролей для встроенных ролей Azure.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте больше о назначении ролей Azure удостоверению Azure с помощью [портал Azure](../role-based-access-control/role-assignments-portal.md), [Azure CLI](../role-based-access-control/role-assignments-cli.md)или других средств Azure.

* Ознакомьтесь с дополнительными сведениями о [вариантах проверки подлинности](container-registry-authentication.md) для Реестра контейнеров Azure.

* Сведения о включении [разрешений в области репозитория](container-registry-repository-scoped-permissions.md) (Предварительная версия) в реестре контейнеров.
