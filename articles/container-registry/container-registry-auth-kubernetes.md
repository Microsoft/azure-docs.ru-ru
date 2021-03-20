---
title: Проверка подлинности из кластера Kubernetes
description: Узнайте, как предоставить кластеру Kubernetes доступ к образам в реестре контейнеров Azure путем создания опрашивающего секрета с помощью субъекта-службы.
ms.topic: article
author: karolz-ms
ms.author: karolz
ms.reviewer: danlep
ms.date: 05/28/2020
ms.openlocfilehash: fbf5dfd68b823b600b11cad3643e5d4004b85ff5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84309821"
---
# <a name="pull-images-from-an-azure-container-registry-to-a-kubernetes-cluster"></a>Извлечение образов из реестра контейнеров Azure в кластер Kubernetes

Вы можете использовать реестр контейнеров Azure в качестве источника образов контейнеров с любым кластером Kubernetes, включая локальные кластеры Kubernetes, такие как [minikube](https://minikube.sigs.k8s.io/) и [Kind](https://kind.sigs.k8s.io/). В этой статье показано, как создать Kubernetes опрашивающий секрет на основе субъекта-службы Azure Active Directory. Затем используйте секрет для извлечения образов из реестра контейнеров Azure в развертывании Kubernetes.

> [!TIP]
> Если вы используете управляемую [службу Kubernetes Azure](../aks/intro-kubernetes.md), вы также можете [интегрировать кластер](../aks/cluster-container-registry-integration.md?toc=/azure/container-registry/toc.json&bc=/azure/container-registry/breadcrumb/toc.json) с целевым реестром контейнеров Azure для извлечения образа. 

В этой статье предполагается, что вы уже создали частный реестр контейнеров Azure. Кроме того, необходимо запустить кластер Kubernetes и получить доступ к нему через `kubectl` программу командной строки.

[!INCLUDE [container-registry-service-principal](../../includes/container-registry-service-principal.md)]

Если пароль субъекта-службы не сохранен или не запоминать, его можно сбросить с помощью команды [AZ AD SP Credential rereset][az-ad-sp-credential-reset] :

```azurecli
az ad sp credential reset  --name http://<service-principal-name> --query password --output tsv
```

Эта команда возвращает новый, допустимый пароль для субъекта-службы.

## <a name="create-an-image-pull-secret"></a>Создание секрета для извлечения образа

Kubernetes использует *секрет опрашивающего образа* для хранения сведений, необходимых для проверки подлинности в реестре. Чтобы создать секрет для получения по запросу для реестра контейнеров Azure, укажите идентификатор субъекта-службы, пароль и адрес реестра. 

Создайте секрет для извлечения образа с помощью следующей `kubectl` команды:

```console
kubectl create secret docker-registry <secret-name> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
```
где:

| Значение | Описание |
| :--- | :--- |
| `secret-name` | Имя секрета для получения образа, например запись *контроля доступа — секретный код* |
| `namespace` | Пространство имен Kubernetes для помещения секрета в <br/> Требуется только в том случае, если вы хотите поместить секрет в пространство имен, отличное от пространства имен по умолчанию |
| `container-registry-name` | Имя реестра контейнеров Azure, например *myregistry*<br/><br/>`--docker-server`— Это полное имя сервера входа в реестр.  |
| `service-principal-ID` | ИДЕНТИФИКАТОР субъекта-службы, который будет использоваться Kubernetes для доступа к реестру. |
| `service-principal-password` | Пароль субъекта-службы |

## <a name="use-the-image-pull-secret"></a>Использование секрета для извлечения образа

После создания секрета на извлечение образа его можно использовать для создания модулей Kubernetes и развертываний. Укажите имя секрета в `imagePullSecrets` файле развертывания. Пример:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-awesome-app-pod
  namespace: awesomeapps
spec:
  containers:
    - name: main-app-container
      image: myregistry.azurecr.io/my-awesome-app:v1
      imagePullPolicy: IfNotPresent
  imagePullSecrets:
    - name: acr-secret
```

В предыдущем примере `my-awesome-app:v1` — это имя образа, который необходимо извлечь из реестра контейнеров Azure, а  `acr-secret` — это имя опрашивающего секрета, созданного для доступа к реестру. При развертывании Pod Kubernetes автоматически извлекает образ из реестра, если он еще не существует в кластере.


## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о работе с субъектами-службами и реестром контейнеров Azure см. [в статье Проверка подлинности реестра контейнеров Azure с помощью субъектов-служб](container-registry-auth-service-principal.md) .
* Дополнительные сведения о секретных параметрах образа см. в [документации по Kubernetes](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod) .


<!-- IMAGES -->

<!-- LINKS - External -->
[acr-scripts-cli]: https://github.com/Azure/azure-docs-cli-python-samples/tree/master/container-registry
[acr-scripts-psh]: https://github.com/Azure/azure-docs-powershell-samples/tree/master/container-registry

<!-- LINKS - Internal -->
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset