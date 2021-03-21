---
title: Смена сертификатов в службе Kubernetes Azure (AKS)
description: Узнайте, как поворачивать сертификаты в кластере Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 11/15/2019
ms.openlocfilehash: fa26762c54ad54835b174b8d814a2e77cb38b885
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102619041"
---
# <a name="rotate-certificates-in-azure-kubernetes-service-aks"></a>Смена сертификатов в службе Kubernetes Azure (AKS)

Служба Azure Kubernetes Service (AKS) использует сертификаты для проверки подлинности с множеством своих компонентов. Периодически может потребоваться повернуть эти сертификаты для обеспечения безопасности или политики. Например, у вас может быть политика для смены всех сертификатов каждые 90 дней.

В этой статье показано, как поворачивать сертификаты в кластере AKS.

## <a name="before-you-begin"></a>Перед началом

Для работы с этой статьей требуется Azure CLI версии 2.0.77 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="aks-certificates-certificate-authorities-and-service-accounts"></a>AKS сертификаты, центры сертификации и учетные записи служб

AKS создает и использует следующие сертификаты, центры сертификации и учетные записи служб:

* Сервер API AKS создает центр сертификации (ЦС), называемый ЦС кластера.
* Сервер API имеет ЦС кластера, который подписывает сертификаты для односторонней связи от сервера API к кубелетс.
* Каждый kubelet также создает запрос подписи сертификата (CSR), который подписывается ЦС кластера для обмена данными между kubelet и сервером API.
* Агрегатор API использует ЦС кластера для выдаче сертификатов для взаимодействия с другими API. Агрегатор API также может иметь собственный ЦС для выдачи этих сертификатов, но в настоящее время использует ЦС кластера.
* Каждый узел использует маркер учетной записи службы (SA), подписанный центром сертификации кластера.
* У `kubectl` клиента есть сертификат для взаимодействия с кластером AKS.

> [!NOTE]
> В кластерах AKS, созданных до 2019 марта, срок действия сертификатов истекает через два года. Любой кластер, созданный после марта 2019 или любого кластера с повернутыми сертификатами, содержит сертификаты ЦС кластера, срок действия которых истекает через 30 лет. Срок действия всех остальных сертификатов истекает через два года. Чтобы проверить, когда был создан кластер, используйте `kubectl get nodes` для просмотра *возраста* пулов узлов.
> 
> Кроме того, можно проверить дату окончания срока действия сертификата кластера. Например, следующая команда Bash отображает сведения о сертификате для кластера *myAKSCluster* .
> ```console
> kubectl config view --raw -o jsonpath="{.clusters[?(@.name == 'myAKSCluster')].cluster.certificate-authority-data}" | base64 -d | openssl x509 -text | grep -A2 Validity
> ```

## <a name="rotate-your-cluster-certificates"></a>Смена сертификатов кластера

> [!WARNING]
> Смена сертификатов с помощью `az aks rotate-certs` может привести к простою в течение 30 минут бездействия для кластера AKS.

Для входа в кластер AKS используйте команду [AZ AKS Get-Credential][az-aks-get-credentials] . Эта команда также скачивает и настраивает `kubectl` сертификат клиента на локальном компьютере.

```azurecli
az aks get-credentials -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME
```

Используйте `az aks rotate-certs` для смены всех сертификатов, центров сертификации и SAS в кластере.

```azurecli
az aks rotate-certs -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME
```

> [!IMPORTANT]
> Для завершения может потребоваться до 30 минут `az aks rotate-certs` . Если команда не выполняется до завершения, используйте `az aks show` для проверки состояния кластера на *вращение сертификата*. Если кластер находится в состоянии сбоя, повторно запустите `az aks rotate-certs` его, чтобы снова повернуть сертификаты.

Убедитесь, что старые сертификаты больше не являются допустимыми, выполнив `kubectl` команду. Так как вы не обновили сертификаты, используемые `kubectl` , вы увидите ошибку.  Пример:

```console
$ kubectl get no
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "ca")
```

Обновите сертификат, используемый службами `kubectl` , выполнив `az aks get-credentials` .

```azurecli
az aks get-credentials -g $RESOURCE_GROUP_NAME -n $CLUSTER_NAME --overwrite-existing
```

Убедитесь, что сертификаты обновлены, выполнив `kubectl` команду, которая теперь будет выполнена успешно. Пример:

```console
kubectl get no
```

> [!NOTE]
> Если у вас есть службы, работающие поверх AKS, например [Azure dev Spaces][dev-spaces], может потребоваться также [обновить сертификаты, связанные с этими службами][dev-spaces-rotate] .

## <a name="next-steps"></a>Следующие шаги

В этой статье показано, как автоматически поворачивать сертификаты, CAs и SAs кластера. Рекомендации [по обеспечению безопасности и обновления кластера в службе Azure Kubernetes (AKS)][aks-best-practices-security-upgrades] см. в статье рекомендации по обеспечению безопасности AKS.


[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[aks-best-practices-security-upgrades]: operator-best-practices-cluster-security.md
[dev-spaces]: ../dev-spaces/index.yml
[dev-spaces-rotate]: ../dev-spaces/troubleshooting.md#error-using-dev-spaces-after-rotating-aks-certificates
