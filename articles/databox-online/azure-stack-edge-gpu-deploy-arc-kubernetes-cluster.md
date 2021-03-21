---
title: Включение дуги Azure на Kubernetes на устройстве Azure Stack ребра Pro GPU | Документация Майкрософт
description: В этой статье описывается, как включить дугу Azure в существующем кластере Kubernetes на устройстве с Azure Stack ребра Pro GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/05/2021
ms.author: alkohli
ms.openlocfilehash: 4d75986880075f6eb07aa31b9322bdae15535802
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102437610"
---
# <a name="enable-azure-arc-on-kubernetes-cluster-on-your-azure-stack-edge-pro-gpu-device"></a>Включение службы "Дуга Azure" в кластере Kubernetes на устройстве с Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье показано, как включить дугу Azure в существующем кластере Kubernetes на устройстве с Azure Stack погранично Pro. 

Эта процедура предназначена для тех, кто ознакомился с [Kubernetes рабочими нагрузками на устройстве на Azure Stack пограничной Pro](azure-stack-edge-gpu-kubernetes-workload-management.md) и знаком с концепциями того, [что такое служба "Дуга Azure Kubernetes (Предварительная версия)"](../azure-arc/kubernetes/overview.md).


## <a name="prerequisites"></a>Предварительные условия

Прежде чем включать службу "Дуга Azure" в кластере Kubernetes, убедитесь, что выполнены следующие предварительные требования на устройстве Azure Stack пограничной Pro и на клиенте, который будет использоваться для доступа к устройству:

### <a name="for-device"></a>Для устройств

1. У вас есть учетные данные для входа на 1 узел Azure Stack пограничным устройством Pro.
    1. Устройство активировано. См. раздел [Активация устройства](azure-stack-edge-gpu-deploy-activate.md).
    1. Устройство имеет роль вычислений, настроенную через портал Azure и имеющую кластер Kubernetes. См. раздел [Настройка вычислений](azure-stack-edge-gpu-deploy-configure-compute.md).

1. Вы владеете доступом к подписке. Этот доступ потребуется на этапе назначения роли для субъекта-службы.
 

### <a name="for-client-accessing-the-device"></a>Для клиента, обращающегося к устройству

1. У вас есть клиентская система Windows, которая будет использоваться для доступа к устройству Azure Stack погранично Pro.
  
    - Клиент работает под управлением Windows PowerShell 5,0 или более поздней версии. Чтобы скачать последнюю версию Windows PowerShell, перейдите к разделу [Установка Windows PowerShell](/powershell/scripting/install/installing-powershell-core-on-windows).
    
    - Также можно использовать любой другой клиент с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device) . В этой статье описывается процедура использования клиента Windows. 
    
1. Вы выполнили процедуру, описанную в статье [доступ к кластеру Kubernetes на устройстве с Azure Stack ребр Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Вы выполнили следующие задачи:
    
    - Установлен `kubectl` на клиенте.    
    - Убедитесь, что `kubectl` версия клиента отклонена, но не имеет более одной версии из главной версии Kubernetes, работающей на устройстве Azure Stack погранично Pro. 
      - Используйте `kubectl version` для проверки версии kubectl, работающей на клиенте. Запишите полную версию.
      - В локальном пользовательском интерфейсе устройства Azure Stack ребра Pro перейдите в раздел **Обновление программного обеспечения** и запишите номер версии сервера Kubernetes. 
    
        ![Проверка номера версии сервера Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/verify-kubernetes-version-1.png)      
      
      - Убедитесь, что эти две версии совместимы. 


## <a name="register-kubernetes-resource-providers"></a>Регистрация поставщиков ресурсов Kubernetes

Перед включением дуги Azure в кластере Kubernetes необходимо включить и зарегистрировать `Microsoft.Kubernetes` `Microsoft.KubernetesConfiguration` подписку. 

1. Чтобы включить поставщик ресурсов, в портал Azure перейдите к подписке, которую вы планируете использовать для развертывания. Перейдите к **поставщику ресурсов**. 
1. На правой панели найдите поставщиков, которых нужно добавить. В этом примере — `Microsoft.Kubernetes` и `Microsoft.KubernetesConfiguration` .

    ![Регистрация поставщиков ресурсов Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-1.png)

1. Выберите поставщика ресурсов и в верхней части панели команд выберите **зарегистрировать**. Регистрация занимает несколько минут. 

    ![Регистрация поставщиков ресурсов Kubernetes 2](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-2.png)

1. Обновите пользовательский интерфейс, пока не появится регистрация поставщика ресурсов. Повторите эту процедуру для обоих поставщиков ресурсов.
    
    ![Регистрация поставщиков ресурсов Kubernetes 3](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-4.png)

Вы также можете зарегистрировать поставщики ресурсов с помощью `az cli` . Дополнительные сведения см. [в статье регистрация двух поставщиков для Kubernetes с поддержкой ARC в Azure](../azure-arc/kubernetes/connect-cluster.md#register-the-two-providers-for-azure-arc-enabled-kubernetes) .

## <a name="create-service-principal-assign-role"></a>Создание субъекта-службы, назначение роли

1. Убедитесь, что у вас есть `Subscription ID` имя группы ресурсов, которую вы использовали для развертывания ресурсов для службы Azure Stack ребра. Чтобы получить идентификатор подписки, перейдите к ресурсу Azure Stack ребра в портал Azure. Перейдите к **обзору > Essentials**.

    ![Получение идентификатора подписки](media/azure-stack-edge-gpu-connect-powershell-interface/get-subscription-id-1.png)

    Чтобы получить имя группы ресурсов, перейдите в раздел **Свойства**.

    ![Получить имя группы ресурсов](media/azure-stack-edge-gpu-connect-powershell-interface/get-resource-group-name-1.png)

1. Чтобы создать субъект-службу, используйте следующую команду через `az cli` .

    `az ad sp create-for-rbac --skip-assignment --name "<Informative name for service principal>"`  

    Сведения о том, как войти в `az cli` , [запустите Cloud Shell в портал Azure](../cloud-shell/quickstart-powershell.md#start-cloud-shell)

    Ниже приведен пример. 
    
    ```azurecli
    PS /home/user> az ad sp create-for-rbac --skip-assignment --name "https://azure-arc-for-ase-k8s"
    {
      "appId": "aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b",
      "displayName": "azure-arc-for-ase-k8s",
      "name": "https://azure-arc-for-ase-k8s",
      "password": "<password>",
      "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
    }
    PS /home/user>
    ```

1. Запишите `appID` , `name` , `password` и, `tenantID` как это будет использоваться в качестве входных данных в следующей команде.

1. После создания нового субъекта-службы назначьте `Kubernetes Cluster - Azure Arc Onboarding` роль только что созданному участнику. Это встроенная роль Azure (используйте идентификатор роли в команде) с ограниченными разрешениями. Используйте следующую команду:

    `az role assignment create --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 --assignee <appId-from-service-principal> --scope /subscriptions/<SubscriptionID>/resourceGroups/<Resource-group-name>`

    Ниже приведен пример.
    
    ```azurecli
    PS /home/user> az role assignment create --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 --assignee aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b --scope /subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1
    {
      "canDelegate": null,
      "id": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1/providers/Microsoft.Authorization/roleAssignments/59272f92-e5ce-4aeb-9c0c-62532d8caf25",
      "name": "59272f92-e5ce-4aeb-9c0c-62532d8caf25",
      "principalId": "b045b3fe-8745-4097-9674-91cb0afaad91",
      "principalType": "ServicePrincipal",
      "resourceGroup": "myaserg1",
      "roleDefinitionId": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/providers/Microsoft.Authorization/roleDefinitions/34e09817-6cbe-4d01-b1a2-e0eac5743d41",
      "scope": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    PS /home/user>
    ```
    Дополнительные сведения о создании субъекта-службы и выполнении назначения ролей см. в статье [Создание субъекта-службы адаптации с поддержкой ARC в Azure](../azure-arc/kubernetes/create-onboarding-service-principal.md).


## <a name="enable-arc-on-kubernetes-cluster"></a>Включение Arc в кластере Kubernetes

Выполните следующие действия, чтобы настроить кластер Kubernetes для управления службой "Дуга Azure".

1. [Подключитесь к интерфейсу PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) устройства.

1. Тип:

    `Set-HcsKubernetesAzureArcAgent -SubscriptionId "<Your Azure Subscription Id>" -ResourceGroupName "<Resource Group Name>" -ResourceName "<Azure Arc resource name (shouldn't exist already)>" -Location "<Region associated with resource group>" -TenantId "<Tenant Id of service principal>" -ClientId "<App id of service principal>" -ClientSecret "<Password of service principal>"`


    > [!NOTE]
    > - Чтобы развернуть дугу Azure на устройстве, убедитесь, что вы используете [поддерживаемый регион для дуги Azure](../azure-arc/kubernetes/overview.md#supported-regions). 
    > - Используйте `az account list-locations` команду, чтобы выяснить точное имя расположения для передачи в `Set-HcsKubernetesAzureArcAgent` командлет. Имена расположений обычно форматируются без пробелов.
    
    Пример:
   
    ```powershell
    [10.128.44.240]: PS>Set-HcsKubernetesAzureArcAgent -SubscriptionId "062c67a6-019b-40af-a775-c4dc1abe56ed" -ResourceGroupName "myaserg1" -ResourceName "myasetestresarc" -Location "westeurope" -TenantId "72f988bf-86f1-41af-91ab-2d7cd011db47" -ClientId "aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b" -ClientSecret "<password>"
        [10.128.44.240]: PS>
    ```
    
    В портал Azure ресурс должен быть создан с именем, указанным в предыдущей команде.

    ![Переход к ресурсу Arc Azure](media/azure-stack-edge-gpu-connect-powershell-interface/verify-azure-arc-enabled-1.png)

1. Чтобы убедиться, что служба "Дуга Azure" включена успешно, выполните следующую команду в интерфейсе PowerShell:

    `kubectl get deployments -n azure-arc`

    Эта команда находит все приложения, развернутые в `azure-arc` пространстве имен, соответствующем службе "Дуга Azure".

    Ниже приведен пример выходных данных, в которых показаны агенты ARC для Azure, которые были развернуты в кластере Kubernetes в `azure-arc` пространстве имен. 


    ```powershell
    [10.128.44.240]: PS>kubectl get deployments -n azure-arc
    NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
    cluster-metadata-operator   1/1     1            1           45m
    clusteridentityoperator     1/1     1            1           45m
    config-agent                1/1     1            1           45m
    connect-agent               1/1     1            1           45m
    controller-manager          1/1     1            1           45m
    flux-logs-agent             1/1     1            1           45m
    metrics-agent               1/1     1            1           45m
    resource-sync-agent         1/1     1            1           45m
    [10.128.44.240]: PS>
    ```

    Вы также можете получить список модулей Pod, выполняющихся в кластере Kubernetes `azure-arc` , в пространстве имен. Pod — это контейнер приложения или процесс, выполняющийся в кластере Kubernetes. 

    Используйте следующую команду:
    
    `kubectl get pods -n azure-arc`
    
    Ниже приведен пример выходных данных.
    
    ```powershell
    [10.128.44.240]: PS>kubectl get pods -n azure-arc
    NAME                                         READY   STATUS    RESTARTS   AGE
    cluster-metadata-operator-64cbdf95b4-s2q52   2/2     Running   0          16m
    clusteridentityoperator-6f6dbccf7-nwnxg      3/3     Running   0          16m
    config-agent-7df5bf497b-mjm8k                3/3     Running   0          16m
    connect-agent-5d4c766764-m7h46               1/1     Running   0          16m
    controller-manager-777555fb57-t7tdp          3/3     Running   0          16m
    flux-logs-agent-845476c899-zcmtj             2/2     Running   0          16m
    metrics-agent-84d6fc8f4d-g9jkm               2/2     Running   0          16m
    resource-sync-agent-8f88dbf96-zgxjj          3/3     Running   0          16m
    [10.128.44.240]: PS>
    ```


Как видно из предыдущих выходных данных, Kubernetes с поддержкой дуги Azure состоит из нескольких агентов (операторов), которые выполняются в кластере, развернутом в `azure-arc` пространстве имен.

- `config-agent` наблюдает за подключенным кластером и ищет ресурсы конфигурации системы управления версиями, примененные к этому кластеру, а также обновляет состояние соответствия.
- `controller-manager` является оператором для операторов, то есть управляет взаимодействием между компонентами Azure Arc.
- `metrics-agent` собирает метрики от других агентов Azure Arc, чтобы обеспечить оптимальную производительность этих агентов.
- `cluster-metadata-operator`: собирает метаданные кластера — версию кластера, число узлов и версию агента ARC для Azure.
- `resource-sync-agent` синхронизирует упомянутые выше метаданные кластера с Azure.
- `clusteridentityoperator`: Служба "Дуга Azure" Kubernetes сейчас поддерживает назначенное системой удостоверение. клустеридентитйоператор хранит сертификат управляемого удостоверения службы (MSI), используемый другими агентами для взаимодействия с Azure.
- `flux-logs-agent`— собирает журналы из операторов Flux, развернутых в составе конфигурации системы управления версиями.
- `connect-agent`: речь идет о ресурсе Arc Azure.

### <a name="remove-arc-from-the-kubernetes-cluster"></a>Удаление дуги из кластера Kubernetes

Чтобы удалить Управление службой "Дуга Azure", выполните следующие действия.

1. 1. [Подключитесь к интерфейсу PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) устройства.
2. Тип:

    `Remove-HcsKubernetesAzureArcAgent` 


> [!NOTE]
> По умолчанию, когда ресурс `yamls` удаляется из репозитория Git, соответствующие ресурсы не удаляются из кластера Kubernetes. Необходимо задать `--sync-garbage-collection`  в Arc операторпарамс, чтобы разрешить удаление ресурсов при удалении из репозитория Git. Дополнительные сведения см. [в разделе Удаление конфигурации](../azure-arc/kubernetes/use-gitops-connected-cluster.md#additional-parameters) .

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как запустить развертывание Azure Arc, см. в статье [развертывание приложения PHP без отслеживания состояния `Guestbook` с помощью Redis через гитопс на устройстве Pro на Azure Stack](azure-stack-edge-gpu-deploy-stateless-application-git-ops-guestbook.md) .
