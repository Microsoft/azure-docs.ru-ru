---
title: включить файл
description: включить файл
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 12/14/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: f2d2b655e80f5b9694fb1948b136aac918312ca9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102244969"
---
## <a name="create-a-service-principal"></a>Создание субъекта-службы

Чтобы создать субъект-службу с доступом к реестру контейнеров, выполните следующий скрипт в [Azure Cloud Shell](../articles/cloud-shell/overview.md) или локальную установку [Azure CLI](/cli/azure/install-azure-cli). Скрипт отформатирован для оболочки Bash.

Перед выполнением сценария обновите переменную `ACR_NAME` с именем реестра контейнеров. Значение `SERVICE_PRINCIPAL_NAME` должно быть уникальным в клиенте Azure Active Directory. Если вы получаете ошибку "`'http://acr-service-principal' already exists.`", укажите другое имя для субъекта-службы.

Можно изменять значение `--role` в команде [az ad sp create-for-rbac][az-ad-sp-create-for-rbac], если необходимо предоставить другие разрешения. Полный список ролей см. в разделе [Роли и разрешения реестра контейнеров Azure](https://github.com/Azure/acr/blob/master/docs/roles-and-permissions.md).

После запуска скрипта запишите **идентификатор** и **пароль** субъекта-службы. Сохранив эти учетные данные, вы можете настроить приложения и службы для аутентификации в качестве субъектов-служб в реестре контейнеров.

<!-- https://github.com/Azure-Samples/azure-cli-samples/blob/master/container-registry/service-principal-create/service-principal-create.sh -->
[!code-azurecli-interactive[acr-sp-create](~/cli_scripts/container-registry/service-principal-create/service-principal-create.sh)]

### <a name="use-an-existing-service-principal"></a>Использование существующего субъекта-службы

Чтобы предоставить субъекту-службе доступ к реестру, нужно назначить новую роль субъекту-службе. Как и при создании субъекта-службы, вы, помимо прочего, можете предоставить разрешения на извлечение данных, отправку и извлечение данных, а также разрешения владельца.

В следующем скрипте используется команда [az role assignment create][az-role-assignment-create], чтобы предоставить разрешения на *извлечение* субъекту-службе, указанному в переменной `SERVICE_PRINCIPAL_ID`. Измените значение `--role`, если нужно предоставить другой уровень доступа.


<!-- https://github.com/Azure-Samples/azure-cli-samples/blob/master/container-registry/service-principal-assign-role/service-principal-assign-role.sh -->
[!code-azurecli-interactive[acr-sp-role-assign](~/cli_scripts/container-registry/service-principal-assign-role/service-principal-assign-role.sh)]

<!-- LINKS - Internal -->
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
