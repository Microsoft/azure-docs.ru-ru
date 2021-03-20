---
title: Развертывание образа контейнера из реестра контейнеров Azure
description: Узнайте, как развертывать контейнеры в службе "экземпляры контейнеров Azure" путем извлечения образов контейнеров из реестра контейнеров Azure.
services: container-instances
ms.topic: article
ms.date: 07/02/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: cca1001f0f84f4e4fc87df233f872fc1efdb3267
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92736736"
---
# <a name="deploy-to-azure-container-instances-from-azure-container-registry"></a>Развертывание в службе "Экземпляры контейнеров Azure" из реестра контейнеров Azure

[Реестр контейнеров Azure](../container-registry/container-registry-intro.md) — это управляемая служба реестра контейнеров на базе Azure, используемая для хранения частных образов контейнеров Docker. В этой статье описывается, как извлечь образы контейнеров, хранящиеся в реестре контейнеров Azure, при развертывании в службе "экземпляры контейнеров Azure". Для настройки доступа к реестру рекомендуется создать субъект-службу Azure Active Directory и пароль, а также сохранить учетные данные входа в хранилище ключей Azure.

## <a name="prerequisites"></a>Предварительные условия

**Реестр контейнеров Azure**. для выполнения действий, описанных в этой статье, требуется реестр контейнеров Azure, а также по крайней мере один образ контейнера в реестре. Если вам нужен реестр, см. сведения о его создании в статье [Краткое руководство. Создание реестра контейнеров с использованием Azure CLI](../container-registry/container-registry-get-started-azure-cli.md).

**Azure CLI**. Примеры командной строки в этой статье используют [Azure CLI](/cli/azure/) и отформатированы для оболочки Bash. Вы можете [установить Azure CLI](/cli/azure/install-azure-cli) локально или использовать [Azure Cloud Shell][cloud-shell-bash].

## <a name="limitations"></a>Ограничения

* Вы не можете пройти проверку подлинности в реестре контейнеров Azure, чтобы извлечь образы во время развертывания группы контейнеров с помощью [управляемого удостоверения](container-instances-managed-identity.md) , настроенного в той же группе контейнеров.
* В настоящее время невозможно извлечь образы из [реестра контейнеров Azure](../container-registry/container-registry-vnet.md) , развернутые в виртуальную сеть Azure.

## <a name="configure-registry-authentication"></a>Настройка проверки подлинности в реестре

В рабочем сценарии, где предоставляется доступ к службам и приложениям без монитора, рекомендуется настроить доступ к реестру с помощью [субъекта-службы](../container-registry/container-registry-auth-service-principal.md). Субъект-служба позволяет предоставлять [Управление доступом на основе ролей Azure (Azure RBAC)](../container-registry/container-registry-roles.md) для образов контейнеров. Например, вы можете настроить субъект-службу с доступом только на извлечение к реестру.

Реестр контейнеров Azure предоставляет дополнительные [Параметры проверки подлинности](../container-registry/container-registry-authentication.md).

В следующем разделе вы создадите субъект-службу и сохраните его учетные данные в хранилище ключей Azure, которое также создается в этом разделе.

### <a name="create-key-vault"></a>Создание хранилища ключей

Если у вас еще нет хранилища в [Azure Key Vault](../key-vault/general/overview.md), создайте его с помощью Azure CLI, используя следующие команды.

Обновите переменную `RES_GROUP` именем существующей группы ресурсов, в которой будет создано хранилище ключей, а `ACR_NAME` — именем вашего реестра контейнеров. Для краткости в командах в этой статье предполагается, что реестр, хранилище ключей и экземпляры контейнеров созданы в одной группе ресурсов.

 Укажите имя нового хранилища ключей в `AKV_NAME`. Имя хранилища должно быть уникальным в Azure и содержать от 3 до 24 буквенно-цифровых символов. Имя должно начинаться с буквы, заканчиваться буквой или цифрой и не может содержать последовательные дефисы.

```azurecli
RES_GROUP=myresourcegroup # Resource Group name
ACR_NAME=myregistry       # Azure Container Registry registry name
AKV_NAME=mykeyvault       # Azure Key Vault vault name

az keyvault create -g $RES_GROUP -n $AKV_NAME
```

### <a name="create-service-principal-and-store-credentials"></a>Создание учетной записи субъекта-службы и сохранение учетных данных

Теперь создайте субъект-службу и сохраните его учетные данные в хранилище ключей.

Следующая команда использует [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] для создания субъекта-службы и [az keyvault secret set][az-keyvault-secret-set] для сохранения **пароля** субъекта-службы в хранилище.

```azurecli
# Create service principal, store its password in vault (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name http://$ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)
```

Аргумент `--role` в предыдущей команде настраивает субъект-службу с ролью *acrpull*, которая предоставляет доступ только для извлечения к реестру. Чтобы предоставить доступ для отправки и извлечения данных, измените аргумент `--role` на *acrpush*.

Затем сохраните *AppID* субъекта-службы в хранилище, которое представляет собой **имя пользователя** , передаваемое в реестр контейнеров Azure для проверки подлинности.

```azurecli
# Store service principal ID in vault (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

Вы создали хранилище ключей Azure и сохранили в нем два секрета:

* `$ACR_NAME-pull-usr`: идентификатор субъекта-службы для использования в качестве **имени пользователя** реестра контейнеров.
* `$ACR_NAME-pull-pwd`: пароль субъекта-службы для использования в качестве **пароля** реестра контейнеров.

Теперь вы можете ссылаться на эти секреты по имени, когда вы или ваши приложения и службы извлекают образы из реестра.

## <a name="deploy-container-with-azure-cli"></a>Развертывание контейнера с помощью Azure CLI

Теперь, когда учетные данные субъекта-службы хранятся в секретах Аzure Key Vault, ваши приложения и службы могут использовать их для доступа к вашему частному реестру.

Сначала получите имя сервера входа в реестр с помощью команды [az acr show][az-acr-show]. Имя сервера для входа представлено в нижнем регистре и подобно `myregistry.azurecr.io`.

```azurecli
ACR_LOGIN_SERVER=$(az acr show --name $ACR_NAME --resource-group $RES_GROUP --query "loginServer" --output tsv)
```

Выполните следующую команду [az container create][az-container-create], чтобы развернуть экземпляр контейнера. Команда использует учетные данные субъекта-службы, хранящиеся в Azure Key Vault, для проверки подлинности в вашем реестре контейнеров и предполагает, что вы ранее отправили образ [aci-helloworld](container-instances-quickstart.md) в ваш реестр. Обновите значение `--image`, если вы хотите использовать другой образ из реестра.

```azurecli
az container create \
    --name aci-demo \
    --resource-group $RES_GROUP \
    --image $ACR_LOGIN_SERVER/aci-helloworld:v1 \
    --registry-login-server $ACR_LOGIN_SERVER \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME -n $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label aci-demo-$RANDOM \
    --query ipAddress.fqdn
```

Значение `--dns-name-label` должно быть уникальным в пределах Azure, поэтому предыдущая команда добавляет случайное число к метке DNS-имени контейнера. В выходных данных команды содержится полное доменное имя (FQDN) контейнера, например:

```output
"aci-demo-25007.eastus.azurecontainer.io"
```

Когда контейнер будет запущен, вы можете перейти к его FQDN в браузере, чтобы убедиться, что приложение работает успешно.

## <a name="deploy-with-azure-resource-manager-template"></a>Развертывание с помощью шаблона Azure Resource Manager

Свойства реестра контейнеров Azure можно указать в шаблоне Azure Resource Manager, включив `imageRegistryCredentials` свойство в определение группы контейнеров. Например, можно напрямую указать учетные данные реестра:

```JSON
[...]
"imageRegistryCredentials": [
  {
    "server": "imageRegistryLoginServer",
    "username": "imageRegistryUsername",
    "password": "imageRegistryPassword"
  }
]
[...]
```

Полные параметры группы контейнеров см. в [справочнике по шаблону диспетчер ресурсов](/azure/templates/Microsoft.ContainerInstance/2019-12-01/containerGroups).    

Дополнительные сведения об использовании ссылки на секреты Azure Key Vault в шаблоне Resource Manager см. в статье [Использование Azure Key Vault для передачи защищенного значения параметра во время развертывания](../azure-resource-manager/templates/key-vault-parameter.md).

## <a name="deploy-with-azure-portal"></a>Развертывание с помощью портала Azure

Если образы контейнеров хранятся в реестре контейнеров Azure, можно легко создать контейнер в службе "Экземпляры контейнеров Azure" с помощью портала Azure. Чтобы развернуть экземпляр контейнера из реестра контейнеров с помощью портала, необходимо включить [учетную запись администратора реестра](../container-registry/container-registry-authentication.md#admin-account). Учетная запись администратора предоставляет доступ к реестру одному пользователю. Она предназначена, главным образом, для тестирования. 

1. На портале Azure перейдите в реестр контейнеров.

1. Чтобы подтвердить, что учетная запись администратора включена, выберите **Ключи доступа**, а затем в разделе **Пользователь-администратор** выберите **Включить**.

1. Щелкните **Репозитории**, а затем выберите репозиторий, из которого необходимо выполнить развертывание. Щелкните правой кнопкой мыши тег для образа контейнера, который нужно развернуть, и выберите пункт **Запустить экземпляр**.

    ![Пункт "Запустить экземпляр" в реестре контейнеров Azure на портале Azure][acr-runinstance-contextmenu]

1. Введите имя контейнера и группы ресурсов. При необходимости можно изменить значения по умолчанию.

    ![Меню "Создание" для службы "Экземпляры контейнеров Azure"][acr-create-deeplink]

1. После завершения развертывания можно перейти к группе контейнеров в области уведомлений, чтобы найти ее IP-адрес и другие свойства.

    ![Представление сведений для группы контейнеров службы "Экземпляры контейнеров Azure"][aci-detailsview]

## <a name="next-steps"></a>Дальнейшие действия

Более подробные сведения об аутентификации в реестре контейнеров Azure см. в статье [Аутентификация с помощью частного реестра контейнеров Docker](../container-registry/container-registry-authentication.md).

<!-- IMAGES -->
[acr-create-deeplink]: ./media/container-instances-using-azure-container-registry/acr-create-deeplink.png
[aci-detailsview]: ./media/container-instances-using-azure-container-registry/aci-detailsview.png
[acr-runinstance-contextmenu]: ./media/container-instances-using-azure-container-registry/acr-runinstance-contextmenu.png

<!-- LINKS - External -->
[cloud-shell-bash]: https://shell.azure.com/bash
[cloud-shell-try-it]: https://shell.azure.com/powershell

<!-- LINKS - Internal -->
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-container-create]: /cli/azure/container#az-container-create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
