---
title: Использование нескольких пулов узлов в Службе контейнеров Azure
description: Сведения о создании нескольких пулов узлов для кластера в службе Kubernetes Azure (AKS) и управление такими пулами узлов
services: container-service
ms.topic: article
ms.date: 02/11/2021
ms.openlocfilehash: bb10e2023187c74a9e8b9a2e4c72115841e89a84
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106552603"
---
# <a name="create-and-manage-multiple-node-pools-for-a-cluster-in-azure-kubernetes-service-aks"></a>Создание нескольких пулов узлов для кластера в службе Kubernetes Azure (AKS) и управление такими пулами узлов

В службе Azure Kubernetes Service (AKS) узлы одной и той же конфигурации группируются в *пулы узлов*. Эти узлы содержат базовые виртуальные машины, на которых выполняются ваши приложения. При создании кластера AKS вы указываете начальное количество и размер узлов, которые составляют [пул системных узлов][use-system-pool]. Для поддержки приложений, имеющих различные требования к вычислению или хранению, можно создать дополнительные *пулы узлов пользователей*. Пулы системных узлов служат основной цели размещения критически важных системных модулей, таких как CoreDNS и tunnelfront. Пулы узлов пользователей служат основной цели размещения модулей Pod приложения. Однако модули приложений можно запланировать в пулах системных узлов, если в кластере AKS требуется только один пул. Пулы узлов пользователей — это место размещения модулей Pod для конкретного приложения. Например, можно воспользоваться этими дополнительными пулами узлов пользователей, чтобы предоставить графические процессоры для приложений с большим объемом вычислений или доступ к высокопроизводительному хранилищу SSD.

> [!NOTE]
> Эта функция обеспечивает более высокий контроль над созданием нескольких пулов узлов и управлением ими. В результате для создания, обновления и удаления требуются отдельные команды. Ранее кластерные операции посредством `az aks create` или `az aks update` использовали API managedCluster и были единственным вариантом изменения уровня управления и пула с одним узлом. Эта функция предоставляет отдельный набор операций для пулов агентов через API agentPool и требует использования набора команд `az aks nodepool` для выполнения операций в пуле отдельных узлов.

В этой статье показано, как создать несколько пулов узлов и управлять ими в кластере AKS.

## <a name="before-you-begin"></a>Подготовка к работе

Необходимо установить и настроить Azure CLI версии 2.2.0 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="limitations"></a>Ограничения

При создании и администрировании кластеров AKS, поддерживающих несколько пулов узлов, действуют следующие ограничения:

* См. раздел [Квоты, ограничения размера виртуальной машины и доступность региона в службе Kubernetes Azure (AKS)][quotas-skus-regions].
* Пулы системных узлов можно удалить при условии, что в кластере AKS имеется другой пул системных узлов.
* Системные пулы должны содержать по крайней мере один узел, а пулы узлов пользователей могут содержать ноль или более узлов.
* Кластер AKS должен использовать подсистему балансировки нагрузки уровня "Стандартный" для использования нескольких пулов узлов. Эта функция не поддерживается для подсистем балансировки нагрузки уровня "Базовый".
* Кластер AKS должен использовать масштабируемые наборы виртуальных машин для узлов.
* Имя пула узлов может содержать только буквы в нижнем регистре и должно начинаться с буквы в нижнем регистре. Для пулов узлов Linux длина должна составлять от 1 до 12 символов. Для пулов узлов Windows длина должна составлять от 1 до 6 символов.
* Все пулы узлов должны находиться в одной виртуальной сети.
* При создании нескольких пулов узлов во время создания кластера все версии Kubernetes, используемые пулами узлов, должны соответствовать набору версий для уровня управления. Обновление можно выполнить после подготовки кластера с помощью операций пула для каждого узла.

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

> [!Important]
> При запуске одного пула системных узлов для кластера AKS в рабочей среде мы рекомендуем использовать по крайней мере три узла для пула узлов.

Чтобы приступить к работе, создайте кластер AKS с одним пулом узлов. С помощью команды [az group create][az-group-create] создайте группу ресурсов с именем *myResourceGroup* в расположении *eastus*. После этого кластер AKS с именем *myAKSCluster* создается с помощью команды [AZ AKS Create][az-aks-create].

> [!NOTE]
> *Базовая* подсистема балансировки нагрузки **не поддерживается** при использовании нескольких пулов узлов. По умолчанию кластеры AKS создаются с SKU *Стандартный* балансировщика нагрузки из Azure CLI и портале Azure.

```azurecli-interactive
# Create a resource group in East US
az group create --name myResourceGroup --location eastus

# Create a basic single-node AKS cluster
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --vm-set-type VirtualMachineScaleSets \
    --node-count 2 \
    --generate-ssh-keys \
    --load-balancer-sku standard
```

Создание кластера занимает несколько минут.

> [!NOTE]
> Чтобы обеспечить надежное функционирование кластера, необходимо запустить по крайней мере 2 узла в пуле узлов по умолчанию, так как базовые системные службы работают в этом пуле узлов.

Когда кластер будет готов, выполните команду [az aks get-credentials][az-aks-get-credentials], чтобы получить учетные данные кластера для использования с `kubectl`:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="add-a-node-pool"></a>Добавление пула узлов

Кластер, созданный на предыдущем шаге, имеет один пул узлов. Добавим второй пул узлов с помощью команды [az aks nodepool add][az-aks-nodepool-add]. В следующем примере создается пул узлов с именем *mynodepool*, который запускает *3* узла:

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 3
```

> [!NOTE]
> Имя пула узлов должно начинаться с буквы в нижнем регистре и может содержать только буквенно-цифровые символы. Для пулов узлов Linux длина должна составлять от 1 до 12 символов. Для пулов узлов Windows длина должна составлять от 1 до 6 символов.

Чтобы просмотреть состояние пулов узлов, используйте команду [az aks node pool list][az-aks-nodepool-list] и укажите имена группы ресурсов и кластера:

```azurecli-interactive
az aks nodepool list --resource-group myResourceGroup --cluster-name myAKSCluster
```

В следующем примере выходных данных показано, что *mynodepool* успешно создан с тремя узлами в пуле узлов. При создании кластера AKS на предыдущем шаге создается *nodepool1* по умолчанию с числом узлов, равным *2*.

```output
[
  {
    ...
    "count": 3,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

> [!TIP]
> Если при добавлении пула узлов не указан параметр *VmSize*, размер по умолчанию — *Standard_D2s_v3* для пулов узлов Windows и *Standard_DS2_v2* для пулов узлов Linux. Если параметр *OrchestratorVersion* не указан, по умолчанию используется та же версия, что и в плоскости управления.

### <a name="add-a-node-pool-with-a-unique-subnet-preview"></a>Добавление пула узлов с уникальной подсетью (предварительная версия)

Для логической изоляции может потребоваться разделение узлов кластера на отдельные пулы. Эта изоляция может поддерживаться в отдельных подсетях, выделенных для каждого пула узлов в кластере. Это может решить такие требования, как наличие несмежного адресного пространства виртуальной сети для разделения между пулами узлов.

#### <a name="limitations"></a>Ограничения

* Все подсети, назначенные пулам узлов, должны принадлежать одной виртуальной сети.
* Системные модули безопасности должны иметь доступ ко всем узлам и модулям Pod в кластере, чтобы обеспечить такие важные функциональные возможности, как разрешение DNS и туннелирование kubectl logs/exec/port-forward proxy.
* Если вы развернете виртуальную сеть после создания кластера, необходимо обновить кластер (выполнить любую управляемую операцию для кластера, но не для пула узлов, т. к. они не считаются) перед добавлением подсети за пределами исходной CIDR. AKS выдаст ошибку при добавлении пула агентов, хотя изначально это было разрешено. Если вы не уверены, как согласовать свой кластер, обратитесь с запросом в службу поддержки. 
* Политика сети Calico не поддерживается. 
* Политика сети Azure не поддерживается.
* Kube-proxy ожидает одну смежную CIDR и использует ее в трех нижеследующих оптимизациях. См. раздел [Предложения по улучшению Kubernetes](https://github.com/kubernetes/enhancements/tree/master/keps/sig-network/2450-Remove-knowledge-of-pod-cluster-CIDR-from-iptables-rules) и воспользуйтесь командой --cluster-cidr [отсюда](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) для получения дополнительных сведений. В Azure CNI подсеть пула первого узла будет передана kube-proxy. 

Чтобы создать пул узлов с выделенной подсетью, передайте идентификатор ресурса подсети в качестве дополнительного параметра при создании пула узлов.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 3 \
    --vnet-subnet-id <YOUR_SUBNET_RESOURCE_ID>
```

## <a name="upgrade-a-node-pool"></a>Обновление пула узлов

> [!NOTE]
> Операции обновления и масштабирования в кластере или пуле узлов не могут выполняться одновременно, при попытке сделать это будет возвращена ошибка. Вместо этого каждый тип операции должен быть завершен на целевом ресурсе перед следующим запросом к этому же ресурсу. Дополнительные сведения см. в нашем [руководстве по устранению неполадок](./troubleshooting.md#im-receiving-errors-when-trying-to-upgrade-or-scale-that-state-my-cluster-is-being-upgraded-or-has-failed-upgrade).

Команды в этом разделе объясняют, как обновить отдельный пул узлов. Связь между обновлением Kubernetes версии уровня управления и обновлением пула узлов описывается в [разделе ниже](#upgrade-a-cluster-control-plane-with-multiple-node-pools).

> [!NOTE]
> Версия образа ОС пула узлов привязана к версии Kubernetes кластера. Обновления образа ОС будут получаться только после обновления кластера.

Так как в этом примере имеются два пула узлов, для обновления пула узлов необходимо использовать команду [az aks nodepool upgrade][az-aks-nodepool-upgrade]. Чтобы просмотреть доступные обновления, используйте команду [az aks get-upgrades][az-aks-get-upgrades]

```azurecli-interactive
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster
```

Давайте выполним обновление *mynodepool*. Используйте команду [az aks nodepool upgrade][az-aks-nodepool-upgrade], чтобы обновить пул узлов, как показано в следующем примере:

```azurecli-interactive
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --kubernetes-version KUBERNETES_VERSION \
    --no-wait
```

Снова перечислите состояние пулов узлов с помощью команды [az aks node pool list][az-aks-nodepool-list]. В следующем примере показано, что *mynodepool* находится в состоянии *обновления* до версии *KUBERNETES_VERSION*:

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 3,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "KUBERNETES_VERSION",
    ...
    "provisioningState": "Upgrading",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

Обновление узлов до указанной версии занимает несколько минут.

Рекомендуется обновить все пулы узлов в кластере AKS до одной версии Kubernetes. Поведение по умолчанию `az aks upgrade` — обновление всех пулов узлов вместе с уровнем управления для достижения этого выравнивания. Возможность обновления индивидуальных пулов узлов позволяет выполнять последовательное обновление и распланировать обновление модулей Pod в пулах узлов, чтобы поддерживать время работы приложения в указанных выше ограничениях.

## <a name="upgrade-a-cluster-control-plane-with-multiple-node-pools"></a>Обновление уровня управления кластера с несколькими пулами узлов

> [!NOTE]
> Kubernetes использует стандартную схему управления версиями. См. раздел [Семантическое управление версиями](https://semver.org/). Номер версии выражается в виде *x. y. z*, где *x* — основная версия, *y* — дополнительный номер версии, а *z* — версия исправления. Например, в версии *1.12.6* 1 — основной номер версии, 12 — дополнительный номер версии, а 6 — версия исправления. Версия Kubernetes плоскости управления и начальный пул узлов устанавливаются во время создания кластера. При добавлении в кластер для всех дополнительных пулов узлов задается версия Kubernetes. Версии Kubernetes могут отличаться между пулами узлов, а также между пулом узлов и уровнем управления.

Кластер AKS содержит два объекта ресурсов кластера со связанными версиями Kubernetes.

1. Версия Kubernetes уровня управления кластером.
2. Пул узлов с версией Kubernetes.

Уровень управления сопоставляется с одним или несколькими пулами узлов. Поведение операции обновления зависит от того, какая команда Azure CLI используется.

Для обновления уровня управления AKS необходимо использовать `az aks upgrade`. Эта команда обновляет версию уровня управления и все пулы узлов в кластере.

Использование команды `az aks upgrade` с флагом `--control-plane-only` обновляет только уровень управления кластером. Ни один из связанных пулов узлов в кластере не изменяется.

Для обновления отдельных пулов узлов необходимо использовать `az aks nodepool upgrade`. Эта команда обновляет только целевой пул узлов до указанной версии Kubernetes

### <a name="validation-rules-for-upgrades"></a>Правила проверки для обновлений

Допустимые обновления Kubernetes для уровня управления и пулов узлов кластера проверяются следующими наборами правил.

* Правила для допустимых версий для обновления пулов узлов:
   * Версия пула узлов должна иметь ту же *основную* версию, что и уровень управления.
   * *Дополнительная* версия пула узлов должна быть в пределах двух промежуточных *дополнительных* версий уровня управления.
   * Версия пула узлов не может быть больше версии `major.minor.patch`.

* Правила для отправки операции обновления.
   * Нельзя перейти на более раннюю версию уровня управления или пула узлов Kubernetes.
   * Если не указана версия Kubernetes пула узлов, поведение зависит от используемого клиента. Объявление в шаблонах диспетчера ресурсов возвращается к существующей версии, определенной для пула узлов, если она используется; если не задано значение, для возврата применяется версия уровня управления.
   * Можно обновить или масштабировать плоскость управления или пул узлов в определенный момент времени, но нельзя одновременно отправлять несколько операций в один уровень управления или ресурс пула узлов.

## <a name="scale-a-node-pool-manually"></a>Масштабирование пула узлов вручную

По мере изменения требований к рабочей нагрузке приложения может потребоваться масштабирование количества узлов в пуле узлов. Число узлов можно увеличивать или уменьшать.

<!--If you scale down, nodes are carefully [cordoned and drained][kubernetes-drain] to minimize disruption to running applications.-->

Чтобы масштабировать количество узлов в пуле узлов, используйте команду [az aks node pool scale][az-aks-nodepool-scale]. В следующем примере показано масштабирование количества узлов в *mynodepool* до *5*:

```azurecli-interactive
az aks nodepool scale \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 5 \
    --no-wait
```

Снова перечислите состояние пулов узлов с помощью команды [az aks node pool list][az-aks-nodepool-list]. В следующем примере показано, что *mynodepool* находится в состоянии *масштабирования* с новым числом узлов, равным *5*.

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 5,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Scaling",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

Операция масштабирования занимает несколько минут.

## <a name="scale-a-specific-node-pool-automatically-by-enabling-the-cluster-autoscaler"></a>Автоматическое масштабирование определенного пула узлов путем включения автомасштабирования кластера

AKS предлагает отдельную функцию автоматического масштабирования пулов узлов с помощью функции, которая называется [Автомасштабирование кластера](cluster-autoscaler.md). Эту функцию можно включить для любого пула узлов с уникальными минимальным и максимальным числом узлов на каждый пул. Узнайте, как [использовать автомасштабирование кластера на пуле узлов](cluster-autoscaler.md#use-the-cluster-autoscaler-with-multiple-node-pools-enabled).

## <a name="delete-a-node-pool"></a>Удаление пула узлов

Если пул больше не нужен, можно удалить его и подлежащие узлы виртуальных машин. Чтобы удалить пул узлов, выполните команду [az aks node pool delete][az-aks-nodepool-delete] и укажите имя пула узлов. В следующем примере удаляется *mynodepool*, созданный на предыдущих шагах.

> [!CAUTION]
> Отсутствуют варианты восстановления для потери данных, которые могут возникнуть при удалении пула узлов. Если не удается запланировать модули Pod в других пулах узлов, эти способы применения недоступны. Следите за тем, чтобы пул узлов не удалялся, если используемые в нем приложения не применяют резервное копирование данных или не могут выполняться на других пулах узлов в кластере.

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster --name mynodepool --no-wait
```

Следующий пример выходных данных команды [az aks node pool list][az-aks-nodepool-list] показывает, что *mynodepool* находится в состоянии *удаления*:

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 5,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Deleting",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

Удаление узлов и пула узлов занимает несколько минут.

## <a name="specify-a-vm-size-for-a-node-pool"></a>Указание размера виртуальной машины для пула узлов

В предыдущих примерах для создания пула узлов использовался размер виртуальной машины по умолчанию для узлов, созданных в кластере. Более распространенный сценарий — создание пулов узлов с различными размерами и возможностями виртуальных машин. Например, можно создать пул узлов, содержащий узлы с большими объемами ресурсов ЦП или памяти, или пул узлов, который обеспечивает поддержку GPU. На следующем шаге вы будете использовать [отметки и толерантности](#setting-nodepool-taints), чтобы сообщить планировщику Kubernetes, как ограничить доступ к модулям Pod, которые могут выполняться на этих узлах.

В следующем примере мы создадим пул узлов на основе GPU, в котором используется размер виртуальной машины *Standard_NC6*. Эти виртуальные машины созданы на базе видеокарты Tesla K80 от NVIDIA. См. сведения о доступных размерах виртуальных машин в разделе [Размер виртуальных машин Linux в Azure][vm-sizes].

Создайте пул узлов с помощью команды [az aks node pool add][az-aks-nodepool-add]. На этот раз укажите имя *gpunodepool* и используйте параметр `--node-vm-size`, чтобы указать размер *Standard_NC6*.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name gpunodepool \
    --node-count 1 \
    --node-vm-size Standard_NC6 \
    --no-wait
```

В следующем примере выходных данных команды [az aks node pool list][az-aks-nodepool-list] показано, что *gpunodepool* *создает* узлы с указанным параметром *VmSize*.

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 1,
    ...
    "name": "gpunodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "vmSize": "Standard_NC6",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

Создание *gpunodepool* занимает несколько минут.

## <a name="specify-a-taint-label-or-tag-for-a-node-pool"></a>Указание параметров taint (отметка), метки или тега для пула узлов

### <a name="setting-nodepool-taints"></a>Настройка отметок (taints) пула узлов

При создании пула узлов можно добавить отметки, метки или теги в этот пул узлов. При добавлении отметки, метки или тега все узлы в этом пуле узлов также получают эти отметку, метку или тег.

Чтобы создать пул узлов с отметкой, используйте команду [az aks nodepool add][az-aks-nodepool-add]. Укажите имя *taintnp* и используйте параметр `--node-taints`, чтобы указать *sku=gpu:NoSchedule* для отметки.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name taintnp \
    --node-count 1 \
    --node-taints sku=gpu:NoSchedule \
    --no-wait
```

> [!NOTE]
> Отметку можно задать для пулов узлов только во время его создания.

В следующем примере выходных данных команды [az aks nodepool list][az-aks-nodepool-list] показано, что *taintnp* *создает* узлы с указанным параметром *nodeTaints*.

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster

[
  {
    ...
    "count": 1,
    ...
    "name": "taintnp",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "nodeTaints":  [
      "sku=gpu:NoSchedule"
    ],
    ...
  },
 ...
]
```

Сведения об отметках отображаются в Kubernetes для обработки правил планирования для узлов. Для ограничения рабочих нагрузок, которые могут выполняться на узлах, в планировщике Kubernetes используются параметры taint (отметка) и toleration (толерантность).

* **Отметка** применяется к узлу, указывая, что на него могут назначаться только определенные модули pod.
* Затем к модулям pod применяется параметр **toleration**, указывающий, что они *допускают* метку узла.

Дополнительные сведения о том, как использовать расширенные функции Kubernetes Schedule, см. в статье [Рекомендации по использованию расширенных функций планировщика в AKS][taints-tolerations].

На предыдущем шаге вы применили отметку *sku=gpu:NoSchedule* при создании пула узлов. В следующем примере манифеста YAML используется разрешение, позволяющее планировщику Kubernetes запускать модуль NGINX на узле в этом пуле узлов.

Создайте файл `nginx-toleration.yaml` и скопируйте в него следующий пример кода YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.9-alpine
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 1
        memory: 2G
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

Запланируйте модуль с помощью команды `kubectl apply -f nginx-toleration.yaml`.

```console
kubectl apply -f nginx-toleration.yaml
```

Планирование объекта Pod и извлечение образа NGINX займет несколько секунд. Чтобы просмотреть состояние объекта Pod, используйте команду [kubectl describe pod][kubectl-describe]. В следующем сжатом примере выходных данных показано, что применяется толерантность *sku=gpu:NoSchedule*. В разделе событий планировщик назначил ему узел *aks-taintnp-28993262-vmss000000*.

```console
kubectl describe pod mypod
```

```output
[...]
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
                 sku=gpu:NoSchedule
Events:
  Type    Reason     Age    From                Message
  ----    ------     ----   ----                -------
  Normal  Scheduled  4m48s  default-scheduler   Successfully assigned default/mypod to aks-taintnp-28993262-vmss000000
  Normal  Pulling    4m47s  kubelet             pulling image "mcr.microsoft.com/oss/nginx/nginx:1.15.9-alpine"
  Normal  Pulled     4m43s  kubelet             Successfully pulled image "mcr.microsoft.com/oss/nginx/nginx:1.15.9-alpine"
  Normal  Created    4m40s  kubelet             Created container
  Normal  Started    4m40s  kubelet             Started container
```

На узлах в *taintnp* можно планировать только те модули, к которым применена эта толерантность. Любой другой модуль будет запланирован в пуле узлов *nodepool1*. Если вы создаете дополнительные пулы узлов, вы можете использовать дополнительные отметки и толерантности, чтобы ограничить возможности планирования модулей в этих ресурсах узлов.

### <a name="setting-nodepool-labels"></a>Настройка меток (labels) пула узлов

Добавить метки в пул узлов можно также во время его создания. Метки, заданные в пуле узлов, добавляются в каждый узел в пуле узлов. Эти [метки отображаются в Kubernetes][kubernetes-labels] для обработки правил планирования для узлов.

Чтобы создать пул узлов с меткой, используйте команду [az aks nodepool add][az-aks-nodepool-add]. Укажите имя *labelnp* и используйте параметр `--labels`, чтобы указать *dept=IT* и *costcenter=9999* для меток.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name labelnp \
    --node-count 1 \
    --labels dept=IT costcenter=9999 \
    --no-wait
```

> [!NOTE]
> Метку можно задать для пулов узлов только во время его создания. Метки также должны быть парой "ключ-значение" и иметь [допустимый синтаксис][kubernetes-label-syntax].

В следующем примере выходных данных команды [az aks nodepool list][az-aks-nodepool-list] показано, что *labelnp* *создает* узлы с указанным параметром *nodeLabels*.

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster

[
  {
    ...
    "count": 1,
    ...
    "name": "labelnp",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "nodeLabels":  {
      "dept": "IT",
      "costcenter": "9999"
    },
    ...
  },
 ...
]
```

### <a name="setting-nodepool-azure-tags"></a>Настройка тегов Azure (tags) для пула узлов

Вы можете применить тег Azure к пулам узлов в кластере AKS. Теги, применяемые к пулу узлов, применяются к каждому узлу в пуле узлов и сохраняются при обновлении. Теги также применяются к новым узлам, добавленным в пул узлов во время операций горизонтального увеличения масштаба. Добавление тега может помочь в таких задачах, как отслеживание политик или оценка затрат.

Теги Azure содержат ключи, которые не учитывают регистр для операций, например при извлечении тега путем поиска по ключу. В этом случае тег с заданным ключом будет обновляться или извлекаться независимо от регистра. Значения тегов чувствительны к регистру.

В AKS, если несколько тегов имеют одинаковые ключи, но различаются по регистру, используется тег, который является первым в алфавитном порядке. Например, при задании `{"Key1": "val1", "kEy1": "val2", "key1": "val3"}` будут использоваться `Key1` и `val1`.

Создайте пул узлов с помощью команды [az aks nodepool add][az-aks-nodepool-add]. Укажите имя *tagnodepool* и используйте параметр `--tag`, чтобы указать *dept=IT* и *costcenter=9999* для тегов.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name tagnodepool \
    --node-count 1 \
    --tags dept=IT costcenter=9999 \
    --no-wait
```

> [!NOTE]
> Можно также использовать параметр `--tags` в команде [az aks nodepool update][az-aks-nodepool-update], а также во время создания кластера. Во время создания кластера параметр `--tags` применяет тег к начальному пулу узлов, созданному вместе с кластером. Все имена тегов должны соответствовать ограничениям, приведенным в разделе [Использование тегов для организации ресурсов Azure][tag-limitation]. При обновлении пула узлов с параметром `--tags` все существующие значения тегов обновляются, а новые теги добавляются. Например, если в пуле узлов были заданы значения *dept=ИТ* и *costcenter = 9999* для тегов и вы обновили его, указав в команде *team=dev* и *costcenter=111*, для тегов пула узлов будут актуальны значения *dept=IT*, *costcenter = 111* и *team=dev*.

В следующем примере выходных данных команды [az aks nodepool list][az-aks-nodepool-list] показано, что *tagnodepool* *создает* узлы с указанным параметром *tag*.

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 1,
    ...
    "name": "tagnodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "tags": {
      "dept": "IT",
      "costcenter": "9999"
    },
    ...
  },
 ...
]
```

## <a name="manage-node-pools-using-a-resource-manager-template"></a>Управление пулами узлов с помощью шаблона диспетчера ресурсов

При использовании шаблона Azure Resource Manager для создания и управления ресурсами, как правило, можно обновить параметры в шаблоне и повторно развернуть, чтобы обновить ресурс. При использовании пулов узлов в AKS изначальный профиль пула узлов не может быть обновлен после создания кластера AKS. Такое поведение означает, что вы не можете обновить существующий шаблон диспетчер ресурсов, внести изменения в пулы узлов и повторно развернуть. Вместо этого необходимо создать отдельный шаблон диспетчера ресурсов, который обновляет только пулы узлов для существующего кластера AKS.

Создайте шаблон, например `aks-agentpools.json`, и вставьте следующий пример манифеста. В примере шаблона задаются нижеследующие настройки.

* Обновляется пул узлов *Linux* с именем *myagentpool* для запуска трех узлов.
* Для узлов в пуле применяется запуск Kubernetes версии *1.15.7*.
* Определяется размер узла как *Standard_DS2_v2*.

Измените эти значения, чтобы обновить, добавить или удалить пулы узлов по мере необходимости.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "clusterName": {
            "type": "string",
            "metadata": {
                "description": "The name of your existing AKS cluster."
            }
        },
        "location": {
            "type": "string",
            "metadata": {
                "description": "The location of your existing AKS cluster."
            }
        },
        "agentPoolName": {
            "type": "string",
            "defaultValue": "myagentpool",
            "metadata": {
                "description": "The name of the agent pool to create or update."
            }
        },
        "vnetSubnetId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The Vnet subnet resource ID for your existing AKS cluster."
            }
        }
    },
    "variables": {
        "apiVersion": {
            "aks": "2020-01-01"
        },
        "agentPoolProfiles": {
            "maxPods": 30,
            "osDiskSizeGB": 0,
            "agentCount": 3,
            "agentVmSize": "Standard_DS2_v2",
            "osType": "Linux",
            "vnetSubnetId": "[parameters('vnetSubnetId')]"
        }
    },
    "resources": [
        {
            "apiVersion": "2020-01-01",
            "type": "Microsoft.ContainerService/managedClusters/agentPools",
            "name": "[concat(parameters('clusterName'),'/', parameters('agentPoolName'))]",
            "location": "[parameters('location')]",
            "properties": {
                "maxPods": "[variables('agentPoolProfiles').maxPods]",
                "osDiskSizeGB": "[variables('agentPoolProfiles').osDiskSizeGB]",
                "count": "[variables('agentPoolProfiles').agentCount]",
                "vmSize": "[variables('agentPoolProfiles').agentVmSize]",
                "osType": "[variables('agentPoolProfiles').osType]",
                "storageProfile": "ManagedDisks",
                "type": "VirtualMachineScaleSets",
                "vnetSubnetID": "[variables('agentPoolProfiles').vnetSubnetId]",
                "orchestratorVersion": "1.15.7"
            }
        }
    ]
}
```

Разверните этот шаблон с помощью команды [az deployment group create][az-deployment-group-create], как показано в следующем примере. Появится запрос на ввод имени и расположения существующего кластера AKS.

```azurecli-interactive
az deployment group create \
    --resource-group myResourceGroup \
    --template-file aks-agentpools.json
```

> [!TIP]
> Вы можете добавить тег в пул узлов, добавив свойство *Tag* в шаблон, как показано в следующем примере.
> 
> ```json
> ...
> "resources": [
> {
>   ...
>   "properties": {
>     ...
>     "tags": {
>       "name1": "val1"
>     },
>     ...
>   }
> }
> ...
> ```

Обновление кластера AKS может занять несколько минут в зависимости от параметров пула узлов и операций, определенных в шаблоне диспетчера ресурсов.

## <a name="assign-a-public-ip-per-node-for-your-node-pools"></a>Назначение общедоступного IP-адреса на узел для пулов узлов

Узлы AKS не нуждаются в собственных общедоступных IP-адресах для обмена данными. Однако сценарии могут потребовать, чтобы узлы в пуле узлов получали собственные выделенные общедоступные IP-адреса. Типичный сценарий — для игровых рабочих нагрузок, в которых консоль должна установить прямое подключение к облачной виртуальной машине для снижения числа прыжков. Этот сценарий можно достичь на AKS, задав общедоступный IP-адрес узла.

В первую очередь создается новая группа ресурсов.

```azurecli-interactive
az group create --name myResourceGroup2 --location eastus
```

Создайте новый кластер AKS и подключите общедоступный IP-адрес для узлов. Каждый узел в пуле узлов получает уникальный общедоступный IP-адрес. Это можно проверить, просмотрев экземпляры масштабируемых наборов виртуальных машин.

```azurecli-interactive
az aks create -g MyResourceGroup2 -n MyManagedCluster -l eastus  --enable-node-public-ip
```

Для существующих кластеров AKS можно также добавить новый пул узлов и подключить общедоступный IP-адрес для узлов.

```azurecli-interactive
az aks nodepool add -g MyResourceGroup2 --cluster-name MyManagedCluster -n nodepool2 --enable-node-public-ip
```

### <a name="use-a-public-ip-prefix"></a>Добавление префикса общедоступных IP-адресов

[Использование префикса общедоступного IP-адреса][public-ip-prefix-benefits] имеет ряд преимуществ. AKS поддерживает использование адресов из существующего префикса общедоступного IP-адреса для узлов, передавая идентификатор ресурса с флагом `node-public-ip-prefix` при создании нового кластера или добавлении пула узлов.

Сначала создайте префикс общедоступного IP-адреса с помощью команды [az network public-ip prefix create][az-public-ip-prefix-create]:

```azurecli-interactive
az network public-ip prefix create --length 28 --location eastus --name MyPublicIPPrefix --resource-group MyResourceGroup3
```

Просмотрите выходные данные и запишите значение `id` для префикса.

```output
{
  ...
  "id": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup3/providers/Microsoft.Network/publicIPPrefixes/MyPublicIPPrefix",
  ...
}
```

Наконец, при создании нового кластера или добавлении нового пула узлов используйте флаг `node-public-ip-prefix` и передайте идентификатор ресурса префикса.

```azurecli-interactive
az aks create -g MyResourceGroup3 -n MyManagedCluster -l eastus --enable-node-public-ip --node-public-ip-prefix /subscriptions/<subscription-id>/resourcegroups/MyResourceGroup3/providers/Microsoft.Network/publicIPPrefixes/MyPublicIPPrefix
```

### <a name="locate-public-ips-for-nodes"></a>Поиск общедоступных IP-адресов для узлов

Общедоступные IP-адреса для узлов можно узнать различными способами.

* Используйте команду Azure CLI [az vmss list-instance-public-ips][az-list-ips].
* Используйте [команды PowerShell или bash][vmss-commands]. 
* Общедоступные IP-адреса можно также просмотреть в портале Azure, просмотрев экземпляры в масштабируемом наборе виртуальных машин.

> [!Important]
> [Группа ресурсов узла][node-resource-group] содержит узлы и их общедоступные IP-адреса. Используйте группу ресурсов узла при выполнении команд, чтобы найти общедоступные IP-адреса узлов.

```azurecli
az vmss list-instance-public-ips -g MC_MyResourceGroup2_MyManagedCluster_eastus -n YourVirtualMachineScaleSetName
```

## <a name="clean-up-resources"></a>Очистка ресурсов

В этой статье вы создали кластер AKS, содержащий узлы на основе GPU. Чтобы уменьшить излишнюю стоимость, можно удалить *gpunodepool* или весь кластер AKS.

Чтобы удалить пул узлов на основе GPU, используйте команду [az aks nodepool delete][az-aks-nodepool-delete], как показано в следующем примере.

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster --name gpunodepool
```

Чтобы удалить сам кластер, воспользуйтесь командой удаления [az group delete][az-group-delete], чтобы удалить группу ресурсов AKS.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

Кроме того, можно удалить дополнительный кластер, созданный для сценария общедоступного IP-адреса для пулов узлов.

```azurecli-interactive
az group delete --name myResourceGroup2 --yes --no-wait
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [пулах системных узлов][use-system-pool].

В этой статье показано, как создать несколько пулов узлов и управлять ими в кластере AKS. Дополнительные сведения об управлении модулями Pod в пулах узлов см. в разделе рекомендации [по использованию расширенных функций планировщика в AKS][operator-best-practices-advanced-scheduler].

Сведения о создании и использовании пулов узлов контейнера Windows Server см. в разделе [Создание контейнера Windows Server в AKS][aks-windows].

Использование [групп размещения близкого взаимодействия][reduce-latency-ppg], чтобы сократить задержку приложений AKS.

<!-- EXTERNAL LINKS -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-taint]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubernetes-labels]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
[kubernetes-label-syntax]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set

<!-- INTERNAL LINKS -->
[aks-windows]: windows-container-cli.md
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest&preserve-view=true#az_aks_get_credentials
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest&preserve-view=true#az_aks_create
[az-aks-get-upgrades]: /cli/azure/aks?view=azure-cli-latest&preserve-view=true#az_aks_get_upgrades
[az-aks-nodepool-add]: /cli/azure/aks/nodepool?view=azure-cli-latest&preserve-view=true#az_aks_nodepool_add
[az-aks-nodepool-list]: /cli/azure/aks/nodepool?view=azure-cli-latest&preserve-view=true#az_aks_nodepool_list
[az-aks-nodepool-update]: /cli/azure/aks/nodepool?view=azure-cli-latest&preserve-view=true#az_aks_nodepool_update
[az-aks-nodepool-upgrade]: /cli/azure/aks/nodepool?view=azure-cli-latest&preserve-view=true#az_aks_nodepool_upgrade
[az-aks-nodepool-scale]: /cli/azure/aks/nodepool?view=azure-cli-latest&preserve-view=true#az_aks_nodepool_scale
[az-aks-nodepool-delete]: /cli/azure/aks/nodepool?view=azure-cli-latest&preserve-view=true#az_aks_nodepool_delete
[az-extension-add]: /cli/azure/extension?view=azure-cli-latest&preserve-view=true#az_extension_add
[az-extension-update]: /cli/azure/extension?view=azure-cli-latest&preserve-view=true#az_extension_update
[az-group-create]: /cli/azure/group?view=azure-cli-latest&preserve-view=true#az_group_create
[az-group-delete]: /cli/azure/group?view=azure-cli-latest&preserve-view=true#az_group_delete
[az-deployment-group-create]: /cli/azure/deployment/group?view=azure-cli-latest&preserve-view=true#az_deployment_group_create
[gpu-cluster]: gpu-cluster.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[quotas-skus-regions]: quotas-skus-regions.md
[supported-versions]: supported-kubernetes-versions.md
[tag-limitation]: ../azure-resource-manager/management/tag-resources.md
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[vm-sizes]: ../virtual-machines/sizes.md
[use-system-pool]: use-system-pools.md
[ip-limitations]: ../virtual-network/virtual-network-ip-addresses-overview-arm#standard
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[vmss-commands]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md#public-ipv4-per-virtual-machine
[az-list-ips]: /cli/azure/vmss?view=azure-cli-latest&preserve-view=true#az_vmss_list_instance_public_ips
[reduce-latency-ppg]: reduce-latency-ppg.md
[public-ip-prefix-benefits]: ../virtual-network/public-ip-address-prefix.md#why-create-a-public-ip-address-prefix
[az-public-ip-prefix-create]: /cli/azure/network/public-ip/prefix?view=azure-cli-latest&preserve-view=true#az_network_public_ip_prefix_create
