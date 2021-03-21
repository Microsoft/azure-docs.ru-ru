---
title: Использование реестра контейнеров Azure с Azure Red Hat OpenShift
description: Узнайте, как извлечь и запустить контейнер из реестра контейнеров Azure в кластере Azure Red Hat OpenShift.
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 01/10/2021
ms.openlocfilehash: 651b73db084e8090f59faeffa9991c2ac468ca08
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100634419"
---
# <a name="use-azure-container-registry-with-azure-red-hat-openshift-aro"></a>Использование реестра контейнеров Azure с Azure Red Hat OpenShift (АТО)

Реестр контейнеров Azure (запись контроля доступа) — это управляемая служба реестра контейнеров, которую можно использовать для хранения частных образов контейнеров DOCKER с корпоративными возможностями, такими как Георепликация. Чтобы получить доступ к записи контроля доступа из кластера АТО, этот кластер может пройти проверку подлинности в записи контроля доступа, сохранив учетные данные входа DOCKER в Kubernetes секрете.  Аналогичным образом кластер АТО может использовать Имажепуллсекрет в спецификации Pod для проверки подлинности в реестре при извлечении образа. Из этой статьи вы узнаете, как настроить реестр контейнеров Azure с помощью кластера Azure Red Hat OpenShift для хранения и извлечения частных образов контейнеров DOCKER.

## <a name="prerequisites"></a>Предварительные условия

В этом учебнике предполагается наличие существующего реестра контейнеров Azure. В противном случае используйте портал Azure или [Azure CLI инструкции](../container-registry/container-registry-get-started-azure-cli.md) для создания реестра контейнеров.

В этой статье также предполагается, что у вас есть существующий кластер Azure Red Hat OpenShift и `oc` установлен интерфейс командной строки. Если это не так, следуйте инструкциям в [учебнике Создание кластера АТО](tutorial-create-cluster.md).

## <a name="get-a-pull-secret"></a>Получение секрета для получения по запросу

Чтобы получить доступ к реестру из кластера АТО, вам потребуется опрашивающий секрет из записи контроля доступа.

Чтобы получить учетные данные опрашивающего секрета, можно использовать либо портал Azure, либо Azure CLI.

Если вы используете портал Azure, перейдите к своему экземпляру записи контроля доступа и выберите **ключи Access**.  `docker-username`Укажите имя реестра контейнеров, используйте пароль или password2 для `docker-password` .

![Ключи доступа](./media/acr-access-keys.png)

Вместо этого можно использовать Azure CLI для получения этих учетных данных:
```azurecli
az acr credential show -n <your registry name>
```

## <a name="create-the-kubernetes-secret"></a>Создание секрета Kubernetes

Теперь мы будем использовать эти учетные данные для создания секрета Kubernetes. Выполните следующую команду с учетными данными записи контроля доступа:

```
oc create secret docker-registry \
    --docker-server=<your registry name>.azurecr.io \
    --docker-username=<your registry name> \
    --docker-password=******** \
    --docker-email=unused \
    acr-secret
```

>[!NOTE]
>Этот секрет будет храниться в текущем проекте OpenShift (пространство имен Kubernetes) и будет ссылаться только на модули, созданные в этом проекте.  Дополнительные сведения о создании общесистемного секрета для получения сведений о кластере см. в этом [документе](https://docs.openshift.com/container-platform/4.4/openshift_images/managing_images/using-image-pull-secrets.html) .

## <a name="create-a-pod-using-a-private-registry-image"></a>Создание Pod с помощью частного образа реестра

Теперь, когда мы подключились к кластеру АТО к записи контроля доступа, давайте создадим образ из записи контроля доступа, чтобы создать модуль.

Начните с Подспек и укажите секрет, созданный в качестве Имажепуллсекрет:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: hello-world
    image: <your registry name>.azurecr.io/hello-world:v1
  imagePullSecrets:
  - name: acr-secret
```

Чтобы убедиться, что модуль установлен **и работает,** выполните эту команду и дождитесь ее состояния:

```bash
$ oc get pods --watch
NAME         READY   STATUS    RESTARTS   AGE
hello-world  1/1     Running   0          30s
```

## <a name="next-steps"></a>Дальнейшие действия

* [Реестр контейнеров Azure](../container-registry/container-registry-concepts.md)
* [Краткое руководство. Создание закрытого реестра контейнеров с помощью Azure CLI](../container-registry/container-registry-get-started-azure-cli.md)
