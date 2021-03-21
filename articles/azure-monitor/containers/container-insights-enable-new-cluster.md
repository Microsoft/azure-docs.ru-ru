---
title: Мониторинг нового кластера Azure Kubernetes Service (AKS) | Документация Майкрософт
description: Узнайте, как включить мониторинг для нового кластера Azure Kubernetes Service (AKS) с подпиской Container Insights.
ms.topic: conceptual
ms.date: 04/25/2019
ms.custom: devx-track-terraform, devx-track-azurecli
ms.openlocfilehash: 9b6c4f8a05b8e7a350ebd5afd677e8bb2ee6e9b4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101717576"
---
# <a name="enable-monitoring-of-a-new-azure-kubernetes-service-aks-cluster"></a>Включение мониторинга нового кластера Azure Kubernetes Service (AKS)

В этой статье описывается, как настроить контейнерную аналитику для мониторинга управляемого кластера Kubernetes, размещенного в [службе Kubernetes Azure](../../aks/index.yml) , которая готовится к развертыванию в вашей подписке.

Мониторинг кластера AKS можно включить с помощью одного из поддерживаемых методов:

* Azure CLI
* Terraform

## <a name="enable-using-azure-cli"></a>Включение мониторинга с помощью Azure CLI

Чтобы включить мониторинг нового кластера AKS, созданного с помощью Azure CLI, следуйте указаниям в разделе [Создание кластера AKS](../../aks/kubernetes-walkthrough.md#create-aks-cluster).  

>[!NOTE]
>Если вы решили использовать Azure CLI, необходимо сначала установить интерфейс командной строки и использовать его локально. Необходимо запустить Azure CLI версии 2.0.74 или более поздней. Для определения версии выполните `az --version`. Если вам необходимо установить или обновить Azure CLI, ознакомьтесь со статьей [Установка Azure CLI 2.0](/cli/azure/install-azure-cli). Если вы установили расширение CLI AKS-Preview версии 0.4.12 или более поздней, удалите все внесенные изменения, чтобы включить предварительную версию расширения, так как она может переопределить поведение Azure CLI по умолчанию, так как функции предварительной версии AKS недоступны в Azure US министерстве Cloud.

## <a name="enable-using-terraform"></a>Включение с помощью Terraform

Если вы [развертываете новый кластер AKS с помощью Terraform](/azure/developer/terraform/create-k8s-cluster-with-tf-and-aks), нужно указать аргументы, необходимые в профиле [для создания рабочей области Log Analytics](https://www.terraform.io/docs/providers/azurerm/r/log_analytics_workspace.html), если вы не решили указать уже существующую. 

>[!NOTE]
>Если вы решили использовать Terraform, необходимо запустить Terraform Azure RM Provider версии 1.17.0 или выше.

Сведения о добавлении в рабочую область сведений о контейнере см. в разделе [azurerm_log_analytics_solution](https://www.terraform.io/docs/providers/azurerm/r/log_analytics_solution.html) и завершение профиля путем включения [**addon_profile**](https://www.terraform.io/docs/providers/azurerm/r/kubernetes_cluster.html#addon_profile) и указания **oms_agent**. 

После включения мониторинга и успешного выполнения всех задач настройки можно отслеживать производительность кластера одним из двух способов:

* Непосредственно в кластере AKS, выбрав **Работоспособность** в левой области.
* Выбрав плитку **Monitor Container insights** (Мониторинг аналитических сведений о контейнере) на странице кластеров AKS для выбранного кластера. В Azure Monitor выберите **Работоспособность** в левой области. 

  ![Параметры для выбора контейнера аналитики в AKS](./media/container-insights-onboard/kubernetes-select-monitoring-01.png)

После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть метрики работоспособности кластера. 

## <a name="verify-agent-and-solution-deployment"></a>Проверка развертывания агента и решения
В версии агента *06072018* и выше есть возможность удостовериться, что агент и решение были успешно развернуты. В более ранних версиях агента возможна только проверка развертывания агента.

### <a name="agent-version-06072018-or-later"></a>Версия агента 06072018 или более поздняя
Выполните следующую команду, чтобы проверить успешное развертывание агента. 

```
kubectl get ds omsagent --namespace=kube-system
```

Результат должен выглядеть приблизительно, как показано ниже, что означает успешное выполнение развертывания:

```
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```  

Чтобы проверить развертывание решения, выполните следующую команду:

```
kubectl get deployment omsagent-rs -n=kube-system
```

Результат должен выглядеть приблизительно, как показано ниже, что означает успешное выполнение развертывания:

```
User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system 
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
omsagent   1         1         1            1            3h
```

### <a name="agent-version-earlier-than-06072018"></a>Агент версии более ранней, чем 06072018

Чтобы проверить, правильно ли развернут агент Log Analytics версии более ранней, чем *06072018*, выполните следующую команду:  

```
kubectl get ds omsagent --namespace=kube-system
```

Результат должен выглядеть приблизительно, как показано ниже, что означает успешное выполнение развертывания:  

```
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system 
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```  

## <a name="view-configuration-with-cli"></a>Просмотр конфигурации с помощью CLI
С помощью команды `aks show` можно получить определенные сведения, например включено ли решение, идентификатор ресурса рабочей области Log Analytics, а также сводные данные о кластере.  

```azurecli
az aks show -g <resourceGroupofAKSCluster> -n <nameofAksCluster>
```

Через несколько минут выполнение команды завершается и отображаются сведения о решении в формате JSON.  В результатах выполнения команды должен отобразиться профиль надстройки наблюдения. Вы должны увидеть результат, аналогичный следующему:

```
"addonProfiles": {
    "omsagent": {
      "config": {
        "logAnalyticsWorkspaceResourceID": "/subscriptions/<WorkspaceSubscription>/resourceGroups/<DefaultWorkspaceRG>/providers/Microsoft.OperationalInsights/workspaces/<defaultWorkspaceName>"
      },
      "enabled": true
    }
  }
```

## <a name="next-steps"></a>Дальнейшие действия

* Если при попытке подключить решение у вас возникли проблемы, ознакомьтесь с [руководством по устранению неполадок](container-insights-troubleshoot.md).

* С включенным наблюдением для получения сведений о работоспособности и использовании ресурсов кластером AKS и рабочими нагрузками, которые выполняются на них, Узнайте, [как использовать](container-insights-analyze.md) аналитику контейнера.

