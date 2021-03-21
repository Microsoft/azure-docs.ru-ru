---
title: Управление агентом контейнеров аналитики | Документация Майкрософт
description: В этой статье описывается управление наиболее распространенными задачами обслуживания с контейнерным Log Analytics агентом, используемым контейнерной аналитикой.
ms.topic: conceptual
ms.date: 07/21/2020
ms.openlocfilehash: 2a0c32ef797a953eca794e16fe0ace5e967f339f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101713802"
---
# <a name="how-to-manage-the-container-insights-agent"></a>Управление агентом контейнеров аналитики

В контейнере аналитики используется контейнерная версия агента Log Analytics для Linux. После первоначального развертывания агента есть некоторые рутинные или необязательные задачи, которые нужно будет выполнить в течение его жизненного цикла. В этой статье подробно описывается, как вручную обновить агент и отключить коллекцию переменных среды для определенного контейнера. 

## <a name="how-to-upgrade-the-container-insights-agent"></a>Обновление агента Container Insights

В контейнере аналитики используется контейнерная версия агента Log Analytics для Linux. При выпуске новой версии агент обновляется автоматически в управляемых кластерах Kubernetes, размещенных в Службе Azure Kubernetes (AKS) и Azure Red Hat OpenShift версии 3.x. В [гибридных кластерах Kubernetes](container-insights-hybrid-setup.md) и Azure Red Hat OpenShift версии 4. x агент не управляется и необходимо вручную обновить агент.

На случай сбоя обновления агента для кластера, размещенного в AKS или Azure Red Hat OpenShift версии 3.x, в этой статье также описывается процесс обновления агента вручную. Выпущенные версии см. в [объявлениях о выпусках агента](https://github.com/microsoft/docker-provider/tree/ci_feature_prod).

### <a name="upgrade-agent-on-aks-cluster"></a>Обновление агента в кластере AKS

Процесс обновления агента в кластере AKS состоит из двух простых шагов. Первым шагом является отключение мониторинга с помощью контейнера Insights с использованием Azure CLI. Выполните действия, описанные в [этой статье](container-insights-optout.md?#azure-cli). С помощью Azure CLI можно удалить агент из узлов в кластере, не влияя на решение и соответствующие данные, хранящиеся в рабочей области. 

>[!NOTE]
>Пока вы выполняете обслуживание, узлы в кластере не переадресовывают собранные данные, а представления производительности не будут отображать данные в период между удалением агента и установкой новой версии. 
>

Чтобы установить новую версию агента, выполните действия, описанные в статье о [включении мониторинга с помощью Azure CLI](container-insights-enable-new-cluster.md#enable-using-azure-cli).  

После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть обновленные метрики работоспособности кластера. Чтобы проверить успешность обновления агента, можно выполнить одно из следующих действий.

* Выполните команду: `kubectl get pod <omsagent-pod-name> -n kube-system -o=jsonpath='{.spec.containers[0].image}'` . В возвращенном состоянии Обратите внимание на значение в разделе **Image** for omsagent в разделе *Containers (контейнеры* ) выходных данных.
* На вкладке **узлы** выберите узел кластера и на панели **свойства** справа запишите значение в поле **тег образа агента**.

Указанная версия агента должна соответствовать последней версии, указанной на странице [журнала выпусков](https://github.com/microsoft/docker-provider/tree/ci_feature_prod) .

### <a name="upgrade-agent-on-hybrid-kubernetes-cluster"></a>Обновление агента в гибридном кластере Kubernetes

Выполните следующие действия, чтобы обновить агент в кластере Kubernetes, работающем на:

* самоуправляемых кластерах Kubernetes, размещенных в Azure с использованием AKS Engine;
* самоуправляемых кластерах Kubernetes, размещенных на Azure Stack или локально с использованием AKS Engine;
* Red Hat OpenShift версии 4.х.

Если рабочая область Log Analytics находится в коммерческой службе Azure, выполните следующую команду.

```console
$ helm upgrade --name myrelease-1 \
--set omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<my_prod_cluster> incubator/azuremonitor-containers
```

Если Рабочая область Log Analytics находится в Azure для Китая (21Vianet), выполните следующую команду:

```console
$ helm upgrade --name myrelease-1 \
--set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
```

Если рабочая область Log Analytics находится в Azure для государственных организаций США, выполните следующую команду.

```console
$ helm upgrade --name myrelease-1 \
--set omsagent.domain=opinsights.azure.us,omsagent.secret.wsid=<your_workspace_id>,omsagent.secret.key=<your_workspace_key>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
```

### <a name="upgrade-agent-on-azure-red-hat-openshift-v4"></a>Обновление агента на Azure Red Hat OpenShift версии 4

Выполните следующие действия, чтобы обновить агент в кластере Kubernetes, работающем на Azure Red Hat OpenShift версии 4.х. 

>[!NOTE]
>Azure Red Hat OpenShift версии 4.x поддерживает только работу в коммерческом облаке Azure.
>

```console
curl -o upgrade-monitoring.sh -L https://aka.ms/upgrade-monitoring-bash-script
export azureAroV4ClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.RedHatOpenShift/OpenShiftClusters/<clusterName>"
bash upgrade-monitoring.sh --resource-id $ azureAroV4ClusterResourceId
```

Дополнительные сведения об использовании субъекта-службы с этой командой см. в статье **Использование субъекта-службы** в разделе [Включение мониторинга кластера Kubernetes с поддержкой ARC в Azure](container-insights-enable-arc-enabled-clusters.md#enable-using-bash-script) .

### <a name="upgrade-agent-on-azure-arc-enabled-kubernetes"></a>Обновление агента в службе "Дуга" в Azure с включенной Kubernetes

Выполните следующую команду, чтобы обновить агент в кластере Kubernetes с включенной службой "Дуга Azure".

```console
curl -o upgrade-monitoring.sh -L https://aka.ms/upgrade-monitoring-bash-script
export azureArcClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
bash upgrade-monitoring.sh --resource-id $azureArcClusterResourceId
```

Дополнительные сведения об использовании субъекта-службы с этой командой см. в статье **Использование субъекта-службы** в разделе [Включение мониторинга кластера Kubernetes с поддержкой ARC в Azure](container-insights-enable-arc-enabled-clusters.md#enable-using-bash-script) .


## <a name="how-to-disable-environment-variable-collection-on-a-container"></a>Как отключить коллекцию переменных среды для контейнера

Служба "аналитика контейнеров" собирает переменные среды из контейнеров, работающих в Pod, и представляет их на панели свойств выбранного контейнера в представлении " **контейнеры** ". Этим поведением можно управлять путем отключения коллекции для определенного контейнера либо во время развертывания кластера Kubernetes, либо после, задав переменную среды *AZMON_COLLECT_ENV*. Эта возможность доступна в версии агента ciprod11292018 и более поздних версиях.  

Чтобы отключить коллекцию переменных среды для нового или существующего контейнера, задайте переменную **AZMON_COLLECT_ENV** со значением **False** в YAML-файле конфигурации развертывания Kubernetes. 

```yaml
- name: AZMON_COLLECT_ENV  
  value: "False"  
```

Выполните следующую команду, чтобы применить изменения к кластерам Kubernetes, иным чем Azure Red Hat OpenShift: `kubectl apply -f  <path to yaml file>`. Чтобы изменить ConfigMap и применить это изменение для кластеров Azure Red Hat OpenShift, выполните следующую команду.

```bash
oc edit configmaps container-azm-ms-agentconfig -n openshift-azure-logging
```

Это откроет текстовый редактор по умолчанию. После задания переменной сохраните файл в редакторе.

Чтобы убедиться, что изменение конфигурации было применено, выберите контейнер в представлении " **контейнеры** " в поле "аналитика контейнера" и на панели свойств разверните узел **переменные среды**.  В разделе должна отображаться только переменная, созданная ранее, — **AZMON_COLLECT_ENV=FALSE**. Для всех других контейнеров в разделе переменных среды должны быть перечислены все обнаруженные переменные среды.

Можно повторно включить обнаружение переменных среды. Для этого примените тот же процесс, что и ранее, и измените значение с **False** на **True**, а затем снова запустите команду `kubectl`, чтобы обновить контейнер.  

```yaml
- name: AZMON_COLLECT_ENV  
  value: "True"  
```  

## <a name="next-steps"></a>Дальнейшие действия

Если при попытке обновления агента у вас возникли проблемы, ознакомьтесь с [руководством по устранению неполадок](container-insights-troubleshoot.md).
