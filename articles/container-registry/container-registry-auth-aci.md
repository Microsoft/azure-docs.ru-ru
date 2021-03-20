---
title: Доступ из экземпляров контейнеров
description: Узнайте, как обеспечить доступ к образам в закрытом реестре контейнеров из службы "Экземпляры контейнеров Azure" с помощью субъекта-службы Azure Active Directory.
ms.topic: article
ms.date: 04/23/2018
ms.openlocfilehash: b1bc8119c495dea99c6bdc4923db198d041a1e9e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74456517"
---
# <a name="authenticate-with-azure-container-registry-from-azure-container-instances"></a>Аутентификация с помощью реестра контейнеров Azure из службы "Экземпляры контейнеров Azure"

Вы можете обеспечить доступ к закрытым реестрам контейнеров в реестре контейнеров Azure с помощью субъекта-службы Azure Active Directory (Azure AD).

Из этой статьи вы узнаете, как создать и настроить субъект-службу Azure AD с разрешениями на *извлечение данных* в реестре. Затем вы запустите контейнер в службе "Экземпляры контейнеров Azure" (ACI), которая извлекает образ контейнера из закрытого реестра, с помощью субъекта-службы для аутентификации.

## <a name="when-to-use-a-service-principal"></a>Когда следует использовать субъект-службу

Субъект-службу следует использовать для аутентификации из ACI в **сценариях автоматического входа**, например в приложениях или службах, которые создают экземпляры контейнеров автоматически или другим способом без участия пользователя.

Например, для автоматического скрипта, который выполняется ночью и создает [экземпляр контейнера на основе задач](../container-instances/container-instances-restart-policy.md) для обработки некоторых данных, можно использовать субъект-службу с разрешениями только на извлечение для аутентификации в реестре. Затем можно сменить учетные данные субъекта-службы или полностью отменить его доступ, что никак не отразится на других службах и приложениях.

Субъекты-службы также следует использовать, если учетная запись [администратора](container-registry-authentication.md#admin-account) реестра отключена.

[!INCLUDE [container-registry-service-principal](../../includes/container-registry-service-principal.md)]

## <a name="authenticate-using-the-service-principal"></a>Аутентификация с помощью субъекта-службы

Чтобы запустить контейнер в службе "Экземпляры контейнеров Azure", используя субъект-службу, укажите идентификатор субъекта-службы для параметра `--registry-username` и пароль для параметра `--registry-password`.

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerregistry.azurecr.io/myimage:v1 \
    --registry-login-server mycontainerregistry.azurecr.io \
    --registry-username <service-principal-ID> \
    --registry-password <service-principal-password>
```

## <a name="sample-scripts"></a>Примеры сценариев

На GitHub можно найти предыдущие примеры сценариев для Azure CLI, а также версии для Azure PowerShell:

* [Azure CLI][acr-scripts-cli]
* [Azure PowerShell][acr-scripts-psh]

## <a name="next-steps"></a>Дальнейшие действия

В следующих статьях приводятся дополнительные сведения о работе с субъектами-службами и реестром контейнеров Azure.

* [Аутентификация в реестре контейнеров Azure с помощью субъектов-служб](container-registry-auth-service-principal.md)
* [Аутентификация с помощью Реестра контейнеров Azure из службы Azure Kubernetes](../aks/cluster-container-registry-integration.md)

<!-- IMAGES -->

<!-- LINKS - External -->
[acr-scripts-cli]: https://github.com/Azure/azure-docs-cli-python-samples/tree/master/container-registry
[acr-scripts-psh]: https://github.com/Azure/azure-docs-powershell-samples/tree/master/container-registry

<!-- LINKS - Internal -->
