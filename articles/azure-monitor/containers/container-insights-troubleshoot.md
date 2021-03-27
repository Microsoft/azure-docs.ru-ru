---
title: Устранение неполадок в контейнере "аналитика" | Документация Майкрософт
description: В этой статье описывается, как устранять неполадки и устранять проблемы с помощью аналитики контейнера.
ms.topic: conceptual
ms.date: 03/25/2021
ms.openlocfilehash: b7618e9073308da67a8e17c82375a0f05925a542
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105627121"
---
# <a name="troubleshooting-container-insights"></a>Устранение неполадок в аналитике контейнера

При настройке мониторинга кластера Azure Kubernetes Service (AKS) с помощью Container Insights может возникнуть ошибка, препятствующая сбору данных или состоянию отчетов. В этой статье описаны некоторые распространенные проблемы и действия по устранению неполадок.

## <a name="authorization-error-during-onboarding-or-update-operation"></a>Ошибка авторизации при подключении или обновлении

При включении аналитики контейнера или обновлении кластера для поддержки сбора метрик может появиться сообщение об ошибке, похожее на следующее: *клиент <удостоверение пользователя> "с идентификатором объекта" <> ObjectID пользователя "не имеет разрешения на выполнение действия" Microsoft. Authorization/roleAssignments/Write "над областью*

В процессе адаптации или обновления предпринимается ошибка назначения роли **издателя метрик мониторинга** для ресурса кластера. Пользователь, инициирующий процесс включения аналитики контейнера или обновления для поддержки коллекции метрик, должен иметь доступ к разрешению **Microsoft. Authorization/roleAssignments/Write** в области ресурсов кластера AKS. Доступ к этому разрешению предоставляется только членам встроенных ролей " **владелец** " и " **администратор доступа пользователей** ". Если политикам безопасности требуется назначить разрешения детализированного уровня, рекомендуется просмотреть [пользовательские роли](../../role-based-access-control/custom-roles.md) и назначить их пользователям, которым он необходим.

Вы также можете вручную предоставить эту роль из портал Azure, выполнив следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).
2. На портале Azure щелкните **Все службы** в нижнем левом углу. В списке ресурсов введите **Kubernetes**. Как только вы начнете вводить символы, список отфильтруется соответствующим образом. Выберите **Azure Kubernetes**.
3. В списке кластеров Kubernetes выберите один из списка.
2. В меню слева щелкните **Управление доступом (IAM)**.
3. Выберите **+ Добавить** , чтобы добавить назначение роли и выберите роль **издателя метрики мониторинга** и в поле **выбрать** введите **AKS** , чтобы отфильтровать результаты по только тем субъектам службы кластера, которые определены в подписке. Выберите один из списка, относящегося к этому кластеру.
4. Выберите **Сохранить**, чтобы завершить назначение роли.

## <a name="container-insights-is-enabled-but-not-reporting-any-information"></a>Служба "аналитика контейнеров" включена, но не сообщает о каких-либо сведениях

Если аналитика контейнера успешно включена и настроена, но вы не можете просматривать сведения о состоянии или нет результатов, возвращаемых запросом журнала, вы можете диагностировать проблему, выполнив следующие действия.

1. Проверьте состояние агента, выполнив эту команду:

    `kubectl get ds omsagent --namespace=kube-system`

    Результат должен выглядеть, как показано в примере ниже, что означает успешное выполнение развертывания:

    ```
    User@aksuser:~$ kubectl get ds omsagent --namespace=kube-system
    NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
    omsagent   2         2         2         2            2           beta.kubernetes.io/os=linux   1d
    ```
2. Если у вас есть узлы Windows Server, проверьте состояние агента, выполнив команду:

    `kubectl get ds omsagent-win --namespace=kube-system`

    Результат должен выглядеть, как показано в примере ниже, что означает успешное выполнение развертывания:

    ```
    User@aksuser:~$ kubectl get ds omsagent-win --namespace=kube-system
    NAME                   DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR                   AGE
    omsagent-win           2         2         2         2            2           beta.kubernetes.io/os=windows   1d
    ```
3. Проверьте состояние развертывания для агента версии *06072018* или более поздней версии с помощью этой команды:

    `kubectl get deployment omsagent-rs -n=kube-system`

    Результат должен выглядеть, как показано в примере ниже, что означает успешное выполнение развертывания:

    ```
    User@aksuser:~$ kubectl get deployment omsagent-rs -n=kube-system
    NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
    omsagent   1         1         1            1            3h
    ```

4. Проверьте состояние группы pod, чтобы убедиться в ее работоспособности, выполнив эту команду: `kubectl get pods --namespace=kube-system`.

    Выходные данные должны выглядеть, как в примере ниже, и содержать состояние omsagent *Running* (Выполняется).

    ```
    User@aksuser:~$ kubectl get pods --namespace=kube-system
    NAME                                READY     STATUS    RESTARTS   AGE
    aks-ssh-139866255-5n7k5             1/1       Running   0          8d
    azure-vote-back-4149398501-7skz0    1/1       Running   0          22d
    azure-vote-front-3826909965-30n62   1/1       Running   0          22d
    omsagent-484hw                      1/1       Running   0          1d
    omsagent-fkq7g                      1/1       Running   0          1d
    omsagent-win-6drwq                  1/1       Running   0          1d
    ```

## <a name="error-messages"></a>Сообщения об ошибках

В следующей таблице перечислены известные ошибки, которые могут возникнуть при использовании контейнера аналитики.

| Сообщения об ошибках  | Действие |
| ---- | --- |
| Сообщение об ошибке `No data for selected filters`  | Потребуется некоторое время, чтобы установить мониторинг потока данных для только что созданных кластеров. Чтобы данные отображались в кластере, подождите не менее 10 – 15 минут. |
| Сообщение об ошибке `Error retrieving data` | Хотя кластер службы Kubernetes Azure настраивается для мониторинга работоспособности и производительности, между кластером и рабочей областью Azure Log Analytics устанавливается подключение. Рабочая область Log Analytics используется для хранения всех данных мониторинга для кластера. Эта ошибка может возникать, если Рабочая область Log Analytics удалена. Проверьте, удалена ли Рабочая область и если она была, необходимо снова включить мониторинг кластера с помощью Container Insights и указать существующую или создать новую рабочую область. Для повторного включения необходимо [Отключить](container-insights-optout.md) мониторинг кластера и снова [включить](container-insights-enable-new-cluster.md) аналитику контейнера. |
| `Error retrieving data` После добавления аналитики контейнера с помощью команды AZ AKS CLI | При включении мониторинга с помощью `az aks cli` может быть неправильно развернуто контейнерная аналитика. Проверьте, развернуто ли решение. Чтобы проверить, перейдите в рабочую область Log Analytics и проверьте, доступно ли решение, выбрав **решения** на панели слева. Чтобы устранить эту проблему, необходимо повторно развернуть решение, следуя инструкциям в разделе [развертывание аналитики контейнера](container-insights-onboard.md) . |

Чтобы помочь в диагностике проблемы, мы предоставили [сценарий устранения неполадок](https://github.com/microsoft/Docker-Provider/tree/ci_dev/scripts/troubleshoot).

## <a name="container-insights-agent-replicaset-pods-are-not-scheduled-on-non-azure-kubernetes-cluster"></a>В кластере Azure Kubernetes не запланированы модули контейнеров для реплики контейнера аналитики.

Контейнеры Pod для агента аналитики контейнеров имеют зависимость от следующих селекторов узлов на узлах рабочей роли (или агента) для планирования.

```
nodeSelector:
  beta.kubernetes.io/os: Linux
  kubernetes.io/role: agent
```

Если к рабочим узлам не подключены Метки узлов, то планирование модулей для реплики агентов не будет запланировано. Инструкции по присоединению метки см. в разделе [Назначение меток Kubernetes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/) .

## <a name="performance-charts-dont-show-cpu-or-memory-of-nodes-and-containers-on-a-non-azure-cluster"></a>Диаграммы производительности не отображают ЦП или память узлов и контейнеров в кластере, отличном от Azure

Для сбора метрик производительности в модулях Pod агента аналитики контейнеров используется конечная точка cAdvisor на агенте узла. Убедитесь, что контейнерный агент на узле настроен так, чтобы его `cAdvisor port: 10255` можно было открывать на всех узлах в кластере для сбора метрик производительности.

## <a name="non-azure-kubernetes-cluster-are-not-showing-in-container-insights"></a>Кластер Kubernetes, не связанный с Azure, не отображается в службе "аналитика контейнера"

Чтобы просмотреть кластер Kubernetes, не относящийся к Azure, в службе "аналитика контейнера" требуется доступ на чтение в рабочей области Log Analytics, поддерживающей эту аналитику, и в ресурсе Resource Insights **контаинеринсигхтс (*Рабочая область*)**.

## <a name="metrics-arent-being-collected"></a>Метрики не собираются

1. Убедитесь, что кластер находится в [поддерживаемом регионе для пользовательских метрик](../essentials/metrics-custom-overview.md#supported-regions).

2. Убедитесь, что назначение роли **издателя метрик мониторинга** существует с помощью следующей команды CLI:

    ``` azurecli
    az role assignment list --assignee "SP/UserassignedMSI for omsagent" --scope "/subscriptions/<subid>/resourcegroups/<RG>/providers/Microsoft.ContainerService/managedClusters/<clustername>" --role "Monitoring Metrics Publisher"
    ```
    Для кластеров с MSI назначенный пользователю идентификатор клиента для omsagent меняется каждый раз, когда наблюдение включено и отключено, поэтому назначение ролей должно существовать в текущем идентификаторе клиента MSI. 

3. Для кластеров с включенным удостоверением Azure Active Directory Pod и использованием MSI:

   - Проверьте требуемую метку **kubernetes.Azure.com/ManagedBy: AKS**  в модулях Pod omsagent с помощью следующей команды:

        `kubectl get pods --show-labels -n kube-system | grep omsagent`

    - Убедитесь, что исключения включены при включении удостоверения Pod с помощью одного из поддерживаемых методов в https://github.com/Azure/aad-pod-identity#1-deploy-aad-pod-identity .

        Выполните следующую команду, чтобы проверить следующее:

        `kubectl get AzurePodIdentityException -A -o yaml`

        Должны появиться выходные данные, аналогичные приведенным ниже.

        ```
        apiVersion: "aadpodidentity.k8s.io/v1"
        kind: AzurePodIdentityException
        metadata:
        name: mic-exception
        namespace: default
        spec:
        podLabels:
        app: mic
        component: mic
        ---
        apiVersion: "aadpodidentity.k8s.io/v1"
        kind: AzurePodIdentityException
        metadata:
        name: aks-addon-exception
        namespace: kube-system
        spec:
        podLabels:
        kubernetes.azure.com/managedby: aks
        ```



## <a name="next-steps"></a>Дальнейшие шаги

Если включен мониторинг для сбора метрик работоспособности узлов и групп pod кластера AKS, эти метрики будут доступными на портале Azure. Сведения о том, как использовать аналитику контейнеров, см. в статье [Просмотр работоспособности службы Azure Kubernetes](container-insights-analyze.md).
