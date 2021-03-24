---
title: Перемещение ресурсов между регионами с помощью PowerShell в Azure Resource Mover
description: Узнайте, как перемещать ресурсы между регионами с помощью PowerShell в Azure Resource Mover.
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: tutorial
ms.date: 02/21/2021
ms.author: raynew
ms.openlocfilehash: b728170521570ff0d83b67671109e82adb7fa7b0
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102583351"
---
# <a name="move-resources-across-regions-in-powershell"></a>Перемещение ресурсов между регионами в PowerShell

Из этой статьи вы узнаете, как перемещать ресурсы Azure в другой регион с помощью PowerShell в [Azure Resource Mover](overview.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Проверка предварительных требований.
> * Настройка коллекции перемещения.
> * Добавление ресурсов в коллекцию перемещения и разрешение зависимостей.
> * Подготовка и перемещение исходной группы ресурсов. 
> * Подготовка и перемещение остальных ресурсов.
> * Отмена и фиксация перемещения. 
> * Удаление ресурсов в исходном регионе после перемещения (при необходимости).

> [!NOTE]
> В учебниках показан самый быстрый способ выполнения сценария и используются параметры по умолчанию. 

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/pricing/free-trial/), прежде чем начинать работу. Затем войдите на [портал Azure](https://portal.azure.com).

## <a name="prerequisites"></a>Предварительные требования
**Требование** | **Описание**
--- | ---
**Права доступа к подпискам** | Убедитесь, что у вас есть доступ *владельца* в подписке, содержащей ресурсы, которые вы хотите переместить<br/><br/> **Зачем мне нужен доступ владельца?** При первом добавлении ресурса для определенной пары источника и назначения в подписке Azure Resource Mover создает [управляемое удостоверение, назначаемое системой](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (ранее известное как управляемая службой идентификация (MSI)), которая является доверенной для подписки. Чтобы создать удостоверение и назначить ему требуемую роль (участника или администратора доступа пользователя в исходной подписке), учетной записи, используемой для добавления ресурсов, требуются разрешения *владельца* в подписке. [Дополнительные сведения](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) о ролях Azure.
**Поддержка Resource Mover** | [Ознакомьтесь](common-questions.md) с поддерживаемыми регионами и другими распространенными вопросами.
**Поддержка виртуальных машин** |  Убедитесь, что виртуальные машины, которые требуется переместить, поддерживаются.<br/><br/> - [Убедитесь](support-matrix-move-region-azure-vm.md#windows-vm-support) в наличии поддерживаемых виртуальных машин Windows.<br/><br/> - [Убедитесь](support-matrix-move-region-azure-vm.md#linux-vm-support) в наличии поддерживаемых виртуальных машин Linux и версий ядра.<br/><br/> - Убедитесь, что параметры [вычислений](support-matrix-move-region-azure-vm.md#supported-vm-compute-settings), [хранилища](support-matrix-move-region-azure-vm.md#supported-vm-storage-settings) и [сети](support-matrix-move-region-azure-vm.md#supported-vm-networking-settings) являются поддерживаемыми.
**Поддержка SQL** | Если вы хотите переместить ресурсы SQL, ознакомьтесь со [списком требований SQL](tutorial-move-region-sql.md#check-sql-requirements).
**Целевая подписка** | Для подписки в целевом регионе требуется достаточный объем квот, чтобы создать ресурсы, перемещаемые в целевой регион. Если квота отсутствует, [запросите дополнительные ресурсы](../azure-resource-manager/management/azure-subscription-service-limits.md).
**Расходы на целевой регион** | Проверьте цены в целевом регионе, в который вы перемещаете виртуальные машины. Оцените затраты с помощью [калькулятора цен](https://azure.microsoft.com/pricing/calculator/).

### <a name="review-powershell-requirements"></a>Проверка требований PowerShell

Большинство операций перемещения ресурсов одинаковы, независимо от того, используете ли вы портал Azure или PowerShell, с несколькими исключениями.

**Операция** | **PowerShell** | **Портал**
--- | --- | ---
**Создание коллекции перемещения** | Коллекция перемещения (список всех перемещаемых ресурсов) создается автоматически. Необходимые разрешения удостоверений назначаются порталом в серверной части. | Командлеты PowerShell используются в таких случаях:<br/><br/> — создание группы ресурсов для перемещения коллекции и указание ее расположения;<br/><br/> — назначение управляемого удостоверения коллекции;<br/><br/> — добавление ресурсов в коллекцию.
**Удаление коллекции перемещения** | Невозможно удалить коллекцию перемещения непосредственно на портале. | Для удаления коллекции перемещения используется командлет PowerShell.
**Операции перемещения ресурсов**<br/><br/> (Подготовка, начало перемещения, фиксация и т. д.)| Отдельные шаги с автоматической проверкой в Resource Mover. | Командлеты Powershell используются в таких случаях:<br/><br/> 1. Проверка зависимостей.<br/><br/> 2. Выполнение перемещения.
**Удаление исходных ресурсов** | Непосредственно на портале Resource Mover. | Командлеты PowerShell на уровне ресурсов.


### <a name="sample-values"></a>Примеры значений

Мы используем эти значения в наших примерах сценария:

**Параметр** | **Значение** 
--- | ---
Идентификатор подписки | subscription-id
Исходный регион |  Центральная часть США
Целевой регион | центрально-западная часть США
Группа ресурсов (в которой хранятся метаданные для коллекции перемещения) | RG-MoveCollection-demoRMS
Имя коллекции перемещения | PS-centralus-westcentralus-demoRMS
Группа ресурсов (исходный регион) | PSDemoRM 
Группа ресурсов (целевой регион) | PSDemoRM-target
Расположение службы перемещения ресурсов | восточная часть США 2
IdentityType | SystemAssigned
Виртуальная машина для перемещения | PSDemoVM


## <a name="sign-into-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью командлета Connect-AzAccount.

```azurepowershell-interactive
Connect-AzAccount – Subscription "<subscription-id>"
```

## <a name="set-up-the-move-collection"></a>Настройка коллекции перемещения

Объект MoveCollection содержит метаданные и сведения о конфигурации ресурсов, которые требуется переместить. Чтобы настроить коллекцию перемещения, сделайте следующее:

- Создайте группу ресурсов для коллекции перемещения.
- Зарегистрируйте поставщик службы в подписке, чтобы можно было создать ресурс MoveCollection.
- Создайте объект MoveCollection с управляемым удостоверением. Для доступа к подписке, в которой находится служба Resource Mover, объекту MoveCollection требуется [управляемое удостоверение, назначаемое системой](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) (прежнее название — управляемое удостоверение службы (MSI)), которое является доверенным для подписки.
- Предоставьте доступ к подписке Resource Mover для управляемого удостоверения.

### <a name="create-the-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов для метаданных и сведений о конфигурации перемещения коллекции следующим образом:

```azurepowershell-interactive
New-AzResourceGroup -Name "RG-MoveCollection-demoRMS" -Location "East US 2"
```
**Выходные данные**:

![Выходной текст после создания группы ресурсов](./media/tutorial-move-region-powershell/create-metadata-resource-group.png)

### <a name="register-the-resource-provider"></a>Регистрация поставщика ресурсов

1. Зарегистрируйте поставщик ресурсов Microsoft.Migrate, чтобы можно было создать ресурс MoveCollection, следующим образом:

    ```azurepowershell-interactive
    Register-AzResourceProvider -ProviderNamespace Microsoft.Migrate
    
2. Wait for registration:

    ```azurepowershell-interactive 
    While(((Get-AzResourceProvider -ProviderNamespace Microsoft.Migrate)| where {$_.RegistrationState -eq "Registered" -and $_.ResourceTypes.ResourceTypeName -eq "moveCollections"}|measure).Count -eq 0)
    {
        Start-Sleep -Seconds 5
        Write-Output "Waiting for registration to complete."
    }
    ```
### <a name="create-a-movecollection-object"></a>Создание объекта MoveCollection

Создайте объект MoveCollection и назначьте ему управляемое удостоверение, как показано ниже. 

```azurepowershell-interactive
New-AzResourceMoverMoveCollection -Name "PS-centralus-westcentralus-demoRMS"  -ResourceGroupName "RG-MoveCollection-demoRMS" -SourceRegion "centralus" -TargetRegion "westcentralus" -Location "centraluseuap" -IdentityType "SystemAssigned"
```

**Выходные данные**:

![Выходной текст после создания коллекции перемещения](./media/tutorial-move-region-powershell/output-move-collection.png)


### <a name="grant-access-to-the-managed-identity"></a>Предоставление доступа управляемому удостоверению

Предоставьте управляемому удостоверению доступ к подписке Resource Mover следующим образом. Вы должны быть владельцем подписки.

1. Получите сведения об удостоверении из объекта MoveCollection.

    ```azurepowershell-interactive
    $moveCollection = Get-AzResourceMoverMoveCollection -SubscriptionId $subscriptionId -ResourceGroupName "RG-MoveCollection-demoRMS" -Name "PS-centralus-westcentralus-demoRMS"

    $identityPrincipalId = $moveCollection.IdentityPrincipalId   
    ``` 

2. Назначьте удостоверению необходимые роли, чтобы служба Azure Resource Mover могла получить доступ к вашей подписке и упростить перемещение ресурсов.

    ```azurepowershell-interactive
    New-AzRoleAssignment -ObjectId $identityPrincipalId -RoleDefinitionName Contributor -Scope "/subscriptions/$subscriptionId"

    New-AzRoleAssignment -ObjectId $identityPrincipalId -RoleDefinitionName "User Access Administrator" -Scope "/subscriptions/$subscriptionId"
    ``` 

## <a name="add-resources-to-the-move-collection"></a>Добавление ресурсов в коллекцию перемещения

Получите идентификаторы для имеющихся исходных ресурсов, которые требуется переместить. Создайте объект параметров целевого ресурса, а затем добавьте ресурсы в коллекцию перемещения.

> [!NOTE]
> Ресурсы, добавленные в коллекцию перемещения, должны относиться к одной подписке, но могут находиться в разных группах ресурсов.

Добавьте ресурсы следующим образом:

1. Получите идентификатор исходного ресурса:

    ```azurepowershell-interactive
    Get-AzResource -Name PSDemoVM -ResourceGroupName PSDemoRM
    ```

    **Выходные данные**

    ![Выходной текст после получения идентификатора ресурса](./media/tutorial-move-region-powershell/output-retrieve-resource.png)

2. Создайте целевой объект параметров ресурсов в соответствии с перемещаемым ресурсом. В нашем случае это виртуальная машина.

    ```azurepowershell-interactive
    $targetResourceSettingsObj = New-Object Microsoft.Azure.PowerShell.Cmdlets.ResourceMover.Models.Api202101.VirtualMachineResourceSettings
    ```

3. Задайте тип ресурса и имя целевого ресурса для объекта.

    ```azurepowershell-interactive
    $targetResourceSettingsObj.ResourceType = "Microsoft.Compute/virtualMachines"
    $targetResourceSettingsObj.TargetResourceName = "PSDemoVM"
    ```
    > [!NOTE]
    > Целевая виртуальная машина имеет то же имя, что и виртуальная машина в исходном регионе. Вы можете выбрать другой регион.

4. Добавьте исходные ресурсы в коллекцию перемещения с помощью полученного или созданного объекта идентификатора ресурса и целевых параметров.

    ```azurepowershell-interactive
    Add-AzResourceMoverMoveResource -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -SourceId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx xxxxxxxxxxxx/resourceGroups/
    PSDemoRM/providers/Microsoft.Compute/virtualMachines/PSDemoVM" -Name "PSDemoVM" -ResourceSetting $targetResourceSettingsObj
    ```

    **Выходные данные** ![Выходной текст после добавления ресурса](./media/tutorial-move-region-powershell/output-add-resource.png)

## <a name="validate-and-add-dependencies"></a>Проверка и добавление зависимостей

Проверьте, имеют ли добавленные ресурсы зависимости от других ресурсов, и добавьте их при необходимости. 

1. Проверьте зависимости следующим образом:

    ```azurepowershell-interactive
    Resolve-AzResourceMoverMoveCollectionDependency -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"
    ```
    **Выходные данные (при наличии зависимостей)**

    ![Выходной текст после проверки зависимостей](./media/tutorial-move-region-powershell/dependency-output.png)

2. Удостоверение без зависимостей:

    - Чтобы получить список всех недостающих зависимостей, сделайте следующее:

        ```azurepowershell-interactive
        Get-AzResourceMoverUnresolvedDependency -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -ResourceGroupName "RG-MoveCollection-demoRMS" -DependencyLevel Descendant"
        ```
        **Выходные данные** ![Выходной текст после получения списка всех зависимостей](./media/tutorial-move-region-powershell/dependencies-list.png)  

    - Чтобы получить только зависимости первого уровня (прямые зависимости для ресурса), сделайте следующее:

        ```azurepowershell-interactive
        Get-AzResourceMoverUnresolvedDependency -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -ResourceGroupName "RG-MoveCollection-demoRMS" -DependencyLevel Direct
        ```
        **Выходные данные** ![Выходной текст после получения списка зависимостей первого уровня](./media/tutorial-move-region-powershell/dependencies-list-direct.png)  

3. Чтобы добавить все необработанные недостающие зависимости, повторите приведенные выше инструкции для [добавления ресурсов в коллекцию перемещения](#add-resources-to-the-move-collection) и выполните повторную проверку, пока необработанных ресурсов не останется.

> [!NOTE]
> Если по какой-либо причине требуется удалить ресурсы из коллекции, следуйте инструкциям, приведенным в [этой статье](remove-move-resources.md).

## <a name="add-the-source-resource-group"></a>Добавление исходной группы ресурсов

Добавьте исходную группу, содержащую ресурсы, которые необходимо переместить в коллекцию перемещения.

1. Получите идентификатор группы ресурсов.

    ```azurepowershell-interactive
    Get-AzResourceMoverUnresolvedDependency -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -ResourceGroupName "RG-MoveCollection-demoRMS" -DependencyLevel Direct
    ```
    **Выходные данные** ![Выходной текст после получения идентификатора исходной группы ресурсов](./media/tutorial-move-region-powershell/source-resource-group-id.png)  

    > [!NOTE]
    > Мы используем группу ресурсов, которая уже находится в целевом регионе.

 
2. Используйте полученный идентификатор, чтобы добавить группу ресурсов в коллекцию.

    ```azurepowershell-interactive
    Add-AzResourceMoverMoveResource -ResourceGroupName "RG-MoveCollection-demoRMS"  -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -SourceId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/psdemorm"  -Name "psdemorm"  -ExistingTargetId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/PSDemoRM-target"
    ```
    **Выходные данные** ![Выходной текст после добавления исходной группы ресурсов в коллекцию перемещения](./media/tutorial-move-region-powershell/add-source-resource-group.png)

3. Проверьте зависимости, чтобы убедиться, что после добавления группы ресурсов вы ничего не пропустили.

    ```azurepowershell-interactive
    Resolve-AzResourceMoverMoveCollectionDependency -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"
    ```
4. Мы видим, что здесь нет необработанных зависимостей.

    **Выходные данные** ![Выходной текст после проверки зависимостей](./media/tutorial-move-region-powershell/all-dependencies-added.png)


## <a name="prepare-resources"></a>Подготовка ресурсов

Обычно необходимо подготовить ресурсы в исходном регионе перед перемещением. Например:

- Чтобы переместить ресурсы без отслеживания состояния, такие как виртуальные сети, сетевые адаптеры, подсистемы балансировки нагрузки и группы безопасности сети Azure, может потребоваться экспортировать шаблон Azure Resource Manager.
- Для перемещения ресурсов с отслеживанием состояния, таких как виртуальные машины Azure и базы данных SQL, может потребоваться запустить репликацию ресурсов из источника в регион назначения.

В этом руководстве, так как мы перемещаем виртуальные машины, необходимо подготовить исходную группу ресурсов, а затем инициировать и зафиксировать перемещение, прежде чем начать подготовку виртуальных машин.

> [!NOTE]
> Если у вас есть целевая группа ресурсов, можно напрямую зафиксировать перемещение для исходной группы ресурсов и пропустить этапы подготовки и начала перемещения.

  
### <a name="prepare-the-source-resource-group"></a>Подготовка исходной группы ресурсов:

1. Подготовка группы ресурсов:

    ```azurepowershell-interactive
    Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource “PSDemoRM”
    ```
    **Выходные данные**

    ![Выходной текст после подготовки исходной группы ресурсов](./media/tutorial-move-region-powershell/prepare-source-resource-group.png)

2. Начало перемещения исходной группы ресурсов.

    ```azurepowershell-interactive
    "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource “PSDemoRM”
    ```
    ![Выходной текст после начала перемещения исходной группы ресурсов](./media/tutorial-move-region-powershell/initiate-move-source-resource-group.png)

3. Зафиксируйте перемещение для исходной группы ресурсов.

    ```azurepowershell-interactive
    Invoke-AzResourceMoverCommit -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource “PSDemoRM”
    ```
    **Выходные данные**

    ![Выходной текст после фиксации исходной группы ресурсов](./media/tutorial-move-region-powershell/commit-move-source-resource-group.png)


### <a name="prepare-vm-resources"></a>Подготовка ресурсов виртуальной машины

После подготовки и перемещения исходной группы ресурсов можно подготовить ресурсы виртуальной машины для перемещения.

1. Проверьте зависимости перед подготовкой ресурсов виртуальной машины.

    ```azurepowershell-interactive
    $resp = Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm') -ValidateOnly
    ```
    **Выходные данные**

    ![Выходной текст после проверки виртуальной машины перед ее подготовкой](./media/tutorial-move-region-powershell/validate-vm-before-move.png)

2. Получите зависимые ресурсы, которые необходимо подготовить вместе с виртуальной машиной.

    ```azurepowershell-interactive
    $resp.AdditionalInfo[0].InfoMoveResource
    ```
    **Выходные данные**

    ![Выходной текст после получения зависимых ресурсов виртуальной машины](./media/tutorial-move-region-powershell/get-resources-before-prepare.png)

3. Начните процесс подготовки для всех зависимых ресурсов.

    ```azurepowershell-interactive
    Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('PSDemoVM','psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg')
    ```
    **Выходные данные**

    ![Выходной текст после инициирования подготовки всех ресурсов](./media/tutorial-move-region-powershell/initiate-prepare-all.png)


    > [!NOTE]
    > В качестве входных параметров для командлета Prepare, а также в командлетах Initiate Move и Commit можно указать идентификатор исходного ресурса вместо имени ресурса. Для этого выполните следующие действия:


    ```azurepowershell-interactive
        Invoke-AzResourceMoverPrepare -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS" -MoveResourceInputType MoveResourceSourceId  -MoveResource $('/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/PSDemoRMS/providers/Microsoft.Network/networkSecurityGroups/PSDemoVM-nsg')
    ```

## <a name="initiate-move-of-vm-resources"></a>Начало перемещения ресурсов виртуальной машины

1. Убедитесь, что ресурсы виртуальной машины находятся в состоянии *Initiate Move Pending* (Ожидание начала перемещения):

    ```azurepowershell-interactive
    Get-AzResourceMoverMoveResource  -SubscriptionId “ xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx “ -ResourceGroupName “RG-MoveCollection-demoRMS” -MoveCollectionName “PS-centralus-westcentralus-demoRMS ”   | Where-Object {  $_.MoveStatusMoveState -eq “InitiateMovePending” } | Select Name
    ```    

    **Выходные данные**

    ![Выходной текст после проверки состояния запуска](./media/tutorial-move-region-powershell/verify-initiate-move-pending.png)

2. Начало перемещения:

    ```azurepowershell-interactive
    Invoke-AzResourceMoverInitiateMove -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’) -MoveResourceInputType "MoveResourceId"
    ```    

    **Выходные данные**

    ![Выходной текст после начала перемещения ресурсов](./media/tutorial-move-region-powershell/initiate-resources-move.png)


## <a name="discard-or-commit"></a>Отмена или фиксация

После первоначального перемещения нужно решить, следует ли фиксировать перемещение или отменить его. 

- **Отмена**. Если вы выполняете тестирование и не хотите фактически перемещать исходный ресурс, вы можете отменить перемещение. При отмене перемещения ресурс возвращается в состояние *ожидания начала перемещения*. При необходимости можно снова запустить перемещение.
- **Фиксация**. Фиксация завершает перемещение в целевой регион. После фиксации исходный ресурс будет находиться в состоянии *ожидания удаления источника*. Затем вы можете его удалить.

### <a name="discard-the-move"></a>Отмена перемещения

Чтобы отменить перемещение, сделайте следующее:

```azurepowershell-interactive
Invoke-AzResourceMoverDiscard -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’) -MoveResourceInputType "MoveResourceId"
```
**Выходные данные**

![Выходной текст после отмены перемещения](./media/tutorial-move-region-powershell/discard-move.png)


### <a name="commit-the-move"></a>Фиксация перемещения

1. Зафиксируйте перемещение, сделав следующее:

    ```azurepowershell-interactive
    Invoke-AzResourceMoverCommit -ResourceGroupName "RG-MoveCollection-demoRMS" -MoveCollectionName "PS-centralus-westcentralus-demoRMS"  -MoveResource $('psdemovm111', 'PSDemoRM-vnet','PSDemoVM-nsg', ‘PSDemoVM’) -MoveResourceInputType "MoveResourceId"
    ```
    **Выходные данные**

    ![Выходной текст после фиксации перемещения](./media/tutorial-move-region-powershell/commit-move.png)

2. Убедитесь, что все ресурсы перемещены в целевой регион:

    ```azurepowershell-interactive
    Get-AzResourceMoverMoveResource  -ResourceGroupName “RG-MoveCollection-demoRMS ” -MoveCollectionName “PS-centralus-westcentralus-demoRMS”   
    ```
    Все ресурсы теперь находятся в состоянии *Delete Source Pending* (Ожидается удаление исходных ресурсов) в целевом регионе.

## <a name="delete-source-resources"></a>Удаление исходных ресурсов

После фиксации перемещения и проверки правильности работы ресурсов в целевом регионе можно удалить каждый исходный ресурс на [портале Azure](../azure-resource-manager/management/manage-resources-portal.md#delete-resources) [с помощью PowerShell](../azure-resource-manager/management/manage-resources-powershell.md#delete-resources) или [Azure CLI](../azure-resource-manager/management/manage-resources-cli.md#delete-resources).

## <a name="next-steps"></a>Дальнейшие действия

Изучив это руководство, вы:

> [!div class="checklist"]
> * Переместили виртуальные машины Azure в другой регион Azure с помощью PowerShell.
> * Переместили ресурсы, связанные с виртуальными машинами, в другой регион.

Теперь попробуйте переместить виртуальные машины Azure с помощью портала.

> [!div class="nextstepaction"]
> [Перемещение виртуальных машин Azure на портале](./tutorial-move-region-virtual-machines.md)


