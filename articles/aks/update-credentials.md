---
title: Сброс учетных данных для кластера
titleSuffix: Azure Kubernetes Service
description: Узнайте, как обновить или сбросить учетные данные субъекта-службы или приложения AAD для кластера Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/11/2019
ms.openlocfilehash: 91d40a5a738737098b0941bf3e41e4edd515e6df
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105640478"
---
# <a name="update-or-rotate-the-credentials-for-azure-kubernetes-service-aks"></a>Обновление или смена учетных данных для службы Azure Kubernetes (AKS)

В кластерах AKS, созданных с помощью субъекта-службы, время истечения срока действия составляет один год. Когда будет истекать срок действия, вы можете сбросить учетные данные, чтобы продлить срок действия субъекта-службы на дополнительный период времени. Вы также можете обновить или сменить учетные данные как часть определенной политики безопасности. В этой статье подробно описано, как обновить эти учетные данные для кластера AKS.

Вы также можете [интегрировать кластер AKS с Azure Active Directory][aad-integration]и использовать его в качестве поставщика проверки подлинности для кластера. В этом случае у вас будет еще 2 удостоверения, созданные для кластера, серверное приложение AAD и клиентское приложение AAD. Эти учетные данные также можно сбросить.

Кроме того, для разрешений можно использовать управляемое удостоверение вместо субъекта-службы. Управляемые удостоверения проще в управлении, чем субъекты-службы, и не нуждаются в обновлениях или сменах. Дополнительные сведения см. в статье о том, [как использовать управляемые удостоверения](use-managed-identity.md).

## <a name="before-you-begin"></a>Подготовка

Требуется Azure CLI версии 2.0.65 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="update-or-create-a-new-service-principal-for-your-aks-cluster"></a>Обновление или создание нового субъекта-службы для кластера AKS

Если требуется обновить учетные данные для кластера AKS, можно выбрать один из следующих способов.

* Обновите учетные данные для существующего субъекта-службы.
* Создайте новый субъект-службу и обновите кластер, чтобы использовать эти новые учетные данные. 

> [!WARNING]
> Если вы решили создать *Новый* субъект-службу, обновление большого кластера AKS для использования этих учетных данных может занять много времени.

### <a name="check-the-expiration-date-of-your-service-principal"></a>Проверка даты окончания срока действия субъекта-службы

Чтобы проверить дату окончания срока действия субъекта-службы, используйте команду [AZ AD SP Credential List][az-ad-sp-credential-list] . В следующем примере получается идентификатор субъекта-службы для кластера с именем *myAKSCluster* в группе ресурсов *myResourceGroup* с помощью команды [AZ AKS показывать][az-aks-show] . ИДЕНТИФИКАТОР субъекта-службы задается как переменная с именем *SP_ID* для использования в команде [AZ AD SP Credential List][az-ad-sp-credential-list] .

```azurecli
SP_ID=$(az aks show --resource-group myResourceGroup --name myAKSCluster \
    --query servicePrincipalProfile.clientId -o tsv)
az ad sp credential list --id $SP_ID --query "[].endDate" -o tsv
```

### <a name="reset-the-existing-service-principal-credential"></a>Сброс существующих учетных данных субъекта-службы

Чтобы обновить учетные данные для существующей субъект-службы, получите идентификатор субъект-службы своего кластера с помощью команды [az aks show][az-aks-show]. В следующем примере возвращается идентификатор для кластера *myAKSCluster* в группе ресурсов *myResourceGroup*. ИДЕНТИФИКАТОР субъекта-службы задается как переменная с именем *SP_ID* для использования в дополнительной команде. Эти команды используют синтаксис bash.

> [!WARNING]
> При сбросе учетных данных кластера в кластере AKS, использующем масштабируемые наборы виртуальных машин Azure, выполняется [Обновление образа узла][node-image-upgrade] для обновления узлов новыми учетными данными.

```azurecli-interactive
SP_ID=$(az aks show --resource-group myResourceGroup --name myAKSCluster \
    --query servicePrincipalProfile.clientId -o tsv)
```

С помощью набора переменных, который содержит идентификатор субъект-службы, сбросьте учетные данные с помощью [учетных данных az ad sp][az-ad-sp-credential-reset]. Следующий пример позволяет платформе Azure создать новый безопасный секрет для субъект-службы. Этот новый безопасный секрет также хранится в виде переменной.

```azurecli-interactive
SP_SECRET=$(az ad sp credential reset --name $SP_ID --query password -o tsv)
```

Теперь продолжайте [обновлять кластер AKS, используя новые учетные данные субъекта-службы](#update-aks-cluster-with-new-service-principal-credentials). Этот шаг необходим для отражения изменений субъекта-службы в кластере AKS.

### <a name="create-a-new-service-principal"></a>Создание нового субъекта-службы

Если вы решили обновить существующие учетные данные субъект-службы в предыдущем разделе, пропустите этот шаг. Продолжайте [обновлять кластер AKS, используя новые учетные данные субъекта-службы](#update-aks-cluster-with-new-service-principal-credentials).

Чтобы создать субъект-службу, а затем обновить кластер AKS для использования этих новых учетных данных, используйте команду [az ad sp create-for-rbac][az-ad-sp-create]. В следующем примере параметр `--skip-assignment` запрещает присваивание любых дополнительных назначений по умолчанию.

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

Результат будет похож на следующий пример. Запишите свои `appId` и `password`. Эти значения используются на следующем шаге.

```json
{
  "appId": "7d837646-b1f3-443d-874c-fd83c7c739c5",
  "name": "7d837646-b1f3-443d-874c-fd83c7c739c",
  "password": "a5ce83c9-9186-426d-9183-614597c7f2f7",
  "tenant": "a4342dc8-cd0e-4742-a467-3129c469d0e5"
}
```

Теперь определите переменные для идентификатора субъект-службы и секрета клиента, используя выходные данные своей собственной команды [az ad sp create-for-rbac][az-ad-sp-create], как показано в следующем примере. *SP_ID* — это ваш *appId*, а *SP_SECRET* — это ваш *пароль*.

```console
SP_ID=7d837646-b1f3-443d-874c-fd83c7c739c5
SP_SECRET=a5ce83c9-9186-426d-9183-614597c7f2f7
```

Теперь продолжайте [обновлять кластер AKS, используя новые учетные данные субъекта-службы](#update-aks-cluster-with-new-service-principal-credentials). Этот шаг необходим для отражения изменений субъекта-службы в кластере AKS.

## <a name="update-aks-cluster-with-new-service-principal-credentials"></a>Обновить кластер AKS с новыми учетными данными субъекта-службы

> [!IMPORTANT]
> Для больших кластеров обновление кластера AKS с новым субъектом-службой может занять длительное время.

Независимо от того, решили ли вы создать или обновить учетные данные для существующей субъект-службы, теперь вы обновите кластер AKS новыми учетными данными, используя команду [az aks update-credentials][az-aks-update-credentials]. Используются переменные для *--service-principal* и *--client-secret*.

```azurecli-interactive
az aks update-credentials \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --reset-service-principal \
    --service-principal $SP_ID \
    --client-secret $SP_SECRET
```

Для кластеров малых и средних объемы учетных данных субъекта-службы в AKS занимают несколько секунд.

## <a name="update-aks-cluster-with-new-aad-application-credentials"></a>Обновление кластера AKS с новыми учетными данными приложения AAD

Вы можете создать новые серверные и клиентские приложения AAD, следуя [инструкциям по интеграции AAD][create-aad-app]. Или сбросьте существующие приложения AAD, следуя тому [же методу, что и для сброса субъекта-службы](#reset-the-existing-service-principal-credential). После этого нужно просто обновить учетные данные приложения AAD кластера с помощью той же команды [AZ AKS Update-Credential][az-aks-update-credentials] , но с помощью переменных *--Reset-AAD* .

```azurecli-interactive
az aks update-credentials \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --reset-aad \
    --aad-server-app-id <SERVER APPLICATION ID> \
    --aad-server-app-secret <SERVER APPLICATION SECRET> \
    --aad-client-app-id <CLIENT APPLICATION ID>
```


## <a name="next-steps"></a>Дальнейшие шаги

В этой статье субъект-служба для самого кластера AKS и приложений интеграции AAD обновлен. Дополнительные сведения о том, как управлять удостоверениями для рабочих нагрузок в кластере, см. в статье [Рекомендации по аутентификации и авторизации в службе Azure Kubernetes (AKS)][best-practices-identity].

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-aks-update-credentials]: /cli/azure/aks#az-aks-update-credentials
[best-practices-identity]: operator-best-practices-identity.md
[aad-integration]: ./azure-ad-integration-cli.md
[create-aad-app]: ./azure-ad-integration-cli.md#create-azure-ad-server-component
[az-ad-sp-create]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-ad-sp-credential-list]: /cli/azure/ad/sp/credential#az-ad-sp-credential-list
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset
[node-image-upgrade]: ./node-image-upgrade.md
