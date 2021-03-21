---
title: Использование политики Azure для защиты кластера
description: Используйте политику Azure для защиты кластера Azure Kubernetes Service (AKS).
ms.service: container-service
ms.topic: how-to
ms.date: 02/17/2021
ms.custom: template-how-to
ms.openlocfilehash: 46e92e6842204cd323992a2561e71302bb9cc722
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102193505"
---
# <a name="secure-your-cluster-with-azure-policy"></a>Защита кластера с помощью политики Azure

Чтобы повысить безопасность кластера Azure Kubernetes Service (AKS), можно применять и применять встроенные политики безопасности в кластере с помощью политики Azure. [Политика Azure][azure-policy] помогает применять организационные стандарты и оценивать соответствие при масштабировании. После установки [надстройки политики Azure для AKS][kubernetes-policy-reference]можно применить индивидуальные определения политик или группы определений политик, которые называются инициативами (иногда называемые policysets) в кластере. Полный список определений AKS политики и инициатив см. [в статье встроенные определения политики Azure для AKS][aks-policies] .

В этой статье показано, как применить определения политик к кластеру и проверить, применяются ли эти назначения.

## <a name="prerequisites"></a>Предварительные условия

- Существующий кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].
- Надстройка политики Azure для AKS, установленная в кластере AKS. Выполните следующие [действия, чтобы установить надстройку политики Azure][azure-policy-addon].

## <a name="assign-a-built-in-policy-definition-or-initiative"></a>Назначение встроенного определения политики или инициативы

Чтобы применить определение политики или инициативу, используйте портал Azure.

1. Перейдите к службе "политика Azure" в портал Azure.
1. Выберите **Определения** на левой панели страницы "Политика Azure".
1. В разделе **категории** выберите `Kubernetes` .
1. Выберите определение политики или инициативу, которую необходимо применить. В этом примере выберите `Kubernetes cluster pod security baseline standards for Linux-based workloads` инициативу.
1. Выберите **Назначить**.
1. Задайте **область действия** группы ресурсов кластера AKS с включенной надстройкой политики Azure.
1. Выберите страницу **Параметры** и обновите **результат действия** с `audit` на, `deny` чтобы заблокировать новые развертывания, нарушающие базовую инициативу. Можно также добавить дополнительные пространства имен, чтобы исключить из оценки. В этом примере следует использовать значения по умолчанию.
1. Выберите **проверить и создать** , а затем **создать** , чтобы отправить назначение политики.

## <a name="validate-a-azure-policy-is-running"></a>Проверка выполнения политики Azure

Убедитесь, что назначения политики применяются к кластеру, выполнив следующую команду:

```azurecli-interactive
kubectl get constrainttemplates
```

> [!NOTE]
> Для синхронизации назначений политики в каждом кластере может потребоваться до [20 минут][azure-policy-assign-policy] .

Результат должен выглядеть следующим образом:

```console
$ kubectl get constrainttemplate
NAME                                     AGE
k8sazureallowedcapabilities              23m
k8sazureallowedusersgroups               23m
k8sazureblockhostnamespace               23m
k8sazurecontainerallowedimages           23m
k8sazurecontainerallowedports            23m
k8sazurecontainerlimits                  23m
k8sazurecontainernoprivilege             23m
k8sazurecontainernoprivilegeescalation   23m
k8sazureenforceapparmor                  23m
k8sazurehostfilesystem                   23m
k8sazurehostnetworkingports              23m
k8sazurereadonlyrootfilesystem           23m
k8sazureserviceallowedports              23m
```

### <a name="validate-rejection-of-a-privileged-pod"></a>Проверка отклонения привилегированного Pod

Давайте сначала проверим, что происходит при планировании Pod с помощью контекста безопасности `privileged: true` . Этот контекст безопасности повышает привилегии Pod. Инициатива запрещает привилегированные модули, поэтому запрос отклоняется, что приведет к отклонению развертывания.

Создайте файл с именем `nginx-privileged.yaml` и вставьте следующий манифест YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      securityContext:
        privileged: true
```

Создайте модуль Pod с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl apply -f nginx-privileged.yaml
```

Как ожидалось, модуль не может быть запланирован, как показано в следующем примере выходных данных:

```console
$ kubectl apply -f privileged.yaml

Error from server ([denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}): error when creating "privileged.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}
```

Модуль не достигает этапа планирования, поэтому нет ресурсов для удаления перед тем, как вы перейдете.

### <a name="test-creation-of-an-unprivileged-pod"></a>Тестирование создания непривилегированного Pod

В предыдущем примере образ контейнера автоматически пытался использовать root для привязки NGINX к порту 80. Этот запрос был отклонен инициативой политики, поэтому не удается запустить модуль. Давайте попробуем запустить этот же модуль NGINX без привилегированного доступа.

Создайте файл с именем `nginx-unprivileged.yaml` и вставьте следующий манифест YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
```

Создайте модуль Pod с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl apply -f nginx-unprivileged.yaml
```

Модуль POD успешно запланирован. При проверке состояния Pod с помощью команды [kubectl Get][kubectl-get] Pod *выполняется*:

```console
$ kubectl get pods

NAME                 READY   STATUS    RESTARTS   AGE
nginx-unprivileged   1/1     Running   0          18s
```

В этом примере показано, как базовая инициатива влияет только на развертывания, которые нарушают политики в коллекции. Разрешенные развертывания продолжают функционировать.

Удалите непривилегированный модуль NGINX с помощью команды [kubectl Delete][kubectl-delete] и укажите имя манифеста YAML:

```console
kubectl delete -f nginx-unprivileged.yaml
```

## <a name="disable-a-policy-or-initiative"></a>Отключение политики или инициативы

Чтобы удалить базовую инициативу, выполните следующие действия.

1. Перейдите в область политики на портал Azure.
1. На левой панели выберите **назначения** .
1. Нажмите кнопку **...** рядом с `Kubernetes cluster pod security baseline standards for Linux-based workloads` инициативой.
1. Выберите **удалить назначение**.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как работает политика Azure:

- [Что такое служба "Политика Azure"][azure-policy]
- [Инициативы и политики Azure для AKS][aks-policies]
- Удалите [надстройку политики Azure][azure-policy-addon-remove].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs

<!-- LINKS - internal -->
[aks-policies]: policy-reference.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[azure-policy]: ../governance/policy/overview.md
[azure-policy-addon]: ../governance/policy/concepts/policy-for-kubernetes.md#install-azure-policy-add-on-for-aks
[azure-policy-addon-remove]: ../governance/policy/concepts/policy-for-kubernetes.md#remove-the-add-on-from-aks
[azure-policy-assign-policy]: ../governance/policy/concepts/policy-for-kubernetes.md#assign-a-built-in-policy-definition
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[kubernetes-policy-reference]: ../governance/policy/concepts/policy-for-kubernetes.md
