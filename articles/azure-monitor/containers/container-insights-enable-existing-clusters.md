---
title: Мониторинг кластера службы Azure Kubernetes Service (AKS) | Документация Майкрософт
description: Узнайте, как включить мониторинг кластера Azure Kubernetes Service (AKS) с помощью контейнера аналитики, уже развернутой в вашей подписке.
ms.topic: conceptual
ms.date: 09/12/2019
ms.custom: devx-track-terraform, devx-track-azurecli
ms.openlocfilehash: fba12fb8dd5c6c91883a88d4acfdca6c6d332c24
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105109630"
---
# <a name="enable-monitoring-of-azure-kubernetes-service-aks-cluster-already-deployed"></a>Включение мониторинга уже развернутого кластера Azure Kubernetes Service (AKS)

В этой статье описывается, как настроить контейнер аналитики для мониторинга управляемого кластера Kubernetes, размещенного в [службе Kubernetes Azure](../../aks/index.yml) , которая уже была развернута в вашей подписке.

Вы можете включить мониторинг кластера AKS, который уже развернут, с помощью одного из поддерживаемых методов:

* Azure CLI
* Terraform
* [Из Azure Monitor](#enable-from-azure-monitor-in-the-portal) или [непосредственно из кластера AKS](#enable-directly-from-aks-cluster-in-the-portal) в портал Azure
* С [предоставленным шаблоном Azure Resource Manager](#enable-using-an-azure-resource-manager-template) с помощью командлета Azure PowerShell `New-AzResourceGroupDeployment` или с помощью Azure CLI.

Если вы подключаете существующий кластер AKS к рабочей области Azure Log Analytics в другой подписке, поставщик ресурсов Microsoft. ContainerService должен быть зарегистрирован в подписке, в которой была создана Log Analytics Рабочая область. Дополнительные сведения см. в статье [Регистрация поставщика ресурсов](../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com).

## <a name="enable-using-azure-cli"></a>Включение мониторинга с помощью Azure CLI

Ниже показано, как включить мониторинг кластера AKS с помощью Azure CLI. В этом примере не требуется предварительно создавать или указывать существующую рабочую область. Эта команда упрощает процесс включения за счет создания рабочей области по умолчанию в стандартной группе ресурсов подписки кластера AKS, если эта область еще не создана в регионе.  Созданная Рабочая область по умолчанию напоминает формат *DefaultWorkspace- \<GUID> - \<Region>*.

```azurecli
az aks enable-addons -a monitoring -n MyExistingManagedCluster -g MyExistingManagedClusterRG
```

Результат должен выглядеть так:

```output
provisioningState       : Succeeded
```

### <a name="integrate-with-an-existing-workspace"></a>Интеграция с существующей рабочей областью

Если вы предпочитаете интегрировать с существующей рабочей областью, выполните следующие действия, чтобы сначала указать полный идентификатор ресурса Log Analytics рабочей области, необходимый для `--workspace-resource-id` параметра, а затем выполнить команду, чтобы включить надстройку мониторинга для указанной рабочей области.

1. Перечислите все подписки, к которым у вас есть доступ, с помощью следующей команды:

    ```azurecli
    az account list --all -o table
    ```

    Результат должен выглядеть так:

    ```output
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   68627f8c-91fO-4905-z48q-b032a81f8vy0  Enabled  True
    ```

    Скопируйте значение для **SubscriptionId**.

2. Перейдите к подписке, в которой размещена рабочая область Log Analytics, с помощью следующей команды:

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

3. В следующем примере отображается список рабочих областей в подписках в формате JSON по умолчанию.

    ```azurecli
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    В выходных данных найдите имя рабочей области, а затем скопируйте полный идентификатор ресурса этой Log Analytics рабочей области под **идентификатором** поля.

4. Выполните следующую команду, чтобы включить надстройку мониторинга, заменив значение `--workspace-resource-id` параметра. Строковое значение должно быть заключено в двойные кавычки:

    ```azurecli
    az aks enable-addons -a monitoring -n ExistingManagedCluster -g ExistingManagedClusterRG --workspace-resource-id "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<WorkspaceName>"
    ```

    Результат должен выглядеть так:

    ```output
    provisioningState       : Succeeded
    ```

## <a name="enable-using-terraform"></a>Включение с помощью Terraform

1. Добавьте дополнительный профиль **oms_agent** к существующему [ресурсу azurerm_kubernetes_cluster](https://www.terraform.io/docs/providers/azurerm/d/kubernetes_cluster.html#addon_profile).

   ```
   addon_profile {
    oms_agent {
      enabled                    = true
      log_analytics_workspace_id = "${azurerm_log_analytics_workspace.test.id}"
     }
   }
   ```

2. Добавьте [azurerm_log_analytics_solution](https://www.terraform.io/docs/providers/azurerm/r/log_analytics_solution.html), выполнив шаги, описанные в документации Terraform.

## <a name="enable-from-azure-monitor-in-the-portal"></a>Включение из Azure Monitor на портале

Чтобы включить мониторинг кластера AKS на портале Azure из раздела Azure Monitor, выполните следующие действия:

1. На портале Azure выберите **Монитор**.

2. В списке выберите **Контейнеры**.

3. На странице **мониторинг — контейнеры** выберите **Неотслеживаемые кластеры**.

4. В списке неотслеживаемых кластеров найдите контейнер в списке и нажмите кнопку **включить**.

5. Если у вас есть рабочая область Log Analytics в той же подписке, что и кластер, на странице подключение **к контейнеру "аналитика** " выберите ее из раскрывающегося списка.
    В списке предварительно выбрана рабочая область по умолчанию и расположение, в которое контейнер AKS развертывается в подписке.

    ![Включение мониторинга аналитических сведений о контейнере AKS](./media/container-insights-onboard/kubernetes-onboard-brownfield-01.png)

    >[!NOTE]
    >Если вы хотите создать рабочую область Log Analytics для хранения данных мониторинга из кластера, выполните инструкции, описанные в статье [Создание рабочей области Log Analytics на портале Azure](../logs/quick-create-workspace.md). Создайте рабочую область в той же подписке, в которой развернут контейнер AKS.

После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть метрики работоспособности кластера.

## <a name="enable-directly-from-aks-cluster-in-the-portal"></a>Включение непосредственно из кластера AKS на портале

Чтобы включить мониторинг непосредственно из одного из кластеров AKS в портал Azure, выполните следующие действия.

1. На портале Azure щелкните **Все службы**.

2. В списке ресурсов введите **Контейнеры**.  Как только вы начнете вводить данные, список отфильтруется соответствующим образом.

3. Выберите **Службы Kubernetes**.
    
4. В списке служб Kubernetes выберите службу.

5. На странице Обзор службы Kubernetes выберите **мониторинг-Insights**.

6. Если у вас есть рабочая область Log Analytics в той же подписке, что и кластер, на странице подключение **к контейнеру "аналитика** " выберите ее в раскрывающемся списке.
    В списке предварительно выбрана рабочая область по умолчанию и расположение, в которое контейнер AKS развертывается в подписке.

    ![Включение мониторинга работоспособности контейнера AKS](./media/container-insights-onboard/kubernetes-onboard-brownfield-02.png)

    >[!NOTE]
    >Если вы хотите создать рабочую область Log Analytics для хранения данных мониторинга из кластера, выполните инструкции, описанные в статье [Создание рабочей области Log Analytics на портале Azure](../logs/quick-create-workspace.md). Создайте рабочую область в той же подписке, в которой развернут контейнер AKS.

После включения мониторинга может потребоваться подождать около 15 минут, прежде чем вы сможете просмотреть операционные данные кластера.

## <a name="enable-using-an-azure-resource-manager-template"></a>Включение с помощью шаблона Azure Resource Manager

В этом методе используются два шаблона JSON. Один шаблон задает конфигурацию для включения мониторинга, а другой содержит значения параметров, которые можно настроить, чтобы указать следующее:

* идентификатор ресурса контейнера AKS;
* группу ресурсов, в которой развернут кластер;

>[!NOTE]
>Развертывание шаблона должно проходить в той же группе ресурсов, что и у кластера.
>

Перед включением мониторинга с помощью Azure PowerShell или CLI необходимо создать рабочую область Log Analytics. Для создания рабочей области можно использовать [Azure Resource Manager](../logs/resource-manager-workspace.md), [PowerShell](../logs/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json) или [портал Azure](../logs/quick-create-workspace.md).

Если вы не знакомы с концепцией развертывания ресурсов с помощью шаблона, ознакомьтесь со статьями:

* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)

* [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../../azure-resource-manager/templates/deploy-cli.md)

Если вы решили использовать Azure CLI, необходимо сначала установить интерфейс командной строки и использовать его локально. Необходимо запустить Azure CLI версии 2.0.59 или более поздней. Для определения версии выполните `az --version`. Если вам необходимо установить или обновить Azure CLI, ознакомьтесь со статьей [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

### <a name="create-and-execute-a-template"></a>Создание и выполнение шаблона

1. Скопируйте и вставьте в него следующий синтаксис JSON:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aksResourceId": {
          "type": "string",
          "metadata": {
            "description": "AKS Cluster Resource ID"
          }
        },
        "aksResourceLocation": {
          "type": "string",
          "metadata": {
            "description": "Location of the AKS resource e.g. \"East US\""
          }
        },
        "aksResourceTagValues": {
          "type": "object",
          "metadata": {
            "description": "Existing all tags on AKS Cluster Resource"
          }
        },
        "workspaceResourceId": {
          "type": "string",
          "metadata": {
            "description": "Azure Monitor Log Analytics Resource ID"
          }
        }
      },
      "resources": [
        {
          "name": "[split(parameters('aksResourceId'),'/')[8]]",
          "type": "Microsoft.ContainerService/managedClusters",
          "location": "[parameters('aksResourceLocation')]",
          "tags": "[parameters('aksResourceTagValues')]",
          "apiVersion": "2018-03-31",
          "properties": {
            "mode": "Incremental",
            "id": "[parameters('aksResourceId')]",
            "addonProfiles": {
              "omsagent": {
                "enabled": true,
                "config": {
                  "logAnalyticsWorkspaceResourceID": "[parameters('workspaceResourceId')]"
                }
              }
            }
          }
        }
      ]
    }
    ```

2. Сохраните этот файл как **existingClusterOnboarding.json** в локальную папку.

3. Вставьте в него следующий синтаксис JSON:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "aksResourceId": {
          "value": "/subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroup>/providers/Microsoft.ContainerService/managedClusters/<ResourceName>"
        },
        "aksResourceLocation": {
          "value": "<aksClusterLocation>"
        },
        "workspaceResourceId": {
          "value": "/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroup>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>"
        },
        "aksResourceTagValues": {
          "value": {
            "<existing-tag-name1>": "<existing-tag-value1>",
            "<existing-tag-name2>": "<existing-tag-value2>",
            "<existing-tag-nameN>": "<existing-tag-valueN>"
          }
        }
      }
    }
    ```

4. Измените значения **аксресаурцеид** и **аксресаурцелокатион** , используя значения на странице **Обзор AKS** для кластера AKS. Значение для **workspaceResourceId** — это полный идентификатор ресурса рабочей области Log Analytics, который включает имя рабочей области.

    Измените значения для **аксресаурцетагвалуес** , чтобы они совпадали с существующими значениями тегов, указанными для кластера AKS.

5. Сохраните этот файл как **existingClusterParam.json** в локальную папку.

6. Теперь вы можете развернуть этот шаблон.

   * Чтобы выполнить развертывание с Azure PowerShell, используйте следующие команды в папке, содержащей шаблон:

       ```powershell
       New-AzResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <ResourceGroupName> -TemplateFile .\existingClusterOnboarding.json -TemplateParameterFile .\existingClusterParam.json
       ```

       Изменение конфигурации может занять несколько минут. После завершения настройки конфигурации появится сообщение, похожее на приведенное ниже, с таким результатом:

       ```output
       provisioningState       : Succeeded
       ```

   * Чтобы выполнить развертывание с Azure CLI, выполните следующие команды:

       ```azurecli
       az login
       az account set --subscription "Subscription Name"
       az deployment group create --resource-group <ResourceGroupName> --template-file ./existingClusterOnboarding.json --parameters @./existingClusterParam.json
       ```

       Изменение конфигурации может занять несколько минут. После завершения настройки конфигурации появится сообщение, похожее на приведенное ниже, с таким результатом:

       ```output
       provisioningState       : Succeeded
       ```

       После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть метрики работоспособности кластера.

## <a name="verify-agent-and-solution-deployment"></a>Проверка развертывания агента и решения

В версии агента *06072018* и выше есть возможность удостовериться, что агент и решение были успешно развернуты. В более ранних версиях агента возможна только проверка развертывания агента.

### <a name="agent-version-06072018-or-later"></a>Версия агента 06072018 или более поздняя

Выполните следующую команду, чтобы проверить успешное развертывание агента.

```
kubectl get ds omsagent --namespace=kube-system
```

Результат должен выглядеть приблизительно, как показано ниже, что означает успешное выполнение развертывания:

```output
User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
```

Если в кластере есть узлы Windows Server, можно выполнить следующую команду, чтобы убедиться, что агент успешно развернут.

```
kubectl get ds omsagent-win --namespace=kube-system
```

Результат должен выглядеть приблизительно, как показано ниже, что означает успешное выполнение развертывания:

```output
User@aksuser:~$ kubectl get ds omsagent-win --namespace=kube-system
NAME                   DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                   AGE
omsagent-win           2         2         2         2            2           beta.kubernetes.io/os=windows   1d
```

Чтобы проверить развертывание решения, выполните следующую команду:

```
kubectl get deployment omsagent-rs -n=kube-system
```

Результат должен выглядеть приблизительно, как показано ниже, что означает успешное выполнение развертывания:

```output
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

```output
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

```output
"addonProfiles": {
    "omsagent": {
      "config": {
        "logAnalyticsWorkspaceResourceID": "/subscriptions/<WorkspaceSubscription>/resourceGroups/<DefaultWorkspaceRG>/providers/Microsoft.OperationalInsights/workspaces/<defaultWorkspaceName>"
      },
      "enabled": true
    }
  }
```

## <a name="next-steps"></a>Следующие шаги

* Если при попытке подключить решение у вас возникли проблемы, ознакомьтесь с [руководством по устранению неполадок](container-insights-troubleshoot.md).

* С включенным наблюдением для получения сведений о работоспособности и использовании ресурсов кластером AKS и рабочими нагрузками, которые выполняются на них, Узнайте, [как использовать](container-insights-analyze.md) аналитику контейнера.
